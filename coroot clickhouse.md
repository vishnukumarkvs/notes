coroot clickhouse
=================

1. Empty query ID path: b[3] >= 0x40
When b[1] == 0 (empty query ID), the packet layout is:
b[0]: 0x01    (packet type)
b[1]: 0x00    (query ID length = 0)
b[2]: 0x01/02 (query kind)
b[3]: ...     (ClientInfo begins here — first field: initial_user VarUInt length)
b[3] is the VarUInt-encoded length of the initial_user string. A database username is almost always short ASCII (< 32 chars). We reject b[3] >= 0x64 (100 decimal, but the commit uses 0x40 = 64). More importantly, if b[3] has the high bit set (>= 0x80), it means the VarUInt is multi-byte, which would mean a string length > 127 — also rejected.
This catches cases where b[1] happens to be 0x00 in a non-ClickHouse binary stream, and b[2] happens to be 1 or 2, but the next byte is far too large for a legitimate username length.
2. UUID query ID path: hyphen checks
When b[1] == 36 (query ID is a UUID string), the packet layout is:
b[0]:  0x01        (packet type)
b[1]:  0x24        (query ID length = 36)
b[2..37]: UUID     (36 bytes, e.g., "e80c819b-c3e3-4f95-80bf-9194fd7235b1")
b[10]: '-'         (hyphen at UUID index 8)
b[15]: '-'         (hyphen at UUID index 13)
b[20]: '-'         (hyphen at UUID index 18)
b[25]: '-'         (hyphen at UUID index 23)
b[38]: 0x01/02    (query kind)
We check that bytes at b[10], b[15], b[20], b[25] are all '-' (ASCII 0x2D). A UUIDv4 string has hyphens at exactly those positions.
This drops the false-positive probability for this branch from ~1/256 to ~1/4.2 billion (4 independent hyphen checks, each with 1/256 chance in random data → (1/256)^4 if we assume uniform distribution, but practically even better since real binary protocols rarely have that exact hyphen pattern).


1. ebpftracer/l7/clickhouse.go (Go — OOM hardening)
- Added safeStr() — bounded string reader that caps VarUInt lengths before allocation
- Added "io" import (for io.ErrUnexpectedEOF)
- Replaced 2 direct r.Str() calls (top-level string + inter-server secret) with safeStr(r, 1024)
- Kept ClientInfo.DecodeAware and Setting.Decode from the library unchanged (Option B approach)



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


Yes, that would help. And it's consistent with how the other DB protocols already handle it.
Looking at the existing patterns in tracing.go:
- PostgresQuery — if t == nil || query == "" { return } (line 179)
- MysqlQuery — if t == nil || query == "" { return } (line 189)
- MongoQuery — if t == nil || query == "" { return } (line 199)
- ClickhouseQuery — if t == nil { return } (line 239) — missing the empty check

Yes, exactly. A single process can:
1. Open listening sockets (e.g., an HTTP server) → fires EventTypeListenOpen
2. Open outbound connections (e.g., making API calls) → fires EventTypeConnectionOpen

You're right — I missed that. attachTlsUprobes reads c.processes[pid] (a map) without holding c.lock, relying on the invariant that it's only called from the single event-loop goroutine. The timer goroutine breaks that — concurrent map read (from timer) and write (e.g., onProcessExit deleting from the map) causes a runtime panic in Go.
The tlsDelayLock I added only protects Process-level fields (tlsDelayTimer, *Checked bools), not the Container-level processes map.
The clean fix: instead of time.AfterFunc calling attachTlsUprobes directly in a new goroutine, have it send the PID to the event loop via a channel:
// Registry gets a new channel
tlsAttachCh chan uint32

// Timer fires → send PID to event loop
p.tlsDelayTimer = time.AfterFunc(wait, func() {
    r.tlsAttachCh <- pid
})

// Event loop select picks it up
case pid := <-r.tlsAttachCh:
    if c := r.containersByPid[pid]; c != nil {
        c.attachTlsUprobes(r.tracer, pid)
    }
This keeps attachTlsUprobes on the event-loop goroutine, preserving the original single-goroutine invariant for c.processes map access. Want me to implement this?

coroot-node-agent on  add-tls-uprobe-instrumentation-delay-1 ☸ kind-kind () ❯ git stash
Saved working directory and index state WIP on add-tls-uprobe-instrumentation-delay-1: ca364ab Add delay timer when attaching go tls uprobes to a process


---------------------------------------------------------------------------------------------------------------

# Overview tab pagination issue


 Thought for 10s, 1.6k tokens
  Prioritizing Tool Usage
  You are incredibly observant, and you are 100% correct.

  While Coroot strictly limits the raw traces list (the waterfall view) to 100 rows, the Summary Table on the Overview tab has NO limit. It is pulling all 50,000 rows into memory and shipping them to your browser in one giant JSON payload. The pagination you see is entirely "client-side"
  (meaning your browser downloaded all 50K rows at once and is just hiding them on different pages).

  This is absolutely causing your massive load times!

  ### Why is this happening?

  In  clickhouse/traces.go , the query that builds that summary table looks like this:

    SELECT ServiceName, SpanName, roundDown(Duration/1000000, @buckets), count(1), countIf(StatusCode = 'STATUS_CODE_ERROR')
    FROM otel_traces
    WHERE ...
    GROUP BY ServiceName, SpanName, DurationBucket

  Notice that there is no  LIMIT  clause on this query. It groups your traffic by every unique  ServiceName  and  SpanName  combination.

  ### The Root Cause: High Cardinality Span Names

  According to OpenTelemetry best practices, a  SpanName  should be a "low cardinality" template. For example, it should be:
   GET /api/users/{user_id}

  However, many default instrumentations (or custom code) accidentally put the actual dynamic data into the Span Name, like this:

  •  GET /api/users/1001
  •  GET /api/users/1002
  •  GET /api/users/1003

  If your system generates 50,000 unique URLs/IDs over an hour, ClickHouse treats them as 50,000 unique  SpanName s. It groups them into 50,000 rows, Coroot calculates the math on all 50K, and your browser chokes trying to load the massive JSON response.

  ### How to fix it:

  1. Fix the Instrumentation (Recommended): The true fix is checking your OpenTelemetry instrumentation (in your application code or Otel collector) to ensure it uses grouped Route Names (like  /api/users/:id ) rather than raw URLs for the  SpanName . If you fix this, those 50,000 rows
  will instantly collapse down to maybe 50 or 100 clean API routes, and the UI will load instantly.
  2. Backend Limit: If you can't fix the instrumentation, we could modify  clickhouse/traces.go  to inject a hard  LIMIT 1000  on the  getTraceSpanStats  query and sort it by the highest volume spans, so the UI never crashes your browser.

  Would you like to explore patching Coroot to enforce a hard limit on that table, or is fixing the OpenTelemetry instrumentation an option for your team?
  
  
 ---------------------------------------------------------------------------------------------------
 
 
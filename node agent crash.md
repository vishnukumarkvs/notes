node agent crash
================

https://github.com/coroot/coroot-node-agent/issues/376


## Core issue

Think attendance register.

Agent asks kernel every 15s: "how long did each process wait for CPU/disk?"

\Kernel clerk goes to copy the process name-tag (16-letter slot) onto a 32-letter form. Old instruction says "always read 32 letters." Safety guard watches and shouts on letter 17: "you're reading past the tag!" and pulls the emergency brake — whole node reboots.

Most name-tags are short, guard never notices. One node had a long name, guard tripped. Fix = tell clerk to stop at the end of the tag, or stop asking for that form.



Both, in stages.
1. Kernel builds struct taskstats (with ac_comm[32]) in kernel memory as the netlink reply (skb).
2. That reply is sent via netlink to userspace; agent's library casts it to a Go struct.

The crash happens at stage 1, before sending — while copying tsk->comm[16] to the in-kernel ac_comm[32].


### Linux version
amazon linux version: 6.12.79-101.147.amzn2023.x86_64

AL2023 kernel6.12 package build
6.12.79-101.147.amzn2023.x86_64 = Amazon Linux 2023, kernel 6.12 variant (not default 6.1).

Shipped: late-April 2026. Evidence in search



###

Almost — not PID, comm (process name).
- src = tsk->comm[16], dst = ac_comm[32]. Copy fails under FORTIFY.
- coroot-node-age in your Oops is the querier (truncated coroot-node-agent, 17 -> 15+NUL), not necessarily the victim. Victim is whichever target PID fillstats was copying at that moment.
- Yes: short comm (<15 chars, early NUL) stops strnlen early — safe. Only long comms (~15-16 chars, NUL at/near byte 16, word-read tries byte 17) trip it. That's why it's random per node.


~~~
A. In-kernel connection-level sampling (biggest lever)
There is currently no sampling anywhere — handle_request/handle_response in ebpftracer/ebpf/l7/l7.c run full protocol detection + up-to-1KB payload copy + bpf_perf_event_output on every frame of every connection on the host. For gRPC/HTTP-2 (Cortex's traffic), that's a per-message cost.
Fix, elegant because it preserves request/response pairing:
- Add a sampling flag to struct connection (ebpftracer/ebpf/tcp/state.c) and roll it once at connection creation via bpf_get_prandom_u32() % N == 0.
- Gate handle_request/handle_response on conn->sampled and early-return otherwise.
- Make N a --l7-sampling flag written into a small BPF map (readable by the tracer).
- Add an l7_events_dropped kernel-side counter so the cost of sampling is visible.
Result: L7 stays fully functional on sampled connections, and per-syscall cost (which is what pushed node load) drops by N for the whole node — for both the apps and the agent. Sampling ratio can degrade gracefully on hot pools.
B. Shrink MaxPayloadSize from 1024 to ~256 bytes
MaxPayloadSize = 1024 (ebpftracer/tracer.go) drives four costs at once:
- 1KB copied per syscall in the hot path,
- struct l7_event is ~1100 bytes written into the perf ring on every event,
- active_l7_requests holds up to 10240 × 1KB payloads in kernel memory (~10 MB),
- per-event allocation + copy on the userspace side.
Protocol detection only needs headers — the HTTP/2 frame header is 9 bytes, an HTTP request line fits in the first couple of hundred bytes. Reducing to 256 cuts all four by ~4x. Make it a flag (--l7-payload-size) so environments that rely on deep payload parsing (MySQL query text, Redis args) can raise it.
C. Scope L7 tracing to monitored processes/containers
Right now the syscall tracepoints (sys_enter_write/read/...) run for every process on the node, including host daemons, kubelet, node-exporter, etc. Add a kernel-side pid/cgroup allowlist (populated from the existing containers registry, piggybacking on the process-start/exit event stream in proc.c), and early-return in trace_enter_write/read when the pid isn't traced. On a dedicated Cortex pool this eliminates the host/system noise entirely. Bigger change than A/B, but removes a whole class of tracepoint work.
D. Switch L7 delivery from perf ring buffer to BPF ringbuf
l7_events uses BPF_MAP_TYPE_PERF_EVENT_ARRAY read via perf.Reader with WakeupEvents: 100 (tracer.go). BPF_MAP_TYPE_RINGBUF + bpf_ringbuf_reserve/commit is measurably cheaper at high throughput (no per-event copy, better batching). Isolated refactor: mostly l7.c map/emit code + the tracer.go reader (cilium/ebpf has a ringbuf.Reader with the same goroutine shape).
E. Userspace: kill per-event allocation churn
The event pipeline is one select loop (Registry.handleEvents) fed by perf-reader goroutines, with per-event allocations at every step:
- runEventsReader calls bytes.NewBuffer + binary.Read + copies a payload slice (with MaxPayloadSize bytes) per event;
- L7Stats.get + GetMetricWithLabelValues(...) do map lookups per request in containers/l7.go.
Fixes: sync.Pool for Event/RequestData; direct binary.LittleEndian field reads instead of bytes.NewBuffer; drain the channel in batches; cache the resolved *L7Metrics on the connection (the destination is already resolved in onConnectionOpen/onL7Request in container.go) so the hot loop is just c.Inc() + Observe().
F. Self-adaptive rate limiting (the "set-and-forget" version)
A small userspace controller that watches L7 event rate / perf LostSamples; when sustained rate exceeds a configurable budget, it writes a lower sampling ratio into the kernel map from (A), and exposes node_agent_l7_sampling_ratio + dropped counters. Effect: L7 is enabled by default everywhere, and on a 500M-series node it transparently degrades to sampled mode instead of exploding.
Ranking for your scenario (where the node load is the problem): A > B > C directly cut the in-kernel per-syscall cost that's attributed to your Cortex apps — that's what produced the load-average spike. D and E reduce the agent's own footprint and improve throughput ceilings; F is the production-grade wrapper around A.
The tradeoff to communicate: with A/F you trade request count completeness for cost (sampled visibility), with B you shorten payloads (headers only), and C is lossless but scoped. None of them remove L7.
I could implement A + the --l7-sampling flag + drop counter as a self-contained first PR — it's the highest-value, lowest-risk change and directly targets what you measured. Want me to sketch that out in code, or start with something else (e.g., B's payload-size flag)?
~~~


~~~
The current L7 tracer is an all-or-nothing pipeline in ebpftracer/ebpf/l7/l7.c:
1. Every socket read/write syscall on the host is sniffed. When L7 is enabled, the agent attaches 9 syscall tracepoints (write, writev, sendto, sendmsg, sendmmsg, read, readv, recvfrom, recvmsg + exits). On every one of them, the eBPF program copies up to 1024 bytes of payload and runs it through 15+ protocol detectors in sequence — and DNS is literally the second-to-last check in the chain (is_dns_request is line 412 of l7.c, checked last, right before FoundationDB). So HTTP, Postgres, Redis, Kafka, etc. traffic all pay the DNS-sniffing cost too, and DNS traffic pays for every other detector first.
2. Every matched event is ~1.1 KB in the ring buffer. The l7_event struct embeds a fixed char payload[1024] (see l7.c:62-75), and send_event() does bpf_perf_event_output(..., sizeof(*e)). A 40-byte DNS query still burns ~1,072 bytes of the l7_events per-CPU perf ring (32 pages ≈ 128 KB/CPU, tracer.go:324). Overflow → LostSamples → "lost samples" logged.
3. One shared userspace path. All events (proc, TCP, file, L7) funnel into a single events channel (registry.go:115, buffer 10000) processed by one handleEvents loop. High L7 volume backs everything up.
~~~


~~~
Here's the flow in plain terms. Think of the agent as a mailroom inspector that sits inside the node and sniffs every letter every program sends or receives over the network.
Setup: 9 mail slots
The node has 9 "mail slots" the inspector watches — they're just different ways letters get passed to the network (like courier types):
- Slots for sending: write, writev, sendto, sendmsg, sendmmsg
- Slots for receiving: read, readv, recvfrom, recvmsg
A program picks one slot to send a letter. If it uses sendto, the letter goes through the sendto slot only — it never also passes write or writev. So the inspector never sees the same letter 9 times. (Vectored slots like writev are "send in parts" — the inspector glues up to 32 pieces together into one letter, which is naturally heavier.)
Sending a request (e.g., an app calling a database)
1. The app calls write(fd, data, size) → the letter enters the write slot.
2. The inspector photocopies up to 1 KB of the letter and starts guessing what it is. It flips to page 1 and checks: "Is this HTTP? Postgres? Redis? … DNS?" — it runs through 16 protocol checks in order, and DNS is near the end of the queue. So an HTTP letter still gets sniffed by the 15 other checks first, and each check is wasted work.
3. When it recognizes the protocol, it files a photocopy into a filing cabinet (active_l7_requests), noting "app X sent a Postgres query, waiting for the reply."
Receiving the reply
When the reply arrives (read slot):
1. The inspector photocopies the reply, opens the filing cabinet, and matches the reply to the earlier request (for DNS it matches by the transaction ID in the header).
2. It then photocopies the request copy out of the cabinet again, glues request + reply info into one report card (~1.1 KB — because the report card has a fixed 1 KB photo slot even if the actual message was 50 bytes).
3. That fat report card goes into the outbox (a per-CPU mailbox, ~128 KB per CPU). A background mailroom worker (the agent's Go code) is supposed to empty the outbox into its inbox and process every card.
Why the mailbox overflows on busy nodes
- Every photo the inspector takes makes the letter 4× bigger on paper: capture → file copy → re-photocopy → outbox.
- The agent's mailroom worker must parse every card: read 1 KB, guess protocol again user-side, build prometheus counters, generate traces. Its CPU is the throughput limit.
- If cards pile in faster than the worker can process them → the mailbox fills → "lost samples" → the inspector throws away letters (dropped metrics).
- On a busy Cortex pool, the letters are mostly HTTP/API traffic — DNS is only a small fraction. But since DNS letters ride in the same mailbox, they get dropped too, and the extra CPU load from sniffing every socket saturates the node.
The fix idea (DNS-only tracer)
Imagine a redesign where DNS gets its own tiny mail slot:
- The DNS inspector only watches sendto/recvfrom going to port 53 (DNS's home port) — the only two slots DNS actually uses (UDP).
- Instead of photocopying 1 KB, it reads just the 12-byte header (the "to/from + size" line on the envelope).
- Instead of a 1 KB report card, it writes a ~64-byte sticky note: "query example.com → 1.2.3.4, 3 ms, OK."
- It has its own small mailbox and its own worker.
So compared to now: 9 slots → 2 slots, 1 KB copies → 12 bytes, 1 KB report cards → 64 bytes, one shared mailbox → dedicated one. The remaining load is proportional to actual DNS traffic — which on normal k8s nodes (kube-dns/NodeLocalDNSCache absorbing repeats) is a tiny drop compared to the HTTP flood. That's why DNS-only should not saturate Cortex pools in normal operation — you'd need an absurdly chatty workload (200–300× the old DNS rate) to recreate today's problem.
~~~
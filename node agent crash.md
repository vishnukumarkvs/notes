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
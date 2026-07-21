coroot node agent
=================

/usr/bin/coroot-node-agent
  CVE-2026-41567 (7.2) in github.com/docker/docker
  CVE-2026-41568 (6.1) in github.com/docker/docker
  CVE-2026-42306 (7.2) in github.com/docker/docker
  CVE-2026-50195 (5.6) in github.com/containerd/containerd
  CVE-2026-53489 (7.1) in github.com/containerd/containerd
  CVE-2026-53492 (8.4) in github.com/containerd/containerd
  CVE-2026-42154 (7.5) in github.com/prometheus/prometheus, fixed in 0.305.2
  CVE-2026-40179 (5.3) in github.com/prometheus/prometheus, fixed in 0.311.2-0.20260410083055-07c6232d159b
  CVE-2026-42151 (7.5) in github.com/prometheus/prometheus, fixed in 0.311.3
  CVE-2026-44903 (5.1) in github.com/prometheus/prometheus, fixed in 0.311.3
  CVE-2026-33814 (7.5) in golang.org/x/net, fixed in 0.53.0
  CVE-2026-25680 (6.5) in golang.org/x/net, fixed in 0.55.0
  CVE-2026-25681 (6.1) in golang.org/x/net, fixed in 0.55.0
  CVE-2026-27136 (6.1) in golang.org/x/net, fixed in 0.55.0
  CVE-2026-39821 (9.6) in golang.org/x/net, fixed in 0.55.0
  CVE-2026-42502 (6.1) in golang.org/x/net, fixed in 0.55.0
  CVE-2026-42506 (6.1) in golang.org/x/net, fixed in 0.55.0
  CVE-2026-33726 (4.3) in github.com/cilium/cilium, fixed in 1.17.14
  CVE-2026-41520 (4.4) in github.com/cilium/cilium, fixed in 1.17.15
  CVE-2026-53935 (6.9) in github.com/cilium/cilium, fixed in 1.17.16
  CVE-2026-2303 (6.9) in go.mongodb.org/mongo-driver, fixed in 1.17.7
  CVE-2026-39882 (5.3) in go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp, fixed in 1.43.0
  CVE-2026-39883 (7.3) in go.opentelemetry.io/otel/sdk, fixed in 1.43.0
  CVE-2026-46680 (7.3) in github.com/containerd/containerd, fixed in 1.7.32
  CVE-2026-47262 (5.3) in github.com/containerd/containerd, fixed in 1.7.33
  CVE-2026-53488 (8.7) in github.com/containerd/containerd, fixed in 1.7.33
  CVE-2026-33186 (9.1) in google.golang.org/grpc, fixed in 1.79.3
  CVE-2026-33997 (6.8) in github.com/docker/docker, fixed in 29.3.1
  CVE-2026-34040 (7.8) in github.com/docker/docker, fixed in 29.3.1


Done. Nothing staged/committed. Here's where things landed:

Applied (working tree only, not committed)

┌─────────────────────────────┬────────────────────────────────────┬────────┐
│         Dependency          │           Before → After           │ Result │
├─────────────────────────────┼────────────────────────────────────┼────────┤
│ docker/docker               │ v27.4.0 → v28.5.2                  │ clean  │
├─────────────────────────────┼────────────────────────────────────┼────────┤
│ containerd/containerd       │ v1.7.29 → v1.7.33                  │ clean  │
├─────────────────────────────┼────────────────────────────────────┼────────┤
│ golang.org/x/net            │ v0.46.0 → v0.55.0                  │ clean  │
├─────────────────────────────┼────────────────────────────────────┼────────┤
│ go.mongodb.org/mongo-driver │ v1.14.0 → v1.17.7                  │ clean  │
├─────────────────────────────┼────────────────────────────────────┼────────┤
│ otel/sdk + otlptracehttp    │ v1.40.0/v1.28.0 → v1.43.0          │ clean  │
├─────────────────────────────┼────────────────────────────────────┼────────┤
│ google.golang.org/grpc      │ v1.72.1 → v1.80.0 (auto, indirect) │ clean  │
└─────────────────────────────┴────────────────────────────────────┴────────┘


above clean

So prometheus/prometheus is left at v0.51.2 for now, still vulnerable to CVE-2026-42154/40179/42151/44903. It's coupled to the Cilium problem, not separable from it — fixing one properly requires solving both together (most likely: rewriting containers/cilium.go's eBPF backend-map reads against Cilium 1.18+/1.19+'s restructured LB map API, which also happens to bring client-go compatibility).


labels.labels changed in new prometheus package

remoed V2 version ebpf maps. su
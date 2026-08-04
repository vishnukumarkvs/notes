moby
====

Docker — this is the one I want to flag before touching: moby deprecated github.com/docker/docker as of v29; the fixed release (v29.3.1) isn't even resolvable via go get (they renamed git tags to docker-vX.Y.Z, which breaks Go's module tag matching). The supported replacement is two new, independently-versioned modules — github.com/moby/moby/client and github.com/moby/moby/api. I checked: this isn't a rename, it's an API change — e.g. ContainerInspect(ctx, id) becomes ContainerInspect(ctx, id, options ContainerInspectOptions) (ContainerInspectResult, error), and the result type is a new struct, not the old types.ContainerJSON. It touches containers/dockerd.go and windows/containers/docker.go, including the port/network/env/mount field-mapping logic that feeds container metadata for the whole agent.


go.mod: removed github.com/docker/docker entirely, added github.com/moby/moby/client@v0.5.1 + github.com/moby/moby/api@v1.55.0 — the CVE-fixed release line moby now ships (the old module is deprecated and its v29.3.1 fix isn't even resolvable via go get, since moby switched git tag naming and stopped publishing to that path).

containers/dockerd.go (Linux client) and windows/containers/docker.go — mechanical but real API updates:
- client.NewClientWithOpts → client.New (old name deprecated)
- Ping(ctx) + separate NegotiateAPIVersion(ctx) call → single Ping(ctx, client.PingOptions{NegotiateAPIVersion: true})
- ContainerInspect(ctx, id) → ContainerInspect(ctx, id, client.ContainerInspectOptions{}), returns a wrapper struct (.Container holds what used to be the direct return value)
- ContainerList(ctx, container.ListOptions{}) → ContainerList(ctx, client.ContainerListOptions{}), returns .Items instead of a bare slice
- ContainerStatsOneShot → ContainerStats(ctx, id, client.ContainerStatsOptions{})
- network.Port.Proto() now returns a typed network.IPProtocol, compared against network.TCP instead of the string literal "tcp"
- PortBinding.HostIP and EndpointSettings.{IPAddress,GlobalIPv6Address} changed from string to netip.Addr — updated the string-concat/parsing call sites accordingly (.String(), .IsValid())

I verified correctness two ways: an isolated smoke-test package built cleanly against the real moby/moby/client/api modules for the Linux code path, and GOOS=windows GOARCH=amd64 go build ./windows/containers/... compiled clean with zero errors for the Windows path. (I couldn't run a full native Linux build here — this Mac can't cross-compile the NVML/cgo pieces elsewhere in the repo, a pre-existing limitation unrelated to this change, confirmed by reproducing the same failure against the untouched original code.)


	if _, err := c.Ping(ctx, client.PingOptions{NegotiateAPIVersion: true}); err != nil {


client.PingOptions is the new options struct for the Ping method — in the old docker/docker client, Ping took no arguments and API-version negotiation was a separate call (c.NegotiateAPIVersion(ctx), which you had to remember to invoke afterward). The new moby/moby/client merged both into one call.

From the struct definition:

type PingOptions struct {
    // Queries the server and downgrades the client's API version to match
    // if the server is older; caps at the client's max version if the
    // server is newer. Skipped if a version is already negotiated or
    // manually pinned (via WithAPIVersion/DOCKER_API_VERSION).
    NegotiateAPIVersion bool

    // Forces re-negotiation even if it already happened or a version
    // was pinned. Only matters if NegotiateAPIVersion is also true.
    ForceNegotiate bool
}

And what Ping does with it:

func (cli *Client) Ping(ctx context.Context, options PingOptions) (PingResult, error) {
    if !options.NegotiateAPIVersion {
        return cli.ping(ctx)          // just ping, no negotiation
    }
    if cli.negotiated.Load() && !options.ForceNegotiate {
        return cli.ping(ctx)          // already negotiated, skip
    }
    // ...negotiate API version based on the ping response...
}

So client.PingOptions{NegotiateAPIVersion: true} tells the client: "while you're pinging the daemon to check it's alive, also read its reported API version off that same response and pick the highest version both sides support." That's exactly what the old two-line sequence (c.Ping(ctx) then c.NegotiateAPIVersion(ctx)) did — it's now one call instead of two, and this is why I removed the separate c.NegotiateAPIVersion(ctx) line in the migration.


				if ipp, err := netaddr.ParseIPPort(b.HostIP.String() + ":" + b.HostPort); err == nil {

type PortBinding struct {
    HostIP   netip.Addr `json:"HostIp"`   // was: string
    HostPort string     `json:"HostPort"` // unchanged
}

# Containerd migration (v1 to v2)

Containerd migration is done (uncommitted). Summary:

go.mod: github.com/containerd/containerd v1.7.33 → github.com/containerd/containerd/v2 v2.2.6 — this fixes CVE-2026-50195/53489/53492, which were never patched on the v1.7.x branch. I picked v2.2.6 over the newer v2.3.x because v2.3.x bumps the minimum Go requirement to 1.26.3; v2.2.6 still only needs Go 1.25.0, so your toolchain requirement doesn't change.

containers/containerd.go: same shape as the docker migration but smaller — no client method signatures changed, just import paths:
- github.com/containerd/containerd → .../v2/client (containerd.Client→client.Client, containerd.New→client.New, containerd.WithDefaultNamespace→client.WithDefaultNamespace, containerd.WithTimeout→client.WithTimeout)
- github.com/containerd/containerd/oci → .../v2/pkg/oci
- github.com/containerd/containerd/pkg/cri/constants no longer exists publicly in v2 (moved to internal/), so its one constant, K8sContainerdNamespace = "k8s.io", is now a local literal in this file

ContainerService().Get(...), c.Labels, c.Image, c.Spec, c.Extensions, and oci.Spec all kept the same shape, so the rest of ContainerdInspect is untouched.

Verified with an isolated smoke-test package built against the real containerd/v2 module (clean build), plus confirmed no other errors surface once go mod tidy resolved the new dependency graph (only the pre-existing, unrelated NVML cross-compile limitation remains on this Mac, same as before).

One side effect worth flagging: golang.org/x/text got pulled to v0.38.0 as a transitive dependency of containerd/v2 — still short of the v0.39.0 that fixes CVE-2026-56852, so that CVE isn't resolved yet. It's on the list for the remaining "simple bumps" pass whenever you want to do those.

Nothing committed — ready for you to review and commit.


┌────────────────────────────────────────────┬──────────────────────────────┬──────────────────────────────────────────────────────────┐
│                    CVE                     │           Package            │                          Status                          │
├────────────────────────────────────────────┼──────────────────────────────┼──────────────────────────────────────────────────────────┤
│ CVE-2026-41567, 41568, 42306, 33997, 34040 │ docker/docker                │ ✅ Done (uncommitted) — migrated to moby/moby/client+api │
├────────────────────────────────────────────┼──────────────────────────────┼──────────────────────────────────────────────────────────┤
│ CVE-2026-50195, 53489, 53492               │ containerd/containerd        │ ❌ Not done — needs migration, same shape as docker      │
├────────────────────────────────────────────┼──────────────────────────────┼──────────────────────────────────────────────────────────┤
│ CVE-2026-56852                             │ golang.org/x/text            │ ❌ Not done — plain version bump                         │
├────────────────────────────────────────────┼──────────────────────────────┼──────────────────────────────────────────────────────────┤
│ CVE-2026-46600                             │ golang.org/x/net             │ ❌ Not done — plain version bump                         │
├────────────────────────────────────────────┼──────────────────────────────┼──────────────────────────────────────────────────────────┤
│ CVE-2026-41178                             │ go.opentelemetry.io/otel     │ ❌ Not done — plain version bump                         │
├────────────────────────────────────────────┼──────────────────────────────┼──────────────────────────────────────────────────────────┤
│ CVE-2026-54332, 54345                      │ github.com/gopacket/gopacket │ ❌ Not done — plain version bump                         │
├────────────────────────────────────────────┼──────────────────────────────┼──────────────────────────────────────────────────────────┤
│ GHSA-hrxh-6v49-42gf                        │ google.golang.org/grpc       │ ❌ Not done — plain version bump                         │
└────────────────────────────────────────────┴──────────────────────────────┴──────────────────────────────────────────────────────────┘


# fix other cve

All five remaining CVEs are now addressed (uncommitted, as before):

┌───────────────────────┬───────────────────────────────────────────────────────────────────────┬──────────────────────┐
│          CVE          │                                Package                                │      Old → New       │
├───────────────────────┼───────────────────────────────────────────────────────────────────────┼──────────────────────┤
│ CVE-2026-56852        │ golang.org/x/text                                                     │ 0.37.0 → 0.39.0      │
├───────────────────────┼───────────────────────────────────────────────────────────────────────┼──────────────────────┤
│ CVE-2026-46600        │ golang.org/x/net                                                      │ 0.55.0 → 0.56.0      │
├───────────────────────┼───────────────────────────────────────────────────────────────────────┼──────────────────────┤
│ CVE-2026-41178        │ go.opentelemetry.io/otel (+ trace/sdk/metric/otlptrace/otlptracehttp) │ 1.43.0 → 1.44.0      │
├───────────────────────┼───────────────────────────────────────────────────────────────────────┼──────────────────────┤
│ CVE-2026-54332, 54345 │ github.com/gopacket/gopacket                                          │ 1.3.2-pseudo → 1.6.1 │
├───────────────────────┼───────────────────────────────────────────────────────────────────────┼──────────────────────┤
│ GHSA-hrxh-6v49-42gf   │ google.golang.org/grpc                                                │ 1.80.0 → 1.82.1      │
└───────────────────────┴───────────────────────────────────────────────────────────────────────┴──────────────────────┘

# Cluster agent bump cve

❯ pwd
/Users/kvsvishnukumar/VishnuKvs/Workspace/myrepos/coroot-cluster-agent



/usr/bin/coroot-cluster-agent
  GHSA-gcjh-h69q-9w9g (6.3) in github.com/google/cel-go, fixed in 0.29.0
  CVE-2026-56852 (7.5) in golang.org/x/text, fixed in 0.39.0
  CVE-2026-46600 (7.5) in golang.org/x/net, fixed in 0.56.0
  GHSA-hrxh-6v49-42gf (8.8) in google.golang.org/grpc, fixed in 1.82.1


## Cluster agent crash

Yes, it's clean — I verified the YAML round-trip works correctly, with one important catch that I also had to fix.

The discovery.Configs round-trip is safe by design: Configs (the type behind ServiceDiscoveryConfigs) implements matched MarshalYAML/UnmarshalYAML via a registry mechanism, dispatching each entry by its discovery type (static_configs, kubernetes_sd_configs, etc.). I proved this empirically with an isolated smoke test against the real prometheus@v0.311.3 module, using the exact config shapes from scraper.go — static targets, k8s pod discovery, relabel/metric-relabel rules, and remote-write TLS/headers all survived the round-trip intact.

But the round trip caught a second, pre-existing bug: k8sDiscovery() builds &kubernetes.SDConfig{Role: ..., Selectors: ...} directly, leaving HTTPClientConfig as the Go zero value instead of kubernetes.DefaultSDConfig's proper default. Once actually validated via config.Load() (which the raw-struct approach was silently skipping before), Prometheus rejects that as "custom HTTP client configuration ... provide 'api_server'". Fixed by sourcing the SD config from kubernetes.DefaultSDConfig.


v1's client.New() used grpc.WithBlock() + a blocking dial with a real timeout — so if a socket was wrong/unreachable, New() itself returned an error, and the for _, socket := range sockets loop correctly moved on to the next candidate.

v2's client.New() switched to grpc.NewClient(...) (client.go:166) — grpc-go's newer lazy, non-blocking constructor. It never dials during construction; the connection is only established on the first real RPC. That means client.New() now returns err == nil for any syscall-valid socket path, whether or not a daemon is actually listening there. So the loop always picks the first socket in the list (/var/snap/microk8s/common/run/containerd.sock) and logs "using" it — even on hosts where that's not the real socket — and the actual failure only surfaces later, at the first real RPC (ContainerdInspect), which is exactly the timeout you're seeing.






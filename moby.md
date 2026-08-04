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




moby
====

Docker — this is the one I want to flag before touching: moby deprecated github.com/docker/docker as of v29; the fixed release (v29.3.1) isn't even resolvable via go get (they renamed git tags to docker-vX.Y.Z, which breaks Go's module tag matching). The supported replacement is two new, independently-versioned modules — github.com/moby/moby/client and github.com/moby/moby/api. I checked: this isn't a rename, it's an API change — e.g. ContainerInspect(ctx, id) becomes ContainerInspect(ctx, id, options ContainerInspectOptions) (ContainerInspectResult, error), and the result type is a new struct, not the old types.ContainerJSON. It touches containers/dockerd.go and windows/containers/docker.go, including the port/network/env/mount field-mapping logic that feeds container metadata for the whole agent.



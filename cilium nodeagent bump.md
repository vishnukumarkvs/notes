cilium nodeagent bump
=====================

Whats a cilium loadbalancer map - lbmap?


What Cilium's load-balancer map (lbmap) is

Cilium is a Kubernetes CNI that replaces kube-proxy's iptables/IPVS-based Service routing with eBPF. When a pod calls connect() to a Kubernetes Service's ClusterIP:Port, Cilium's eBPF programs intercept that connection in the kernel and transparently rewrite (DNAT) the destination to one of the Service's real backend Pod IPs — before a packet ever leaves the host. The application only ever sees the virtual Service address it dialed; the kernel silently redirected it.

To make that work, Cilium maintains two separate eBPF map families, pinned to the host's bpffs (/sys/fs/bpf/tc/globals/...) so they persist and are readable outside Cilium's own process:

- ctmap (conntrack maps) — Cilium's own per-flow connection tracking table (cilium_ct4_global/cilium_ct6_global). For every flow it load-balances, it stores an entry keyed by a 5-tuple (src/dst IP+port, protocol) that records which backend was picked for that specific connection — as a BackendID, just a small integer index.
- lbmap (load-balancer maps) — the actual backend pool: a table mapping BackendID → real Pod IP:Port. This is the "answer key" that turns the integer index into an actual address. This is the package that moved (pkg/maps/lbmap → pkg/loadbalancer/maps) and dropped its V2 map format in Cilium 1.18, which is what broke containers/cilium.go.

How coroot-node-agent uses it

containers/cilium.go:81-172 (lookupCiliumConntrackTable / lookupCilium4 / lookupCilium6) exists to solve one specific problem: coroot-node-agent's own eBPF tracer captures TCP connect() events independently of Cilium, recording src and dst as the application saw them. In a Cilium-managed cluster, dst is very often just the virtual Service ClusterIP, not the real pod that ended up serving the connection — Cilium DNAT'd it in the kernel, and the app never knew. If Coroot reported connections by that raw dst, its dependency graph would show every service call terminating at an opaque, meaningless Service IP instead of showing which pod actually talked to which pod.

So node-agent reaches into Cilium's own maps to recover the truth, in containers/container.go:636 (onConnectionOpen):

if a := lookupCiliumConntrackTable(src, dst); a != nil {
    actualDst = *a
}

The lookup itself, e.g. lookupCilium4 (containers/cilium.go:93-132):

1. Query Cilium's conntrack map with a key built from the connection tuple — note it swaps ports (SourcePort: dst.Port(), DestPort: src.Port()) because Cilium records these entries from the reverse/return-path perspective, and sets TUPLE_F_SERVICE to signal a service-direction lookup. This returns a *ctmap.CtEntry containing BackendID.
2. Query the lbmap backend table with that BackendID (lbmap.NewBackend4KeyV3(...)) to get a *lbmap.Backend4ValueV3 — the real backend Pod's IP and port.
3. Convert it out of eBPF's on-wire byte representation (.ToHost(), then .AsNetIP() post-1.18) into a plain netaddr.IPPort and hand it back.

init() (containers/cilium.go:37-81) does this setup once at agent startup: it tries to bpf.OpenMap(...) the four pinned map files directly off the host's bpffs (proc.HostPath(...) because node-agent runs in a container with the host filesystem bind-mounted elsewhere). If Cilium isn't installed, these opens just fail and get logged — ciliumCt4/backends4Map stay nil, and the lookup functions become silent no-ops. So this entire mechanism is opportunistic: it only activates when Cilium is actually present.

Why this is the thing that keeps breaking on Cilium bumps

Unlike a normal integration, node-agent isn't calling a stable, versioned Cilium client API — it's directly importing Cilium's own internal Go packages (pkg/maps/ctmap, pkg/loadbalancer/maps, pkg/bpf) to hand-decode the raw memory layout of Cilium's private eBPF maps. Those packages carry no compatibility guarantee between releases — Cilium is free to rename, move, or restructure them at will (as it did with the lbmap package path and the V2 format in 1.18). That's precisely why every Cilium version bump forces a fresh look at this one file.


# Bits

This is the package that moved (pkg/maps/lbmap → pkg/loadbalancer/maps) and dropped its V2 map format in Cilium 1.18, which is what broke containers/cilium.go.



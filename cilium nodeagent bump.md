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


How They Work Together
When a pod sends a request to a Kubernetes Service, both maps are heavily involved in the datapath:

Load Balancing (lbmap): The packet arrives at the eBPF program. The eBPF code looks up the destination IP in the lbmap. It recognizes it as a VIP, selects a backend ID, and rewrites the packet's destination IP to the actual Pod IP.

Connection Tracking (ctmap): Before the packet is sent out, eBPF logs this new connection tuple in the ctmap, along with a flag indicating that this connection underwent a Load Balancer translation (and records the necessary RevNAT ID).

Return Traffic: When the backend pod replies, eBPF checks the ctmap first. It sees this is an established connection that was load-balanced, looks up the RevNAT ID, and rewrites the Source IP back to the original VIP before sending it to the client.


Is 4/6 IPv4/IPv6? Yes — Backend4Map* handles IPv4 backends, Backend6Map* handles IPv6 backends. Cilium keeps two completely separate map families because the address field width differs (4 bytes vs 16 bytes), so it can't share one map schema between them.


Concretely, comparing the actual Go structs across versions:

┌─────────────────────────────┬───────────────────────────────────────────────┬──────────────────────────────────────────┐
│                             │                Key (BackendID)                │               Value fields               │
├─────────────────────────────┼───────────────────────────────────────────────┼──────────────────────────────────────────┤
│ V1 — cilium_lb4_backends    │ uint16 (max 65,535 backends cluster-wide)     │ Address, Port, Proto, Flags              │
├─────────────────────────────┼───────────────────────────────────────────────┼──────────────────────────────────────────┤
│ V2 — cilium_lb4_backends_v2 │ full uint32 BackendID (fixed the 65k ceiling) │ Address, Port, Proto, Flags (same as V1) │
├─────────────────────────────┼───────────────────────────────────────────────┼──────────────────────────────────────────┤
│ V3 — cilium_lb4_backends_v3 │ same uint32 key as V2                         │ adds ClusterID (uint16) and Zone (uint8) │
└─────────────────────────────┴───────────────────────────────────────────────┴──────────────────────────────────────────┘

- V1→V2 fixed a real scaling limit: a uint16 ID caps you at 65,535 total backends across the whole cluster.
- V2→V3 added ClusterID (for Cluster Mesh — multi-cluster service backends, where an IP alone isn't globally unique once you're routing across clusters) and Zone (for topology/zone-aware routing — lets the datapath prefer a same-AZ backend to cut cross-zone traffic cost).


Is the old one really removed, not just deprecated? Yes, and I checked this rather than assuming: in pkg/loadbalancer/maps/lbmaps.go at v1.20.0, NewBackend4Map() (the function Cilium's own agent calls to actually create/populate its backend map) hardcodes Backend4MapV3Name — it never touches V1 or V2 anymore. 


in cilim 18 version, V2 support is gone,  struct removed entirely

Before (Cilium ≤1.18.12), CtEntry looked like:
type CtEntry struct {
    Reserved0 uint64 `align:"reserved0"`
    BackendID uint64 `align:"backend_id"`
    Packets   uint64 `align:"packets"`
    ...
}
Two separate, explicitly named uint64 fields. e.BackendID read the second 8-byte slot directly.

After (Cilium ~1.19+), it became:
type CtEntry struct {
    Union0   [2]uint64 `align:"$union0"`
    Packets  uint64    `align:"packets"`
    ...
}


 ok, now explain this line. backendIP, _ := netaddr.FromStdIP(backend.GetAddress().AsNetIP()) why new AsNetIp() ?
 
 What changed: backend.GetAddress() used to return a plain net.IP (the standard library type). In Cilium 1.18+, Backend4ValueV3.GetAddress() returns a different type: cmtypes.AddrCluster:

type AddrCluster struct {
    addr      netip.Addr
    clusterID uint32
}

This is a compound type introduced for Cluster Mesh (multi-cluster support) — it bundles the raw address together with a clusterID, because in a multi-cluster setup a bare IP isn't necessarily globally unique;



Spec.RewriteConstants change

# PowerDNS

## Three pieces

1. PowerDNS Recursor (pdns-recursor)
   - A recursive resolver. It answers client queries by walking the DNS tree (root → TLD → authoritative) and caches results.
   - Used when you want a caching/recursive DNS server for clients (like 8.8.8.8-style behavior), not hosting your own zones.

2. PowerDNS Authoritative Server (pdns_server)
   - Hosts your own DNS zones (A, AAAA, MX, TXT, etc.) and answers authoritatively.
   - It does not recurse for unknown names (unless configured with a recursor/forwarder).

3. dnsdist (PowerDNS load balancer / DNS proxy)
   - Sits in front of one or more backends (authoritative servers and/or recursors).
   - Routes, balances, filters, rate-limits, caches, and protects DNS traffic.
   - Often used as the public-facing DNS entry point, sending queries to backend authoritative/recursor instances.

Typical architecture:
Clients → dnsdist (load balancer/proxy) → PowerDNS Recursor (recursive) and/or Authoritative Server (hosted zones)

So: Recursor = recursive resolver; Authoritative = zone hoster; dnsdist = load balancer/proxy. "Resolver" alone is the general concept.

## Dig (useful commands)

Basic record lookups:

```
dig f5.com                  # A record (default)
dig f5.com AAAA             # IPv6
dig f5.com MX               # mail servers
dig f5.com TXT              # TXT (SPF, verification, etc.)
dig f5.com NS               # authoritative nameservers
dig f5.com SOA              # zone SOA
dig f5.com ANY              # (often blocked/truncated by servers)
```

See the full resolution path (recursive walk):

```
dig +trace f5.com           # root -> TLD -> authoritative (how a Recursor walks the tree)
```

Query a specific resolver:

```
dig @8.8.8.8 f5.com         # Google recursive resolver
dig @1.1.1.1 f5.com         # Cloudflare
dig @<your-recursor-ip> f5.com
```

Reverse lookup (IP -> name):

```
dig -x <ip-address>
```

Useful flags:

```
dig +short f5.com           # just the answer
dig +noall +answer f5.com   # answer section only
dig +stats f5.com           # timing/cache info
```

Check authoritative servers directly:

```
dig f5.com NS +short
dig @<that-ns> f5.com       # query the authoritative directly
```

Caching behavior (TTL):

```
dig f5.com | grep -i "ttl\|answer"   # run twice, watch TTL countdown
```

Read the output:
- flags: qr rd ra  -> recursive answer (resolver walked the tree)
- flags: qr aa     -> authoritative answer (from the zone hoster)
- status: NXDOMAIN -> name does not exist
- status: NOERROR  -> name exists (answer may be empty)
- SERVER: ...#53   -> which resolver answered

Example (f5.com):

```
dig f5.com  -> A 159.60.134.0, TTL ~15s (short, behind load balancer)
dig -x 159.60.134.0 -> NXDOMAIN (no PTR; IP block managed by acorus-networks.net)
```

## PowerDNS CLI tools

PowerDNS ships separate CLI tools per component:

Authoritative Server:

```
pdnsutil  -> manage zones/records
  pdnsutil create-zone example.com
  pdnsutil add-record example.com www A 192.0.2.1
  pdnsutil check-zone example.com
  pdnsutil rectify-zone example.com
  pdnsutil list-zones
  pdnsutil show-zone example.com
pdns_control -> runtime control of pdns_server
  pdns_control reload
  pdns_control cycle
  pdns_control stats
```

Recursor:

```
rec_control -> runtime control of pdns-recursor
  rec_control get qps
  rec_control dump-cache /tmp/cache.txt
  rec_control wipe-cache example.com
  rec_control reload-zones
  rec_control top-remotes
  rec_control quit
```

dnsdist:

```
dnsdist is the daemon; control via its Lua console / Unix socket / HTTP API
  dnsdist -c                 # run with config
  dnsdist -e "showServers()"  # evaluate expression
  # interactive console socket configured in config (controlSocket)
```

Daemons:

```
pdns_server   -> authoritative server daemon
pdns-recursor -> recursive resolver daemon
dnsdist       -> load balancer/proxy daemon
```

Stats/metrics are usually exposed over HTTP (Prometheus/carbon) and queried with curl, not a dedicated CLI.

## F5 Distributed Cloud DNS

F5 Distributed Cloud offers DNS as a Service on a geo-distributed anycast network, with built-in DDoS protection, DNSSEC, and DNS load balancing.

Offerings:
- Primary DNS: replace your DNS servers with F5 XC nameservers (anycast, DDoS protection, DNSSEC).
- Dual-Primary DNS: run two authoritative primaries in parallel (managed programmatically via API).
- Secondary DNS: XC acts as secondary to your existing primary (zone transfers, redundancy, DDoS mitigation).
- Secondary DNS with hidden primary: primary stays hidden; only XC nameservers are public.
- DNS load balancing: DNS-based load balancing (how-to guide available).
- DNSSEC: auto-generates DS record for the parent zone.
- API-driven: DNS Zone API for programmatic management.

Public docs:
- Product page: https://www.f5.com/products/distributed-cloud-services/dns
- Docs quickstart: https://docs.cloud.f5.com/docs-v2/dns-management/quickstart/dns-services
- API docs: https://docs.cloud.f5.com/docs-v2/api (DNS Zone API)
- Community: https://community.f5.com

## DNS delegation

DNS delegation is when a parent zone hands off authority for a subdomain to another set of nameservers.

- The parent zone holds NS records pointing to the authoritative servers for the child zone (plus glue A/AAAA records when the nameserver names live inside the delegated zone).
- The parent no longer answers authoritatively for names in the child zone; it refers queriers to the child's nameservers.
- This is what builds the DNS tree: root (.) -> TLD (com.) -> domain (f5.com.) -> subdomains.

Example flow for www.f5.com:
1. Root (.) delegates com. to the .com servers.
2. com. delegates f5.com. to ns1.f5.com / ns2.f5.com (with glue IPs).
3. f5.com. servers answer authoritatively for www.f5.com.

Key points:
- Delegation != hosting: the parent only knows WHO to ask; the child hosts the actual records.
- Glue records break the circular dependency when a nameserver's name is inside the zone it serves (e.g., ns1.f5.com for f5.com).
- Broken/missing delegation (wrong NS, stale glue, lame delegation) is a common cause of resolution failures.

## Zone transfer

A zone transfer is the mechanism by which a DNS server copies the entire contents of a zone from another server.

- AXFR (full zone transfer): transfers the whole zone in one TCP connection.
- IXFR (incremental zone transfer): transfers only the changes since a given SOA serial.

Typical use: primary -> secondary replication. The secondary pulls the zone from the primary (or gets notified via NOTIFY and then pulls) to stay in sync. This is how F5 Distributed Cloud Secondary DNS keeps its copy up to date from your hidden/primary DNS.

Key points:
- Usually done over TCP (port 53), not UDP.
- Should be restricted by IP/ACL (allow-list) so only authorized secondaries can pull; otherwise anyone can download your entire zone (data leak).
- Triggered by SOA serial increase + NOTIFY, or polling.
- Full zone transfers of large zones can be heavy; IXFR is preferred when supported.

## Primary vs secondary zones (HA / DR)

Primary zone:
- The authoritative, writable copy of a zone. Holds the actual records and the SOA; all edits happen here.

Secondary zone:
- A read-only replica pulled from the primary via zone transfer (AXFR/IXFR). Serves queries authoritatively but cannot be edited directly.

Is it for HA / DR? Yes, largely:
- HA / load distribution: multiple secondaries (often geo-distributed/anycast) serve queries, so one server failure doesn't take the zone down and query load is spread.
- DR: a secondary in another region/datacenter keeps serving if the primary site is lost.
- Performance: secondaries closer to users reduce latency.
- Protection: hiding the primary (hidden primary) so only secondaries are public reduces attack surface.

Caveats:
- This is replication, not automatic failover of writes. If the primary dies, secondaries keep serving (possibly stale) data until promoted (manual or scripted).
- All secondaries depend on the primary for updates; if transfers break, data drifts.
- Not multi-master by default (unlike some cloud DNS offerings that sync via API).

## SOA (Start of Authority)

SOA is a required record at the apex of every DNS zone. It declares who is authoritative and key timing parameters.

An SOA record contains:
- MNAME: primary nameserver for the zone.
- RNAME: admin contact email (with . instead of @).
- Serial: version number of the zone.
- Refresh: how often a secondary checks for updates.
- Retry: retry interval if refresh fails.
- Expire: how long a secondary keeps serving the zone if it can't reach the primary.
- Minimum (TTL): negative caching TTL.

SOA serial is the zone's version counter. Bump it on every edit (commonly YYYYMMDDNN). Secondaries compare their serial to the primary's; if the primary's is higher, they trigger a zone transfer (AXFR/IXFR).

"Like a checkpoint?" Yes conceptually: it's a version marker/checkpoint of the zone's state. Each change increments it; secondaries use it to decide "am I up to date?" It is not a data snapshot itself, just a monotonic version number.

Dig commands (SOA-related):
```
  dig f5.com SOA                      # show SOA record
  dig f5.com SOA +short               # concise SOA
  dig @ns1.f5.com f5.com SOA          # query primary directly
  dig @secondary.f5.com f5.com SOA    # compare serial on secondary
  dig f5.com SOA +noall +answer       # answer section only
  # compare serials between primary and secondary to confirm sync:
  dig @primary f5.com SOA +short
  dig @secondary f5.com SOA +short
  dig +trace f5.com SOA               # full resolution path to SOA
  dig f5.com SOA @8.8.8.8             # via a recursive resolver
```

## SOA MNAME in secondary setups (F5 as secondary)

- If F5 is a secondary, the SOA is copied from the primary via zone transfer.
- The SOA MNAME (primary nameserver field) still points to your primary, not F5.
- F5's nameservers appear in the NS records and serve the zone authoritatively.
- So: SOA MNAME = source of truth (primary); NS set = who serves (includes F5 secondaries).
- Standard behavior keeps the primary's MNAME; some setups rewrite it, but serial comparison is what drives sync.

## Nameservers vs resolvers

- Authoritative nameserver: answers only for zones it hosts. Asked for unrelated names, returns REFUSED/NXDOMAIN (or refers). Does not walk the DNS tree.
- Recursive resolver (PowerDNS Recursor, 8.8.8.8): walks root -> TLD -> authoritative and caches results for any name.
- You can point clients at an authoritative server, but it won't resolve arbitrary internet names.
- Some software can run both roles on one instance, but mixing is discouraged (security/performance). Best practice: keep them separate; optionally put dnsdist in front.

## GSLB (Global Server Load Balancing)

GSLB is DNS-based load balancing across multiple geographically distributed sites/datacenters.

How it works:
- A GSLB-enabled DNS server (or dedicated GSLB device) answers DNS queries for a hostname with different IPs based on policy: client geo-location, backend health, latency, load, etc.
- Clients get the "best" endpoint's IP and connect directly (unlike an L7 load balancer which proxies traffic).
- Health checks monitor backends; unhealthy sites are removed from DNS responses (failover).

Common uses:
- Multi-region / multi-datacenter HA and DR.
- Geo-routing (send EU users to EU DC).
- Disaster recovery failover.

Relation to PowerDNS/dnsdist:
- PowerDNS Authoritative can do basic DNS-based tricks, but full GSLB (health checks, geo, dynamic weighting) usually needs a dedicated GSLB product (F5 GTM/BIG-IP DNS, Citrix ADC) or managed DNS with GSLB (F5 Distributed Cloud DNS load balancing, NS1, Route53 latency routing).
- dnsdist can do routing/health-aware backend selection but is more a DNS proxy/LB than a full GSLB.

Caveats:
- DNS caching/TTL means failover isn't instant (clients may hold stale IPs until TTL expires).
- Works at DNS layer, so no per-connection/session stickiness like an L7 LB.

"Best endpoint" clarification:
- "Best" is policy-driven: geo-proximity (closest), lowest latency, least load, health, or weighted split (e.g., 70/30 for canary).
- Multi-DC global example: user in India -> Mumbai DC IP; user in US -> Virginia DC IP (geo-based GSLB, like a CDN but for your app origin).
- Single-site app: GSLB mostly doesn't matter for routing (only one answer). Still useful for failover/DR if a standby site is added, or health-based removal. For global edge caching of a single origin, a CDN is the better tool.

## GSLB vs dnsdist

- GSLB = DNS answer intelligence (which IP to return for a name based on geo/health/latency/load). Lives at the authoritative DNS layer.
- dnsdist = DNS traffic proxy/load balancer in front of backends (authoritative servers and/or recursors): routing, filtering, rate-limiting, caching.
- They can stack: Clients -> dnsdist (proxy/filter/DDoS protection) -> GSLB-enabled authoritative backend (returns best endpoint IP).
- Some GSLB products embed proxy/LB internally; dnsdist can mimic light GSLB-like behavior (backend health checks, weighted routing among local backends) but true geo/global GSLB needs a GSLB-capable DNS product.
- Neither is the other: GSLB decides the answer; dnsdist moves/protects the query.

## Scenario: Route53 (primary) + F5 (secondary)

- Route53 = primary (writable source of truth); F5 Distributed Cloud = secondary (read-only replica).
- F5 pulls the zone from Route53 via zone transfer (AXFR/IXFR) and serves it from its anycast network.
- Parent zone NS records list BOTH Route53 and F5 nameservers (add F5 NS for secondary DNS).
- Both are authoritative for the zone: either answers queries with the aa flag; F5 is authoritative for reads even as a secondary.
- SOA MNAME stays Route53; serial bumps on Route53 trigger F5 to re-transfer.
- Writes only on Route53; F5 is a read-only replica.
- If Route53 is down, F5 keeps serving the last transferred zone (until TTL/expire), but no updates propagate until primary returns (promoting F5 to primary is a manual/DR step).
- Requires Route53 to allow zone transfers to F5 IPs (allow-list). (Route53 outbound AXFR support is limited/conditional - verify with AWS; many use a self-hosted primary as the transfer source.)

## dnsdist forwarding flow (zones & backends)

Where dnsdist forwards:
- To explicitly configured backend pools (authoritative servers and/or recursors). You list backend IPs/ports, weights, and health checks in config. It does not auto-discover nameservers via NS records.

Flow with two zones (zone A, zone B):
1. Client query arrives at dnsdist (e.g., foo.zoneA.com).
2. dnsdist matches a rule (by qname/suffix) and selects the backend pool for that zone.
3. It forwards to one healthy backend in the pool (round-robin / least-outstanding / etc.).
4. That backend (authoritative for the zone) answers; dnsdist returns it to the client.

Primary vs secondary backends:
- All authoritative servers serving a zone (primary + secondaries) can answer queries; secondaries are read-only replicas but fully authoritative for reads.
- dnsdist can load-balance queries across primary AND secondaries of the same zone.
- Writes/edits happen only on the primary; dnsdist handles query traffic, not writes.
- Health checks remove failed backends, so traffic flows to remaining healthy ones (including secondaries).

Note: load balancing is based on configured backends + algorithm + health, not on NS records. Zone transfers (primary -> secondary) are a separate replication path.


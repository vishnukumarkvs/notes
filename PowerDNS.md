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

## SOA MNAME in secondary setups (F5 as secondary)

- If F5 is a secondary, the SOA is copied from the primary via zone transfer.
- The SOA MNAME (primary nameserver field) still points to your primary, not F5.
- F5's nameservers appear in the NS records and serve the zone authoritatively.
- So: SOA MNAME = source of truth (primary); NS set = who serves (includes F5 secondaries).
- Standard behavior keeps the primary's MNAME; some setups rewrite it, but serial comparison is what drives sync.
```


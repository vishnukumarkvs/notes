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

---

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

---

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

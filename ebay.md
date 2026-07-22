ebay
====

# Traffic

1. Edge & Global Traffic ManagementThese questions assess how you handle traffic before it even touches your application cluster.Global Load Balancing & Anycast:"If our primary US-East data center goes down, how do we shift 100k requests/sec to US-West instantly without overloading the target region or causing DNS propagation delays?""How would you compare BGP Anycast routing vs. Latency-based DNS routing for edge traffic ingress? What are the failover tradeoffs of each?"CDN & Edge Caching Strategy:"During a massive site-wide promotion, the cache hit ratio at the CDN drops from 95% to 60%. How do you diagnose whether this is a cache invalidation storm, a thundering herd problem, or bad cache headers?""How do you design an edge traffic architecture that protects backend services from cache stampedes on hot items (e.g., a high-demand item going live for auction)?"2. Ingress & Traffic ShapingThese questions test your understanding of Layer 4 vs. Layer 7 load balancing, API gateways, and rate limiting.  Rate Limiting & Abuse Prevention:"How would you implement a distributed rate limiter across multiple regions to prevent scraper bots from scraping listing prices without introducing cross-region latency on every request?""What is the difference between a Token Bucket and a Leaky Bucket algorithm? In what traffic scenario would you choose one over the other at the API gateway level?"Protocol & Network Level Traffic:"Walk me through what happens at the network layer when a client upgrades an HTTP/1.1 connection to HTTP/2 or HTTP/3 (QUIC) at our ingress gateway. How does multiplexing impact downstream connection pools?""You notice an ingress proxy dropping connections with Connection reset by peer under high load, but CPU and Memory are at 40%. What low-level kernel or network parameter bottlenecks do you suspect?"3. Microservice Routing & Load Balancing AlgorithmsThese questions evaluate your knowledge of internal service mesh routing, load shedding, and connection management.Load Balancing Algorithms:"When would Round-Robin or Least Connections fail in a microservice architecture with heterogeneous hardware or variable request latencies? How does Peak EWMA (Exponentially Weighted Moving Average) or Power of Two Random Choices solve this?"Load Shedding & Graceful Degradation:"A core backend search service is hit with a traffic spike that exceeds capacity by 200%. Auto-scaling cannot spin up pods fast enough. How do you implement load shedding so the service doesn't enter a crash-loop?""How do you differentiate critical traffic (e.g., 'Checkout/Place Order') from non-critical traffic (e.g., 'Recommended Items') during traffic mitigation at the ingress layer?"4. Scenario-Based Deep Dives (Architectural & Operational)Senior SRE interviews almost always include scenario-based questions where the interviewer pushes for technical depth.                  +-------------------+
                  |   Global Edge     |
                  |  (CDN / Anycast)  |
                  +---------+---------+
                            |
                            v
                  +-------------------+
                  |  Ingress Gateway  |
                  |   (Envoy / NGINX) |
                  +---------+---------+
                            |
              +-------------+-------------+
              |                           |
              v                           v
      +---------------+           +---------------+
      |  Order Service|           | Search Service|
      |   (Cluster A) |           |   (Cluster B) |
      +---------------+           +---------------+
Scenario 1: The Cascading Connection Timeout"You deploy a new API routing rule. Minutes later, tail latency ($p99$) on the Checkout service spikes from 50ms to 8s, causing cascading HTTP 504 Gateway Timeouts upstream. CPU on the Checkout pods is low. How do you approach isolating whether this is connection pool exhaustion, DNS resolution blockages, or slow downstream dependencies?"Scenario 2: The "Thundering Herd" Retry Storm"A temporary network partition causes 5% of database calls to fail. Clients automatically retry requests, which doubles incoming traffic to the API gateway and completely crashes the backend. How do you redesign client-side and proxy-level retry behaviors (e.g., exponential backoff, jitter, circuit breakers) to stop retry storms?"









What if pop goaes down
- Anycast has a single IP called VIP. This is glovbal. Clients connect to this VIP.  From there, the traffic is distributed to multiple regions (edge pops)
- If a pop goes down,  anycast using healthchecks will disrtribute traffic to other datacenters
- Anycast acts as CDN as well, it always forwards to nearest data centers


- Anycast - Using BGP, anounces multiple edge pops with same ip
- from anycast, it chooses shortest ASN path
- once it reaches pop, it uses l4 load balancer (argo etc) and forwards it to envoy
- Envoy processes the http packet and it resolves
- Envoy has listenersm, routes and clusters


Rate limit

- How many requests client can make on a timewindow
- Ratelimit hit: 429 too many requests
- 100 requests per min, counter reset at interval.
- Use in-memory datastore for this. app in memory wont work if we scale to 2 instances, he m ight get 200 req per min
- 

Token bucket vs leaky bucket
- Fized tokens. Every request consumes a token, 100 tokens added per interval
- token bucket for bursts
- leaky bucket - fixed output, constant output

HTTPS connection

TCP handshake - 3 way - syn -> syn-ack -> ack
Certificate exchager - server sends it certificatewith public key, Laos, cipher suite TLS1.3 etc selection
Key exchange - Client creates symmetric key, encrypts with pub;lic key of server and sends to server

Then the session uses this session symmetric key


Syn flood
- When malicios actor sends flood of syn packets, server sends with syn-ack but then the actor never sends ack packet. Server waits and causes real connections to drop of
- To mitigate, bloc the malicious actor ip
- Or enable syncookies in kernel. It helps with memory resource usage. A
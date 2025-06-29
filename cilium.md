cilium
======

Its a Serrrvice Mesh, Security, Observability


https://www.youtube.com/watch?v=e10kDBEsZw4

- Containers in pod share same network namespace
- Sidecar proxy can handle tls, load balancing etc
- Cilium moves service mesh to kernel
- L2/L3/L4 is moved to kernel except L7
- L7 = Http, GRPC etc
- cilium has observability to l7 requests
- cilium hubble for observabality
- CliumNetworkPolicy - can act on L7 rules
- Cilium uses envoy proxy for L7
- Cilium replaces kube-proxy

Service Mesh
- Observability
- Ingress
- - Load balacing
- - Protocol Parsing
- - Path based routing
- L7 Traffic management
- - Service Load balancing
- - Rules (canary rollouts, retries)
- Identity Based security

Ingess
- CiliumEnvoyConfig

Reduce Resource Usage
- By going sidecarless approach
- Network path is reduced. L7 uses envoy proxy, L3/L4 uses ebpf

Performace
- Cilium thoughtput is slightly better than istio
- But pod startuptime latency is significantly better tan istio


mutual authentication and encryption
- Certificate manager like cert-manager, SPIFEE etc


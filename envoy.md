envoy
=====

# Intro

- created by lyft
- graduated by cncf
- implemented in c++
- envoy can be dynamically configurable, unlike traditional api gateways with a static route file
- envoy is foundation of many cloud native networking products including mesh platforms like istio

# Envoy admin interface

- envoy has an admin interface
- you can view listeners, clusters, tweak settings etc

```
admin:
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 9901
```

# Listeners

- envoy is a proxy
- envoy can act as L4 proxy (TCP) as well as L7 proxy (Application/HTTP)
- every listener is associated with a filter

```
static_resources:

  listeners:
  - name: listener_0
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 8000
    filter_chains:
    - filters:
      - name: envoy.filters.network.direct_response
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.direct_response.v3.Config
          response:
            inline_string: "hello world"
```

# Filters
- Every listener is associated with a filter
- Filters are one of the most powerful features of Envoy
- They form a pipeline to process traffic and flows through proxy
- We have listener filters, network filters, http filters etc
- Full documentation here: https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/filter/filter
- With help of filters, envoy can process any request, might be tcp, http, tls grpc, jwt, and so on
- It also supports many proxies like redis proxy, wasm proxy, zookeper proxy etc
- It can handle various scenarions like auth, rbac, rate limit etc

# Clusters

- a cluster is a group of logically similar upstream hosts to which envoy connects
- envoy proxies to cluste which intern proxies to backend
- cluster does service discovery on backend

```
static_resources:
  listeners:
  - name: tcp_listener
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 9000
    filter_chains:
    - filters:
      - name: envoy.filters.network.tcp_proxy
        typed_config:
          "@type": "type.googleapis.com/envoy.extensions.filters.network.tcp_proxy.v3.TcpProxy"
          stat_prefix: tcp_stats
          cluster: tcp_backend

  clusters:
  - name: tcp_backend
    connect_timeout: 5s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: tcp_backend
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: backend-service
                port_value: 8080
```

# Http Connection Manager

- used for L7 proxies. Is core of envoy
- used to handle http connections

```
static_resources:
  listeners:
  - name: http_listener
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 10000
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": "type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager"
          stat_prefix: http_stats
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains:
              - "candletower.com"
              routes:
              - match:
                  prefix: "/hello"
                route:
                  cluster: hello_service
          http_filters:
          - name: envoy.filters.http.router
            typed_config:
              "@type": "type.googleapis.com/envoy.extensions.filters.http.router.v3.Router"

  clusters:
  - name: hello_service
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: hello_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: hello-backend
                port_value: 8080
```

- a virtual host or a "vhost" is an envoy concept that allows single proxy instance to route traffic to different backend services

# Downstream vs Upstream

Downstream
- Traffic flowing from client to envoy
- incoming direction

Upstream
- Traffic flowing from envoy to backend
- envoy -> services
- outgoing direction

# Stats

- envoy has a ton of metrics which are very useful

# Retries,  enhancements

- if you have a backend server which is unreliable, then we can use reties

```
retry_policy:
  retry_on: "5xx"
  num_retries: 10
```

- metrics are also present
```
upstream_rq_total - Total requests
upstream_rq_active - Active requests
upstream_rq_time - Request timing
upstream_rq_retry - Request retries
upstream_rq_retry_overflow - Retry overflow
upstream_rq_pending_overflow - Pending overflow
```

# Security

- envoy supports TLS , both on downstream and upstream

This gives you several deployment options:

- TLS termination: Decrypt at Envoy, send unencrypted to backends
- TLS passthrough: Pass encrypted traffic directly to backends
- TLS re-encryption: Decrypt at Envoy, then re-encrypt to backends
- mTLS: Mutual TLS authentication in either direction

Example of Downstream TLS
- happens at listener level

```
  listeners:
  - name: downstream_tls_listener
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 443
    filter_chains:
    - transport_socket:
        name: envoy.transport_sockets.tls
        typed_config:
          "@type": "type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.DownstreamTlsContext"
          common_tls_context:
            tls_certificates:
            - certificate_chain:
                filename: "/etc/envoy/certs/cert.pem"
              private_key:
                filename: "/etc/envoy/certs/key.pem"
```


# Observability

- We can use prometheus and grafana
- The strong point of envoy is its observability and ton of metrics support
- We have metrics for everything, like downstream, upstream, cluster health, load balancing, live connectons , resource usage etc

# Access Logs

- Envoy access logs are key component in monitoring envoy traffic
- Envoy has logs for whole timeline of a request
- Some of the fields in a log include
  - the source of request
  - time of request
  - duration
  - headers
  - failure reason

- It can be added in http_connection_manager

```
  # Access log configuration
          access_log:
          - name: envoy.access_loggers.file
            typed_config:
              "@type": "type.googleapis.com/envoy.extensions.access_loggers.file.v3.FileAccessLog"
              path: "/var/log/envoy/access.log"
              format: "[%START_TIME%] \"%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%\" %RESPONSE_CODE% %RESPONSE_FLAGS% %BYTES_RECEIVED% %BYTES_SENT% %DURATION% %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% \"%REQ(X-FORWARDED-FOR)%\" \"%REQ(USER-AGENT)%\" \"%REQ(X-REQUEST-ID)%\" \"%REQ(:AUTHORITY)%\" \"%UPSTREAM_HOST%\"\n"
```

- It also supports mny formating types including json

# Control Plane

- until now, we used static configuration for envoy which is a file
- envoy can  get its confiuration on the fly from connectiong to an XDS Server
- This is used for dynamic endpoints, tls cert rotation etc
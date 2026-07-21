otel-demo
=========

 helm upgrade --install my-otel-demo opentelemetry-demo --repo=https://open-telemetry.github.io/opentelemetry-helm-charts --namespace otel-demo


kubectl --namespace otel-demo port-forward svc/frontend-proxy 8080:8080


# Values

~~~
Here's the complete values.yaml for a lean OTel demo (no Jaeger, no OpenSearch, no Grafana):
# Lean OTel Demo - no Jaeger, OpenSearch, or Grafana
# Deploy with:
#   helm install my-otel-demo open-telemetry/opentelemetry-demo \
#     --namespace otel-demo --create-namespace \
#     --values values-otel-lean.yaml

# --- Disable heavy infrastructure ---
jaeger:
  enabled: false

opensearch:
  enabled: false

grafana:
  enabled: false

# --- OTel Collector: remove disabled backend exporters ---
opentelemetry-collector:
  enabled: true
  image:
    repository: "otel/opentelemetry-collector-contrib"
  fullnameOverride: otel-collector
  mode: daemonset
  resources:
    limits:
      memory: 200Mi
  config:
    receivers:
      otlp:
        protocols:
          http:
            cors:
              allowed_origins:
                - "http://*"
                - "https://*"
      kafkametrics:
        scrapers:
          - brokers
          - topics
          - consumers
        brokers:
          - kafka:9092
        collection_interval: 10s
    exporters:
      otlphttp/prometheus:
        endpoint: http://prometheus:9090/api/v1/otlp
        tls:
          insecure: true
        sending_queue:
          batch: {}
    processors:
      memory_limiter:
        check_interval: 5s
        limit_percentage: 80
        spike_limit_percentage: 25
      resourcedetection:
        detectors: [env, system]
      transform:
        error_mode: ignore
        trace_statements:
          - context: span
            conditions:
              - span.kind == SPAN_KIND_SERVER and resource.attributes["service.name"] == "frontend" and span.attributes["http.route"] == nil
            statements:
              - set(span.attributes["http.route"], "/api/cart") where IsMatch(span.attributes["http.target"], "\\/api\\/cart")
              - set(span.attributes["http.route"], "/api/checkout") where IsMatch(span.attributes["http.target"], "\\/api\\/checkout")
              - set(span.attributes["http.route"], "/api/products/{productId}") where IsMatch(span.attributes["http.target"], "\\/api\\/products\\/.*")
              - set(span.attributes["http.route"], "/api/recommendations") where IsMatch(span.attributes["http.target"], "\\/api\\/recommendations")
              - set(span.attributes["http.route"], "/api/data") where IsMatch(span.attributes["http.target"], "\\/api\\/data.*")
          - context: span
            statements:
              - set_semconv_span_name("1.37.0", "unsanitized_span_name")
      resource:
        attributes:
          - key: service.instance.id
            from_attribute: k8s.pod.uid
            action: insert
    connectors:
      spanmetrics: {}
    service:
      pipelines:
        traces:
          processors: [memory_limiter, resourcedetection, resource, transform, batch]
          exporters: [debug, spanmetrics]
        metrics:
          receivers: [otlp, kafkametrics, spanmetrics]
          processors: [memory_limiter, resourcedetection, resource, batch]
          exporters: [otlphttp/prometheus, debug]
        logs:
          processors: [memory_limiter, resourcedetection, resource, batch]
          exporters: [debug]
      telemetry:
        metrics:
          level: detailed
          readers:
            - periodic:
                interval: 10000
                timeout: 5000
                exporter:
                  otlp:
                    protocol: http/protobuf
                    endpoint: http://otel-collector:4318
                    insecure: true

# --- Prom kept for metrics ---
prometheus:
  enabled: true
  alertmanager:
    enabled: false
  configmapReload:
    prometheus:
      enabled: false
  kube-state-metrics:
    enabled: false
  prometheus-node-exporter:
    enabled: false
  prometheus-pushgateway:
    enabled: false
  server:
    fullnameOverride: prometheus
    extraFlags:
      - "enable-feature=exemplar-storage"
      - "web.enable-otlp-receiver"
    retention: 7d
    tsdb:
      out_of_order_time_window: 30m
    persistentVolume:
      enabled: false
    service:
      servicePort: 9090
    resources:
      limits:
        memory: 400Mi

# --- Frontend proxy: remove Jaeger/Grafana routes ---
frontend-proxy:
  enabled: true
  env:
    - name: ENVOY_PORT
      value: "8080"
    - name: ENVOY_ADDR
      value: "0.0.0.0"
    - name: ENVOY_ADMIN_PORT
      value: "10000"
    - name: FRONTEND_HOST
      value: frontend
    - name: FRONTEND_PORT
      value: "8080"
    - name: LOCUST_WEB_HOST
      value: load-generator
    - name: LOCUST_WEB_PORT
      value: "8089"
    - name: OTEL_COLLECTOR_HOST
      value: otel-collector
    - name: OTEL_COLLECTOR_PORT_GRPC
      value: "4317"
    - name: OTEL_COLLECTOR_PORT_HTTP
      value: "4318"
  resources:
    limits:
      memory: 65Mi
~~~
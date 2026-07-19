otel-demo
=========

 helm upgrade --install my-otel-demo opentelemetry-demo --repo=https://open-telemetry.github.io/opentelemetry-helm-charts --namespace otel-demo


kubectl --namespace otel-demo port-forward svc/frontend-proxy 8080:8080



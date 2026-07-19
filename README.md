# Notes


## Open Source Apps interested
- coroot
- greptimedb

## Demo apps
- https://github.com/googlecloudplatform/microservices-demo
  - helm upgrade --install onlineboutique oci://us-docker.pkg.dev/online-boutique-ci/charts/onlineboutique --namespace=default
- https://github.com/open-telemetry/opentelemetry-demo
  - helm upgrade --install my-otel-demo opentelemetry-demo --repo=https://open-telemetry.github.io/opentelemetry-helm-charts


## Mac setup
- kind can be ok for small k8s, its not fullsca;e
- multipass ubuntu vm + k3s is ideal

## Multipass

Installation
brew install --cask multipass
multipass launch --name k3s-vm --cpus 2 --memory 4G --disk 20G

multipass list

multipass shell k3s-vm

coroot prometheus
=================

KUBECONFIG=~/.kube/config-k3s kubectl port-forward -n coroot svc/coroot-prometheus 9099:9099



kubectl port-forward -n coroot svc/coroot-prometheus 9099:9099


~ $ /promviewerctl cardinality --metric-regex 'container.*'


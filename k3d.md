k3d
===

wrapper on k3s
used by rancher desktop


k3d cluster create two-node-cluster --agents 2


kubectl -n kube-system patch daemonset myDaemonset -p '{"spec": {"template": {"spec": {"nodeSelector": {"non-existing": "true"}}}}}'

kubectl -n kube-system patch daemonset myDaemonset --type json -p='[{"op": "remove", "path": "/spec/template/spec/nodeSelector/non-existing"}]'

k3s coroot
==========

KUBECONFIG=/Users/kvsvishnukumar/.kube/config-k3s kubectl label ns coroot pod-security.kubernetes.io/enforce=privileged --overwrite && KUBECONFIG=/Users/kvsvishnukumar/.kube/config-k3s kubectl get pods -n coroot


KUBECONFIG=/Users/kvsvishnukumar/.kube/config-k3s kubectl get pods -n coroot -w --request-timeout=30s 2>&1 & sleep 5; KUBECONFIG=/Users/kvsvishnukumar/.kube/config-k3s kubectl get pods -n coroot; echo "---"; KUBECONFIG=/Users/kvsvishnukumar/.kube/config-k3s kubectl get coroot -n coroot -o yaml 2>&1 | head -80



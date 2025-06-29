kubernetes
==========

Exec into k8s node
- k debug node/colima -it --image=ubuntu
- chroot /host

CNI specific file in node is: ls /etc/cni/net.d/

-  k debug node/rough-worker -it --image=ubuntu
-  k debug node/rough-control-plane -it --image=ubuntu

No—kube-proxy is not a CNI plugin.
- CNI (Container Network Interface) plugins (like kindnet’s PTP, Calico, Flannel, etc.) are responsible for wiring up pod-to-pod L2/L3 networking and IPAM.
- kube-proxy sits on each node to implement Kubernetes Services at L4 by programming iptables or IPVS rules so that ClusterIP, NodePort and LoadBalancer traffic gets routed to the right pod endpoints.



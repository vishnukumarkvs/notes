multipass
=========

k3s with multipass

brew install --cask multipass
multipass launch --name k3s-vm --cpus 2 --memory 4G --disk 20G

multipass shell k3s-vm
curl -sfL https://get.k3s.io | sh -

 ❯ multipass list
Name                    State             IPv4             Image
k3s-vm                  Running           192.168.2.2      Ubuntu 26.04 LTS

$ multipass info k3s-vm | grep -E "Memory|Disk|CPU"
CPU(s):         2
Disk usage:     15.0GiB out of 19.3GiB
Memory usage:   3.3GiB out of 3.8GiB



multipass mount "$(pwd)" k3s-vm:/home/ubuntu/coroot-operator

$ sudo kubectl get po -A
NAMESPACE     NAME                                      READY   STATUS              RESTARTS   AGE
kube-system   coredns-76c974cb66-lpdr7                  1/1     Running             0          28s
kube-system   helm-install-traefik-6j7nf                0/1     Completed           1          24s
kube-system   helm-install-traefik-crd-rfgz4            0/1     Completed           0          24s
kube-system   local-path-provisioner-8686667995-x7ppt   1/1     Running             0          28s
kube-system   metrics-server-c8774f4f4-7hvp4            0/1     Running             0          28s
kube-system   svclb-traefik-fbcee8c7-t294x              0/2     ContainerCreating   0          6s
kube-system   traefik-c5c8bf4ff-9zz6b                   0/1     ContainerCreating   0          6s
ubuntu@k3s-vm:~$


~/rough ❯ tree
.
└── tests
    └── tls
        ├── ca.crt
        ├── ca.key
        ├── ca.srl
        ├── cert.conf
        ├── coroot-test-tls.yaml
        ├── server.crt
        ├── server.csr
        └── server.key

3 directories, 8 files

~/rough ❯ pwd
/Users/kvsvishnukumar/rough

~/rough ❯
\

holding statements, transaction sttemen


multipass exec k3s-vm -- sudo cat /etc/rancher/k3s/k3s.yaml | sed 's/127.0.0.1/192.168.252.2/g' > /Users/kvsvishnukumar/.kube/config-k3s

kubectl --kubeconfig /Users/kvsvishnukumar/.kube/config-k3s get nodes

export KUBECONFIG=~/.kube/config-k3s


sleep 15 && multipass exec k3s-vm -- free -h && multipass exec k3s-vm -- nproc
total        used        free      shared  buff/cache   available
Mem:           7.7Gi       2.8Gi       1.7Gi        22Mi       3.4Gi       4.9Gi
Swap:             0B          0B          0B
4




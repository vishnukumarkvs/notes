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
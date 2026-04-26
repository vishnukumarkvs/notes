coroot opertor tls
==================

The core components (coroot, node-agent, cluster-agent) now support TLS and CA verification.
*   coroot: Supports HTTP_DISABLED (bool) and HTTPS_LISTEN (string).
*   node-agent: Supports CA_FILE (string).
*   cluster-agent: Supports CA_FILE (string).
I'll now summarize the proposed updates for the operator.
2. Update Plan for Coroot Operator
A. API Changes (api/v1/coroot_types.go)
*   Coroot Configuration:
    *   Add HTTPSListen (string) to CorootSpec.
    *   Add CASecret (*corev1.SecretKeySelector) to TLSSpec (optional, for internal CA if needed).
*   Agent Configuration:
    *   Create a new AgentTLSSpec struct:
                type AgentTLSSpec struct {
            CASecret *corev1.SecretKeySelector `json:"caSecret,omitempty"`
            SkipVerify bool `json:"skipVerify,omitempty"`
        }
            *   Add TLS *AgentTLSSpec to NodeAgentSpec and ClusterAgentSpec.
    *   Add CASecret *corev1.SecretKeySelector to AgentsOnlySpec.
B. Controller Changes (controller/coroot.go)
*   Service & Container Configuration:
    *   Update corootStatefulSet to handle HTTPS_LISTEN and HTTP_DISABLED environment variables.
    *   Ensure the HTTPSPort in ServiceSpec (if set) maps to the HTTPSListenAddress.
    *   Update the ConfigMap generation to include https_listen_address and http_disabled in config.yaml.
    *   Update validateCoroot to handle the new fields.
C. Agent Controller Changes (controller/node_agent.go & controller/cluster_agent.go)
*   Environment & Volumes:
    *   If TLS.CASecret is provided:
        *   Add a projected volume or individual secret volume to the agent pods.
        *   Mount the secret (e.g., to /etc/coroot-ca/ca.crt).
        *   Set the CA_FILE environment variable to the mount path.
    *   Ensure INSECURE_SKIP_VERIFY is still handled correctly.
    *   Update nodeAgentDaemonSet and clusterAgentDeployment to incorporate these changes.
Analysis Conclusion
The operator currently handles basic TLS (secret mounts for cert/key) but needs to be extended to:
1.  Expose HTTPSListen as a configurable field instead of hardcoding :8443.
2.  Enable HTTPDisabled to strictly enforce TLS.
3.  Provide a way to pass CA certificates to agents so they can verify the Coroot server when using self-signed or private CAs.
I have already drafted some of these changes in the previous (aborted) attempt, but I will wait for your confirmation before proceeding with any edits


Operators use CustomResource, not helm values or helm charts

# Open Tasks

In controller/coroot.go, shoud we specify default 8443 ? HTTPSPort in line 45. Shouldnt it get it from HTTPSListenAddress

If the user provides 0.0.0.0:8443, strings.Cut will still work correctly because it splits the string at the first occurrence of the colon. The "after" value will be 8443, which is exactly what we want.
However, strings.Cut would fail if the user provides an IPv6 address (e.g., [::1]:8443), as it would split at the first of many colons. 
To be more robust and handle all formats (IPv4, IPv6, or just :port), you can use strings.LastIndex. Here is the refined version:
	if cr.Spec.TLS != nil && cr.Spec.Service.HTTPSPort == 0 {
		httpsListen := cmp.Or(cr.Spec.HTTPSListen, ":8443")
		// Find the last colon to safely handle IPv6 [::1]:8443
		if i := strings.LastIndex(httpsListen, ":"); i >= 0 {
			portStr := httpsListen[i+1:]
			if p, err := resource.ParseQuantity(portStr); err == nil {
				cr.Spec.Service.HTTPSPort = int32(p.Value())
			}
		}
		// Final fallback if parsing failed
		if cr.Spec.Service.HTTPSPort == 0 {
			cr.Spec.Service.HTTPSPort = 8443
		}
	}
This logic is safer because it always looks for the port at the very end of the string. Also, if you apply this, you might want to update the similar logic at line 568 in corootStatefulSet to use strings.LastIndex as well, so the behavior is identical throughout the file.


Is it good to make 8443 as default port? Should we take it from HttpsListen param?



# Test

### Instructions to Test Locally

I have placed all necessary files in the `tests/tls/` directory. You can follow these steps to verify the TLS support in your local `minikube` or `kind` cluster:

**1. Set up the Environment & Secrets:**
Run these commands to prepare the `coroot` namespace and the required secrets.
```bash
# Create namespace
kubectl create namespace coroot

# Create Server TLS Secret (Cert and Key)
kubectl create secret generic coroot-server-tls \
  --from-file=tls.crt=tests/tls/server.crt \
  --from-file=tls.key=tests/tls/server.key \
  -n coroot

# Create CA Secret (used by Agents to trust the server)
kubectl create secret generic coroot-ca-cert \
  --from-file=ca.crt=tests/tls/ca.crt \
  -n coroot

# Create a placeholder API Key Secret
kubectl create secret generic coroot-api-key \
  --from-literal=key=test-api-key-123 \
  -n coroot
```

with sudo
sudo kubectl create secret generic coroot-server-tls   --from-file=tls.crt=tests/tls/server.crt   --from-file=tls.key=tests/tls/server.key   -n coroot
sudo kubectl create secret generic coroot-ca-cert   --from-file=ca.crt=tests/tls/ca.crt   -n coroot
sudo kubectl create secret generic coroot-api-key --from-literal=key=test-api-key-123  -n coroot

**2. Deploy the Operator & Custom Resource:**
Ensure your operator is running, then apply the test CR: make run in coroot-operator folder
```bash
# Apply the CRD first (if not already applied)
kubectl apply --server-side -f config/crd/coroot.com_coroots.yaml

# Apply the TLS test configuration
kubectl apply -f tests/tls/coroot-test-tls.yaml
```

**3. Verification Commands:**
Use these commands to verify that the components are communicating securely over TLS:

*   **Check Service Ports**: Verify that port `8443` is active.
    ```bash
    kubectl get svc -n coroot coroot-coroot -o jsonpath='{.spec.ports}'
    ```

*   **Check Agent Environment & Mounts**: Verify the agents are using `https://` and have the `CA_FILE` environment variable.
    ```bash
    # Verify Node Agent (DaemonSet)
    kubectl get ds -n coroot coroot-node-agent -o yaml | grep -A 5 "CA_FILE"
    
    # Verify Cluster Agent (Deployment)
    kubectl get deploy -n coroot coroot-cluster-agent -o yaml | grep -A 5 "CA_FILE"
    ```

*   **Check Agent Logs**: Ensure there are no TLS handshake errors.
    ```bash
    # Node Agent logs
    kubectl logs -n coroot -l app.kubernetes.io/component=coroot-node-agent -c node-agent
    
    # Cluster Agent logs
    kubectl logs -n coroot -l app.kubernetes.io/component=coroot-cluster-agent -c cluster-agent
    ```

If the setup is correct, you should see the agents successfully sending telemetry to the Coroot server via `https://coroot-coroot.coroot:8443`.

---

 kubectl port-forward svc/coroot-coroot 8443:8443 -n coroot
 
 https://localhost:8443 - open insecure , will see UI
 
 # 21
 
 apiVersion: coroot.com/v1
kind: Coroot
metadata:
  name: coroot
  namespace: coroot
spec:
  tls:
    certSecret:
      name: coroot-server-tls
      key: tls.crt
    keySecret:
      name: coroot-server-tls
      key: tls.key
  
  httpDisabled: true
  
  apiKeySecret:
    name: coroot-api-key
    key: key

  projects:
    - name: "My Cluster"
      apiKeys:
        - keySecret:
            name: coroot-api-key
            key: key

  clusterAgent:
    tls:
      caSecret:
        name: coroot-ca-cert
        key: ca.crt
      tlsSkipVerify: false

  nodeAgent:
    tls:
      caSecret:
        name: coroot-ca-cert
        key: ca.crt
      tlsSkipVerify: false


container_resources_memory_rss_bytes


StoreMetricsInClickhouse: true



Summary of findings:
1. Node-agent IS collecting metrics - it sends data to Prometheus successfully
2. Node-agent sees ~17 containers in Prometheus (kube-system, local-path-storage, coroot-node-agent itself)
3. Node-agent does NOT see coroot pods (coroot-coroot-0, coroot-prometheus, coroot-clickhouse-*, coroot-cluster-agent)
4. The cgroup paths exist and have processes, but node-agent isn't collecting their metrics
Root cause: This appears to be a cgroup namespace isolation issue specific to kind clusters. The coroot pods are in besteffort QoS class while node-agent is in burstable, and cgroup2 namespace delegation (nsdelegate) may be preventing node-agent from accessing the besteffort cgroup subtree properly.
Why you see CPU/memory graphs: You're likely seeing metrics from other namespaces (kube-system pods like coredns, kube-proxy, etcd) which work fine.
Solutions to get coroot namespace metrics:
1. Deploy node-agent as a global DaemonSet (not in coroot namespace) 
2. Or use Prometheus scrape configs with kubernetes_sd_configs to discover pods directly
3. Or enable storeMetricsInClickhouse: true in Coroot CR to bypass this issue


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


I don't see the tests folder in the repo now. It may have been in your local copy or generated elsewhere. For TLS certs, here's how to create them:
# Create test TLS certificates
cd tests/tls
# 1. Create CA
openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 365 -key ca.key -out ca.crt -subj "/CN=Coroot-Test-CA"
# 2. Create server key and CSR
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr -subj "/CN=coroot.coroot.svc"
# 3. Create server cert (using CA)
echo "subjectAltName = DNS:coroot.coroot.svc,DNS:coroot.coroot,DNS:coroot-coroot.coroot" > cert.conf
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -extfile cert.conf -days 365

#Run this on your Mac
multipass exec <vm-name> -- sudo cat /etc/rancher/k3s/k3s.yaml > ~/.kube/config
#Edit the config file to point to the VM's IP
#Replace '127.0.0.1' with the VM's actual IP (find it with 'multipass info <vm-name>')

#Run this on your Mac
kubectl port-forward svc/<service-name> 8080:80


multipass exec k3s-vm -- sudo cat /etc/rancher/k3s/k3s.yaml > ~/.kube/config
#Edit the 'server' line in the file
sed -i 's/127.0.0.1/192.168.2.2/g' ~/.kube/config


 ❯ multipass info k3s-vm
Name:           k3s-vm
State:          Running
Snapshots:      0
IPv4:           192.168.2.2
                10.42.0.0
                10.42.0.1
Release:        Ubuntu 26.04 LTS
Image hash:     9daa955c3d4c (Ubuntu 26.04 LTS)
CPU(s):         2
Load:           0.12 0.19 0.18
Disk usage:     4.5GiB out of 19.3GiB
Memory usage:   1.1GiB out of 3.8GiB
Mounts:         /Users/kvsvishnukumar/VishnuKvs/Workspace/myrepos/coroot-operator => /home/ubuntu/coroot-operator
                    UID map: 501:default
                    GID map: 20:default

coroot-operator/tests/tls on  add-tls-support ❯ multipass umount k3s-vm:/home/ubuntu/coroot-operator

coroot-operator/tests/tls on  add-tls-support ❯ multipass info k3s-vm
Name:           k3s-vm
State:          Running
Snapshots:      0
IPv4:           192.168.2.2
                10.42.0.0
                10.42.0.1
Release:        Ubuntu 26.04 LTS
Image hash:     9daa955c3d4c (Ubuntu 26.04 LTS)
CPU(s):         2
Load:           0.11 0.19 0.18
Disk usage:     4.5GiB out of 19.3GiB
Memory usage:   1.1GiB out of 3.8GiB
Mounts:         --

coroot-operator/tests/tls on  add-tls-support ❯


Below worked
multipass exec k3s-vm -- sudo cat /etc/rancher/k3s/k3s.yaml > ~/.kube/k3s-fixed.yaml
server: https://192.168.2.2:6443
export KUBECONFIG=~/.kube/k3s-fixed.yaml


To specify a custom port in config/samples/coroot_tls.yaml, you need to set httpsListen (for the container) and service.httpsPort (for the Kubernetes Service) in the spec section:
spec:
  httpsListen: ":9443" # Port the container listens on
  service:
    httpsPort: 9443    # Port the Service exposes


Coroot frontend
docker build -f dev.Dockerfile -t vishnukumarkvs/coroot-ui-select-all .
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

**2. Deploy the Operator & Custom Resource:**
Ensure your operator is running, then apply the test CR: make run in coroot-op
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
 
 

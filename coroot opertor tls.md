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


# Open Tasks

In coroot.go, shoudl we specify default 8443 ? HTTPSPort in line 45. Shouldnt it get it from HTTPSListenAddress


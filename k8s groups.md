k8s groups
==========

API Group,Purpose,Key Resources
""""" (Core/Legacy)",Foundational cluster resources.,"pods, services, configmaps, secrets, namespaces, nodes"
apps/v1,Workload and application management.,"deployments, daemonsets, statefulsets, replicasets"
batch/v1,Task and job scheduling.,"jobs, cronjobs"
networking.k8s.io/v1,Networking and traffic rules.,"ingresses, networkpolicies"
rbac.authorization.k8s.io/v1,User permissions and access control.,"roles, clusterroles, rolebindings"
storage.k8s.io/v1,Storage and volume management.,"storageclasses, volumeattachments"
autoscaling/v2,Scaling application resources.,horizontalpodautoscalers


One of the most powerful features of Kubernetes is that it is extensible. You can create your own API groups using Custom Resource Definitions (CRDs)


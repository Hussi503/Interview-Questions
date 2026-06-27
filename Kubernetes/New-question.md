# Kubernetes Interview Preparation Roadmap (5-6+ Years DevOps)

## Section 1: Kubernetes Architecture & Core Components
### 1. Explain the architecture of Kubernetes with master and worker node components.
### 2. What are the roles of kubelet, kube-apiserver, kube-proxy, etcd, scheduler, and controller manager?
kube-apiserver is the central control plane entry point. In real environments like AKS or EKS, every operation—whether it’s a deployment from Azure DevOps, kubectl command, 
or autoscaler action—first goes through the API server. It validates requests, enforces authentication and authorization, and updates the desired state in etcd. So practically, 
it acts like the “front door and brain coordinator” of the entire cluster.

etcd is the persistent key-value store that holds the entire cluster state. This includes deployments, pods, config maps, secrets, and node information. In production, etcd is 
highly critical because it is the single source of truth; if etcd is corrupted or lost, the cluster state is effectively lost, even if workloads are still running temporarily.

kube-scheduler is responsible for deciding where a pod should run. When a new pod is created, it doesn’t have a node assigned. The scheduler evaluates available worker nodes based on CPU,
memory, taints/tolerations, affinity rules, and assigns the best-fit node. In real production clusters, this is key for workload distribution and performance optimization.

kubelet runs on every worker node and is responsible for the actual execution of workloads. Once the scheduler assigns a pod to a node, the kubelet pulls the pod specification 
from the API server and ensures the container is running using the container runtime like containerd. It also continuously performs health checks and reports pod and node status back to the control plane.

kube-proxy handles the networking layer inside the cluster. It maintains network rules using iptables or IPVS so that Kubernetes Services can route traffic correctly to backend pods.
In real production scenarios, whenever a service IP is accessed, kube-proxy ensures the request is load balanced across healthy pods.

controller-manager runs all the control loops that continuously reconcile the desired state with the actual state. For example, if a deployment expects 3 replicas and one pod crashes, 
the controller manager detects the mismatch and recreates the pod automatically. It includes controllers like Deployment, ReplicaSet, Node, and Job controllers.

3. What happens internally when you run kubectl apply?
87. What is a static pod?
88. What are Kubernetes operators? Have you used them?
89. What are admission controllers? How have you used them?
102. How does Kubernetes help with reliability?
106. How does Kubernetes manage a large number of Docker containers?
115. What is the difference between a Pod and a container?

---

## Section 2: Workload Controllers
4. What is the difference between a Deployment and a StatefulSet?
5. What is a DaemonSet? Give examples.
6. What is the difference between a ReplicaSet and a Deployment?
7. What is the difference between Pod, Deployment, ReplicaSet, StatefulSet, and DaemonSet?
8. Can you attach a volume to a Deployment?
9. What could cause a StatefulSet pod to fail when rescheduled to a different AZ?
94. If a DaemonSet pod is pending, how would you troubleshoot?
95. Why would a DaemonSet create two pods per node?
116. What is the difference between a Job and a CronJob?

---

## Section 3: Storage
10. How do PV and PVC behave across zones?
117. What is the difference between a PV and PVC?
118. What is the difference between a StorageClass and a PV?
12. What is a Pod Disruption Budget (PDB)?
52. How do you handle database migrations safely in Kubernetes?
80. What happens if etcd is corrupted? How do you recover?
82. How do you backup and restore etcd?

---

## Section 4: Services, Ingress & Networking
11. What is a headless service?
22. What are the different Service types?
23. What is the difference between Ingress and LoadBalancer?
24. How does Ingress work?
25. How will you manage SSL certificates for ALB in EKS?
26. What type of load balancer is created when using Ingress in EKS?
27. Where will you mention the load balancer type in ingress YAML?
28. How does kube-proxy work?
29. How does service discovery work with CoreDNS?
30. Explain the Kubernetes networking model.
31. How do you restrict pod-to-pod communication using Network Policies?
72. Service reachable internally but not externally.
73. NodePort not accessible.
74. Ingress returns 502.
75. Debug CNI issues.
76. CoreDNS crashes.
77. NetworkPolicy blocking traffic.
78. Capture network traffic inside a pod.
108. Pod-to-pod ping not working.
114. How does DNS resolution work inside a pod?
124. How do you expose a pod using ALB?

---

## Section 5: Scheduling
16. How does the scheduler decide where to place pods?
17. What are taints and tolerations?
18. What are node affinity and anti-affinity?
19. How do you assign pods using taints/tolerations and affinity?
20. What happens when a node goes NotReady?
21. Node is Ready but pods are not scheduling.
90. What is pod priority and preemption?
91. What are pod topology spread constraints?
111. Troubleshoot unscheduled pods.

---

## Section 6: Security
32. ConfigMap vs Secret.
33. How are Secrets stored and encrypted in etcd?
34. Kubernetes Secrets are base64 encoded. How do you secure them?
35. What is RBAC?
36. How do you provide least privileged access?
37. Role vs ClusterRole.
38. What is a ServiceAccount?
39. What is IRSA?
100. How do you implement network security?
104. How do you secure Kubernetes clusters?
136. Where do you store secrets securely?
137. What is a Network Policy?

---

## Section 7: Resource Management & Autoscaling
40. Resource requests vs limits.
41. What happens if no limits are defined?
42. What is overcommit?
43. HPA vs VPA vs Cluster Autoscaler.
44. Why might HPA not scale?
45. How does HPA get metrics?
46. What is Karpenter?
132. How to tune Cluster Autoscaler?
133. Noisy neighbor problem.
134. One pod consuming excessive resources.
135. Resource isolation using namespaces.

---

## Section 8: Deployments & Release Strategies
47. RollingUpdate, Recreate, Blue-Green, Canary.
48. How do you ensure zero downtime?
49. What deployment strategies do you use?
50. Rolling update stuck.
51. What is a canary release?
97. Canary release unstable.
98. Configuration drift.
99. Why avoid latest tag?
125. Write an NGINX Deployment YAML.
127. Write a Service YAML.

---

## Section 9: Troubleshooting
53. CrashLoopBackOff.
54. ImagePullBackOff.
55. Pod eviction.
56. OOMKilled.
57. logs vs describe.
58. Pod running but app inaccessible.
59. Service unreachable.
60. Pod Pending.
61. Node out of capacity.
62. Pending due to insufficient memory.
63. Check resource usage.
64. Pod scheduled on wrong node.
65. Wrong imagePullSecret.
66. Pod stuck in Terminating.
67. Node NotReady.
68. DiskPressure, MemoryPressure, PIDPressure.
69. Cordon and Drain.
70. Node CPU at 100%.
71. Eviction policy.

---

## Section 10: EKS
84. Upgrade EKS with zero downtime.
85. Pre-checks before EKS upgrade.
86. Control plane crash during upgrade.
109. EKS vs Self-Managed Kubernetes.
110. Node groups in EKS.
121. EKS components.
122. CIDR range used for EKS.
123. Multiple instance types in node groups.
138. Have you provisioned EKS clusters?
142. Multi-region Kubernetes setup.

---

## Section 11: Monitoring & Observability
101. How do you monitor applications and debug memory leaks?
120. Debug kubelet high CPU.
139. Centralized logging and monitoring.
140. Reduce blast radius.
141. Multi-tenant Kubernetes cluster.

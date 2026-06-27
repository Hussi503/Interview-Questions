# Kubernetes Interview Preparation Roadmap (5-6+ Years DevOps)

## Section 1: Kubernetes Architecture & Core Components
### 1. Explain the architecture of Kubernetes with master and worker node components.
### 2. What are the roles of kubelet, kube-apiserver, kube-proxy, etcd, scheduler, and controller manager?
**kube-apiserver** is the central control plane entry point.  it first goes through the API server. It validates requests, enforces authentication and authorization, '
and updates the desired state in etcd. So practically, it acts like the “front door and brain coordinator” of the entire cluster.

**etcd** is the persistent key-value store that holds the entire cluster state. This includes deployments, pods, config maps, secrets, and node information. In production, etcd is 
highly critical because it is the single source of truth; if etcd is corrupted or lost, the cluster state is effectively lost, even if workloads are still running temporarily.

**kube-scheduler** is responsible for deciding where a pod should run. When a new pod is created, it doesn’t have a node assigned. The scheduler evaluates available worker nodes based on CPU,
memory, taints/tolerations, affinity rules, and assigns the best-fit node. In real production clusters, this is key for workload distribution and performance optimization.

**controller-manager** runs all the control loops that continuously reconcile the desired state with the actual state. For example, if a deployment expects 3 replicas and one pod crashes, 
the controller manager detects the mismatch and recreates the pod automatically. It includes controllers like Deployment, ReplicaSet, Node, and Job controllers.

**kubelet** runs on every worker node and is responsible for the actual execution of workloads. Once the scheduler assigns a pod to a node, the kubelet pulls the pod specification 
from the API server and ensures the container is running using the container runtime like containerd. It also continuously performs health checks and reports pod and node status back to the control plane.

**kube-proxy** handles the networking layer inside the cluster. It maintains network rules using iptables or IPVS so that Kubernetes Services can route traffic correctly to backend pods.
In real production scenarios, whenever a service IP is accessed, kube-proxy ensures the request is load balanced across healthy pods.

### 3. What happens internally when you run kubectl apply?
First, the kubectl client reads the YAML manifest and sends it as a REST request to the kube-apiserver. The API server authenticates the request using credentials,
then authorizes it using RBAC policies.

Once validated, the API server performs a schema validation and admission control process. Admission controllers may mutate or validate the request—for example injecting defaults, 
enforcing security policies, or blocking non-compliant deployments.

After that, the API server stores the desired state in etcd, which is the cluster’s source of truth. At this point, the resource is created or updated, but nothing is running yet.

Then the controller-manager detects the new desired state (for example, a Deployment with 3 replicas). It creates or updates ReplicaSets, and ensures the desired number of pods exist. 
If pods are missing, it triggers creation.

Next, the scheduler picks appropriate worker nodes for unscheduled pods based on available resources, taints/tolerations, affinity rules, and policies. It assigns each pod to a specific node.

After scheduling, the kubelet on each selected node pulls the pod spec from the API server and instructs the container runtime (containerd) to start the containers. 
It also starts health checks (liveness/readiness probes) and reports status back to the control plane.

Finally, kube-proxy configures networking rules so the service can route traffic to the newly created pods, making them reachable inside the cluster.

### 87. What is a static pod?
**A static pod is a pod that is directly managed by the kubelet on a specific node instead of being managed by the Kubernetes API server or controllers**
In a production Kubernetes cluster, most pods are created through the API server and managed via Deployments, ReplicaSets, or StatefulSets.
However, static pods are different—they are defined locally on each node in a directory like /etc/kubernetes/manifests.
A static pod is a pod that is directly managed by the kubelet on a specific node instead of being managed by the Kubernetes API server or controllers.”

In a production Kubernetes cluster, most pods are created through the API server and managed via Deployments, ReplicaSets, or StatefulSets. However, 
static pods are different—they are defined locally on each node in a directory like /etc/kubernetes/manifests.

In real-world use cases, static pods are mainly used for critical control plane components in self-managed clusters, such as API server, controller manager, and scheduler.

**So in simple terms, static pods are node-level pods managed directly by kubelet, mainly used for system or control plane components, and they bypass the Kubernetes scheduler.**
### 89. What are Kubernetes operators? Have you used them?
A Kubernetes Operator is a custom controller that extends Kubernetes to automate the complete lifecycle of complex applications. It uses Custom Resource Definitions (CRDs) to 
introduce new resource types into the cluster, and the Operator continuously watches those resources to perform tasks such as installation, configuration, scaling, upgrades, backups, 
failover, and recovery automatically.

In my projects, I have not developed a custom Operator, but I have worked with Operator-managed applications. For example, we deployed monitoring components using Operators, 
and the Operator automatically handled version upgrades, configuration reconciliation, and Pod recovery. From a DevOps perspective, my responsibility was deploying the Operator 
through Helm or manifests, managing RBAC and CRDs, and monitoring its health.

### 90. What are admission controllers? How have you used them?
In Kubernetes, Admission Controllers act as the last checkpoint before a resource is stored in etcd. When someone runs kubectl apply, the request reaches the API Server, 
and before Kubernetes creates or updates the resource, admission controllers validate it or even modify it automatically. This is where we enforce organization-wide security
and compliance policies, so developers don't have to remember every standard manually.

In my projects, we used admission controllers mainly to enforce security and governance. For example, we ensured every pod had CPU and memory requests and limits, 
blocked privileged containers, prevented pods from running as the root user, and allowed images to be pulled only from our approved private Azure Container Registry. 
We also enforced mandatory labels like environment, application, and owner, because our monitoring, cost reporting, and deployment automation depended on those labels. 
If any of these policies were violated, the deployment was rejected immediately before it reached the cluster.

### 102. How does Kubernetes help with reliability
Kubernetes improves reliability by continuously ensuring that applications stay in their desired state, 
even when failures happen.

For example, if a pod crashes or a node goes down, Kubernetes automatically detects the failure and 
recreates the pod on a healthy node without manual intervention. Using Deployments and ReplicaSets, it 
maintains the required number of running replicas, so the application remains available.

It also uses readiness and liveness probes to ensure only healthy pods receive traffic and automatically 
restarts unhealthy containers. Services provide built-in load balancing across healthy pods, preventing 
requests from going to failed instances.

For deployments, Kubernetes supports rolling updates and automatic rollbacks, allowing new versions to be 
released with minimal downtime. If the new version becomes unhealthy, it can be rolled back quickly.

For scaling, Horizontal Pod Autoscaler (HPA) adds or removes pods based on CPU, memory, or custom metrics, 
while the Cluster Autoscaler adds or removes worker nodes when needed. This helps the application remain 
stable during traffic spikes.

In our production environment, we use multiple replicas across different worker nodes, health probes, HPA, 
rolling updates, and Pod Disruption Budgets. This combination ensures high availability, self-healing, and
minimal downtime during failures, deployments, or maintenance.
### 106. How does Kubernetes manage a large number of Docker containers?
Kubernetes manages a large number of Docker containers by grouping them into Pods and continuously 
monitoring their desired state. Instead of managing individual containers manually, we tell Kubernetes how 
many replicas we need, and it takes care of scheduling, scaling, networking, and recovery automatically.

The Scheduler decides which worker node should run each pod based on available CPU, memory, and 
scheduling rules. The kubelet on each node ensures the containers are running as expected, and if a 
container or pod crashes, Kubernetes automatically recreates it. Services provide load balancing across 
multiple pod replicas, while Deployments handle rolling updates and maintain the required number of pods.

In production, we don't manage containers directly. We define the desired state in YAML files, and 
Kubernetes continuously ensures that state is maintained, even when nodes fail or traffic increases. This 
allows us to manage hundreds or even thousands of containers efficiently with minimal manual intervention.
### 115. What is the difference between a Pod and a container?
A container is the smallest unit that runs an application. It contains the application code, runtime, libraries,
and dependencies needed to run the application.

A Pod is the smallest deployable unit in Kubernetes. A pod acts as a wrapper around one or more containers.
All containers inside the same pod share the same IP address, network namespace, storage volumes, and 
lifecycle, allowing them to communicate with each other using localhost.

In production, most pods contain only one container, such as a Java application or an NGINX container. We 
use multiple containers in the same pod only when they are tightly coupled—for example, an application 
container with a sidecar container for log collection, monitoring, or service mesh functionality.

---

## Section 2: Workload Controllers
### 4. What is the difference between a Deployment and a StatefulSet?
The main difference is that Deployment is used for stateless applications, while StatefulSet is used for
stateful applications that need stable identities and persistent storage.

A Deployment creates identical pods that are interchangeable. If a pod fails, Kubernetes creates a new one
with a different name, and any local data is lost unless external storage is used. Deployments are ideal for
applications like web servers, APIs, and microservices.

 A StatefulSet gives each pod a stable identity, including a fixed pod name (like mysql-0, mysql-1) and 
its own persistent volume. Even if a pod is recreated, it keeps the same name and reconnects to the same
storage. StatefulSets also start and stop pods in a specific order, which is important for databases and 
clustered applications.
### 6. What is a DaemonSet? Give examples.
A DaemonSet ensures that one copy of a pod runs on every worker node, making it ideal for node-level services 
like log collection, monitoring, networking, and security agents.
### 7. What is the difference between a ReplicaSet and a Deployment?
### 8. What is the difference between Pod, Deployment, ReplicaSet, StatefulSet, and DaemonSet?

 **Pod** is the smallest deployable unit in Kubernetes. It contains one or more containers that share the 
  same network and storage. In production, we rarely create Pods directly because if a Pod fails, 
**Kubernetes** won't recreate it automatically.

**ReplicaSet** ensures that a specified number of identical Pods are always running. If one Pod crashes, it 
creates a new one. However, ReplicaSets are usually not created directly. They are automatically 
managed by a Deployment.

**Deployment** is what we use for most stateless applications like APIs, web applications, and microservices. 
It manages ReplicaSets and provides rolling updates, rollbacks, scaling, and self-healing. In production,
almost all application workloads are deployed using Deployments.

**StatefulSet** is used for stateful applications such as MySQL, PostgreSQL, MongoDB, Kafka, or Elasticsearch.
Each Pod gets a stable hostname, stable network identity, and its own persistent storage. Even if a Pod is recreated, 
it keeps the same identity and data.

**DaemonSet** ensures that one Pod runs on every worker node. It's mainly used for node-level services like 
log collectors (Fluent Bit), monitoring agents (Node Exporter), and networking plugins (Calico). When a new node 
joins the cluster, Kubernetes automatically schedules a DaemonSet Pod on it.

### 9. Can you attach a volume to a Deployment?
Yes, absolutely. A Deployment itself doesn't directly store data, but the Pods created by the Deployment can mount volumes.
In production, we commonly attach volumes to Deployments for applications that need persistent or shared storage.

For example, if I'm deploying a web application that needs to store uploaded files or read shared configuration files,
I define a PersistentVolumeClaim (PVC) in the Pod template of the Deployment. Kubernetes then mounts the storage into every Pod. 
The actual storage can come from Azure Disk, Azure Files, AWS EBS, EFS, NFS, or any CSI-supported storage.

However, there's an important consideration. If the Deployment has multiple replicas, I need to choose the storage type carefully. 
For Azure Disk or AWS EBS, a volume can generally be attached to only one node at a time (ReadWriteOnce), so it's suitable when only 
one Pod needs to write. If multiple Pods across different nodes need to read and write the same data, I use a shared filesystem like 
Azure Files or AWS EFS, which supports ReadWriteMany.

### 10. What could cause a StatefulSet pod to fail when rescheduled to a different AZ?
One of the most common reasons is the Persistent Volume (PV) is tied to a specific Availability Zone (AZ). In production, 
if a StatefulSet pod is using storage like Azure Managed Disk or AWS EBS, the disk is created in a particular AZ and 
cannot be attached to a node in another AZ.

For example, suppose my StatefulSet pod is running in AZ-1 with an Azure Disk attached. If the node fails and Kubernetes 
tries to reschedule that pod to a node in AZ-2, the scheduler cannot attach the same disk because it's zonal storage. 
As a result, the pod remains in the Pending state, and you'll typically see events like "volume node affinity conflict" 
or "failed to attach volume".

To avoid this in production, we follow a few best practices:

   i. Use a StorageClass with WaitForFirstConsumer, so Kubernetes provisions the volume in the same AZ where the pod is scheduled.
      Spread StatefulSet replicas across AZs, ensuring each replica gets its own volume in its local AZ.
   ii. If the application requires storage accessible from multiple AZs, use a shared storage solution like Azure Files, 
       AWS EFS, or another ReadWriteMany (RWX) storage instead of zonal block storage.
       
### 94. If a DaemonSet pod is pending, how would you troubleshoot?
If a DaemonSet pod is stuck in the Pending state, I start by checking why the scheduler couldn't place it on 
the node rather than guessing.

My first step is to run kubectl describe pod <pod-name> and look at the Events section. In production, this
usually tells me the exact reason, such as insufficient CPU or memory, node selector mismatch, taints without
matching tolerations, or volume-related issues.

Next, I verify whether the target node is Ready using kubectl get nodes. If the node is NotReady or under 
resource pressure, the DaemonSet pod won't be scheduled there. I also check if the DaemonSet has a 
nodeSelector, nodeAffinity, or tolerations configured correctly. I've seen cases where a node label changed 
after a cluster upgrade, and the DaemonSet stopped scheduling because no nodes matched the selector.

If the DaemonSet uses a Persistent Volume, I verify whether the storage can actually be attached to that
node. I also check resource availability using kubectl describe node to see if CPU, memory, or pod limits
have been exhausted.

In one production incident, our log collection DaemonSet stopped scheduling on newly added nodes 
because those nodes had a new taint, and the DaemonSet didn't have the corresponding toleration. 
After updating the DaemonSet with the required toleration, the pods were scheduled immediately.

### 95. Why would a DaemonSet create two pods per node?
Under normal circumstances, a DaemonSet should create only one pod per eligible node. If I see two 
DaemonSet pods on the same node, I immediately investigate because it's usually due to a configuration or
rollout issue rather than expected behavior.

The first thing I check is whether there are multiple DaemonSets deploying the same application. I've seen
cases where an old DaemonSet wasn't deleted after a migration, and both DaemonSets were targeting the 
same nodes, resulting in two pods per node.

Another possibility is a rolling update. During a DaemonSet upgrade, Kubernetes may temporarily run both 
the old and new pod on the same node depending on the update strategy (especially with maxSurge, which
is supported for DaemonSets in newer Kubernetes versions). Once the new pod becomes healthy, the old 
one is terminated, so this is expected and temporary.
### 116. What is the difference between a Job and a CronJob?
A **Job** is used to run a task only once until it completes successfully. Kubernetes ensures the task finishes, 
and if the pod fails, it automatically retries based on the Job configuration. We typically use Jobs for one
-time activities like database migrations, data imports, application initialization, or backup restoration.

A **CronJob**, on the other hand, is used to run the same Job on a schedule, similar to a Linux cron. Instead of
manually triggering it, Kubernetes automatically creates a new Job at the specified time. We use CronJobs for 
recurring tasks like nightly database backups, log cleanup, report generation, certificate renewal scripts, or cache cleanup.

In one of my production environments, we used a Job during every application release to execute database schema migrations before 
deploying the new application version. Separately, we had a CronJob that ran every night to clean old application logs and upload 
database backups to cloud storage. This kept our deployment process automated while ensuring regular maintenance tasks happened 
without manual intervention.

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

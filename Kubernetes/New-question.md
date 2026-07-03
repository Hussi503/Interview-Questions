# Kubernetes Interview Preparation Roadmap (5-6+ Years DevOps)

## Section 1: Kubernetes Architecture & Core Components
# Kubernetes Interview Notes (Production Grade)

---

# 2. What are the roles of kubelet, kube-apiserver, kube-proxy, etcd, scheduler, and controller-manager?

## Production-Grade Interview Answer

### kube-apiserver

The **kube-apiserver** is the central entry point of the Kubernetes control plane. Every request—whether from `kubectl`, CI/CD pipelines, or internal Kubernetes components—first goes through the API server.

Its responsibilities include:

- Authenticating requests.
- Authorizing requests using RBAC.
- Validating Kubernetes objects.
- Running admission controllers.
- Persisting the desired state in **etcd**.

In production, it acts as the **front door and coordinator** of the entire Kubernetes cluster.

---

### etcd

**etcd** is Kubernetes' distributed, highly available key-value database.

It stores the complete cluster state, including:

- Pods
- Deployments
- Services
- ConfigMaps
- Secrets
- Nodes
- RBAC objects

In production, **etcd is the single source of truth**. If etcd is corrupted or lost without a backup, Kubernetes loses the desired cluster state, even though some workloads may continue running temporarily.

---

### kube-scheduler

The **kube-scheduler** decides **which worker node should run a Pod**.

When a new Pod is created without a node assignment, the scheduler evaluates available worker nodes based on:

- CPU availability
- Memory availability
- Taints and tolerations
- Node affinity and anti-affinity
- Pod affinity and anti-affinity
- Resource constraints
- Scheduling policies

It then selects the most suitable node and assigns the Pod.

In production, efficient scheduling improves workload distribution, resource utilization, and application performance.

---

### controller-manager

The **controller-manager** runs Kubernetes control loops that continuously compare the **desired state** with the **actual state**.

Examples include:

- Deployment Controller
- ReplicaSet Controller
- Node Controller
- Job Controller
- StatefulSet Controller

For example, if a Deployment specifies **3 replicas** and one Pod crashes, the controller-manager detects the difference and automatically creates a replacement Pod.

This reconciliation process keeps the cluster self-healing.

---

### kubelet

The **kubelet** runs on every worker node.

Its responsibilities include:

- Watching for Pods assigned to its node.
- Pulling Pod specifications from the API Server.
- Starting containers through the container runtime (containerd).
- Executing liveness, readiness, and startup probes.
- Reporting Pod and Node status back to the control plane.

The kubelet ensures that containers are running exactly as specified.

---

### kube-proxy

The **kube-proxy** manages networking on every worker node.

It:

- Creates networking rules using **iptables** or **IPVS**.
- Implements Kubernetes Service networking.
- Load balances traffic across healthy backend Pods.
- Updates routing rules as Pods are created or deleted.

In production, whenever a Service IP receives traffic, kube-proxy routes requests to healthy Pods automatically.

---

# Easy Way to Remember

| Component | Responsibility |
|-----------|----------------|
| kube-apiserver | Entry point of the cluster; validates and processes all requests |
| etcd | Stores the entire cluster state |
| kube-scheduler | Chooses the best node for new Pods |
| controller-manager | Maintains the desired state by running controllers |
| kubelet | Runs Pods on worker nodes |
| kube-proxy | Provides Service networking and load balancing |

---

# 3. What happens internally when you run `kubectl apply`?

## Production-Grade Interview Answer

When `kubectl apply` is executed, Kubernetes performs several steps internally.

### Step 1 – kubectl sends the request

The `kubectl` client reads the YAML manifest and sends it as a **REST API request** to the **kube-apiserver**.

---

### Step 2 – Authentication & Authorization

The API Server:

- Authenticates the user.
- Authorizes the request using **RBAC**.

If the user lacks permissions, the request is rejected immediately.

---

### Step 3 – Validation & Admission Controllers

The API Server performs:

- Schema validation.
- Resource validation.
- Admission Controller processing.

Admission Controllers may:

- Inject default values.
- Enforce security policies.
- Validate configurations.
- Reject non-compliant resources.

---

### Step 4 – Store Desired State

After validation, the API Server stores the desired state in **etcd**.

At this stage, Kubernetes knows **what should exist**, but nothing has been scheduled yet.

---

### Step 5 – Controller Manager

The **controller-manager** notices the new resource.

For example:

Deployment → ReplicaSet → Pods

If the Deployment requests three replicas, it creates a ReplicaSet that creates three Pods.

---

### Step 6 – Scheduling

The **kube-scheduler** evaluates available worker nodes based on:

- CPU
- Memory
- Taints/Tolerations
- Affinity Rules
- Scheduling Policies

It assigns each Pod to the best worker node.

---

### Step 7 – kubelet Starts Containers

The **kubelet** on each assigned node:

- Retrieves the Pod specification.
- Pulls container images.
- Starts containers through **containerd**.
- Executes health probes.
- Reports status to the API Server.

---

### Step 8 – Networking

The **kube-proxy** configures Service networking using **iptables/IPVS**.

Applications become reachable through Kubernetes Services.

---

# Flow Diagram

```text
kubectl apply
        │
        ▼
kube-apiserver
        │
        ▼
Authentication + RBAC
        │
        ▼
Validation + Admission Controllers
        │
        ▼
etcd (Desired State Stored)
        │
        ▼
Controller Manager
        │
        ▼
ReplicaSet
        │
        ▼
Scheduler
        │
        ▼
Worker Node
        │
        ▼
kubelet
        │
        ▼
containerd
        │
        ▼
Running Pods
        │
        ▼
kube-proxy
        │
        ▼
Service Traffic
```

---

# 87. What is a Static Pod?

## Production-Grade Interview Answer

A **Static Pod** is a Pod that is **managed directly by the kubelet** on a specific node instead of being managed through the Kubernetes API Server or controllers.

Unlike Deployments or ReplicaSets, Static Pods are defined as local manifest files on the node, typically in:

```text
/etc/kubernetes/manifests
```

The kubelet continuously monitors this directory.

If a manifest is:

- Added → kubelet creates the Pod.
- Modified → kubelet recreates the Pod.
- Deleted → kubelet removes the Pod.

Static Pods are **not scheduled** by the Kubernetes Scheduler because they are already tied to a specific node.

### Production Use Case

Static Pods are commonly used for Kubernetes control plane components in self-managed clusters, including:

- kube-apiserver
- kube-controller-manager
- kube-scheduler
- etcd

These components must always run, even before the API Server becomes available.

---

# Easy Way to Remember

- Managed by **kubelet**
- Stored as **local manifest files**
- Runs on **one specific node**
- Not scheduled by the **scheduler**
- Mainly used for **control plane components**

---

# 89. What are Kubernetes Operators? Have you used them?

## Production-Grade Interview Answer

A **Kubernetes Operator** is a custom controller that extends Kubernetes to automate the lifecycle of complex applications.

Operators use **Custom Resource Definitions (CRDs)** to introduce new resource types into the cluster.

The Operator continuously watches these resources and automatically performs operations such as:

- Installation
- Configuration
- Scaling
- Upgrades
- Backups
- Failover
- Recovery

This reduces manual operational work.

### My Experience

I have not developed a custom Operator, but I have worked with **Operator-managed applications**.

For example:

- Deploying monitoring stacks through Operators.
- Managing Operators using Helm.
- Installing CRDs.
- Configuring RBAC.
- Monitoring Operator health.
- Performing Operator upgrades.

The Operator automatically handled reconciliation, upgrades, and Pod recovery.

---

# 90. What are Admission Controllers? How have you used them?

## Production-Grade Interview Answer

**Admission Controllers** act as the **final checkpoint** before a resource is stored in **etcd**.

When someone executes `kubectl apply`, the request reaches the API Server.

Before Kubernetes creates or updates the resource, Admission Controllers can:

- Validate requests.
- Modify requests.
- Reject requests that violate organizational policies.

They help enforce security, governance, and compliance across the cluster.

### Production Use Cases

In our projects, Admission Controllers were used to enforce:

- CPU requests and limits.
- Memory requests and limits.
- Blocking privileged containers.
- Preventing containers from running as the root user.
- Allowing images only from the approved private Azure Container Registry.
- Mandatory labels such as:
  - environment
  - application
  - owner

If any deployment violated these policies, it was rejected before reaching the cluster.

This ensured consistent security standards across all environments without relying on developers to remember every policy.

---

# Easy Way to Remember

Admission Controllers = **Validate → Mutate → Enforce**

They act as Kubernetes' final security and compliance gate before objects are persisted in **etcd**.

### 🔴102. How does Kubernetes help with reliability
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
### 🔴106. How does Kubernetes manage a large number of Docker containers?
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
### 🔴 4. What is the difference between a Deployment and a StatefulSet?
The main difference is that Deployment is used for stateless applications, while StatefulSet is used for
stateful applications that need stable identities and persistent storage.

A Deployment creates identical pods that are interchangeable. If a pod fails, Kubernetes creates a new one
with a different name, and any local data is lost unless external storage is used. Deployments are ideal for
applications like web servers, APIs, and microservices.

 A StatefulSet gives each pod a stable identity, including a fixed pod name (like mysql-0, mysql-1) and 
its own persistent volume. Even if a pod is recreated, it keeps the same name and reconnects to the same
storage. StatefulSets also start and stop pods in a specific order, which is important for databases and 
clustered applications.
### 🔴5. What is a DaemonSet? Give examples of when you would use it.
A DaemonSet ensures that one copy of a pod runs on every worker node, making it ideal for node-level services 
like log collection, monitoring, networking, and security agents.
### 🔴7. What is the difference between a ReplicaSet and a Deployment?
### 🔴8. What is the difference between Pod, Deployment, ReplicaSet, StatefulSet, and DaemonSet?

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

### 8. Can you attach a volume to a Deployment?  How is it different from a StatefulSet
Yes, absolutely. A Deployment itself doesn't directly store data, but the Pods created by the Deployment can mount volumes.
In production, we commonly attach volumes to Deployments for applications that need persistent or shared storage.

For example, if I'm deploying a web application that needs to store uploaded files or read shared configuration files,
I define a PersistentVolumeClaim (PVC) in the Pod template of the Deployment. Kubernetes then mounts the storage into every Pod. 
The actual storage can come from Azure Disk, Azure Files, AWS EBS, EFS, NFS, or any CSI-supported storage.

However, there's an important consideration. If the Deployment has multiple replicas, I need to choose the storage type carefully. 
For Azure Disk or AWS EBS, a volume can generally be attached to only one node at a time (ReadWriteOnce), so it's suitable when only 
one Pod needs to write. If multiple Pods across different nodes need to read and write the same data, I use a shared filesystem like 
Azure Files or AWS EFS, which supports ReadWriteMany.

### 🔴9. What could cause a StatefulSet pod to fail when rescheduled to a different AZ?
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
       
### 🔴94. If a DaemonSet pod is pending, how would you troubleshoot?
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

### 🔴95. Why would a DaemonSet create two pods per node?
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
### 🔴116. What is the difference between a Job and a CronJob?
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
### 🔴10. How do PV and PVC behave across zones?
The behavior of Persistent Volumes (PVs) and Persistent Volume Claims (PVCs) across Availability Zones
depends on the type of storage backend being used.

In production, if the PVC is backed by Azure Managed Disk or AWS EBS, the volume is zonal. That means
the PV is created in a specific Availability Zone and can only be attached to worker nodes in that same zone. 
If Kubernetes tries to schedule the pod in another AZ, the volume cannot be attached, and the pod stays in 
the Pending state with errors like "volume node affinity conflict" or "failed to attach volume."

To avoid this, we use a StorageClass with volumeBindingMode: WaitForFirstConsumer. This ensures
Kubernetes first decides which node the pod will run on, and only then provisions the disk in that node's
Availability Zone.

If the application needs to be accessible from multiple AZs, we don't use zonal block storage. Instead, we 
use shared storage like Azure Files or AWS EFS, which supports ReadWriteMany (RWX). In that case, pods 
running in different AZs can mount the same storage simultaneously. 

### 🔴117. What is the difference between a PV and PVC?
Persistent Volume (**PV**) is the actual storage available in the Kubernetes cluster.
Persistent Volume Claim (**PVC**) is a request for that storage made by an application.

In production, developers never directly use a PV. They create a PVC by specifying the required size, access 
mode, and StorageClass. Kubernetes then dynamically provisions a PV using the configured storage
backend, such as Azure Managed Disk, Azure Files, AWS EBS, or AWS EFS, and binds it to the PVC. The
application pod simply mounts the PVC and doesn't need to know where the storage actually resides.

### 🔴118. What is the difference between a StorageClass and a PV?
A StorageClass is a template or blueprint that tells Kubernetes how to create storage. It defines things like 
the storage provisioner (Azure Disk, Azure Files, AWS EBS, AWS EFS), disk type (SSD/HDD), reclaim policy,
volume binding mode, and other storage parameters. It does not provide storage by itself.

A Persistent Volume (PV) is the actual storage resource that gets created. It could be an Azure Managed Disk,
Azure Files share, AWS EBS volume, or NFS share. This is what ultimately stores the application's data.

12. What is a Pod Disruption Budget (PDB)?
A Pod Disruption Budget (PDB) is used to protect application availability during voluntary disruptions in
Kubernetes. It tells Kubernetes the minimum number of pods that must remain available or the maximum
number of pods that can be unavailable during operations like node draining, cluster upgrades, or cluster
autoscaler scale-down.

For example, if my application has 5 replicas, I can configure a PDB with minAvailable: 4 or 
maxUnavailable: 1. This means Kubernetes will never voluntarily evict more than one pod at a time. As a
result, even during maintenance or upgrades, the application continues serving traffic without significant
downtime.
### 🔴52. How do you handle database migrations safely in Kubernetes?
I run database migrations as a separate Kubernetes Job or CI/CD pipeline stage before deploying the application,
with backups, backward-compatible changes, and deployment gates so the application is updated only after the 
database migration succeeds.

### 🔴80. What happens if etcd is corrupted? How do you recover?
etcd is the brain of the Kubernetes cluster. It stores the entire cluster state, including Pods, Deployments,
Services, ConfigMaps, Secrets, RBAC, and node information. If etcd becomes corrupted or unavailable, the
Kubernetes API Server cannot read or write the cluster state. As a result, you won't be able to create, update,
 or delete Kubernetes resources. Existing application pods may continue running for some time, but cluster
management operations stop.

In production, the recovery approach depends on whether it's a managed Kubernetes service or a self-managed cluster.

For AKS, EKS, or GKE, etcd is fully managed by the cloud provider, so I don't have direct access to it. If there's an
etcd issue, I rely on the cloud provider's control plane recovery mechanisms and open a support case if required.

For a self-managed Kubernetes cluster, we regularly take etcd snapshots. If etcd gets corrupted, I stop the API Server, 
restore the latest healthy snapshot using etcdctl snapshot restore, replace the corrupted data directory with the restored
one, and restart the etcd and control plane components. After recovery, I verify the cluster health by checking nodes,
workloads, and application functionality.

### 🔴82. How do you backup and restore etcd?
In a self-managed Kubernetes cluster, I back up etcd by taking regular snapshots using etcdctl. Since etcd
contains the entire cluster state, these backups are critical for disaster recovery. After taking the snapshot, I
 store it in a secure location such as remote storage or object storage and periodically verify that it can be
restored.

To take a backup, I use a command like:

**ETCDCTL_API=3 etcdctl snapshot save etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=ca.crt \
  --cert=server.crt \
  --key=server.key**

 If etcd becomes corrupted, I restore it using:

**ETCDCTL_API=3 etcdctl snapshot restore etcd-snapshot.db**

After the restore, I update etcd to use the restored data directory, restart the etcd service, and then verify that the API Server, nodes, and workloads are healthy.


## Section 4: Services, Ingress & Networking
### 🔴11. What is a headless service?  how does it work?
A Headless Service is a Kubernetes Service created by setting clusterIP: None. Unlike a normal Service, it 
does not allocate a virtual ClusterIP and does not perform load balancing. Instead, when a client performs
a DNS lookup, Kubernetes returns the IP addresses of all matching Pods directly.

We primarily use Headless Services with StatefulSets, where each Pod requires a stable network identity. For 
example, in databases like MongoDB, Cassandra, Kafka, Elasticsearch, or Redis Cluster, each node needs to 
communicate with a specific peer rather than through a load balancer.
    
### 🔴22. What are the different Service types?
**1. ClusterIP (Default)**
ClusterIP exposes the application only within the Kubernetes cluster. Kubernetes assigns a virtual IP,
and kube-proxy load balances traffic across healthy Pods behind the Service. This is the most commonly used
Service type for internal microservice communication. For example, if a Payment service needs to call an
Order service, it uses a ClusterIP Service.

**spec:
  type: ClusterIP**

**2. NodePort**
NodePort exposes the application on a static port (30000–32767) on every worker node. Traffic sent to
<NodeIP>:NodePort is forwarded to the Service and then to a backend Pod. It's mainly useful for
development, testing, or environments without a cloud load balancer. In production, it's rarely exposed
directly because it has security and scalability limitations.

**spec:
  type: NodePort**

**Example:

http://192.168.1.20:30080**

**3. LoadBalance**r
LoadBalancer is commonly used in cloud environments such as Azure AKS, AWS EKS, or Google GKE. When 
you create this Service, Kubernetes asks the cloud provider to provision an external load balancer with a
public IP, which forwards traffic to the worker nodes and then to the Pods. This is the standard way to 
expose public-facing applications.

**spec:
  type: LoadBalancer**

Flow:

Internet
    ↓
Azure Load Balancer / AWS ELB
    ↓
Worker Nodes
    ↓
Pods

**4. ExternalName**
ExternalName doesn't create a proxy or allocate a ClusterIP. Instead, it maps the Service name to an external
DNS name using a CNAME record. This is useful when applications inside Kubernetes need to access an external 
service without changing application configuration.

**spec:
  type: ExternalName
  externalName: db.company.com**
  
### 🔴23. What is the difference between Ingress and LoadBalancer?
 The main difference is that a LoadBalancer Service exposes a single Kubernetes Service externally, whereas
 Ingress provides Layer 7 (HTTP/HTTPS) routing and can expose multiple Services through a single 
 external Load Balancer.

In production, we almost never create a separate LoadBalancer for every microservice because it increases 
cost and becomes difficult to manage. Instead, we deploy an Ingress Controller (such as NGINX Ingress 
Controller or cloud-native controllers), expose only the Ingress Controller using a LoadBalancer Service,
and let the Ingress resource route traffic to the appropriate backend Services based on the hostname or URL 
path.

| LoadBalancer                   | Ingress                                     |
| ------------------------------ | ------------------------------------------- |
| Exposes one Service externally | Routes traffic to multiple Services         |
| Layer 4 (TCP/UDP)              | Layer 7 (HTTP/HTTPS)                        |
| Gets its own external IP       | Shares one external IP across many Services |
| No path or host-based routing  | Supports host and path-based routing        |
| No built-in SSL termination    | Supports SSL termination and TLS            |
| Higher cost if many Services   | More cost-effective for many web services   |

### 🔴24. How does Ingress work? How do you create an ingress controller in AWS EKS?

Ingress itself doesn't handle traffic. It is simply a set of routing rules. The actual traffic handling is done by 
an Ingress Controller such as NGINX Ingress Controller, Azure Application Gateway Ingress Controller
(AGIC), or Traefik.
                     Internet
                        │
                 Public IP Address
                        │
            Azure Load Balancer (Service Type=LoadBalancer)
                        │
             NGINX Ingress Controller Pods
                        │
        Reads Ingress rules from Kubernetes API
                        │
        ┌───────────────┼────────────────┐
        │               │                │
    frontend        order-service    payment-service
    (ClusterIP)      (ClusterIP)       (ClusterIP)
        │               │                │
      Pods            Pods             Pods

**Step-by-Step Flow**
**Step 1**: User sends a request
https://shop.company.com/orders

The request first reaches the public IP of the Kubernetes cluster.

**Step 2**: Azure Load Balancer receives the request

The LoadBalancer Service exposes the NGINX Ingress Controller.

                         Internet
                            │
                     Azure Load Balancer
                            │
                   NGINX Ingress Controller

Notice that the Load Balancer doesn't know about your applications. It only forwards traffic to the Ingress Controller Pods.

**Step 3**: Ingress Controller receives the request

The Ingress Controller continuously watches the Kubernetes API Server.

Whenever you create or update an Ingress resource like:

apiVersion: networking.k8s.io/v1
kind: Ingress
spec:
  rules:
  - host: shop.company.com
    http:
      paths:
      - path: /orders
        backend:
          service:
            name: order-service
            port:
              number: 80

The controller immediately updates its routing configuration.

**Step 4**: Route Matching

           NGINX checks:

                    Hostname
                    URL Path
                    TLS Configuration

           Example:

                 shop.company.com/
                        ↓
                 Frontend Service

               shop.company.com/orders
                        ↓
                    Order Service

                shop.company.com/payment
                         ↓
                    Payment Service
**Step 5**: Forward to ClusterIP Service

The request is forwarded to the corresponding ClusterIP Service.

Ingress Controller
        │
        ▼
Order Service (ClusterIP)
**Step 6**: kube-proxy Load Balances

The ClusterIP Service selects one of the healthy Pods.

               Order Service
                     │
                ┌────┴─────┐
                │          │
               Pod-1    Pod-2

kube-proxy distributes traffic to an available Pod.


**step 3: eks

| Step                                                    | AKS (Azure)                                                                    | EKS (AWS)                                                                                  |
| ------------------------------------------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| **1. User Request**                                     | User hits `https://shop.company.com`                                           | Same                                                                                       |
| **2. DNS**                                              | Azure DNS/Public DNS → Azure Load Balancer                                     | Route 53/Public DNS → AWS ALB                                                              |
| **3. Who watches Ingress?**                             | NGINX Ingress Controller (or AGIC if using Application Gateway)                | AWS Load Balancer Controller (or NGINX Ingress Controller)                                 |
| **4. What happens after `kubectl apply ingress.yaml`?** | NGINX updates its routing configuration (`nginx.conf`)                         | AWS Load Balancer Controller calls AWS APIs to create/update ALB, listeners, target groups |
| **5. External Load Balancer**                           | Azure Load Balancer already exists because the NGINX Service is `LoadBalancer` | ALB can be created automatically from the Ingress (AWS-native approach)                    |
| **6. Routing**                                          | NGINX routes traffic to ClusterIP Services                                     | ALB listener rules (or NGINX if using NGINX Ingress) route traffic to Services             |
| **7. Service**                                          | ClusterIP                                                                      | ClusterIP                                                                                  |
| **8. Pod Selection**                                    | kube-proxy selects a healthy Pod                                               | kube-proxy selects a healthy Pod                                                           |


      

### 🔴25. How will you manage SSL certificates for ALB in EKS?
In EKS, I manage SSL certificates using AWS Certificate Manager (ACM) and the AWS Load Balancer 
Controller. This is the AWS-recommended approach because ACM handles certificate lifecycle, including renewals, 
and the controller automatically attaches the certificate to the Application Load Balancer (ALB).

The process is:

  i .Request or import the certificate into ACM.
               If the domain is managed in Route 53, DNS validation is straightforward.
               For external DNS providers, add the required CNAME records for validation.
  ii. Install the AWS Load Balancer Controller in the EKS cluster with the appropriate IAM permissions
     (using IAM Roles for Service Accounts is the recommended practice).
  iii. Reference the ACM certificate in the Ingress using annotations.
                  ## AWS ALB Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:region:account:certificate/xxxxxxxx
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'
spec:
  ingressClassName: alb
```

### Explanation

| Field | Purpose |
|-------|---------|
| `apiVersion` | Kubernetes API version for the Ingress resource. |
| `kind` | Specifies that this resource is an Ingress. |
| `metadata.name` | Name of the Ingress object. |
| `kubernetes.io/ingress.class` | Tells Kubernetes to use the AWS ALB Ingress Controller. |
| `alb.ingress.kubernetes.io/certificate-arn` | ACM certificate ARN used for HTTPS. |
| `alb.ingress.kubernetes.io/listen-ports` | Configures the ALB to listen on port 443. |
| `alb.ingress.kubernetes.io/ssl-redirect` | Redirects HTTP traffic to HTTPS. |
| `spec.ingressClassName` | Specifies the IngressClass (`alb`) to handle this Ingress. |

When I apply this manifest:

**kubectl apply -f ingress.yaml**

The AWS Load Balancer Controller detects the Ingress, configures the ALB HTTPS listener, attaches the ACM certificate, and optionally creates an HTTP listener that redirects traffic to HTTPS.
 
### 🔴26. What type of Load Balancer is created when using Ingress in EKS?

The type of Load Balancer created in Amazon EKS depends on the **Ingress Controller** being used.

---

## 1. AWS Load Balancer Controller (Most Common in Production)

If you're using the **AWS Load Balancer Controller**, Kubernetes automatically provisions an **Application Load Balancer (ALB)** whenever you create an Ingress resource.

### Why ALB?

Ingress operates at **Layer 7 (HTTP/HTTPS)**, and ALB is designed for Layer 7 traffic. It provides:

- Host-based routing
- Path-based routing
- SSL/TLS termination
- HTTP to HTTPS redirection
- Integration with AWS Certificate Manager (ACM)
- AWS WAF integration

### Architecture

```text
                Internet
                    │
                 Route53
                    │
      Application Load Balancer (ALB)
                    │
     AWS Load Balancer Controller
                    │
          ClusterIP Services
                    │
                  Pods
```

---

## 2. NGINX Ingress Controller

If you're using the **NGINX Ingress Controller**, Kubernetes **does not create an ALB automatically**.

Instead:

- The NGINX Ingress Controller is exposed using a **Service of type LoadBalancer**.
- AWS provisions a **Network Load Balancer (NLB)** by default (or a **Classic Load Balancer (CLB)** in older Kubernetes versions or based on specific annotations).
- The NLB forwards traffic to the NGINX Ingress Controller.
- The NGINX Ingress Controller performs the Layer 7 routing based on the Ingress rules.

### Architecture

```text
                Internet
                    │
     Network Load Balancer (NLB)
                    │
      NGINX Ingress Controller
                    │
          ClusterIP Services
                    │
                  Pods
```

---

## Interview Answer (1 Minute)

> In EKS, the type of Load Balancer created depends on the Ingress Controller. In production, we commonly use the **AWS Load Balancer Controller**, which automatically
>  provisions an **Application Load Balancer (ALB)** whenever an Ingress resource is created. ALB is a Layer 7 load balancer and supports host-based routing, path-based
> routing, SSL termination using ACM, HTTP-to-HTTPS redirection, and AWS WAF integration. Traffic flows from Route53 to the ALB, then through the AWS Load Balancer
> Controller to Kubernetes ClusterIP Services and finally to the Pods.
>
> If we're using the **NGINX Ingress Controller**, Kubernetes doesn't create an ALB. Instead, the NGINX controller is exposed through a **LoadBalancer Service**,
> which provisions a **Network Load Balancer (NLB)** by default. The NLB forwards traffic to the NGINX Ingress Controller, and NGINX handles all the Layer 7 routing internally.
   

      
 
### 🔴27. Where will you mention the Load Balancer type in Ingress YAML?

The **Load Balancer type is not mentioned directly** in the Ingress YAML.

In production, the type of Load Balancer is determined by the **Ingress Controller** and the **IngressClass** being used.

For example:

- If you're using the **AWS Load Balancer Controller**, creating an Ingress resource automatically provisions an **Application Load Balancer (ALB)**.
- The controller watches Ingress resources that have the appropriate **IngressClass** (`alb`) and creates the ALB automatically.
- We **do not specify ALB or NLB as a field** in the Ingress specification.

If we need to customize the ALB, such as:

- Internet-facing or Internal
- IP or Instance target type
- SSL/TLS configuration
- Listener ports
- ACM Certificate
- Subnets
- AWS WAF

we configure those using **annotations** in the Ingress YAML.

---

## Production Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sample-ingress
  annotations:
    kubernetes.io/ingress.class: alb

    # ALB Scheme
    alb.ingress.kubernetes.io/scheme: internet-facing

    # Target Type
    alb.ingress.kubernetes.io/target-type: ip

    # Listener Ports
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'

    # SSL Certificate (ACM)
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:region:account:certificate/xxxxxxxx

spec:
  ingressClassName: alb

  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

---

## Key Point

The following line tells the **AWS Load Balancer Controller** to manage this Ingress:

```yaml
spec:
  ingressClassName: alb
```

or in older Kubernetes versions:

```yaml
metadata:
  annotations:
    kubernetes.io/ingress.class: alb
```

This **does not explicitly specify the load balancer type**. Instead, it tells Kubernetes which **Ingress Controller** should process the Ingress resource. Since the AWS Load Balancer Controller is responsible for the `alb` IngressClass, it automatically provisions an **Application Load Balancer (ALB)**.

---

## Interview Answer (1 Minute)

> We don't explicitly mention **ALB** or **NLB** anywhere in the Ingress specification. The type of load balancer is determined by the **Ingress Controller** associated with the IngressClass. In AWS EKS, if we're using the **AWS Load Balancer Controller**, we specify `ingressClassName: alb` (or the older `kubernetes.io/ingress.class: alb` annotation). The controller watches those Ingress resources and automatically provisions an **Application Load Balancer (ALB)**. If we need to customize the ALB—for example, making it internet-facing or internal, selecting IP or instance target type, configuring SSL certificates, listener ports, subnets, or AWS WAF—we do that using **ALB-specific annotations** in the Ingress YAML, not by specifying the load balancer type directly.

### 🔴28 How does kube-proxy work?

kube-proxy is a networking component that runs as a DaemonSet, meaning one pod runs on every worker node. Its primary responsibility is to implement the Kubernetes Service abstraction by routing traffic from a Service IP (ClusterIP) to one of the healthy backend Pods.

Internally, kube-proxy continuously watches the Kubernetes API Server for changes to Services and Endpoints/EndpointSlices. Whenever a new Service or Pod is created, deleted, or updated, kube-proxy automatically updates the node's networking rules. In production, it typically operates in iptables mode or IPVS mode, where it programs Linux kernel networking rules instead of acting as a userspace proxy, making packet forwarding highly efficient.

---

## Real-Time Flow

Let's say an application Pod wants to access a Service called **payment-service**.

- The application sends traffic to the ClusterIP of **payment-service**.
- The request reaches the worker node.
- kube-proxy has already created iptables/IPVS rules for that Service.
- These rules select one of the healthy backend Pods behind the Service using load balancing.
- The packet is forwarded directly to the selected Pod.
- If a Pod fails or a new Pod is added due to autoscaling, kube-proxy updates the rules automatically without any manual intervention.

So, applications always communicate with the Service IP, while kube-proxy transparently routes traffic to healthy Pods.

### 🔴29.How does CoreDNS provide service discovery in Kubernetes?

In Kubernetes, CoreDNS provides service discovery by automatically resolving Service names to their corresponding ClusterIP. Instead of applications communicating using Pod IPs, which are dynamic and change when Pods restart, they communicate using the stable Service name. CoreDNS continuously watches the Kubernetes API Server for Services and Endpoints and maintains DNS records dynamically. When a Pod makes a DNS request for a Service, CoreDNS returns the Service's ClusterIP, and then kube-proxy routes the traffic to one of the healthy backend Pods.

---

## Real-Time Flow

Suppose we have two microservices:

- frontend
- payment-service

The frontend application calls:

```text
http://payment-service
```

Here's what happens internally:

- The frontend Pod sends a DNS query for payment-service.
- The DNS request goes to CoreDNS.
- CoreDNS looks up the Service in the Kubernetes API and returns its ClusterIP.
- The frontend sends traffic to that ClusterIP.
- kube-proxy uses its iptables/IPVS rules to load balance the request to one of the healthy payment Pods.
- If Pods scale up, restart, or fail, CoreDNS and kube-proxy automatically update their records, so the application continues working without any configuration changes.
### 🔴30. Explain the Kubernetes networking model.

The Kubernetes networking model is based on a simple principle: every Pod gets its own unique IP address, and every Pod can communicate directly with every other Pod across the cluster without NAT. This is achieved using a Container Network Interface (CNI) plugin such as Calico, Cilium, or Amazon VPC CNI in EKS. The CNI is responsible for assigning Pod IPs, configuring network interfaces, and ensuring connectivity between Pods running on different worker nodes.

---

## Key Components

- **CNI Plugin** – Assigns Pod IPs and enables Pod-to-Pod communication across nodes.
- **CoreDNS** – Provides DNS-based service discovery by resolving Service names to ClusterIPs.
- **kube-proxy** – Implements Kubernetes Services by routing traffic from the Service IP to healthy backend Pods using iptables or IPVS.
- **Network Policies** – Control which Pods are allowed to communicate with each other, providing network-level security.

---

## Production Best Practices

In production, we use a reliable CNI plugin based on the environment—for example, Amazon VPC CNI in EKS or Calico when advanced network policies are required. We never rely on Pod IPs because Pods are recreated during deployments or failures. Instead, all application communication happens through Services, and we enforce Network Policies to restrict unnecessary Pod-to-Pod communication following the principle of least privilege.

### 🔴31. How do you restrict pod-to-pod communication using Network Policies?

In production, we use Network Policies to implement micro-segmentation, meaning Pods are only allowed to communicate with the Pods they actually need. By default, Kubernetes allows all Pod-to-Pod communication. Once a Network Policy selects a Pod, that Pod becomes isolated, and only the traffic explicitly allowed by the policy is permitted. This helps enforce the principle of least privilege and limits the impact of a compromised Pod.

---

## Production Best Practices

In production, we first apply a default deny Network Policy for each namespace and then create explicit allow rules for required communication. This prevents accidental exposure of internal services. We also ensure our CNI plugin supports Network Policies—for example, Calico, Cilium, or Amazon VPC CNI with Network Policy support. We use labels consistently because policies are label-based, and we validate them thoroughly to avoid breaking application communication.

### 🔴72. Service reachable internally but not externally.

If a Service is reachable internally but not externally, it tells me the application and Service are most likely healthy, and the issue is somewhere in the ingress path. In production, I troubleshoot layer by layer instead of guessing. I start from the application and move outward until I identify where the traffic is getting blocked.

First, I verify that the application is healthy by accessing the Service from another Pod using **curl**. If that works, I know the Pods, Service, and kube-proxy are functioning correctly. Next, I check whether the Service type is correct. If external access is required, it should typically be exposed through an **Ingress** or a **LoadBalancer** Service, not just a **ClusterIP**.

If we're using Ingress, I verify that the Ingress resource is created correctly, the Ingress Controller is running, and the Load Balancer has been provisioned successfully. I then check whether the Service name and port in the Ingress backend match the actual Service. A wrong backend port is a very common production issue.

Next, I verify the cloud Load Balancer. In EKS, I ensure the ALB or NLB is in the Active state, its target group shows healthy targets, and the security groups allow inbound traffic on ports **80** and **443**. I also check that the worker node security groups and NACLs are not blocking traffic. If DNS is used, I confirm that the domain correctly resolves to the Load Balancer's DNS name.

Finally, I review the Ingress Controller logs and application logs for errors like **404**, **502**, or **503**, which usually indicate routing issues, unhealthy Pods, or backend connectivity problems.

---

## Production Troubleshooting Order

- Verify Pod health.
- Verify Service and Endpoints (`kubectl get endpoints`).
- Test the Service internally using **curl**.
- Check Service type (ClusterIP/NodePort/LoadBalancer).
- Verify Ingress configuration and backend mapping.
- Check Ingress Controller logs.
- Verify Load Balancer health checks and target groups.
- Check Security Groups, NACLs, and firewall rules.
- Verify DNS resolution and SSL certificate if HTTPS is used.

### 🔴74. Ingress returns 502 error — what are possible reasons.

A 502 Bad Gateway from an Ingress means the Ingress Controller was able to receive the request, but it couldn't get a valid response from the backend Service. In production, I don't assume it's an application issue—I troubleshoot each layer systematically because a 502 can occur due to multiple reasons.

The first thing I check is whether the backend Pods are healthy and Ready. If Pods are in CrashLoopBackOff, failing readiness probes, or not in the Ready state, the Ingress has no healthy backend to forward traffic to. Next, I verify that the Service has healthy Endpoints using `kubectl get endpoints`. If the Endpoints list is empty, it's usually a label mismatch between the Service selector and the Pods.

If the Endpoints are present, I check whether the Ingress backend configuration is correct. A wrong Service name or incorrect target port is one of the most common production mistakes. For example, if the Service exposes port 80 but forwards to 8080, and the Ingress points to the wrong port, the request reaches the Service but fails to connect to the application.

I then verify that the application is actually listening on the expected port by testing it from another Pod using `curl`. If the Service works internally but not through the Ingress, I review the Ingress Controller logs because they usually indicate whether the issue is connection refused, upstream timeout, or no healthy upstreams.

In cloud environments like EKS, I also check the ALB Target Group health. If health checks are failing because of an incorrect health check path, port, or application response, the ALB marks all targets as unhealthy and returns 502. Finally, I verify Network Policies, Security Groups, and SSL/TLS configuration, especially if backend HTTPS is configured incorrectly.

---

## Common Production Causes

- Backend Pods are not Ready or are crashing.
- Service has no Endpoints due to selector mismatch.
- Incorrect Service name, port, or targetPort in the Ingress.
- Application is not listening on the configured port.
- Failed ALB/NGINX health checks.
- Backend connection timeout or application not responding.
- SSL/TLS mismatch between the Ingress and backend Service.
- Network Policies or firewall rules blocking traffic.

---

## Commands I Use During Troubleshooting

```bash
kubectl get pods
kubectl get svc
kubectl get endpoints
kubectl describe ingress <ingress-name>
kubectl logs <ingress-controller-pod>
kubectl exec -it <test-pod> -- curl http://<service-name>:<port>
```

### 🔴75. How do you debug CNI plugin issues?

When I suspect a CNI plugin issue, my first step is to confirm whether it's actually a networking problem and not an application issue. In production, the common symptoms are Pods unable to communicate with each other, DNS failures, Pods stuck in ContainerCreating, or new Pods not getting an IP address. I follow a layer-by-layer approach instead of immediately restarting the CNI.

First, I check whether the Pod has received a valid IP address using `kubectl get pods -o wide`. If the IP is missing or the Pod is stuck in ContainerCreating, I describe the Pod to look for events like **Failed to create pod sandbox**, **CNI plugin not initialized**, or **failed to assign IP**.

Next, I verify that the CNI DaemonSet is healthy on every worker node. In EKS, for example, I check the **aws-node** Pods; in Calico, I check the **calico-node** Pods. If any CNI Pod is crashing or missing on a node, networking for Pods on that node can fail. I then review the CNI logs to identify IP allocation failures, permission issues, or communication problems with the Kubernetes API.

If the CNI Pods are healthy, I test connectivity by launching a temporary Pod and verifying Pod-to-Pod communication, Service communication, and DNS resolution. If Pod-to-Pod communication fails but the application is healthy, it usually indicates a CNI routing issue. I also check whether Network Policies are blocking traffic because they can sometimes appear to be CNI issues.

Finally, I inspect the worker node itself. I verify that the node has sufficient IP addresses available, the CNI configuration files exist under `/etc/cni/net.d`, and there are no routing or iptables issues. In cloud environments like EKS, I also ensure the node hasn't exhausted its available ENI IP addresses, which is a common cause of Pods remaining in ContainerCreating.

---

## Production Troubleshooting Commands

```bash
kubectl get pods -A -o wide
kubectl describe pod <pod-name>
kubectl get ds -A
kubectl logs <cni-pod> -n kube-system
kubectl get nodes
kubectl exec -it <test-pod> -- ping <pod-ip>
kubectl exec -it <test-pod> -- nslookup kubernetes.default
kubectl exec -it <test-pod> -- curl http://<service-name>
```

---

## Common Production Issues

- CNI DaemonSet is not running on all worker nodes.
- Pods are not receiving IP addresses.
- IP address pool or ENI IPs are exhausted.
- Incorrect CNI configuration after an upgrade.
- Network Policies blocking traffic.
- Node routing or iptables corruption.
- IAM permission issues (especially in EKS with Amazon VPC CNI).

---

## Interview Closing Statement

> "In production, I first confirm whether it's truly a CNI issue by checking Pod IP allocation and connectivity. Then I verify the CNI DaemonSet, review its logs, test Pod-to-Pod and Service communication, and finally inspect node-level networking and IP allocation. This structured approach helps isolate whether the problem is the CNI itself, the node, or a Network Policy instead of making unnecessary changes."
> 
### 🔴76. CoreDNS crashes — what is the impact, and how do you debug DNS resolution?

If CoreDNS crashes, the biggest impact is on service discovery. Existing applications that already communicate using cached DNS entries may continue to work for some time, but any new DNS lookup will fail. As a result, microservices won't be able to resolve Service names like `payment-service`, causing inter-service communication failures. This can lead to application errors, API timeouts, failed database connections, and even readiness or liveness probe failures if they depend on DNS.

In production, I first confirm whether it's actually a DNS issue. I launch a temporary Pod and run `nslookup kubernetes.default` or `nslookup payment-service`. If the lookup fails, I check whether the CoreDNS Pods are running in the `kube-system` namespace. If they're crashing, I inspect their logs to identify issues such as configuration errors, API Server connectivity problems, memory exhaustion, or plugin failures.

Next, I verify that the CoreDNS Service and its Endpoints exist because even healthy CoreDNS Pods won't receive traffic if the Service is misconfigured. I also check whether the Pod's `/etc/resolv.conf` is pointing to the correct Cluster DNS IP. If DNS works using the ClusterIP but not by Service name, I investigate CoreDNS configuration and networking.

If CoreDNS is healthy but DNS still fails, I move to the networking layer. I verify kube-proxy, the CNI plugin, and any Network Policies that might be blocking traffic between application Pods and CoreDNS. In EKS, I've also seen DNS failures caused by CNI issues where Pods couldn't reach the CoreDNS Service despite the Pods themselves being healthy.

---

## Production Troubleshooting Commands

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system -l k8s-app=kube-dns
kubectl get svc -n kube-system
kubectl get endpoints -n kube-system
kubectl exec -it <test-pod> -- nslookup kubernetes.default
kubectl exec -it <test-pod> -- nslookup payment-service
kubectl exec -it <test-pod> -- cat /etc/resolv.conf
```

---

## Common Production Causes

- CoreDNS Pods are in CrashLoopBackOff.
- Incorrect CoreDNS ConfigMap.
- Insufficient CPU or memory causing frequent restarts.
- Kubernetes API Server connectivity issues.
- CNI or kube-proxy networking issues.
- Network Policies blocking access to CoreDNS.
- Incorrect DNS configuration in Pods (`resolv.conf`).
### 🔴77. A NetworkPolicy is blocking traffic — how do you confirm?

If I suspect a NetworkPolicy is blocking traffic, I first verify that the application itself is healthy. In production, I never assume the NetworkPolicy is the problem until I've confirmed that the Pods are running, the Service has healthy Endpoints, and the application is reachable when the policy is not in the path.

Next, I launch a temporary test Pod in the same namespace and try to connect to the target Pod or Service using `curl`, `nc`, or `telnet`. If the connection times out while the Pods and Services are healthy, it strongly indicates a network-level restriction. I then check whether any NetworkPolicies are applied to the namespace and whether the destination Pod is selected by one of those policies. Once a Pod is selected by a NetworkPolicy, all traffic is denied unless it's explicitly allowed.

After that, I inspect the policy carefully. I verify the `podSelector`, `namespaceSelector`, `ingress`, `egress`, and `policyTypes`. In production, the most common issues are incorrect labels, missing ingress or egress rules, or forgetting that once a Pod is selected by a NetworkPolicy, the default behavior becomes deny unless allowed.

If everything looks correct, I temporarily test by removing the policy in a non-production environment or by creating a temporary allow policy. If traffic immediately starts working, I've confirmed that the NetworkPolicy was the root cause. In production, I avoid deleting policies directly; instead, I validate the rule logic and update it through the deployment pipeline.

---

## Production Troubleshooting Commands

```bash
kubectl get networkpolicy -A

kubectl describe networkpolicy <policy-name>

kubectl get pods --show-labels

kubectl get svc

kubectl get endpoints

kubectl exec -it <test-pod> -- curl http://<service-name>

kubectl exec -it <test-pod> -- nc -zv <pod-ip> 8080
```

---

## What I Verify

- Is the destination Pod selected by a NetworkPolicy?
- Do the Pod labels match the `podSelector`?
- Are the source Pod labels or namespace labels allowed?
- Is the required Ingress rule present?
- Is the required Egress rule present? (Often overlooked.)
- Is the CNI plugin (Calico, Cilium, etc.) enforcing NetworkPolicies?

---

## Production Example

Suppose the backend should connect to the database.

```text
Backend → Database ❌ (Timeout)

Database Pod is Running
Service has healthy Endpoints
curl from another Pod fails
kubectl describe networkpolicy shows only app=api is allowed to access the database.
The backend Pod has the label app=backend.
```

In this case, the labels don't match the policy, so Kubernetes correctly blocks the traffic. Updating the NetworkPolicy or the Pod labels resolves the issue.

---

## Interview Closing Statement

> "In production, I confirm a NetworkPolicy issue by first ruling out the application, Service, and Endpoints. Then I test connectivity from a Pod, inspect which NetworkPolicies select the destination Pod, verify the label selectors and ingress/egress rules, and finally validate the fix through a controlled policy update. This approach ensures I don't misdiagnose networking issues as application failures."

### 108. If you ping one pod's IP from another and it's unreachable, where would you check?

If one Pod cannot reach another Pod's IP, I troubleshoot from the network layer inward. In production, Pod-to-Pod communication should work across nodes, so if it's failing, the issue is usually related to the CNI plugin, Network Policies, node networking, or routing.

First, I verify that both Pods are actually running and have valid IP addresses using `kubectl get pods -o wide`. Then I check whether the Pods are on the same node or different nodes. If communication works on the same node but fails across nodes, that immediately points me toward a CNI or routing issue.

Next, I check whether any Network Policies are applied to either Pod. A common production issue is that a Pod is selected by a policy that unintentionally blocks ingress or egress traffic. I inspect all policies in the namespace and validate the selectors and allowed rules.

If Network Policies are not the issue, I move to the CNI plugin. I verify that the CNI DaemonSet is healthy on all nodes, review CNI logs, and confirm that routing tables and Pod network configurations are correct. In EKS, I specifically check the aws-node Pods and verify that ENI IP allocation hasn't been exhausted.

I also verify node health and connectivity. If the Pods are on different worker nodes, I ensure the nodes can communicate with each other and that there are no security group, NACL, firewall, or route table issues blocking Pod traffic. Finally, I test communication using tools like `curl` or `nc` because many containers do not respond to ICMP ping even when the application is healthy.

---

## Production Troubleshooting Order

- Verify Pod status and IPs.
- Check whether Pods are on the same node or different nodes.
- Verify Network Policies.
- Check Service and Endpoint configuration if accessing via Service.
- Review CNI plugin health and logs.
- Verify node-to-node connectivity and routes.
- Check cloud networking (Security Groups, NACLs, Route Tables).
- Test with `curl` or `nc` instead of relying only on ping.

---

## Commands I Use

```bash
kubectl get pods -o wide

kubectl get networkpolicy -A

kubectl logs -n kube-system <cni-pod>

kubectl describe pod <pod-name>

kubectl exec -it <source-pod> -- curl http://<destination-pod-ip>:<port>

kubectl exec -it <source-pod> -- nc -zv <destination-pod-ip> <port>
```

---

## Interview Closing Statement

> "If one Pod cannot reach another Pod's IP, I first verify the Pods and their IP assignments, then check Network Policies, followed by the CNI plugin and node networking. If the Pods are on different nodes, I focus heavily on CNI routing and cloud network configuration. I also avoid relying solely on ping because many containers don't respond to ICMP; application-level tests like curl give a more accurate picture of connectivity."
### 🔴114. How does DNS resolution work inside a pod?

Inside a Pod, DNS resolution is handled by CoreDNS. Every Pod gets a `/etc/resolv.conf` file automatically created by Kubernetes, which points to the Cluster DNS Service IP (CoreDNS). When an application tries to access a Service using its name, the DNS query is sent to CoreDNS, which resolves the Service name to its ClusterIP. The application then sends traffic to that ClusterIP, and kube-proxy forwards the request to one of the healthy backend Pods.

---

## Real-Time Flow

Suppose the frontend application wants to call the payment service using:

```text
http://payment-service
```

Here's what happens:

- The application makes a DNS request for `payment-service`.
- The Pod checks its `/etc/resolv.conf`.
- The DNS request is sent to the CoreDNS Service IP.
- CoreDNS looks up the Service in the Kubernetes API and returns the ClusterIP.
- The application connects to the ClusterIP.
- kube-proxy routes the request to one of the healthy payment Pods.

The application never needs to know the actual Pod IPs, which is important because Pods are recreated frequently during deployments and autoscaling.

---

## Example `/etc/resolv.conf`

Inside every Pod, you'll typically see something like:

```text
nameserver 10.100.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

- **nameserver** → CoreDNS ClusterIP
- **search** → Allows using short Service names like `payment-service`
- **ndots:5** → Controls how DNS names are resolved within the cluster

---

## DNS Search Domains

If the frontend and payment service are in the same namespace, we simply call:

```text
http://payment-service
```

If they're in different namespaces, we use:

```text
http://payment-service.backend
```

or the fully qualified domain name (FQDN):

```text
payment-service.backend.svc.cluster.local
```

---

## Production Troubleshooting

If DNS isn't working, I verify it in this order:

- Check DNS resolution from inside the Pod.

  ```bash
  kubectl exec -it <pod> -- nslookup payment-service
  ```

- Verify the Pod's DNS configuration.

  ```bash
  kubectl exec -it <pod> -- cat /etc/resolv.conf
  ```

- Check CoreDNS Pods.

  ```bash
  kubectl get pods -n kube-system
  ```

- Review CoreDNS logs.

  ```bash
  kubectl logs -n kube-system -l k8s-app=kube-dns
  ```

- Verify the CoreDNS Service and Endpoints.

  ```bash
  kubectl get svc,endpoints -n kube-system
  ```

- If CoreDNS is healthy but resolution still fails, investigate kube-proxy, the CNI plugin, and Network Policies.

---

## Interview Closing Statement

> "Inside a Pod, DNS resolution starts with `/etc/resolv.conf`, which points to the CoreDNS Service. CoreDNS resolves the Service name to its ClusterIP by querying the Kubernetes API, and kube-proxy then forwards the traffic to a healthy backend Pod. In production, if DNS fails, I first test `nslookup` from the Pod, then verify CoreDNS health, its Service and Endpoints, and finally check networking components like kube-proxy and the CNI plugin."
### 🔴124. Suppose a Pod is running an application. How will you expose it to the internet using ALB in EKS?

In EKS, I expose a Pod to the internet using an Ingress resource together with the AWS Load Balancer Controller, which automatically provisions an Application Load Balancer (ALB). We never expose Pods directly because Pod IPs are ephemeral. Instead, we expose the application through a Service, and then the Ingress routes external traffic from the ALB to that Service.

---

## Real-Time Production Flow

Suppose I have a Pod running an application on port **8080**.

- Deploy the application Pod using a Deployment.
- Create a ClusterIP Service that exposes the Pod internally.
- Install the AWS Load Balancer Controller in the EKS cluster (if it's not already installed).
- Create an Ingress with `ingressClassName: alb` and the required ALB annotations.
- The AWS Load Balancer Controller watches the Ingress resource and automatically provisions an ALB in AWS.
- The controller creates Target Groups, registers the backend Pods (or nodes depending on the target type), configures listeners, and health checks.
- The ALB gets a public DNS name, which I map to my domain using Route 53. For HTTPS, I attach an ACM certificate through an Ingress annotation.

The traffic flow is:

```text
User
   │
   ▼
Internet
   │
   ▼
Application Load Balancer (ALB)
   │
   ▼
Ingress
   │
   ▼
ClusterIP Service
   │
   ▼
Application Pods
```

---

## Production Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:region:account:certificate/xxxxx
spec:
  ingressClassName: alb
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

---

## Production Best Practices

- Use ClusterIP as the backend Service because the ALB communicates through the Ingress Controller.
- Use IP target mode (`alb.ingress.kubernetes.io/target-type: ip`) so the ALB registers Pod IPs directly.
- Configure readiness probes so only healthy Pods are added to the ALB target group.
- Terminate SSL at the ALB using an ACM certificate.
- Use Route 53 for DNS mapping.
- Enable ALB access logs and monitor target group health and Ingress Controller logs for troubleshooting.---

## Section 5: Scheduling
### 🔴16. How does the scheduler decide where to place pods?

The Kubernetes Scheduler is responsible for assigning Pods to worker nodes. It doesn't randomly pick a node—it follows a two-phase process: **Filtering** and **Scoring**.

During **Filtering**, it eliminates nodes that cannot run the Pod based on:

- Resource availability (CPU, memory)
- Node health
- Taints and tolerations
- Node selectors
- Affinity/anti-affinity rules
- Storage requirements
- Other scheduling constraints

After identifying the eligible nodes, it **scores** them based on factors like:

- Balanced resource utilization
- Topology spread
- Affinity preferences
- Scheduling policies

The node with the highest score is selected, and the scheduler binds the Pod to that node through the Kubernetes API Server.

---

## Real-Time Production Scenario

For example, if I deploy a Pod requesting **2 vCPUs and 4 GB RAM**, and my cluster has **five worker nodes**, the scheduler first filters out nodes that don't have sufficient free resources or don't match the Pod's constraints.

Let's say only **Node-2** and **Node-4** qualify.

It then scores both nodes, considering factors such as:

- Which node has better resource balance
- Whether the Pod should be placed close to related services
- Whether Pods should be spread across availability zones

If **Node-4** receives the highest score, the scheduler binds the Pod to **Node-4**.

The kubelet running on that node then pulls the container image and starts the Pod.
### 🔴 17. What are taints and tolerations?

Taints and Tolerations are used to control where Pods can be scheduled. In production, we use them to dedicate specific worker nodes for certain workloads such as databases, monitoring, GPU workloads, or critical applications. A taint is applied to a node to repel Pods, and a toleration is added to a Pod to allow it to be scheduled on that tainted node. It's important to note that a toleration doesn't force a Pod onto a node—it only allows the scheduler to consider that node. If we want to force scheduling, we combine tolerations with node affinity or node selectors.

---

## Real-Time Production Scenario

For example, in one of our production clusters, we had dedicated nodes for Prometheus, Grafana, and Elasticsearch because they consumed significant CPU and memory. We applied a taint to those monitoring nodes:

```bash
kubectl taint nodes worker-node-3 dedicated=monitoring:NoSchedule
```

This prevents normal application Pods from being scheduled on that node.

Then, only the monitoring Pods were configured with the following toleration:

```yaml
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "monitoring"
  effect: "NoSchedule"
```

As a result, only monitoring workloads could run on those nodes, while application workloads were scheduled on the remaining worker nodes. This helped us isolate workloads and avoid resource contention.

---

## Taint Effects

There are three taint effects that we commonly use:

- **NoSchedule** – New Pods without a matching toleration are not scheduled on the node. Existing Pods continue running.
- **PreferNoSchedule** – Kubernetes tries to avoid scheduling Pods on the node, but may still schedule them if necessary.
- **NoExecute** – Existing Pods without the required toleration are evicted, and new Pods are also prevented from being scheduled.
### 🔴 18. What are node affinity and anti-affinity?

Node Affinity and Node Anti-Affinity are used to control where a Pod should or should not run based on node labels.

- **Node Affinity** tells Kubernetes "Run this Pod on nodes that match these labels."
- **Node Anti-Affinity** tells Kubernetes "Don't run this Pod on nodes with these labels."

Unlike taints and tolerations, which block or allow Pods, node affinity is used to choose the right node for a Pod.

---

## Real-Time Production Scenario

For example, in production, we had dedicated high-memory nodes for Elasticsearch.

We labeled those nodes:

```bash
kubectl label node worker-3 workload=elasticsearch
```

Then, in the Elasticsearch Deployment, we configured Node Affinity so those Pods would run only on nodes with the label `workload=elasticsearch`.

This ensures Elasticsearch doesn't get scheduled on normal application nodes.

Similarly, if we don't want test applications to run on production nodes, we use Node Anti-Affinity to prevent scheduling on nodes labeled `environment=production`.

---

## Difference Between Affinity and Taints

- **Taints & Tolerations** → Node decides who is allowed to enter.
- **Node Affinity** → Pod decides where it wants to run.
### 🔴 19. How do you assign pods using taints/tolerations and affinity?

In production, I assign Pods to specific nodes by combining taints/tolerations and node affinity.

- Taints and tolerations ensure that only authorized Pods can run on a node.
- Node affinity ensures that the Pod is scheduled on the intended node.

Using both together gives us strict workload isolation.

---

## Real-Time Production Scenario

For example, suppose I have dedicated nodes for Elasticsearch.

### Step 1: I label the node.

```bash
kubectl label node worker-3 workload=elasticsearch
```

### Step 2: I taint the node.

```bash
kubectl taint node worker-3 dedicated=elasticsearch:NoSchedule
```

Now, no normal application Pods can run on this node.

### Step 3: In the Elasticsearch Deployment, I add:

- Node Affinity to select nodes with `workload=elasticsearch`.
- Toleration to allow the Pod onto the tainted node.

```yaml
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "elasticsearch"
    effect: "NoSchedule"

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: workload
            operator: In
            values:
            - elasticsearch
```

---

## Why Use Both?

- If I use only Node Affinity, another application with the same affinity could also run on that node.

- If I use only Taints/Tolerations, the Pod is allowed to run on the node, but Kubernetes may still schedule it on another suitable node.### 20. What happens when a node goes NotReady?
### 🔴 21. Node is Ready but pods are not scheduling.

If a node is in Ready state but Pods are not getting scheduled, it means the node is healthy, but the scheduler has found some constraint that prevents scheduling. In production, I don't assume it's a node issue. I first check why the scheduler rejected the node by describing the Pod.

The first command I run is:

```bash
kubectl describe pod <pod-name>
```

The Events section usually tells the exact reason, such as **Insufficient CPU**, **Insufficient Memory**, **node affinity mismatch**, **untolerated taint**, or **volume binding failure**.

Then I verify the node's available resources using:

```bash
kubectl describe node <node-name>

kubectl top node
```

If the node has enough resources, I check whether the Pod has Node Affinity, Node Selector, or Tolerations that don't match the node. For example, I've seen production issues where a Deployment expected nodes labeled `environment=prod`, but the node label was missing, so the scheduler skipped that node.

---

## Common Production Reasons

- Insufficient CPU or Memory.
- Node has a taint but Pod has no toleration.
- Node Affinity or Node Selector mismatch.
- PVC is not bound.
- Resource Quotas or LimitRanges in the namespace.
- Maximum Pods per node has been reached.
- Scheduler events indicating scheduling constraints.

---

## Commands I Use

```bash
kubectl describe pod <pod-name>

kubectl describe node <node-name>

kubectl top node

kubectl get pvc

kubectl get nodes --show-labels

kubectl describe node <node-name> | grep Taints
```

---

## Interview Closing Statement

> "In production, whenever a Ready node isn't scheduling Pods, my first step is `kubectl describe pod` because the scheduler events usually tell the exact reason. Then I verify resources, node labels, affinity rules, taints and tolerations, and PVC status. This systematic approach helps identify the root cause quickly instead of guessing."### 90. What is pod priority and preemption?
### 🔴91. What are pod topology spread constraints?

Pod Topology Spread Constraints are used to evenly distribute Pods across failure domains, such as worker nodes or availability zones (AZs). The main goal is to improve high availability and avoid placing all replicas in the same location.

Without topology spread constraints, Kubernetes might schedule multiple replicas on the same node or in the same availability zone if resources are available. If that node or AZ fails, multiple application replicas are lost at once. Topology spread constraints prevent this by telling the scheduler to spread Pods as evenly as possible.

---

## Real-Time Production Scenario

For example, we have a payment application with **6 replicas** running in an EKS cluster spread across **3 Availability Zones**.

Without topology spread constraints:

```text
AZ-1 → 5 Pods ❌
AZ-2 → 1 Pod ❌
AZ-3 → 0 Pods ❌
```

If AZ-1 goes down, **5 out of 6 Pods** are lost.

With topology spread constraints:

```text
AZ-1 → 2 Pods ✅
AZ-2 → 2 Pods ✅
AZ-3 → 2 Pods ✅
```

Now, even if one AZ fails, the application continues serving traffic from the other two AZs.

---

## Where Do We Use It?

In production, we use topology spread constraints for:

- Critical microservices
- API servers
- Payment applications
- Authentication services
- Any application with multiple replicas that requires high availability
### 🔴 111. How do you troubleshoot if pods are not getting scheduled?

When Pods are not getting scheduled and remain in the **Pending** state, I follow a structured troubleshooting approach instead of guessing. My first step is always to identify why the Kubernetes Scheduler rejected the Pod.

The first command I run is:

```bash
kubectl describe pod <pod-name>
```

The **Events** section usually tells the exact reason, such as **Insufficient CPU**, **Insufficient Memory**, **untolerated taint**, **node affinity mismatch**, or **unbound PVC**.

Once I know the reason, I verify the cluster step by step:

- Check node resources to ensure there's enough CPU and memory.
- Verify node status and confirm the node is not cordoned (`SchedulingDisabled`).
- Check taints and tolerations to see if the Pod is blocked from the node.
- Verify node affinity or node selector to ensure the node labels match the Pod requirements.
- Check PVC status if the Pod uses persistent storage.
- Verify ResourceQuota and LimitRange in the namespace.
- Check the maximum Pods limit on the node, especially in EKS where ENI/IP limits can prevent additional Pods from being scheduled.

---

## Real-Time Production Scenario

For example, in one of our production deployments, a new version of the application remained in the **Pending** state. When I checked the Pod events, I found:

```text
0/6 nodes are available:
4 Insufficient memory.
2 node(s) had untolerated taint.
```

The application team had increased the memory request from **512Mi** to **4Gi**, so none of the existing nodes had enough free memory. We scaled the node pool using the Cluster Autoscaler, and once the new nodes joined the cluster, the Pods were scheduled automatically.

---

## Commands I Use

```bash
kubectl describe pod <pod-name>

kubectl get nodes

kubectl describe node <node-name>

kubectl top nodes

kubectl get pvc

kubectl get nodes --show-labels
```

---

## Section 6: Security
### 32. ConfigMap vs Secret.

# ConfigMap vs Secret in Kubernetes

In Kubernetes, both **ConfigMap** and **Secret** are used to externalize configuration from the application, but the key difference is the type of data they store.

## ConfigMap

A **ConfigMap** is used to store **non-sensitive configuration** such as:

- Application properties
- Environment variables
- Feature flags
- API endpoints
- Log levels
- Port numbers
- Timeout values

### Production Example
In production, we store values like:

- `APP_ENV=production`
- API URLs
- Timeout configurations
- Feature toggle values

Using a ConfigMap allows us to **modify application configuration without rebuilding the Docker image**, making deployments more flexible.

---

## Secret

A **Secret** is used to store **sensitive information** such as:

- Database passwords
- API keys
- OAuth tokens
- SSH keys
- TLS certificates

Although Kubernetes stores Secret values as **Base64-encoded data**, **Base64 is not encryption**.

### Production Best Practices

In production environments, we always:

- Enable **etcd encryption at rest**
- Restrict access using **RBAC (Role-Based Access Control)**
- Grant Secret access only to the required pods and service accounts
- Avoid storing plaintext secrets in Git repositories

---

## Key Differences

| Feature | ConfigMap | Secret |
|---------|-----------|--------|
| Purpose | Non-sensitive configuration | Sensitive data |
| Examples | Environment variables, URLs, log levels | Passwords, API keys, certificates |
| Storage | Plain text | Base64-encoded |
| Security | No special protection | Should use RBAC + etcd encryption |
| Production Usage | Application configuration | Credentials and confidential information |

---

## Interview Answer (1 Minute)

> In Kubernetes, both ConfigMap and Secret are used to externalize configuration from the application, but the main difference is the type of data they store. A ConfigMap is used for non-sensitive configuration like environment variables, API endpoints, feature flags, log levels, and port numbers. This allows us to change configuration without rebuilding the Docker image. A Secret is used for sensitive information such as database passwords, API keys, OAuth tokens, SSH keys, and TLS certificates. Although Kubernetes stores Secrets as Base64-encoded values, Base64 is not encryption. In production, we always enable etcd encryption at rest, restrict access using RBAC, and ensure only authorized workloads can access Secrets. So, ConfigMaps are for application configuration, while Secrets are for confidential data.

### 🔴33. How are Secrets stored and encrypted in etcd?


### 🔴34. Kubernetes Secrets are Base64 encoded — how do you prevent exposure in Git?
By default, Kubernetes stores **Secrets** in **etcd** as **Base64-encoded** data, which is **not encryption**—it's just encoding. Anyone with access to etcd can decode and read the Secret values.

In production environments, we always enable **Encryption at Rest** using an **EncryptionConfiguration** file on the **Kubernetes API Server**.

This ensures that before Secrets are written to etcd, they are encrypted using encryption providers such as:

- **AES-CBC (aescbc)**
- **Secretbox**
- **KMS (Key Management Service)** integrated with:
  - AWS KMS
  - Azure Key Vault
  - Google Cloud KMS

When an authorized application or user requests a Secret, the **Kubernetes API Server transparently decrypts** it before returning it. Applications never need to perform the decryption themselves.

## Production Security Best Practices

- Restrict Secret access using **RBAC** (Principle of Least Privilege).
- Encrypt communication between the **API Server** and **etcd** using **TLS**.
- Rotate Secrets and encryption keys regularly.
- Enable **Encryption at Rest** for all Secrets.
- Avoid storing sensitive values directly in Git repositories.
- Use external secret management solutions such as:
  - External Secrets Operator
  - AWS Secrets Manager
  - Azure Key Vault
  - HashiCorp Vault

## Interview Follow-up

### Is Base64 encryption?

**No.**

Base64 is only an **encoding** mechanism used to represent binary data as text. It provides **no security** because anyone can decode it. The actual security comes from:

- Encryption at Rest
- RBAC
- TLS
- External Secret Management

### 🔴35. What is RBAC? What are its components (Role, ClusterRole, RoleBinding, ClusterRoleBinding)?

**RBAC (Role-Based Access Control)** is the authorization mechanism in Kubernetes that controls **who can perform what actions on which resources**. It implements the **Principle of Least Privilege**, ensuring that users and ServiceAccounts have only the permissions required to perform their tasks.

In production environments, RBAC is one of the primary security controls used to protect the Kubernetes cluster from unauthorized access and accidental changes.

## RBAC Components

### 1. Role

A **Role** defines a set of permissions **within a single namespace**.

It specifies:
- **Resources** (Pods, Deployments, ConfigMaps, Secrets, etc.)
- **Actions (verbs)** such as `get`, `list`, `watch`, `create`, `update`, `patch`, and `delete`.

**Example:**
A developer can create, update, and view Pods only in the **development** namespace.

---

### 2. ClusterRole

A **ClusterRole** defines permissions at the **cluster level** or permissions that can be applied across multiple namespaces.

It is commonly used for:
- Cluster administrators
- Monitoring tools
- Logging agents
- Ingress Controllers
- CI/CD Service Accounts

**Example:**
A monitoring application like Prometheus requires permission to read Pods, Nodes, and Services across the entire cluster.

---

### 3. RoleBinding

A **RoleBinding** assigns a **Role** to a:
- User
- Group
- ServiceAccount

within a **specific namespace**.

**Example:**
Bind the **Developer Role** to the **dev-team** group only in the **development** namespace.

---

### 4. ClusterRoleBinding

A **ClusterRoleBinding** assigns a **ClusterRole** to a:
- User
- Group
- ServiceAccount

across the **entire Kubernetes cluster**.

**Example:**
Grant the **cluster-admin** ClusterRole to the DevOps team or assign a read-only ClusterRole to a monitoring ServiceAccount.

---

# Real-Time Production Example

Suppose a GitHub Actions pipeline deploys applications only to the **production** namespace.

Instead of giving the pipeline **cluster-admin** access, we:

- Create a **Role** with permissions to manage Deployments, Pods, Services, and ConfigMaps only in the **production** namespace.
- Create a dedicated **ServiceAccount** for the pipeline.
- Use a **RoleBinding** to bind the Role to that ServiceAccount.

This ensures the pipeline cannot modify resources in other namespaces or perform cluster-wide administrative actions.

### 🔴36. How do you provide least-privileged access to pods using RBAC?

o provide **least-privileged access** to Pods using RBAC, we first identify exactly what the application running inside the Pod needs to access. Instead of granting broad permissions, we create a **dedicated ServiceAccount** for the application, define a **Role** with only the minimum required permissions, and then associate that Role with the ServiceAccount using a **RoleBinding**.

For example, if an application only needs to read **ConfigMaps** in its own namespace, we create a **Role** that grants only the following permissions:

- `get`
- `list`
- `watch`

on the **ConfigMaps** resource.

We then create a **RoleBinding** to bind that Role to the application's **ServiceAccount**. Finally, when deploying the Pod, we specify that ServiceAccount in the Pod specification. As a result, the application can access **only the required ConfigMaps** in that namespace and nothing else.

In production, we follow these security best practices:

- Create a **dedicated ServiceAccount** for each application or workload.
- Avoid using the **default ServiceAccount** because it may unintentionally inherit permissions.
- Follow the **Principle of Least Privilege** by granting only the minimum required permissions.
- Use **namespace-scoped Roles** whenever possible.
- Grant **cluster-wide permissions** only when absolutely necessary.
- If an application requires access across multiple namespaces or cluster-level resources, use a **ClusterRole** and **ClusterRoleBinding**, while still limiting the permissions to only what the application requires.
- Regularly review and audit RBAC policies to remove unnecessary permissions.

## Real-Time Production Example

Suppose a microservice needs to read application configuration stored in **ConfigMaps** but does not need to create, update, or delete any Kubernetes resources.

In this case, we:

1. Create a dedicated **ServiceAccount** for the microservice.
2. Create a **Role** that allows only `get`, `list`, and `watch` on **ConfigMaps**.
3. Create a **RoleBinding** to bind the Role to the ServiceAccount.
4. Configure the Pod to use that ServiceAccount.

This ensures the application has only the permissions it requires, reducing the attack surface and preventing accidental or unauthorized access to other Kubernetes resources.

## Interview Closing (1 Minute)

> "In production, we never grant Pods unnecessary Kubernetes permissions. We create a dedicated ServiceAccount for each application, define a Role with only the minimum permissions required, and bind it using a RoleBinding. Namespace-scoped Roles are preferred, and cluster-wide permissions are granted only when absolutely necessary using ClusterRoles and ClusterRoleBindings. This approach follows the Principle of Least Privilege and significantly improves the security of the Kubernetes cluster."

### 🔴37. What is the difference between a Role and a ClusterRole?

The main difference between a **Role** and a **ClusterRole** is the **scope of permissions** they provide.

### Role

A **Role** provides permissions **within a single namespace**.

It is used when a user, ServiceAccount, or application needs access only to resources in a specific namespace.

For example, if a developer should manage Pods only in the **development** namespace, we create a **Role** and bind it to the user or ServiceAccount using a **RoleBinding**.

---

### ClusterRole

A **ClusterRole** provides permissions **across the entire Kubernetes cluster**.

It is used for:

- Cluster-scoped resources such as:
  - Nodes
  - PersistentVolumes
  - Namespaces
- Applications that require access across multiple namespaces
- Cluster-wide components such as monitoring, logging, and ingress controllers

A **ClusterRole** can be assigned in two ways:

- **ClusterRoleBinding** → Grants permissions across the entire cluster.
- **RoleBinding** → Reuses the ClusterRole's permissions within a single namespace.

---

# Real-Time Production Example

In one of our projects:

### Developer Access

Developers needed access only to the **development** namespace.

So we:

- Created a **Role** with permissions to manage:
  - Pods
  - Deployments
  - Services
- Bound it using a **RoleBinding**.

This ensured developers could work only in the development namespace and had no access to other environments such as QA or Production.

---

### Monitoring Access

Our **Prometheus** monitoring setup needed to:

- Discover Pods in every namespace.
- Read Services and Endpoints.
- Access Node information.

Since this required cluster-wide visibility, we:

- Created a **ClusterRole** with read-only permissions.
- Assigned it using a **ClusterRoleBinding**.

This allowed Prometheus to collect metrics across the entire Kubernetes cluster while following the principle of least privilege.

---

# Easy Way to Remember

| Role | ClusterRole |
|------|-------------|
| Namespace-level permissions | Cluster-wide permissions |
| Access only within one namespace | Access across all namespaces and cluster-scoped resources |
| Used for applications or users limited to one namespace | Used for cluster-wide components such as Prometheus, Fluentd, Ingress Controllers, etc. |
| Bound using **RoleBinding** | Usually bound using **ClusterRoleBinding** (or **RoleBinding** if reused within a single namespace) |

---

# Interview Closing (1 Minute)

> "The key difference is the scope of permissions. A Role is namespace-scoped and is used when users or applications need access only within a specific namespace. A ClusterRole is cluster-scoped and is used for cluster-wide resources or applications that need access across multiple namespaces. In production, we use Roles for application teams and namespace-specific workloads, while ClusterRoles are typically used for infrastructure components like Prometheus, logging agents, and ingress controllers. We always follow the principle of least privilege and grant only the minimum permissions required."

### 🔴38. What is a ServiceAccount, and how is it different from a user?

A **ServiceAccount** is an identity used by **applications or Pods** running inside a Kubernetes cluster, whereas a **User** represents a **human**, such as a developer, administrator, or DevOps engineer, who accesses the cluster.

### ServiceAccount

A **ServiceAccount** is designed for workloads running inside Kubernetes.

When a Pod needs to communicate with the Kubernetes API—for example, to:

- Read ConfigMaps
- Read Secrets
- Create or update Kubernetes resources
- Watch Pods or Deployments

it authenticates using a **ServiceAccount**.

Every namespace has a **default ServiceAccount**, but in production we rarely use it because it may unintentionally receive permissions.

Instead, we:

- Create a **dedicated ServiceAccount** for each application or workload.
- Grant only the required permissions using **RBAC**.
- Follow the **Principle of Least Privilege**.

This minimizes security risks and ensures workloads have access only to the resources they require.

---

### User

A **User** represents a **human identity** that accesses the Kubernetes cluster.

Examples include:

- Developers
- DevOps Engineers
- Cluster Administrators
- SREs

Unlike ServiceAccounts, **Kubernetes does not create or manage User accounts**.

Users are authenticated through an **external identity provider**, such as:

- AWS IAM
- Azure Active Directory (Microsoft Entra ID)
- LDAP
- OIDC (OpenID Connect)
- Client Certificates

After successful authentication, Kubernetes uses **RBAC** to determine what actions the user is authorized to perform.


### 🔴100. How do you implement network security in Kubernetes?

In production, network security in Kubernetes is implemented using a **defense-in-depth** approach, where multiple security controls work together instead of relying on a single mechanism.

### 1. NetworkPolicies (Pod-to-Pod Communication)

The first layer of security is **NetworkPolicies**, which control communication between Pods.

By default, Pods in Kubernetes can communicate with each other. In production, we create **NetworkPolicies** that follow a **default-deny** approach and explicitly allow only the required traffic.

This helps:

- Prevent unauthorized Pod-to-Pod communication.
- Reduce the attack surface.
- Stop lateral movement if a Pod is compromised.

---

### 2. Secure External Access

For traffic entering the cluster, we use an **Ingress Controller** with **TLS certificates**.

This ensures:

- All client communication is encrypted using **HTTPS**.
- Certificates are managed securely (for example, using **cert-manager** with Let's Encrypt or cloud-managed certificates).

---

### 3. RBAC and ServiceAccounts

We implement **RBAC** along with **dedicated ServiceAccounts** to ensure that only authorized users and applications can access Kubernetes resources.

Production best practices include:

- Creating a dedicated ServiceAccount for each application.
- Granting only the minimum required permissions.
- Avoiding the use of the default ServiceAccount.

---

### 4. Secrets Management

Sensitive information such as:

- Passwords
- API Keys
- Database Credentials
- TLS Certificates

is stored in **Kubernetes Secrets** with **Encryption at Rest** enabled.

For enhanced security, we integrate Kubernetes with external secret management solutions such as:

- AWS Secrets Manager
- Azure Key Vault
- HashiCorp Vault
- External Secrets Operator

This avoids storing sensitive credentials directly in Kubernetes or Git repositories.

---

### 5. Workload Security

At the workload level, we follow container security best practices by:

- Running containers as **non-root** users.
- Disabling **privileged containers**.
- Dropping unnecessary Linux capabilities.
- Applying **Pod Security Standards (PSS)**.
- Scanning container images for vulnerabilities before deployment using tools like **Trivy** or cloud-native image scanners.
- Using trusted and minimal base images.

---

### 6. Monitoring and Auditing

We continuously monitor the Kubernetes cluster to detect security issues and unusual activity.

This includes:

- Kubernetes Audit Logs
- Prometheus
- Grafana
- Cloud-native monitoring services (Amazon CloudWatch, Azure Monitor, etc.)
- Alerting for suspicious activities and policy violations

This enables quick detection and response to security incidents.

---

# Real-Time Production Example

In one of our production environments:

- We implemented **default-deny NetworkPolicies**, allowing communication only between frontend, backend, and database Pods.
- All external traffic entered through an **NGINX Ingress Controller** secured with **TLS**.
- Applications used **dedicated ServiceAccounts** with minimal RBAC permissions.
- Database credentials were retrieved from **AWS Secrets Manager** using **External Secrets Operator**, rather than storing them directly in Kubernetes Secrets.
- Container images were scanned using **Trivy** during the CI/CD pipeline before deployment.
- Cluster activity and application health were monitored using **Prometheus**, **Grafana**, and **CloudWatch**, with alerts configured for abnormal behavior.

---

# Interview Closing (1 Minute)

> "In production, we secure Kubernetes using multiple layers of defense. We use NetworkPolicies with a default-deny approach to control Pod communication, secure external traffic through an Ingress Controller with TLS, enforce least-privilege access using RBAC and dedicated ServiceAccounts, protect sensitive data using encrypted Kubernetes Secrets or external secret managers, harden workloads by running non-root containers and scanning images for vulnerabilities, and continuously monitor the cluster using audit logs, Prometheus, Grafana, and cloud-native monitoring services. This defense-in-depth strategy significantly strengthens the overall security posture of the Kubernetes environment."

### 🔴104. How do you secure Kubernetes clusters?

Securing a Kubernetes cluster requires implementing security at **multiple layers** rather than relying on a single control. In production, we follow a **defense-in-depth** approach where identity, workloads, networking, secrets, and monitoring all work together to protect the cluster.

---

## 1. Authentication and Authorization

The first layer of security is controlling **who can access the cluster**.

Users authenticate through an external identity provider such as:

- AWS IAM (EKS)
- Azure Entra ID (AKS)
- LDAP
- OIDC

Once authenticated, access is controlled using **RBAC (Role-Based Access Control)** based on the **Principle of Least Privilege**, ensuring users and applications receive only the permissions they require.

---

## 2. Workload Security

We secure workloads by following Kubernetes security best practices.

This includes:

- Creating **dedicated ServiceAccounts** for each application.
- Enforcing **Pod Security Standards (PSS)**.
- Running containers as **non-root users**.
- Avoiding **privileged containers**.
- Using **read-only root file systems** wherever possible.
- Dropping unnecessary Linux capabilities.
- Using trusted and minimal container base images.

These controls reduce the attack surface and limit the impact of a compromised container.

---

## 3. Secrets Management

Sensitive information such as:

- Passwords
- API Keys
- Database Credentials
- TLS Certificates

is stored using **Kubernetes Secrets** with **Encryption at Rest** enabled.

In production, we integrate Kubernetes with external secret management solutions such as:

- AWS Secrets Manager
- Azure Key Vault
- HashiCorp Vault
- External Secrets Operator

This prevents sensitive credentials from being stored in Git repositories or application manifests.

---

## 4. Network Security

We secure network communication at multiple levels.

### Internal Communication

- Implement **NetworkPolicies** with a **default-deny** approach.
- Allow only the required Pod-to-Pod communication.
- Prevent unauthorized lateral movement within the cluster.

### External Communication

- Use an **Ingress Controller** with **TLS**.
- Encrypt all client traffic using **HTTPS**.
- Restrict public exposure of services wherever possible.

---

## 5. Image Security

Before deploying applications, we scan container images for vulnerabilities using tools such as:

- Trivy
- Amazon ECR Image Scanning
- Microsoft Defender for Containers

We also:

- Use trusted base images.
- Remove unnecessary packages.
- Keep images up to date with security patches.

---

## 6. Infrastructure Security

We secure the Kubernetes control plane and worker nodes by:

- Keeping Kubernetes versions updated.
- Applying regular security patches.
- Restricting API Server access.
- Using private clusters where possible.
- Encrypting communication between cluster components using **TLS**.

---

## 7. Monitoring and Auditing

Continuous monitoring helps detect and respond to security incidents quickly.

We use:

- Kubernetes Audit Logs
- Prometheus
- Grafana
- Amazon CloudWatch / Azure Monitor
- Falco (for runtime threat detection, if required)

Alerts are configured to notify the operations team of suspicious activities, failed authentication attempts, or unauthorized access.

---

# Real-Time Production Example

In one of our production environments:

- Developers authenticated using **AWS IAM**, and RBAC limited access to their respective namespaces.
- Each microservice had a dedicated **ServiceAccount** with only the required permissions.
- **NetworkPolicies** implemented a default-deny model, allowing communication only between approved services.
- Secrets were stored in **AWS Secrets Manager** and synchronized into Kubernetes using **External Secrets Operator**.
- Container images were scanned with **Trivy** during the CI/CD pipeline before deployment.
- External traffic was secured using an **NGINX Ingress Controller** with **TLS**.
- Cluster health and security events were monitored using **Prometheus**, **Grafana**, and **CloudWatch**.

---

# Interview Closing (1 Minute)

> "In production, Kubernetes security is implemented using a defense-in-depth strategy. We secure access through IAM and RBAC, protect workloads with dedicated ServiceAccounts and Pod Security Standards, manage sensitive data using encrypted Kubernetes Secrets or external secret managers, control network traffic with NetworkPolicies and TLS-enabled Ingress, scan container images for vulnerabilities before deployment, harden the underlying infrastructure, and continuously monitor the cluster using audit logs, Prometheus, Grafana, and cloud-native monitoring tools. This layered approach provides strong protection against both external and internal security threats."

### 🔴 136. If you want to store secrets in Kubernetes, where can you store them securely?

In Kubernetes, sensitive information such as **database passwords, API keys, certificates, access tokens, and encryption keys** should be stored using **Kubernetes Secrets** instead of hardcoding them in application code, container images, or configuration files.

---

## 1. Store Sensitive Data in Kubernetes Secrets

Kubernetes **Secrets** are designed to store confidential information securely and make it available to applications when required.

Typical examples include:

- Database passwords
- API Keys
- Access Tokens
- TLS Certificates
- SSH Keys

Applications can consume Secrets as:

- Environment variables
- Mounted files inside Pods
- Volumes

This avoids exposing sensitive information in application source code.

---

## 2. Enable Encryption at Rest

By default, Kubernetes stores Secrets in **etcd** as **Base64-encoded** data, which is **not encryption**.

In production, we always enable **Encryption at Rest** using an **EncryptionConfiguration** on the Kubernetes API Server.

This ensures that Secrets are encrypted before being stored in etcd, protecting them even if someone gains access to the etcd database.

---

## 3. Use External Secret Management

For higher security, we generally avoid managing long-lived sensitive credentials directly in Kubernetes.

Instead, we integrate Kubernetes with external secret management solutions such as:

- AWS Secrets Manager
- Azure Key Vault
- HashiCorp Vault
- Google Cloud Secret Manager

Using tools like:

- **External Secrets Operator (ESO)**
- **Secrets Store CSI Driver**

Secrets are securely synchronized or mounted into Pods at runtime without embedding sensitive values in Kubernetes manifests or Git repositories.

---

## 4. Restrict Access Using RBAC

Access to Secrets is tightly controlled using **RBAC**.

Production best practices include:

- Creating dedicated **ServiceAccounts** for each application.
- Granting only the minimum required permissions.
- Allowing applications to access only the specific Secrets they need.
- Avoiding the use of the default ServiceAccount.

This follows the **Principle of Least Privilege**.

---

## 5. Additional Production Best Practices

- Never store secrets in Git repositories.
- Rotate secrets and encryption keys regularly.
- Enable **TLS** for communication between Kubernetes components.
- Audit access to Secrets using Kubernetes Audit Logs.
- Use short-lived credentials whenever possible.

---

# Real-Time Production Example

In one of our production environments running on **Amazon EKS**, application credentials were stored in **AWS Secrets Manager** instead of Kubernetes Secrets.

We used the **External Secrets Operator (ESO)** to synchronize the required secrets into Kubernetes automatically.

Each application had its own **ServiceAccount** with RBAC permissions to access only its required secrets. The synchronized Kubernetes Secrets were also protected with **Encryption at Rest**, ensuring security both inside and outside the cluster.

---

# Interview Closing (1 Minute)

> "In production, we manage sensitive information using Kubernetes Secrets, but we always enable Encryption at Rest because Secrets are only Base64-encoded by default. For stronger security, we integrate Kubernetes with external secret managers such as AWS Secrets Manager, Azure Key Vault, or HashiCorp Vault using tools like External Secrets Operator or the Secrets Store CSI Driver. Access to secrets is controlled through dedicated ServiceAccounts and RBAC, ensuring that applications can retrieve only the secrets they require while following the principle of least privilege."

### 🔴 137. Is there scope for NetworkPolicies? What is a NetworkPolicy?

Yes, **NetworkPolicies** are widely used in production Kubernetes environments to implement **micro-segmentation** and secure communication between workloads.

A **NetworkPolicy** is a Kubernetes resource that controls **which Pods are allowed to communicate with other Pods or external endpoints**. It acts like a **firewall at the Pod level** by defining:

- **Ingress** – Controls incoming traffic to a Pod.
- **Egress** – Controls outgoing traffic from a Pod.

By default, Kubernetes allows **all Pods to communicate with each other**. Once a **NetworkPolicy** is applied (and the CNI plugin supports it), only the traffic **explicitly allowed** by the policy is permitted, while all other traffic is denied.

This helps:

- Prevent unauthorized Pod-to-Pod communication.
- Reduce the attack surface.
- Stop lateral movement if a Pod is compromised.
- Enforce the **Principle of Least Privilege** at the network level.

> **Note:** NetworkPolicies are enforced only if the cluster uses a CNI plugin that supports them, such as **Calico**, **Cilium**, or **Azure CNI** (with NetworkPolicy support).

---

# Real-Time Production Example

In one of our microservices applications, we had three components:

- **Frontend**
- **Backend**
- **Database**

We implemented **NetworkPolicies** with a **default-deny** approach so that:

- The **Frontend** could communicate **only** with the **Backend**.
- The **Backend** could communicate **only** with the **Database**.
- The **Database** accepted traffic **only** from the **Backend**.
- All other Pod-to-Pod communication was blocked.

This architecture ensured that even if the Frontend Pod was compromised, it could not directly access the Database or other internal services, significantly improving the cluster's security posture.

---

# Production Best Practices

- Implement a **default-deny** NetworkPolicy and explicitly allow only required traffic.
- Define both **Ingress** and **Egress** rules where applicable.
- Apply NetworkPolicies to all production namespaces.
- Restrict access to databases, message brokers, and internal APIs.
- Verify that the Kubernetes CNI plugin supports NetworkPolicies.
- Regularly review and update policies as applications evolve.

---

# Interview Closing (1 Minute)

> "Yes, we use NetworkPolicies extensively in production to secure Pod-to-Pod communication. A NetworkPolicy acts as a firewall at the Pod level by controlling Ingress and Egress traffic. We follow a default-deny model and explicitly allow only the required communication between services. For example, in a three-tier application, the Frontend can communicate only with the Backend, the Backend only with the Database, and the Database accepts traffic only from the Backend. This micro-segmentation reduces the attack surface and prevents unauthorized lateral movement within the Kubernetes cluster."

---

## Section 7: Resource Management & Autoscaling
### 🔴 40. What is the difference between resource requests and limits?

### 🔴 41. What happens if no resource limits are defined?

### 🔴 42. What is an overcommit scenario, and what is its impact?

### 🔴 43. HPA vs VPA vs Cluster Autoscaler.

### 🔴 44. Why might HPA not scale even when CPU usage is high?

### 🔴 45. How does HPA get metrics?

### 🔴 46. What is Karpenter, and how does it differ from Cluster Autoscaler?

### 🔴 132. How to tune Cluster Autoscaler?

### 🔴 133. Pod uses less CPU but node overloaded - why and how to detect noisy neighbor?

### 🔴 134. Suppose one pod consumes more CPU/memory causing another to crash - how to prevent?

### 🔴 135. Are you aware of namespaces? Can they help achieve resource isolation?

---

## Section 8: Deployments & Release Strategies
### 🔴 47. What are the different deployment strategies (RollingUpdate, Recreate, Blue-Green, Canary), and how do you implement them?

### 🔴 48. How do you ensure zero downtime?

### 🔴 49. What deployment strategies are you following in your organization?

### 🔴 50. How do you handle a rolling update that gets stuck?

### 🔴 51. What is a canary release? How do you implement it?

### 🔴 97. A canary release is unstable — what actions will you take?

### 🔴 98. How do you handle configuration drift between environments?

### 🔴 99. Why should you avoid using the latest tag in production?

### 🔴 125. Can you write a Kubernetes Deployment YAML file for NGINX?

### 🔴 126. Why are you using a hyphen ("-") before the container name in YAML, and what does it represent?

### 🔴 127. If you want to write a Service YAML for a Deployment, what will you write?
---

## Section 9: Troubleshooting
### 🔴53. How do you troubleshoot a pod stuck in CrashLoopBackOff?

When a Pod is in **CrashLoopBackOff**, it means the container starts, crashes, and Kubernetes keeps restarting it with an increasing backoff time. In production, I don't restart the Pod immediately. My goal is to identify why the application is crashing.

The first thing I do is check the Pod events and the application logs.

```bash
kubectl describe pod <pod-name>

kubectl logs <pod-name>
```

If the container is restarting very quickly, I check the logs from the previous crashed container:

```bash
kubectl logs <pod-name> --previous
```

The logs usually reveal the root cause. Common issues I've encountered include incorrect environment variables, database connection failures, missing Secrets or ConfigMaps, invalid application configuration, image issues, or the application failing to bind to the expected port.

If the logs don't reveal the issue, I inspect the Pod configuration. I verify that the image is correct, environment variables are present, Secrets and ConfigMaps are mounted properly, resource limits are appropriate, and the application port matches the container configuration. I also check the liveness and readiness probes, because an incorrect liveness probe can continuously kill a healthy application, resulting in a CrashLoopBackOff.

If the application was working previously, I compare the current deployment with the last successful version to identify any recent changes. In production, this is often caused by a bad deployment, and if required, I perform a rollback while investigating the root cause.

---

## Common Production Causes

- Application code crashes.
- Missing or incorrect ConfigMap/Secret.
- Database or external service unavailable.
- Wrong environment variables.
- Incorrect startup command or entrypoint.
- Liveness probe misconfiguration.
- Out of Memory (OOMKilled).
- Wrong container image or application port.

---

## Commands I Use

```bash
kubectl describe pod <pod-name>

kubectl logs <pod-name>

kubectl logs <pod-name> --previous

kubectl get events --sort-by=.metadata.creationTimestamp

kubectl describe deployment <deployment-name>
```

---

## Real-Time Production Scenario

For example, during one deployment, all application Pods entered **CrashLoopBackOff** immediately after release. The logs showed:

```text
Error: DATABASE_CONNECTION_STRING not found
```

The new deployment referenced a Secret that hadn't been created in the production namespace. After creating the missing Secret and restarting the Deployment, the Pods started successfully.

### 54. How do you troubleshoot a Pod stuck in ImagePullBackOff?

When a Pod is in **ImagePullBackOff**, it means Kubernetes is unable to pull the container image from the registry. In production, I first identify why the image pull failed instead of recreating the Pod.

The first thing I do is check the Pod events because Kubernetes clearly reports the image pull failure.

```bash
kubectl describe pod <pod-name>
```

The **Events** section usually tells the exact reason, such as **image not found**, **authentication failed**, **access denied**, or **network timeout**.

Next, I verify that the image name and tag in the Deployment are correct. A typo in the image name or an incorrect tag is one of the most common production issues. If the image is stored in a private registry like Azure Container Registry (ACR) or Amazon ECR, I verify that the cluster has permission to pull the image. This includes checking `imagePullSecrets` or the node's IAM/managed identity permissions.

I also confirm that the image actually exists in the registry and that the worker nodes have network connectivity to the registry. If the image was recently pushed, I verify that the CI/CD pipeline completed successfully and pushed the correct version.

---

## Common Production Causes

- Incorrect image name or tag.
- Image does not exist in the registry.
- Authentication failure to a private registry.
- Missing or incorrect `imagePullSecrets`.
- IAM or Managed Identity permission issues (EKS/AKS).
- Registry is unavailable or network connectivity issues.
- Image was never pushed by the CI/CD pipeline.

---

## Commands I Use

```bash
kubectl describe pod <pod-name>

kubectl get pod <pod-name> -o yaml

kubectl describe secret <image-pull-secret>

kubectl describe deployment <deployment-name>
```

---

## Real-Time Production Scenario

For example, during one production deployment, all Pods went into **ImagePullBackOff**. When I checked the Pod events, I saw:

```text
Failed to pull image "myapp:v2.5"
Error response from registry: manifest unknown
```

The CI/CD pipeline had pushed the image as **v2.4**, but the Deployment YAML referenced **v2.5**. After updating the Deployment with the correct image tag and redeploying, the Pods started successfully.

In another case, the image existed, but the `imagePullSecret` had expired. After updating the registry credentials, Kubernetes was able to pull the image successfully.
### 🔴 55. What causes a pod to be evicted?

A Pod is evicted when the kubelet removes it from a node because the node is under resource pressure or due to a scheduling policy. In production, the most common reason is resource exhaustion, especially memory pressure, where Kubernetes evicts lower-priority Pods to keep the node healthy.

The first thing I do is check the Pod status and events to identify the eviction reason.

```bash
kubectl describe pod <pod-name>
```

The **Events** section usually shows messages like **"The node was low on memory"** or **"Pod was evicted due to ephemeral-storage pressure."**

The most common causes I've seen in production are:

- **Memory Pressure** – Node runs out of RAM, so kubelet evicts Pods.
- **Disk Pressure** – Node runs out of disk or ephemeral storage.
- **PID Pressure** – Too many processes are running on the node.
- **Node Drain** – During maintenance or cluster upgrades, Pods are evicted gracefully.
- **NoExecute Taint** – If a node gets a `NoExecute` taint, Pods without a matching toleration are evicted.
- **Node becomes NotReady/Unreachable** – Kubernetes eventually evicts Pods and reschedules them on healthy nodes.

---

## Real-Time Production Scenario

For example, one of our worker nodes started consuming excessive memory because of multiple Java applications. The node entered **MemoryPressure**, and Kubernetes evicted some low-priority Pods to protect the node. We identified the issue using:

```bash
kubectl describe node <node-name>

kubectl top node
```

We found memory utilization was above **95%**. We added another worker node through the Cluster Autoscaler, redistributed the workload, and the evictions stopped.

---

## Commands I Use

```bash
kubectl describe pod <pod-name>

kubectl describe node <node-name>

kubectl top node

kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## Common Production Causes

- Memory pressure (most common).
- Disk/ephemeral storage pressure.
- PID pressure.
- Node drain during maintenance.
- NoExecute taints.
- Node failure or prolonged NotReady state.
### 🔴 56. How do you troubleshoot OOMKilled pods?

An **OOMKilled** Pod means the container exceeded its configured memory limit, so the Linux OOM (Out of Memory) Killer terminated the process. In production, this is usually caused by either an application memory leak, insufficient memory limits, or incorrect resource sizing. My goal is to determine whether the issue is with the application or the Kubernetes resource configuration.

The first thing I do is confirm that the Pod was actually OOMKilled.

```bash
kubectl describe pod <pod-name>

kubectl get pod <pod-name> -o wide
```

In the Pod description, I check the **Last State**, which will show:

```text
Reason: OOMKilled
Exit Code: 137
```

Next, I review the application logs to understand what the application was doing before it was terminated.

```bash
kubectl logs <pod-name> --previous
```

Then, I verify the Pod's resource requests and limits. If the memory limit is too low compared to the application's actual usage, Kubernetes will kill the container when it crosses that limit.

I also check the node's memory utilization to ensure the issue isn't node-level memory pressure.

```bash
kubectl top pod

kubectl top node
```

If the application consistently exceeds its memory limit, I work with the development team to determine whether it's an application memory leak or simply a workload increase. If it's expected memory usage, I increase the Pod's memory limits and requests. If it's a memory leak, the application needs to be fixed rather than just allocating more memory.

---

## Common Production Causes

- Memory limit is too low.
- Application memory leak.
- Sudden traffic spike causing higher memory usage.
- Large batch jobs or file processing.
- JVM/Node.js/.NET memory not configured correctly.
- Incorrect resource requests and limits.

---

## Commands I Use

```bash
kubectl describe pod <pod-name>

kubectl logs <pod-name> --previous

kubectl top pod

kubectl top node

kubectl describe deployment <deployment-name>
```

---

## Real-Time Production Scenario

For example, one of our Java microservices started restarting continuously during peak traffic. The Pod status showed:

```text
Reason: OOMKilled
Exit Code: 137
```

The Deployment had a **512Mi** memory limit, but during peak load the application was consuming around **900Mi**. We confirmed this using monitoring dashboards and `kubectl top pod`. As an immediate fix, we increased the memory limit to **1Gi** and adjusted the JVM heap settings. Later, the development team optimized the application to reduce memory consumption. After that, the OOMKilled events stopped.
### 🔴57. What is the difference between `kubectl logs` and `kubectl describe`?

I use `kubectl logs` and `kubectl describe` together because they provide different information.

- `kubectl logs` shows the application logs generated by the container. I use it to troubleshoot application-level issues such as exceptions, database connection failures, API errors, or startup failures.
- `kubectl describe` shows the Kubernetes object's details and events. It provides information about scheduling, image pulling, probes, resource allocation, Pod conditions, volume mounts, and the Events section, which often explains why Kubernetes is unable to run the Pod.

---

## Real-Time Production Scenario

For example, suppose a Pod is in **CrashLoopBackOff**.

My first command is:

```bash
kubectl describe pod <pod-name>
```

This tells me whether the Pod failed due to:

- ImagePullBackOff
- Failed Scheduling
- OOMKilled
- Failed Mount
- Probe failures
- Event messages from Kubernetes

If the Pod has started and the container is crashing, I then use:

```bash
kubectl logs <pod-name>

kubectl logs <pod-name> --previous
```

to check the application logs and identify the exact error, such as:

- Database connection failed
- NullPointerException
- Configuration file missing

---

## Simple Difference

| `kubectl describe` | `kubectl logs` |
|--------------------|----------------|
| Kubernetes information | Application information |
| Pod Events | Container logs |
| Scheduling issues | Application errors |
| Image pull failures | Stack traces |
| Probe failures | Runtime exceptions |
| Volume mount issues | Startup logs |

---

## Interview Closing Statement

> "kubectl describe tells me why Kubernetes is having trouble with the Pod, while kubectl logs tells me why the application inside the Pod is failing. In production, I always start with `kubectl describe` to understand the Pod's state and events, then use `kubectl logs` to identify the application-level root cause."
### 🔴 58. A pod is running but the application is not accessible — walk through the troubleshooting steps.

If the Pod is in **Running** state but the application is not accessible, I don't assume the application is healthy because **Running** only means the container is running, not that the application inside it is working. In production, I troubleshoot layer by layer—from the application to the network.

First, I verify whether the application is actually listening on the expected port. I check the application logs for startup errors or exceptions.

```bash
kubectl logs <pod-name>

kubectl exec -it <pod-name> -- netstat -tulnp
```

If the application is running correctly, I verify the Service. I check whether the Service selector matches the Pod labels and whether the Service has healthy Endpoints.

```bash
kubectl get svc

kubectl get endpoints
```

If the Endpoints are empty, it's usually a label mismatch between the Service and the Pods.

Next, I test the application from inside the cluster.

```bash
kubectl exec -it <test-pod> -- curl http://<service-name>:<port>
```

- If this fails, the issue is between the Pod and the Service.
- If this works, the problem is likely in the Ingress or Load Balancer.

Then I verify the Ingress configuration, the Ingress Controller logs, and ensure the backend Service and ports are correctly configured. If I'm running on EKS or AKS, I also check the Load Balancer health checks, security groups/firewall rules, and DNS configuration.

Finally, I verify that Network Policies aren't blocking traffic and confirm that the application's readiness probe is passing. If the readiness probe fails, the Pod may be **Running**, but Kubernetes won't send traffic to it.

---

## Production Troubleshooting Flow

```text
Pod Running
      │
      ▼
Check Application Logs
      │
      ▼
Verify Application Listening Port
      │
      ▼
Check Service & Endpoints
      │
      ▼
Test Service Internally (curl)
      │
      ▼
Check Ingress / Load Balancer
      │
      ▼
Verify Network Policies & Firewall
```

---

## Commands I Use

```bash
kubectl logs <pod-name>

kubectl exec -it <pod-name> -- netstat -tulnp

kubectl get svc

kubectl get endpoints

kubectl exec -it <test-pod> -- curl http://<service-name>:<port>

kubectl describe ingress <ingress-name>
```

---

## Real-Time Production Scenario

For example, one of our APIs was in the **Running** state, but users were getting **503 Service Unavailable**. The application logs were clean, but `kubectl get endpoints` returned no endpoints. We found that the Service selector was `app=backend`, while the Deployment label had been changed to `app=backend-v2` during the last release. Because the labels didn't match, the Service had no backend Pods. After correcting the selector, the Endpoints were populated, and the application became accessible immediately.
### 59. Pods are running but the Service is unreachable — what are the possible causes?

### 🔴60. A pod is stuck in the Pending state — how do you debug it?

If a Pod is stuck in the **Pending** state, it means the Pod has been created, but the Kubernetes Scheduler hasn't been able to assign it to a worker node. In production, I don't start by checking the nodes—I first check why the scheduler couldn't place the Pod.

The first command I run is:

```bash
kubectl describe pod <pod-name>
```

The **Events** section usually gives the exact reason, such as **Insufficient CPU**, **Insufficient Memory**, **untolerated taints**, **node affinity mismatch**, or **unbound PVC**.

Based on the scheduler event, I troubleshoot further. I verify that the worker nodes have enough CPU and memory, check if the node is cordoned (`SchedulingDisabled`), validate node labels and affinity rules, inspect taints and tolerations, and ensure any required PersistentVolumeClaim is in the **Bound** state.

I also verify namespace-level restrictions like **ResourceQuota** or **LimitRange**, because they can prevent Pods from being scheduled even if the cluster has free resources.

---

## Production Troubleshooting Steps

### Check Pod events.

```bash
kubectl describe pod <pod-name>
```

### Verify node health and available resources.

```bash
kubectl get nodes

kubectl top nodes

kubectl describe node <node-name>
```

### Check node labels and affinity.

```bash
kubectl get nodes --show-labels
```

### Check taints.

```bash
kubectl describe node <node-name> | grep Taints
```

### Verify PVC status.

```bash
kubectl get pvc
```

### Check namespace quotas.

```bash
kubectl describe resourcequota

kubectl describe limitrange
```

---

## Common Production Causes

- Insufficient CPU or Memory.
- Node Affinity/Node Selector mismatch.
- Untolerated taints.
- PVC is not bound.
- Node is cordoned (`SchedulingDisabled`).
- Maximum Pods per node reached.
- ResourceQuota or LimitRange restrictions.
### 🔴61. If `kubectl describe pod` says "node is out of capacity," what will you do?

If `kubectl describe pod` shows that the node is out of capacity, it means the scheduler couldn't find a worker node with enough available resources to run the Pod. In production, I first identify which resource is exhausted—CPU, memory, ephemeral storage, or the maximum Pod limit.

The first thing I do is check the scheduler events and then verify the node's resource utilization.

```bash
kubectl describe pod <pod-name>

kubectl top nodes

kubectl describe node <node-name>
```

If it's CPU or memory, I identify whether existing workloads are consuming most of the resources. If the cluster has Cluster Autoscaler enabled, I verify that it's scaling the node pool. If not, I manually add more worker nodes or increase the node size.

If the issue is the maximum Pods per node (common in EKS because of ENI/IP limits), I either scale out by adding more nodes or choose a larger instance type that supports more Pods.

I also check whether the Pod's resource requests are set too high. Sometimes developers accidentally request **4 CPUs** and **8Gi** memory for a lightweight application, making it impossible for the scheduler to place the Pod. In that case, I work with the application team to right-size the resource requests.

---

## Commands I Use

```bash
kubectl describe pod <pod-name>

kubectl top nodes

kubectl describe node <node-name>

kubectl get nodes
```

---

## Real-Time Production Scenario

For example, during a production deployment, a new version of a Java application remained in **Pending**. The scheduler event showed:

```text
0/6 nodes are available: Insufficient memory.
```

When I checked the Deployment, I found that the memory request had been increased from **1Gi** to **6Gi**. None of our worker nodes had that much free memory. Since the application genuinely needed more memory, we scaled the node pool using the Cluster Autoscaler, and once the new nodes joined the cluster, the Pods were scheduled successfully.
### 🔴 62. A node has 8 GB RAM free, and you request 2 GB for a pod, but it remains in the Pending state with "Insufficient memory." Why?

Just because a node appears to have **8 GB free** doesn't mean Kubernetes can use all **8 GB** for Pods. The scheduler makes its decision based on the node's **Allocatable** memory, not the total or visible free memory.

The first thing I would check is the node's allocatable memory:

```bash
kubectl describe node <node-name>
```

In production, there are several reasons this can happen:

- Kubernetes Reserved Memory – Some memory is reserved for the OS, kubelet, and system processes, so it's not available for Pods.
- Existing Pod Requests – The scheduler considers resource requests, not actual usage. Even if Pods are currently using less memory, if they've already requested most of the allocatable memory, the scheduler won't place another Pod.
- Resource Fragmentation – The cluster may have free memory spread across multiple nodes, but no single node has enough allocatable memory for the Pod.
- ResourceQuota or LimitRange restrictions.
- The node may be under MemoryPressure, causing the scheduler to avoid it.

---

## Real-Time Production Example

For example, suppose a worker node has **16 GB RAM**.

- **2 GB** reserved for the OS and Kubernetes components.
- **Allocatable memory = 14 GB.**

Now imagine existing Pods have already requested **13 GB**, even though they're currently using only **6 GB**.

If I deploy a Pod requesting **2 GB**, Kubernetes rejects it because:

```text
13 GB (already requested)
+2 GB (new Pod request)
-----------------------
15 GB > 14 GB allocatable
```

Even though monitoring shows around **8 GB** is currently free, the scheduler only looks at **requested resources**, not live memory usage.

---

## Commands I Use

```bash
kubectl describe node <node-name>

kubectl top node

kubectl describe pod <pod-name>
```

---

## Interview Closing Statement

> "In production, the scheduler doesn't use free memory shown by monitoring tools. It schedules Pods based on the node's allocatable memory and existing resource requests. So even if a node appears to have 8 GB free, the Pod can still remain Pending if the allocatable memory has already been reserved by other Pods. My first step is always to check `kubectl describe node` and compare allocatable memory with the total requested resources."

---

## Interview Tip (This impresses interviewers)

> **"Kubernetes schedules based on requests, not actual resource usage."**
### 🔴 63. How do you check resource usage in Kubernetes?

In production, when I need to check resource usage, I look at both actual resource consumption and configured resource requests/limits. I don't rely on a single command because each provides different information.

First, I check the current CPU and memory usage of nodes and Pods using the Metrics Server.

```bash
kubectl top nodes

kubectl top pods

kubectl top pods -A
```

This tells me which nodes or Pods are consuming the most CPU and memory.

Next, I inspect the Pod configuration to verify its resource requests and limits.

```bash
kubectl describe pod <pod-name>
```

This shows the CPU and memory requests and limits configured for the container.

If I want to check the overall capacity and allocatable resources of a node, I use:

```bash
kubectl describe node <node-name>
```

This shows:

- Total Capacity
- Allocatable resources
- Currently allocated resources
- Node conditions such as MemoryPressure or DiskPressure

If I suspect namespace restrictions, I also verify:

```bash
kubectl describe resourcequota

kubectl describe limitrange
```

---

## Commands I Use

```bash
kubectl top nodes

kubectl top pods

kubectl top pods -A

kubectl describe pod <pod-name>

kubectl describe node <node-name>

kubectl describe resourcequota

kubectl describe limitrange
```

---

## Real-Time Production Scenario

For example, one of our Java applications started responding slowly during peak traffic. I first ran:

```bash
kubectl top pods -A
```

I found one Pod consuming **95%** of its memory limit. Then I checked:

```bash
kubectl describe pod <pod-name>
```

and noticed the memory limit was configured too low. We increased the memory limit and monitored the application. The performance issue was resolved without any further Pod restarts.

---

## Interview Closing Statement

> "In production, I primarily use `kubectl top` to monitor live CPU and memory usage, `kubectl describe pod` to verify resource requests and limits, and `kubectl describe node` to check node capacity and allocatable resources. Using these commands together helps me quickly identify whether the issue is resource exhaustion, incorrect sizing, or node capacity."
### 🔴 64. How do you debug if a pod is scheduled on the wrong node?

If a Pod is scheduled on the wrong node, I first verify why the scheduler selected that node. In production, this is usually caused by incorrect or missing node affinity, node selector, taints/tolerations, or node labels.

First, I identify which node the Pod is running on.

```bash
kubectl get pod <pod-name> -o wide
```

Then I inspect the Pod configuration.

```bash
kubectl describe pod <pod-name>
```

I check whether the Pod has:

- nodeSelector
- Node Affinity
- Tolerations

Next, I verify the labels on the node.

```bash
kubectl get nodes --show-labels
```

A common production issue is that the Deployment expects a label like:

```yaml
nodeSelector:
  environment: production
```

but the node is actually labeled:

```yaml
env=production
```

Since the labels don't match, Kubernetes ignores that node selection rule or schedules the Pod on another eligible node.

I also check whether the node has any taints and whether the Pod has matching tolerations.

```bash
kubectl describe node <node-name>
```

If everything looks correct, I review the Deployment YAML and compare it with the previous working version to identify any recent configuration changes.

---

## Commands I Use

```bash
kubectl get pod <pod-name> -o wide

kubectl describe pod <pod-name>

kubectl get nodes --show-labels

kubectl describe node <node-name>

kubectl describe deployment <deployment-name>
```

---

## Real-Time Production Scenario

For example, we had dedicated GPU nodes for ML workloads. A new ML Pod was scheduled on a normal worker node instead of the GPU node. When I checked the Deployment, I found that the node affinity expected:

```yaml
gpu=true
```

but the node label had been changed to:

```yaml
accelerator=nvidia
```

Because of this label mismatch, the scheduler couldn't identify the correct node. After updating the affinity rule and redeploying, the Pod was scheduled on the GPU node.

---

## Interview Closing Statement

> "In production, if a Pod is running on the wrong node, I first verify the node where it's running, then check the Pod's nodeSelector, node affinity, and tolerations. After that, I validate the node labels and taints. Most issues are caused by label mismatches or incorrect scheduling rules rather than Kubernetes itself."

---

## Interview Tip

> If the interviewer says **"the Pod is already running on the wrong node"**, remember that Kubernetes does **not** automatically move running Pods when labels or affinity change. You typically fix the scheduling rule and recreate or roll out the Pod so the scheduler can place it on the correct node.
### 🔴 64. How do you debug if a pod is scheduled on the wrong node?

If a Pod is scheduled on the wrong node, I first verify why the scheduler selected that node. In production, this is usually caused by incorrect or missing node affinity, node selector, taints/tolerations, or node labels.

First, I identify which node the Pod is running on.

```bash
kubectl get pod <pod-name> -o wide
```

Then I inspect the Pod configuration.

```bash
kubectl describe pod <pod-name>
```

I check whether the Pod has:

- nodeSelector
- Node Affinity
- Tolerations

Next, I verify the labels on the node.

```bash
kubectl get nodes --show-labels
```

A common production issue is that the Deployment expects a label like:

```yaml
nodeSelector:
  environment: production
```

but the node is actually labeled:

```yaml
env=production
```

Since the labels don't match, Kubernetes ignores that node selection rule or schedules the Pod on another eligible node.

I also check whether the node has any taints and whether the Pod has matching tolerations.

```bash
kubectl describe node <node-name>
```

If everything looks correct, I review the Deployment YAML and compare it with the previous working version to identify any recent configuration changes.

---

## Commands I Use

```bash
kubectl get pod <pod-name> -o wide

kubectl describe pod <pod-name>

kubectl get nodes --show-labels

kubectl describe node <node-name>

kubectl describe deployment <deployment-name>
```

---

## Real-Time Production Scenario

For example, we had dedicated GPU nodes for ML workloads. A new ML Pod was scheduled on a normal worker node instead of the GPU node. When I checked the Deployment, I found that the node affinity expected:

```yaml
gpu=true
```

but the node label had been changed to:

```yaml
accelerator=nvidia
```

Because of this label mismatch, the scheduler couldn't identify the correct node. After updating the affinity rule and redeploying, the Pod was scheduled on the GPU node.

---

## Interview Closing Statement

> "In production, if a Pod is running on the wrong node, I first verify the node where it's running, then check the Pod's nodeSelector, node affinity, and tolerations. After that, I validate the node labels and taints. Most issues are caused by label mismatches or incorrect scheduling rules rather than Kubernetes itself."

---

## Interview Tip

> If the interviewer says **"the Pod is already running on the wrong node"**, remember that Kubernetes does **not** automatically move running Pods when labels or affinity change. You typically fix the scheduling rule and recreate or roll out the Pod so the scheduler can place it on the correct node.
###  🔴66. A pod is stuck in the Terminating state — how do you safely force delete it?

If a Pod is stuck in the **Terminating** state, it usually means Kubernetes is trying to gracefully delete the Pod, but something is preventing it from completing. In production, I don't force delete it immediately because that can lead to data corruption or incomplete cleanup, especially for stateful applications.

My first step is to identify why the Pod is stuck.

```bash
kubectl describe pod <pod-name>
```

I check for:

- Finalizers preventing deletion.
- Attached Persistent Volumes.
- Application taking too long to shut down.
- Node becoming NotReady.
- Container runtime or kubelet issues.

If the Pod is part of a StatefulSet or is writing data, I first ensure there are no active transactions or writes before deleting it.

If everything looks safe and the Pod is no longer serving traffic, I force delete it using:

```bash
kubectl delete pod <pod-name> --grace-period=0 --force
```

If the Pod still remains in the API, I check whether the node is healthy. Sometimes the node itself is **NotReady**, and the kubelet cannot complete the deletion. In that case, I troubleshoot the node or remove it from the cluster if required.

---

## Commands I Use

```bash
kubectl describe pod <pod-name>

kubectl get pod <pod-name> -o yaml

kubectl get node

kubectl delete pod <pod-name> --grace-period=0 --force
```

---

## Real-Time Production Scenario

For example, during a node maintenance activity, one application Pod remained in the **Terminating** state for over **20 minutes**. When I checked, the worker node had become **NotReady**, so the kubelet couldn't complete the Pod cleanup.

Since the application was already running on another replica and there was no impact on users, I safely force deleted the Pod from the API server and then fixed the unhealthy node.

---

## Interview Closing Statement

> "In production, I never force delete a Pod as the first option. I first identify why it's stuck by checking the Pod, node, and any finalizers or storage dependencies. If it's safe to remove and the application is already available through other replicas, I use `kubectl delete pod --grace-period=0 --force`. This ensures I don't risk data loss or interrupt active workloads."

---

## Interview Tip

> If the interviewer asks **"When should you NOT force delete a Pod?"**, answer:
>
> **"I avoid force deleting Pods that belong to StatefulSets or are actively writing to a database unless I've confirmed that all data has been flushed or replicated. Force deletion should always be the last resort."**
### 🔴67. A node is in the NotReady state — what are the possible reasons and their impact?

**NotReady** node means the control plane has detected that the worker node is unhealthy or is no longer able to run workloads properly. In production, this is a high-priority issue because Kubernetes stops scheduling new Pods on that node, and if the problem persists, the Pods running on that node are eventually evicted and recreated on healthy nodes.

The first thing I do is identify why the node became NotReady.

```bash
kubectl get nodes

kubectl describe node <node-name>
```

The **Conditions** section usually tells the reason.

---

## Common Causes (Production Scenarios)

- **Kubelet is down** – The kubelet service has stopped or crashed, so the node stops reporting its health to the control plane.  
- **Node lost network connectivity** – The worker node cannot communicate with the API Server.  
- **High CPU or Memory usage** – The node is under heavy resource pressure, making kubelet unresponsive.  
- **DiskPressure** – The node has run out of disk or ephemeral storage.  
- **MemoryPressure** – The node is critically low on memory.  
- **Container runtime failure** – Docker or containerd has stopped working.  
- **VM or cloud infrastructure issue** – The underlying VM is rebooting, terminated, or has a hardware issue.  

---

## Impact

- No new Pods are scheduled on the node.  
- Existing Pods continue running initially.  
- If the node doesn't recover within the eviction timeout (typically around **5 minutes**), Kubernetes marks the Pods as unhealthy and reschedules them onto healthy nodes (if managed by a Deployment or ReplicaSet).  
- If it's a StatefulSet, recovery may take longer depending on storage attachment and application behavior.  

---

## Commands I Use

```bash
kubectl get nodes

kubectl describe node <node-name>

kubectl get pods -o wide

kubectl top node
```

If I have access to the node, I also check:

```bash
systemctl status kubelet

systemctl status containerd

journalctl -u kubelet
```

---

## Real-Time Production Scenario

For example, during a production deployment, one worker node suddenly became **NotReady**. When I checked the node, I found the kubelet service had stopped after the VM ran out of disk space. Since the node couldn't communicate with the API Server, Kubernetes marked it as NotReady and stopped scheduling new Pods.

We cleaned up disk space, restarted the kubelet service, and the node returned to the **Ready** state. Because our application had multiple replicas, the service remained available throughout the incident.
### 🔴68. What are DiskPressure, MemoryPressure, and PIDPressure node conditions? What happens when they occur?

These are node conditions in Kubernetes that indicate resource exhaustion at the node level. They are continuously monitored by the kubelet, and when any of these conditions are triggered, Kubernetes takes corrective actions like rejecting new Pods or evicting existing Pods to protect node stability.

---

## 🔹 MemoryPressure

This occurs when the node is running low on available RAM.

In production, this usually happens due to:

- High memory-consuming Pods  
- Memory leaks in applications  
- JVM/Node.js processes exceeding limits  

**Impact:**

- Node is marked under pressure  
- New Pods are not scheduled on the node  
- Kubelet starts evicting lower priority Pods to free memory  

---

## 🔹 DiskPressure

This happens when the node runs low on:

- Root filesystem space  
- Container logs  
- Ephemeral storage (`/var/lib/containerd`, `/var/log`)  

**Impact:**

- Node is marked unschedulable for new Pods  
- Pods using high ephemeral storage are evicted first  
- Image pulls and container writes may start failing  

In production, this is very common when logs are not rotated properly.

---

## 🔹 PIDPressure

This occurs when the node runs out of available process IDs (PIDs).

It usually happens when:

- Too many Pods or containers are running  
- A process fork-bomb or uncontrolled thread creation occurs  

**Impact:**

- Node cannot create new processes  
- New Pods are not scheduled  
- Existing workloads may become unstable  

---

## ⚙️ What Kubernetes does when these conditions occur

When any of these pressures are detected:

- Node is marked as **NotReady** or **under pressure**  
- Scheduler stops placing new Pods on that node  
- Kubelet starts evicting Pods based on priority and QoS:
  - BestEffort Pods first  
  - Burstable next  
  - Guaranteed Pods last  

If pressure continues, the node becomes unstable and workloads are moved elsewhere.

---

## 📌 Real-Time Production Scenario

For example, in one production cluster:

- A logging agent started consuming high disk space due to missing log rotation  
- Node entered **DiskPressure**  
- Kubernetes started evicting application Pods  
- Scheduler stopped placing new Pods on that node  

We fixed it by:

- Cleaning up disk space  
- Enabling log rotation  
- Increasing ephemeral storage on the node group  

---

## 🧪 Commands I Use

```bash
kubectl describe node <node-name>

kubectl get nodes

kubectl top node

kubectl get pods -A -o wide
```

If I have node access:

```bash
df -h

ps aux --sort=-%mem

ps -eLf | wc -l
```

### 🔴69. How do you cordon and drain a node safely? What happens to DaemonSet pods during a drain?

In production, when I need to perform maintenance on a node (like patching, upgrading, or replacing a VM), I first cordon the node and then drain it safely to ensure no new workloads are scheduled and existing workloads are gracefully moved.

---

## 🔹 Step 1: Cordon the node

Cordon marks the node as **unschedulable**, meaning Kubernetes will not place any new Pods on it.

```bash
kubectl cordon <node-name>
```

👉 Existing Pods are not affected, only new scheduling is blocked.

---

## 🔹 Step 2: Drain the node

Drain safely evicts all running Pods so they can be rescheduled on other healthy nodes.

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

### What happens during drain:

- Evicts Pods gracefully (respects termination grace period)  
- Reschedules Pods (if managed by Deployment/ReplicaSet)  
- Waits for Pod termination before proceeding  
- Respects Pod Disruption Budgets (PDBs), if defined  

---

## 🔹 What happens to DaemonSet Pods?

This is a key interview point.

👉 DaemonSet Pods are **NOT evicted by default during drain**.

Because:

- DaemonSets are meant to run one Pod per node  
- Examples: CNI plugin, logging agent, monitoring agent  

### So during drain:

- `kubectl drain` automatically skips DaemonSet Pods  
- That’s why we use `--ignore-daemonsets`  

If needed, we manage DaemonSet Pods separately, but in most production cases we do not remove them during node maintenance.

---

## Interview Closing Statement

> "In production, I first cordon the node to stop new workloads and then drain it to safely evict existing Pods. Kubernetes respects Pod Disruption Budgets and gracefully reschedules Pods on other nodes. DaemonSet Pods are not evicted during drain by default because they are meant to run on every node, so we use `--ignore-daemonsets` to allow the drain operation to proceed."
### 🔴70. A node's CPU utilization is at 100% — how do you identify the root cause?

### 🔴71. Pods are being evicted — why does this happen, and how does the kubelet eviction policy work?

When Pods are being evicted in production, it means the node is under resource pressure and kubelet is actively reclaiming resources to protect node stability. Eviction is not a failure—it is a controlled safety mechanism used by Kubernetes.

The first thing I do is identify the eviction reason from the Pod or node events.

```bash
kubectl describe pod <pod-name>

kubectl describe node <node-name>
```

---

## 🔹 Why Pods get evicted (Production reasons)

Pods are usually evicted due to node-level resource pressure, such as:

- **MemoryPressure (most common)** – Node is running out of RAM  
- **DiskPressure** – Ephemeral storage or root filesystem is full  
- **PIDPressure** – Too many processes running on the node  
- **Node NotReady / instability**  
- **Kubelet eviction threshold breach**  

---

## 🔹 How kubelet eviction policy works (important concept)

Kubelet continuously monitors node resources and compares them against eviction thresholds.

There are two types:

---

### 1. Hard Eviction Thresholds

When crossed, kubelet immediately starts eviction.

Examples:

- Memory critically low  
- Disk space almost exhausted  

👉 Immediate Pod eviction starts

---

### 2. Soft Eviction Thresholds

Kubelet tries to recover first (grace period).

If pressure continues → eviction starts after the grace period.

---

## 🔹 Eviction priority (VERY IMPORTANT in interviews)

When eviction starts, Kubernetes follows this order:

### 1. BestEffort Pods (first to be evicted)
- No requests/limits defined  

### 2. Burstable Pods
- Have some requests but can burst  

### 3. Guaranteed Pods (last to be evicted)
- Strict requests = limits (highest protection)  

---

## 🔹 Key Production Insight

👉 Production-critical workloads are usually protected using **Guaranteed QoS**, so they are least likely to be evicted during resource pressure.
### 🔴 73. A NodePort Service is not accessible — what could be the possible reasons?
### 🔴 78. How do you capture network traffic inside a pod?
### 🔴 79. What happens if the Kubernetes API Server goes down?



---

## Section 10: EKS
### 🔴 84. How do you upgrade an EKS cluster with zero downtime?
### 🔴 85. What pre-checks do you perform before an EKS upgrade?
### 🔴 86. If the EKS control plane becomes unavailable during an upgrade, how do pods continue to communicate?
### 🔴 109. How is Amazon EKS different from a self-managed Kubernetes cluster?
### 🔴 110. How do you manage node groups in Amazon EKS?
### 🔴 121. What are the different components available in Amazon EKS, and what is their purpose?
### 🔴 122. Which CIDR range did you use for your Amazon EKS cluster?
### 🔴 123. How many node groups do you have, and can a single node group contain multiple instance types?
### 🔴 138. Have you provisioned Amazon EKS clusters for large-scale workloads?
### 🔴 142. How do you design a multi-region Kubernetes architecture?

---

## Section 11: Monitoring & Observability
### 🔴 101. How do you monitor Kubernetes applications and debug memory leaks?### 120. How do you debug high kubelet CPU usage?
### 🔴 139. How do you implement centralized logging and monitoring for Kubernetes?
### 🔴 140. How do you reduce the blast radius in a large Kubernetes cluster?
### 🔴 141. How do you design a multi-tenant Kubernetes cluster with isolation?

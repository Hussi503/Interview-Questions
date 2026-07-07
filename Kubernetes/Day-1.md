### 🔴 1. Kubernetes Architecture & Core Components

  Kubernetes follows a Control Plane and Worker Node architecture. 
  
  The Control Plane is responsible for managing the cluster, while Worker Nodes run the actual application workloads. 
  
  The core Control Plane components are API Server,ETCD, Scheduler, and Controller Manager.
  
  Worker Nodes contain Kubelet, Kube Proxy, and Container Runtime. 
  
  Kubernetes continuously compares the desired state with the actual state and automatically takes corrective actions to     maintain application availability."
 
####  Why Do We Need It?

   Centralized cluster management

   Automated scheduling

   Self-healing

   Auto-scaling

   High availability


In our EKS environment, the Control Plane manages scheduling, scaling, and cluster operations, while application workloads run on worker nodes spread across multiple Availability Zones


We had pods stuck in Pending state. After investigating Scheduler events, we identified that worker nodes lacked sufficient CPU resources. After scaling the node group, pods were scheduled successfully.

---

####  Troubleshooting

```bash
kubectl get nodes

kubectl cluster-info

kubectl get componentstatus

kubectl describe pod <pod-name>
```

Check:

```text
Node status
Scheduler events
Cluster health
Control plane communication
```

---

#### Strong Closing Line

**"Kubernetes architecture is built around desired state management, enabling automated deployment, scaling, recovery, and high availability of applications."**

---

### 🔴 2. What happens internally when you run kubectl apply?

**"When we run kubectl apply, the manifest is sent to the Kubernetes API Server. The API Server validates the request and stores the desired state in ETCD. The Controller Manager continuously monitors this desired state, and if pods need to be created, the Scheduler selects an appropriate node. The Kubelet on that node then communicates with the container runtime to create and start the containers."**

---

####  Why Do We Need It?

   Declarative deployments

   Automated reconciliation

   Configuration management

   Infrastructure consistency

---

#### Production Usage

**"Our CI/CD pipelines use kubectl apply to deploy Deployments, Services, ConfigMaps, and Ingress resources into EKS clusters."**


####  Troubleshooting

```bash
kubectl get events

kubectl describe deployment <deployment>

kubectl describe pod <pod>

kubectl rollout status deployment <deployment>
```

Check:

```text
Manifest validation
Scheduling failures
Image issues
Resource shortages
```

---

####  Strong Closing Line

**"kubectl apply updates the desired state in Kubernetes, and the control plane works continuously to make the actual state match it."**

---

### 🔴 3. What is a Static Pod?


**"A Static Pod is a pod that runs directly on a node using a local manifest file rather than being created through the normal Kubernetes deployment process. 

Its primary purpose is to run critical system components that need to be available even if the Kubernetes API Server is down. 

In kubeadm clusters, control plane components like kube-apiserver, etcd, scheduler, and controller-manager are deployed as Static Pods. 

The Kubelet monitors the local manifest file and ensures the pod remains running.

---
### Why Do We Need It?

    Critical services must start before Kubernetes is fully available

    Local node-level management

    Control plane dependency reduction

---

####  Production Usage

Static Pods are commonly used for:

```text
kube-apiserver
etcd
kube-scheduler
kube-controller-manager
```

---

###  Real-Time Scenario

**"In kubeadm-based clusters, I verified control plane health by checking Static Pod manifests located under `/etc/kubernetes/manifests`."**

---

###  Troubleshooting

```bash
ls /etc/kubernetes/manifests

journalctl -u kubelet

crictl ps

crictl logs <container-id>
```

Check:

```text
Manifest syntax
Kubelet status
Container runtime
Node resources
```

---

###  Strong Closing Line

**"Static Pods are mainly used for critical Kubernetes control-plane components that must remain available independently of the API Server."**

---

### 🔴 4. What are Kubernetes Operators? Have you used them?

**"Kubernetes Operators extend Kubernetes functionality by automating the lifecycle management of complex applications. They use Custom Resource Definitions (CRDs) and custom controllers to automate deployment, upgrades, backups, failover, and scaling activities. Operators essentially bring operational knowledge into Kubernetes."**

---

###  Why Do We Need It?

   Automate repetitive operational tasks

   Reduce manual intervention

   Simplify lifecycle management

   Enable self-healing for complex applications

---

####  Production Usage

Examples:

```text
Prometheus Operator
ElasticSearch Operator
MongoDB Operator
Kafka Operator
```

---

####  Real-Time Scenario

**"We used the Prometheus Operator to deploy and manage monitoring infrastructure. It automated upgrades, ServiceMonitor creation, and configuration management without manual changes."**

---

####  Troubleshooting

```bash
kubectl get crds

kubectl get pods

kubectl logs <operator-pod>

kubectl describe <custom-resource>
```

Check:

```text
Operator health
CRD availability
Controller logs
Custom resource status
```

---

####  Strong Closing Line

**"Operators automate complex operational procedures and make Kubernetes capable of managing sophisticated stateful applications efficiently."**

---

### 🔴 5. What are Admission Controllers? How have you used them?

**"Admission Controllers are Kubernetes components that intercept API requests after authentication and authorization but before objects are stored in ETCD. They can validate, modify, or reject requests based on organizational policies and security requirements."**

---

####  Why Do We Need It?

   Governance

   Security

   Compliance

   Policy enforcement

---

####  Production Usage

Examples:

```text
Enforce resource limits
Prevent privileged containers
Require labels and annotations
Enforce non-root execution
Restrict hostPath volumes
```

---

#### Real-Time Scenario

**"We used Kyverno Admission Policies to block containers running as root and to ensure every deployment had CPU and memory limits defined before being admitted into the cluster."**

---

#### Troubleshooting

```bash
kubectl describe pod

kubectl get events

kubectl logs <admission-controller-pod>
```

Look for:

```text
Policy violations
Rejected requests
Mutation failures
Validation errors
```

---

####  Strong Closing Line

**"Admission Controllers act as a gatekeeper for the Kubernetes API and are critical for enforcing security, governance, and compliance policies in production environments."**

### 🔴 6. How does Kubernetes help with reliability?

**"Kubernetes improves application reliability through self-healing, replication, health checks, rolling updates, auto-scaling, and automatic workload rescheduling. If a pod crashes, Kubernetes recreates it automatically. If a node fails, workloads are rescheduled to healthy nodes. This ensures applications remain available with minimal manual intervention."**

---

### 🔴 Why Do We Need It?

🔴 High Availability

🔴 Fault Tolerance

🔴 Self-Healing

🔴 Automatic Recovery

🔴 Reduced Downtime

---

#### 🔴 Production Usage

**"In production, our applications run with multiple replicas across worker nodes in different Availability Zones. Kubernetes automatically handles pod failures and node failures without impacting users."**

---

#### 🔴 Real-Time Scenario

**"One of our worker nodes became unhealthy during peak traffic. Kubernetes automatically evicted pods from the failed node and scheduled them onto healthy nodes, preventing service disruption."**

---

#### 🔴 Troubleshooting

```bash
kubectl get pods

kubectl get nodes

kubectl describe pod <pod-name>

kubectl get events
```

Check:

```text
Failed nodes
Pod restarts
Health check failures
Scheduling issues
```

---

#### 🔴 Strong Closing Line

**"Kubernetes provides reliability by continuously maintaining the desired state and automatically recovering from failures."**

---

### 🔴 7. How does Kubernetes manage a large number of Docker containers?

**"Kubernetes manages large numbers of containers through abstractions such as Pods, Deployments, ReplicaSets, Services, and Controllers. Instead of managing containers individually, we define the desired state, and Kubernetes automatically handles scheduling, scaling, networking, monitoring, and recovery."**

---

### 🔴 Why Do We Need It?

🔴 Container Orchestration

🔴 Centralized Management

🔴 Automated Scaling

🔴 Simplified Operations

🔴 High Availability

---

#### 🔴 Production Usage

**"Our EKS clusters manage hundreds of application pods across multiple worker nodes. Kubernetes automatically distributes workloads and maintains the required replica count."**

---

### 🔴 Real-Time Scenario

**"During a product launch, traffic increased significantly. HPA automatically increased pod replicas while Cluster Autoscaler added more worker nodes to handle the load."**

---

#### 🔴 Troubleshooting

```bash
kubectl get deployments

kubectl get rs

kubectl get pods

kubectl top pods

kubectl top nodes
```

Check:

```text
Resource utilization
Scaling issues
Node capacity
Pod distribution
```

---

#### 🔴 Strong Closing Line

**"Kubernetes allows us to manage thousands of containers efficiently by automating deployment, scaling, recovery, and resource management."**

---

### 🔴 8. What is the difference between a Pod and a Container?

**"A Container is the actual running application process packaged with its dependencies. A Pod is the smallest deployable unit in Kubernetes that contains one or more containers. Containers inside the same pod share networking, storage, and can communicate through localhost."**

---

#### 🔴 Why Do We Need Pods?

🔴 Shared Networking

🔴 Shared Storage

🔴 Sidecar Pattern

🔴 Kubernetes Scheduling Unit

---

#### 🔴 Production Usage

**"Most of our microservices run one container per pod. For logging and monitoring use cases, we deploy sidecar containers within the same pod."**

---

#### 🔴 Real-Time Scenario

**"We used a Fluent Bit sidecar container alongside application containers to collect and forward logs to Elasticsearch."**

---

#### 🔴 Quick Comparison

```text
Container → Runs Application

Pod → Wraps one or more containers
      Provides networking and storage
      Smallest deployable unit in Kubernetes
```

---

#### 🔴 Troubleshooting

```bash
kubectl describe pod <pod-name>

kubectl logs <pod-name>

kubectl get pods -o wide
```

---

#### 🔴 Strong Closing Line

**"Containers run applications, while Pods provide the Kubernetes abstraction that manages and schedules those containers."**

---

### 🔴 9. What is the difference between Pod, Deployment, ReplicaSet, StatefulSet, and DaemonSet?

#### 🔴 Pod

**"Smallest deployable unit in Kubernetes that runs one or more containers."**

#### 🔴 ReplicaSet

**"Ensures a specified number of pod replicas are always running."**

#### 🔴 Deployment

**"Manages ReplicaSets and provides rolling updates, rollbacks, and version control."**

#### 🔴 StatefulSet

**"Used for stateful applications requiring stable identities and persistent storage."**

#### 🔴 DaemonSet

**"Ensures one pod runs on every eligible node."**

---

#### 🔴 Why Do We Need Them?

```text
Pod         → Run Containers

ReplicaSet  → Maintain Pod Count

Deployment  → Manage Application Lifecycle

StatefulSet → Manage Stateful Workloads

DaemonSet   → Node-Level Services
```

---

#### 🔴 Production Usage

```text
Deployment  → Java APIs, Microservices

StatefulSet → MongoDB, Kafka, Cassandra

DaemonSet   → Fluent Bit, Datadog Agent, Node Exporter
```

---

#### 🔴 Real-Time Scenario

**"Our microservices use Deployments, MongoDB runs as StatefulSets, and Fluent Bit runs as a DaemonSet on every worker node for centralized logging."**

---

#### 🔴 Troubleshooting

```bash
kubectl get deploy

kubectl get rs

kubectl get statefulset

kubectl get daemonset

kubectl describe deployment <name>
```

---

#### 🔴 Strong Closing Line

**"Each Kubernetes controller serves a specific workload type, and choosing the correct controller is essential for stability and maintainability."**

---

### 🔴 10. Can you attach a volume to a Deployment? How is it different from a StatefulSet?

**"Yes, a Deployment can use Persistent Volumes through PVCs. However, Deployment pods are generally interchangeable and do not have stable identities. StatefulSets provide stable pod identities and dedicated persistent storage for each pod, making them suitable for databases and other stateful applications."**

---

#### 🔴 Why Do We Need StatefulSets?

🔴 Stable Network Identity

🔴 Dedicated Storage

🔴 Ordered Startup & Shutdown

🔴 Data Persistence

---

#### 🔴 Production Usage

```text
Deployment → Stateless Applications

StatefulSet → Databases
             Kafka
             Elasticsearch
```

---

#### 🔴 Real-Time Scenario

**"We used Deployments for Spring Boot microservices because pods were stateless. For MongoDB, we used StatefulSets because each database pod required its own persistent volume and stable hostname."**

---

#### 🔴 Quick Comparison

```text
Deployment

✅ Stateless
✅ Shared PVC possible
✅ Pods interchangeable

StatefulSet

✅ Stateful
✅ Dedicated PVC per pod
✅ Stable pod identity
✅ Ordered deployment and deletion
```

---

#### 🔴 Troubleshooting

```bash
kubectl get pvc

kubectl get pv

kubectl describe statefulset <name>

kubectl describe pod <pod-name>
```

Check:

```text
Volume attachment issues
PVC binding failures
Storage class configuration
Node affinity issues
```

---

#### 🔴 Strong Closing Line

**"While Deployments can use persistent storage, StatefulSets are specifically designed for applications that require stable identities, persistent data, and predictable pod management."**
### 🔴 11. What could cause a StatefulSet pod to fail when rescheduled to a different AZ?


**"The most common reason is storage affinity. In cloud environments like AWS, EBS volumes are tied to a specific Availability Zone. If a StatefulSet pod gets scheduled to a node in a different AZ, Kubernetes may not be able to attach the existing volume, causing the pod to remain in Pending state."**

---

#### 🔴 Why Do We Need to Understand This?

🔴 Stateful applications depend on persistent storage

🔴 Storage location affects pod scheduling

🔴 Incorrect storage design can impact availability

---

#### 🔴 Production Usage

```text
MongoDB
PostgreSQL
MySQL
Kafka
Elasticsearch
```

All these workloads store persistent data and commonly run as StatefulSets.

---

#### 🔴 Real-Time Scenario

**"In EKS, one of our worker nodes became unavailable. Kubernetes attempted to schedule a MongoDB pod on another node in a different AZ. Since the EBS volume was attached to the original AZ, the pod remained pending until scheduling aligned with the volume's zone."**

---

#### 🔴 Troubleshooting

```bash
kubectl describe pod <pod-name>

kubectl get pvc

kubectl get pv

kubectl describe pv <pv-name>
```

Look for:

```text
Volume node affinity conflict
Volume attachment failure
AZ mismatch
PVC binding issues
```

---

#### 🔴 Strong Closing Line

**"For StatefulSets, storage design is critical because persistent volumes may have zone-specific limitations that directly impact pod scheduling."**

---

### 🔴 12. If a DaemonSet pod is pending, how would you troubleshoot?

**"If a DaemonSet pod is stuck in Pending state, I start by checking pod events and node conditions. Common causes include insufficient node resources, node selectors, taints and tolerations mismatch, image pull failures, or node affinity restrictions."**

---

#### 🔴 Why Do We Need This Troubleshooting?

🔴 DaemonSets are usually used for cluster-level services

🔴 Logging and monitoring agents must run on every node

🔴 Missing DaemonSet pods can impact observability

---

#### 🔴 Production Usage

```text
Fluent Bit
Datadog Agent
Node Exporter
CloudWatch Agent
```

---

#### 🔴 Real-Time Scenario

**"A Fluent Bit DaemonSet was not scheduled on newly added worker nodes. Investigation showed that node taints existed, but the DaemonSet lacked the required tolerations."**

---

#### 🔴 Troubleshooting

Check pod details:

```bash
kubectl describe pod <pod-name>
```

Check node status:

```bash
kubectl get nodes

kubectl describe node <node-name>
```

Check events:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Verify:

```text
Node resources
Taints
Tolerations
Node selectors
Affinity rules
Image pull errors
```

---

#### 🔴 Strong Closing Line

**"Most DaemonSet scheduling issues are caused by node constraints, taints, tolerations, or insufficient resources."**

---

### 🔴 13. Why would a DaemonSet create two pods per node?

**"Under normal circumstances, a DaemonSet should run only one pod per node. If two pods are observed on a node, it is usually due to a rolling update, duplicate DaemonSets, overlapping selectors, or a terminating old pod while a new pod is being created."**

---

#### 🔴 Why Do We Need to Understand This?

🔴 Helps identify deployment issues

🔴 Prevents duplicate logging or monitoring

🔴 Avoids unnecessary resource consumption

---

#### 🔴 Production Usage

Commonly seen with:

```text
Fluent Bit upgrades
Datadog upgrades
Security agent upgrades
```

---

#### 🔴 Real-Time Scenario

**"While upgrading Fluent Bit, a new pod was created before the old pod was completely terminated. For a short period, two pods existed on the same node."**

---

#### 🔴 Troubleshooting

Check DaemonSet configuration:

```bash
kubectl get daemonset

kubectl describe daemonset <daemonset-name>
```

Check pods:

```bash
kubectl get pods -o wide
```

Verify:

```text
Rolling update strategy
Duplicate DaemonSets
Pod termination delays
Selector conflicts
```

---

#### 🔴 Strong Closing Line

**"Two DaemonSet pods on a node are usually temporary during upgrades or indicate a configuration problem that needs investigation."**

---

### 🔴 14. What is the difference between a Job and a CronJob?

**"A Job is used to run a task once until completion, whereas a CronJob is used to run Jobs on a recurring schedule similar to Linux cron. Jobs are typically used for one-time activities, while CronJobs automate repeated operational tasks."**

---

#### 🔴 Why Do We Need Them?

#### Job

🔴 One-time execution

🔴 Batch processing

🔴 Data migration

#### CronJob

🔴 Scheduled execution

🔴 Periodic maintenance

🔴 Automated operations

---

#### 🔴 Production Usage

#### Job

```text
Database Migration
Schema Updates
One-Time Scripts
```

#### CronJob

```text
Daily Backups
Log Cleanup
Report Generation
Cache Cleanup
```

---

#### 🔴 Real-Time Scenario

**"We used a Job during a release to migrate database schemas and a CronJob to perform nightly backups of production databases."**

---

#### 🔴 Troubleshooting

Check Job status:

```bash
kubectl get jobs

kubectl describe job <job-name>
```

Check CronJob:

```bash
kubectl get cronjobs

kubectl describe cronjob <cronjob-name>
```

Check logs:

```bash
kubectl logs <pod-name>
```

Verify:

```text
Schedule configuration
Pod failures
Retry limits
Application errors
```

---

#### 🔴 Strong Closing Line

**"Jobs perform one-time tasks, while CronJobs automate recurring operations based on a defined schedule."**

---

### 🔴 15. How do PV and PVC behave across zones?

**"PVC is a request for storage, while PV is the actual storage resource. Their behavior across Availability Zones depends on the underlying storage technology. Some storage systems are zone-bound, while others support multi-zone access."**

---

#### 🔴 Why Do We Need to Understand This?

🔴 Impacts workload scheduling

🔴 Affects disaster recovery

🔴 Determines storage architecture

---

#### 🔴 Production Usage

#### EBS

```text
Single Availability Zone
Used for databases
```

#### EFS

```text
Multi-AZ
Shared storage
```

---

#### 🔴 Real-Time Scenario

**"In EKS, StatefulSet workloads used EBS-backed PVCs, which required scheduling within the same AZ. Shared application data was stored on EFS because it could be accessed from multiple Availability Zones."**

---

#### 🔴 Troubleshooting

Check PVC status:

```bash
kubectl get pvc

kubectl describe pvc <pvc-name>
```

Check PV details:

```bash
kubectl get pv

kubectl describe pv <pv-name>
```

Verify:

```text
Storage Class
Volume Binding Mode
Node Affinity
Availability Zone
Volume Attachments
```

---

#### 🔴 Strong Closing Line

**"PVCs provide storage abstraction, but cross-zone behavior depends on the underlying storage platform. Understanding storage architecture is essential for designing highly available Kubernetes workloads."**

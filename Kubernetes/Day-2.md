### 🔴1. What is the difference between a PV and PVC?
In Kubernetes, a **Persistent Volume (PV)** is the actual storage resource available in the cluster,

whereas a **Persistent Volume Claim (PVC)** is a request made by an application to consume that storage.

In real-time environments, infrastructure or platform teams usually create and manage PVs, which can be backed by EBS, Azure Disk, NFS, NetApp, or any storage system.

Application teams don't directly consume PVs; instead, they create PVCs by specifying requirements such as storage size, access mode, and storage class.

Kubernetes then automatically binds the PVC to a matching PV.
### 🔴2. What is the difference between a StorageClass and a PV?

A **Persistent Volume (PV)** is the actual storage resource that gets attached to a pod, 

whereas a **StorageClass** is a template or blueprint that defines how that storage should be created. 

In production, StorageClass sits one level above PV and automates the provisioning process.

For example, in AWS, a StorageClass can define whether the storage should be gp3, io2, encrypted, replication settings, reclaim policy, and other parameters.

When an application creates a PVC, Kubernetes checks the StorageClass associated with that PVC and dynamically provisions a PV based on the StorageClass configuration.

The newly created PV is then bound to the PVC automatically.


### 🔴3. How do you handle database migrations safely in Kubernetes?
In production, I never perform database migrations as part of the application container startup because if multiple pod replicas start simultaneously, the migration can run multiple times and cause schema conflicts or application downtime.

Instead, I execute migrations as a separate Kubernetes Job or as part of the CI/CD pipeline before deploying the new application version.

The approach I typically follow is: 

first take a database backup or ensure a rollback strategy exists, then run the migration job, validate that it completes successfully, and only after that proceed with the application deployment.

For critical systems, I prefer backward-compatible migrations, such as adding new columns or tables first, deploying the application, and removing old columns in a later release. This avoids breaking running services during rolling updates.


### 🔴4. What happens if etcd is corrupted? How do you recover?
### 🔴5. How do you backup and restore etcd?
### 🔴6. What is a headless service? how does it work?
### 🔴 7. What are the different Service types?
### 🔴8. What is the difference between Ingress and LoadBalancer?
### 🔴 9. How does Ingress work? How do you create an ingress controller in AWS EKS?
### 🔴10. How will you manage SSL certificates for ALB in EKS?
### 🔴11. What type of Load Balancer is created when using Ingress in EKS?
### 🔴12. Where will you mention the Load Balancer type in Ingress YAML?
### 🔴13. How does kube-proxy work?
### 🔴14. How does CoreDNS provide service discovery in Kubernetes?
### 🔴 15. Explain the Kubernetes networking model.

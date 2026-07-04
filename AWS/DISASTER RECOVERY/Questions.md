# 1. What is Disaster Recovery? What are RTO and RPO?

## Answer

In production, Disaster Recovery (DR) is the strategy we design to recover an application, infrastructure, and data after a major failure such as an AWS Regional outage, accidental deletion, ransomware attack, database corruption, or infrastructure failure. The objective is not simply to restore servers but to restore business operations within the agreed Service Level Agreement (SLA). Every DR design starts with understanding business requirements because different applications have different recovery expectations.

The two most important metrics are **RTO (Recovery Time Objective)** and **RPO (Recovery Point Objective)**.

**RTO** defines the maximum acceptable downtime. It answers the question: **"How quickly should the application be available after a disaster?"**

**RPO** defines the maximum acceptable data loss. It answers: **"How much recent data can the business afford to lose?"**

For example, in one of our production environments supporting customer transactions, the business required an **RTO of 30 minutes** and an **RPO of less than 5 minutes**. To achieve this, we maintained infrastructure in a secondary region using Terraform, enabled Cross-Region Replication for S3, configured cross-region database replication, and automated application deployment through GitHub Actions. During our quarterly DR drill, we successfully recovered the application within the defined SLA.

The key point is that DR is driven by business requirements, and AWS services are selected based on those recovery objectives—not the other way around.

---

## Production Example

Our application served thousands of users daily. Since downtime directly impacted revenue, we invested in a Warm Standby DR architecture instead of relying only on backups. Although it increased infrastructure costs slightly, it significantly reduced recovery time and met the business SLA.

---

## Best Practice

Always define RTO and RPO with business stakeholders before designing the DR solution. Technology should support business requirements—not define them.

---

# 2. How will you plan Disaster Recovery?

## Answer

In production, I approach Disaster Recovery as a structured process rather than simply enabling backups. The first step is to perform a Business Impact Analysis (BIA), where I identify all critical applications and determine the acceptable RTO and RPO for each workload. Customer-facing applications usually require much lower recovery times than internal tools.

Once business objectives are clear, I identify every dependency involved in the application. This includes networking, IAM roles, security groups, databases, S3 buckets, Kubernetes clusters, secrets, DNS, certificates, monitoring, and CI/CD pipelines. DR planning is incomplete if even one dependency is overlooked.

Infrastructure is fully managed through Terraform so that the entire environment can be recreated consistently in another AWS region. Databases are protected using Multi-AZ for high availability and cross-region replication or snapshots for disaster recovery. S3 buckets are configured with Versioning, Cross-Region Replication, and AWS Backup. Route 53 handles DNS failover, while GitHub Actions automatically deploys applications into the DR environment.

Finally, we document detailed recovery runbooks and conduct quarterly DR drills to validate actual RTO and RPO. If recovery exceeds the agreed SLA, we improve automation and update the runbooks.

---

## Production Example

During one DR exercise, our recovery took nearly 45 minutes instead of the expected 30 minutes because a Terraform module didn't recreate Route 53 health checks. We fixed the module immediately and validated it in the next DR drill. This is why regular testing is as important as planning.

---

## Best Practice

A Disaster Recovery plan that is never tested is only documentation. Real confidence comes from successful recovery drills.

---

# 3. Explain different DR strategies – Backup & Restore vs Warm Standby.

## Answer

AWS recommends multiple Disaster Recovery strategies depending on business requirements and budget. The two most common approaches are **Backup & Restore** and **Warm Standby**.

**Backup & Restore** is the most cost-effective option. In this model, only backups are maintained in another region. During a disaster, Terraform recreates the infrastructure, databases are restored from snapshots, applications are deployed through CI/CD pipelines, and DNS is updated. Since everything is created after the failure occurs, both RTO and RPO are relatively high. This strategy is suitable for internal applications where downtime is acceptable.

**Warm Standby** maintains a fully functional but scaled-down version of the application in the DR region. Core infrastructure such as networking, EKS clusters, databases, and application services are already running with minimal resources. During a disaster, Auto Scaling rapidly increases capacity, Route 53 redirects traffic, and the application becomes fully operational within minutes.

In production, we selected Warm Standby for customer-facing applications because the business required an RTO of under 30 minutes. Although the infrastructure cost was slightly higher than Backup & Restore, it significantly reduced downtime and ensured better customer experience.

---

## Production Example

Our DR region continuously ran a small EKS cluster with minimal worker nodes. During failover, Cluster Autoscaler provisioned additional nodes automatically, and GitHub Actions deployed the latest application version. Traffic switched using Route 53 without requiring manual intervention.

---

## Best Practice

Choose the DR strategy based on business impact—not infrastructure cost. Downtime often costs much more than maintaining a warm standby environment.

---

# 4. How do you design DR for EC2, databases, and EKS workloads?

## Answer

In production, I design Disaster Recovery at the application level instead of treating EC2, databases, and Kubernetes separately because recovering only one component doesn't restore the complete service.

For **EC2**, infrastructure is managed entirely through Terraform. Auto Scaling Groups recreate instances automatically using prebuilt AMIs, and application artifacts are stored in S3 or ECR. AWS Backup copies AMIs and EBS snapshots to the DR region to enable rapid recovery.

For **databases**, I use Multi-AZ for high availability within the same region and cross-region replication or scheduled snapshots for disaster recovery. The replication strategy depends on the business RPO and the database engine being used.

For **EKS**, the cluster itself is recreated using Terraform. Kubernetes manifests, Helm charts, and configuration files are stored in Git repositories. Container images are replicated to Amazon ECR in the DR region. Persistent volumes are protected using Velero or AWS Backup, while secrets are managed through AWS Secrets Manager or External Secrets.

During a disaster, Terraform recreates the infrastructure, GitHub Actions deploys workloads automatically, databases are promoted in the DR region, Route 53 updates DNS, and monitoring verifies application health before production traffic is redirected.

---

## Production Example

We performed quarterly DR drills where the entire EKS environment was recreated from Terraform in another AWS region. Application deployment completed through GitHub Actions, databases were promoted, Route 53 redirected traffic, and users experienced only a few minutes of interruption.

---

## Best Practice

Never rely on manual recovery. Every infrastructure component, deployment process, and failover step should be fully automated.

---

# 5. How will you set up DR strategy for S3 regional outages?

## Answer

For production workloads, I start by enabling **S3 Versioning** because it protects against accidental deletion and object overwrites. Next, I configure **Cross-Region Replication (CRR)** so that all new objects are automatically replicated to a secondary AWS region. If the bucket uses SSE-KMS encryption, I also configure the appropriate KMS permissions for replication.

Applications should never directly reference regional bucket names. Instead, I expose S3 through **CloudFront** or **S3 Multi-Region Access Points**, which provide a single endpoint and automatically direct requests to the healthy region.

To handle regional failures, **Route 53** monitors application health and performs DNS failover. Even though CRR replicates data, I still configure **AWS Backup** because replication alone cannot recover deleted or corrupted objects.

Finally, we regularly simulate regional outages by temporarily redirecting traffic to the replicated bucket and verifying application functionality, data consistency, monitoring, and recovery time.

---

## Production Example

One of our applications stored customer invoices in S3. We configured Versioning, Cross-Region Replication, AWS Backup, and CloudFront. During a DR exercise, we blocked access to the primary region and successfully served all files from the replicated bucket without requiring any application code changes.

---

## Best Practice

For production applications, Versioning + Cross-Region Replication + AWS Backup should be treated as complementary controls rather than alternatives.

---

# 6. What RPO and RTO would you define for S3 DR?

## Answer

RPO and RTO for S3 should always be defined based on business requirements rather than technical capabilities.

For most customer-facing production applications storing documents, invoices, user uploads, or application assets, I would typically target an **RPO of less than 15 minutes**. This is achieved through S3 Cross-Region Replication, which continuously replicates new objects to another AWS region.

For **RTO**, I generally aim for **15 to 30 minutes**, where Route 53 or S3 Multi-Region Access Points redirect user requests to the replicated bucket. If applications use CloudFront, failover can be even faster because users continue accessing the same distribution endpoint.

For highly regulated industries such as banking or healthcare, the business may require an RPO close to zero. In such cases, additional application-level replication mechanisms and multi-region active-active architectures may be necessary because asynchronous replication alone may not satisfy the requirement.

The important point is that RTO and RPO are business decisions. AWS provides multiple services to achieve them, but the recovery objectives should always be agreed upon before the architecture is designed.

---

## Production Example

In one production environment, customer-uploaded documents were business-critical. We implemented Versioning, Cross-Region Replication, AWS Backup, CloudFront, and Route 53. During quarterly DR drills, the application successfully met an RPO of under 5 minutes and an RTO of approximately 20 minutes, satisfying the business SLA.

---

## Best Practice

Avoid promising unrealistic recovery objectives. Every reduction in RTO or RPO increases architectural complexity and operational cost, so the solution should always align with business priorities.
7. How does Cross-Region Replication help in DR?
8. What is S3 Multi-Region Access Point and why is it used?
9. How does Route 53 help in S3 DR?
10. Why is AWS Backup required even if CRR is enabled?
11. How do you test DR readiness in AWS?
12. Why is single-region S3 not recommended for production?
13. How do you enforce DR policies using Terraform?
14. What preventive controls avoid data loss?
15. How do you design defense-in-depth for S3?
16. What lessons learned would you document after a data loss incident?
17. How do you implement DNS failover?
18. How do you test DR readiness?

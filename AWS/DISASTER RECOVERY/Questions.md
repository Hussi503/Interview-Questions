# AWS Disaster Recovery (DR) Interview Handbook
## Part 1 (Q1 & Q2)
**Production Grade | 5-6+ Years DevOps Engineer | Interview Ready**

---

# 🔴 1. What is Disaster Recovery? What are RTO and RPO?

## Direct Answer

Disaster Recovery (DR) is the process of recovering an application's infrastructure, data, and services after a major failure like an AWS Regional outage, database failure, ransomware attack, or accidental deletion.

The goal is to restore the application within the business-defined **Recovery Time Objective (RTO)** and **Recovery Point Objective (RPO)**. In production, DR is always designed based on business requirements, not based on AWS services.

---

## Understanding RTO & RPO

### ✅ RTO (Recovery Time Objective)

**Definition**

The maximum amount of time the application can remain unavailable after a disaster.

**Example**

```
Business Requirement

RTO = 30 Minutes
```

This means the complete application should be available within **30 minutes**.

---

### ✅ RPO (Recovery Point Objective)

**Definition**

The maximum amount of data the business is willing to lose.

**Example**

```
Business Requirement

RPO = 5 Minutes
```

This means losing more than **5 minutes** of data is unacceptable.

---

## Production Implementation

Whenever I design a DR solution, I first discuss RTO and RPO with the business team.

For example,

If the business requires:

```
RTO = 30 Minutes

RPO = < 5 Minutes
```

I typically design the solution using:

- Terraform for Infrastructure as Code
- Route53 for DNS Failover
- Cross-Region Replication for S3
- Cross-region RDS Replication
- AWS Backup
- GitHub Actions / Azure DevOps for automated deployment

This ensures infrastructure, applications, and data can be recovered within the agreed SLA.

---

## Real-Time Scenario

In one of our production projects, the application was customer-facing, so downtime directly impacted business.

The agreed SLA was:

- **RTO:** 30 Minutes
- **RPO:** Less than 5 Minutes

To achieve this, we maintained a **Warm Standby** environment in another AWS Region. Infrastructure was managed using Terraform, S3 data was replicated using CRR, RDS had cross-region replication, and Route53 handled DNS failover.

During quarterly DR drills, the application was successfully recovered within the agreed SLA.

---

## Best Practices

✔ Always define RTO & RPO before designing the architecture.

✔ Automate infrastructure recovery using Terraform.

✔ Perform regular DR drills.

✔ Keep recovery runbooks updated.

✔ Monitor recovery time after every DR test.

---

## Interview Tip

Instead of saying:

> "DR is taking backups."

Say:

> **"Disaster Recovery is a business continuity strategy that combines infrastructure automation, data replication, backups, DNS failover, and application deployment to recover services within the agreed RTO and RPO."**

---

## Remember This

```
Business Requirement

↓

RTO & RPO

↓

Choose DR Strategy

↓

Design Architecture

↓

Automate Recovery

↓

Test Regularly
```

---

# 🔴 2. How will you plan Disaster Recovery?

## Direct Answer

I always start Disaster Recovery planning by understanding the business requirements rather than selecting AWS services.

First, I identify the application's criticality and define the required **RTO** and **RPO**. Then I identify all infrastructure dependencies, choose the appropriate DR strategy, automate the recovery process, and regularly validate it through DR drills.

A DR plan is successful only when it can recover the complete application within the agreed business SLA.

---

## Production Implementation

I usually follow these five steps.

### Step 1 — Understand Business Requirements

I first discuss with stakeholders:

- What is the acceptable downtime?
- What is the acceptable data loss?
- Which applications are business-critical?
- What is the financial impact of downtime?

These answers determine the entire DR architecture.

---

### Step 2 — Identify All Dependencies

I prepare a complete dependency checklist.

This includes:

- VPC
- Subnets
- Security Groups
- Load Balancers
- EC2 Instances
- EKS Clusters
- RDS Databases
- S3 Buckets
- IAM Roles & Policies
- Route53
- Secrets Manager
- SSL Certificates
- CloudWatch Monitoring
- CI/CD Pipelines

Recovering only EC2 instances is not enough. Every dependency required by the application must be available.

---

### Step 3 — Select the DR Strategy

Based on business requirements, I choose the most suitable DR model.

For example:

| Business Requirement | DR Strategy |
|----------------------|------------|
| High downtime acceptable | Backup & Restore |
| Moderate downtime | Warm Standby |
| Near-zero downtime | Active-Active |

The decision depends on RTO, RPO, and business budget.

---

### Step 4 — Automate the Recovery

To reduce manual effort and recovery time, I automate:

- Infrastructure provisioning using Terraform
- Application deployment using GitHub Actions or Azure DevOps
- Database replication using AWS native services
- S3 protection using Versioning + CRR + AWS Backup
- DNS failover using Route53

Automation reduces human error and helps achieve consistent recovery.

---

### Step 5 — Validate the DR Plan

Finally, we conduct regular DR drills.

During each drill, we verify:

- Infrastructure recovery
- Database failover
- Application deployment
- Route53 failover
- Monitoring & Logging
- Actual RTO
- Actual RPO

If any recovery step exceeds the agreed SLA, we update the automation or recovery runbook.

---

## Real-Time Scenario

In one production project, our business required:

```
RTO = 30 Minutes

RPO = Less than 5 Minutes
```

We implemented:

- Terraform for infrastructure
- Warm Standby environment
- Cross-region RDS replication
- S3 Cross-Region Replication
- AWS Backup
- Route53 Failover
- GitHub Actions for deployment

During one quarterly DR drill, we noticed that Route53 health checks were missing from our Terraform code. Although the infrastructure recovered successfully, DNS failover did not happen automatically.

We fixed the Terraform module, validated it in the next DR drill, and updated the recovery runbook. This reinforced the importance of testing the complete recovery process—not just restoring infrastructure.

---

## Best Practices

✔ Start with business RTO & RPO.

✔ Identify every application dependency.

✔ Automate recovery using Terraform.

✔ Maintain DR runbooks.

✔ Conduct quarterly DR drills.

✔ Measure actual RTO & RPO after every test.

---

## Interview Tip

A strong closing statement is:

> **"For me, Disaster Recovery planning is not just about creating backups. It's about understanding business requirements, identifying all application dependencies, automating the recovery process, and continuously validating that we can recover within the agreed RTO and RPO."**

---

## Remember This

```
Business Requirement

↓

Identify Dependencies

↓

Choose DR Strategy

↓

Automate Recovery

↓

Test DR

↓

Improve Continuously
```

---

# 🔴 3. Explain different DR strategies - Backup & Restore vs Warm Standby.

## Direct Answer

AWS provides multiple Disaster Recovery strategies, but in most production environments, the commonly used ones are **Backup & Restore** and **Warm Standby**.

The choice depends on three business factors:

- Required **RTO**
- Required **RPO**
- Budget

If the business can tolerate higher downtime, I choose **Backup & Restore**. If the application is customer-facing and requires faster recovery, I prefer **Warm Standby** because it significantly reduces downtime.

---

## Backup & Restore

### How it Works

In this approach, only backups are maintained in the DR Region.

During a disaster:

```
Infrastructure Creation
        ↓
Restore Database
        ↓
Deploy Application
        ↓
Switch DNS
        ↓
Application Available
```

Nothing is running in the DR Region until a disaster occurs.

---

### Production Implementation

Typical setup includes:

- Terraform for Infrastructure
- AWS Backup
- RDS Snapshots
- EBS Snapshots
- S3 Versioning
- S3 Cross-Region Backup
- GitHub Actions / Azure DevOps

Recovery Steps:

1. Create infrastructure using Terraform.
2. Restore database from snapshots.
3. Deploy application.
4. Restore storage.
5. Update Route53.
6. Validate application.

---

### Suitable For

- Internal applications
- Development environments
- Reporting applications
- Applications with low traffic
- Cost-sensitive workloads

---

### Advantages

✔ Lowest infrastructure cost

✔ Simple to maintain

✔ Easy to implement

---

### Limitations

❌ Higher RTO

❌ Higher RPO

❌ Longer recovery time

---

## Warm Standby

### How it Works

A smaller version of the production environment is always running in another Region.

During a disaster:

```
Scale Resources
      ↓
Promote Database
      ↓
Deploy Latest Code
      ↓
Route53 Failover
      ↓
Application Available
```

Since infrastructure already exists, recovery is much faster.

---

### Production Implementation

Typical setup:

- Small EKS Cluster
- Small Auto Scaling Group
- Small RDS Instance / Read Replica
- S3 Cross-Region Replication
- Route53 Failover
- CloudWatch Monitoring
- Terraform

During failover:

- Auto Scaling increases capacity.
- Database replica becomes primary.
- Route53 redirects traffic.
- CI/CD deploys the latest application version if required.

---

### Suitable For

- Customer Portals
- Banking Applications
- Healthcare Systems
- E-Commerce
- SaaS Applications

---

### Advantages

✔ Low RTO

✔ Faster recovery

✔ Minimal business disruption

---

### Limitations

❌ Higher infrastructure cost

❌ More resources to maintain

---

## Comparison

| Feature | Backup & Restore | Warm Standby |
|----------|-----------------|--------------|
| Cost | Low | Medium |
| Infrastructure Running | No | Yes (Minimal) |
| Recovery Time | High | Low |
| RTO | Hours | Minutes |
| RPO | Higher | Lower |
| Suitable For | Internal Apps | Production Apps |

---

## Real-Time Scenario

In one of our production projects, the application processed customer transactions, and downtime directly impacted business revenue.

The business requirement was:

```
RTO = 30 Minutes

RPO = Less than 5 Minutes
```

Instead of Backup & Restore, we implemented a **Warm Standby** architecture.

Our DR Region contained:

- Small EKS Cluster
- Cross-region RDS Replica
- Replicated S3 Buckets
- Route53 Failover
- Terraform-managed Infrastructure

During quarterly DR testing, we simply scaled the EKS worker nodes, promoted the RDS replica, and Route53 redirected traffic.

The application was fully available in around **20 minutes**, comfortably meeting the business SLA.

---

## Best Practices

✔ Choose DR strategy based on business SLA.

✔ Automate infrastructure provisioning.

✔ Keep infrastructure synchronized.

✔ Perform quarterly DR drills.

✔ Monitor recovery metrics.

---

## Interview Tip

Instead of saying:

> "Warm Standby is faster."

Say:

> **"For customer-facing applications, I usually recommend Warm Standby because the infrastructure is already available in the DR Region. During a disaster, we only scale resources and redirect traffic, which significantly reduces RTO compared to Backup & Restore."**

---

## Remember This

```
Less Budget

↓

Backup & Restore

↓

Higher Recovery Time


Higher Budget

↓

Warm Standby

↓

Lower Recovery Time
```

---

# 🔴 4. How do you design DR for EC2, Databases, and EKS workloads?

## Direct Answer

I never design Disaster Recovery for individual AWS services in isolation. I design it for the complete application because recovering only EC2 or only the database doesn't bring the application back online.

A production DR solution should cover infrastructure, application deployment, databases, storage, networking, secrets, and DNS to ensure the entire application can recover within the agreed RTO and RPO.

---

## EC2 Disaster Recovery

### Production Implementation

For EC2, I avoid manual server recovery.

Instead, I use:

- Terraform for infrastructure
- Launch Templates
- Auto Scaling Groups
- AMIs
- EBS Snapshots
- AWS Backup

Recovery Process:

```
Terraform

↓

Create Infrastructure

↓

Launch EC2

↓

Attach EBS

↓

Deploy Application

↓

Health Check

↓

Serve Traffic
```

---

## Database Disaster Recovery

### Production Implementation

Within the same Region:

- Multi-AZ for High Availability

Across Regions:

- Cross-region Read Replica
- Cross-region Snapshots
- AWS Backup

Choice depends on:

- Business RTO
- Business RPO
- Database Engine

---

## EKS Disaster Recovery

### Production Implementation

For EKS, I protect more than just the cluster.

I protect:

- Terraform code
- Helm Charts
- Kubernetes Manifests
- ECR Images
- Persistent Volumes
- Secrets
- ConfigMaps

Storage protection:

- Velero
- AWS Backup

Container Images:

- Amazon ECR Cross-Region Replication

Deployment:

- GitHub Actions / Azure DevOps

---

## Complete Recovery Flow

```
Terraform

↓

Networking

↓

EKS Cluster

↓

Database Recovery

↓

Restore Storage

↓

Deploy Application

↓

Route53 Failover

↓

Smoke Testing

↓

Production Traffic
```

---

## Real-Time Scenario

In one production environment, our application ran on Amazon EKS with Amazon RDS PostgreSQL.

Business Requirement:

```
RTO = 30 Minutes

RPO = Less than 5 Minutes
```

Implementation:

- Terraform recreated networking and EKS.
- Amazon ECR replicated container images.
- Velero restored persistent volumes.
- Cross-region RDS replica was promoted.
- Route53 switched traffic.
- GitHub Actions deployed the application.

The complete application recovered within **25 minutes**, meeting the agreed SLA.

---

## Best Practices

✔ Manage infrastructure using Terraform.

✔ Replicate container images.

✔ Backup persistent volumes.

✔ Store secrets outside the cluster.

✔ Automate deployments.

✔ Validate recovery regularly.

---

## Interview Tip

A strong closing statement is:

> **"Recovering an EKS cluster alone doesn't recover the application. A successful DR strategy must also recover databases, persistent storage, container images, networking, DNS, secrets, and deployment pipelines."**

---

## Remember This

```
Infrastructure

↓

Database

↓

Storage

↓

Application

↓

DNS

↓

Users
```
# 5. How will you set up DR strategy for S3 Regional Outages?

## Direct Answer

For production S3 workloads,

I implement multiple layers of protection instead of relying on a single AWS feature.

---

## Production Architecture

```
S3 Versioning

↓

Cross Region Replication

↓

AWS Backup

↓

CloudFront

↓

Route53 / MRAP
```

---

## Production Implementation

Configure:

- Versioning
- Cross-Region Replication
- KMS Encryption
- AWS Backup
- Lifecycle Policies
- CloudFront
- Route53

Applications never access bucket names directly.

Instead,

they access CloudFront or Multi-Region Access Point.

---

## Real-Time Example

Customer invoices stored in Mumbai.

Replicated automatically to Singapore.

Mumbai unavailable.

CloudFront + Route53 served files from Singapore.

No application changes required.

---

## Best Practices

✔ Enable Versioning.

✔ Configure CRR.

✔ Configure Backup.

✔ Test Regional failover.

---

## Common Mistakes

❌ Using only CRR.

❌ No Versioning.

❌ Hardcoding bucket names.

---

## Interview Tip

Say:

> "Versioning protects against accidental deletion, CRR protects against Regional failures, and AWS Backup protects against corruption."

---

# 6. What RTO and RPO would you define for S3 DR?

## Direct Answer

There is no standard RTO or RPO.

They depend entirely on business requirements.

---

## Typical Production Values

Customer Documents

```
RTO

15–30 Minutes
```

```
RPO

< 15 Minutes
```

Financial Systems

```
RTO

< 10 Minutes
```

```
RPO

Near Zero
```

Requires more advanced multi-region architecture.

---

## Production Implementation

To achieve these targets:

- Versioning
- Cross-Region Replication
- AWS Backup
- Route53
- CloudFront
- Multi-Region Access Point

---

## Real-Time Example

Business Requirement

```
RTO

20 Minutes
```

```
RPO

5 Minutes
```

Implementation

- CRR
- AWS Backup
- Route53 Failover
- CloudFront

Quarterly DR testing consistently met the SLA.

---

## Best Practices

✔ Start with business SLA.

✔ Design architecture accordingly.

✔ Validate RTO & RPO during every DR drill.

---

## Common Mistakes

❌ Promising unrealistic RTO.

❌ Never measuring actual recovery time.

❌ Confusing HA with DR.

---

## Interview Tip

A senior DevOps engineer always says:

> "RTO and RPO are business decisions. AWS services are selected afterwards to meet those objectives."

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

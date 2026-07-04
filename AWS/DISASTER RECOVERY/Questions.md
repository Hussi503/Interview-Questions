# 1. What is Disaster Recovery? What are RTO and RPO?

## Direct Answer

Disaster Recovery (DR) is the strategy we use to recover our application, infrastructure, and data after a major failure such as:

- AWS Regional outage
- Database corruption
- Ransomware attack
- Accidental deletion
- Infrastructure failure

The goal is to restore business services within the agreed **RTO** and **RPO**.

---

## Understanding RTO & RPO

### RTO (Recovery Time Objective)

- Maximum acceptable downtime.
- Defines how quickly the application should be available again.

**Example**

```
RTO = 30 Minutes
```

Application must be restored within **30 minutes**.

---

### RPO (Recovery Point Objective)

- Maximum acceptable data loss.
- Defines how much recent data the business can afford to lose.

**Example**

```
RPO = 5 Minutes
```

We should not lose more than **5 minutes** of data.

---

## Production Implementation

In production, I never start by selecting AWS services.

I first understand:

- Business criticality
- Acceptable downtime
- Acceptable data loss
- Compliance requirements

Based on those requirements, I design the DR solution.

Typical production implementation includes:

- Terraform for Infrastructure as Code
- Cross-Region Replication for S3
- Cross-region database replication
- AWS Backup
- Route53 Failover
- GitHub Actions for automated deployment
- CloudWatch for monitoring

---

## Real-Time Example

Our production application had:

Business Requirement

- RTO → 30 Minutes
- RPO → Less than 5 Minutes

Implementation

- Warm Standby in another Region
- Cross-Region Replication
- RDS Read Replica
- Terraform
- GitHub Actions
- Route53 Failover

During quarterly DR drills, the application recovered within the agreed SLA.

---

## Best Practices

✔ Define RTO & RPO before designing DR.

✔ Automate infrastructure recovery.

✔ Conduct DR drills regularly.

✔ Maintain recovery runbooks.

---

## Common Mistakes

❌ Thinking DR means only backups.

❌ Never testing DR.

❌ Ignoring DNS, IAM, Secrets, Certificates during recovery.

---

## Interview Tip

Say:

> "Disaster Recovery is a business continuity strategy, not just a backup strategy."

---

# 2. How will you plan Disaster Recovery?

## Direct Answer

I always start DR planning with the business—not AWS services.

The first step is identifying:

- Critical applications
- Business impact
- Acceptable downtime
- Acceptable data loss

These determine the overall DR architecture.

---

## Production Implementation

My DR planning typically follows this sequence:

### Step 1 – Business Impact Analysis

Identify:

- Tier-1 applications
- Tier-2 applications
- Non-critical applications

Assign:

- RTO
- RPO

---

### Step 2 – Identify Dependencies

I list every dependency including:

- VPC
- IAM
- Security Groups
- Load Balancers
- Databases
- S3 Buckets
- EKS Clusters
- Secrets
- DNS
- SSL Certificates

---

### Step 3 – Design Recovery

Infrastructure

- Terraform

Application

- GitHub Actions

Database

- Cross-region replication

Storage

- CRR + AWS Backup

Networking

- Route53 Failover

---

### Step 4 – Testing

Every quarter we perform:

- DR Drill
- Recovery validation
- RTO verification
- RPO verification

---

## Real-Time Example

During one DR drill,

Terraform recreated everything successfully.

But Route53 Health Checks were missing.

Failover didn't happen.

We fixed the Terraform module and updated the DR runbook.

---

## Best Practices

✔ Automate everything.

✔ Document every recovery step.

✔ Test quarterly.

✔ Review DR after every architecture change.

---

## Common Mistakes

❌ Planning only infrastructure recovery.

❌ Forgetting IAM.

❌ Forgetting Secrets.

❌ No DR documentation.

---

## Interview Tip

Say:

> "If even one dependency like IAM or DNS is missed, the application won't recover successfully."

---

# 3. Explain different DR strategies – Backup & Restore vs Warm Standby.

## Direct Answer

The DR strategy depends on business requirements.

The two most common strategies are:

- Backup & Restore
- Warm Standby

The trade-off is cost versus recovery time.

---

## Backup & Restore

### How it works

Only backups exist.

During disaster:

- Create infrastructure
- Restore database
- Deploy application
- Switch DNS

---

### Advantages

✔ Lowest cost

✔ Simple

---

### Disadvantages

❌ High RTO

❌ High RPO

Suitable for:

- Internal applications
- Development workloads

---

## Warm Standby

### How it works

A smaller production environment already runs in another Region.

During disaster:

- Scale infrastructure
- Promote database
- Switch DNS

---

### Advantages

✔ Low downtime

✔ Faster recovery

---

### Disadvantages

❌ Higher infrastructure cost

Suitable for:

- Customer-facing applications
- Business-critical systems

---

## Real-Time Example

Our customer portal required:

```
RTO < 30 Minutes
```

We used:

- Small EKS Cluster
- Small Database
- Minimal EC2 Capacity

During failover,

Auto Scaling increased capacity,

Route53 switched traffic,

Application became fully operational within minutes.

---

## Best Practices

✔ Match DR strategy to business SLA.

✔ Don't choose based only on cost.

---

## Common Mistakes

❌ Choosing Backup & Restore for critical applications.

❌ No automation.

---

## Interview Tip

A senior answer:

> "Business requirements determine the DR strategy—not AWS services."

---

# 4. How do you design DR for EC2, Databases and EKS workloads?

## Direct Answer

I design DR at the application level—not component by component.

Recovering EC2 alone doesn't recover the application.

Everything must recover together.

---

## EC2 Recovery

Infrastructure

- Terraform

Images

- AMIs

Storage

- EBS Snapshots

Scaling

- Auto Scaling Groups

Deployment

- GitHub Actions

---

## Database Recovery

High Availability

- Multi-AZ

Disaster Recovery

- Cross-region replica
- Cross-region snapshots

Depends on:

- Database Engine
- Business RPO

---

## EKS Recovery

Infrastructure

- Terraform

Application

- Helm

Container Images

- Amazon ECR Replication

Persistent Volumes

- Velero
- AWS Backup

Secrets

- AWS Secrets Manager

---

## Recovery Flow

```
Terraform

↓

Infrastructure

↓

Database Recovery

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

## Real-Time Example

During quarterly DR testing,

Entire EKS cluster was recreated.

Applications deployed automatically.

Database promoted.

Traffic switched.

Total recovery:

```
25 Minutes
```

---

## Best Practices

✔ Infrastructure as Code.

✔ GitOps/CI-CD deployment.

✔ Backup persistent volumes.

✔ Automate failover.

---

## Common Mistakes

❌ Backing up only Kubernetes manifests.

❌ Forgetting persistent volumes.

❌ No container image replication.

---

## Interview Tip

Mention:

> "Kubernetes manifests alone are not enough. We must also recover storage, images, secrets, databases, and networking."

---

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

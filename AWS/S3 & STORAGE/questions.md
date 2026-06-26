# AWS S3 Interview Prep (Production-Level Answers)

---

## 1. If an S3 bucket in us-east-1 is down, how will you restore it?

In production, we don’t rely on restoring S3 after failure. S3 is a regional service with high durability, so outages are rare. Instead, we design for failure using Cross-Region Replication (CRR) to a secondary region. If the primary bucket is unavailable, traffic is redirected to the replicated bucket using CloudFront or Route53 failover. Recovery depends on pre-configured multi-region architecture, not post-failure restore.

---

## 2. Can you restore an S3 bucket if the entire AWS region is down?

No, you cannot restore a bucket if the entire region is down unless replication or backup was configured beforehand. S3 is region-specific, so availability depends on that region. In production, we use Cross-Region Replication (CRR) and failover strategies to ensure continuity instead of attempting restoration after failure.

---

## 3. What is the correct approach to handle S3 regional outages?

The correct approach is proactive design. We enable CRR to a secondary region, maintain identical bucket structure, and configure failover via CloudFront or Route53. We also define RPO, monitor replication lag, and ensure application-level failover is in place. The goal is seamless continuity, not post-failure recovery.

---

## 4. Is S3 globally available or region-specific?

S3 is region-specific in terms of data storage and operations. However, it uses a globally unique namespace and can be architected for global access using services like CloudFront and replication strategies.

---

## 5. What is the difference between restore and failover in S3?

Restore is a reactive approach used after data loss or corruption. Examples include restoring from versioning, Glacier, or reprocessing data. It involves downtime.

Failover is a proactive strategy where traffic is switched to another system (e.g., replicated bucket) with minimal downtime using CRR, CloudFront, or Route53.

---

## 6. Without S3 versioning, what options are there to restore data?

If versioning is not enabled, recovery options are limited:
- Restore from external backups or replicated buckets
- Reprocess data from source systems (APIs, DBs, streams)
- Restore from Glacier if archived earlier

S3 itself provides no native recovery without versioning.

---

## 7. Can deleted objects be recovered without versioning?

No. Deleted objects in a non-versioned bucket are permanently removed and cannot be recovered.

---

## 8. Does Cross-Region Replication work without versioning?

No. CRR requires versioning. Replication is based on object versions, and without versioning, S3 cannot track or replicate changes.

---

## 9. What happens if there is no backup and no replication?

Data loss is permanent. Without versioning, backup, or replication, S3 provides no recovery mechanism. In production, this is considered a critical design failure.

---

## 10. Why is versioning mandatory for production S3 buckets?

Versioning is essential for:
- Protecting against accidental deletion or overwrite
- Enabling replication (CRR)
- Supporting audit and rollback
It ensures recoverability in case of failures.

---

## 11. What is S3, and what are the different storage classes?

Amazon S3 is a scalable object storage service designed for high durability and availability.

### Storage Classes:

- **S3 Standard**  
  Frequently accessed data

- **S3 Standard-IA**  
  Infrequent access, lower cost

- **S3 One Zone-IA**  
  Single AZ, cheaper, less resilient

- **S3 Glacier (Flexible Retrieval)**  
  Archival storage (minutes to hours retrieval)

- **S3 Glacier Deep Archive**  
  Lowest cost, long-term archival

- **S3 Intelligent-Tiering**  
  Automatically moves data between tiers based on usage

---

## 12. Explain S3 lifecycle policies

S3 lifecycle policies automate data transition and deletion over time.

### Actions:
- Move data to cheaper storage classes (Standard → IA → Glacier)
- Delete data after retention period
- Manage non-current versions (if versioning enabled)

### Example:
- 0–30 days → Standard  
- 30–90 days → IA  
- 90+ days → Glacier  
- 365 days → Delete  

Lifecycle policies help reduce cost and manage data efficiently without manual effort.

---
``

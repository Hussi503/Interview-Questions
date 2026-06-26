# AWS S3 Interview Q&A (Production Level)

---

## 1. If an S3 bucket in us-east-1 is down, how will you restore it?

---

## 2. Can you restore an S3 bucket if the entire AWS region is down?

---

## 3. What is the correct approach to handle S3 regional outages?

---

## 4. Is S3 globally available or region-specific?

---

## 5. What is the difference between restore and failover in S3?

The key difference between restore and failover in S3 is in how you handle failures and recovery strategy.

👉 Restore is a reactive approach — it’s used when data is lost, deleted, or corrupted. For example:
- Recovering objects using versioning  
- Restoring data from Glacier or Deep Archive  
- Reprocessing data from upstream systems  

In this case, the system experiences downtime or disruption, and we bring data back after the issue.

👉 Failover, on the other hand, is a proactive high-availability strategy. Instead of recovering after failure, we switch to another active or standby system. In S3, this is typically achieved using:
- Cross-Region Replication (CRR)  
- CloudFront origin failover  
- Route53 failover routing  

So if the primary region or bucket is unavailable, traffic is automatically redirected to a secondary replicated bucket with minimal downtime.

---

## 6. Without S3 versioning, what options are there to restore data?

If S3 versioning is not enabled, recovery options are very limited, and that’s why in production we always enable versioning by default.

In such cases, recovery depends on how the data was originally designed and ingested:

👉 First option is external backup or replication — if we had Cross-Region Replication (CRR), downstream data copies (like in another S3 bucket, data lake layer, or warehouse), we can restore from there.

👉 Second option is reprocessing from source systems — for example, if the data came from APIs, databases, or streaming systems like Kafka/Kinesis, we can re-run the ingestion pipeline to rebuild the data.

👉 Third option is Glacier restore, but only if lifecycle policies had moved older copies to Glacier before deletion. Otherwise, this won’t help.

---

## 7. Can deleted objects be recovered without versioning?

No — deleted objects cannot be recovered if versioning is not enabled. Once an object is deleted in a non-versioned bucket, it is permanently removed, and S3 does not retain any history or backup internally.

---

## 8. Does Cross-Region Replication work without versioning?

No, Cross-Region Replication (CRR) does NOT work without versioning — it is a mandatory requirement.

CRR is fundamentally built on object versioning, because S3 tracks replication at the version level — every new object, update, or delete marker is treated as a separate version and then replicated to the destination bucket. Without versioning, S3 has no mechanism to reliably track and replicate changes.

---

## 9. What happens if there is no backup and no replication?

If there’s no versioning, backup, or replication, then S3 data loss is permanent. In production, this is unacceptable — we always enforce versioning, replication, and lifecycle policies as baseline safeguards to prevent such irreversible situations.

---

## 10. Why is versioning mandatory for production S3 buckets?

Versioning is mandatory for production S3 buckets because it is the primary mechanism for data protection and recoverability. Without versioning, any delete or overwrite is permanent, which makes the system highly vulnerable to data loss.

In real production environments, data issues are very common — like accidental deletions, faulty deployments, pipeline bugs, or partial overwrites.

Versioning ensures that every change creates a new object version, so we can always roll back to a previous state without relying on external recovery.

---

## 11. What is S3, and what are the different storage classes?

Amazon S3 is a fully managed object storage service that stores data as objects inside buckets. It is designed for high durability (11 9’s), high availability, and massive scalability, and is commonly used for data lakes, backups, logs, and analytics workloads.

Each object consists of:
- Data  
- Metadata  
- Unique key  

S3 automatically handles storage, replication across AZs, and scaling.

### S3 Storage Classes:

👉 1. S3 Standard  
- Default class for frequently accessed data  
- Low latency, high throughput  
- Used for: active datasets, applications, data lakes raw layer  

👉 2. S3 Standard-IA (Infrequent Access)  
- Lower cost than Standard  
- Slight retrieval cost  
- Used for: backups, older data accessed occasionally  

👉 3. S3 One Zone-IA  
- Stored in a single AZ (not multi-AZ)  
- Cheaper but less resilient  
- Used for: non-critical or easily reproducible data  

👉 4. S3 Glacier (Flexible Retrieval)  
- Archival storage  
- Retrieval takes minutes to hours  
- Used for: compliance, long-term backups  

👉 5. S3 Glacier Deep Archive  
- Lowest cost storage  
- Retrieval takes hours (up to ~12 hrs)  
- Used for: long-term archival (years)  

👉 6. S3 Intelligent-Tiering  
- Automatically moves data between access tiers  
- No need to manage lifecycle manually  
- Used when access patterns are unpredictable  

---

## 12. Explain S3 lifecycle policies

S3 lifecycle policies are a cost optimization and data management feature that allow us to automatically transition or delete objects based on predefined rules over time.

Instead of manually managing data, we define policies so S3 handles it in the background.

### Actions:

👉 Transition actions  
- Move data across storage classes based on age  

Example:
- After 30 days → Standard → Standard-IA  
- After 90 days → Glacier  

👉 Expiration actions  
- Delete data after a certain period  

Example:
- Delete logs after 365 days  

👉 Non-current version handling (if versioning enabled)  
- Delete old versions  
- Move older versions to cheaper storage
``

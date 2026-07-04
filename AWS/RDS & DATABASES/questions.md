# AWS RDS Interview Questions
**Production Grade | 5–6+ Years DevOps Engineer | Easy to Explain**

---

# 🔴 1. What is RDS, and how is it different from running a database on EC2?

## Direct Answer

Amazon RDS is a fully managed relational database service provided by AWS. Instead of installing and managing the database on an EC2 instance, AWS takes care of activities like backups, patching, failover, monitoring, and maintenance. This allows the team to focus more on the application instead of database administration.

### • Fully Managed Service

With RDS, AWS manages database installation, operating system updates, patching, backups, and monitoring. On EC2, we are responsible for all these activities.

### • High Availability

RDS supports Multi-AZ deployment, where AWS automatically performs failover if the primary database becomes unavailable. On EC2, we have to build and manage this setup ourselves.

### • Backups & Monitoring

RDS provides automated backups, snapshots, CloudWatch monitoring, and Performance Insights. On EC2, all these features need to be configured manually.

### • Easy Scaling

We can easily scale the database instance or storage with minimal effort. On EC2, scaling usually requires manual planning and migration.

### • Real-Time Example

In one of my projects, we migrated a MySQL database from EC2 to RDS. After migration, backups became automatic, Multi-AZ improved availability, and patching was handled by AWS. This significantly reduced our operational effort and improved reliability.

## Interview Tip

> **"For production workloads, I always prefer RDS because AWS manages the database infrastructure, while my team can focus on application delivery and performance."**

---

# 🔴 2. How does RDS Multi-AZ failover work?

## Direct Answer

RDS Multi-AZ is designed for High Availability. AWS creates a standby database in another Availability Zone and continuously keeps it synchronized with the primary database. If the primary database fails, AWS automatically promotes the standby instance without requiring manual intervention.

### • Primary and Standby Database

When Multi-AZ is enabled, AWS creates two database instances in different Availability Zones. The standby database is not used for application traffic; it is only for failover.

### • Synchronous Replication

Every write made to the primary database is synchronously replicated to the standby database. This ensures both databases remain in sync and minimizes data loss.

### • Automatic Failover

If the primary instance fails because of hardware issues, Availability Zone failure, or planned maintenance, AWS automatically switches the application to the standby database.

### • Same Database Endpoint

The application always connects using the same RDS endpoint. During failover, AWS updates the endpoint automatically, so no application changes are required.

### • Real-Time Example

In one production environment, AWS performed maintenance on our primary RDS instance. Since Multi-AZ was enabled, the standby instance was promoted automatically. The application experienced only a short interruption of about one minute, and no manual action was required from our team.

## Interview Tip

> **"Multi-AZ is meant for High Availability and automatic failover. It is not used to improve read performance; for that, we use Read Replicas."**

---

# 🔴 3. How do you troubleshoot a slow RDS database?

## Direct Answer

Whenever I receive a complaint that the database is slow, I don't immediately increase the database size. My first priority is to identify the root cause. In most production environments, slow performance is usually caused by inefficient queries or application issues rather than lack of resources.

### • Check CloudWatch Metrics

I first check CPU utilization, memory, IOPS, storage space, and database connections. This helps me understand whether the problem is related to infrastructure.

### • Analyze Performance Insights

If the infrastructure looks healthy, I use Performance Insights to identify slow SQL queries, long-running transactions, or blocking sessions that are affecting performance.

### • Review Database and Application

I check database logs for deadlocks or errors and also verify whether the application is opening too many database connections or executing unnecessary queries.

### • Optimize Before Scaling

If I find missing indexes or inefficient SQL queries, I optimize them first. Only if the database is genuinely resource constrained do I scale the instance or add Read Replicas.

### • Real-Time Example

In one production issue, users reported that the application was responding slowly. CloudWatch showed normal CPU usage, but Performance Insights identified a reporting query performing a full table scan. After adding the required index, query execution time reduced significantly, and the issue was resolved without scaling the database.

## Interview Tip

> **"My approach is always to identify the bottleneck first. In production, optimization comes before scaling because most database performance issues are related to queries, not infrastructure."**
4. How do you install a database on EC2? What are prerequisites?
5. How do you secure a database running on EC2?
6. How do you allow database access only from application servers?
7. How do you monitor and backup a database running on EC2?
8. Database on EC2 vs RDS — which do you prefer and why?

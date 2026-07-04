# 🔴 1. What is RDS, and how is it different from running a database on EC2?


Amazon RDS is a fully managed relational database service where AWS manages the database infrastructure. Compared to running a database on EC2, it reduces operational effort by handling backups, patching, monitoring, and failover automatically.


• **Fully Managed**
- AWS manages installation, patching, upgrades and maintenance.
- On EC2, everything is our responsibility.

• **High Availability**
- RDS supports Multi-AZ with automatic failover.
- On EC2, we have to configure HA manually.

• **Backup & Recovery**
- Automated backups and snapshots are built in.
- EC2 requires manual backup strategy.

• **Monitoring**
- CloudWatch and Performance Insights are available out of the box.
- EC2 needs manual monitoring setup.

• **Real-Time**
- In my project we migrated MySQL from EC2 to RDS mainly to reduce maintenance effort and improve availability.

**Closing Line**

> "For production, I always prefer RDS because AWS manages the database, while we focus on application delivery."

---
# 🔴 2. How does RDS Multi-AZ failover work?


RDS Multi-AZ provides High Availability by maintaining a standby database in another Availability Zone. If the primary database fails, AWS automatically promotes the standby instance.


• Primary database in one AZ

• Standby database in another AZ

• Synchronous replication keeps both databases in sync

• Automatic failover during failure or maintenance

• Same database endpoint, so application changes are not required

• Usually failover completes within 1–2 minutes

• Multi-AZ is for High Availability, not for read scaling

• Real-Time
- We enabled Multi-AZ for production databases.
- During AWS maintenance, failover happened automatically without manual intervention.

**Closing Line**

> "Multi-AZ improves availability, while Read Replicas improve performance."

---
# 🔴 3. How do you troubleshoot a slow RDS database?


Whenever a database becomes slow, I first identify the bottleneck instead of directly increasing the database size.


• Check CloudWatch
- CPU
- Memory
- IOPS
- Connections

• Check Performance Insights
- Slow queries
- Blocking sessions
- Long-running transactions

• Check database logs
- Deadlocks
- Errors

• Verify application
- Connection pooling
- Too many database connections

• Optimize first
- Add indexes
- Optimize queries

• Scale only if required
- Bigger instance
- Read Replica

• Real-Time
- We had a slow reporting query due to a missing index.
- After adding the index, performance improved without scaling RDS.

**Closing Line**

> "My approach is optimize first, scale later."
4. How do you install a database on EC2? What are prerequisites?
5. How do you secure a database running on EC2?
6. How do you allow database access only from application servers?
7. How do you monitor and backup a database running on EC2?
8. Database on EC2 vs RDS — which do you prefer and why?

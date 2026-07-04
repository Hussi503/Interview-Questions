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
# AWS Database on EC2 Interview Questions
**Production Grade | Easy to Remember | 5–6+ Years DevOps Engineer**

---

# 🔴 4. How do you install a database on EC2? What are the prerequisites?


If a project requires a self-managed database, I first provision an EC2 instance, prepare the operating system, install the required database software, and then configure security, storage, backups, and monitoring before making it available to the application.


### • Launch EC2

- Choose the required OS (Amazon Linux, RHEL, Ubuntu)
- Select instance type based on workload
- Attach EBS volumes for database storage

### • Prepare the Server

- Update OS packages
- Configure hostname and timezone
- Install required dependencies

### • Install Database

- Install MySQL / PostgreSQL / SQL Server / Oracle
- Initialize the database
- Create database and users

### • Configure Database

- Enable required port (3306/5432)
- Configure authentication
- Tune basic database parameters

### • Configure Storage & Backup

- Use EBS volumes
- Schedule snapshots
- Configure backup strategy

### • Enable Monitoring

- CloudWatch Agent
- Database logs
- Disk utilization
- CPU & Memory monitoring

### • Real-Time

- In one project, we installed MySQL on EC2 because the application required database-level customization that wasn't supported in RDS. We automated the installation using Ansible and monitored it with CloudWatch.

### Closing Line

> **"For production, I ensure the database is secured, monitored, and backed up before allowing application traffic."**

---

# 🔴 5. How do you secure a database running on EC2?


When a database runs on EC2, security becomes our responsibility. I secure it by restricting network access, enforcing authentication, encrypting data, and continuously monitoring the server.


### • Restrict Network Access

- Allow database port only from application servers
- No public access
- Use private subnet

### • Secure Authentication

- Strong passwords
- Separate users for applications and administrators
- Least privilege permissions

### • Encrypt Data

- Encrypt EBS volumes
- Enable SSL/TLS for database connections
- Protect backup files

### • Patch Regularly

- Keep OS updated
- Apply database security patches
- Remove unused packages

### • Enable Monitoring

- CloudWatch
- Database logs
- Audit logs
- CloudTrail for infrastructure changes

### • Real-Time

- In one production environment, our database EC2 instance was deployed in a private subnet with no public IP. Only application servers could connect, and all administrative access was through a bastion host.

### Closing Line

> **"My approach is simple—keep the database private, encrypt the data, restrict access, and continuously monitor it."**

---

# 🔴 6. How do you allow database access only from application servers?


Instead of allowing access using IP addresses, I use Security Groups. I configure the database Security Group to accept traffic only from the application server Security Group, ensuring no other servers can connect.


### • Private Deployment

- Database stays in a private subnet
- No public IP

### • Security Group Rules

- Allow only database port (3306/5432)
- Source should be Application Server Security Group
- Don't allow 0.0.0.0/0

### • Separate Security Groups

- One Security Group for application servers
- One Security Group for database
- Reference the application Security Group as the source

### • Administrative Access

- Access through Bastion Host or VPN
- No direct internet access

### • Verify Connectivity

- Test from application server
- Confirm other EC2 instances cannot connect

### • Real-Time

- In one project, only the EKS worker nodes were allowed to connect to the MySQL database on port 3306 through Security Group referencing. Even if another EC2 instance existed in the same VPC, it couldn't access the database unless it belonged to the approved Security Group.

### Closing Line

> **"In production, I never whitelist public IPs. I always use Security Group-to-Security Group communication because it's more secure and easier to manage."**
7. How do you monitor and backup a database running on EC2?
8. Database on EC2 vs RDS — which do you prefer and why?

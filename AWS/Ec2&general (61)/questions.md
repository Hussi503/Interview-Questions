# 🚀 AWS DevOps Interview Questions

---

# 🌐 Networking & VPC

### 🔹 Q1. Explain the Serverless Services in AWS
### ✅ Answer
Serverless in AWS means we don't manage servers, operating systems, patching, scaling, or capacity planning. 
AWS handles the infrastructure, and we only focus on writing code or building applications.

In real projects, the most commonly used serverless services are AWS Lambda, API Gateway, DynamoDB, SQS, SNS, 
EventBridge, and Step Functions.

For example, in a production environment, when a user uploads a file to an S3 bucket, an S3 event can trigger 
a Lambda function. The Lambda processes the file, stores metadata in DynamoDB, and sends notifications
through SNS. No EC2 instances are required, and AWS automatically scales based on demand.

Another common use case is exposing REST APIs using API Gateway integrated with Lambda. This is useful for
microservices where traffic is unpredictable because we only pay for actual requests instead of keeping 
servers running 24/7.

From a DevOps perspective, serverless reduces infrastructure management overhead, improves scalability, and 
lowers costs for event-driven workloads. However, we need to consider limitations such as Lambda execution timeout,
cold starts, monitoring, and debugging distributed workflows.

Overall, serverless is best suited for event-driven applications, automation tasks, APIs, data processing pipelines,
and workloads with variable traffic patterns.

### 🔹 Q2. How do you ensure an application is highly available in AWS?
### ✅ Answer
In production, high availability means eliminating single points of failure across every layer of the application. 

Typically, I deploy application servers in an Auto Scaling Group spread across at least two Availability Zones 
and place them behind an Application Load Balancer. If one instance or even an entire Availability Zone goes down, 
the load balancer automatically routes traffic to healthy instances in the other AZ.

For the database layer, I use Multi-AZ deployments, such as RDS Multi-AZ, so AWS automatically performs failover if the primary database becomes unavailable. Static content is stored in S3, which is highly durable by design, and Route 53 
is used for DNS-based routing and health checks. 

I also configure CloudWatch alarms and monitoring to proactively detect failures and trigger notifications.
    
In one of my projects, Sitecore applications were deployed across multiple App Service instances with Azure-like HA concepts, but in AWS the equivalent approach would be ALB + Auto Scaling across multiple AZs with Multi-AZ databases. 

This ensures that infrastructure failures, instance failures, or traffic spikes do not impact application availability.

So, my approach is always to design HA at compute, load balancer, database, storage, and monitoring layers rather than relying on a single component to provide availability.
 

### 🔹 Q3. What is the difference between a Public Subnet and a Private Subnet?
### ✅ Answer
A Public Subnet has a route to an Internet Gateway (IGW) in its route table, which allows resources inside that subnet to communicate directly with the internet. Typically, we place internet-facing components such as Application Load Balancers, Bastion Hosts, or public web servers in public subnets because they need to receive traffic from external users.

A Private Subnet, on the other hand, does not have a direct route to the Internet Gateway. Resources inside private subnets cannot be accessed directly from the internet. We usually place backend application servers,databases, Redis clusters, internal services, and Kubernetes worker nodes in private subnets for security reasons. 

If these resources need outbound internet access for downloading patches, updates, or container images, we route the traffic through a NAT Gateway deployed in a public subnet.

### 🔹 Q4. What is a VPC?
### ✅ Answer
A VPC (Virtual Private Cloud) is a logically isolated network within AWS where we launch and manage our cloud resources such as EC2 instances, Load Balancers, RDS databases, EKS clusters, and other services. 

It gives us complete control over networking, including IP address ranges, subnets, route tables, security groups, and network access.

In real production environments, the first thing we design before deploying applications is the VPC architecture. For example, we create a VPC with a CIDR range like 10.0.0.0/16, divide it into public and private subnets across multiple Availability Zones, deploy Load Balancers in public subnets, application servers in private subnets, and databases in separate private subnets. This provides both security and high availability.

### 🔹 Q5. How many subnets can be created in a VPC?
### ✅ Answer

AWS allows up to around 200 subnets per VPC by default, but practically the number depends on the VPC CIDR range and subnet design. In production, we create subnets based on architecture requirements rather than trying to reach the maximum limit.

### 🔹 Q6. How do Security Groups and NACLs affect EC2 connectivity?
### ✅ Answer
Security Groups and NACLs both control network traffic to and from an EC2 instance, but they operate at different levels and serve different purposes.

 A Security Group acts as a virtual firewall at the instance level. It controls traffic entering and leaving the EC2 instance. Security Groups are stateful, which means if an inbound request is allowed,the response traffic is automatically allowed without needing an explicit outbound rule. In production, Security Groups are our primary mechanism for controlling access to EC2 instances.

A NACL (Network Access Control List) operates at the subnet level and applies to all resources within that subnet. NACLs are stateless, meaning if you allow inbound traffic, you must also explicitly allow the corresponding outbound traffic. NACLs provide an additional layer of security and are often used for broader subnet-level restrictions.

For example, if I am unable to SSH to an EC2 instance, I first verify that the Security Group allows inbound port 22 from my source IP. If that looks correct and connectivity still fails, I check the subnet's NACL to ensure both inbound and outbound rules allow SSH traffic and ephemeral ports. Even if the Security Group allows traffic, a NACL deny rule can still block the connection.

### 🔹 Q7. What is the difference between blocking a CIDR using Security Groups vs NACLs?
### ✅ Answer
The key difference is that Security Groups cannot explicitly deny or block a CIDR, whereas NACLs support both Allow and Deny rules.

Security Groups are stateful firewalls that work on an allow-only model. If I want to restrict access from a specific CIDR, I can only do it indirectly by not allowing that CIDR in the Security Group rules. There is no option to create a "Deny 10.10.10.0/24" rule.

NACLs, however, are stateless and support explicit deny rules. If I need to immediately block a malicious IP range, suspicious traffic, or a specific CIDR across an entire subnet, I can add a Deny rule in the NACL. The traffic will be dropped before it reaches the EC2 instances.
### Real-Time Example
Suppose my web application is running on multiple EC2 instances in a subnet, and a suspicious CIDR block 203.0.113.0/24 is sending unwanted requests.With Security Groups, I cannot create a deny rule for that CIDR. I would have to modify allow rules, which may become difficult to manage.
With NACLs, I can simply add a rule like:
         Rule 100: Deny 203.0.113.0/24
         Port: Any
         Protocol: Any

The traffic gets blocked at the subnet boundary before reaching any EC2 instance.

### 🔹 Q8. Can CIDR blocking be enforced at the subnet level?
### ✅ Answer

Yes, CIDR blocking can be enforced at the subnet level using Network ACLs (NACLs).Since NACLs are attached to subnets and support both Allow and Deny rules, we can block a specific IP address or CIDR range before the traffic reaches any resource inside that subnet. 

This makes NACLs useful when we want to enforce a network-wide restriction for all EC2 instances, Load Balancers, or other resources within a subnet.

### 🔹 Q9. Why is OS-level firewall blocking not preferred in AWS?
### ✅ Answer
In AWS, OS-level firewalls such as iptables, firewalld, or Windows Firewall are generally not the preferred primary security mechanism because AWS already provides centralized network security controls through Security Groups and NACLs.

From a production operations perspective, managing firewall rules at the OS level becomes difficult when you have hundreds of EC2 instances. Every server must be configured, monitored, and kept consistent. 

If a new instance is launched through an Auto Scaling Group, the firewall rules must also be applied correctly, otherwise it can create connectivity issues.

Security Groups and NACLs are managed centrally at the AWS network layer. They provide better visibility, easier auditing, infrastructure-as-code support through Terraform, and consistent security enforcement across all instances. 
This makes them much easier to manage at scale.
### 🔹 Q10. How does NAT Gateway work?
### ✅ Answer
A NAT Gateway allows resources in a private subnet to access the internet while preventing the internet from initiating connections back to those resources.

In a production environment, application servers, EKS worker nodes, and backend services are usually deployed in private subnets for security reasons. 

These servers may still need internet access to download OS patches, pull Docker images from Docker Hub or ECR, install packages, or communicate with external APIs. Since private subnets do not have direct internet access, we use a NAT Gateway.

Internally, the NAT Gateway is deployed in a public subnet and is associated with an Elastic IP. When a private EC2 instance sends traffic to the internet, the route table of the private subnet forwards that traffic to the NAT Gateway.

The NAT Gateway replaces the private IP of the EC2 instance with its own public Elastic IP and sends the request to the internet. When the response comes back, the NAT Gateway tracks the connection, translates the traffic back to
the original private IP, and forwards it to the EC2 instance.

The important point is that communication is outbound only. Internet users cannot directly initiate connections to resources in private subnets through a NAT Gateway.

### 🔹 Q11. How do you secure EC2 instances in a Private Subnet and allow access from outside?
### ✅ Answer
In production environments, we never expose application EC2 instances directly to the internet. Instead, we deploy the EC2 instances in private subnets and expose only the required entry point such as an Application Load Balancer (ALB) or Bastion Host in public subnets.

For application access, users connect to the public ALB, and the ALB forwards traffic to EC2 instances in private subnets. The EC2 Security Group is configured to accept traffic only from the ALB Security Group, ensuring that no direct internet traffic can reach the servers.

For administrative access, earlier we used Bastion Hosts, but nowadays the preferred approach is AWS Systems Manager Session Manager, which allows secure shell access without opening port 22 to the internet.

This eliminates the need for public IPs and significantly reduces the attack surface.

### 🔹 Q12. Have you worked with VPC Endpoints? In what use case?
### ✅ Answer
Yes, I have worked with VPC Endpoints in production environments. We use them when resources inside private subnets need to access AWS services without traversing the public internet or requiring a NAT Gateway.

A VPC Endpoint creates a private connection between the VPC and AWS services such as S3, DynamoDB, ECR, Secrets Manager, SSM, CloudWatch, and others. This improves security because traffic remains within the AWS network and never leaves to the internet.

One common use case I implemented was for EKS worker nodes running in private subnets. The nodes needed to pull container images from ECR and communicate with AWS services. Instead of routing traffic through a NAT Gateway,
    
we configured VPC Endpoints for ECR, S3, and Systems Manager. This reduced NAT Gateway data processing costs and improved security by keeping traffic private.

Another common scenario is accessing S3 from private EC2 instances. By using an S3 Gateway Endpoint, the instances can access S3 directly without internet connectivity.

From a production perspective, VPC Endpoints are often used for security, compliance requirements, and cost optimization, especially when large amounts of traffic are going to AWS services.

---

# 🔐 Security & IAM

### 🔹 Q13. What is a Security Group and why is it needed for EC2?
### ✅ Answer
A Security Group is a stateful virtual firewall that controls inbound and outbound traffic for AWS resources such as EC2 instances, Load Balancers, RDS databases, and EKS nodes. 
    
It is one of the primary security mechanisms in AWS and is attached directly to the resource rather than the subnet.

Security Groups are needed because, by default, we should follow the principle of least privilege, meaning only the required ports and sources should be allowed. 

Without proper Security Group rules, either the application becomes inaccessible or resources become unnecessarily exposed to the internet, creating security risks.
### 🔹 Q14. How will you block a particular CIDR block in AWS permanently?

### ✅ Answer

If I need to permanently block a particular CIDR block in AWS, my preferred approach depends on the scope of the block.

For subnet-level blocking, I would use a **Network ACL (NACL)** because it supports explicit **Deny** rules.

I can add a rule to deny the specific CIDR range, and the traffic will be blocked before reaching any resource in that subnet.

For internet-facing applications behind an ALB, CloudFront, or API Gateway, I would typically use **AWS WAF**. WAF allows me to create **IP sets** and block malicious IP addresses or CIDR ranges centrally. This is usually the preferred enterprise approach because it is easier to manage, audit, and update compared to NACLs.

If the requirement is organization-wide and long-term, I would manage the blocking rules through **Terraform or CloudFormation** so the configuration remains permanent, version-controlled, and automatically reapplied during infrastructure changes.
     
### 🔹 Q15. When would you use AWS WAF instead of Security Groups?

### ✅ Answer

I use Security Groups for network-level access control, whereas I use AWS WAF for application-layer protection.

Security Groups operate at **Layer 3 and Layer 4 (IP, TCP, UDP)** and are mainly used to control which IPs or resources can access specific ports.  
For example:
- Allowing HTTPS traffic on port 443 to an ALB  
- Allowing database access only from application servers  

AWS WAF operates at **Layer 7 (HTTP/HTTPS)** and can inspect the actual web requests. This allows it to:
- Block SQL Injection attacks  
- Prevent Cross-Site Scripting (XSS)  
- Block malicious bots  
- Rate-limit requests  
- Geo-block traffic from specific countries  
- Filter requests based on URLs, headers, cookies, or request patterns  

---

### 🔹 Q16. How do you manage IAM at scale and audit AWS environments?

### ✅ Answer

In large AWS environments, managing IAM manually becomes difficult and error-prone, so I always follow an **IAM-as-Code and least-privilege approach**.

For IAM management, I create roles, policies, and permissions using **Terraform** rather than through the AWS Console. This ensures all access controls are:
- Version-controlled  
- Peer-reviewed  
- Auditable  

Instead of creating IAM users wherever possible, I use:
- **IAM Roles**
- Temporary credentials through **AWS STS**

For cross-account access, I use **role assumption** rather than sharing credentials.

For auditing, **AWS CloudTrail** is the primary service I use. CloudTrail records all API activities such as:
- Resource creation  
- Security Group changes  
- Infrastructure deletion  
- IAM policy updates  

The logs are stored in a centralized **S3 bucket** with:
- Retention policies  
- Encryption enabled  
``

# 📊 Monitoring & CloudWatch

### 🔹 Q17. What are CloudWatch Metrics and how do you set up alerts in CloudWatch?
### ✅ Answer
CloudWatch Metrics are time-series data points that help us monitor the health, performance, and utilization of AWS resources. AWS automatically publishes standard metrics for services like EC2, ALB, RDS, EKS, Lambda, NAT Gateway, and others.

For example, on EC2 I regularly monitor CPU Utilization, Network In/Out, Disk Operations, and Status Check metrics. For ALBs, I monitor Request Count, Target Response Time, HTTP 4XX/5XX errors, and Healthy Host Count. These metrics help us proactively identify performance bottlenecks before users are impacted.

To set up alerts, I create a CloudWatch Alarm on a specific metric and define a threshold. When the threshold is breached for a configured period, the alarm changes state and triggers an SNS notification, which can send emails, Slack messages, PagerDuty alerts, or even invoke Lambda functions for automated remediation.

### 🔹 Q18. How do you enable Detailed Monitoring for EC2 instances?
### ✅ Answer
By default, EC2 instances use Basic Monitoring, where metrics are sent to CloudWatch every 5 minutes. If we need faster visibility and quicker alerting, we enable Detailed Monitoring, which publishes metrics every 1 minute.

We can enable Detailed Monitoring during EC2 instance creation or later through the AWS Console, CLI, Terraform, or CloudFormation. In production environments, I usually enable it on critical application servers, EKS worker nodes, and instances that participate in Auto Scaling, while avoiding unnecessary costs on non-critical systems.

For memory and disk metrics, I still need to install and configure the CloudWatch Agent on the EC2 instance.
---

# 🖥️ EC2 Administration & Troubleshooting

### 🔹 Q19. How do you resolve EC2 instance timeout errors? What are common reasons for SSH timeout?
In real production, SSH timeout on an EC2 instance usually comes down to network path issues, instance reachability, or OS-level problems, so I approach it systematically from outside → inside.

First, I check Security Groups and NACLs. In most cases, SSH timeout is simply because port 22 is not खुला (allowed) from my source IP. I verify inbound rules in SG and also ensure NACLs are not blocking ephemeral ports. In real scenarios, I’ve seen NACL deny rules override SGs causing intermittent timeouts.

Next, I validate network reachability. I check if the instance has a public IP / Elastic IP and is in a subnet with a proper route table (0.0.0.0/0 → IGW). If it’s a private subnet, I confirm I’m connecting via bastion host or VPN. Many times, engineers try direct SSH to private EC2, which leads to timeout.

Then I verify instance health and status checks in AWS. If 2/2 checks are failing, it indicates OS-level hang or underlying host issue. In such cases, I either reboot the instance or use EC2 Serial Console / detach root volume and inspect logs like /var/log/messages for kernel panic or disk issues.

If network is fine, I move to OS-level debugging. I check whether the sshd service is running and listening on port 22. Sometimes high CPU/utilization or disk full situations cause SSH daemon to become unresponsive. In production, I’ve seen cases where too many connections / fail2ban / iptables rules block incoming SSH.

### 🔹 Q20. What checks do you perform when an EC2 instance is running but not reachable? 
refer q19

### 🔹 Q21. How do ALB/NLB timeouts differ from EC2-level timeouts?
ALB/NLB timeouts are at the load balancer layer, meaning the request is reaching AWS but failing before getting a proper response from the target (EC2/app). For ALB, the most common is the idle timeout (default 60s)- if the backend application doesn’t respond within that time, ALB returns a 504 Gateway Timeout. So here, the problem is usually slow application, thread exhaustion, DB latency, or long-running API calls, not network reachability.

In NLB, it's more L4-based, so timeouts usually mean target is not responding or connection is being dropped, often due to app crashes or port-level issues.

On the other hand, EC2-level timeout is a connectivity problem — the request never properly reaches the instance. This is where you see SSH timeouts or app completely unreachable. Causes here are usually Security Groups, NACLs, routing issues, missing public IP, or OS-level problems like sshd down or high resource usage.

If I see 504 errors from ALB, I immediately check **target group health**, **response times**, and **application logs**, because traffic is reaching but app is not responding in time.

If I see connection timeout (no response at all), I start with **SG/NACL, subnet routing, instance status checks**, because request isn’t reaching EC2.

### 🔹 Q22. How do you increase the size of an EBS volume attached to EC2 without downtime?

I use AWS Elastic Volumes to modify the EBS size live, then inside the instance I extend the partition using growpart and resize the filesystem using xfs_growfs or resize2fs depending on FS type — all without downtime

### 🔹 Q23. What commands do you use after resizing an EBS volume?
First, I verify the new disk size:  **lsblk**

This confirms whether AWS-level resize is reflected at the OS level.
Then I check filesystem type: **df-T**

Next, I extend the partition : **sudo growpart /dev/xvda 1**

After that, I resize the filesystem based on type:
         For XFS (Amazon Linux / RHEL): **sudo xfs_growfs /**
         
For EXT4 (Ubuntu, etc.): **sudo resize2fs /dev/xvda1**

Finally, I validate everything: **df -h**

After resizing EBS, I verify disk using lsblk, extend the partition with growpart, resize filesystem using xfs_growfs or resize2fs based on FS type, and finally validate with df -h — ensuring the new space is actually usable.

### 🔹 Q24. Can you attach a single EBS volume to multiple EC2 instances?
By default, an EBS volume can be attached to only one EC2 instance at a time. However, using EBS Multi-Attach with io1/io2 volumes, we can attach a single volume to multiple instances within the same AZ. But in production, this is used only for cluster-aware applications because standard file systems like EXT4/XFS can cause data corruption. For general shared storage use cases, we prefer EFS instead.

### 🔹 Q25. How do you create a custom AMI from an EC2 instance?
In production, we use custom AMIs to standardize our EC2 deployments instead of configuring each server manually. Once an EC2 instance is fully configured—with the required OS patches, security hardening, application dependencies, monitoring agents like CloudWatch, and any organization-specific configurations—we create it as a golden image.

Before creating the AMI, I ensure the application is in a consistent state. For stateless application servers, we can usually create the AMI without downtime using the --no-reboot option if acceptable, but for stateful workloads, we prefer a reboot or brief maintenance window to ensure filesystem consistency. Then, from the EC2 console or AWS CLI, I create the AMI, which automatically takes snapshots of all attached EBS volumes and registers the image.

In our environment, we never launch production instances directly from old AMIs. Instead, we version our AMIs, update the Launch Template with the new AMI ID, and perform a rolling deployment through the Auto Scaling Group. If any issue occurs, rollback is straightforward—we simply point the Launch Template back to the previous AMI version. This approach gives us consistent infrastructure, eliminates configuration drift, and significantly reduces provisioning time for new servers."

### 🔹 Q26. How do you migrate an EC2 instance to another Subnet, Availability Zone, or Region?

The migration approach depends on whether we're moving the instance to another subnet, Availability Zone, or AWS Region, because AWS doesn't allow us to directly move a running EC2 instance between them.

In production, the first step is to create an AMI of the existing EC2 instance after validating that the application is stable.

If the application stores data on EBS volumes, those volumes are captured as snapshots along with the AMI.

Then, I launch a new EC2 instance from that AMI in the target subnet or Availability Zone by selecting the appropriate VPC, subnet, security groups, IAM role, and key pair.

After the instance is up, I validate the application, update any required configurations such as private IPs or database connection strings if needed, and then switch traffic using the Load Balancer or update DNS records. Once everything is verified, I decommission the old instance.

For **cross-region migration**, the process is similar, but first I copy the AMI to the target region because AMIs are region-specific

After the copy completes, I launch the new instance in the destination region and perform the same validation and cutover process.

We always plan the migration with a rollback strategy, so the original instance remains available until the new environment is fully tested."


### 🔹 Q27. How do you recover a stopped EC2 instance that won't start?

The first thing I do is identify why the instance isn't starting rather than immediately trying random fixes. I begin by checking the EC2 instance state, System Status Checks, Instance Status Checks, and the EC2 console output to determine whether it's an infrastructure issue or an operating system issue.

If the system status check fails, it's usually an AWS host issue, and a stop/start operation often moves the instance to healthy underlying hardware.

If the instance status check fails, it's typically an OS-level problem such as a corrupted file system, failed boot process, or incorrect network configuration.

If the instance still doesn't boot, I stop it, detach the root EBS volume, and attach it as a secondary volume to a healthy recovery EC2 instance in the same Availability Zone. From there, I inspect system logs, repair configuration files, fix disk issues, or correct boot-related problems. Once the issue is resolved, I detach the volume, reattach it as the root volume to the original instance, and start it again.

If recovery isn't possible within the expected maintenance window, I don't spend hours troubleshooting production. If we have an Auto Scaling Group or a recent golden AMI, I launch a replacement instance immediately to restore service and then investigate the failed instance separately. Our priority in production is always service availability before root cause analysis."


### 🔹 Q28. How do you handle EC2 key pair loss when SSM is disabled?
If the EC2 key pair is lost and AWS Systems Manager (SSM) isn't enabled, we can't simply retrieve or regenerate the existing private key because AWS never stores it. In production, my priority is to regain access without risking data loss.

If the EC2 key pair is lost and AWS Systems Manager (SSM) isn't enabled, we can't simply retrieve or regenerate the existing private key because AWS never stores it. In production, my priority is to regain access without risking data loss.

I then mount the filesystem and manually add a new public SSH key to the authorized_keys file for the required user, or fix any SSH configuration issues if needed.

After that, I detach the volume, reattach it to the original instance as the root volume, and start the instance. Once it's running, I can log in using the new private key.

After that, I detach the volume, reattach it to the original instance as the root volume, and start the instance. Once it's running, I can log in using the new private key.


### 🔹 Q29. How do you connect to EC2 through a Bastion Host?
In production, we never expose private EC2 instances directly to the internet. Instead, we deploy a Bastion Host in a public subnet, while application and database servers remain in private subnets with no public IPs.

To connect, I first SSH into the Bastion Host using its public IP and my authorized key or enterprise authentication.

From the Bastion Host, I SSH into the target EC2 instance using its private IP.

The security groups are configured so that the Bastion Host accepts SSH only from our corporate IP range or VPN, and the private EC2 instances allow SSH only from the Bastion Host's security group—not from the internet.

This ensures that administrative access is tightly controlled and all production servers remain isolated from public networks.

in modern AWS environments, our preferred approach is AWS Systems Manager Session Manager, which eliminates the need for a Bastion Host altogether by providing secure, auditable access without opening port 22. We use a Bastion Host only when there's a specific business or legacy requirement."

### 🔹 Q30. How do you troubleshoot slow SSH connections to EC2?
When SSH is slow, I don't assume it's a network issue immediately. I first identify where the delay is occurring—whether it's during connection establishment, authentication, or after login. If the delay is before the SSH prompt appears

I check the Security Groups, NACLs, route tables, VPN connectivity, and network latency. I also verify the instance health and CloudWatch metrics to ensure the EC2 instance isn't under CPU, memory, or disk pressure.

If the delay occurs after authentication, I log in and check system resource utilization using tools like top, htop, iostat, vmstat, and df -h

One common issue I've seen is high CPU utilization, low memory causing swap usage, or a full root filesystem, all of which can make SSH sessions appear slow.



### 🔹 Q31. How do you check EC2 system logs for boot errors?

When an EC2 instance fails to boot or doesn't become reachable, the first thing I do is determine whether it's an AWS infrastructure issue or an operating system issue by checking the EC2 System Status Checks and Instance Status Checks.

If the instance is failing at the OS level, I review the EC2 System Log (Console Output) from the AWS Console because it captures the boot sequence even if I can't SSH into the server.

In the EC2 Console, I select the instance and navigate to Actions → Monitor and troubleshoot → Get system log (or Get console output). I review the boot messages for errors such as kernel panic, filesystem corruption, failed services, disk mount failures, or bootloader issues. These logs often indicate why the instance didn't complete the boot process.

If the instance boots but SSH isn't available, I use the EC2 Serial Console (if enabled) to access the instance and investigate further. If neither method helps and the instance remains inaccessible, I detach the root EBS volume, attach it to a recovery instance, and inspect logs like /var/log/messages, /var/log/boot.log, /var/log/dmesg, or journalctl depending on the operating system



### 🔹 Q32. How do you troubleshoot failed EC2 status checks?

The first thing I do is identify which status check is failing, because AWS performs two different health checks: System Status Check and Instance Status Check. My troubleshooting approach depends on the failed check rather than applying a generic fix.

If the System Status Check fails, it usually indicates an issue with the underlying AWS infrastructure, such as host hardware, networking, or power. In that case, I perform a stop/start operation, which migrates the instance to healthy underlying hardware. I also verify the AWS Health Dashboard to check if there's an ongoing infrastructure event.

If the Instance Status Check fails, the problem is inside the operating system. I first review the EC2 Console Output and Serial Console to identify boot errors. If I can access the server, I check CPU, memory, disk usage, filesystem health, failed services, and system logs using journalctl and dmesg. If the instance is inaccessible, I detach the root EBS volume, attach it to a recovery instance, and analyze the logs offline.



### 🔹 Q33. How do you schedule EC2 instances to stop/start automatically?

In production, we don't schedule critical application or database servers because they need to be available 24/7. However, for non-production environments like Development, QA, or UAT, we automate start and stop schedules to optimize costs.

The approach I've used is Amazon EventBridge with AWS Lambda. We create EventBridge schedules using cron expressions—for example, starting EC2 instances at 8 AM and stopping them at 8 PM on weekdays.

The EventBridge rule triggers a Lambda function, which uses the AWS SDK to start or stop EC2 instances based on tags such as Environment=Dev or AutoSchedule=True. This tag-based approach avoids hardcoding instance IDs and makes the solution scalable as new instances are added.





### 🔹 Q34. How do you use User Data to install software automatically on EC2?

In production, we use User Data for initial bootstrapping of EC2 instances. Instead of manually logging into every server to install software, we provide a startup script when launching the instance. When the EC2 instance boots for the first time, the cloud-init service executes the script automatically.

We typically use User Data to install packages like Nginx, Apache, Docker, CloudWatch Agent, or the SSM Agent, configure the application environment, download deployment artifacts from S3, and register the instance with monitoring services. This ensures every instance is configured consistently and eliminates manual intervention.

However, we keep User Data lightweight. We don't use it for complex application deployments or long-running configuration tasks because if the script fails, troubleshooting becomes difficult.

---

# 💾 Storage (S3, EBS, EFS)

### 🔹 Q35. What is the difference between S3 Standard and S3 Glacier?

The primary difference is the access pattern and cost. S3 Standard is designed for frequently accessed data where low latency and high availability are required. We use it for application assets, static websites, log files, deployment artifacts, and any data that applications need to access immediately

S3 Glacier, on the other hand, is designed for long-term archival. It's much cheaper to store data, but retrieval is not instant and can take from a few minutes to several hours depending on the retrieval option. Because of that, it's not suitable for applications that need real-time access.


### 🔹 Q36. How do you attach, mount, and increase the size of an EBS volume?

this usually happens when application data or log files are growing and the existing disk is running out of space. Before making any changes, I first verify the current disk utilization using df -h and confirm which EBS volume needs to be expanded. As a best practice, I also take an EBS snapshot so we have a rollback option.

If it's a new EBS volume, I create the volume in the same Availability Zone as the EC2 instance, attach it through the EC2 console or CLI, verify that the operating system detects it using lsblk, create a filesystem using mkfs, mount it to the required directory, and update /etc/fstab so it mounts automatically after a reboot.

If the requirement is to increase the size of an existing EBS volume, AWS allows online volume expansion. I modify the volume size from the AWS Console, wait for the optimization to complete, and then extend the partition and filesystem inside the operating system using tools like growpart, xfs_growfs, or resize2fs depending on the filesystem. This allows us to increase storage with little or no downtime, which is how we typically handle production environments."

### 🔹 Q37. How do you create an EBS Snapshot and recover data from it?
In production, EBS snapshots are our primary backup mechanism for EC2 volumes. We typically create snapshots before activities such as OS patching, application upgrades, volume expansion, or any infrastructure change that could impact data. Snapshots are incremental, so after the first full snapshot, AWS stores only the changed blocks, which makes them storage-efficient

If I need to recover data, I don't restore the snapshot directly to the existing volume unless it's absolutely necessary. Instead, I create a new EBS volume from the snapshot in the same Availability Zone, attach it to either the original EC2 instance or a recovery instance, and verify the data. If it's only a few files that were deleted, I simply copy those files back. If the entire volume is corrupted, I replace the old volume with the new volume created from the snapshot. This approach minimizes risk because the original volume remains untouched until we've verified the recovery.

### 🔹 Q38. What is the difference between EBS and EFS?

The main difference is that EBS is block storage attached to a single EC2 instance, whereas EFS is a managed network file system that can be mounted by multiple EC2 instances simultaneously.

In production, I use EBS when the application requires high-performance storage for a single server, such as the operating system, databases, or application data. Since it's block storage, the operating system formats and mounts it as a disk. EBS volumes are tied to a single Availability Zone and are typically attached to one EC2 instance at a time.

I use EFS when multiple application servers need to access the same files. For example, in an Auto Scaling environment where several EC2 instances are running behind an Application Load Balancer, they may all need access to shared uploads, images, or configuration files. Instead of synchronizing files between servers, all instances mount the same EFS file system over NFS. Since EFS is a regional service, it can be accessed from multiple Availability Zones, making it highly available without additional replication."

---

# ⚖️ Load Balancers & High Availability

### 🔹 Q39. How do you design a Highly Available Architecture in AWS? What components are required?

When I design a highly available architecture in AWS, the objective is to eliminate single points of failure so that the application continues to serve traffic even if an Availability Zone or an individual server fails.

Typically, I start with a VPC spanning at least two Availability Zones. Public-facing components like the Application Load Balancer (ALB) are deployed across both AZs. The application servers run in private subnets as part of an Auto Scaling Group, with instances distributed across multiple Availability Zones. This ensures that if one instance or even an entire Availability Zone fails, the Load Balancer automatically routes traffic to healthy instances in the other AZ, and the Auto Scaling Group replaces failed instances automatically.

For the database layer, I use Amazon RDS Multi-AZ, which provides synchronous replication and automatic failover. Static content such as images or deployment artifacts is stored in Amazon S3, and if multiple application servers need shared file storage, I use Amazon EFS. DNS is managed through Route 53, and monitoring is handled using CloudWatch with alarms for CPU, memory, application health, and Load Balancer metrics. For security, application servers remain in private subnets, security groups follow least privilege, and secrets are stored in AWS Secrets Manager. This architecture provides high availability, scalability, and resilience while minimizing downtime

### 🔹 Q40. What is the difference between Application Load Balancer (ALB) and Network Load Balancer (NLB)?

The primary difference is the OSI layer they operate on and the type of traffic they handle. Application Load Balancer (ALB) operates at Layer 7 (HTTP/HTTPS), whereas Network Load Balancer (NLB) operates at Layer 4 (TCP/UDP/TLS).

In production, I use ALB for web applications and REST APIs because it provides advanced routing capabilities such as host-based routing, path-based routing, SSL termination, HTTP-to-HTTPS redirection, and integration with AWS WAF. For example, if multiple microservices like /api, /admin, and /payment are hosted behind the same load balancer, ALB can intelligently route requests to different target groups.

I choose NLB when the application requires extremely high performance, very low latency, or uses non-HTTP protocols like TCP or UDP. NLB preserves the client source IP by default, supports static IP addresses and Elastic IPs, and is commonly used for applications such as databases, gaming servers, or other TCP-based services.

### 🔹 Q41. How will you create a Load Balancer through the AWS Console?

In production, we generally provision Load Balancers using Terraform or CloudFormation so the infrastructure is version-controlled and reproducible. However, if I need to create one through the AWS Console for testing or troubleshooting, I follow a structured approach.

I first go to EC2 → Load Balancers → Create Load Balancer and choose the appropriate type, usually an Application Load Balancer for web applications. Then I provide the name, select Internet-facing or Internal based on the application, choose the VPC, and select at least two public subnets across different Availability Zones to ensure high availability.

Next, I configure a Security Group that allows HTTP or HTTPS traffic, create or select a Target Group, define the health check path such as /health or /healthz, and register the EC2 instances or let the Auto Scaling Group register them automatically. For HTTPS, I attach an SSL certificate from AWS Certificate Manager (ACM). Finally, I review the configuration, create the Load Balancer, and verify that the targets become healthy before updating DNS in Route 53 or pointing the application to the Load Balancer's DNS name.

### 🔹 Q42. How do you design a Highly Available EC2 Architecture across multiple Availability Zones?

When I design a highly available EC2 architecture, my primary goal is to eliminate single points of failure so that the application continues to run even if an EC2 instance or an entire Availability Zone becomes unavailable.

I start by creating a VPC with public and private subnets spread across at least two Availability Zones. The Application Load Balancer is deployed in the public subnets, while the EC2 application servers are deployed in the private subnets as part of an Auto Scaling Group. The Auto Scaling Group is configured to launch instances across both Availability Zones and uses a Launch Template so every instance is built from the same golden AMI.

The ALB continuously performs health checks on the application. If an instance becomes unhealthy, the ALB stops sending traffic to it, and the Auto Scaling Group automatically terminates and replaces it with a new healthy instance. For the database, I use RDS Multi-AZ, and if the application requires shared storage across multiple EC2 instances, I use Amazon EFS. Monitoring is configured through CloudWatch, and DNS is managed using Route 53. This architecture provides high availability, self-healing, and automatic scaling without manual intervention.

---

# 📈 Auto Scaling & Deployments

### 🔹 Q43. How does Auto Scaling work and how do you handle sudden traffic spikes?
In production, I never treat Auto Scaling as just adding more EC2 instances. It's a combination of Elastic Load Balancer, Auto Scaling Groups, CloudWatch metrics, and application design working together.

Typically, I configure an Auto Scaling Group with a minimum, desired, and maximum capacity spread across at least two or three Availability Zones for high availability.

The Auto Scaling Group continuously monitors CloudWatch metrics such as CPU utilization, Request Count per Target, ALB response time, or custom application metrics. When the configured threshold is breached, it launches new EC2 instances automatically using a Launch Template.

These instances bootstrap themselves, pull the latest application version, complete health checks, and only then are registered with the Application Load Balancer.

Similarly, when traffic reduces, instances are terminated gracefully after connection draining so that users don't experience interrupted sessions.

One lesson I've learned in production is that Auto Scaling alone cannot solve performance issues. If the application has a database bottleneck or memory leak, adding more EC2 instances simply increases the load on the backend

So, before tuning scaling policies, I always identify the actual bottleneck using CloudWatch dashboards, ALB metrics, application logs, and APM tools, and then scale the appropriate layer rather than blindly increasing compute capacity.

### 🔹 Q44. How do you achieve Zero-Downtime Deployments?

In production, zero-downtime deployment is not just about deploying new code—it's about ensuring users never notice the deployment.

The approach I follow depends on the application architecture, but for EC2-based applications behind an ALB, I typically use Blue-Green or Rolling deployments with an Auto Scaling Group.

The deployment pipeline first creates or updates a new set of instances with the latest application version. These instances are validated through application health checks, and only after they become healthy are they registered with the Application Load Balancer.

The ALB starts routing traffic gradually to the new instances while the old instances continue serving requests.

Once the new environment is stable, the old instances are drained using the ALB's deregistration delay, allowing existing user sessions to complete before the instances are terminated.

If any health checks fail or CloudWatch alarms are triggered, the deployment is automatically rolled back by directing traffic back to the previous healthy version.




****Kubernetes** AKS/EKS**
In Kubernetes environments like EKS or AKS, I achieve zero-downtime deployments using the **Rolling Update** strategy, which is the default deployment strategy. I configure parameters like maxUnavailable: 0 and maxSurge: 1 or 25%, so Kubernetes first 
creates new pods before terminating the old ones. This ensures the desired number of healthy pods are always available to serve traffic.

In Kubernetes environments like EKS or AKS, I achieve zero-downtime deployments using the Rolling Update strategy, which is the default deployment strategy. I configure parameters like maxUnavailable: 0 and maxSurge: 1 or 25%, so Kubernetes first creates new pods before terminating the old ones. This ensures the desired number of healthy pods are always available to serve traffic.

During deployment, the old pods continue serving requests while the new pods start up, initialize, and become healthy.

Once the new pods are ready, Kubernetes gradually terminates the old pods. We also configure a Pod Disruption Budget (PDB) to ensure a minimum number of replicas remain available during deployments or node maintenance, preventing service interruptions.

For critical production applications, we don't rely only on Rolling Updates. We implement Blue-Green or Canary deployments using tools like Argo Rollouts.

This allows us to shift traffic gradually—for example, 10%, then 30%, then 50%, and finally 100%—while continuously monitoring latency, error rates, and business metrics.

If any issue is detected, traffic is immediately shifted back to the stable version without affecting users. This approach significantly reduces deployment risk compared to sending all traffic to the new version at once.




### 🔹 Q45. If a web application experiences unpredictable traffic spikes, how would you configure Auto Scaling?

### 🔹 Q46. In what scenario is Scheduled Scaling more appropriate than Dynamic Scaling?

---

# 🗄️ Database Administration

### 🔹 Q47. How do you install a Database on EC2? What are the prerequisites?

### 🔹 Q48. How do you secure a Database running on EC2?

### 🔹 Q49. How do you allow Database access only from Application Servers?

### 🔹 Q50. How do you monitor and back up a Database running on EC2?

### 🔹 Q51. Database on EC2 vs RDS — which do you prefer and why?

---

# 🚀 Serverless Services

### 🔹 Q52. Do you have exposure to API Gateway and Lambda Functions?

### 🔹 Q53. Explain AWS Lambda functionality and its purpose.

---

# 🐳 Containers & Compute

### 🔹 Q54. What is the difference between EC2 and ECS?

---

# 🏗️ Real-Time Project Experience

### 🔹 Q55. What AWS services have you worked on?

### 🔹 Q56. Can you elaborate on AWS services and your confidence level?

### 🔹 Q57. What are the services you have end-to-end configured in past projects?

### 🔹 Q58. Write code to create an EC2 instance and attach a Load Balancer and Auto Scaling Group.

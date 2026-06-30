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

### 🔹 Q25. How do you create a custom AMI from an EC2 instance?

### 🔹 Q26. How do you migrate an EC2 instance to another Subnet, Availability Zone, or Region?

### 🔹 Q27. How do you recover a stopped EC2 instance that won't start?

### 🔹 Q28. How do you handle EC2 key pair loss when SSM is disabled?

### 🔹 Q29. How do you connect to EC2 through a Bastion Host?

### 🔹 Q30. How do you troubleshoot slow SSH connections to EC2?

### 🔹 Q31. How do you check EC2 system logs for boot errors?

### 🔹 Q32. How do you troubleshoot failed EC2 status checks?

### 🔹 Q33. How do you schedule EC2 instances to stop/start automatically?

### 🔹 Q34. How do you use User Data to install software automatically on EC2?

---

# 💾 Storage (S3, EBS, EFS)

### 🔹 Q35. What is the difference between S3 Standard and S3 Glacier?

### 🔹 Q36. How do you attach, mount, and increase the size of an EBS volume?

### 🔹 Q37. How do you create an EBS Snapshot and recover data from it?

### 🔹 Q38. What is the difference between EBS and EFS?

---

# ⚖️ Load Balancers & High Availability

### 🔹 Q39. How do you design a Highly Available Architecture in AWS? What components are required?

### 🔹 Q40. What is the difference between Application Load Balancer (ALB) and Network Load Balancer (NLB)?

### 🔹 Q41. How will you create a Load Balancer through the AWS Console?

### 🔹 Q42. How do you design a Highly Available EC2 Architecture across multiple Availability Zones?

---

# 📈 Auto Scaling & Deployments

### 🔹 Q43. How does Auto Scaling work and how do you handle sudden traffic spikes?

### 🔹 Q44. How do you achieve Zero-Downtime Deployments?

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

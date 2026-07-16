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

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
In production, for unpredictable traffic, I avoid using only simple CPU-based scaling because CPU often reacts after the application is already under load. Instead, I use a combination of Target Tracking, Step Scaling, and proper application design.

I configure the application behind an Application Load Balancer with an Auto Scaling Group spread across multiple Availability Zones.

The ASG has a minimum capacity to handle normal traffic, a desired capacity based on average load, and a maximum capacity to protect against runaway scaling.

For scaling, I primarily use Target Tracking Policies, for example maintaining average CPU around 50–60% or scaling based on ALB Request Count Per Target, which is a much better metric for web applications than CPU alone.

If traffic increases rapidly, additional instances are launched automatically. For extreme spikes, I also configure Step Scaling so that if CPU exceeds a higher threshold, such as 80–85%, multiple instances are added in a single scaling event instead of one at a time.

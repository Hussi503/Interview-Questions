### 🔴 1. What roles are defined in AWS in your organization? Hierarchy level?
In my organization, we follow role-based access control and the principle of least privilege.

We don't give broad administrator access to everyone. Access is granted based on responsibility and operational requirements.

At the top level, we have the **Cloud Platform** or **Enterprise Admin** team, which manages AWS Organizations, billing, SCPs, account creation, and overall governance.

Below that, we have Cloud Administrators who manage core infrastructure such as VPCs, IAM, networking, security controls, EKS clusters, and shared services.

For DevOps engineers like us, we typically have PowerUser or custom DevOps roles that allow managing CI/CD pipelines, EKS, EC2, ALBs, Route53, CloudWatch, and application infrastructure, but with restricted access to sensitive IAM and billing functions.

Developers usually have limited access focused on application-related services such as viewing logs, accessing specific S3 buckets, ECR repositories, and monitoring dashboards. They generally don't get infrastructure-level permissions.

For production environments, we use separate read-only, operational, and break-glass admin roles. Access is assumed using IAM roles through SSO, and all activities are audited through CloudTrail.


### 🔴 2. Apart from Administrator and Read-only, what other roles do you define?

Yes, apart from Administrator and ReadOnly, we define multiple custom IAM roles based on responsibilities. In mature organizations, we rarely give full Administrator access because it's a security risk.

For DevOps teams, we usually have a DevOps Engineer role with permissions to manage EKS, EC2, ALB, Auto Scaling, CloudWatch, ECR, Route53, and CI/CD resources, but restricted access to IAM, billing, and organization-level settings.

For developers, we create application-specific roles that allow access to logs, ECR, S3, Parameter Store, Secrets Manager, and deployment pipelines, but they cannot modify core infrastructure.

For support teams, we generally have Operations or Support roles that can restart services, view monitoring dashboards, access logs, and perform basic troubleshooting.

For security teams, we create Security Auditor roles with access to CloudTrail, Config, GuardDuty, Security Hub, IAM reports, and compliance-related services.

### 🔴 3. What kind of permissions does Power User have?

**Power User** is a commonly used role in AWS that provides almost all permissions to AWS services and resources, but it does not allow users to manage IAM users, roles, policies, or account-level security settings.

a Power User can create and manage resources such as EC2 instances, EKS clusters, Load Balancers, Auto Scaling Groups, RDS databases, S3 buckets, Lambda functions, CloudWatch dashboards, and other infrastructure components.

However, they cannot create new IAM users, attach admin policies, modify permission boundaries, or elevate their own privileges.

In my projects, DevOps engineers often work with a customized Power User role rather than the AWS-managed one. For example, they can manage EKS, EC2, ALB, Route53, ECR, CloudWatch, and CI/CD services, but access to IAM, Organizations, SCPs, Billing, and account governance remains restricted to platform administrators.

The main advantage of using a Power User role is that engineers can perform day-to-day infrastructure and deployment activities independently while preventing privilege escalation and reducing security risks.


### 🔴4. In cloud terminology, what is the role name for modifying architecture?

In cloud terminology, the role responsible for modifying or designing architecture is typically called a Cloud Architect, Solutions Architect, or Enterprise Architect, depending on the organization's structure.

From an AWS perspective, a Solutions Architect usually has the responsibility and permissions to design, review, and approve architectural changes such as VPC redesign, multi-region deployments, EKS architecture, networking strategies, security models, disaster recovery plans, and scalability improvements.

### 🔴5. Do you work on AWS Redshift, SNS, SQS, Glue, Kinesis?

### 🔴6. What storage services have you worked on?
I've worked primarily with Amazon S3, EBS, EFS, and to some extent FSx depending on the application requirements

**S3** is the storage service I use most frequently for artifacts, application backups, log archival, Terraform state files, static website hosting, and CI/CD artifacts.

In production, we also use lifecycle policies to move older data to cheaper storage classes and enable versioning for recovery purposes.

For EKS and EC2 workloads, I've extensively worked with **EBS** volumes. We use EBS for persistent storage where low latency and high performance are required, such as application data, databases, and container persistent volumes through the EBS CSI driver.

I've also used EFS for shared storage scenarios. For example, when multiple pods across different nodes need access to the same file system, EFS works well because it can be mounted simultaneously by multiple pods.


### 🔴7. If SNS is there, why should we use Prometheus and Grafana?
Actually SNS and Prometheus/Grafana solve completely different problems, so they are not alternatives.

SNS is a notification service. Its job is to send messages or alerts to email, Slack, Lambda, SQS, or other subscribers when an event occurs. It doesn't store performance metrics, provide dashboards, or help with troubleshooting.

**Prometheus** and **Grafana** are monitoring and observability tools. Prometheus continuously collects metrics such as CPU utilization, memory usage, pod restarts, API latency, request count, and error rates.

**Grafana** visualizes those metrics through dashboards and helps us identify trends and bottlenecks.

In a real production environment, Prometheus detects that CPU utilization on an EKS pod has crossed 80% for the last 10 minutes. 

Grafana shows the trend and impact. 

Then Alertmanager can trigger an alert through SNS, which sends a notification to the DevOps team on email, Slack, or PagerDuty.

So SNS is only the messenger. Prometheus identifies the problem, Grafana helps us


### 🔴8. What does it mean to run an ECS service in a private subnet?

Running an ECS service in a private subnet means the ECS tasks or containers do not get public IP addresses and are not directly accessible from the internet. This is a security best practice for production workloads.

the ECS tasks run inside private subnets, while an Application Load Balancer (ALB) sits in public subnets.

Internet traffic reaches the ALB first, and the ALB forwards requests to the ECS tasks in the private subnet.

This ensures the application is accessible to users, but the containers themselves are not exposed to the public internet.

If the containers need outbound internet access—for example, to pull Docker images from ECR, download patches, or call external APIs—we provide that access through a NAT Gateway instead of assigning public IPs.


### 🔴9. How do you move an ECS service from public to private subnets?

In production, I wouldn't modify the existing ECS service directly because it introduces risk. Instead, I would create a new deployment in private subnets and perform a controlled cutover.

First, I would ensure the private subnets have the required routes and NAT Gateway access because ECS tasks may need outbound connectivity to pull images from ECR, send logs to CloudWatch, or call external APIs.

Next, I would update the ECS service network configuration to use the private subnets and disable public IP assignment. If the application is internet-facing, I would place an ALB in public subnets and configure the target group to route traffic to the ECS tasks running in private subnets.

Before switching traffic, I would validate that the tasks can register successfully with the target group, pass health checks, communicate with dependent services such as databases, and send logs and metrics correctly.

Once validation is complete, I would perform a rolling deployment or blue-green deployment, monitor CloudWatch metrics, ALB target health, and application logs, and then gradually shift traffic to the new tasks. After confirming stability, I would decommission the old public-subnet deployment.


10. What happens if ECS tasks in private subnets don't have NAT Gateway access?
11. Why must an internet-facing ALB be replaced with an internal ALB when making ECS private?
12. Can we convert an existing public ALB to internal? Why or why not?
13. How do ECS tasks pull images from ECR when running in private subnets?
14. What security group changes are required when converting ECS from public to private?
15. How do you expose a private ECS service to on-prem users?

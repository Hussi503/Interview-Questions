# DevOps Engineer – Daily Work, CI/CD Flow & Incident Handling

---

## Daily Roles & Responsibilities

As a DevOps Engineer, my typical day starts by checking the health of production and staging environments using CloudWatch and Grafana dashboards. I ensure that EC2 instances, EKS clusters, and Jenkins jobs are running without issues and there are no active alerts.

I then attend daily stand-ups with developers and QA teams to discuss deployments, blockers, and upcoming releases. Post that, I review Jenkins pipelines to ensure builds and deployments are successful. If any jobs fail, I troubleshoot issues, update pipeline scripts, and fine-tune build stages.

A significant part of my work involves infrastructure management using Terraform and Ansible. I write reusable Terraform modules for provisioning AWS resources like EC2, S3, IAM, and EKS. For configuration management, I use Ansible to automate server setup and configuration across environments.

I also work extensively with Docker and Kubernetes — building container images, pushing them to ECR, and deploying them on EKS using Helm charts. Throughout the day, I monitor logs via CloudWatch or ELK and respond to alerts, performing root cause analysis when required.

Towards the end of the day, I focus on optimizing CI/CD pipelines, automating repetitive tasks, and maintaining documentation in Confluence or Jira. My overall goal is to ensure smooth deployments, stable infrastructure, and continuous automation improvements.

---

## CI/CD Pipeline Architecture

In our CI/CD setup, the pipeline is triggered when developers push code to GitHub. A webhook notifies Jenkins, which pulls the latest code and starts the build process.

During the build stage:
- Unit tests are executed  
- Static code analysis is performed using SonarQube  
- Security scans are conducted  

Once the build is successful:
- A Docker image is created  
- Tagged using build number and commit ID  
- Pushed to Amazon ECR  

In the deployment phase:
- Jenkins triggers Helm deployments  
- Applications are deployed to EKS clusters  
- Rolling update strategy ensures zero downtime  

Post-deployment:
- Smoke tests and health checks are executed  
- If failures occur, automatic rollback is triggered using Helm revision history  

Notifications:
- Integrated with Slack and email  
- Keeps teams informed about build and deployment status  

Monitoring:
- CloudWatch and Grafana used for observability  
- Alerts configured for proactive issue detection  

This pipeline is fully automated and version-controlled, ensuring faster and reliable releases.

---

## Role Summary (Short Impact Version)

In my current role, I manage highly available CI/CD pipelines and cloud infrastructure across multiple environments (dev, QA, staging, prod).

- Reduced deployment time by 40% via automation  
- Implemented blue-green deployments for zero downtime  
- Used Terraform with remote state (S3 + DynamoDB locking)  
- Built Docker images and deployed to EKS using Helm  
- Ensured security via IAM roles and AWS Secrets Manager  
- Designed observability using Prometheus, Grafana, and CloudWatch  

I focus on automation, reliability, scalability, and smooth releases.

---

## P0 Incident Handling (Production Scenario)

### Incident

We faced a P0 incident where there was a sudden spike in **5xx errors** in a production application behind an Application Load Balancer.

### Detection

- Alerts triggered via Prometheus (latency + error rate)  
- Verified ALB metrics in CloudWatch  
- Identified that errors were from backend targets  

### Debugging Steps

1. Checked Kubernetes pods:
   - `kubectl get pods` → pods were running  
2. Checked logs:
   - `kubectl logs` → found DB connection timeout errors  
3. Verified database:
   - RDS instance hit **maximum connections limit**  

### Immediate Actions

- Increased DB connection limit temporarily  
- Restarted application pods to reset connections  
- Scaled application replicas to handle load  
- Enabled retries to reduce failure rate  

### Root Cause

A recent deployment introduced inefficient DB connection handling (no proper pooling), causing connection exhaustion under load.

### Prevention Measures

- Implemented proper DB connection pooling  
- Added alerts for DB connection threshold  
- Configured rate limiting on ALB  
- Included DB load testing in pre-release pipeline  

---

## Key Takeaways

- Strong CI/CD automation ensures faster releases  
- Monitoring and alerting are critical for incident response  
- Proper resource management prevents production issues  
- Automated rollback strategies reduce downtime  
- Continuous optimization is essential for scalability

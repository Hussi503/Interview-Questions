# 🚀 AWS DevOps Interview Questions

---

# 🌐 Networking & VPC

### 🔹 Q1. Explain the Serverless Services in AWS
 ### ✅ Answer
      Serverless in AWS means we don't manage servers, operating systems, patching, scaling, or capacity planning. AWS handles the infrastructure, and we only focus on writing code or building applications.

       In real projects, the most commonly used serverless services are AWS Lambda, API Gateway, DynamoDB, SQS, SNS, EventBridge, and Step Functions.

       For example, in a production environment, when a user uploads a file to an S3 bucket, an S3 event can trigger a Lambda function. The Lambda processes the file, stores metadata in DynamoDB, and sends notifications through SNS. 
       No EC2 instances are required, and AWS automatically scales based on demand.

       Another common use case is exposing REST APIs using API Gateway integrated with Lambda. This is useful for microservices where traffic is unpredictable because we only pay for actual requests instead of keeping servers running 24/7.

       From a DevOps perspective, serverless reduces infrastructure management overhead, improves scalability, and lowers costs for event-driven workloads. However, we need to consider limitations such as Lambda execution timeout, cold starts, 
       monitoring, and debugging distributed workflows.

       Overall, serverless is best suited for event-driven applications, automation tasks, APIs, data processing pipelines, and workloads with variable traffic patterns.

### 🔹 Q2. How do you ensure an application is highly available in AWS?
  ### ✅ Answer
        In production, high availability means eliminating single points of failure across every layer of the application. Typically, I deploy application servers in an Auto Scaling Group spread across at least two Availability Zones 
        and place them behind an Application Load Balancer. If one instance or even an entire Availability Zone goes down, the load balancer automatically routes traffic to healthy instances in the other AZ.

         For the database layer, I use Multi-AZ deployments, such as RDS Multi-AZ, so AWS automatically performs failover if the primary database becomes unavailable. Static content is stored in S3, which is highly durable by design, 
         and Route 53 is used for DNS-based routing and health checks. I also configure CloudWatch alarms and monitoring to proactively detect failures and trigger notifications.
    
         In one of my projects, Sitecore applications were deployed across multiple App Service instances with Azure-like HA concepts, but in AWS the equivalent approach would be ALB + Auto Scaling across multiple AZs with Multi-AZ databases. 
         This ensures that infrastructure failures, instance failures, or traffic spikes do not impact application availability.

         So, my approach is always to design HA at compute, load balancer, database, storage, and monitoring layers rather than relying on a single component to provide availability.
 

### 🔹 Q3. What is the difference between a Public Subnet and a Private Subnet?
   ### ✅ Answer
        A Public Subnet has a route to an Internet Gateway (IGW) in its route table, which allows resources inside that subnet to communicate directly with the internet. Typically, we place internet-facing components such as
        Application Load Balancers, Bastion Hosts, or public web servers in public subnets because they need to receive traffic from external users.

         A Private Subnet, on the other hand, does not have a direct route to the Internet Gateway. Resources inside private subnets cannot be accessed directly from the internet. We usually place backend application servers,
         databases, Redis clusters, internal services, and Kubernetes worker nodes in private subnets for security reasons. If these resources need outbound internet access for downloading patches, updates, or container images, 
         we route the traffic through a NAT Gateway deployed in a public subnet.

### 🔹 Q4. What is a VPC?
### ✅ Answer
      A VPC (Virtual Private Cloud) is a logically isolated network within AWS where we launch and manage our cloud resources such as EC2 instances, Load Balancers, RDS databases, EKS clusters, and other services. 
      It gives us complete control over networking, including IP address ranges, subnets, route tables, security groups, and network access.

      In real production environments, the first thing we design before deploying applications is the VPC architecture. For example, we create a VPC with a CIDR range like 10.0.0.0/16, divide it into public and private subnets
      across multiple Availability Zones, deploy Load Balancers in public subnets, application servers in private subnets, and databases in separate private subnets. This provides both security and high availability.

### 🔹 Q5. How many subnets can be created in a VPC?
### ✅ Answer

      AWS allows up to around 200 subnets per VPC by default, but practically the number depends on the VPC CIDR range and subnet design. In production, we create subnets based on architecture requirements rather than trying to reach the maximum limit.

### 🔹 Q6. How do Security Groups and NACLs affect EC2 connectivity?
### ✅ Answer

      Security Groups and NACLs both control network traffic to and from an EC2 instance, but they operate at different levels and serve different purposes.

      A Security Group acts as a virtual firewall at the instance level. It controls traffic entering and leaving the EC2 instance. Security Groups are stateful, which means if an inbound request is allowed,
      the response traffic is automatically allowed without needing an explicit outbound rule. In production, Security Groups are our primary mechanism for controlling access to EC2 instances.

      A NACL (Network Access Control List) operates at the subnet level and applies to all resources within that subnet. NACLs are stateless, meaning if you allow inbound traffic, you must also explicitly
      allow the corresponding outbound traffic. NACLs provide an additional layer of security and are often used for broader subnet-level restrictions.

      For example, if I am unable to SSH to an EC2 instance, I first verify that the Security Group allows inbound port 22 from my source IP. If that looks correct and connectivity still fails, 
      I check the subnet's NACL to ensure both inbound and outbound rules allow SSH traffic and ephemeral ports. Even if the Security Group allows traffic, a NACL deny rule can still block the connection.

### 🔹 Q7. What is the difference between blocking a CIDR using Security Groups vs NACLs?

### 🔹 Q8. Can CIDR blocking be enforced at the subnet level?

### 🔹 Q9. Why is OS-level firewall blocking not preferred in AWS?

### 🔹 Q10. How does NAT Gateway work?

### 🔹 Q11. How do you secure EC2 instances in a Private Subnet and allow access from outside?

### 🔹 Q12. Have you worked with VPC Endpoints? In what use case?

---

# 🔐 Security & IAM

### 🔹 Q13. What is a Security Group and why is it needed for EC2?

### 🔹 Q14. How will you block a particular CIDR block in AWS permanently?

### 🔹 Q15. When would you use AWS WAF instead of Security Groups?

### 🔹 Q16. How do you manage IAM at scale and audit AWS environments?

---

# 📊 Monitoring & CloudWatch

### 🔹 Q17. What are CloudWatch Metrics and how do you set up alerts in CloudWatch?

### 🔹 Q18. How do you enable Detailed Monitoring for EC2 instances?

---

# 🖥️ EC2 Administration & Troubleshooting

### 🔹 Q19. How do you resolve EC2 instance timeout errors? What are common reasons for SSH timeout?

### 🔹 Q20. What checks do you perform when an EC2 instance is running but not reachable?

### 🔹 Q21. How do ALB/NLB timeouts differ from EC2-level timeouts?

### 🔹 Q22. How do you increase the size of an EBS volume attached to EC2 without downtime?

### 🔹 Q23. What commands do you use after resizing an EBS volume?

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

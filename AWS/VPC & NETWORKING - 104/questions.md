# AWS / Networking / VPC / ECS / DevOps Interview Questions (Topic-wise)

---

# 🔷 Networking (OSI, Basics)

1. Can you explain the OSI model and its seven layers?  
2. What's the difference between the data link layer and transport layer?  
3. What is flow control and error control in the transport layer?  

---

# 🔷 VPC Fundamentals

### 4. What is an internet gateway?  
An Internet Gateway (IGW) is a highly available and scalable AWS-managed component that enables communication between a VPC
and the public internet. It acts as a bridge for inbound and outbound internet traffic for resources inside the VPC.

In a production setup, attaching an IGW to a VPC alone is not enough—we must configure route tables to direct traffic (typically 0.0.0.0/0) to the IGW. Only subnets associated with this route are considered public subnets. Additionally, resources like EC2 instances need a public IP or Elastic IP to communicate with the internet.

From a real-world perspective, IGW is used to expose only required components, such as load balancers or bastion hosts.
### 5. How can a private subnet access the internet? 

A private subnet cannot directly access the internet because it doesn’t have a route to an Internet Gateway and its resources don’t have public IPs. In production, we enable outbound internet access using a NAT Gateway (or NAT instance in older setups) placed in a public subnet.

The flow works like this: the private subnet’s route table points internet-bound traffic (0.0.0.0/0) to the NAT Gateway, and the NAT Gateway, which is associated with an Internet Gateway, forwards the request to the internet. The response is then routed back through the NAT Gateway to the private instance.
### 6. How would you connect your VPC to a customer's AWS account privately?  
To connect a VPC to a customer’s AWS account privately, we typically use VPC Peering or AWS PrivateLink, depending on the use case and scalability needs.

In simple setups, we use VPC Peering, which establishes a direct private connection between two VPCs using AWS’s internal network. Both VPCs exchange routes in their route tables, allowing instances to communicate using private IPs without traversing the internet. However, peering is one-to-one and doesn’t scale well for large environments.

In more secure and scalable production setups, we prefer AWS PrivateLink. Here, the service provider exposes a service via a Network Load Balancer, and the customer connects using an Interface Endpoint in their VPC. This ensures traffic stays completely private, avoids CIDR overlap issues, and provides better control and isolation.

In enterprise scenarios involving multiple VPCs and accounts, we may also use a Transit Gateway to centrally manage connectivity, but for strictly private and controlled cross-account access, PrivateLink is usually the preferred approach.
### 8. How does traffic flow from the internet to a private EC2 instance in a VPC?  
In a production setup, traffic from the internet never directly reaches a private EC2 instance

The request comes from the internet through the Internet Gateway (IGW) attached to the VPC.
It reaches a public resource, usually an Application Load Balancer (ALB) or Nginx/Reverse Proxy, which is placed in a public subnet with a public IP.
The load balancer then forwards the traffic to targets (private EC2 instances) in a private subnet using their private IPs.

From a routing perspective, the public subnet has a route to the IGW, while the private subnet has no direct internet route. Security groups ensure only the load balancer can access the private instances on required ports (like 80/443 or application ports).
### 9. What is the role of Internet Gateway and NAT Gateway? 
The Internet Gateway (IGW) and NAT Gateway serve different but complementary roles in enabling internet connectivity within a VPC.

The Internet Gateway is used to allow direct inbound and outbound internet access for resources in a public subnet. It acts as a bridge between the VPC and the internet, and resources must have a public or elastic IP along with a route (0.0.0.0/0) pointing to the IGW to communicate externally.

The NAT Gateway, on the other hand, is used to allow outbound-only internet access for resources in private subnets. It is deployed in a public subnet and is associated with an Internet Gateway. Private subnet traffic is routed to the NAT Gateway, which forwards the request to the internet and returns the response, without exposing the private instances to inbound internet traffic.

In production, we use IGW to expose only necessary components like load balancers or bastion hosts, while NAT Gateway ensures backend services in private subnets can still access external services securely. This combination helps maintain a secure, layered architecture with controlled internet exposure.

---

# 🔷 CIDR / Subnetting

# AWS VPC CIDR Interview Questions
**Production Grade | Easy to Remember | 5–6+ Years DevOps Engineer**

---

### 🔴 10. Can two subnets in the same VPC have overlapping CIDR blocks? Why?


No. Two subnets within the same VPC **cannot have overlapping CIDR blocks**. AWS enforces this because every IP address inside a VPC must be unique. If subnets overlap, AWS cannot determine where to route the network traffic.


### • Unique IP Range

- Every subnet must have its own unique CIDR block.
- No two subnets can share the same IP range.

### • Routing Issue

- If subnets overlap, AWS won't know which subnet should receive the traffic.
- This creates routing ambiguity.

### • AWS Validation

- AWS checks for overlapping CIDRs during subnet creation.
- It prevents the subnet from being created.

### • Production Planning

- Plan CIDR ranges in advance.
- Leave space for future subnets and expansion.

### • Real-Time Example

- While expanding a production VPC, we needed new private subnets for EKS worker nodes. Before creating them, we verified the existing CIDR ranges to ensure there was no overlap. Proper planning avoided deployment failures and future networking issues.

### Closing Line

> **"Every subnet inside a VPC must have a unique CIDR block because AWS routing depends on unique IP ranges."**

---

### 🔴 11. What does the error "CIDR address overlaps with existing subnet CIDR" mean?

This error means the CIDR block you're trying to assign to a new subnet is already being used, either fully or partially, by another subnet in the same VPC. AWS blocks this to prevent IP conflicts and routing problems.

### • Existing CIDR Conflict

- The new subnet overlaps with an existing subnet.
- Even a partial overlap is not allowed.

### • AWS Validation

- AWS validates CIDR ranges before creating the subnet.
- If overlap is detected, subnet creation fails.

### • Check Existing Subnets

- Review all subnet CIDRs in the VPC.
- Choose a non-overlapping range.

### • Production Practice

- Maintain a documented IP addressing plan.
- Reserve unused CIDR ranges for future growth.

### • Real-Time Example

- During a Terraform deployment, subnet creation failed with this error because another team had already created a subnet using part of the same IP range. We reviewed the VPC CIDRs, selected an unused range, updated Terraform, and the deployment completed successfully.

### Closing Line

> **"This error simply indicates an IP conflict. The solution is to choose a unique, non-overlapping CIDR block."**

---

### 🔴 12. How do you plan CIDR ranges for a production VPC?


In production, I don't allocate CIDR blocks randomly. I plan the VPC with future growth in mind by reserving enough IP addresses for application servers, databases, Kubernetes clusters, load balancers, and future expansion.


### • Start with a Large VPC

- Choose a CIDR like **10.0.0.0/16**.
- This provides enough IP space for future growth.

### • Separate by Tier

- Public subnets for Load Balancers.
- Private subnets for Applications.
- Separate private subnets for Databases.
- Dedicated subnets for EKS or other services if required.

### • Plan for Expansion

- Leave unused CIDR ranges.
- Avoid consuming all IP space on day one.

### • Keep It Consistent

- Follow a standard subnet naming and IP allocation pattern across environments.
- Makes troubleshooting and automation easier.

### • Real-Time Example

- In one project, we designed the VPC with a **/16 CIDR** and divided it into separate public, application, and database subnets across three Availability Zones. We also reserved additional CIDR space for future EKS clusters. A year later, when new services were introduced, we added subnets without modifying the existing network.

### Closing Line

> **"Good CIDR planning isn't just about today's requirement—it's about ensuring the network can scale without redesigning the VPC later."**

---

# 🔷 VPC Architecture

# AWS VPC Interview Questions
**Production Grade | Easy to Remember | 5–6+ Years DevOps Engineer**

---

### 🔴 13. Why is a Custom VPC preferred over the Default VPC?


In production, I always prefer a **Custom VPC** because it gives complete control over networking and security. The Default VPC is mainly meant for quick testing or learning, whereas a Custom VPC allows us to design the network according to business and security requirements.


### • Better Security

- Create separate Public and Private subnets.
- Keep databases and application servers away from the internet.

### • Network Control

- Design our own CIDR ranges.
- Create subnets based on application requirements.

### • Better Access Control

- Configure Route Tables, NACLs, and Security Groups as per security standards.
- Control traffic between different application tiers.

### • Scalability

- Easily add new subnets for EKS, databases, or future applications.
- Better suited for growing environments.

### • Real-Time Example

- In one project, we created a Custom VPC with separate Public, Application, and Database subnets across three Availability Zones. Only the Load Balancer was internet-facing, while application servers and databases remained in private subnets, improving security and availability.

### Closing Line

> **"The Default VPC is suitable for testing, but for production, I always use a Custom VPC because it provides better security, flexibility, and scalability."**

---

### 🔴 14. Explain a Production-Ready VPC Architecture.


A production-ready VPC is designed with **High Availability, Security, and Scalability** in mind. Instead of placing all resources in one subnet, we separate them into different layers and distribute them across multiple Availability Zones to eliminate single points of failure.


### • Public Layer

- Internet Gateway
- Application Load Balancer
- NAT Gateway
- Bastion Host (if required)

### • Private Application Layer

- EC2 instances
- ECS/EKS worker nodes
- Auto Scaling Groups

### • Private Database Layer

- RDS Multi-AZ
- Database EC2
- ElastiCache

### • Security

- Security Groups
- Network ACLs
- Least Privilege Access

### • High Availability

- Deploy resources across at least 2–3 Availability Zones.
- Use Auto Scaling and Load Balancers.

### • Real-Time Example

- In one production project, we designed a VPC with three Availability Zones. The ALB was deployed in public subnets, application servers ran in private subnets behind an Auto Scaling Group, and the database was deployed as RDS Multi-AZ in private database subnets. NAT Gateways allowed outbound internet access without exposing private resources.

### Closing Line

> **"A production VPC should isolate each application tier, eliminate single points of failure, and provide secure communication between components."**

---

# 🔴 76. From an architecture point of view, what does Three-Tier Architecture mean in VPC?


Three-tier architecture means separating the application into **Presentation**, **Application**, and **Database** layers. Each layer is deployed in separate subnets and communicates only with the layer it needs, improving security, scalability, and maintenance.


### • Presentation Tier

- Public Subnets
- Application Load Balancer
- Receives user traffic from the internet.

### • Application Tier

- Private Subnets
- EC2, ECS, or EKS
- Processes business logic.
- Receives traffic only from the Load Balancer.

### • Database Tier

- Private Database Subnets
- RDS or Database on EC2
- Accessible only from the application tier.

### • Benefits

- Better Security
- Independent Scaling
- Easier Maintenance
- High Availability

### • Real-Time Example

- In one project, users accessed the application through an ALB in the public subnet. The ALB forwarded requests to EC2 instances running in private subnets, and those application servers connected to an RDS database deployed in private database subnets. The database was never exposed to the internet, making the architecture secure and production-ready.

### Closing Line

> **"The idea behind three-tier architecture is simple—users can access only the presentation layer, the application communicates with the database, and each layer is isolated for better security and scalability."**


---

### 🔴 77. What are the main components of Three-Tier Architecture?


Three-tier architecture is a design pattern where an application is divided into **three separate layers**: **Presentation Layer**, **Application Layer**, and **Database Layer**. Each layer has its own responsibility, making the application more secure, scalable, and easier to maintain.


### • Presentation Layer (Frontend)

- Handles user requests.
- Usually consists of **Application Load Balancer (ALB)**, web server, or frontend application.
- This is the only layer users access directly.

### • Application Layer

- Contains the business logic.
- Runs on **EC2, ECS, or EKS**.
- Processes user requests and communicates with the database.

### • Database Layer

- Stores application data.
- Uses **Amazon RDS** or a database running on EC2.
- Accessible only from the application layer.

### • Benefits

- Better Security
- Independent Scaling
- Easier Maintenance
- High Availability

### • Real-Time Example

- In one project, users accessed the application through an ALB. The request was forwarded to application servers running on EKS, which processed the request and fetched data from an RDS database deployed in private subnets.

### Closing Line

> **"Three-tier architecture separates responsibilities, making the application easier to secure, scale, and manage."**

---

### 🔴 78. In AWS VPC, where will each layer (Frontend, App, DB) belong? Which Subnet?


In a production VPC, each layer is placed in a different subnet based on its purpose. Only the frontend needs internet access, while the application and database layers remain private for security.


### • Frontend Layer

- **Public Subnet**
- Hosts the **Application Load Balancer (ALB)**.
- Receives traffic from internet users.

### • Application Layer

- **Private Subnet**
- Runs EC2, ECS, or EKS workloads.
- Receives traffic only from the Load Balancer.

### • Database Layer

- **Private Database Subnet**
- Hosts RDS or database on EC2.
- Accessible only from the application layer.

### • Why This Design?

- Prevents direct internet access to application and database servers.
- Improves security and follows AWS best practices.

### • Real-Time Example

- In our production environment, the ALB was deployed in public subnets, EKS worker nodes were in private subnets, and the RDS database was in dedicated private database subnets. Users could only reach the ALB, while the backend resources remained protected.

### Closing Line

> **"Only the entry point is public. The application and database always stay in private subnets to reduce the attack surface."**

---

### 🔴 79. Which layer will you put in a Public Subnet?

In production, **only the Presentation Layer** is placed in a **public subnet** because it needs to receive requests from internet users. The application and database layers always remain in private subnets.

### • Public Subnet

- Application Load Balancer (ALB)
- Internet Gateway
- NAT Gateway (for outbound internet access from private subnets)

### • Private Subnet

- EC2 Application Servers
- ECS Services
- EKS Worker Nodes

### • Private Database Subnet

- Amazon RDS
- MySQL/PostgreSQL on EC2
- ElastiCache

### • Why?

- Users should never directly access application servers or databases.
- The Load Balancer acts as the secure entry point.
- This reduces security risks and follows AWS best practices.

### • Real-Time Example

- In one project, customers accessed the application through the ALB in the public subnet. The ALB forwarded requests to EKS pods running in private subnets, and the pods accessed an RDS database in private database subnets. Neither the application servers nor the database had public IPs.

### Closing Line

> **"Only the Load Balancer belongs in the public subnet. Application servers and databases should always remain private in a production environment."**
80. How will the frontend communicate with app and DB layers if they're in private subnets?  
81. If hosting a public website, how will user requests route to the DB server?  

---

# 🔷 Load Balancers (ALB / NLB)



---

### 🔴 7. What are the types of Load Balancers in AWS?


AWS provides multiple types of Load Balancers, but in production I mostly work with **Application Load Balancer (ALB)** and **Network Load Balancer (NLB)**. The choice depends on the application protocol and traffic requirements.


### • Application Load Balancer (ALB)

- Works with HTTP and HTTPS traffic.
- Supports path-based and host-based routing.
- Commonly used for web applications, APIs, ECS, and EKS.

### • Network Load Balancer (NLB)

- Works with TCP, UDP, and TLS traffic.
- Designed for very high performance and low latency.
- Commonly used for databases, gaming, and real-time applications.

### • Gateway Load Balancer (GWLB)

- Used with virtual firewalls and security appliances.
- Helps inspect and filter network traffic.

### • Classic Load Balancer (CLB)

- Older generation Load Balancer.
- Mostly used for legacy applications.
- AWS recommends using ALB or NLB for new deployments.

### • Real-Time Example

- In one project, our frontend application was exposed through an **Application Load Balancer** because we needed path-based routing for multiple microservices. Another project used an **NLB** because the application required TCP traffic with very low latency.

### Closing Line

> **"In my day-to-day work, ALB is my default choice for web applications, while NLB is used when the application requires TCP/UDP traffic or very high performance."**

---

### 🔴 15. At which OSI layer does ALB work? NLB?


The main difference is the protocol they understand.

- **Application Load Balancer (ALB)** works at **Layer 7 (Application Layer)** because it understands HTTP and HTTPS requests.
- **Network Load Balancer (NLB)** works at **Layer 4 (Transport Layer)** because it forwards traffic based on TCP or UDP without inspecting the application data.


### • ALB (Layer 7)

- HTTP / HTTPS
- Path-based routing
- Host-based routing
- SSL Termination
- Web applications & APIs

### • NLB (Layer 4)

- TCP / UDP / TLS
- High throughput
- Low latency
- Static IP support
- Suitable for non-HTTP traffic

### • Real-Time Example

- We used **ALB** in front of our EKS cluster because different URLs were routed to different microservices.
- We used **NLB** for an application that communicated over TCP and required very low latency.

### Closing Line

> **"A simple way to remember it is—ALB understands HTTP requests, whereas NLB only forwards network packets."**

---

### 🔴 16. Where exactly is an ALB placed inside a VPC?


In a production environment, the **Application Load Balancer is deployed in Public Subnets** across multiple Availability Zones. It acts as the entry point for internet traffic and forwards requests to application servers running in private subnets.


### • Public Subnets

- ALB is deployed in at least **two Public Subnets**.
- Connected to an Internet Gateway.

### • Receives User Traffic

- Internet users connect to the ALB.
- ALB performs health checks and distributes traffic.

### • Forwards to Private Subnets

- EC2, ECS, or EKS workloads are deployed in Private Subnets.
- Backend servers don't require public IPs.

### • High Availability

- ALB spans multiple Availability Zones.
- Continues serving traffic even if one AZ fails.

### • Real-Time Example

- In one production project, the ALB was deployed in two public subnets across different Availability Zones. It received HTTPS traffic from users and routed requests to EKS worker nodes running in private subnets. The backend infrastructure remained completely isolated from the internet.

### Closing Line

> **"The ALB is the only public-facing component. Application servers remain in private subnets, making the architecture secure and highly available."**
# AWS Load Balancer Interview Questions
**Production Grade | Easy to Remember | 5–6+ Years DevOps Engineer**

---

### 🔴 17. What is the difference between Internet-facing ALB and Internal ALB?


The main difference is **who can access the Load Balancer**. An **Internet-facing ALB** accepts traffic from the internet, whereas an **Internal ALB** accepts traffic only from within the VPC. The choice depends on whether the application is public-facing or internal.


### • Internet-facing ALB

- Deployed in **Public Subnets**.
- Has a public DNS name.
- Used for websites, mobile apps, and public APIs.
- Receives traffic from internet users.

### • Internal ALB

- Deployed in **Private Subnets**.
- Accessible only within the VPC or connected networks.
- Used for internal microservices and backend APIs.
- Not reachable from the internet.

### • Real-Time Example

- In one microservices project, customers accessed the application through an **Internet-facing ALB**. The frontend service then communicated with backend services through an **Internal ALB**, keeping internal APIs hidden from external users.

### Closing Line

> **"If users need internet access, I use an Internet-facing ALB. If communication is only between internal services, I use an Internal ALB."**

---

### 🔴 18. When would you choose NLB over ALB?

I choose **NLB** when the application requires **TCP/UDP traffic, very high performance, low latency, or static IP addresses**. For web applications using HTTP/HTTPS, I prefer **ALB** because it provides advanced routing features.


### • Use NLB When

- Application uses **TCP, UDP, or TLS**.
- Very low latency is required.
- High throughput is expected.
- Static IP or Elastic IP is needed.

### • Use ALB When

- Application uses HTTP or HTTPS.
- Need path-based routing.
- Need host-based routing.
- Running web applications or REST APIs.

### • Real-Time Example

- In one project, our web application used an **ALB** because requests had to be routed to different microservices based on the URL path. Another application used an **NLB** because it handled TCP traffic and required very low network latency.

### Closing Line

> **"If the application understands HTTP, I choose ALB. If it works at the TCP/UDP level or needs maximum performance, I choose NLB."**

---

### 🔴 19. Does NLB support Security Groups? Why or why not?


Traditionally, **Network Load Balancers did not support Security Groups** because they operate at **Layer 4** and simply forward TCP/UDP traffic. Security was enforced on the backend resources like EC2 instances. **However, AWS now supports attaching Security Groups to NLBs**, but only when the NLB is associated with a VPC. This provides an additional layer of traffic filtering.

### Points to Cover

### • Layer 4 Load Balancer

- NLB works at the Transport Layer.
- Optimized for speed and low latency.

### • Security

- Earlier, Security Groups were applied only to backend EC2 instances.
- Now, Security Groups can also be attached to VPC-based NLBs.
- Backend Security Groups are still required.

### • Why Backend Security Is Important

- Protects application servers even if the Load Balancer configuration changes.
- Ensures only required traffic reaches the instances.

### • Real-Time Example

- In one production environment, we used an NLB for TCP traffic. We restricted access using Security Groups on the backend EC2 instances and later attached Security Groups to the NLB as an additional security layer after AWS introduced the feature.

### Closing Line

> **"Even though NLB now supports Security Groups, I still secure the backend resources because defense in depth is a production best practice."**
# AWS Load Balancer Interview Questions
**Production Grade | Easy to Remember | 5–6+ Years DevOps Engineer**

---

### 🔴 21. Can ALB targets be in Private Subnets?

Yes. In fact, **this is the recommended production architecture**. The ALB is deployed in **public subnets** to receive internet traffic, while its targets (EC2, ECS tasks, or EKS pods) are deployed in **private subnets**. This keeps the backend servers secure because they are not directly accessible from the internet.


### • ALB in Public Subnets

- Receives HTTP/HTTPS requests from users.
- Connected to the Internet Gateway.

### • Targets in Private Subnets

- EC2 instances
- ECS Tasks
- EKS Pods (via Target Groups)

### • Traffic Flow

- User sends request to ALB.
- ALB performs health checks.
- ALB forwards the request to healthy targets in private subnets.
- Backend sends the response through the ALB.

### • Why This Design?

- Backend servers don't need public IPs.
- Reduces the attack surface.
- Follows AWS security best practices.

### • Real-Time Example

- In one production project, our ALB was deployed in two public subnets across different Availability Zones. It forwarded HTTPS traffic to EKS worker nodes running in private subnets. Users could access only the ALB, while the application servers remained completely private.

### Closing Line

> **"Yes, and that's actually the recommended production design. Only the ALB is public—the backend servers should always remain in private subnets."**

---

### 🔴 22. Explain Client IP handling in ALB vs NLB.


The main difference is **how the backend application receives the client's IP address**.

- **ALB** works at **Layer 7**, so it forwards the original client IP using the **X-Forwarded-For** HTTP header.
- **NLB** works at **Layer 4**, so it preserves the original client IP at the network level and forwards it directly to the backend.

### • ALB

- Layer 7 (HTTP/HTTPS)
- Adds the **X-Forwarded-For** header.
- Backend applications read this header to identify the real client IP.

### • NLB

- Layer 4 (TCP/UDP)
- Preserves the original client IP automatically.
- No HTTP header is required.

### • Why Client IP Matters

- Application logging
- Security auditing
- Rate limiting
- IP-based access control
- Troubleshooting

### • Real-Time Example

- In one project, we used an ALB for our web application. Since traffic passed through the ALB, our application read the **X-Forwarded-For** header to log the actual client IP instead of the ALB's IP. In another project using an NLB for TCP traffic, the backend automatically received the original client IP without additional configuration.

### Closing Line

> **"ALB passes the client IP through the X-Forwarded-For header, whereas NLB preserves the original client IP at the network layer."**
### 🔴 23. ALB is up but the application is not reachable — how do you troubleshoot?


If the **ALB is healthy but the application isn't reachable**, I troubleshoot layer by layer instead of making assumptions. I start from the Load Balancer and move towards the application. This helps identify whether the issue is with networking, security, target health, or the application itself.


### • Check Target Health

- Verify whether the targets are **Healthy** or **Unhealthy**.
- If targets are unhealthy, check the health check path, port, and application status.

### • Verify the Application

- SSH into the EC2 instance (or check the container).
- Ensure the application/service is running.
- Confirm it's listening on the correct port.

### • Check Security Groups

- ALB Security Group should allow HTTP/HTTPS from users.
- EC2 Security Group should allow traffic **only from the ALB Security Group**.

### • Verify Health Check Configuration

- Confirm the health check URL (for example, `/health`).
- Ensure the application returns **HTTP 200 OK**.
- Check health check interval, timeout, and port.

### • Check Network Configuration

- Verify Route Tables and Network ACLs.
- Ensure the ALB can reach the private subnet where the application is running.

### • Check Logs

- Review application logs.
- Check ALB access logs (if enabled).
- Review CloudWatch metrics and logs for errors.

### • Real-Time Example

- In one production deployment, the ALB was healthy, but all targets were showing **Unhealthy**. After checking the health check configuration, we found the ALB was calling **/** while the application exposed **/health**. After updating the health check path, the targets became healthy, and the application started serving traffic immediately.

### Closing Line

> **"I always troubleshoot from the Load Balancer to the application—Target Health, Application, Security Groups, Health Checks, Network, and Logs. This systematic approach helps identify the root cause quickly."**
---

# 🔷 Kubernetes / EKS

20. How does ALB integrate with EKS?  
26. For deploying a Kubernetes cluster in AWS, what things do you consider?  
27. If an architect gave a design (1 master, 3 workers), how would you implement it?  

---

# 🔷 IAM / Roles

28. What roles are defined in AWS in your organization? Hierarchy level?  
29. Apart from Administrator and Read-only, what other roles do you define?  
30. What kind of permissions does Power User have?  
31. In cloud terminology, what is the role name for modifying architecture?  

---

# 🔷 AWS Services (General)

32. Do you work on AWS Redshift, SNS, SQS, Glue, Kinesis?  
33. What storage services have you worked on?  
34. If SNS is there, why should we use Prometheus and Grafana?  

---

# 🔷 ECS (Containers)

35. What does it mean to run an ECS service in a private subnet?  
36. How do you move an ECS service from public to private subnets?  
37. What happens if ECS tasks in private subnets don't have NAT Gateway access?  
38. Why must an internet-facing ALB be replaced with an internal ALB when making ECS private?  
39. Can we convert an existing public ALB to internal? Why or why not?  
40. How do ECS tasks pull images from ECR when running in private subnets?  
41. What security group changes are required when converting ECS from public to private?  
42. How do you expose a private ECS service to on-prem users?  
43. What is the role of Cloud Map in private ECS services?  
44. How would you validate that an ECS service is fully private after migration?  

---

# 🔷 VPC Peering

45. What is VPC peering, and why is it used? How does it differ from VPC endpoints?  
46. Can VPC peering work across AWS accounts and regions?  
47. Why are overlapping CIDR blocks not allowed in VPC peering?  
48. What route table changes are required for VPC peering?  
49. Do security groups automatically allow traffic over VPC peering?  
50. Can you use security group references across peered VPCs?  
51. What happens if routing is correct but traffic is still blocked?  
52. Why is DNS resolution important in VPC peering?  
53. How do private Route53 records work across peered VPCs?  
54. What does "VPC peering is non-transitive" mean?  
55. Can you use VPC peering as a hub-and-spoke model?  
56. Why is VPC peering not recommended for large-scale architectures?  

---

# 🔷 Transit Gateway

57. When should Transit Gateway be used instead of VPC peering?  
73. What AWS service replaces large-scale VPC peering?  
91. Suppose multiple VPCs across multiple accounts (e.g., 20). How to enable communication?  
92. How many transit gateways are required? What other components are needed?  
93. How will you share a transit gateway with other accounts?  
94. Can you walk through the transit gateway setup?  
98. What is a Transit Gateway, and how does it help with VPC communication?  
99. What is the difference between VPC Peering and Transit Gateway?  

---

# 🔷 Troubleshooting Scenarios

24. An AWS region goes down — how do you recover your application?  
25. CloudWatch shows high CPU but low traffic — what could be the issue?  
58. ECS tasks in private subnets are unhealthy after migration — how do you troubleshoot?  
59. An ECS service cannot pull images from ECR after moving to a private subnet — why?  
60. An application in VPC-A cannot access RDS in VPC-B via peering — list your checks.  
61. Route tables are updated but traffic still fails over VPC peering — why?  
75. Can you explain a critical VPC issue you resolved (like peering issues)?  

---

# 🔷 Multi-VPC / Security Design

62. How would you design secure communication between ECS in one VPC and RDS in another?  
63. A monitoring VPC needs access to multiple application VPCs — peering or Transit Gateway?  
64. How would you design a fully private ECS architecture across multiple VPCs?  
65. What are the security risks of VPC peering?  
66. How do you audit and monitor traffic flowing over VPC peering?  
67. Can NACLs block traffic over VPC peering? Explain.  
68. How does VPC peering impact latency and bandwidth?  
69. Is VPC peering encrypted?  
70. Can ECS tasks in a private subnet have public IPs?  
71. Can we attach a NAT Gateway to a private subnet?  
72. Can one VPC peer with multiple VPCs?  
95. What are the pre-requisites for VPC peering between two VPCs?  
96. What problems occur when two VPCs have overlapping CIDR blocks?  
97. How can you enable communication between overlapping CIDR VPCs?  
100. How can a jump server (bastion host) be used in overlapping network scenarios?  
101. Can you explain transitive routing between VPCs A, B, and C?  

---

# 🔷 DNS / Route53 / Cluster Networking

83. Do you work on Route53? What is a hosted zone?  
84. In Route53, if records are not loading for a cluster, where will you verify?  
85. What is CoreDNS? Why does it point to a particular IP?  
86. Which IP should be exposed outside for a cluster?  
87. In which namespace does CoreDNS run?  
88. If it's an internal cluster, why expose a public IP? Where should the exposed IP be defined?  
89. What kind of routing policy have you set up in Route53?  
90. How many Route53 hosted zones in your project — single or multiple? Why multiple?  

---

# 🔷 Connectivity (On-prem / Hybrid)

102. How do you connect on-premises infrastructure to a VPC?  
103. What is the difference between VPC Peering and Direct Connect?  

---

# 🔷 Misc

74. Have you been involved in creating VPCs, subnets, route tables, and gateways?  
104. What is a Route Table, and how does it differ from Auto Scaling?  

---

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

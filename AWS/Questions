CGI Questions:
---------------------------------------------------------------------------------------
1.Explain the complete request flow from a user accessing an application hosted in AWS.

User opens:   https://myapp.com
Flow:
User ->Route 53 -> cloudfront -> AWS WAF -> Application Load Balancer -> EC2 / ECS / EKS -> RDS Database

1. User sends request
   User browser says: "I want to open myapp.com"
2. Route 53 (DNS)
   Route 53 finds where the application is running.
      Example:
         myapp.com  ---> ALB address
        It sends the user request to Load Balancer.
3. CloudFront (CDN)
   CloudFront checks:
  "Do I already have this content cached?"
  If yes:
    CloudFront ---> User
      Fast response.
    If no:
     CloudFront ---> ALB
4. AWS WAF (Security)
  Before reaching application: WAF checks:
                                    Is this a normal request?
                                    Is it an attack?
                                    Too many requests?
                        Example:
                           Normal request  ---> Allow
                           Attack request  ---> Block
5. Application Load Balancer (ALB)
       ALB receives the request.
      It checks:
            Which application?
            Which server is free?
            Is the server healthy?
          Example:

          ALB

       /    |    \
      /     |     \
   EC2-1  EC2-2  EC2-3

It sends traffic to one healthy server.

6. Application Server (EC2/ECS/EKS)
   Your application runs here.
    Example:
      User asks:
            Show my orders
            Application says:
            "I need data from database"

7. Database (RDS)
     Application connects to RDS:   Application  ---> RDS
     Database returns the data.
8. Response comes back
   Same way back: RDS--> Application --> ALB --> CloudFront -->  User
    User sees the page
*******************************************************************************************************************************************************************************************************************************************************
2. Difference between Security Groups and NACLs?
   Security Groups operate at the instance or ENI level and are stateful, meaning return traffic is automatically allowed. NACLs operate at subnet level and are stateless, meaning inbound and outbound traffic must be explicitly allowed. 
   Security Groups only support allow rules, while NACLs support both allow and deny rules.
*********************************************************************************************************************************************************************************************************************************************************
3. How does a NAT Gateway work internally?
4. How would you design a highly available architecture across multiple Availability Zones?
Difference between ALB, NLB, and CLB?
How do you provide cross-account access in AWS?
What happens when an EC2 instance in an Auto Scaling Group becomes unhealthy?
How would you troubleshoot connectivity issues between private and public subnets?
How does EKS integrate with IAM?
How would you optimize AWS infrastructure costs?

------------------------------------------------------------------------------------
CapGemini
------------------------------------------------------------------------------------
Explain your current AWS architecture end-to-end.
How do you design a highly available application in AWS?
Explain the complete request flow from Route 53 → ALB → EKS/EC2.
How do you secure workloads running in AWS?
Difference between Security Groups and NACLs with real use cases.
How do you troubleshoot high CPU utilization on an EC2 instance?
How have you implemented Auto Scaling in your project?
-------------------------------------------------------------------------------------

Wipro
--------------------------------------------------------------------------------------


What is an application gateway and load balancer?
Suppose you have deployed a web application on an EC2 instance inside a VPC. The instance is running and health checks are passing, but users are unable to access the application from the internet. What will you do?
In that case, what components in the VPC would you verify?
Suppose an EC2 instance is showing high CPU utilization (above 90%) in CloudWatch. How will you identify the root cause?
----------------------------------------------------------------------------------------------------------------------------

1. How do you integrate AWS with Jenkins?
2. Why is IAM role better than access keys?
3. What permissions are required for Jenkins to create EC2/VPC?
4. How do you test AWS access from Jenkins?
5.How does Jenkins deploy infrastructure using Terraform?
6. Suppose you have high CPU (60%), high memory (70%), and some network traffic. What else will you check before stopping an EC2 instance?
7. How will you check if the instance is behind a load balancer or part of an autoscaling group?
8. How do you check which processes or services are running on that instance? Which commands do you use?
9. How do you check the active ports on that instance?
10. How do you migrate objects from one S3 bucket to another?
11. If an instance is stopped, how do you check in AWS Cloud which instance was stopped and when?



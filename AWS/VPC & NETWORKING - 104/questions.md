# AWS / Networking / VPC / ECS / DevOps Interview Questions (Topic-wise)

---

# 🔷 Networking (OSI, Basics)

1. Can you explain the OSI model and its seven layers?  
2. What's the difference between the data link layer and transport layer?  
3. What is flow control and error control in the transport layer?  

---

# 🔷 VPC Fundamentals

4. What is an internet gateway?  
5. How can a private subnet access the internet?  
6. How would you connect your VPC to a customer's AWS account privately?  
8. How does traffic flow from the internet to a private EC2 instance in a VPC?  
9. What is the role of Internet Gateway and NAT Gateway?  

---

# 🔷 CIDR / Subnetting

10. Can two subnets in the same VPC have overlapping CIDR blocks? Why?  
11. What does error "CIDR address overlaps with existing subnet CIDR" mean?  
12. How do you plan CIDR ranges for a production VPC?  

---

# 🔷 VPC Architecture

13. Why is a custom VPC preferred over the default VPC?  
14. Explain a production-ready VPC architecture.  
76. From an architecture point of view, what does three-tier architecture mean in VPC?  
77. What are the main components of three-tier architecture?  
78. In AWS VPC, where will each layer (frontend, app, DB) belong? Which subnet?  
79. Which layer will you put in a public subnet?  
80. How will the frontend communicate with app and DB layers if they're in private subnets?  
81. If hosting a public website, how will user requests route to the DB server?  

---

# 🔷 Load Balancers (ALB / NLB)

7. What are the types of load balancers in AWS?  
15. At which OSI layer does ALB work? NLB?  
16. Where exactly is an ALB placed inside a VPC?  
17. What is the difference between internet-facing ALB and internal ALB?  
18. When would you choose NLB over ALB?  
19. Does NLB support security groups? Why or why not?  
21. Can ALB targets be in private subnets?  
22. Explain client IP handling in ALB vs NLB.  
23. ALB is up but the application is not reachable — how do you troubleshoot?  

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

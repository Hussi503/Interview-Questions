CGI
**What is Terraform State and why is it important?**

Ans:Terraform state is a file where Terraform stores the current information about the infrastructure it manages. It maintains the mapping between Terraform configuration files and the real resources deployed in cloud providers like Azure or AWS.
The state file contains resource IDs, attributes, dependencies, and outputs. Terraform uses this state during plan and apply operations to compare the desired configuration with the actual infrastructure and identify required changes.
It is important because Terraform is declarative. The state helps Terraform understand what resources already exist, detect configuration drift, and perform incremental changes instead of recreating everything.
In production environments, we usually store the state remotely using backends like Azure Storage Account with state locking enabled, so multiple team members can work safely without state corruption.

What happens if the Terraform State file gets corrupted?
How do you recover from state file issues?
Difference between count and for_each?
Explain Terraform backend and state locking.
Why do we use DynamoDB with Terraform?
What are Terraform modules and how do you structure them?
How does Terraform identify infrastructure drift?
What is Terraform Import and when would you use it?
How would you manage Terraform code for multiple environments?

----------------------------------------------------------------
CapGemini
----------------------------------------------------------------

Explain your Terraform project structure.
How do you manage Terraform state files?
Why do we use remote backends?
Explain state locking and its importance.
How do you handle multiple environments using Terraform?
What are Terraform modules and how have you used them?
----------------------------------------------------------------

Wipro
----------------------------------------------------------
Do you have experience in creating infrastructure with Terraform?
What are the services you created using Terraform?
May I know what is a Terraform state file?
Where do you store the Terraform state file?
What is provider.tf?
What is variable.tf?
Suppose while executing terraform plan it shows that some resources are being deleted — what will be your next plan of action?
-----------------------------------------------------------------------------------------------------------------------------------


1. How does Terraform state actually work, and why is it critical in production?
2. What happens if two people run 𝐭𝐞𝐫𝐫𝐚𝐟𝐨𝐫𝐦 𝐚𝐩𝐩𝐥𝐲 at the same time? How do you prevent it?
3. Explain a real scenario where Terraform wanted to recreate production resources-and how you avoided downtime.
4. Difference between 𝐜𝐨𝐮𝐧𝐭 and 𝐟𝐨𝐫_𝐞𝐚𝐜𝐡, and why changing between them can destroy resources.
5. How do you manage Terraform state securely across multiple teams and environments?
6. What is infrastructure drift, and how do you detect and fix it safely?
7. How do you refactor Terraform code without impacting live infrastructure?
8. What happens when a resource is deleted manually from the cloud but still exists in Terraform state?
9. How do Terraform modules help at scale, and what mistakes make them hard to reuse?
10. Explain implicit vs explicit dependencies. When does Terraform get it wrong?
11. How do you handle secrets in Terraform without exposing them in state files?
12. What are Terraform workspaces, and why are they risky in large organizations?
13. How do you recover from a failed or partial 𝐭𝐞𝐫𝐫𝐚𝐟𝐨𝐫𝐦 𝐚𝐩𝐩𝐥𝐲?
14. How do provider version mismatches break infrastructure, and how do you prevent this?
15. Tell me about a real Terraform incident you faced and how you fixed it.
16. Terraform apply failed – how do you troubleshoot?
17. What is a Terraform state lock error?
18. How do you fix a corrupted Terraform state?
19. What is terraform force-unlock?
20. What causes CIDR overlap errors?
21. How do you handle “resource already exists” errors?
22. What causes provider version conflicts?
23. How do you create multiple identical EC2 instances in Terraform?
24. Difference between count and for_each?
25. When should you use count?
26. When should you use for_each?
27. How do you create EC2 instances with different sizes?
28. How do you create 5 identical EC2 instances?
29. How do you tag EC2 instances dynamically?


 2. Terraform – Core Concepts
1. What is Terraform and why do we use it?
2. What are Terraform providers?
3. What are Terraform provisioners?
4. Difference between providers and provisioners?
5. Why are provisioners not recommended in production?
6. What is terraform init, plan, and apply?
7. What happens internally during terraform apply?

🟦 3. Terraform Versioning
1. Why do we lock Terraform versions?
2. What is required_version?
3. How do you lock provider versions?
4. What does ~> 5.0 mean in provider versioning?
5. What problems occur if versions are not pinned?
6. How do you upgrade Terraform safely?

🟦 4. Terraform Errors & Failures
1. Terraform apply failed – how do you troubleshoot?
2. What is a Terraform state lock error?
3. How do you fix a corrupted Terraform state?
4. What is terraform force-unlock?
5. What causes CIDR overlap errors?
6. How do you handle “resource already exists” errors?
7. What causes provider version conflicts?

🟦 5. Terraform + EC2 Scenarios
1. How do you create multiple identical EC2 instances in Terraform?
2. Difference between count and for_each?
3. When should you use count?
4. When should you use for_each?
5. How do you create EC2 instances with different sizes?
6. How do you create 5 identical EC2 instances?
7. How do you tag EC2 instances dynamically?
8. Why is Auto Scaling preferred over multiple EC2 instances?


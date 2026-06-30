CGI

## 1.**What is Terraform State and why is it important?**

Ans:Terraform state is a file where Terraform stores the current information about the infrastructure it manages. It maintains the mapping between Terraform configuration files and the real resources deployed in cloud providers like Azure or AWS.
The state file contains resource IDs, attributes, dependencies, and outputs. Terraform uses this state during plan and apply operations to compare the desired configuration with the actual infrastructure and identify required changes.
It is important because Terraform is declarative. The state helps Terraform understand what resources already exist, detect configuration drift, and perform incremental changes instead of recreating everything.
In production environments, we usually store the state remotely using backends like Azure Storage Account with state locking enabled, so multiple team members can work safely without state corruption.

## 2.   What happens if the Terraform State file gets corrupted?

ans:Terraform state corruption means Terraform lost or has invalid tracking information about the infrastructure. The actual cloud resources are not deleted; only the state mapping is affected.
    First, we recover the state from backups such as S3 bucket versioning or Azure Blob versioning. After restoring the state, we run terraform plan to validate the state.
    If no backup is available, we recreate the state by using terraform import to bring existing cloud resources back under Terraform management.
    To prevent this, we use remote backend, versioning, encryption, access control, and state locking.
## 3. How do you recover from state file issues?
Ans: Terraform state issues can happen due to accidental deletion, corruption, manual modification, or state lock problems. First, I identify the issue type.
     If the state file is deleted or corrupted, the actual cloud resources are not deleted; only Terraform's tracking information is lost.
     First approach is to recover the state from the remote backend backup. In production, we store state remotely using S3/Azure Storage with versioning enabled, so we restore the previous working state version.
     After restoring, I validate using terraform state list and terraform plan to make sure Terraform is correctly matching the existing infrastructure.
     If no backup is available, I recreate the state by importing existing resources using terraform import, so Terraform can manage those resources again without recreating them.
## 4.    Difference between count and for_each?
ans: Both count and for_each are Terraform meta-arguments used to create multiple resources. The main difference is how Terraform identifies each resource.
     count uses a numeric index. Resources are created with indexes starting from 0.
     Example:
             resource "aws_instance" "server" {
                   count = 3
                   ami = "ami-123"
                }

               Terraform creates:

                     aws_instance.server[0]
                     aws_instance.server[1]
                     aws_instance.server[2]

        Count is good when all resources are identical and only the number changes.

        for_each uses a map or set of values, and each resource has a unique key.
      Example:

            resource "aws_instance" "server" {

            for_each = {
                web  = "t2.micro"
                app  = "t2.small"
                db   = "t2.large"
               }

            instance_type = each.value
           }

           Creates:

              aws_instance.server["web"]
              aws_instance.server["app"]
              aws_instance.server["db"]
## 5. Explain Terraform backend and state locking.
Ans: Terraform backend defines where Terraform stores its state file and how it accesses it. By default Terraform stores state locally, but in production we use remote backends like AWS S3 or Azure Storage Account.
     Remote backend is important because multiple team members or CI/CD pipelines can work on the same infrastructure. Everyone accesses the same centralized state instead of having different local state files.
     The backend stores resource information like resource IDs, attributes, dependencies, and outputs. Terraform uses this state during plan and apply to compare desired configuration with actual infrastructure.
     State locking ensures only one Terraform operation can modify the state at a time. When one user runs apply, Terraform locks the state, performs changes, updates the state, and releases the lock. This prevents race conditions and state corruption.
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

What is Terraform and why is it used?
What are the advantages of Infrastructure as Code (IaC)?
Explain the Terraform workflow.
What are Providers in Terraform?
What are Resources?
What are Data Sources?
Difference between Resource and Data Source?
What are Variables and Outputs?
What are Local Values?
What is the Terraform State file?
Why is the State file important?
What is Terraform Refresh?
Difference between terraform plan and terraform apply?
What is terraform init?
What is terraform validate?
What is terraform fmt?
What is terraform destroy?
What is a Backend in Terraform?
What is Remote State?
Why do we use S3 as a backend?
Why is DynamoDB used with Terraform?
What is State Locking?
What is State Drift?
How do you detect and resolve state drift?
What is terraform import?
What are Modules in Terraform?
Difference between Root Module and Child Module?
What are Input Variables?
What are Output Variables?
What is count in Terraform?
What is for_each and how is it different from count?
What are Dynamic Blocks?
What is depends_on?
What are Lifecycle Rules?
What is create_before_destroy?
What is prevent_destroy?
What is ignore_changes?
What are Provisioners?
Difference between local-exec and remote-exec?
What are Workspaces?
What is Terraform Cloud?
What is Terragrunt?
How do you manage secrets in Terraform?
How do you secure the Terraform State file?
How do you manage multiple environments using Terraform?
What is a Rolling Update in Terraform?
Terraform apply failed midway. How will you recover?
State file got corrupted. How will you recover it?
How do you troubleshoot state lock issues?
Infrastructure has changed manually outside Terraform. How will you bring it back to the desired state?
• Terraform state file is locked. How do you resolve it?
• Someone manually deleted an AWS resource. How will you fix it?
• terraform apply failed after creating some resources. What will you do?
• How do you manage dev, test, and prod environments?
• How do you avoid accidental deletion of production resources?
• How do you share outputs between modules?

How do you migrate local state to a remote backend?
• How do you recover from state file corruption?
• How do you handle secrets securely in Terraform?
• How do you design reusable Terraform modules for large-scale infrastructure?
8. Why is Auto Scaling preferred over multiple EC2 instances?
------------------------------------------------------------------------------------
Narayana Questions
-----------------------------------------------------------------------------------

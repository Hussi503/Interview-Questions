
1. What is Terraform, and why do we use it?
2. Explain the Terraform workflow (init, plan, apply). What happens internally during apply?
3. What is the difference between terraform plan, apply, and refresh? When would you
use refresh?
4. What is a Terraform state file, and why is it important?
5. How do you manage state files securely using remote backends (S3 + DynamoDB)?
6. Why do we lock Terraform versions? What is required_version?
7. How do you lock provider versions? What does ~> 5.0 mean?
8. What problems occur if versions are not pinned? How do you upgrade Terraform safely?
9. What happens if two people run terraform apply at the same time? What error will the
second person get?

10. What is a state lock error, and how do you fix it using force-unlock?
11. How do you fix a corrupted Terraform state?
12. What is the difference between count and for_each? When should you use each?
13. What are the risks of switching from count to for_each?
14. How do you create multiple identical EC2 instances with different sizes and tags
dynamically?
15. What are Terraform providers and provisioners? What is the difference between them?
16. Why are provisioners not recommended in production?
17. What are Terraform modules, and why do you use them?
18. How do you create reusable Terraform modules?
19. What is the root module in Terraform?
20. How do you call a module from another module?
21. What is a null_resource, and when would you use it?
22. What is a data source in Terraform? How do you use it to fetch existing resource IDs?
23. How do you import an existing AWS resource into Terraform using terraform import?
24. What is terraform taint, and when would you use it?
25. What is terraform validate? What does it check?
26. What is terraform fmt?
27. What does terraform state list do?
28. What is the difference between terraform destroy and terraform apply?
29. How do you prevent accidental destruction of critical resources in Terraform?

30. How do you handle manual changes made directly in the AWS console for Terraform-
managed resources?

31. What is Terraform drift, and how do you detect and fix it?
32. What happens when you run terraform apply after manual changes?
33. How do you reconcile manually changed resources with Terraform?
34. How do you design Terraform for multiple environments (dev, staging, prod)?
35. How do you manage Terraform state for multiple teams and environments?
36. How do you design Terraform for 50+ AWS accounts?
37. What is a Terraform workspace? Why can workspaces be dangerous in large organizations?
38. How do you structure Terraform code for a production environment?
39. What is the recommended folder structure for a production-grade Terraform project?
40. How do you integrate Terraform with CI/CD pipelines?

41. How do you run Terraform safely in CI/CD pipelines?
42. How do you handle Terraform state when migrating from local to remote backend?
43. If a system crashes during terraform apply, what happens? Could Terraform create duplicate
resources?
44. A Terraform apply failed halfway — how do you handle inconsistent states?
45. What are your immediate steps after a Terraform failure in production?
46. Why should you not blindly re-run terraform apply after a failure?
47. How do you compare Terraform state with actual infrastructure?
48. When would you choose rollback vs continue deployment?
49. How do you handle a P0 production issue involving Terraform-managed infrastructure?
50. How do you manage provider version upgrades and migrate state safely?
51. How do you break a monolithic Terraform configuration into micro stacks?
52. How do you test Terraform code before applying to production?
53. How do you handle secrets in Terraform without exposing them in state files?
54. How do you avoid storing secrets in Terraform code?
55. What are the best practices to follow while designing Terraform code?
56. What are the limitations of Terraform? Where can't we use Terraform?
57. Write Terraform code to create two EC2 instances with a security group and VPC subnets.
58. Do you need to mention VPC in Terraform for non-production? How will Terraform know
to use the default VPC?
59. How will you filter AMIs based on owner (e.g., only Amazon-owned AMIs like RHEL 9)?
60. How will you enforce that only t2.large instance types are allowed and throw an error
otherwise?
61. In a single main.tf with EC2 and RDS: if env=dev, create only EC2; if prod, create both —
how do you control this conditionally?
62. Is it possible to dynamically name state files based on resource names?
63. If the backend is configured with one bucket, how will you change to another and migrate
state?
64. How do you manage different environments in Terraform?
65. What are Terraform variables and outputs?
66. How do you debug and troubleshoot Terraform configuration errors?
67. What parameters do you enable for debugging Terraform?
68. Where can you check Terraform logs?

69. How do you configure a Terraform backend using S3 and DynamoDB for state locking?
70. Where do you define the backend configuration?
71. Is state locking and DynamoDB locking the same?
72. In which block do you define backend configuration?
73. What does the provider block do?
74. What causes CIDR overlap errors and "resource already exists" errors?
75. Why is Auto Scaling preferred over multiple EC2 instances?
76. How do you create five EC2 instances with different instance types?
77. How will you prevent accidental resource destruction (e.g., RDS) in Terraform?
78. If an AWS resource was created manually, can you manage it through Terraform later?
How?
79. How do you handle drift detection automatically and enforce policy governance?
80. Have you written any Terraform modules (reusable modules) or resource-based deployment
scripts?
81. What are the different Terraform commands you use to deploy resources?
82. What happens when you run terraform init for the first time?
83. How would you design Terraform for orchestrating infrastructure across multiple clouds?
84. How would you implement Terraform in an organization where servers already exist?
85. How do you manage Terraform state files and reduce blast radius?
86. How would you train junior engineers on Terraform from scratch?
87. Can you write Terraform script to provision two EC2 instances with security group and
VPC subnets?
88. What will you write in the resource block for an EC2 instance?
89. How will you get existing resource IDs (like VPC, subnet) in Terraform using data blocks?
90. How would you deploy Terraform code to multiple AWS accounts?
91. What is the difference between terraform plan and terraform refresh?

92. What are lifecycle meta-arguments (create_before_destroy, prevent_destroy)? Give real-
world use cases.

93. Your Terraform plan shows no changes, but apply still modifies resources — why?
94. How do you restrict resource deletion but allow init, plan, and apply?


TERRAFORM - The "State File Corrupted" Scenarios
1. Terraform apply fails halfway. Resources created but state not updated. How do you recover?
    * Hint: terraform import, manual state editing, terraform refresh, fix and reapply
2. Someone manually deleted resource in AWS. Terraform state still has it. terraform plan shows no changes. How do you fix?
    * Hint: terraform refresh, terraform import, state rm and re-import
3. Your Terraform state file is corrupted. No backup. How do you recreate?
    * Hint: Reverse engineer from AWS, terraform import all, painful but possible
4. How do you manage Terraform state for 50 microservices across 10 environments?
    * Hint: Remote state per component, Terraform workspaces, terragrunt, separate backends
5. Two team members run terraform apply simultaneously. What happens? How to prevent?
    * Hint: State locking with DynamoDB, backend locking, CI/CD serialization
6. You need to refactor Terraform code and move resources to modules. How do you do it without destroying resources?
    * Hint: terraform state mv, careful planning, test in non-prod first
7. Terraform plan shows changes to resource you didn't modify. Why? How to investigate?
    * Hint: Drift, interpolation changes, provider version changes, ignore_changes
8. How do you implement least privilege for Terraform in CI/CD?
    * Hint: IAM roles with specific policies, assume role, dynamic credentials
9. Your Terraform module needs to create resources conditionally. Show count vs for_each with example.
    * Hint: count = var.create ? 1 : 0, for_each = var.enabled ? var.list : []
10. Terraform can't delete S3 bucket because it's not empty. How do you handle this?
    * Hint: force_destroy = true, or use provisioner to empty, or separate lifecycle
11. How do you manage secrets in Terraform code?
    * Hint: No plaintext, use variables, Vault, AWS Secrets Manager, sensitive flag
12. Terraform module outputs are not showing expected values. How do you debug?
    * Hint: terraform console, check resource attributes, depends_on
13. You need to create 20 similar resources but with slight variations. Count or for_each? Why?
    * Hint: for_each for maps, count for lists, for_each better for stable addressing
14. Terraform apply says "Error: Provider produced inconsistent result after apply". What now?
    * Hint: Provider bug, resource schema mismatch, try newer version, file issue
15. How do you test Terraform code before applying to production?
    * Hint: terratest, kitchen-terraform, plan in CI, multiple environments
16. Your Terraform module takes 30 minutes to apply. How do you speed up?
    * Hint: Parallelism, target specific resources, split module, use -parallelism=n
17. How do you handle dependencies between Terraform modules?
    * Hint: Data sources, output dependencies, remote state data, depends_on
18. Terraform state shows resource exists but AWS console doesn't. What happened?
    * Hint: Deleted outside Terraform, region mismatch, account mismatch, drift
19. How do you implement Terraform in an organization with strict change management?
    * Hint: CI/CD with approval, Sentinel policies, OPA, Terraform Cloud runs
20. You need to rotate AWS access keys used by Terraform. How do you do it without breaking pipelines?
    * Hint: Use IAM roles, multiple keys, rotate in advance, update CI/CD secrets


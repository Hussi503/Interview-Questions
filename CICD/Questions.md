
 CI/CD, Jenkins & DevOps – Interview Question Bank

---

## Capgemini

### CI/CD Pipeline & Deployment
### 1. Explain your CI/CD pipeline architecture.
In my projects, the CI/CD pipeline is designed to be fully automated, scalable, and environment-consistent, primarily using tools like GitHub Actions/Jenkins for CI and Terraform for infrastructure provisioning.

The flow starts when a developer pushes code to a Git repository. The CI stage gets triggered, where we perform code checkout, run unit tests, linting, and security scans (like Snyk or SonarQube). Once validation passes, we build the artifact—for example, a Docker image for container workloads or a ZIP package for Lambda—and push it to a repository like ECR or S3.

In the CD stage, we use Infrastructure as Code (Terraform/CloudFormation) to deploy the application. The pipeline updates the infrastructure if needed and deploys the new artifact. For services like Lambda or ECS, we use versioning and deployment strategies like blue-green or rolling deployments to ensure zero downtime.

We also maintain multiple environments (dev, staging, prod) with separate configurations, and use environment-specific variables or secrets stored in Secrets Manager or Parameter Store. Monitoring is integrated using CloudWatch and alerts, and rollback mechanisms are in place in case of deployment failure.

Overall, the pipeline ensures faster releases, consistency across environments, minimal manual intervention, and high reliability, which is critical for production-grade DevOps workflows.

2. How is Jenkins integrated with Kubernetes?
## 3. How do you implement automated deployments?
In production, automated deployments are implemented through a CI/CD pipeline where deployment is fully triggered based on code changes, removing any manual intervention. Typically, when code is pushed to a repository, the CI stage builds and tests the application, and once it passes, the CD stage automatically deploys it to the target environment.

We use pipeline tools like Jenkins or GitHub Actions, integrated with IaC tools like Terraform or Helm, to ensure consistent and repeatable deployments. For containerized applications, the pipeline builds a Docker image, pushes it to a registry like ECR, and deploys it to Kubernetes using Helm charts or kubectl manifests. For serverless, we deploy using CloudFormation/CDK/Terraform with versioning.

From a production perspective, we never directly deploy to live systems—we implement deployment strategies like blue-green, rolling, or canary releases to ensure zero downtime and minimize risk. We also add approval gates for production, along with automated smoke tests and health checks post-deployment.

Additionally, configurations and secrets are managed externally using Parameter Store or Secrets Manager, ensuring secure deployments across environments. This approach enables fast, reliable, and consistent releases while maintaining high availability and rollback capability.

5. What strategies do you use for deployment rollbacks?

---

## Wipro

### Project Experience
1. What were your roles and responsibilities in your previous project?
2. Were you responsible for creating the CI/CD pipeline in that project?

### Pipeline & Process
3. Can you explain the pipeline flow and the different stages you had in the pipeline?
4. What security measures did you take in the pipeline?
5. How was the branching strategy in your project?

---

## Narayana – Jenkins & CI/CD Deep Dive

### Jenkins Basics
1. Write a pipeline code to automate Jenkins.
2. Explain the concept of "pipeline as code" in Jenkins.
3. How do you manually trigger a Jenkins job when manual checking is required?
4. What are agents and runners in Jenkins?

### Deployments & Availability
5. How do you deploy a new Jenkins deployment with zero downtime?
6. Explain how to create multi-stage jobs in Jenkins.

### Troubleshooting
7. A job works fine in dev environment but not in production — how do you troubleshoot?
8. A stage failed in a Jenkins pipeline due to a credentials issue — how do you fix it?

### CI/CD Fundamentals
9. What is Continuous Integration and Continuous Deployment?
10. Explain a CI/CD pipeline in detail from code commit to production.

### Security & Optimization
11. How do you implement security in CI/CD pipelines?
12. A pipeline takes a long time to complete — how do you troubleshoot and optimize?

### Pipeline Types
13. What is the difference between declarative and scripted pipelines?
14. What are the typical stages in a Jenkins pipeline?

### AWS Integration
15. How do you connect Jenkins to an EC2 instance and authenticate to AWS?
16. What is the difference between IAM user credentials and IAM roles in Jenkins?
17. How do you securely store secrets in Jenkins?

### Failure & Debugging
18. What happens if a Jenkins pipeline fails in the resource creation stage?
19. Explain sshagent in Jenkins.
20. How do you deploy code from Jenkins to EC2?
21. A Jenkins pipeline failed while creating AWS resources — what do you check first?
22. Terraform apply failed in Jenkins but works locally — why?
23. EC2 was created but provisioning failed — how do you debug?

### Advanced CI/CD
24. How do you make CI/CD pipelines idempotent?
25. What is the difference between Continuous Delivery and Continuous Deployment?
26. What tools have you used for CI/CD, and how does CI/CD fit into DevOps?
27. How do you handle rollback in CI/CD?
28. What are common CI/CD failures, and how do you debug them?

### Containers & Kubernetes
29. Design a CI/CD pipeline for a containerized application.
30. How do you integrate Jenkins with Docker and securely push images?
31. How does Jenkins authenticate with Kubernetes?
32. What are different ways to deploy to Kubernetes from Jenkins?
33. Why would you use Helm instead of kubectl?

### Deployment Validation
34. How do you perform post-deployment validation and handle rollbacks?
35. How do you manage environment separation (dev/stage/prod) in CI/CD?

### Runtime Issues
36. Pipeline triggered but no job runs — how do you debug?
37. Jenkins job stuck in pending state — reasons?
38. Build passed but app is not accessible — what to check?

### Artifact & Image Issues
39. Docker image build failing — common causes?
40. Artifact version mismatch issue — how to fix?
41. Rollback failed in production — possible reasons?
42. Smoke test failed post-deployment — what will you do?

### Kubernetes Debugging
43. Deployment succeeded but pods crashing — debugging steps?

### Production Design
44. How do you design a production-grade Jenkins pipeline with approval gates?
45. How do you manage Jenkins agents at scale and reduce blast radius?

### Tool Comparison
46. Jenkins vs GitHub Actions vs Argo CD — when to use?
47. What are the operational challenges of Jenkins?
48. How is GitHub Actions different from Jenkins architecturally?
49. Why is Argo CD not considered a CI tool?

### GitOps & CD
50. Explain GitOps and how Argo CD implements it.
51. Push vs pull-based deployment — Jenkins vs Argo CD.
52. Design CI with Jenkins and CD with Argo CD.

### Governance & Control
53. How do you implement approvals in Jenkins and GitHub Actions?
54. What does `agent any` mean in Jenkins pipeline?
55. What is a declarative pipeline, and how do you write one?

### Agents & Execution
56. Why use static vs dynamic agents?
57. What happens in checkout stage?
58. What is a webhook and how does it work?
59. How do you integrate Git webhook with Jenkins?

### Multi-Environment Strategy
60. How do you manage 5 environments (Dev → Prod)?
61. How do you handle deployments from branches (develop vs release)?
62. Have you written pipelines in declarative syntax?

### Optimization & Execution
63. How do you reduce Jenkins build time?
64. Where are Jenkins builds executed?
65. How do you handle parallel releases?

### Release Management
66. What is your release process?
67. Do you use Jenkins shared libraries?
68. Write a sample shared library usage.
69. How to design CI/CD with latest tools?

### Advanced Scenarios
70. How to automate daily API scripts via pipeline?
71. Are you responsible for CI/CD pipelines?
72. Maintaining vs creating pipelines?
73. What is CI/CD optimization?
74. How do you support frequent releases safely?
75. How to design CI/CD for hundreds of microservices?
76. Avoid separate pipelines for each service?
77. Reuse pipeline logic?
78. Trigger pipeline across repos?

### Quality & Security
79. What is a quality gate (SonarQube)?
80. Integrate Trivy into pipeline?
81. Image vulnerability scanning?
82. End-to-end pipeline security?

## Real-world Debugging (2 AM Scenarios)

1. Your Jenkins master crashed. No backup. How do you recover pipelines and jobs?  
   *Hint:* Jenkins home directory backup, job configs (XML), SCM sync plugin, rebuild from Git-based Jenkinsfiles.

2. Pipeline runs fine on Jenkins master but fails on agent with "command not found". Why?  
   *Hint:* PATH differences, missing tools on agent, wrong agent label, environment mismatch.

3. How do you implement CI/CD for database migrations without breaking production?  
   *Hint:* Flyway/Liquibase, backward-compatible changes, schema versioning, blue-green deployment.

4. Your pipeline takes 45 minutes. Developers complain. How do you reduce it to under 10 minutes?  
   *Hint:* Parallel stages, dependency caching, incremental builds, optimized agents, splitting pipelines.

5. A pipeline stage fails intermittently. Sometimes works, sometimes fails. How do you debug?  
   *Hint:* Flaky tests, race conditions, external dependencies, retries, resource contention, logs correlation.

6. How do you implement approval gates in Jenkins pipeline without stopping the entire pipeline?  
   *Hint:* `input` step with timeout, manual approval stage, conditional execution.

7. Jenkins agent runs out of disk space after 10 builds. How do you clean up automatically?  
   *Hint:* Workspace cleanup plugin, artifact retention policy, `docker system prune`, cron cleanup jobs.

8. You need to promote the same build artifact from dev to prod without rebuilding. How?  
   *Hint:* Artifact repository (Nexus/Artifactory), versioning, immutable artifacts, promotion strategy.

9. Multiple teams use same Jenkins. One team's pipeline consumes all resources. How do you isolate?  
   *Hint:* Kubernetes-based dynamic agents, resource quotas, labels, separate Jenkins controllers.

10. Your pipeline has 20 stages. One stage fails. How do you resume from failed stage?  
    *Hint:* Restart from stage (Blue Ocean), pipeline checkpointing, idempotent design.

11. Secrets in Jenkins are exposed in console logs. How do you prevent this?  
    *Hint:* Credentials binding, masking, avoid echoing secrets, `withCredentials`.

12. How do you version control Jenkins pipelines? Show real example.  
    *Hint:* Store Jenkinsfiles in Git, use shared libraries, SCM-based job configuration.

13. Your pipeline needs to deploy to 1000 servers. How do you do it efficiently?  
    *Hint:* Rolling deployments, batch execution, Ansible parallelism, SSH multiplexing.

14. Pipeline fails with "No space left on device" but disk shows free space. Why?  
    *Hint:* Inode exhaustion, Docker overlay storage, tmpfs limits, user quotas.

15. How do you handle pipeline failures due to external dependencies (GitHub down, Docker Hub limits)?  
    *Hint:* Retry logic, caching, mirror registries, circuit breaker pattern, fallback mechanisms.
``

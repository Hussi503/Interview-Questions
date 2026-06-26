## 1. What is Lambda, and how do you deploy artifacts to it?

AWS Lambda is a serverless compute service used to run event-driven workloads without provisioning or managing infrastructure. In production, we use it for use cases like API backends via API Gateway, async processing via SQS/SNS, and data processing triggered from S3 events. It automatically scales based on incoming requests and bills per execution, making it efficient for microservices and burst workloads.

For deployments, we follow a CI/CD-based approach instead of manual uploads. The application is packaged either as a ZIP (for lightweight functions) or a container image (stored in ECR for larger workloads). The pipeline (GitHub Actions/Jenkins) builds the artifact, pushes it to S3 or ECR, and deploys using IaC tools like Terraform, CloudFormation, or CDK. We use Lambda versions and aliases for safe deployments, enabling blue-green or canary releases via CodeDeploy, ensuring zero downtime and easy rollback in production.

---

## 2. How do you establish a secure connection between Lambda and S3?

In production, security between Lambda and S3 is handled entirely via IAM roles and policies, avoiding hardcoded credentials. We assign an execution role to Lambda with least-privilege access, for example allowing only `s3:GetObject` on a specific bucket or path. The S3 bucket policy can also restrict access to only that Lambda role for additional control.

If the Lambda runs inside a VPC, we use a VPC endpoint for S3 to ensure traffic doesn’t traverse the public internet. For sensitive data, we enforce encryption at rest using SSE-S3 or SSE-KMS and in transit via HTTPS. Additionally, we use IAM condition keys or resource-based policies to tightly control who can access the data. This setup ensures secure, auditable, and compliant communication between Lambda and S3 in real environments.

---

## 3. Explain AWS Lambda functionality and its purpose

Lambda works on an event-driven execution model where the function is triggered by AWS services like API Gateway, S3, DynamoDB streams, or messaging services like SQS/SNS. When an event occurs, Lambda provisions the required runtime, executes the function, and scales automatically based on concurrency. It abstracts away infrastructure management, patching, and scaling concerns.

The primary purpose of Lambda in production is to enable building scalable, loosely coupled architectures. It is heavily used in microservices, automation, real-time processing, and backend APIs. It integrates natively with AWS services, making it ideal for event processing pipelines. However, we optimize its usage by managing cold starts, setting memory/timeouts properly, and handling idempotency and retries to ensure reliability at scale.

---

## 4. Do you have exposure to API Gateway and Lambda functions?

Yes, in production we commonly use API Gateway integrated with Lambda to build serverless APIs. API Gateway acts as the entry point, handling request routing, authentication (IAM, Cognito, or JWT), throttling, and caching, while Lambda executes the backend logic.

In real-time scenarios, we design REST or HTTP APIs where API Gateway triggers Lambda functions for each request. We also configure stages (dev, staging, prod), implement custom authorizers for security, and enable logging via CloudWatch for observability. For deployments, we integrate API Gateway and Lambda into CI/CD pipelines and use stage variables or environment configs to manage environments. This setup allows us to build highly scalable, cost-efficient, and secure APIs without managing backend servers.
``

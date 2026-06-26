## 1. What is Lambda, and how do you deploy artifacts to it?

AWS Lambda is a serverless compute service used to run event-driven workloads without provisioning or managing infrastructure. In production, we use it for use cases like API backends via API Gateway, async processing via SQS/SNS, and data processing triggered from S3 events. It automatically scales based on incoming requests and bills per execution, making it efficient for microservices and burst workloads.

For deployments, we follow a CI/CD-based approach instead of manual uploads. The application is packaged either as a ZIP (for lightweight functions) or a container image (stored in ECR for larger workloads). The pipeline (GitHub Actions/Jenkins) builds the artifact, pushes it to S3 or ECR, and deploys using IaC tools like Terraform, CloudFormation, or CDK. We use Lambda versions and aliases for safe deployments, enabling blue-green or canary releases via CodeDeploy, ensuring zero downtime and easy rollback in production.

---

## 2. How do you establish a secure connection between Lambda and S3?

in production, a secure connection between Lambda and S3 is established primarily using IAM roles with least-privilege access, not credentials. We attach an execution role to the Lambda function that explicitly allows only required actions like s3:GetObject or s3:PutObject on a specific bucket or prefix, avoiding wildcard permissions. Additionally, we enforce security at the S3 side using bucket policies to allow access only from that Lambda role, ensuring tighter control.

For network-level security, if Lambda is running inside a VPC, we configure a VPC Gateway Endpoint for S3, so traffic stays within the AWS private network and does not traverse the public internet. From a data protection standpoint, we enable encryption at rest using SSE-S3 or SSE-KMS, and ensure all communication happens over HTTPS for encryption in transit.

In more secure environments, we also use KMS key policies, IAM condition keys, and resource-level restrictions to further lock down access.

---

## 3. Explain AWS Lambda functionality and its purpose

AWS Lambda is a serverless compute service that executes code in response to events without requiring infrastructure management. It follows an event-driven model where services like API Gateway, S3, SQS, or DynamoDB trigger the function, and AWS automatically handles provisioning, scaling, and high availability. In production, Lambda works by allocating a runtime environment, executing the function, and scaling horizontally based on the number of incoming events, with concurrency controls and built-in retry mechanisms for resilience.

The primary purpose of Lambda is to enable highly scalable, loosely coupled architectures, especially in microservices and event-driven systems. It’s widely used for building backend APIs, real-time data processing, automation tasks, and integrating AWS services. From a DevOps perspective, we focus on optimizing cold starts, configuring memory and timeout properly, and ensuring idempotent design for retries. Combined with native integrations and pay-per-use pricing, Lambda helps reduce operational overhead while maintaining scalability, cost efficiency, and faster time to market in production environments.

---

## 4. Do you have exposure to API Gateway and Lambda functions?

Yes, while I haven’t worked on API Gateway and Lambda extensively in a production project, I do have a good understanding of how they integrate and I’ve implemented them in lab and POC setups. Typically, API Gateway acts as the entry point that exposes REST or HTTP endpoints, and it triggers Lambda functions to execute backend logic in a serverless architecture.

I understand how to configure routes, integrate Lambda as a backend, and secure APIs using mechanisms like IAM roles or JWT-based authentication. I’m also familiar with setting up stages for different environments and enabling logging through CloudWatch for basic debugging and monitoring.

From a DevOps perspective, I know how this setup is deployed using IaC tools like Terraform or CloudFormation, and how CI/CD pipelines automate deployments. Although I haven’t handled this in a live production system yet, I’m comfortable with the concepts and can quickly work on it in real-time environments.

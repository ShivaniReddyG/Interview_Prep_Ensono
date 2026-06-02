Q1
Explain the difference between terraform plan, apply, and refresh. When would you use terraform refresh separately?

How do you manage Terraform state in a team environment? What issues arise with local state?

Q3
What is a Terraform module? How do you version and reuse modules across multiple projects?
Q4
How would you handle a situation where terraform apply partially failed and left infra in an inconsistent state?
Q5
Explain Terraform workspaces. Are they suitable for managing prod/staging/dev environments?
Q6
What is the Terraform dependency graph and how does it affect resource creation order?

An L2 (Level 2 / Intermediate-Senior) interview for a **Terraform + Harness + AWS** role at Ensono will focus heavily on how you handle real-world infrastructure-as-code (IaC), continuous delivery (CD) pipelines, and cloud architecture at scale.

Because Ensono is a major managed service provider (MSP), they don't just care if you know the syntax—they want to know how you prevent downtime, manage multi-tenant environments, and troubleshoot when deployments fail.

Here are some tough, scenario-based interview questions you should prepare for, broken down by technology.

---

## 🛠️ Terraform (Advanced State & Scale)

At the L2 level, questions move past simple resource creation and dive into state management failures and modular design.

### 1. "We have a critical production database managed by Terraform. A junior engineer manually modified the security group in the AWS Console. What happens on the next `terraform plan`, and how do you fix it without causing a service interruption?"

* **What they are looking for:** Your understanding of **configuration drift**. You should explain that `terraform plan` will detect the variance and attempt to overwrite the manual change to match the code.
* **The Answer:** If the manual change was an emergency fix that needs to stay, you must update the Terraform code to match the new AWS settings so the plan shows zero changes. If the change was a mistake, you let Terraform roll it back. You should also mention using `terraform refresh` (or how it's implicitly run during a plan) to sync the state file first.

### 2. "How do you handle a scenario where two developers or automated pipelines run `terraform apply` at the exact same time on the same state file?"

* **What they are looking for:** Knowledge of **state locking**.
* **The Answer:** You should talk about using a remote backend (like AWS S3) coupled with a locking mechanism (like a **DynamoDB table**). Explain that the backend uses a `LockID` string attribute in DynamoDB. The first process acquires the lock, and the second process receives a `ResourceLock` error and terminates, preventing state corruption.

### 3. "Explain the difference between `moved` blocks (introduced in Terraform 1.1+) and running `terraform state mv`. When would you use which?"

* **What they are looking for:** Familiarity with modern refactoring best practices.
* **The Answer:** `terraform state mv` is a manual, one-time command-line operation. It modifies the state file directly on your local machine or remote backend, but it doesn't update the code. `moved` blocks are written directly into the `.tf` configuration files. They allow state refactoring (like renaming a module or resource) to happen automatically and safely across team members' environments and CI/CD pipelines without destroying and recreating resources.

---

## 🎛️ Harness (GitOps, Pipelines & Delegates)

Ensono uses Harness for enterprise-grade continuous delivery. Expect questions on architecture, troubleshooting, and deployment strategies.

### 4. "A Harness deployment pipeline fails at the infrastructure provisioning step because it cannot connect to your private AWS VPC. The Harness Delegate is running in that VPC. How do you troubleshoot this?"

* **What they are looking for:** Network troubleshooting skills and understanding of the Harness Delegate architecture.
* **The Answer:**
1. Check if the Delegate pod/instance is actually running and healthy via the Harness UI.
2. Verify IAM roles: Does the Delegate instance profile have the required AWS permissions to provision those specific resources?
3. Check security groups and network ACLs: Can the Delegate talk to the AWS STS/IAM endpoints?
4. Review the Delegate logs for specific token expiration or `AssumeRole` errors.



### 5. "How would you design a canary deployment strategy in Harness for a microservice running on AWS ECS?"

* **What they are looking for:** Understanding of advanced deployment patterns.
* **The Answer:** Break it down into phases:
* **Phase 1:** Harness deploys a small fraction of the new tasks (e.g., 10%) alongside the production tasks.
* **Phase 2:** Route a small percentage of traffic to the new version using an AWS Application Load Balancer (ALB) target group weight.
* **Phase 3:** Insert a **Harness Verification Step** (integrating tools like Prometheus, Datadog, or CloudWatch) to monitor error rates and latency for 5–10 minutes.
* **Phase 4:** If metrics are clean, scale up to 100%. If anomalies are detected, leverage Harness's automatic rollback step to revert traffic instantly.



---

## ☁️ AWS (Architecture & Enterprise Operations)

As an MSP engineer, you need to understand multi-account strategies and secure connectivity.

### 6. "How do you securely allow a Terraform pipeline running in a centralized management AWS account to provision resources in dozens of different client target accounts?"

* **What they are looking for:** Cross-account IAM architecture.
* **The Answer:** You should implement **IAM Role Assumption**.
* In the target client accounts, define an IAM role (e.g., `TerraformDeploymentRole`) with a trust policy that allows the centralized management account's IAM role/execution identity to assume it.
* In your Terraform code, configure the AWS provider to use the `assume_role` block, dynamically passing the Target Account ID and Role ARN. This avoids hardcoding any long-lived access keys.



### 7. "An application hosted on AWS EC2 instances in a private subnet needs to securely access files in an S3 bucket without routing traffic over the public internet. How do you configure this via Terraform?"

* **What they are looking for:** Knowledge of VPC endpoints and secure cloud networking.
* **The Answer:** You need to deploy an **S3 Gateway VPC Endpoint**. In Terraform, you would use the `aws_vpc_endpoint` resource with `service_name = "com.amazonaws.[region].s3"` and type set to `Gateway`. You must also associate this endpoint with the route tables of your private subnets so traffic destined for S3 is automatically routed through the internal AWS network.

---

## 🎯 Pro-Tips for the Ensono Interview

* **Emphasize Multi-Tenancy:** Ensono manages infrastructure for many clients. When answering, mention how you use Terraform workspaces, variables, or distinct Terragrunt/Harness environments to keep client infrastructures strictly isolated.
* **Talk About Rollbacks:** Don't just explain how to build things; emphasize how you safely tear them down or roll them back when a deployment goes sideways.
* **Security First:** Always mention least-privilege access. For instance, when discussing AWS or Harness, emphasize that delegates or execution paths only have the exact permissions required to do their job.

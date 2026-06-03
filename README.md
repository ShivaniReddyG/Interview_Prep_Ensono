Here are **19 tough L2 questions** mapped directly to the JD. A few things to note as you prep:

**Where candidates typically get tripped up:**

- **Terraform state** — interviewers love the stuck-lock and state-split scenarios because they separate people who've actually operated Terraform at scale from those who've only written it.
- **Harness CV** — knowing what Continuous Verification actually does (ML-based anomaly detection, not just health checks) signals real hands-on experience.
- **Tekton vs Harness tradeoffs** — the JD mentions both, so expect a compare/contrast question. The honest answer involves operational cost, not just features.
- **IAM + IRSA** — almost every AWS+EKS interview includes this. Know the trust policy structure cold.

**How to use this:** Click any question to reveal a model answer framework. The goal isn't to memorize these verbatim — it's to ensure you have a concrete, structured answer pattern (problem → mechanics → gotchas → solution) for each area.

Want me to go deeper on any specific topic, add scenario-based behavioral questions, or generate a mock interview dialogue?


https://claude.ai/share/4cf340c9-a025-4bb4-8758-475587ef8371


You have a Terraform root module managing 200+ resources. A plan shows an unexpected destroy on a critical RDS instance. Walk me through your debugging process before applying. Terraformhard
02
Explain how Terraform handles state locking. What happens when a lock is stuck, and how do you safely recover without corrupting state? Terraformhard
03
How do you design Terraform modules for a multi-account AWS landing zone? Walk through your module hierarchy and state separation strategy. Terraformhard
04
What is the difference between `count`, `for_each`, and dynamic blocks? When would you choose each, and what are the pitfalls? Terraformmed
05
You're refactoring a legacy Terraform codebase with all resources in a single state file. How do you split it into smaller state files without downtime? Terraformhard
06
How do you implement policy-as-code with Sentinel or OPA for Terraform? Give a concrete policy example for enforcing mandatory tags. Terraformhard
07
Explain Harness pipeline stages, step groups, and failure strategies. How would you model a multi-env promotion flow with approval gates? Harnesshard
08
How does Harness Continuous Verification work? Walk through setting up CV with Prometheus for a Kubernetes deployment. Harnesshard
09
What are Harness Templates and how do they enable governance at enterprise scale? What's the difference between a Pipeline Template and a Stage Template? Harnessmed
10
How do you implement GitOps-style deployments in Harness using the GitOps agent? How does it differ from Harness's traditional CD approach? Harnesshard
11
Compare Tekton and Harness CI for a large enterprise. When would you recommend Tekton over Harness, and what are the operational tradeoffs? Tektonhard
12
Walk me through designing a Tekton pipeline for building and deploying a containerized app — including caching, parallel tasks, and workspace sharing. Tektonhard
13
How do Tekton Triggers work? Describe how you'd set up a GitHub webhook to trigger a pipeline on PR open. Tektonmed
14
You need to deploy a multi-region active-active application on AWS with Terraform. How do you structure providers and state for multi-region management? AWShard
15
How do you implement least-privilege IAM for a Terraform-managed EKS cluster where pods need access to S3 and DynamoDB? AWShard
16
Walk through designing a secure secrets management strategy for a Terraform+Harness+EKS stack — covering both pipeline secrets and runtime secrets. AWShard
17
Production incident: a Harness pipeline deployed bad code to prod. The rollback didn't work. Walk me through your incident response and what guardrails you'd add afterward. Scenariohard
18
How do you prevent Terraform state files from becoming a security liability? What data can leak, and how do you harden your state backend? Securityhard
19
How do you implement RBAC and team isolation in a shared Harness account used by 20+ application teams? Securityhard



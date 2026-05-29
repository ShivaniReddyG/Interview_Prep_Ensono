const {
  Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell,
  HeadingLevel, AlignmentType, BorderStyle, WidthType, ShadingType,
  LevelFormat, VerticalAlign, PageNumber, Footer
} = require('docx');
const fs = require('fs');

const BLUE = "1F4E79";
const LIGHT_BLUE = "D5E8F0";
const MID_BLUE = "2E75B6";
const TEAL_BG = "E1F5EE";
const PURPLE_BG = "EEEDFE";
const AMBER_BG = "FAEEDA";
const CORAL_BG = "FAECE7";
const BORDER_GRAY = "CCCCCC";

const border = { style: BorderStyle.SINGLE, size: 1, color: BORDER_GRAY };
const borders = { top: border, bottom: border, left: border, right: border };

function heading1(text) {
  return new Paragraph({
    heading: HeadingLevel.HEADING_1,
    spacing: { before: 360, after: 180 },
    children: [new TextRun({ text, bold: true, size: 36, color: BLUE, font: "Arial" })]
  });
}

function heading2(text) {
  return new Paragraph({
    heading: HeadingLevel.HEADING_2,
    spacing: { before: 280, after: 120 },
    border: { bottom: { style: BorderStyle.SINGLE, size: 4, color: MID_BLUE, space: 4 } },
    children: [new TextRun({ text, bold: true, size: 28, color: MID_BLUE, font: "Arial" })]
  });
}

function bodyText(text, opts = {}) {
  return new Paragraph({
    spacing: { before: 80, after: 80 },
    children: [new TextRun({ text, size: 22, font: "Arial", ...opts })]
  });
}

function sectionLabel(text) {
  return new Paragraph({
    spacing: { before: 200, after: 80 },
    children: [new TextRun({ text: text.toUpperCase(), size: 18, bold: true, color: "888780", font: "Arial", characterSpacing: 80 })]
  });
}

function questionCard(num, question, answer, tip, code) {
  const rows = [];

  // Question row (header)
  rows.push(new TableRow({
    children: [
      new TableCell({
        width: { size: 600, type: WidthType.DXA },
        borders,
        shading: { fill: MID_BLUE, type: ShadingType.CLEAR },
        margins: { top: 100, bottom: 100, left: 120, right: 120 },
        verticalAlign: VerticalAlign.CENTER,
        children: [new Paragraph({
          alignment: AlignmentType.CENTER,
          children: [new TextRun({ text: `Q${num}`, bold: true, size: 22, color: "FFFFFF", font: "Arial" })]
        })]
      }),
      new TableCell({
        width: { size: 8760, type: WidthType.DXA },
        borders,
        shading: { fill: LIGHT_BLUE, type: ShadingType.CLEAR },
        margins: { top: 100, bottom: 100, left: 160, right: 120 },
        children: [new Paragraph({
          children: [new TextRun({ text: question, bold: true, size: 22, font: "Arial", color: "1F4E79" })]
        })]
      })
    ]
  }));

  // Answer row
  const answerChildren = [new Paragraph({
    spacing: { before: 60, after: 60 },
    children: [new TextRun({ text: answer, size: 20, font: "Arial" })]
  })];

  if (code) {
    answerChildren.push(new Paragraph({
      spacing: { before: 80, after: 60 },
      shading: { fill: "F5F5F5", type: ShadingType.CLEAR },
      border: { left: { style: BorderStyle.SINGLE, size: 6, color: BORDER_GRAY } },
      children: [new TextRun({ text: code, size: 18, font: "Courier New", color: "333333" })]
    }));
  }

  if (tip) {
    answerChildren.push(new Paragraph({
      spacing: { before: 80, after: 60 },
      border: { left: { style: BorderStyle.SINGLE, size: 6, color: "1D9E75" } },
      shading: { fill: TEAL_BG, type: ShadingType.CLEAR },
      children: [
        new TextRun({ text: "Interviewer tip: ", bold: true, size: 19, font: "Arial", color: "0F6E56" }),
        new TextRun({ text: tip, size: 19, font: "Arial", color: "0F6E56" })
      ]
    }));
  }

  rows.push(new TableRow({
    children: [
      new TableCell({
        width: { size: 600, type: WidthType.DXA },
        borders,
        shading: { fill: "F9F9F9", type: ShadingType.CLEAR },
        margins: { top: 100, bottom: 100, left: 120, right: 120 },
        children: [new Paragraph({ children: [] })]
      }),
      new TableCell({
        width: { size: 8760, type: WidthType.DXA },
        borders,
        margins: { top: 100, bottom: 100, left: 160, right: 120 },
        children: answerChildren
      })
    ]
  }));

  return new Table({
    width: { size: 9360, type: WidthType.DXA },
    columnWidths: [600, 8760],
    rows,
    margins: { bottom: 200 }
  });
}

function spacer() {
  return new Paragraph({ spacing: { before: 120, after: 120 }, children: [] });
}

function badgeRow(badges) {
  return new Paragraph({
    spacing: { before: 80, after: 160 },
    children: badges.flatMap((b, i) => [
      new TextRun({ text: `  ${b.text}  `, size: 18, bold: true, font: "Arial", color: b.color, highlight: undefined }),
      new TextRun({ text: "  ", size: 18 })
    ])
  });
}

const sections_data = [
  {
    title: "Section 1: Terraform (Infrastructure as Code)",
    label: "Terraform — Q1 to Q6",
    questions: [
      {
        num: 1,
        q: "Explain the difference between terraform plan, apply, and refresh. When would you use terraform refresh separately?",
        a: "plan previews changes without touching infra. apply executes them. refresh syncs state with real-world infra without making changes — useful when resources were modified out-of-band (e.g. manually in the AWS console). In Terraform >=0.15, refresh is embedded in plan/apply by default; explicit use is now less common but still valid for debugging state drift.",
        tip: "Listen for mention of state drift and out-of-band changes — this signals real-world experience.",
        code: null
      },
      {
        num: 2,
        q: "How do you manage Terraform state in a team environment? What issues arise with local state?",
        a: "Remote backends (S3 + DynamoDB for locking, Azure Blob with lease-based locking, or Terraform Cloud) are the standard solution. Local state causes race conditions if two people run apply simultaneously, and state files can contain secrets in plain text — both are critical risks. State should be encrypted at rest and access-controlled via IAM/RBAC.",
        tip: null,
        code: 'terraform {\n  backend "s3" {\n    bucket         = "ensono-tfstate"\n    key            = "prod/terraform.tfstate"\n    region         = "us-east-1"\n    dynamodb_table = "terraform-locks"\n    encrypt        = true\n  }\n}'
      },
      {
        num: 3,
        q: "What is a Terraform module? How do you version and reuse modules across multiple projects?",
        a: "Modules are reusable units of Terraform config. Teams publish them to a private Terraform Registry or a Git repo and pin specific versions using a source ref. Version constraints prevent unexpected breaking changes.",
        tip: "Look for: awareness of module versioning, input/output variables, and avoiding hardcoding values inside modules.",
        code: 'module "vpc" {\n  source  = "git::https://github.com/ensono/tf-modules.git//vpc?ref=v1.4.0"\n  cidr    = "10.0.0.0/16"\n  env     = var.environment\n}'
      },
      {
        num: 4,
        q: "How would you handle a situation where terraform apply partially failed and left infra in an inconsistent state?",
        a: "First, run terraform plan to understand the current diff. Use terraform state list and terraform state show to inspect what was created. For orphaned resources not in state, use terraform import. For resources that should be removed from state but not destroyed, use terraform state rm. Fix the root cause before re-running apply. Never manually edit the state file directly.",
        tip: null,
        code: null
      },
      {
        num: 5,
        q: "Explain Terraform workspaces. Are they suitable for managing prod/staging/dev environments?",
        a: "Workspaces allow multiple state files in one backend config. They work well for ephemeral or short-lived envs. However, for prod/staging/dev the recommended pattern is separate directories or repos per environment, as workspaces share the same code and can lead to accidental cross-environment changes. Terragrunt is commonly used to DRY up multi-env configurations safely.",
        tip: "Strong answer: Candidate knows workspaces are NOT the same as environment isolation and can articulate the trade-offs.",
        code: null
      },
      {
        num: 6,
        q: "What is the Terraform dependency graph and how does it affect resource creation order?",
        a: "Terraform builds a DAG (directed acyclic graph) of resources based on references between them. Resources that reference another's output are automatically created after it. You can use depends_on to express implicit dependencies that Terraform can't infer automatically (e.g. a Lambda that needs an S3 bucket policy to exist before invocation).",
        tip: null,
        code: null
      }
    ]
  },
  {
    title: "Section 2: Harness Platform",
    label: "Harness — Q7 to Q12",
    questions: [
      {
        num: 7,
        q: "What are the core entities in Harness CD? Walk through the relationship between pipeline, stage, step group, and step.",
        a: "A Pipeline is the top-level entity. It contains one or more Stages (e.g. Deploy, Approval, Custom). Each Stage has a Step Group that can group steps for conditional execution or rollback. Steps are the atomic actions — shell script, HTTP call, Terraform Apply, etc. Services and Environments are referenced from the Deployment stage and define what is deployed and where.",
        tip: null,
        code: null
      },
      {
        num: 8,
        q: "How does Harness handle secrets? What's the difference between built-in secret manager and integrating with Vault/AWS Secrets Manager?",
        a: "Harness has a built-in Secret Manager (GCP KMS-backed by default on SaaS). For enterprise use, you integrate with HashiCorp Vault, AWS Secrets Manager, or Azure Key Vault via a Connector. The delegate fetches secrets at runtime — they're never stored in Harness. References use the syntax <+secrets.getValue(\"mySecret\")> in YAML and are masked in logs automatically.",
        tip: "Ask follow-up: How do you rotate secrets without redeploying pipelines? (Answer: update the secret in the source; Harness fetches at runtime.)",
        code: null
      },
      {
        num: 9,
        q: "What is a Harness Delegate? How do you size and scale delegates for enterprise workloads?",
        a: "The Delegate is an agent running in the customer's environment (Kubernetes pod or Docker container or VM) that executes pipeline steps and connects to cloud providers. Sizing: Harness recommends 1-2 vCPU and 4-8 GB RAM per delegate; scale horizontally by adding replicas with the same selector tag. For high-availability, run >=2 delegates per environment. Delegate groups with matching tags allow targeting specific delegates per stage.",
        tip: null,
        code: null
      },
      {
        num: 10,
        q: "Explain Harness deployment strategies — canary, blue-green, rolling. When would you choose each?",
        a: "Canary: Route a small % of traffic to new version, validate metrics, then shift full traffic. Best for stateless services with meaningful metrics. Blue-green: Two identical environments; switch traffic at the load balancer. Best when you need instant rollback and can afford duplicate infra cost. Rolling: Replace instances in batches. Lower cost than blue-green; some risk of mixed versions. Good for non-critical internal services.",
        tip: "Strong answer: Links strategy choice to Harness's built-in verification steps (CV) that compare APM/log metrics between canary and baseline automatically.",
        code: null
      },
      {
        num: 11,
        q: "How do you implement Harness IaCM (Infrastructure-as-Code Management) with Terraform? How is it different from running Terraform in a shell script step?",
        a: "Harness IaCM provides a native Terraform workspace concept with built-in state management, drift detection, cost estimation (OPA policies), and approval gates — none of which exist with a raw shell step. It tracks workspace history, allows policy-as-code enforcement on plans before apply, and integrates with Harness RBAC. Shell steps work but lack auditability and governance features that enterprises like Ensono's clients require.",
        tip: null,
        code: null
      },
      {
        num: 12,
        q: "How would you implement environment-specific variable overrides in Harness pipelines without duplicating pipelines?",
        a: "Use Harness Service Variables and Environment Overrides. Define defaults in the Service; override per Environment. Pipelines use <+env.name> and <+serviceVariables.VAR_NAME> expressions that resolve at runtime. For complex scenarios, use Runtime Inputs (<+input>) combined with input sets to parameterize a single pipeline for multiple envs.",
        tip: null,
        code: null
      }
    ]
  },
  {
    title: "Section 3: CI/CD Design & Practices",
    label: "CI/CD — Q13 to Q18",
    questions: [
      {
        num: 13,
        q: "Design a CI/CD pipeline for a microservices application where each service has independent release cadences.",
        a: "Each microservice should have its own pipeline triggered by changes in its source directory (path filters on the repo). Steps: lint -> unit test -> build Docker image -> push to ECR/ACR with commit SHA tag -> update Helm chart values -> deploy to dev -> integration tests -> promote to staging/prod via Harness input sets or approval gates. Use a shared library/template for common steps to avoid duplication. Separate Harness Services per microservice for independent versioning.",
        tip: "Look for: mention of artifact versioning, independent service deployment, and avoiding deploy-everything on every commit.",
        code: null
      },
      {
        num: 14,
        q: "How do you implement automated rollback in a Harness pipeline when post-deployment health checks fail?",
        a: "Use Harness's built-in Rollback Steps in the failure strategy section of a stage. Configure the stage failure strategy to Rollback Stage on step failure. Add HTTP/Shell steps as health checks after deployment — on failure, the rollback steps revert to the previous artifact version. For Kubernetes, Harness automatically reverts the Deployment to the prior rollout. Combine with Continuous Verification (CV) for metric-driven automated rollback.",
        tip: null,
        code: null
      },
      {
        num: 15,
        q: "What are pipeline templates in Harness? How do you enforce their use across teams?",
        a: "Templates are reusable pipeline/stage/step definitions stored at Account, Org, or Project scope. They enforce standards — security scans, approval gates, notification steps — without teams needing to configure them manually. Enforcement is done via OPA (Open Policy Agent) policies in Harness that reject pipelines not referencing required templates. Combined with RBAC to restrict who can create/modify templates, this gives centralized governance.",
        tip: null,
        code: null
      },
      {
        num: 16,
        q: "How do you prevent secrets from appearing in CI/CD pipeline logs?",
        a: "In Harness, secrets referenced via <+secrets.getValue()> are automatically masked in logs. For shell steps, avoid echoing secrets; use environment variables injected from Harness secrets rather than passing as CLI arguments. Add a log sanitization step or use regex-based masking if using third-party tools. Audit logs should be reviewed periodically. Never hardcode credentials in YAML — use Harness connectors or secret references only.",
        tip: null,
        code: null
      },
      {
        num: 17,
        q: "Explain GitOps principles and how Harness GitSync or ArgoCD would fit into an enterprise CD workflow.",
        a: "GitOps treats Git as the single source of truth for both app and infrastructure config. A reconciliation loop (ArgoCD or Flux) detects drift between the Git state and cluster state and corrects it. In Harness, GitSync stores pipeline/service YAML in Git — changes are PR-reviewed before they affect pipelines. Harness CD can also trigger off ArgoCD app health events. The combination gives CI in Harness, GitOps-driven CD via Argo, with full auditability.",
        tip: null,
        code: null
      },
      {
        num: 18,
        q: "How do you handle long-running Terraform apply steps in a CI/CD pipeline without hitting timeouts?",
        a: "Set pipeline step timeout values appropriately (Harness defaults to 10 min; increase for infra apply). Use Harness IaCM which handles async apply with polling. For very long runs, split Terraform into smaller targeted applies using -target (use sparingly). Consider breaking monolithic root modules into smaller modules that apply faster. Also increase delegate task timeout settings on the delegate YAML config.",
        tip: null,
        code: null
      }
    ]
  },
  {
    title: "Section 4: AWS & Azure",
    label: "AWS / Azure — Q19 to Q24",
    questions: [
      {
        num: 19,
        q: "Compare IAM roles vs IAM users for service authentication in AWS. What's the recommended pattern for Terraform and Harness?",
        a: "IAM users with long-lived access keys are a security risk — keys can be leaked or forgotten. IAM roles with assumed-role chains or OIDC are preferred. For Terraform in CI/CD: configure the Harness Delegate with an instance profile or use OIDC federation so the delegate assumes a role dynamically. For AWS connectors in Harness, use IAM role delegation (cross-account roles) rather than access key connectors. This eliminates static credentials entirely.",
        tip: null,
        code: null
      },
      {
        num: 20,
        q: "Your Terraform-provisioned ECS service is not starting. Walk through your debugging approach.",
        a: "1. Check ECS task stopped reason in the AWS console or CLI (describe-tasks). 2. Check CloudWatch Logs for the task's log group. 3. Verify task definition — image URI, environment variables, secrets ARNs. 4. Check ECS task IAM role has permissions for any AWS API calls the container makes. 5. Check security group rules and VPC subnet routing. 6. Verify the ELB target group health check path returns 200. 7. If the image fails to pull, check ECR permissions and VPC endpoint or NAT Gateway.",
        tip: "Strong answer: Systematic, layered — infra -> IAM -> network -> app. Not jumping straight to redeploy.",
        code: null
      },
      {
        num: 21,
        q: "How do you manage Terraform state for multi-account AWS environments (e.g. dev/staging/prod in separate accounts)?",
        a: "Use a centralized S3 bucket in a shared-services account with separate state key paths per account and environment (e.g. aws-accounts/prod/us-east-1/eks/terraform.tfstate). Each account's resources use cross-account role assumption via the AWS provider's assume_role block. DynamoDB for state locking can be in the same shared account. Terragrunt helps manage the repetitive backend config across many account/region combinations.",
        tip: null,
        code: null
      },
      {
        num: 22,
        q: "What is Azure Service Principal vs Managed Identity, and which should you use for Harness Azure connectors?",
        a: "A Service Principal is an app identity with a client secret or certificate — requires secret rotation management. A Managed Identity is assigned to Azure resources (VMs, AKS nodes) and requires no credential management. If the Harness Delegate runs on an Azure VM or AKS, use a User-assigned Managed Identity for the delegate VM/pod and assign it the required RBAC roles. This eliminates secrets entirely and is the recommended approach for production Ensono deployments on Azure.",
        tip: null,
        code: null
      },
      {
        num: 23,
        q: "How do you implement cross-region disaster recovery for an application deployed via Terraform on AWS?",
        a: "Use Terraform's aliased provider blocks to provision resources in multiple regions. RDS with cross-region read replicas (promoted on failover), S3 with Cross-Region Replication, Route 53 health checks with failover routing. Deploy the same Terraform module twice with region-specific vars. For EKS, use Velero for cluster backup. Document and test RTO/RPO targets. Use AWS Resilience Hub or Azure Site Recovery to validate.",
        tip: null,
        code: null
      },
      {
        num: 24,
        q: "A Harness pipeline deploys to AWS EKS. Post-deploy, pods are in CrashLoopBackOff. What do you check?",
        a: "1. kubectl describe pod <pod> — events, image pull errors, OOM kills. 2. kubectl logs <pod> --previous — application stacktrace. 3. Check resource limits — if OOMKilled, increase memory limits. 4. Check liveness probe config — misconfigured probe path causes restarts. 5. Verify ConfigMap/Secret mounts — missing env vars or files cause crashes. 6. Check if the new image tag actually exists in ECR. 7. Compare with last working deployment's resource spec.",
        tip: null,
        code: null
      }
    ]
  },
  {
    title: "Section 5: Ensono-Specific & Situational",
    label: "Ensono Fit — Q25 to Q30",
    questions: [
      {
        num: 25,
        q: "Ensono manages infrastructure for multiple enterprise clients. How would you structure Terraform code to serve different clients without mixing their state or configurations?",
        a: "Use separate Terraform workspaces or, better, separate backend state paths per client (clients/clientA/prod/terraform.tfstate). Shared modules live in a central registry. Client-specific values go in tfvars files version-controlled in client-specific repos or directories. RBAC ensures engineers only have access to their client's state bucket prefix. This mirrors Ensono's managed services model — common platform, client-isolated state.",
        tip: null,
        code: null
      },
      {
        num: 26,
        q: "A client's production Harness pipeline is failing intermittently at the Terraform apply step. How do you investigate without impacting their environment?",
        a: "First, pull the full Harness execution log for the failing runs — look for timeouts vs. actual errors vs. delegate connectivity issues. Check delegate health and task queue. Run terraform plan manually from a delegate with the same credentials to isolate if the issue is Terraform or Harness. Check AWS/Azure API rate limits and throttling errors. If intermittent, add retry logic on the step. Engage Harness support with the delegate logs if delegate-side.",
        tip: "Key trait Ensono looks for: Methodical diagnosis, client communication awareness, safe-first mindset.",
        code: null
      },
      {
        num: 27,
        q: "How would you implement compliance-as-code for a regulated client (e.g. PCI-DSS) in their Terraform and Harness workflows?",
        a: "Use Checkov or Terrascan in the CI pipeline to scan Terraform plans for compliance violations before apply. In Harness, integrate OPA policies to block pipelines from deploying to production without a compliance scan passing. Use Harness Governance policies to require specific approval gates for prod deployments. Tag all resources with compliance metadata via Terraform. Generate automated compliance reports from CloudTrail/Azure Monitor. Version-control all policy-as-code alongside infra code.",
        tip: null,
        code: null
      },
      {
        num: 28,
        q: "How do you handle Terraform provider upgrades across multiple client environments without causing outages?",
        a: "Pin provider versions with ~> constraints (e.g. ~> 5.0) to allow patch updates but not major version jumps. Test provider upgrades in a non-prod environment first. Use terraform providers lock to generate a lock file and commit it to source control — this ensures all team members and CI use identical provider binaries. For major upgrades, review the provider's CHANGELOG and run terraform plan extensively before applying. Stagger rollout across clients.",
        tip: null,
        code: null
      },
      {
        num: 29,
        q: "A client wants to migrate from their current Jenkins CI/CD to Harness. What migration approach would you recommend?",
        a: "Start with a pilot service — ideally a non-critical one. Map Jenkins stages to Harness pipeline stages. Migrate secrets to Harness/Vault connectors. Run both pipelines in parallel for a sprint to validate equivalence. Migrate services in waves by risk level. Use Harness GitSync to store pipeline YAML in the existing Git repos so teams feel continuity. Provide training before cutover. Decommission Jenkins pipelines only after Harness has been proven stable for each service.",
        tip: null,
        code: null
      },
      {
        num: 30,
        q: "How do you track and report infrastructure cost changes caused by Terraform deployments across multiple client accounts?",
        a: "Integrate Infracost into the Harness pipeline — it runs against the Terraform plan and posts a cost diff comment (e.g. on a PR or Harness step). Tag all resources with client, environment, and cost-center tags via Terraform. Use AWS Cost Explorer or Azure Cost Management with tag-based filtering for per-client reports. Harness CCM (Cloud Cost Management) can aggregate costs across accounts and services. Set budget alerts per client account to proactively catch anomalies.",
        tip: null,
        code: null
      }
    ]
  }
];

const docChildren = [];

// Title page
docChildren.push(new Paragraph({
  alignment: AlignmentType.CENTER,
  spacing: { before: 720, after: 120 },
  children: [new TextRun({ text: "L2 Interview Question Guide", bold: true, size: 52, color: BLUE, font: "Arial" })]
}));

docChildren.push(new Paragraph({
  alignment: AlignmentType.CENTER,
  spacing: { before: 0, after: 80 },
  children: [new TextRun({ text: "DevOps Engineer — Terraform & Harness", size: 32, color: MID_BLUE, font: "Arial" })]
}));

docChildren.push(new Paragraph({
  alignment: AlignmentType.CENTER,
  spacing: { before: 0, after: 80 },
  children: [new TextRun({ text: "Ensono | Candidate Preparation Guide", size: 26, color: "888780", font: "Arial", italics: true })]
}));

docChildren.push(new Paragraph({
  alignment: AlignmentType.CENTER,
  spacing: { before: 120, after: 480 },
  children: [new TextRun({ text: "Terraform  |  Harness  |  CI/CD  |  AWS  |  Azure  |  Ensono Fit", size: 22, color: "5F5E5A", font: "Arial" })]
}));

// Stats table
docChildren.push(new Table({
  width: { size: 9360, type: WidthType.DXA },
  columnWidths: [3120, 3120, 3120],
  rows: [
    new TableRow({
      children: ["30 Questions", "5 Topic Areas", "L2 Difficulty"].map(label => new TableCell({
        width: { size: 3120, type: WidthType.DXA },
        borders,
        shading: { fill: LIGHT_BLUE, type: ShadingType.CLEAR },
        margins: { top: 120, bottom: 120, left: 160, right: 160 },
        children: [new Paragraph({
          alignment: AlignmentType.CENTER,
          children: [new TextRun({ text: label, bold: true, size: 24, color: BLUE, font: "Arial" })]
        })]
      }))
    })
  ]
}));

docChildren.push(spacer());
docChildren.push(spacer());

// Sections
for (const sec of sections_data) {
  docChildren.push(heading2(sec.title));
  docChildren.push(sectionLabel(sec.label));
  docChildren.push(spacer());

  for (const q of sec.questions) {
    docChildren.push(questionCard(q.num, q.q, q.a, q.tip, q.code));
    docChildren.push(spacer());
  }
}

const doc = new Document({
  styles: {
    default: {
      document: { run: { font: "Arial", size: 22 } }
    },
    paragraphStyles: [
      {
        id: "Heading1", name: "Heading 1", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 36, bold: true, font: "Arial", color: BLUE },
        paragraph: { spacing: { before: 360, after: 180 }, outlineLevel: 0 }
      },
      {
        id: "Heading2", name: "Heading 2", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 28, bold: true, font: "Arial", color: MID_BLUE },
        paragraph: { spacing: { before: 280, after: 120 }, outlineLevel: 1 }
      }
    ]
  },
  sections: [{
    properties: {
      page: {
        size: { width: 12240, height: 15840 },
        margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 }
      }
    },
    footers: {
      default: new Footer({
        children: [new Paragraph({
          alignment: AlignmentType.CENTER,
          children: [
            new TextRun({ text: "Ensono DevOps L2 Interview Guide  |  Page ", size: 18, color: "888780", font: "Arial" }),
            new PageNumber()
          ]
        })]
      })
    },
    children: docChildren
  }]
});

Packer.toBuffer(doc).then(buf => {
  fs.writeFileSync("/mnt/user-data/outputs/Ensono_DevOps_L2_Interview_Guide.docx", buf);
  console.log("Done");
});

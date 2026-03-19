# Software Factory Core

The Factory Core provides the reusable automation primitives — CI/CD templates, Infrastructure-as-Code modules, policy engines, and service catalogs — that underpin every golden path. These components are centrally maintained, versioned, and consumed by application teams through the Developer Experience layer.

---

## CI/CD Templates

CI/CD templates encapsulate pipeline logic as reusable, versioned artifacts. Application teams consume them rather than authoring pipelines from scratch, ensuring consistency in build, test, scan, and deploy stages across the organization.

### Jenkins Shared Libraries

- **Description** — Groovy-based libraries loaded at pipeline runtime via `@Library` annotations. They define standardized stages (build, SAST, container scan, deploy, smoke test) that individual Jenkinsfiles invoke with minimal configuration.
- **Key Features**
  - Centrally versioned in a dedicated Git repository with semantic versioning.
  - Dynamic stage injection based on project metadata (language, deployment target, compliance tier).
  - Credential management through Jenkins credential store with short-lived tokens where possible.
  - Supports both scripted and declarative pipeline syntax.
- **Architecture Fit** — Jenkins shared libraries are consumed by teams that rely on Jenkins as their CI/CD engine. The libraries call into IaC modules for infrastructure provisioning and invoke security scanning tools (Semgrep, Trivy, Checkov) as pipeline stages.

### GitHub Actions Reusable Workflows

- **Description** — Centralized workflow definitions stored in a `.github` organization repository and referenced via `uses:` in downstream repositories. Supports composite actions for fine-grained reuse.
- **Key Features**
  - Called workflows with defined inputs, outputs, and secrets for clean separation of concerns.
  - Matrix strategies for multi-platform, multi-version builds.
  - OpenID Connect (OIDC) integration for keyless authentication to AWS, Azure, and GCP.
  - Branch protection rules enforce required workflow checks before merge.
- **Architecture Fit** — GitHub Actions reusable workflows are the primary CI/CD mechanism for GitHub-hosted repositories. They integrate natively with GitHub's security features (Dependabot, code scanning, secret scanning) and with Backstage for status visibility.

### Azure Pipelines Templates

- **Description** — YAML templates (`extends`, `stages`, `jobs`, `steps`) stored in a central Azure DevOps repository and referenced via `resources.repositories` in downstream pipeline definitions.
- **Key Features**
  - Template expressions and conditional insertion for environment-specific logic.
  - Service connections for federated authentication to Azure, AWS, and GCP.
  - Environments with approval gates and deployment history tracking.
  - Integration with Azure Artifacts for package management (NuGet, npm, Maven, Python).
- **Architecture Fit** — Azure Pipelines templates serve organizations that standardize on Azure DevOps. They leverage Azure-native integrations (Managed Identity, Key Vault, Azure Policy) while maintaining cross-cloud deployment capability.

### GitLab CI Includes

- **Description** — Reusable CI/CD configuration via `include:` directives that pull from central template repositories, remote URLs, or GitLab's template library.
- **Key Features**
  - `include:template`, `include:project`, and `include:remote` for flexible composition.
  - Parent-child and multi-project pipelines for complex orchestration.
  - Auto DevOps as a zero-configuration baseline that teams can extend.
  - Built-in container registry, package registry, and dependency scanning.
- **Architecture Fit** — GitLab CI includes support organizations using GitLab's integrated DevSecOps platform. The single-application model simplifies integration between source control, CI/CD, security scanning, and artifact management.

---

## Infrastructure-as-Code Modules

IaC modules are versioned, tested, and policy-compliant building blocks for cloud infrastructure. They abstract provider-specific complexity behind standardized interfaces.

### Terraform Registry & Module Strategy

- **Description** — A private Terraform registry (Terraform Cloud/Enterprise, Artifactory, or S3-backed) that hosts organization-approved modules. Modules follow a standard structure (`main.tf`, `variables.tf`, `outputs.tf`, `README.md`, `examples/`, `tests/`).
- **Official Reference:** [https://registry.terraform.io](https://registry.terraform.io)
- **Key Features**
  - Semantic versioning with changelog and migration guides.
  - Automated testing via Terratest or `terraform test` (native testing framework).
  - Pre-commit hooks for `terraform fmt`, `terraform validate`, and Checkov/tfsec scanning.
  - Module composition — higher-order modules compose base modules (e.g., an "application stack" module composes VPC + EKS + RDS).

### Core Module Library

| Module Category | AWS | Azure | GCP |
|---|---|---|---|
| **Networking** | VPC, Transit Gateway, PrivateLink | VNet, Virtual WAN, Private Endpoint | VPC, Shared VPC, Private Service Connect |
| **Compute — Kubernetes** | EKS (managed node groups, Fargate profiles) | AKS (system/user node pools, KEDA) | GKE Autopilot, GKE Standard |
| **Compute — Serverless** | Lambda, Fargate | Azure Functions, Container Apps | Cloud Run, Cloud Functions |
| **Data — Relational** | RDS (PostgreSQL, MySQL), Aurora | Azure SQL, Azure Database for PostgreSQL | Cloud SQL, AlloyDB, Spanner |
| **Data — NoSQL** | DynamoDB | Cosmos DB | Firestore, Bigtable |
| **AI/ML** | SageMaker (endpoints, pipelines, feature store) | Azure ML (workspaces, endpoints, pipelines) | Vertex AI (endpoints, pipelines, feature store) |
| **Storage** | S3, EFS | Blob Storage, Azure Files | GCS, Filestore |
| **Messaging** | SQS, SNS, EventBridge | Service Bus, Event Hubs, Event Grid | Pub/Sub, Eventarc |

### Architecture Fit

IaC modules are invoked by Backstage Software Templates during provisioning and by CI/CD pipelines during infrastructure deployment. Every module is validated against the Policy Engine before apply, ensuring compliance at the infrastructure layer.

---

## Policy Engine

The Policy Engine enforces organizational, security, and compliance rules as code. Policies are evaluated at multiple points: pre-commit, CI/CD pipeline, infrastructure plan, admission control, and runtime.

### Open Policy Agent (OPA)

- **Official Site:** [https://www.openpolicyagent.org](https://www.openpolicyagent.org)
- **Description** — General-purpose policy engine that decouples policy decisions from policy enforcement. Policies are written in Rego, a purpose-built declarative query language.
- **Key Features**
  - Policies as code — versioned, tested, and reviewed like application code.
  - Rego language supports complex logic over structured data (JSON, YAML, Terraform plans, Kubernetes manifests).
  - Conftest for pre-deployment policy checks against Terraform plans, Dockerfiles, and Kubernetes manifests.
  - Gatekeeper (OPA on Kubernetes) for admission control with constraint templates.
  - Decision logging for audit trails.
- **Architecture Fit** — OPA is the cross-cloud policy engine. It evaluates Terraform plans before apply, Kubernetes manifests before admission, and API requests at runtime. Rego policies encode controls from NIST SP 800-53, CIS Benchmarks, and organizational standards.

### Kyverno

- **Description** — Kubernetes-native policy engine that uses familiar YAML syntax rather than a custom language. Policies are Kubernetes custom resources.
- **Key Features**
  - Validate, mutate, generate, and verify image signatures on Kubernetes resources.
  - No custom language — policies are written in YAML with JMESPath expressions.
  - Policy reports for visibility into compliance status across clusters.
  - Integration with Sigstore/cosign for supply chain security.
- **Architecture Fit** — Kyverno complements OPA/Gatekeeper in Kubernetes-centric environments. Its lower learning curve makes it suitable for teams that need admission control without Rego expertise.

### Cloud-Native Policy Services

- **AWS Service Control Policies (SCPs)** — Organization-level guardrails that set permission boundaries across all accounts in an AWS Organization. Used to enforce region restrictions, deny dangerous services, and require encryption.
- **Azure Policy** — Built-in and custom policy definitions assigned at management group, subscription, or resource group scope. Supports deny, audit, append, modify, and deployIfNotExists effects.
- **GCP Organization Policies** — Constraints applied at organization, folder, or project level to restrict resource configurations (e.g., allowed VM machine types, external IP restrictions).

### Architecture Fit

Cloud-native policy services provide the outermost guardrails. OPA/Kyverno operate within the CI/CD pipeline and Kubernetes clusters. Together, they form a defense-in-depth policy architecture: cloud-level guardrails prevent the most dangerous misconfigurations, pipeline-level policies catch issues before deployment, and admission controllers enforce compliance at runtime.

---

## Service Catalog (Cloud-Native)

Cloud provider service catalogs offer curated, pre-approved products that abstract raw infrastructure behind governed interfaces.

### AWS Service Catalog

- **Description** — Enables platform teams to create portfolios of approved CloudFormation or Terraform products. Application teams launch products through a self-service interface with constrained parameters.
- **Key Features**
  - Portfolio sharing across accounts via AWS Organizations.
  - Launch constraints for least-privilege provisioning.
  - TagOptions for mandatory tagging enforcement.
  - Terraform integration via the AWS Service Catalog Engine for Terraform.

### Azure Managed Applications

- **Description** — Packaged ARM/Bicep templates published to a service catalog or Azure Marketplace. Consumers deploy applications without access to underlying resources.
- **Key Features**
  - Managed resource groups isolate infrastructure from consumers.
  - Publisher controls lifecycle (updates, patches) of managed resources.
  - Custom UI definitions for portal-based deployment experiences.

### GCP Service Directory

- **Description** — Managed service registry for discovering and connecting services across environments. Complements GCP's Deployment Manager and Config Controller for catalog-style provisioning.
- **Key Features**
  - DNS-based and API-based service resolution.
  - Namespace isolation for multi-tenant environments.
  - Integration with GKE, Cloud Run, and Traffic Director.

### Architecture Fit

Cloud-native service catalogs extend the self-service model to teams that may not use Backstage directly or need cloud-console-native workflows. They are populated with products built from the same IaC modules used elsewhere in the factory, ensuring consistency.

---

## Further Reading

- [Terraform Module Development Best Practices](https://developer.hashicorp.com/terraform/language/modules/develop)
- [Terraform Testing Framework](https://developer.hashicorp.com/terraform/language/tests)
- [OPA Documentation](https://www.openpolicyagent.org/docs/latest/)
- [Rego Policy Language Reference](https://www.openpolicyagent.org/docs/latest/policy-language/)
- [Kyverno Documentation](https://kyverno.io/docs/)
- [GitHub Actions — Reusing Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Azure Pipelines YAML Templates](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/templates)
- [GitLab CI/CD Includes](https://docs.gitlab.com/ee/ci/yaml/includes.html)
- [Jenkins Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
- [AWS Service Catalog Documentation](https://docs.aws.amazon.com/servicecatalog/)
- [Azure Managed Applications](https://learn.microsoft.com/en-us/azure/azure-resource-manager/managed-applications/)
- [GCP Service Directory](https://cloud.google.com/service-directory/docs)

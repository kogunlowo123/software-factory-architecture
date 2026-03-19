# Developer Experience Layer

The Developer Experience (DevEx) layer is the primary interface between engineering teams and the Software Factory. It abstracts infrastructure complexity behind self-service workflows, enforces organizational standards through golden paths, and provides a unified portal for service ownership, documentation, and provisioning.

---

## Backstage — Internal Developer Portal

**Official Site:** [https://backstage.io](https://backstage.io)

Backstage is an open-source platform originally developed at Spotify for building developer portals. It serves as the single pane of glass for all infrastructure, services, and documentation within the organization.

### Key Features

- **Service Catalog** — Centralized registry of all services, libraries, data pipelines, and infrastructure components with ownership metadata, lifecycle status, and dependency graphs.
- **Software Templates (Scaffolder)** — Codified project generators that produce repositories pre-wired with CI/CD pipelines, IaC modules, security scanning, and observability instrumentation. Templates enforce organizational conventions from day zero.
- **TechDocs** — Docs-as-code system that renders Markdown documentation stored alongside source code. Enables a "docs like code" workflow with pull request reviews and versioning.
- **Plugin Ecosystem** — Extensible architecture with 100+ community plugins covering Kubernetes, CI/CD status, cost dashboards, incident management, and more.
- **Search** — Unified search across the catalog, documentation, and integrated tooling, reducing context-switching for developers.

### Architecture Fit

Backstage acts as the front door to the entire Software Factory. It aggregates data from GitHub/GitLab, CI/CD systems, cloud providers, and observability platforms into a single experience. Software Templates call into the Factory Core (CI/CD templates, IaC modules, policy engines) to provision production-ready services in minutes rather than weeks.

---

## Self-Service Provisioning Patterns

Self-service provisioning eliminates ticket-driven workflows by giving developers the ability to create, configure, and manage infrastructure and services through declarative interfaces with built-in guardrails.

### Key Patterns

- **Template-Driven Provisioning** — Backstage Scaffolder templates backed by Terraform modules and CI/CD pipeline definitions. Developers fill in a form; the factory handles the rest.
- **GitOps-Based Provisioning** — Pull request to a declarative repository triggers automated provisioning. ArgoCD or Flux reconcile desired state with actual state.
- **API-Driven Provisioning** — Internal platform APIs (often fronted by Backstage plugins) that accept service specifications and orchestrate multi-cloud resource creation.
- **Catalog-Based Provisioning** — AWS Service Catalog, Azure Managed Applications, or GCP Service Directory products offered as curated, pre-approved building blocks.

### Architecture Fit

Self-service provisioning is the execution mechanism behind golden paths. Every provisioning action flows through the Policy Engine (OPA, Kyverno) to ensure compliance before any resource is created.

---

## Golden Paths / Paved Roads

Golden paths (also called paved roads) are opinionated, well-supported workflows that represent the recommended way to build and ship software within the organization. They are not mandates — teams can diverge — but the golden path offers the fastest, safest, most compliant route to production.

### Key Characteristics

- **Pre-integrated Toolchain** — Each golden path bundles a language runtime, framework, CI/CD pipeline, IaC module, security scanning configuration, and observability instrumentation.
- **Compliance Built In** — Golden paths satisfy NIST SP 800-53, FedRAMP, HIPAA/HITRUST, and CMMC Level 2 controls out of the box, reducing audit burden for individual teams.
- **Versioned and Maintained** — Golden paths are treated as products with SLOs, changelogs, and migration guides when breaking changes occur.
- **Measurable Adoption** — DORA metrics (deployment frequency, lead time, change failure rate, MTTR) are tracked per golden path to demonstrate value.
- **Escape Hatches** — Teams that need to diverge accept responsibility for meeting compliance and operational requirements independently, typically through an architecture review process.

### Architecture Fit

Golden paths are the highest-level abstraction in the Software Factory. They compose elements from every layer: Backstage templates (DevEx), CI/CD and IaC modules (Factory Core), scanning tools (Security & Compliance), telemetry instrumentation (Observability), and target cloud services (Cloud Execution).

---

## Integration with Source Control & DevOps Platforms

The Developer Experience layer integrates with the organization's source control and DevOps platforms to provide a unified view of code, pipelines, and deployments.

### GitHub

- Backstage GitHub plugins surface repository metadata, pull request status, Actions workflow runs, Dependabot alerts, and code scanning findings.
- Software Templates create repositories via the GitHub API with branch protection rules, CODEOWNERS files, and required status checks pre-configured.
- GitHub Apps provide fine-grained authentication for Backstage catalog ingestion and template execution.

### GitLab

- Backstage GitLab plugins integrate with the GitLab API for project discovery, pipeline status, merge request tracking, and container registry metadata.
- Software Templates leverage GitLab project creation APIs and CI/CD variable injection.
- GitLab's built-in container registry and package registry integrate with the security scanning layer.

### Azure DevOps

- Backstage Azure DevOps plugins surface repositories, build pipelines, release pipelines, and work item status.
- Software Templates create Azure DevOps projects with pre-configured pipeline YAML definitions and service connections.
- Azure Boards integration enables traceability from work items through code changes to production deployments.

### Architecture Fit

Source control platform integration is foundational — it connects the developer's daily workflow (commits, pull requests, code reviews) to the factory's automation (CI/CD triggers, policy checks, artifact publishing). The choice of platform is organizational; the Software Factory abstracts differences behind Backstage's unified catalog model.

---

## Further Reading

- [Backstage Documentation](https://backstage.io/docs)
- [Backstage Software Templates](https://backstage.io/docs/features/software-templates/)
- [Backstage TechDocs](https://backstage.io/docs/features/techdocs/)
- [Backstage Plugin Marketplace](https://backstage.io/plugins)
- [Spotify Engineering — Backstage Origin Story](https://engineering.atspotify.com/2020/03/what-the-heck-is-backstage-anyway/)
- [Gartner — Internal Developer Portals](https://www.gartner.com/en/documents/4017457)
- [CNCF Platforms White Paper](https://tag-app-delivery.cncf.io/whitepapers/platforms/)
- [GitHub REST API Documentation](https://docs.github.com/en/rest)
- [GitLab API Documentation](https://docs.gitlab.com/ee/api/)
- [Azure DevOps REST API Documentation](https://learn.microsoft.com/en-us/rest/api/azure/devops/)

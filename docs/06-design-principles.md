# Design Principles & Business Value

This document articulates the architectural philosophy behind the Software Factory and maps its capabilities to measurable business outcomes. The principles guide every design decision, from tool selection to golden path construction to organizational adoption strategy.

---

## Core Design Principles

### 1. Cloud-Agnostic Control Plane, Cloud-Native Execution

The Software Factory separates the control plane (CI/CD orchestration, policy enforcement, observability aggregation, developer portal) from the execution plane (cloud provider services). The control plane is portable and cloud-agnostic; the execution plane is deliberately cloud-native.

- **Control Plane** — Backstage, GitHub Actions/GitLab CI, OPA, OpenTelemetry, Prometheus/Thanos, Grafana. These run on any Kubernetes cluster or as SaaS and are not bound to a specific cloud provider.
- **Execution Plane** — EKS/AKS/GKE, Lambda/Functions/Cloud Run, RDS/Azure SQL/Cloud SQL, SageMaker/Azure ML/Vertex AI. These leverage provider-specific optimizations, SLAs, and pricing models.
- **Rationale** — Organizations operate in multi-cloud and hybrid realities. Attempting to abstract the execution plane (e.g., running all workloads on Kubernetes regardless of fit) sacrifices performance and cost efficiency. Abstracting the control plane preserves governance consistency while allowing teams to use the best execution target for their workload.

### 2. Security & Compliance by Default (Shift-Left)

Security controls are embedded into every layer of the Software Factory, not bolted on after the fact. The default state of any service provisioned through a golden path is compliant.

- **Pre-commit** — GitLeaks for secrets detection, IDE plugins for Semgrep and tfsec.
- **CI/CD Pipeline** — SAST (Semgrep, CodeQL), SCA (Snyk, Trivy), IaC scanning (Checkov, tfsec), DAST (OWASP ZAP), container scanning (Grype, Docker Scout).
- **Admission Control** — OPA Gatekeeper or Kyverno enforce policies at Kubernetes admission time.
- **Runtime** — Falco, GuardDuty, Defender for Cloud provide continuous threat detection.
- **Supply Chain** — SLSA Level 3 provenance, SBOM generation, image signing with Sigstore/cosign.
- **Rationale** — The cost of fixing a vulnerability increases 10-100x as it moves from development to production. Shift-left security reduces remediation cost, accelerates delivery (fewer late-stage findings), and provides continuous compliance evidence for auditors.

### 3. Self-Service with Guardrails

Developers operate independently within well-defined boundaries. The Software Factory provides maximum autonomy within a governed perimeter.

- **Self-Service** — Backstage templates for provisioning services, infrastructure, and environments. No tickets required for standard operations.
- **Guardrails** — Policy Engine (OPA, Kyverno, AWS SCPs, Azure Policy) enforces organizational constraints. Guardrails are transparent — developers see what is allowed and why constraints exist.
- **Escape Hatches** — Teams with legitimate needs outside golden paths can request exceptions through an architecture review process. Approved exceptions are documented and monitored.
- **Rationale** — Centralizing all infrastructure operations in a platform team creates a bottleneck. Pure self-service without guardrails creates security and compliance risk. The Software Factory balances velocity and governance.

### 4. Opinionated but Extensible

The factory provides strong defaults (opinions) while allowing customization for teams with advanced needs.

- **Opinions** — Default language runtimes, frameworks, CI/CD pipeline stages, infrastructure patterns, and observability configurations. These represent the "golden path" — the fastest, safest route to production.
- **Extensibility** — Plugin architecture in Backstage, custom OPA/Kyverno policies, composable Terraform modules, extensible CI/CD templates with hook points for additional stages.
- **Progressive Disclosure** — New teams use golden paths as-is. As teams mature, they customize pipeline stages, add scanning tools, or create new golden paths for novel workload types.
- **Rationale** — Overly rigid platforms drive shadow IT. Overly flexible platforms provide no value over raw cloud APIs. The right balance is strong defaults with documented extension points.

---

## Business Value

### Faster Delivery

- **Metric** — Deployment frequency increases from monthly/quarterly to daily/on-demand. Lead time for changes decreases from weeks to hours.
- **Mechanism** — Golden paths eliminate per-project setup work. Self-service provisioning removes ticket-based bottlenecks. Automated testing and scanning provide rapid feedback.
- **Evidence** — DORA metrics tracked in Grafana dashboards demonstrate delivery performance trends over time.

### Built-In Security

- **Metric** — Mean time to remediate vulnerabilities decreases by 60-80%. Audit preparation time decreases from months to days.
- **Mechanism** — Shift-left scanning catches vulnerabilities before they reach production. Policy-as-code ensures compliance at provisioning time. Automated evidence collection provides continuous audit readiness.
- **Evidence** — Vulnerability aging reports, policy compliance dashboards, and audit evidence packages generated automatically.

### Cloud Flexibility

- **Metric** — Time to deploy a workload to a new cloud provider decreases from months to weeks. Vendor negotiation leverage improves through demonstrated portability.
- **Mechanism** — Cloud-agnostic control plane means governance, CI/CD, and observability work identically across providers. IaC modules abstract provider-specific details behind standardized interfaces.
- **Evidence** — Multi-cloud deployment inventory, cost optimization reports showing workload placement decisions based on price/performance.

### AI-Ready Platform

- **Metric** — Time to deploy an ML model from experiment to production decreases from months to days.
- **Mechanism** — Golden paths for ML workloads provision SageMaker/Azure ML/Vertex AI endpoints with model monitoring, A/B testing, and feature store integration. CI/CD for ML (MLOps) is a first-class factory capability.
- **Evidence** — Model deployment frequency, inference latency SLOs, model drift detection alerts.

### Full Visibility & Cost Governance

- **Metric** — Cloud waste reduced by 20-40%. Incident detection time (MTTD) decreases by 50%+.
- **Mechanism** — OpenTelemetry instrumentation provides end-to-end tracing. Prometheus/Thanos metrics enable SLO tracking and capacity planning. Cost data correlated with utilization identifies optimization opportunities.
- **Evidence** — SLO compliance reports, cost-per-service dashboards, capacity forecasting models.

---

## CNCF Landscape Alignment

The Software Factory preferentially selects CNCF Graduated and Incubating projects for its core components. This alignment provides confidence in project maturity, community health, and long-term viability.

| Factory Component | CNCF Project | Status |
|---|---|---|
| Instrumentation | OpenTelemetry | Incubating |
| Metrics | Prometheus | Graduated |
| Long-term Metrics | Thanos | Incubating |
| Distributed Tracing | Jaeger | Graduated |
| Container Orchestration | Kubernetes | Graduated |
| Service Mesh | Istio / Envoy | Graduated |
| GitOps | Flux / Argo CD | Graduated / Graduated |
| Runtime Security | Falco | Graduated |
| Policy | OPA | Graduated |
| Autoscaling | KEDA | Graduated |
| Artifact Management | Harbor | Graduated |
| Supply Chain | Sigstore (cosign, Rekor) | -- |
| Observability | Grafana (LGTM stack) | -- |
| Developer Portal | Backstage | Incubating |

### Rationale

CNCF projects share common governance, contribution, and security audit processes. Selecting from this landscape reduces integration friction (projects are designed to interoperate), minimizes vendor lock-in risk, and ensures access to a broad contributor community for support and innovation.

---

## Anti-Patterns to Avoid

Documenting what the factory intentionally avoids is as important as documenting what it adopts.

- **Lowest Common Denominator Abstraction** — Abstracting cloud services to the point where provider-specific advantages are lost. The factory abstracts governance, not execution.
- **Mandated Uniformity** — Forcing all teams onto a single language, framework, or deployment target. Golden paths are recommended, not required. Diversity is managed through guardrails, not prohibition.
- **Big Bang Platform Launch** — Attempting to deliver the entire factory at once. The factory is built incrementally, starting with the highest-value golden path and expanding based on adoption data and team feedback.
- **Platform Team as Bottleneck** — Centralizing all operational knowledge in the platform team. Golden paths include runbooks, and Backstage TechDocs ensure application teams can self-serve for operational tasks.
- **Ignoring Developer Experience** — Building governance tooling that developers work around rather than through. Every guardrail must be transparent, fast, and provide actionable feedback.

---

## Further Reading

- [CNCF Cloud Native Landscape](https://landscape.cncf.io/)
- [CNCF Platforms White Paper](https://tag-app-delivery.cncf.io/whitepapers/platforms/)
- [Team Topologies — Organizing Business and Technology Teams for Fast Flow](https://teamtopologies.com/)
- [DORA State of DevOps Reports](https://dora.dev/)
- [Google SRE Books](https://sre.google/books/)
- [NIST SP 800-53 Rev5](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
- [NIST SP 800-207 — Zero Trust Architecture](https://csrc.nist.gov/publications/detail/sp/800-207/final)
- [SLSA Framework](https://slsa.dev/)
- [Thoughtworks Technology Radar](https://www.thoughtworks.com/radar)
- [Platform Engineering on Kubernetes (Manning)](https://www.manning.com/books/platform-engineering-on-kubernetes)

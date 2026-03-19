# Security & Compliance

The Security & Compliance layer embeds security controls into every stage of the software delivery lifecycle — from code authoring through runtime. Rather than treating security as a gate at the end of the pipeline, the Software Factory shifts security left (into development) and extends it right (into production), ensuring continuous compliance with regulatory frameworks.

---

## Static Application Security Testing (SAST)

SAST tools analyze source code and compiled artifacts for security vulnerabilities without executing the application.

### Semgrep

- **Official Site:** [https://semgrep.dev](https://semgrep.dev)
- **Description** — Lightweight, fast static analysis tool that supports 30+ languages. Uses pattern-matching syntax that mirrors the target language, making rules intuitive for developers to write and review.
- **Key Features**
  - Language-aware pattern matching — rules look like the code they match, reducing false positives.
  - Semgrep Registry with 3,000+ community and pro rules covering OWASP Top 10, CWE, and framework-specific vulnerabilities.
  - Semgrep Supply Chain for reachability analysis — identifies whether a vulnerable dependency function is actually called.
  - CI/CD integration via `semgrep ci` with SARIF output for GitHub code scanning, GitLab SAST, and Azure DevOps.
  - Sub-second scan times for incremental analysis on pull requests.
- **Architecture Fit** — Semgrep runs as a pipeline stage in every CI/CD template. Custom rules encode organization-specific patterns (e.g., banned API usage, required authentication checks). Results surface in Backstage and developer IDEs via the Semgrep VS Code extension.

### CodeQL

- **Official Site:** [https://codeql.github.com](https://codeql.github.com)
- **Description** — Semantic code analysis engine developed by GitHub. Treats code as data by building a queryable database, enabling deep interprocedural analysis for taint tracking and data flow vulnerabilities.
- **Key Features**
  - Deep data-flow and taint-tracking analysis across function boundaries and module imports.
  - Supports Java, C/C++, C#, JavaScript/TypeScript, Python, Go, Ruby, Swift, and Kotlin.
  - Native GitHub integration — results appear as code scanning alerts in pull requests.
  - Custom queries in QL language for organization-specific vulnerability patterns.
  - Variant analysis for finding instances of a known vulnerability pattern across the codebase.
- **Architecture Fit** — CodeQL provides deeper analysis than pattern-matching tools at the cost of longer scan times. It runs on the default branch and on pull requests for critical repositories. Combined with Semgrep (fast, broad) and CodeQL (deep, precise), the factory achieves layered SAST coverage.

---

## Dynamic Application Security Testing (DAST)

DAST tools test running applications from the outside by sending crafted requests and analyzing responses for vulnerabilities.

### OWASP ZAP

- **Official Site:** [https://www.zaproxy.org](https://www.zaproxy.org)
- **Description** — Open-source web application security scanner maintained by the OWASP Foundation. Supports automated scanning, manual testing, and API fuzzing.
- **Key Features**
  - Active and passive scanning modes — passive scanning observes traffic without modification; active scanning probes for vulnerabilities.
  - OpenAPI, GraphQL, and SOAP import for API-centric testing.
  - Authentication support (form-based, token-based, OAuth) for scanning authenticated surfaces.
  - CI/CD integration via Docker container, GitHub Actions, and CLI.
  - AJAX Spider for single-page application crawling.
- **Architecture Fit** — ZAP runs against staging/preview environments deployed during the CI/CD pipeline. It complements SAST by finding runtime vulnerabilities (authentication bypass, CORS misconfiguration, header issues) that static analysis cannot detect. Results are correlated with SAST findings in a centralized vulnerability management dashboard.

---

## Software Composition Analysis (SCA) & Dependency Scanning

SCA tools identify known vulnerabilities, license risks, and outdated dependencies in third-party libraries.

### Snyk

- **Official Site:** [https://snyk.io](https://snyk.io)
- **Description** — Developer-first security platform covering open-source dependencies, container images, IaC configurations, and proprietary code.
- **Key Features**
  - Vulnerability database with proprietary research and remediation advice including automated fix pull requests.
  - License compliance scanning with configurable policies (deny GPL, allow MIT/Apache, etc.).
  - Container image scanning with base image upgrade recommendations.
  - IDE plugins (VS Code, IntelliJ, Visual Studio) for pre-commit vulnerability detection.
  - Snyk API for integration with custom dashboards and reporting workflows.
- **Architecture Fit** — Snyk runs in CI/CD pipelines as a gate for dependency vulnerabilities. Its monitor capability provides continuous monitoring of deployed dependencies. License policies are enforced by the Policy Engine to ensure open-source compliance.

### Trivy

- **Official Site:** [https://trivy.dev](https://trivy.dev)
- **Description** — Comprehensive, open-source vulnerability scanner by Aqua Security. Scans container images, file systems, Git repositories, Kubernetes clusters, and cloud infrastructure.
- **Key Features**
  - All-in-one scanner: OS packages, language-specific dependencies, IaC misconfigurations, secrets, and licenses.
  - SBOM generation in CycloneDX and SPDX formats for supply chain transparency.
  - Kubernetes operator (Trivy Operator) for continuous in-cluster scanning.
  - VEX (Vulnerability Exploitability eXchange) support for suppressing non-applicable findings.
  - Offline mode with downloadable vulnerability database for air-gapped environments.
- **Architecture Fit** — Trivy is the open-source backbone of container and dependency scanning. It runs in CI/CD pipelines alongside Snyk (which provides additional proprietary intelligence and remediation). The Trivy Operator provides runtime scanning in Kubernetes clusters.

---

## Infrastructure-as-Code Scanning

IaC scanning tools detect misconfigurations in Terraform, CloudFormation, Kubernetes manifests, Dockerfiles, and Helm charts before deployment.

### Checkov

- **Official Site:** [https://www.checkov.io](https://www.checkov.io)
- **Description** — Open-source static analysis tool for IaC developed by Prisma Cloud (Palo Alto Networks). Supports Terraform, CloudFormation, Kubernetes, Helm, ARM, Bicep, Serverless Framework, and Dockerfile.
- **Key Features**
  - 1,000+ built-in policies mapped to CIS Benchmarks, NIST SP 800-53, PCI-DSS, HIPAA, and SOC 2.
  - Terraform plan scanning for evaluating computed values and dynamic configurations.
  - Graph-based analysis for cross-resource relationship checks (e.g., "S3 bucket referenced by CloudFront distribution must have encryption enabled").
  - Custom policies in Python or YAML.
  - SARIF, JUnit, and JSON output formats for CI/CD integration.
- **Architecture Fit** — Checkov runs in the CI/CD pipeline after `terraform plan` and before `terraform apply`. It validates that IaC modules comply with organizational and regulatory policies. Custom policies enforce the factory's infrastructure standards.

### tfsec

- **Official Site:** [https://aquasecurity.github.io/tfsec](https://aquasecurity.github.io/tfsec)
- **Description** — Terraform-specific static analysis tool by Aqua Security (now integrated into Trivy). Focuses on AWS, Azure, and GCP misconfigurations with low false-positive rates.
- **Key Features**
  - Terraform HCL and plan JSON analysis with module resolution.
  - Custom rules via Rego (OPA) or YAML definitions.
  - IDE integration (VS Code extension) for real-time feedback during authoring.
  - Detailed remediation guidance with Terraform code examples.
  - Supports tfvars files for environment-specific evaluation.
- **Architecture Fit** — tfsec provides Terraform-focused scanning with a developer-friendly experience. It runs as a pre-commit hook and CI/CD stage. Its integration into Trivy means organizations can consolidate on a single tool for container, dependency, and IaC scanning.

---

## Secrets Detection

### GitLeaks

- **Official Site:** [https://gitleaks.io](https://gitleaks.io)
- **Description** — Open-source tool for detecting hardcoded secrets (API keys, passwords, tokens, certificates) in Git repositories. Scans commit history and staged changes.
- **Key Features**
  - Pre-commit hook integration to prevent secrets from entering the repository.
  - Full Git history scanning for detecting previously committed secrets.
  - Configurable rules with regex patterns and entropy-based detection.
  - Allowlist support for false positive suppression.
  - SARIF output for integration with GitHub code scanning and CI/CD pipelines.
- **Architecture Fit** — GitLeaks runs at two points: as a pre-commit hook on developer workstations and as a CI/CD pipeline stage. Any detected secret triggers a pipeline failure and an alert to the security team. Remediation includes secret rotation and Git history rewriting when necessary.

---

## Runtime Security

Runtime security tools monitor running workloads for anomalous behavior, threats, and policy violations.

### Falco

- **Official Site:** [https://falco.org](https://falco.org)
- **Description** — Cloud-native runtime security project (CNCF Graduated) that detects threats using system call monitoring. Originally created by Sysdig.
- **Key Features**
  - Kernel-level visibility via eBPF or kernel module for system call interception.
  - Rules engine with 50+ default rules covering MITRE ATT&CK techniques (container escape, cryptomining, reverse shells, privilege escalation).
  - Plugins architecture for extending detection to cloud audit logs (AWS CloudTrail, GCP Audit Logs, Azure Activity Logs) and Kubernetes audit logs.
  - Falcosidekick for routing alerts to Slack, PagerDuty, OPA, and SIEM platforms.
  - Talon for automated response (kill container, isolate pod, trigger forensic capture).
- **Architecture Fit** — Falco runs as a DaemonSet on every Kubernetes node, providing runtime threat detection. It complements pre-deployment scanning (Trivy, Checkov) with runtime visibility. Alerts feed into the observability layer (OpenSearch/Splunk) and incident management workflows.

### AWS GuardDuty

- **Description** — Managed threat detection service that analyzes AWS CloudTrail, VPC Flow Logs, DNS logs, EKS audit logs, and S3 data events using machine learning and threat intelligence.
- **Key Features**
  - Zero-configuration deployment — enable per account and region.
  - Malware scanning for EBS volumes attached to EC2 instances and container workloads.
  - EKS Runtime Monitoring for Kubernetes-specific threats.
  - Multi-account aggregation via AWS Organizations delegated administrator.
  - Automated remediation via EventBridge rules triggering Lambda functions or Step Functions.
- **Architecture Fit** — GuardDuty provides AWS-native threat detection that supplements Falco's container-level monitoring with cloud-control-plane visibility.

### Microsoft Defender for Cloud

- **Description** — Cloud-native application protection platform (CNAPP) providing security posture management (CSPM) and workload protection (CWP) across Azure, AWS, and GCP.
- **Key Features**
  - Secure Score with prioritized remediation recommendations.
  - Defender for Containers — runtime threat detection, vulnerability assessment, and admission control.
  - Defender for Servers — endpoint detection and response (EDR) via Microsoft Defender for Endpoint.
  - Regulatory compliance dashboard with built-in assessments for NIST, CIS, PCI-DSS, HIPAA, and ISO 27001.
  - Multi-cloud support — extends coverage to AWS and GCP workloads.
- **Architecture Fit** — Defender for Cloud provides the Azure-native security layer and can extend to multi-cloud environments. Its CSPM capabilities complement the Policy Engine by providing continuous posture assessment against compliance frameworks.

---

## Container Security

### Docker Scout

- **Description** — Docker's native supply chain security tool integrated into Docker Desktop and Docker Hub. Analyzes container images for vulnerabilities, provides remediation guidance, and tracks policy compliance.
- **Key Features**
  - Real-time image analysis in Docker Desktop during development.
  - Policy evaluation for organizational standards (no critical CVEs, approved base images only).
  - SBOM generation and VEX document support.
  - Integration with GitHub Actions and CI/CD pipelines.

### Grype

- **Description** — Open-source vulnerability scanner for container images and filesystems by Anchore. Fast, offline-capable, and designed for CI/CD integration.
- **Key Features**
  - Scans OCI images, Docker archives, and directories.
  - Pairs with Syft for SBOM generation (CycloneDX, SPDX).
  - Configurable severity thresholds and ignore rules.
  - Database updates via ORAS-based OCI artifact distribution.

### Architecture Fit

Container security tools run at multiple stages: during development (Docker Scout), in CI/CD pipelines (Grype, Trivy), and continuously in registries and clusters (Trivy Operator). This layered approach ensures vulnerabilities are caught at the earliest possible point.

---

## Compliance Frameworks

The Software Factory maps its controls to multiple regulatory and industry frameworks, enabling organizations to demonstrate compliance through automated evidence collection.

| Framework | Scope | Key Controls |
|---|---|---|
| **NIST SP 800-53 Rev5** | Federal information systems | 1,189 controls across 20 families (AC, AU, CA, CM, CP, IA, IR, MA, MP, PE, PL, PM, PS, PT, RA, SA, SC, SI, SR, Supply Chain) |
| **FedRAMP High** | Cloud services for federal agencies | NIST SP 800-53 High baseline (421 controls) with continuous monitoring requirements |
| **HIPAA/HITRUST** | Healthcare data protection | Administrative, physical, and technical safeguards for PHI; HITRUST CSF maps to HIPAA, NIST, ISO 27001 |
| **CMMC Level 2** | Defense industrial base | 110 practices from NIST SP 800-171 Rev2 for protecting Controlled Unclassified Information (CUI) |
| **CIS Benchmarks** | Infrastructure hardening | Prescriptive configuration guides for OS, Kubernetes, cloud providers, databases, and network devices |
| **SLSA Level 3** | Software supply chain integrity | Source, build, provenance, and common requirements; hermetic builds with non-falsifiable provenance |
| **Zero Trust (NIST SP 800-207)** | Network architecture | Never trust, always verify; micro-segmentation; continuous authentication; least-privilege access |

### Architecture Fit

Compliance is not a separate activity — it is encoded into every layer of the Software Factory. IaC modules implement CIS Benchmarks. CI/CD templates enforce SLSA Level 3 provenance. The Policy Engine evaluates NIST SP 800-53 controls. Runtime security tools provide continuous monitoring for FedRAMP and HIPAA. Evidence is collected automatically and stored for audit purposes.

---

## Further Reading

- [Semgrep Documentation](https://semgrep.dev/docs/)
- [CodeQL Documentation](https://codeql.github.com/docs/)
- [OWASP ZAP Documentation](https://www.zaproxy.org/docs/)
- [Snyk Documentation](https://docs.snyk.io/)
- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [Checkov Documentation](https://www.checkov.io/1.Welcome/Quick%20Start.html)
- [tfsec Documentation](https://aquasecurity.github.io/tfsec/)
- [GitLeaks Documentation](https://github.com/gitleaks/gitleaks)
- [Falco Documentation](https://falco.org/docs/)
- [AWS GuardDuty Documentation](https://docs.aws.amazon.com/guardduty/)
- [Microsoft Defender for Cloud Documentation](https://learn.microsoft.com/en-us/azure/defender-for-cloud/)
- [NIST SP 800-53 Rev5](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
- [NIST SP 800-207 — Zero Trust Architecture](https://csrc.nist.gov/publications/detail/sp/800-207/final)
- [SLSA Framework](https://slsa.dev/)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)
- [FedRAMP Authorization Process](https://www.fedramp.gov/)
- [HITRUST CSF](https://hitrustalliance.net/csf/)

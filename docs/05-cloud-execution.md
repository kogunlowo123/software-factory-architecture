# Cloud Execution Layer

The Cloud Execution Layer is where workloads run. The Software Factory maintains a cloud-agnostic control plane (CI/CD, policy, observability) while leveraging cloud-native services for execution. This approach maximizes the operational and cost advantages of each cloud provider without sacrificing portability at the automation and governance layers.

---

## Amazon Web Services (AWS)

### EKS — Elastic Kubernetes Service

- **Official Site:** [https://aws.amazon.com/eks](https://aws.amazon.com/eks)
- **Description** — Managed Kubernetes control plane with integrations across the AWS ecosystem. Supports EC2 managed node groups, Fargate serverless nodes, and Karpenter for intelligent autoscaling.
- **Key Features**
  - Managed control plane with automatic Kubernetes version upgrades and security patching.
  - Managed node groups and Karpenter for right-sized, cost-optimized compute provisioning.
  - Fargate profiles for serverless pod execution without node management.
  - IAM Roles for Service Accounts (IRSA) and EKS Pod Identity for fine-grained workload authentication.
  - AWS Load Balancer Controller for ALB/NLB ingress with WAF integration.
- **Architecture Fit** — EKS is the primary container orchestration platform on AWS. IaC modules provision clusters with Karpenter, AWS Load Balancer Controller, EBS CSI driver, and observability agents pre-installed. Kyverno/OPA Gatekeeper enforces admission policies.

### Lambda

- **Description** — Serverless compute service for event-driven functions. Supports Node.js, Python, Java, .NET, Go, Ruby, and custom runtimes via container images.
- **Key Features**
  - Sub-second cold starts with SnapStart (Java) and provisioned concurrency.
  - Event source mappings for SQS, Kinesis, DynamoDB Streams, Kafka, and S3.
  - Lambda@Edge and CloudFront Functions for edge compute.
  - Powertools for AWS Lambda — structured logging, tracing, metrics, and idempotency utilities.
  - 10 GB memory and 15-minute execution timeout.

### Fargate

- **Description** — Serverless compute engine for containers running on ECS or EKS. Eliminates node management while supporting standard container workloads.
- **Key Features**
  - Per-vCPU/memory pricing — pay only for resources consumed by pods.
  - EKS Fargate profiles for Kubernetes-native serverless pod scheduling.
  - ECS Fargate for standalone container tasks and services.
  - Integration with AWS PrivateLink for private networking without NAT gateways.

### RDS / Aurora

- **Description** — Managed relational database services supporting PostgreSQL, MySQL, MariaDB, Oracle, and SQL Server. Aurora provides MySQL/PostgreSQL-compatible engines with enhanced performance and availability.
- **Key Features**
  - Multi-AZ deployments with automatic failover.
  - Aurora Serverless v2 for variable workloads with per-ACU billing.
  - RDS Proxy for connection pooling and IAM-based database authentication.
  - Automated backups, point-in-time recovery, and cross-region read replicas.
  - Performance Insights for database performance monitoring.

### SageMaker

- **Description** — Managed ML platform covering the full model lifecycle: data preparation, training, tuning, hosting, and monitoring.
- **Key Features**
  - SageMaker Studio — integrated IDE for ML development.
  - SageMaker Pipelines — CI/CD for ML with step-based workflow definitions.
  - SageMaker Endpoints — real-time inference with auto-scaling and A/B testing.
  - SageMaker Feature Store — centralized feature repository for training and inference.
  - SageMaker Model Monitor — drift detection and data quality monitoring.

### Additional AWS Services

- **S3** — Object storage for artifacts, data lakes, backups, and static assets. Intelligent-Tiering for cost optimization.
- **SQS** — Managed message queue with standard (at-least-once) and FIFO (exactly-once) modes.
- **SNS** — Pub/sub messaging for fan-out patterns, mobile push, and email/SMS notifications.

---

## Microsoft Azure

### AKS — Azure Kubernetes Service

- **Official Site:** [https://azure.microsoft.com/en-us/products/kubernetes-service](https://azure.microsoft.com/en-us/products/kubernetes-service)
- **Description** — Managed Kubernetes with deep Azure integration. Supports system and user node pools, virtual nodes (ACI), and KEDA-based autoscaling.
- **Key Features**
  - Free control plane tier for standard workloads; uptime SLA tier for production.
  - Azure AD Workload Identity for pod-level Azure RBAC and Managed Identity access.
  - Azure CNI Overlay and Cilium-based networking for advanced network policies.
  - KEDA (Kubernetes Event-Driven Autoscaling) integration for scaling based on Azure Service Bus, Event Hubs, and custom metrics.
  - GitOps with Flux v2 as an AKS extension for declarative cluster configuration.
- **Architecture Fit** — AKS is the primary container orchestration platform on Azure. IaC modules provision clusters with Azure Policy integration, Azure Monitor Container Insights, and Defender for Containers pre-configured.

### Azure Functions

- **Description** — Event-driven serverless compute supporting C#, Java, JavaScript/TypeScript, Python, PowerShell, and custom handlers. Runs on Consumption, Premium, or Dedicated plans.
- **Key Features**
  - Durable Functions for stateful workflows (orchestrations, fan-out/fan-in, human interaction).
  - Event Grid, Service Bus, Event Hubs, Cosmos DB, and Blob Storage triggers.
  - Premium plan with VNET integration, pre-warmed instances, and unlimited execution duration.
  - Deployment slots for zero-downtime deployments and traffic splitting.

### Container Apps

- **Description** — Serverless container hosting built on Kubernetes and KEDA. Abstracts Kubernetes complexity for microservices, APIs, and event-driven workloads.
- **Key Features**
  - Scale-to-zero with KEDA-based autoscaling on HTTP, TCP, or custom metrics.
  - Dapr integration for service invocation, state management, pub/sub, and observability.
  - Revision management with traffic splitting for blue/green and canary deployments.
  - Managed identity and secret references from Azure Key Vault.

### Azure SQL & Cosmos DB

- **Azure SQL** — Managed SQL Server with Hyperscale tier for 100 TB+ databases, serverless compute tier for variable workloads, and elastic pools for multi-tenant SaaS.
- **Cosmos DB** — Globally distributed, multi-model NoSQL database with single-digit millisecond latency. APIs for NoSQL (document), MongoDB, Cassandra, Gremlin (graph), and Table.

### Azure ML

- **Description** — Managed ML platform with workspaces, compute clusters, endpoints, and pipelines.
- **Key Features**
  - Managed online endpoints with blue/green deployment and autoscaling.
  - ML pipelines with component-based authoring and reusable environments.
  - Responsible AI dashboard for model interpretability and fairness assessment.
  - Azure ML registries for cross-workspace model and environment sharing.

### Additional Azure Services

- **Blob Storage** — Object storage with hot, cool, cold, and archive tiers. Immutable storage for compliance.
- **Service Bus** — Enterprise message broker with queues and topics. Supports sessions, dead-lettering, and scheduled delivery.
- **Event Hubs** — Event streaming platform compatible with Apache Kafka. Captures events to Blob Storage or Data Lake for batch processing.

---

## Google Cloud Platform (GCP)

### GKE Autopilot

- **Official Site:** [https://cloud.google.com/kubernetes-engine](https://cloud.google.com/kubernetes-engine)
- **Description** — Fully managed Kubernetes with Google managing the control plane, nodes, and system components. Per-pod billing eliminates over-provisioning.
- **Key Features**
  - Node management fully automated — no node pools, no OS patching, no capacity planning.
  - Per-pod resource billing — pay only for CPU, memory, and ephemeral storage requested by pods.
  - Built-in security: shielded GKE nodes, Workload Identity, Binary Authorization, and Security Posture dashboard.
  - GKE Gateway API for advanced traffic management with Google Cloud Load Balancing.
  - Multi-cluster capabilities via GKE Fleet for cross-cluster service mesh and policy management.
- **Architecture Fit** — GKE Autopilot is the primary container orchestration platform on GCP. Its fully managed model reduces operational overhead. IaC modules provision clusters with Workload Identity, Config Sync (GitOps), and Policy Controller pre-configured.

### Cloud Run

- **Description** — Serverless container platform that scales to zero. Supports any language or framework packaged as a container image.
- **Key Features**
  - Scale-to-zero and rapid scale-up with concurrency-based autoscaling.
  - Direct VPC egress for private networking without Serverless VPC Access connectors.
  - Cloud Run Jobs for batch and scheduled workloads.
  - Revision-based traffic splitting for canary deployments.
  - Second-generation execution environment with full Linux syscall compatibility.

### Cloud Functions

- **Description** — Event-driven serverless functions supporting Node.js, Python, Go, Java, .NET, Ruby, and PHP.
- **Key Features**
  - 2nd gen functions built on Cloud Run for longer timeouts, larger instances, and concurrency.
  - Eventarc triggers for 90+ Google Cloud and third-party event sources.
  - CloudEvents standard for event format portability.

### Cloud SQL & Spanner

- **Cloud SQL** — Managed PostgreSQL, MySQL, and SQL Server with automated backups, high availability, and read replicas.
- **Spanner** — Globally distributed, strongly consistent relational database. Supports SQL with horizontal scaling and 99.999% availability SLA.

### Vertex AI

- **Description** — Unified ML platform for building, deploying, and managing ML models and generative AI applications.
- **Key Features**
  - Vertex AI Pipelines — Kubeflow and TFX-based ML workflow orchestration.
  - Vertex AI Endpoints — online and batch prediction with autoscaling and traffic splitting.
  - Vertex AI Feature Store — managed feature repository for training and serving.
  - Model Garden — access to Google's foundation models (Gemini, PaLM) and open-source models.
  - Vertex AI Agent Builder for building conversational AI and search applications.

### Additional GCP Services

- **GCS (Google Cloud Storage)** — Object storage with Standard, Nearline, Coldline, and Archive classes. Object Lifecycle Management for automated tiering.
- **Pub/Sub** — Global messaging and event streaming service with at-least-once and exactly-once delivery. Supports push and pull subscriptions.

---

## Workload Types

The Software Factory provides golden paths for each common workload type, with pre-configured CI/CD pipelines, IaC modules, security scanning, and observability instrumentation.

### REST / gRPC APIs

- **Description** — Synchronous request/response services exposing HTTP/JSON or gRPC/Protobuf interfaces.
- **Typical Targets** — EKS/AKS/GKE pods behind an ingress controller or API gateway; Lambda/Functions/Cloud Run for low-traffic or burst workloads.
- **Factory Provisions** — API gateway configuration, mTLS or OAuth2 authentication, rate limiting, OpenAPI/Protobuf schema validation, distributed tracing instrumentation.

### SPAs / SSR Applications

- **Description** — Single-page applications (React, Angular, Vue) served from CDN, or server-side rendered applications (Next.js, Nuxt, SvelteKit).
- **Typical Targets** — S3 + CloudFront, Azure Blob + Front Door, GCS + Cloud CDN for SPAs; Container Apps, Cloud Run, or EKS for SSR.
- **Factory Provisions** — CDN configuration, WAF rules, CSP headers, build pipeline with Lighthouse performance checks, real-user monitoring instrumentation.

### Batch / Scheduled Jobs

- **Description** — Periodic or on-demand processing tasks (report generation, data aggregation, cleanup).
- **Typical Targets** — Kubernetes CronJobs, AWS Batch, Azure Container Instances, Cloud Run Jobs.
- **Factory Provisions** — Scheduling configuration (CronJob or cloud scheduler), retry policies, dead-letter handling, execution monitoring with alerting on failure or SLA breach.

### ML Model Endpoints

- **Description** — Real-time and batch inference endpoints for machine learning models.
- **Typical Targets** — SageMaker Endpoints, Azure ML Online Endpoints, Vertex AI Endpoints; EKS/AKS/GKE with GPU node pools for custom serving frameworks (TorchServe, Triton).
- **Factory Provisions** — Model registry integration, A/B testing configuration, autoscaling policies, model monitoring (data drift, prediction quality), SBOM for model dependencies.

### ETL / ELT Pipelines

- **Description** — Data extraction, transformation, and loading workflows for analytics and data warehousing.
- **Typical Targets** — AWS Glue, Azure Data Factory, Dataflow (Apache Beam); Spark on EMR/HDInsight/Dataproc; dbt for transformation.
- **Factory Provisions** — Pipeline orchestration (Airflow/MWAA, Prefect, Dagster), data quality checks (Great Expectations), lineage tracking, access control for data sources and sinks.

### Event-Driven Streaming

- **Description** — Real-time event processing with continuous data streams.
- **Typical Targets** — Amazon Kinesis, Azure Event Hubs, Google Pub/Sub; Apache Kafka (Amazon MSK, Confluent Cloud); Apache Flink for stream processing.
- **Factory Provisions** — Schema registry (Confluent, AWS Glue), consumer group management, dead-letter queue configuration, backpressure handling, exactly-once processing guarantees where required.

---

## Further Reading

- [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [Amazon SageMaker Documentation](https://docs.aws.amazon.com/sagemaker/)
- [Azure AKS Documentation](https://learn.microsoft.com/en-us/azure/aks/)
- [Azure Functions Documentation](https://learn.microsoft.com/en-us/azure/azure-functions/)
- [Azure Container Apps Documentation](https://learn.microsoft.com/en-us/azure/container-apps/)
- [Azure ML Documentation](https://learn.microsoft.com/en-us/azure/machine-learning/)
- [GKE Documentation](https://cloud.google.com/kubernetes-engine/docs)
- [Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Vertex AI Documentation](https://cloud.google.com/vertex-ai/docs)
- [Karpenter Documentation](https://karpenter.sh/docs/)
- [KEDA Documentation](https://keda.sh/docs/)

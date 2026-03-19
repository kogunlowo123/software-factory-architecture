# Observability

The Observability layer provides end-to-end visibility into the health, performance, and behavior of all services running through the Software Factory. It covers the three pillars of observability — traces, metrics, and logs — along with SLO management, DORA metrics, and capacity planning. The architecture favors open standards (OpenTelemetry) to avoid vendor lock-in while leveraging cloud-native services where they provide operational advantage.

---

## OpenTelemetry

- **Official Site:** [https://opentelemetry.io](https://opentelemetry.io)
- **Description** — CNCF project providing a vendor-neutral, open standard for collecting and exporting telemetry data (traces, metrics, logs). It is the convergence of OpenTracing and OpenCensus, and has become the industry standard for instrumentation.

### Key Features

- **Unified API and SDK** — Single instrumentation layer that produces traces, metrics, and logs. Supports auto-instrumentation for Java, .NET, Python, Node.js, Go, Ruby, and PHP.
- **OpenTelemetry Collector** — Vendor-agnostic proxy that receives, processes, and exports telemetry data. Supports receivers (OTLP, Prometheus, Jaeger, Zipkin), processors (batching, filtering, tail sampling, attribute enrichment), and exporters (Prometheus, Jaeger, OTLP, cloud-native backends).
- **Context Propagation** — W3C Trace Context and Baggage standards for distributed context propagation across service boundaries, message queues, and cloud provider boundaries.
- **Semantic Conventions** — Standardized attribute names for HTTP, gRPC, database, messaging, and cloud resource metadata, enabling consistent querying across heterogeneous services.
- **OTLP Protocol** — Native wire protocol (gRPC and HTTP/protobuf) for high-performance telemetry transport.

### Architecture Fit

OpenTelemetry is the instrumentation standard for all services built through the Software Factory. Golden path templates include auto-instrumentation configuration and Collector sidecar/DaemonSet deployment manifests. The Collector acts as the telemetry routing layer, decoupling application instrumentation from backend choice.

---

## Metrics — Prometheus + Thanos

### Prometheus

- **Official Site:** [https://prometheus.io](https://prometheus.io)
- **Description** — CNCF Graduated project for metric collection and alerting. Pull-based model scrapes targets at configurable intervals, stores time-series data in a local TSDB, and evaluates alerting rules.

#### Key Features

- **PromQL** — Expressive query language for aggregation, filtering, and mathematical operations on time-series data.
- **Service Discovery** — Native integration with Kubernetes, Consul, EC2, Azure, and GCP for automatic target discovery.
- **Alertmanager** — Deduplication, grouping, silencing, and routing of alerts to PagerDuty, Slack, OpsGenie, email, and webhooks.
- **Recording Rules** — Pre-computed aggregations for expensive queries, reducing dashboard load times.
- **Ecosystem** — Thousands of exporters for infrastructure (node_exporter, kube-state-metrics), databases, message queues, and custom applications.

### Thanos

- **Official Site:** [https://thanos.io](https://thanos.io)
- **Description** — CNCF Incubating project that extends Prometheus with long-term storage, global query view, and multi-cluster federation. Maintains full compatibility with PromQL and the Prometheus API.

#### Key Features

- **Thanos Sidecar** — Uploads Prometheus TSDB blocks to object storage (S3, GCS, Azure Blob) for long-term retention.
- **Thanos Query** — Global query layer that federates across multiple Prometheus instances and object storage, providing a single PromQL endpoint for all clusters.
- **Thanos Compactor** — Downsamples and compacts historical data to reduce storage costs while maintaining query performance.
- **Thanos Ruler** — Evaluates recording and alerting rules against the global query view for cross-cluster alerting.
- **Multi-tenancy** — Label-based tenancy for isolating metric data between teams or environments.

### Architecture Fit

Each Kubernetes cluster runs a Prometheus instance with Thanos Sidecar. Thanos Query provides a global view across all clusters and clouds. DORA metrics, SLO tracking, and capacity planning dashboards query Thanos for cross-environment visibility. Object storage (S3, GCS, Azure Blob) provides cost-effective long-term retention.

---

## Dashboards & Visualization — Grafana

- **Official Site:** [https://grafana.com](https://grafana.com)
- **Description** — Open-source observability and data visualization platform. Supports 100+ data sources including Prometheus, Thanos, Jaeger, OpenSearch, CloudWatch, Azure Monitor, and Google Cloud Monitoring.

### Key Features

- **DORA Metrics Dashboards** — Pre-built dashboards tracking deployment frequency, lead time for changes, change failure rate, and mean time to recovery. Data sourced from CI/CD pipeline events and incident management systems.
- **SLO Tracking** — Grafana SLO plugin for defining, tracking, and alerting on service-level objectives. Burn-rate alerting notifies teams before error budgets are exhausted.
- **Dashboard-as-Code** — Dashboards defined in JSON or Jsonnet, stored in Git, and provisioned via Grafana's provisioning API or Terraform provider.
- **Alerting** — Unified alerting with support for multi-dimensional rules, notification policies, and contact points (PagerDuty, Slack, OpsGenie, Microsoft Teams).
- **Explore** — Ad-hoc querying interface for Prometheus (PromQL), Loki (LogQL), Jaeger (trace search), and Tempo (TraceQL).

### Architecture Fit

Grafana is the primary visualization layer for all observability data. Dashboard definitions are version-controlled and deployed through CI/CD pipelines. Backstage embeds Grafana dashboards via iframe or the Grafana Backstage plugin, giving developers service-specific observability without leaving the developer portal.

---

## Distributed Tracing — Jaeger

- **Official Site:** [https://www.jaegertracing.io](https://www.jaegertracing.io)
- **Description** — CNCF Graduated distributed tracing platform originally developed at Uber. Collects, stores, and visualizes traces that follow requests across service boundaries.

### Key Features

- **OpenTelemetry Native** — Receives traces via OTLP, Jaeger Thrift, and Zipkin protocols. The recommended deployment uses the OpenTelemetry Collector as the ingestion layer.
- **Adaptive Sampling** — Adjusts sampling rates based on traffic volume and error rates, balancing trace completeness with storage costs.
- **Service Performance Monitoring (SPM)** — Derives RED metrics (rate, errors, duration) from trace data, bridging the gap between tracing and metrics.
- **Trace Comparison** — Side-by-side comparison of traces for debugging performance regressions.
- **Storage Backends** — Supports Elasticsearch, OpenSearch, Cassandra, Kafka (for buffering), and Badger (for local development).

### Architecture Fit

Jaeger receives traces from the OpenTelemetry Collector and provides the trace visualization UI. Trace IDs propagate through HTTP headers (W3C Trace Context) and message queue metadata, enabling end-to-end visibility across synchronous and asynchronous communication patterns. Grafana links metrics alerts to Jaeger traces via exemplars.

---

## Cloud-Native Observability Services

### AWS X-Ray & CloudWatch

- **AWS X-Ray** — Distributed tracing service integrated with Lambda, API Gateway, ECS, EKS, and SNS/SQS. Auto-instrumentation via X-Ray SDK or OpenTelemetry. Trace maps visualize service dependencies.
- **Amazon CloudWatch** — Metrics, logs, alarms, and dashboards for all AWS services. CloudWatch Logs Insights provides SQL-like querying. CloudWatch Container Insights for EKS/ECS observability. Contributor Insights for identifying top-N contributors to operational issues.

### Azure Monitor

- **Description** — Unified monitoring platform covering Application Insights (APM), Log Analytics (log aggregation), Azure Monitor Metrics, and Azure Workbooks (visualization).
- **Key Features**
  - Application Insights — auto-instrumentation for .NET, Java, Node.js, and Python. Live Metrics Stream for real-time telemetry.
  - Log Analytics — KQL (Kusto Query Language) for log and metric queries across Azure resources and hybrid environments.
  - Azure Monitor Agent — unified agent for collecting logs and metrics from VMs, VMSS, and Arc-enabled servers.
  - Integration with OpenTelemetry via Azure Monitor OpenTelemetry Exporter.

### Architecture Fit

Cloud-native observability services complement the open-source stack. OpenTelemetry Collector exports to both open-source backends (Prometheus, Jaeger) and cloud-native services (X-Ray, CloudWatch, Azure Monitor) simultaneously. This dual-export strategy provides cloud-native operational convenience while maintaining portability.

---

## Log Aggregation

### OpenSearch

- **Official Site:** [https://opensearch.org](https://opensearch.org)
- **Description** — Open-source search and analytics suite derived from Elasticsearch 7.10. Provides log ingestion, indexing, querying, and visualization via OpenSearch Dashboards.
- **Key Features**
  - Index lifecycle management for automated rollover, retention, and deletion.
  - Anomaly detection using machine learning for identifying unusual log patterns.
  - Security analytics with pre-built correlation rules for threat detection.
  - Observability plugin with trace analytics, metrics correlation, and log-to-trace linking.
  - Cross-cluster replication for disaster recovery and geographic distribution.

### Splunk

- **Description** — Enterprise log management and SIEM platform with powerful search (SPL), machine learning toolkit, and extensive app ecosystem.
- **Key Features**
  - SPL (Search Processing Language) for complex log analysis and correlation.
  - Splunk Observability Cloud — integrated APM, infrastructure monitoring, and real-user monitoring.
  - Compliance and audit reporting with pre-built apps for PCI, HIPAA, and SOX.
  - SOAR (Security Orchestration, Automation, and Response) for automated incident response.

### Architecture Fit

Log aggregation platforms receive logs from the OpenTelemetry Collector, Kubernetes (Fluent Bit/Fluentd), and cloud-native log services. OpenSearch is the default for cost-effective, open-source log aggregation. Splunk is used where enterprise SIEM capabilities or existing organizational investments require it. Both platforms correlate logs with traces via trace ID fields.

---

## SLO / Error Budget Management

Service-level objectives (SLOs) define the reliability targets for each service. Error budgets — the allowed unreliability — govern the pace of feature delivery versus reliability investment.

### Key Practices

- **SLI Definition** — Service-level indicators (SLIs) are derived from metrics (availability, latency, throughput, correctness). Golden path templates include default SLI configurations.
- **Error Budget Policies** — When the error budget is exhausted, automated policies reduce deployment frequency or require additional review for changes. When the budget is healthy, teams move faster.
- **Burn-Rate Alerting** — Multi-window, multi-burn-rate alerts (per Google SRE book methodology) notify teams of accelerating error budget consumption before the budget is fully spent.
- **SLO Documentation** — Each service's SLO is defined in a YAML file alongside the service code, tracked in the Backstage catalog, and visualized in Grafana.

---

## Capacity Planning

- **Resource Utilization Tracking** — Prometheus metrics (CPU, memory, network, storage) aggregated by namespace, team, and environment.
- **Kubernetes VPA/HPA Recommendations** — Vertical Pod Autoscaler recommendations surface in Grafana to identify over-provisioned and under-provisioned workloads.
- **Cloud Cost Correlation** — Cost data from AWS Cost Explorer, Azure Cost Management, and GCP Billing is joined with utilization metrics to identify waste.
- **Forecasting** — Prometheus recording rules and Grafana's time-series forecasting for predicting resource exhaustion and planning scaling events.

---

## Further Reading

- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- [OpenTelemetry Collector Configuration](https://opentelemetry.io/docs/collector/configuration/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Thanos Documentation](https://thanos.io/tip/thanos/getting-started.md/)
- [Grafana Documentation](https://grafana.com/docs/grafana/latest/)
- [Grafana SLO](https://grafana.com/docs/grafana-cloud/alerting-and-irm/slo/)
- [Jaeger Documentation](https://www.jaegertracing.io/docs/)
- [OpenSearch Documentation](https://opensearch.org/docs/latest/)
- [AWS CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch/)
- [AWS X-Ray Documentation](https://docs.aws.amazon.com/xray/)
- [Azure Monitor Documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/)
- [Google SRE Book — Service Level Objectives](https://sre.google/sre-book/service-level-objectives/)
- [Google SRE Workbook — Implementing SLOs](https://sre.google/workbook/implementing-slos/)

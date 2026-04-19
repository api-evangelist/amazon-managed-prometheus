# Amazon Managed Service for Prometheus (amazon-managed-prometheus)
Amazon Managed Service for Prometheus is a serverless, Prometheus-compatible monitoring service for container metrics. It automatically scales as your monitoring needs increase, works with open-source tools, and integrates with Amazon EKS and other container environments. The service provides fully managed workspaces, alert manager definitions, and rule group namespaces for Prometheus-compatible monitoring at scale.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/amazon-managed-prometheus/refs/heads/main/apis.yml)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - AWS, Containers, Monitoring, Observability, Prometheus

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### Amazon Managed Service for Prometheus API
The Amazon Managed Service for Prometheus API provides programmatic access to create and manage workspaces, alert manager definitions, rule groups namespaces, logging configurations, and scrapers for Prometheus-compatible monitoring. Covers the full workspace lifecycle and monitoring configuration management.

**Human URL:** [https://aws.amazon.com/prometheus/](https://aws.amazon.com/prometheus/)

#### Tags:

 - Containers, Monitoring, Observability, Prometheus

#### Properties

- [Documentation](https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-APIReference.html)
- [OpenAPI](openapi/amazon-managed-prometheus-openapi-original.yaml)
- [GettingStarted](https://aws.amazon.com/prometheus/getting-started/)
- [Pricing](https://aws.amazon.com/prometheus/pricing/)
- [FAQ](https://aws.amazon.com/prometheus/faqs/)

## Common Properties

- [Portal](https://aws.amazon.com/prometheus/)
- [Documentation](https://docs.aws.amazon.com/prometheus/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/mt/tag/amazon-managed-service-for-prometheus/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/prometheus/)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [Login](https://signin.aws.amazon.com/)
- [StatusPage](https://health.aws.amazon.com/health/status)
- [Contact](https://aws.amazon.com/contact-us/)
- [SpectralRules](rules/amazon-managed-prometheus-spectral-rules.yml)
- [Vocabulary](vocabulary/amazon-managed-prometheus-vocabulary.yaml)
- [NaftikoCapability](capabilities/metrics-monitoring-workflow.yaml)

## Features

| Name | Description |
|------|-------------|
| Serverless Prometheus | Run Prometheus-compatible monitoring without managing servers, scaling, or high availability. |
| Alert Manager Definitions | Configure Prometheus AlertManager rules for routing, grouping, and suppressing alerts. |
| Rule Groups Namespaces | Define and manage Prometheus recording and alerting rules organized in namespaces. |
| Managed Scrapers | Create managed scrapers to automatically collect metrics from Amazon EKS clusters. |
| Logging Configuration | Configure logging for Prometheus workspaces to capture operational events. |
| Prometheus-Compatible APIs | Use standard Prometheus remote write and query APIs with existing tooling and clients. |

## Use Cases

| Name | Description |
|------|-------------|
| Kubernetes Cluster Monitoring | Monitor EKS clusters and Kubernetes workloads with Prometheus metrics at any scale. |
| Container Performance Metrics | Collect and analyze container CPU, memory, and network metrics for performance optimization. |
| Microservices Observability | Monitor distributed microservices with Prometheus metrics and custom alert rules. |
| Infrastructure Capacity Planning | Track resource utilization trends over time for infrastructure capacity planning. |
| SLA Monitoring | Define SLO-based alerting rules to monitor service level agreements in real time. |

## Integrations

| Name | Description |
|------|-------------|
| Amazon EKS | Collect metrics from EKS clusters using managed scrapers and Prometheus remote write. |
| Amazon Managed Grafana | Visualize Prometheus metrics in Grafana dashboards using AMP as a data source. |
| AWS Distro for OpenTelemetry | Use ADOT collectors to send metrics to AMP workspaces via remote write. |
| Amazon CloudWatch | Forward Prometheus alerts and metrics to CloudWatch for cross-service monitoring. |
| Prometheus Alertmanager | Use native Prometheus Alertmanager configuration for alert routing and notification. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [Amazon Managed Service for Prometheus OpenAPI](openapi/amazon-managed-prometheus-openapi-original.yaml)

### JSON Schema

79 schema files available in the [json-schema/](json-schema/) directory.

### JSON Structure

79 structure files available in the [json-structure/](json-structure/) directory.

### JSON-LD

- [Amazon Managed Service for Prometheus Context](json-ld/amazon-managed-prometheus-context.jsonld)

### Examples

79 example files available in the [examples/](examples/) directory.

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [Amazon Managed Service for Prometheus](capabilities/shared/managed-prometheus.yaml) — 24 operations for workspace management, alert rules, and scraper configuration

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Metrics Monitoring Workflow](capabilities/metrics-monitoring-workflow.yaml) | Amazon Managed Service for Prometheus | 6 | Platform Engineer, Site Reliability Engineer |

## Vocabulary

- [Amazon Managed Service for Prometheus Vocabulary](vocabulary/amazon-managed-prometheus-vocabulary.yaml) — Unified taxonomy mapping 5 resources, 6 actions, 1 workflow, and 2 personas across operational (OpenAPI) and capability (Naftiko) dimensions

## Rules

- [Amazon Managed Service for Prometheus Spectral Rules](rules/amazon-managed-prometheus-spectral-rules.yml) — 18 rules across 7 categories enforcing Amazon Managed Service for Prometheus API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com

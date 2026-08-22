# Amazon Managed Service for Prometheus (amazon-managed-prometheus)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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

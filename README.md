🚀 Zero-Downtime Deployment System
> Terraform-automated EKS blue/green deployments cutting rollback time by 70%, with OpenTelemetry tracing reducing root-cause analysis time by 60%.
![AWS](https://img.shields.io/badge/AWS_EKS-FF9900?style=flat&logo=amazonaws&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat&logo=terraform&logoColor=white)
![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-000000?style=flat&logo=opentelemetry&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat&logo=grafana&logoColor=white)
---
Overview
With 12M+ records processed daily, deployment failures and production incidents on the Aurora platform had a direct customer impact. I built a zero-downtime deployment system using Terraform-automated blue/green deployments on AWS EKS, and instrumented the entire service mesh with OpenTelemetry distributed tracing — dramatically cutting both rollback time and time-to-resolution for production issues.
---
The Problem
Deployments required manual steps and caused service disruption during updates
When incidents occurred, engineers spent hours tracing failures across distributed services
No consistent observability strategy — each service logged differently, making correlation impossible
Rollbacks were slow, manual, and risky
---
Blue/Green Deployment Architecture
```
                    ┌─────────────────┐
                    │   AWS Route 53  │
                    │  (Traffic Split)│
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │ 100% traffic │              │ 0% traffic
              ▼              │              ▼
    ┌──────────────────┐     │    ┌──────────────────┐
    │   Blue (Live)    │     │    │  Green (Standby) │
    │   EKS Cluster    │     │    │   EKS Cluster    │
    │  Current Version │     │    │   New Version    │
    └──────────────────┘     │    └──────────────────┘
                             │
                    Deploy → Test → Shift Traffic → Decommission Blue
```
On bad deploy: flip traffic back to Blue in under 2 minutes (down from ~10 minutes manually).
---
Terraform Module
The deployment system was fully codified as reusable Terraform modules:
```hcl
module "aurora_blue_green" {
  source = "./modules/blue-green-eks"

  cluster_name     = "aurora-prod"
  blue_version     = var.current_version
  green_version    = var.new_version
  traffic_weight   = var.green_traffic_percent  # 0 → 10 → 50 → 100

  health_check = {
    path                = "/health"
    healthy_threshold   = 2
    unhealthy_threshold = 3
    interval            = 10
  }

  rollback_enabled     = true
  rollback_threshold   = "error_rate > 1%"
}
```
---
Observability Stack
Integrated OpenTelemetry as the unified instrumentation layer across all microservices:
```
Service A ──▶ OTel Collector ──▶ Jaeger (Traces)
Service B ──▶      │          ──▶ Prometheus (Metrics)
Service C ──▶      │          ──▶ Grafana (Dashboards)
                   │
                   └──▶ Alerts (PagerDuty / Slack)
```
Every request carries a trace ID from frontend to backend — making cross-service debugging a matter of searching one ID rather than correlating logs manually.
---
Key Results
Metric	Before	After
Rollback time	~10 minutes (manual)	~2 minutes (automated) — 70% reduction
Root-cause analysis time	Hours of log correlation	Minutes via trace ID — 60% reduction
Deployment downtime	Minutes of disruption	Zero downtime
SonarQube coverage	Variable	>90% enforced
---
Tech Stack
Layer	Technology
Infrastructure	Terraform, AWS EKS
Deployment	Blue/Green, GitLab CI/CD
Tracing	OpenTelemetry, Jaeger
Metrics	Prometheus
Dashboards	Grafana
Code Quality	SonarQube
---
Lessons Learned
Blue/green only works if your health checks are reliable — investing in meaningful health endpoints saved us from several bad deploys
Distributed tracing is most valuable when adopted uniformly — partial adoption creates gaps that defeat the purpose
Automating rollbacks requires defining what "bad" looks like upfront — error rate thresholds, latency spikes, health check failures
---
Contact
Fazal Shaik · LinkedIn · fazal.shaik2025@gmail.com

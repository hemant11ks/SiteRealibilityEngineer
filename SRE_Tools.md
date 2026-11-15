# Site Reliability Engineering (SRE) Tools — Complete README

> A practical, interview-focused guide to the tools, workflows, metrics, commands, and engineering practices used in Site Reliability Engineering.
>
> **Audience:** DevOps Engineers, SREs, Platform Engineers, Cloud Engineers  
> **Focus:** Linux • Kubernetes • AWS/Azure/GCP/OCI • Observability • Automation • Reliability

---

## Table of Contents

1. [What Is SRE?](#1-what-is-sre)
2. [SRE Fundamentals and Metrics](#2-sre-fundamentals-and-metrics)
3. [SRE Tool Map by Category](#3-sre-tool-map-by-category)
4. [Monitoring and Metrics](#4-monitoring-and-metrics)
5. [Logging and Log Analytics](#5-logging-and-log-analytics)
6. [Distributed Tracing and OpenTelemetry](#6-distributed-tracing-and-opentelemetry)
7. [Alerting, On-Call, and Incident Management](#7-alerting-on-call-and-incident-management)
8. [Infrastructure, Containers, and Cloud](#8-infrastructure-containers-and-cloud)
9. [CI/CD, GitOps, and Safe Deployments](#9-cicd-gitops-and-safe-deployments)
10. [Performance, Load Testing, and Chaos Engineering](#10-performance-load-testing-and-chaos-engineering)
11. [Security, Policy, and Reliability](#11-security-policy-and-reliability)
12. [Useful Linux and Kubernetes Troubleshooting Commands](#12-useful-linux-and-kubernetes-troubleshooting-commands)
13. [PromQL and LogQL Cheat Sheet](#13-promql-and-logql-cheat-sheet)
14. [SLO and Burn-Rate Alert Example](#14-slo-and-burn-rate-alert-example)
15. [Reference SRE Architecture](#15-reference-sre-architecture)
16. [Practical End-to-End SRE Project](#16-practical-end-to-end-sre-project)
17. [Common Production Incident Walkthrough](#17-common-production-incident-walkthrough)
18. [Interview Questions and Answers](#18-interview-questions-and-answers)
19. [SRE Learning Roadmap](#19-sre-learning-roadmap)
20. [Official Documentation](#20-official-documentation)

---

## 1. What Is SRE?

**Site Reliability Engineering (SRE)** applies software engineering practices to operations to make services **reliable, observable, scalable, and recoverable**, while supporting safe delivery of new features.

| SRE responsibility | Practical outcome |
|---|---|
| Reliability engineering | Meet clearly defined reliability objectives |
| Observability | Explain what is happening using metrics, logs, and traces |
| Incident response | Detect, mitigate, recover, and learn from outages |
| Automation | Reduce manual, repetitive operational work (toil) |
| Capacity planning | Anticipate and safely handle growing demand |
| Release engineering | Deploy and roll back changes with controlled risk |
| Resilience engineering | Test and improve behavior under failures |
| Performance engineering | Improve request latency, throughput, and efficiency |

**DevOps vs. SRE:** DevOps is a set of collaboration and delivery principles; SRE is a way of implementing reliability engineering using software, SLOs, operational practices, and automation. The roles overlap heavily.

## 2. SRE Fundamentals and Metrics

### 2.1 SLIs, SLOs, SLAs, and error budgets

| Term | Meaning | Example |
|---|---|---|
| **SLI** — Service Level Indicator | A measured property of service behavior | Successful request ratio |
| **SLO** — Service Level Objective | Internal reliability target for an SLI | 99.9% successful requests over 30 days |
| **SLA** — Service Level Agreement | Customer-facing agreement, possibly with remedies | Contractual uptime commitment |
| **Error budget** | Amount of unreliability allowed by an SLO | 0.1% of requests may fail for a 99.9% success SLO |

**Request-based availability SLI:**

```text
Availability = Good requests / Eligible requests × 100
Error budget fraction = 1 − SLO target fraction
```

**Time-based illustration:** for a 30-day period (43,200 minutes), 99.9% uptime allows **43.2 minutes (43 min 12 sec)** of downtime. Actual *request-based* error budgets are counted in eligible requests, not minutes.

### 2.2 The Four Golden Signals

| Signal | What to measure | Example |
|---|---|---|
| **Latency** | How long requests take | p95/p99 HTTP duration |
| **Traffic** | Demand placed on a service | Requests per second (RPS) |
| **Errors** | Failed requests or invalid results | HTTP 5xx rate |
| **Saturation** | How close resources are to capacity | Queue depth, CPU throttling, memory pressure |

Other useful methods:

- **RED** for request-driven services: **Rate, Errors, Duration**.
- **USE** for resources: **Utilization, Saturation, Errors**.
- **MTTD**: Mean Time To Detect; **MTTR**: Mean Time To Restore/Recover (define the chosen convention within your team).
- **Toil**: repetitive, manual, automatable operational work that grows with service scale.
- **RCA/postmortem**: blameless analysis of causes, contributing conditions, response, and preventive actions.

## 3. SRE Tool Map by Category

> Tools are alternatives or complements; **you do not need all of them**. Choose according to scale, existing platforms, security, and team maturity.

| Category | Important tools | Primary use case |
|---|---|---|
| Metrics monitoring | **Prometheus**, VictoriaMetrics, InfluxDB | Collect and query time-series metrics |
| Dashboards | **Grafana**, Kibana | Visualize service health and trends |
| Long-term metrics | Thanos, Grafana Mimir, Cortex | Scalable metrics retention and querying |
| Exporters/collectors | Node Exporter, Blackbox Exporter, kube-state-metrics, cAdvisor | Expose infrastructure and application signals |
| Centralized logging | **Grafana Loki**, Elasticsearch, OpenSearch | Search and aggregate logs |
| Log pipelines | Fluent Bit, Vector, Grafana Alloy, Logstash | Collect, transform, and forward logs |
| Distributed tracing | **Jaeger**, Grafana Tempo, Zipkin | Find latency across microservice calls |
| Telemetry standard | **OpenTelemetry** (OTel) | Instrument, collect, and export metrics/logs/traces |
| APM / managed observability | Datadog, New Relic, Dynatrace, Splunk Observability | Hosted end-to-end application observability |
| Alert evaluation/routing | Prometheus rules, **Alertmanager**, Grafana Alerting | Trigger, group, route, silence alerts |
| On-call / incidents | **PagerDuty**, incident.io, FireHydrant, Jira Service Management | Paging, coordination, escalation, postmortems |
| SLO management | Sloth, Pyrra, Nobl9 | Create or manage SLO dashboards and alerts |
| OS / networking | Linux, systemd, `ss`, `tcpdump`, `dig`, `curl` | Diagnose host and network issues |
| Containers / orchestration | **Docker**, **Kubernetes**, Helm | Package and operate workloads |
| Kubernetes scaling | HPA, VPA, KEDA, Karpenter | Scale workloads or compute capacity |
| IaC and configuration | **Terraform**, OpenTofu, Ansible, Pulumi | Provision and configure infrastructure |
| CI/CD | **Jenkins**, GitHub Actions, GitLab CI, Azure Pipelines | Build, test, and release software |
| GitOps | **Argo CD**, Flux | Reconcile deployed state from Git |
| Progressive delivery | Argo Rollouts, Flagger | Canary/blue-green and metric-driven rollouts |
| Service connectivity | NGINX, Envoy, Istio, Linkerd | Ingress, routing, policies, service-to-service networking |
| Load/performance testing | **k6**, JMeter, Locust, Gatling | Validate latency and throughput under load |
| Synthetic monitoring | Blackbox Exporter, Checkly, Pingdom | Test external availability and workflows |
| Chaos / resilience | LitmusChaos, Chaos Mesh, Gremlin | Safely inject controlled failures |
| Security / posture | Trivy, Falco, Kyverno, OPA Gatekeeper | Detect risks and enforce guardrails |
| Cloud-native monitoring | AWS CloudWatch/X-Ray, Azure Monitor/Application Insights, Google Cloud Monitoring/Trace, OCI Monitoring/Logging | Monitor cloud workloads using native services |

## 4. Monitoring and Metrics

### 4.1 Prometheus

**Use case:** scrape metric endpoints, store time series, evaluate PromQL queries, and fire alert rules.

- Common targets: application `/metrics`, Node Exporter, kube-state-metrics, cAdvisor, Blackbox Exporter.
- Deployment options: standalone Prometheus; **Prometheus Operator / kube-prometheus-stack** for Kubernetes.
- Best for: service and infrastructure metrics with a strong cloud-native ecosystem.
- Watch out for: high-cardinality labels, retention sizing, and HA/long-term-storage design.

Example scrape configuration (`prometheus.yml`, illustration):

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "sample-api"
    static_configs:
      - targets: ["api:8000"]  # reachable host:port exposing /metrics
```

> This presumes the application exposes valid Prometheus-format metrics at `/metrics`. `api` must resolve from the Prometheus runtime (e.g., within the same Docker network).

### 4.2 Grafana

**Use case:** dashboards, exploration, and alerting across Prometheus, Loki, Tempo, cloud sources, and other data sources.

Recommended panels for an API: requests/sec, success %, p95 latency, 5xx errors, pod restarts, CPU throttling, memory utilization, HPA replicas, and deployment annotations.

### 4.3 Exporters

| Exporter | What it provides | When to use |
|---|---|---|
| Node Exporter | Host CPU, memory, disk, networking | Linux VM/host monitoring |
| Blackbox Exporter | HTTP, TCP, DNS, ICMP probing | External endpoint availability |
| kube-state-metrics | Kubernetes object state | Deployment replicas, pod phases, node conditions |
| cAdvisor | Container resource metrics | Per-container CPU/memory usage |
| Custom app exporter/instrumentation | Business/application metrics | Request latency, error count, queue depth |

## 5. Logging and Log Analytics

| Tool | Role | Example use case |
|---|---|---|
| **Grafana Loki** | Label-indexed log aggregation and LogQL search | Search errors from Kubernetes workloads |
| **Elasticsearch / OpenSearch** | Search and analytics engines | Full-text search over application logs |
| **Kibana / OpenSearch Dashboards** | Log exploration UI | Visualize errors and log patterns |
| **Fluent Bit / Vector / Grafana Alloy** | Log/telemetry collection and forwarding | Read container stdout and ship logs |
| **Logstash** | Log transformation pipeline | Parse and transform heterogeneous logs |

### Logging best practices

- Prefer structured JSON logs: `timestamp`, `level`, `service`, `environment`, `request_id`, `trace_id`, `message`.
- Correlate logs with traces via `trace_id` and/or `span_id`.
- Do **not** log passwords, secrets, API tokens, or unnecessary personal information.
- Use sensible retention, access controls, sampling/filtering, and low-cardinality Loki labels.
- Prefer standard, supported log shippers; choose collection mechanisms based on current vendor guidance.

Example structured log:

```json
{"timestamp":"2026-11-15T12:00:00Z","level":"ERROR","service":"payments-api","trace_id":"abc123","message":"database query timed out"}
```

## 6. Distributed Tracing and OpenTelemetry

### Tracing concepts

- **Trace:** one end-to-end operation, such as `POST /checkout`.
- **Span:** a step inside the trace, such as an HTTP call or database query.
- **Trace context:** propagates identifiers between services so spans can be correlated.
- **Distributed tracing:** shows which service or dependency contributed to slow requests.

| Tool | Purpose |
|---|---|
| **OpenTelemetry SDK / automatic instrumentation** | Generate vendor-neutral telemetry |
| **OpenTelemetry Collector** | Receive, process, and export telemetry |
| **Jaeger** | Store/query/visualize distributed traces |
| **Grafana Tempo** | Distributed trace backend integrated with Grafana |
| **Zipkin** | Distributed tracing system |

**Important:** OpenTelemetry is an instrumentation and collection framework, **not** by itself a database or visualization backend.

```text
API / Worker / Database instrumentation
        │
        ▼
  OpenTelemetry SDKs
        │ OTLP
        ▼
 OpenTelemetry Collector
        ├── metrics ──► Prometheus-compatible backend ──► Grafana
        ├── logs ─────► Loki / other log backend ───────► Grafana
        └── traces ───► Tempo / Jaeger ────────────────► Trace UI
```

## 7. Alerting, On-Call, and Incident Management

### Tool responsibilities

| Tool | Responsibility |
|---|---|
| Prometheus alert rules | Evaluate conditions over metrics |
| Alertmanager | Group, deduplicate, route, silence, inhibit notifications |
| Grafana Alerting | Unified evaluation/notifications across data sources |
| PagerDuty | On-call schedules, escalation, acknowledgment, paging |
| incident.io / FireHydrant | Incident coordination, timelines, workflows, follow-up |
| Jira Service Management | Incidents, service requests, workflows, post-incident actions |

**Typical flow:**

```text
Metrics / health checks
      ↓
Prometheus alert rule
      ↓
Alertmanager (group + route + deduplicate)
      ↓
PagerDuty / chat / email
      ↓
On-call engineer → runbook → mitigate → recover
      ↓
Blameless postmortem + preventive changes
```

### Good alert design

- Page on **actionable symptoms** with user impact: SLO burn, sustained high failure rate, critical dependency failure.
- Prefer symptoms to arbitrary resource thresholds; use capacity alerts when they predict failure and require action.
- Include service, severity, user impact, dashboard, runbook, and relevant owner.
- Use grouping, inhibition, routing, and planned silences to reduce alert fatigue.
- Separate **page now** issues from **ticket for working hours** issues.

## 8. Infrastructure, Containers, and Cloud

### Kubernetes SRE components

| Component | Reliability use case |
|---|---|
| Deployments / StatefulSets | Manage service replicas and rollout state |
| Readiness probes | Control whether a pod receives traffic |
| Liveness probes | Restart unhealthy containers when appropriate |
| Startup probes | Allow slow-starting apps to initialize |
| Resource requests/limits | Reserve capacity and bound resource use |
| HPA | Autoscale pod replicas using metrics |
| VPA | Recommend or adjust CPU/memory requests |
| KEDA | Event-driven application autoscaling |
| PodDisruptionBudget | Limit simultaneous voluntary pod disruptions |
| Topology spread / anti-affinity | Reduce correlated pod placement failure |
| NetworkPolicy | Restrict workload network communication |
| Karpenter / Cluster Autoscaler | Adjust available cluster node capacity |

**Cloud-native observability equivalence:**

| Requirement | AWS | Azure | Google Cloud | OCI |
|---|---|---|---|---|
| Metrics and alerts | CloudWatch | Azure Monitor | Cloud Monitoring | OCI Monitoring |
| Log management | CloudWatch Logs | Azure Monitor Logs / Log Analytics | Cloud Logging | OCI Logging |
| Distributed tracing / APM | X-Ray | Application Insights | Cloud Trace | OCI Application Performance Monitoring |
| Infrastructure deployment | CloudFormation / Terraform | ARM/Bicep / Terraform | Infrastructure Manager / Terraform | Resource Manager / Terraform |

> Product coverage, integrations, and commercial features vary. Validate regional/service availability before production adoption.

## 9. CI/CD, GitOps, and Safe Deployments

| Tool | Use case | SRE value |
|---|---|---|
| Jenkins | Highly customizable build/test/deploy pipelines | Repeatable release and automated quality gates |
| GitHub Actions / GitLab CI / Azure Pipelines | Repository-integrated CI/CD | Automated validation and deployments |
| Argo CD / Flux | GitOps reconciliation | Detect drift and restore desired state |
| Helm | Kubernetes application packaging | Versioned and repeatable configurations |
| Argo Rollouts / Flagger | Progressive delivery | Limit blast radius; pause/rollback bad releases |
| Terraform / OpenTofu | Infrastructure as code | Auditable and repeatable infrastructure changes |
| Ansible | Configuration and operational automation | Standardized host setup and remediation |

**Recommended release strategy:**

```text
Git commit
   ↓
CI: lint → unit/integration tests → image build → security scan
   ↓
Artifact registry (immutable image tag/digest)
   ↓
GitOps deployment manifest update
   ↓
Canary rollout (small percentage of production traffic)
   ↓
SLO/latency/error-rate analysis
   ├── Healthy → promote gradually
   └── Unhealthy → halt/rollback + incident investigation
```

Record release ID, git commit SHA, image digest, deployment time, change owner, and rollback path.

## 10. Performance, Load Testing, and Chaos Engineering

| Tool | Use case |
|---|---|
| **k6** | Scripted HTTP/API load and performance testing |
| **JMeter** | Protocol-oriented load and performance tests |
| **Locust** | Python-based load testing |
| **Gatling** | Code-driven load testing |
| **LitmusChaos / Chaos Mesh** | Kubernetes fault-injection experiments |
| **Gremlin** | Managed resilience testing |
| **Blackbox Exporter / Checkly** | Synthetic service or endpoint checks |

Test conditions: steady-state load, bursts, service restarts, network latency, dependency outages, and node disruption. Start in a controlled non-production environment; define blast-radius limits, experiment owner, metrics, and abort criteria.

Example `k6` script (`load.js`):

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 10,
  duration: '1m',
  thresholds: {
    http_req_failed: ['rate<0.01'],
    http_req_duration: ['p(95)<500'],
  },
};

export default function () {
  const res = http.get('http://localhost:8000/health');
  check(res, { 'HTTP 200': (r) => r.status === 200 });
  sleep(1);
}
```

Run with `k6 run load.js` only against systems you own or are authorized to test. Adjust URL and thresholds to match your service.

## 11. Security, Policy, and Reliability

| Tool | SRE-relevant use case |
|---|---|
| Trivy | Scan container images and IaC for known risks |
| Falco | Detect suspicious runtime behaviors |
| Kyverno | Kubernetes admission and policy controls |
| OPA / Gatekeeper | Enforce policy-as-code for Kubernetes |
| External Secrets / Vault | Secure secret delivery and rotation |
| cert-manager | Automate Kubernetes TLS certificate lifecycle |

Reliability also depends on secure access, credential hygiene, tested backups, restore drills, supply-chain controls, and least privilege.

## 12. Useful Linux and Kubernetes Troubleshooting Commands

### Linux / networking

```bash
# CPU, memory, disk
uptime
free -h
df -h
du -sh /var/log/* 2>/dev/null | sort -h | tail
vmstat 1 5

# Processes and services
ps aux --sort=-%cpu | head
systemctl status nginx
journalctl -u nginx --since '30 minutes ago' --no-pager

# Networking and DNS
ss -tulpn
ip addr
ip route
dig example.com
curl -sS -o /dev/null -w '%{http_code} %{time_total}\n' https://example.com

# Connectivity / packets (requires permissions, use carefully)
traceroute example.com
sudo tcpdump -i any -nn port 443 -c 20
```

### Kubernetes incident triage

```bash
# Resource / workload state
kubectl get nodes
kubectl get pods -A -o wide
kubectl top nodes
kubectl top pods -n default
kubectl get events -n default --sort-by=.lastTimestamp

# Pod inspection
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --tail=200
kubectl logs <pod-name> -n <namespace> --previous

# Service / rollout inspection
kubectl get deploy,svc,endpoints -n <namespace>
kubectl rollout status deployment/<deployment-name> -n <namespace>
kubectl rollout history deployment/<deployment-name> -n <namespace>

# Roll back only with appropriate change/incident authorization
kubectl rollout undo deployment/<deployment-name> -n <namespace>
```

> `kubectl top` requires a working Metrics Server (or compatible metrics API). Commands with placeholders must be customized. A rollback may not reverse database migrations or external side effects.

## 13. PromQL and LogQL Cheat Sheet

> **Assumption:** the application exposes `http_requests_total{job="sample-api",status="..."}` and a classic histogram named `http_request_duration_seconds_bucket`. Metric names and labels vary by application/client library; adapt queries accordingly.

### PromQL (Prometheus)

**Request rate (RPS):**

```promql
sum(rate(http_requests_total{job="sample-api"}[5m]))
```

**Fraction of requests returning HTTP 5xx:**

```promql
sum(rate(http_requests_total{job="sample-api",status=~"5.."}[5m]))
/
sum(rate(http_requests_total{job="sample-api"}[5m]))
```

**p95 latency (seconds) from classic histogram buckets:**

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(http_request_duration_seconds_bucket{job="sample-api"}[5m])
  )
)
```

**CPU use per Kubernetes container (cores):**

```promql
sum by (namespace, pod, container) (
  rate(container_cpu_usage_seconds_total{container!="",image!=""}[5m])
)
```

**Pod restarts over 1 hour:**

```promql
sum by (namespace, pod) (
  increase(kube_pod_container_status_restarts_total[1h])
)
```

### LogQL (Grafana Loki)

**Logs for one application:**

```logql
{namespace="production", app="payments-api"}
```

**Filter for error lines:**

```logql
{namespace="production", app="payments-api"} |= "ERROR"
```

**Count log lines per second for a stream selector:**

```logql
sum(rate({namespace="production", app="payments-api"}[5m]))
```

> Loki label names depend on the log collector's configuration. Use low-cardinality labels; do not use request IDs as indexed labels.

## 14. SLO and Burn-Rate Alert Example

### Example: 99.9% request-success SLO over 30 days

- Error budget = `1 - 0.999 = 0.001` (0.1% of eligible requests).
- Burn rate `1` means errors are occurring at exactly the average allowed rate.
- Burn rate `14.4` means the budget is being consumed 14.4 times faster than the sustainable rate.
- A standard **multiwindow, multi-burn-rate** paging design checks both a long and short window:

| Severity | Long window | Short window | Burn rate | Approx. budget consumed |
|---|---|---|---|---|
| Page | 1 hour | 5 minutes | 14.4× | 2% |
| Page | 6 hours | 30 minutes | 6× | 5% |
| Ticket | 3 days | 6 hours | 1× | 10% |

**Illustrative Prometheus rule** (one fast-burn page path; assumes no traffic gaps and the sample metric names above):

```yaml
groups:
  - name: sample-api-slo
    rules:
      - record: sample_api:error_ratio_rate5m
        expr: |
          sum(rate(http_requests_total{job="sample-api",status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total{job="sample-api"}[5m]))

      - record: sample_api:error_ratio_rate1h
        expr: |
          sum(rate(http_requests_total{job="sample-api",status=~"5.."}[1h]))
          /
          sum(rate(http_requests_total{job="sample-api"}[1h]))

      - alert: SampleAPIHighErrorBudgetBurn
        expr: |
          (sample_api:error_ratio_rate1h > (14.4 * 0.001))
          and
          (sample_api:error_ratio_rate5m > (14.4 * 0.001))
        labels:
          severity: page
          service: sample-api
        annotations:
          summary: "Sample API is rapidly consuming its error budget"
          description: "Check request failure rates, recent deployments and dependencies."
          runbook_url: "https://example.com/runbooks/sample-api" # replace
```

Production setups should also account for low traffic, missing series, valid/invalid request classification, routing, ownership, and the slower-burn alert paths.

Reference: [Google SRE Workbook — Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/).

## 15. Reference SRE Architecture

```text
                  ┌─────────────────────┐
                  │ Developers / GitOps │
                  │ CI + container image│
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │ Kubernetes / VMs    │
                  │ API + dependencies  │
                  └───────┬─────────────┘
                          │
          ┌───────────────┼─────────────────────┐
          ▼               ▼                     ▼
       Metrics           Logs                 Traces
   Prometheus /       Alloy/Fluent Bit       OpenTelemetry
    exporters              │                 Collector
          │                 │                     │
          ▼                 ▼                     ▼
   Prometheus /         Grafana Loki          Tempo / Jaeger
   Thanos/Mimir             │                     │
          └─────────────────┼─────────────────────┘
                            ▼
                      Grafana UI
                  (dashboards + explore)
                            │
                            ▼
                 Alert rules / Alertmanager
                            │
                            ▼
                    PagerDuty / on-call
                            │
                            ▼
              Runbook → mitigation → postmortem
                            │
                            ▼
                 Automation / improvement
```

This is a **reference design**, not a mandatory architecture. A managed platform may provide multiple functions in one service; collectors and telemetry routing vary by environment.

## 16. Practical End-to-End SRE Project

**Project:** Instrument and operate a containerized FastAPI application on Kubernetes.

### Target stack

```text
FastAPI → Docker → Kubernetes (minikube or EKS)
                ├── Prometheus + Grafana (metrics)
                ├── Loki + log collector (logs)
                ├── OpenTelemetry + Tempo (traces)
                ├── Alertmanager + incident notification
                ├── k6 (load testing)
                └── GitHub Actions/Jenkins + Argo CD (delivery)
```

### Implementation checklist

- [ ] **1. App:** Create `/health` (liveness), `/ready` (readiness), and `/metrics`; add representative API routes.
- [ ] **2. Container:** Create non-root Docker image and push it to a registry.
- [ ] **3. Deploy:** Use Helm with probes, requests/limits, replica count, and a PodDisruptionBudget where appropriate.
- [ ] **4. Metrics:** Instrument HTTP request counts, durations, and domain-specific error metrics.
- [ ] **5. Collection:** Install Prometheus and verify scrape targets are `UP`.
- [ ] **6. Dashboard:** Build Golden Signals/RED charts in Grafana.
- [ ] **7. Logs:** Emit JSON logs and forward to Loki; correlate logs with trace IDs.
- [ ] **8. Traces:** Enable OTel instrumentation and send traces to Tempo.
- [ ] **9. Alerts:** Configure actionable high-error-rate and SLO burn-rate rules.
- [ ] **10. Runbook:** Write a repeatable diagnostic and recovery procedure.
- [ ] **11. Load test:** Use k6 to test steady and burst traffic; measure p95/p99 latency.
- [ ] **12. Failure drill:** Cause a controlled pod/dependency failure; validate detection and recovery.
- [ ] **13. Delivery:** Add CI tests, image scan, immutable image reference, and GitOps update.
- [ ] **14. Safer rollout:** Add canary traffic and automated health checks before promotion.
- [ ] **15. Postmortem:** Record detection time, recovery time, impact, contributing causes, and action items.

**Suggested repository layout:**

```text
sre-observability-lab/
├── app/
│   ├── main.py
│   └── requirements.txt
├── tests/
├── Dockerfile
├── helm/
│   └── sample-api/
├── monitoring/
│   ├── prometheus-rules.yaml
│   ├── dashboards/
│   └── otel-collector.yaml
├── load-tests/
│   └── load.js
├── runbooks/
│   └── high-error-rate.md
├── postmortems/
└── README.md
```

**Deliverables to showcase:** screenshots of Grafana panels, sample alerts, a trace waterfall, load-test results, a documented incident drill, and CI/CD configuration. Do not publish secrets or internal production data.

## 17. Common Production Incident Walkthrough

**Scenario:** Checkout API starts returning 15% HTTP 5xx errors immediately after a deployment.

| Step | Investigation | Tool | Action/result |
|---|---|---|---|
| 1. Detect | Sustained error-rate and SLO burn alert | Prometheus + Alertmanager | Page on-call engineer |
| 2. Scope | Which endpoints, regions, versions are affected? | Grafana / PromQL | Identify the impacted workload |
| 3. Correlate | Does the spike match a release? | Grafana annotations + CI/CD | Compare current and previous release |
| 4. Diagnose | Review error lines and trace spans | Loki + Tempo/Jaeger | Find dependency timeouts / failing calls |
| 5. Inspect | Restarts, readiness, events, resource pressure | `kubectl`, `kubectl top` | Rule out OOM/restart/capacity issues |
| 6. Mitigate | Stop canary or roll back safely | Argo Rollouts / deployment tool | Restore healthy service behavior |
| 7. Verify | Confirm error rate and latency returned to normal | Grafana / Prometheus | Check from user-impact perspective |
| 8. Follow up | Write blameless postmortem | Incident platform / wiki | Add tests, alerts, runbook improvements |

**Interview framing:** explain the **signal that triggered detection**, **how you reduced impact**, **what evidence established the cause**, and **what preventive changes you implemented**. Do not invent results or improvement percentages.

## 18. Interview Questions and Answers

**Q1. Prometheus vs. Grafana?**  
Prometheus scrapes and queries time-series metrics and evaluates alert rules. Grafana visualizes data from Prometheus and other sources, and can also provide alerting.

**Q2. Monitoring vs. observability?**  
Monitoring checks expected failure modes and health indicators. Observability helps investigate unexpected behavior using metrics, logs, traces, and other signals.

**Q3. Metrics vs. logs vs. traces?**  
Metrics summarize trends, logs record events, and traces map end-to-end request execution.

**Q4. What is OpenTelemetry?**  
A vendor-neutral framework for instrumenting and exporting telemetry; it does not replace storage/query backends such as Prometheus, Loki, or Tempo.

**Q5. Why use Alertmanager?**  
To route, group, deduplicate, inhibit, and silence metric-driven notifications.

**Q6. What is an SLO?**  
A measurable reliability objective on an SLI over a defined window, such as 99.9% successful eligible requests in 30 days.

**Q7. What is an error budget?**  
The allowed amount of unreliability derived from the SLO; it helps balance release velocity and reliability work.

**Q8. Why monitor p95/p99 latency instead of only averages?**  
Averages can hide slow experiences affecting a meaningful subset of users; percentiles expose the tail.

**Q9. What is alert fatigue?**  
Too many low-quality or duplicate alerts make engineers miss important problems. Use actionable paging, grouping, and SLO-based signals.

**Q10. What is an SLO burn rate?**  
The speed of consuming the remaining reliability allowance relative to the rate sustainable across the SLO window.

**Q11. What is a runbook?**  
A documented, tested response procedure for recurring operational conditions, including verification and rollback steps.

**Q12. Horizontal vs. vertical scaling?**  
Horizontal adds replicas/instances; vertical adds CPU/memory to a given instance. Both have capacity and failure-mode tradeoffs.

**Q13. What is a readiness probe?**  
A signal indicating whether a pod should receive service traffic; it is distinct from liveness and startup probes.

**Q14. Canary vs. blue-green?**  
Canary gradually exposes a subset of traffic to a new release. Blue-green prepares a separate environment and switches serving traffic between versions.

**Q15. How would you troubleshoot high latency?**  
Check request rate, p95/p99, errors, saturation; correlate traces/logs; inspect dependencies, deployments, and resources; mitigate before deep root-cause work if users are affected.

**Q16. How do you reduce MTTR?**  
Improve detection, ownership, runbooks, observability, rollback automation, incident coordination, and postmortem follow-through.

**Q17. What is chaos engineering?**  
Controlled fault injection to validate reliability assumptions with explicit safety limits and rollback/abort criteria.

**Q18. What would your first SRE dashboard show?**  
Traffic, success rate, p95 latency, saturation, SLO/error budget status, and recent deployment changes.

## 19. SRE Learning Roadmap

| Stage | Priorities | Practice outcome |
|---|---|---|
| **1. Foundations** | Linux, networking, HTTP, DNS, TLS, scripting | Diagnose host and service issues |
| **2. Cloud and Kubernetes** | Docker, Kubernetes, load balancing, IAM | Deploy a resilient multi-replica app |
| **3. Observability** | Prometheus, Grafana, Loki, OTel, Tempo | Explain user-impacting latency/errors |
| **4. Reliability** | SLIs, SLOs, error budgets, Alertmanager | Build actionable alert rules and runbooks |
| **5. Operations** | Incident management, postmortems, backups, DR | Recover from a simulated failure |
| **6. Automation** | CI/CD, Terraform, GitOps, Argo Rollouts | Release safely and roll back reliably |
| **7. Advanced** | Capacity planning, k6, chaos, cost controls | Validate resilience under load and failure |

**Suggested first priorities for a DevOps engineer moving into SRE:**

1. Prometheus + PromQL + Grafana dashboards.
2. Linux/Kubernetes incident debugging.
3. Alertmanager + on-call workflow + runbooks.
4. SLI/SLO/error-budget design and burn-rate alerts.
5. Loki + OpenTelemetry + Tempo for correlation.
6. k6, canary deployments, and controlled failure drills.

## 20. Official Documentation

- [Google SRE Book](https://sre.google/sre-book/table-of-contents/) and [SRE Workbook](https://sre.google/workbook/table-of-contents/)
- [Google SRE Workbook — Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
- [Prometheus Documentation](https://prometheus.io/docs/) and [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/)
- [Grafana Documentation](https://grafana.com/docs/grafana/latest/)
- [Grafana Loki](https://grafana.com/docs/loki/latest/) and [Grafana Tempo](https://grafana.com/docs/tempo/latest/)
- [OpenTelemetry](https://opentelemetry.io/docs/)
- [Kubernetes Debugging](https://kubernetes.io/docs/tasks/debug/)
- [Argo Rollouts](https://argoproj.github.io/argo-rollouts/)
- [Argo CD](https://argo-cd.readthedocs.io/)
- [k6 Documentation](https://grafana.com/docs/k6/latest/)
- [KEDA](https://keda.sh/docs/)
- [Terraform](https://developer.hashicorp.com/terraform/docs)

---

**Takeaway:** Great SREs are not defined by the number of tools they know. They are effective at **measuring reliability, detecting user-impacting problems, diagnosing quickly, restoring service safely, automating repetitive work, and learning from incidents**.

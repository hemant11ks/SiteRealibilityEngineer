# Site Reliability Engineer (SRE) Notes

## 1. What is SRE?

**Site Reliability Engineering (SRE)** applies software engineering principles to infrastructure, operations, availability, scalability, and reliability.

Traditional Operations:

`Manual Operations → Monitoring → Fix Problems`

SRE:

`Software Engineering + Automation + Operations + Reliability`

Main objective:

> Keep systems reliable while allowing developers to release features quickly.

---

## 2. SRE vs DevOps

| DevOps | SRE |
|---|---|
| Culture and methodology | Engineering implementation of reliability |
| Focus on development + operations collaboration | Focus on reliability and availability |
| CI/CD automation | Reliability automation |
| Infrastructure automation | SLO/Error Budget management |
| Faster software delivery | Safe and reliable software delivery |

SRE can be considered one practical implementation of DevOps principles.

---

# 3. Core Responsibilities of an SRE

An SRE commonly works on:

- System reliability
- Availability
- Performance
- Scalability
- Monitoring
- Incident management
- Capacity planning
- Infrastructure automation
- CI/CD reliability
- Disaster recovery
- Production troubleshooting
- On-call support
- Root Cause Analysis
- Reducing operational toil

---

# 4. SLI, SLO and SLA

These are among the most important SRE concepts.

## SLI – Service Level Indicator

A **measured value** representing service reliability.

Examples:

- Availability
- Request latency
- Error rate
- Throughput
- Successful request percentage

Example:

```text
Successful Requests
-------------------- × 100
Total Requests
```

If:

```text
9990 successful requests
10000 total requests
```

SLI:

```text
99.9%
```

---

## SLO – Service Level Objective

The reliability target the engineering team wants to maintain.

Example:

```text
Availability SLO = 99.9%
```

Meaning the service should successfully handle at least 99.9% of requests.

---

## SLA – Service Level Agreement

A formal agreement with customers.

Example:

```text
SLA = 99.9% monthly uptime
```

If the SLA is violated, the company might provide:

- Service credits
- Refunds
- Financial compensation

Simple relationship:

```text
SLI = What we measure
SLO = What we target
SLA = What we promise
```

---

# 5. Availability

Availability tells us how often a system is accessible.

Formula:

```text
Availability =
Uptime
---------------- × 100
Uptime + Downtime
```

Typical availability targets:

| Availability | Approximate Downtime / Year |
|---|---:|
| 99% | ~3.65 days |
| 99.9% | ~8.76 hours |
| 99.99% | ~52.6 minutes |
| 99.999% | ~5.26 minutes |

Higher availability normally means higher infrastructure and engineering cost.

---

# 6. Error Budget

Error budget determines how much unreliability is acceptable.

Formula:

```text
Error Budget = 100% - SLO
```

Example:

```text
SLO = 99.9%

Error Budget = 0.1%
```

If the error budget is healthy:

```text
Release features faster
Experiment
Deploy frequently
```

If the error budget is exhausted:

```text
Reduce risky releases
Focus on reliability
Fix recurring failures
Improve testing
```

Error budgets balance:

```text
Innovation ↔ Reliability
```

---

# 7. Four Golden Signals

Google SRE describes four major signals to monitor.

## Latency

How long requests take.

Example:

```text
API latency
p50 = 100ms
p95 = 300ms
p99 = 900ms
```

---

## Traffic

How much demand the application receives.

Examples:

```text
Requests/second
Transactions/second
Active users
Network throughput
```

---

## Errors

Rate of unsuccessful operations.

Examples:

```text
HTTP 500
HTTP 502
Failed database queries
Application exceptions
```

---

## Saturation

How close the system is to its maximum capacity.

Examples:

```text
CPU = 95%
Memory = 90%
Disk = 98%
Connection pool exhausted
```

Remember:

```text
Latency
Traffic
Errors
Saturation
```

---

# 8. Monitoring vs Observability

## Monitoring

Answers:

> Is something wrong?

Example:

```text
CPU usage > 90%
Disk usage > 85%
API error rate > 5%
```

Tools:

- Prometheus
- Grafana
- CloudWatch
- Nagios
- Zabbix

---

## Observability

Answers:

> Why is something wrong?

Three major pillars:

```text
Metrics
Logs
Traces
```

Common stack:

```text
Application
    ↓
Metrics → Prometheus
Logs → Elasticsearch / Loki
Traces → OpenTelemetry
    ↓
Grafana
```

---

# 9. Metrics

Metrics are numerical measurements collected over time.

Examples:

```text
CPU usage
RAM usage
Disk usage
HTTP requests
HTTP errors
Request duration
Build duration
Deployment frequency
```

Prometheus metric types:

### Counter

Value only increases.

```text
http_requests_total
```

### Gauge

Value can increase or decrease.

```text
memory_usage_bytes
```

### Histogram

Measures distributions.

```text
http_request_duration_seconds
```

### Summary

Similar to histogram but calculates quantiles on the client side.

---

# 10. Logs

Logs explain individual events.

Example:

```text
2026-09-26 10:30:01 ERROR Database connection failed
```

A useful log should contain:

```text
timestamp
severity
service
request_id
message
error details
```

Common stack:

```text
Application
     ↓
Logstash / Fluent Bit
     ↓
Elasticsearch
     ↓
Kibana
```

or

```text
Application
     ↓
Loki
     ↓
Grafana
```

---

# 11. Distributed Tracing

Tracing follows a request across multiple services.

Example:

```text
User
 ↓
API Gateway
 ↓
Authentication Service
 ↓
Order Service
 ↓
Payment Service
 ↓
Database
```

Tracing helps identify where latency or failures occur.

Common technologies:

```text
OpenTelemetry
Jaeger
Zipkin
Grafana Tempo
```

---

# 12. Alerting

A good alert must be:

```text
Actionable
Relevant
Urgent
Specific
```

Bad alert:

```text
CPU = 72%
```

Better alert:

```text
API p95 latency > 2 seconds
for 10 minutes
```

Avoid:

```text
Alert fatigue
```

Too many unnecessary alerts make engineers ignore important alerts.

---

# 13. Incident Management

Typical incident lifecycle:

```text
Alert
 ↓
Detection
 ↓
Triage
 ↓
Mitigation
 ↓
Recovery
 ↓
Root Cause Analysis
 ↓
Preventive Action
```

Incident priorities may look like:

```text
P0 – Critical outage
P1 – Major degradation
P2 – Partial impact
P3 – Minor problem
```

---

# 14. MTTR, MTBF, MTTD

## MTTD

Mean Time To Detect.

```text
Failure occurs
↓
Monitoring detects it
```

Lower is better.

## MTTR

Mean Time To Recovery / Repair.

```text
Failure detected
↓
Service restored
```

Lower is better.

## MTBF

Mean Time Between Failures.

```text
Failure
↓
Stable period
↓
Next failure
```

Higher is better.

---

# 15. Root Cause Analysis

RCA is performed after significant incidents.

A good RCA contains:

```text
Incident summary
Timeline
Impact
Root cause
Trigger
Detection method
Resolution
Corrective actions
Preventive actions
```

Important principle:

> Blameless Postmortem

Focus on improving systems rather than blaming individuals.

---

# 16. Toil

Toil is repetitive operational work that:

- Is manual
- Is repetitive
- Can be automated
- Provides little long-term value
- Grows as the system grows

Examples:

```text
Manually restarting services
Manually checking disk space
Manually cleaning old files
Manually deploying builds
Manually generating reports
```

SRE principle:

```text
Automate repetitive operational work.
```

Possible automation:

```text
Python
Bash
Ansible
Terraform
Jenkins
Kubernetes
```

---

# 17. Reliability Patterns

## Load Balancing

```text
Users
 ↓
Load Balancer
 ↓
App1  App2  App3
```

Prevents a single server from handling all requests.

---

## Auto Scaling

Increase or decrease infrastructure depending on demand.

Example:

```text
CPU > 70%
↓
Add pods
```

Kubernetes:

```text
Horizontal Pod Autoscaler
```

---

## Redundancy

Avoid single points of failure.

Bad:

```text
Application → One Database
```

Better:

```text
Application
     ↓
Primary Database
     ↓
Replica Database
```

---

## Retry

Retry temporary failures.

But avoid unlimited retries.

Use:

```text
Retry
+
Exponential Backoff
```

Example:

```text
1 sec
2 sec
4 sec
8 sec
```

---

## Circuit Breaker

Stops repeatedly calling an unhealthy service.

States:

```text
CLOSED
OPEN
HALF-OPEN
```

This prevents cascading failures.

---

## Timeout

Every remote request should have a reasonable timeout.

Bad:

```text
Wait forever
```

Better:

```text
Timeout = 5 seconds
```

---

# 18. High Availability

High availability means reducing system downtime through redundancy.

Example:

```text
             Load Balancer
            /             \
        Server A        Server B
```

If Server A fails:

```text
Traffic → Server B
```

Important concepts:

```text
Failover
Replication
Load Balancing
Health Checks
Multi-AZ deployment
```

---

# 19. Scalability

Two major types.

## Vertical Scaling

Increase server resources.

```text
4 CPU → 16 CPU
8 GB RAM → 64 GB RAM
```

Also called:

```text
Scale Up
```

## Horizontal Scaling

Increase server count.

```text
2 servers → 10 servers
```

Also called:

```text
Scale Out
```

Horizontal scaling is generally preferred for highly distributed systems.

---

# 20. Disaster Recovery

Important terms:

## RTO

Recovery Time Objective.

> Maximum acceptable time to restore the service.

Example:

```text
RTO = 30 minutes
```

## RPO

Recovery Point Objective.

> Maximum acceptable data loss.

Example:

```text
RPO = 5 minutes
```

If:

```text
RPO = 5 minutes
```

the business accepts losing at most approximately five minutes of data.

---

# 21. Kubernetes for SRE

Important Kubernetes concepts:

```text
Pod
Deployment
ReplicaSet
Service
Ingress
ConfigMap
Secret
Namespace
PersistentVolume
PersistentVolumeClaim
DaemonSet
StatefulSet
HPA
Requests
Limits
Liveness Probe
Readiness Probe
```

---

## Readiness Probe

Checks:

> Can this pod receive traffic?

If readiness fails:

```text
Pod remains running
but is removed from Service traffic
```

---

## Liveness Probe

Checks:

> Is this application alive?

If liveness fails:

```text
Kubernetes restarts the container
```

---

# 22. Kubernetes Troubleshooting

Basic workflow:

```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl get events
kubectl top pods
kubectl top nodes
```

Networking:

```bash
kubectl get svc
kubectl get ingress
kubectl get endpoints
```

Deployment:

```bash
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
```

---

# 23. Linux Knowledge for SRE

Important commands:

### CPU

```bash
top
htop
uptime
lscpu
```

### Memory

```bash
free -h
vmstat
```

### Disk

```bash
df -h
du -sh *
lsblk
```

### Processes

```bash
ps aux
top
pgrep
kill
kill -9
```

### Network

```bash
ss -tulpn
ip addr
ip route
ping
curl
traceroute
nslookup
dig
```

### Logs

```bash
journalctl
journalctl -u service
dmesg
tail -f file.log
```

---

# 24. Networking Basics for SRE

Important topics:

```text
TCP/IP
DNS
HTTP
HTTPS
TLS
Load Balancer
Reverse Proxy
Firewall
NAT
CIDR
Subnet
Routing
Ports
```

Common ports:

| Service | Port |
|---|---:|
| SSH | 22 |
| HTTP | 80 |
| HTTPS | 443 |
| DNS | 53 |
| MySQL | 3306 |
| PostgreSQL | 5432 |
| Jenkins | 8080 |
| Prometheus | 9090 |
| Grafana | 3000 |

---

# 25. HTTP Status Codes

Important groups:

```text
1xx → Informational
2xx → Success
3xx → Redirect
4xx → Client Error
5xx → Server Error
```

Important codes:

```text
200 OK
201 Created
301 Permanent Redirect
302 Temporary Redirect
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
429 Too Many Requests
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

---

# 26. CI/CD Reliability

SRE should understand the complete software delivery pipeline.

Example:

```text
Developer
 ↓
Git / Gerrit
 ↓
Jenkins
 ↓
Build
 ↓
Unit Tests
 ↓
Static Analysis
 ↓
Artifact Repository
 ↓
Deployment
 ↓
Kubernetes
 ↓
Monitoring
```

Important metrics:

```text
Build success rate
Deployment frequency
Deployment failure rate
Pipeline duration
Rollback rate
Mean Time To Recovery
```

---

# 27. Infrastructure as Code

Infrastructure should be reproducible.

Common tools:

```text
Terraform
Ansible
Helm
CloudFormation
```

Example Terraform workflow:

```text
terraform init
terraform validate
terraform plan
terraform apply
```

Benefits:

```text
Version controlled
Repeatable
Reviewable
Automated
Consistent
```

---

# 28. Deployment Strategies

## Rolling Deployment

Replace instances gradually.

```text
V1 V1 V1
↓
V2 V1 V1
↓
V2 V2 V1
↓
V2 V2 V2
```

## Blue-Green

```text
Blue = Production V1
Green = New V2
```

Switch traffic when Green is ready.

## Canary

Release to a small percentage first.

```text
95% → V1
5% → V2
```

Then gradually increase V2 traffic.

---

# 29. Capacity Planning

SREs need to answer:

```text
How much traffic can the system handle?
When will resources become insufficient?
How much infrastructure will we need next month?
```

Consider:

```text
CPU
Memory
Disk
Network
Database connections
Requests/sec
User growth
```

---

# 30. On-Call Engineering

On-call engineers handle production incidents.

Good on-call practices:

```text
Clear runbooks
Useful alerts
Escalation policies
Automated remediation
Incident documentation
Postmortems
```

The goal should not be:

```text
Wake engineers up frequently
```

The goal should be:

```text
Build systems reliable enough that pages are rare and meaningful.
```

---

# 31. Runbook

A runbook contains instructions for resolving known incidents.

Example:

```text
Alert:
Disk usage > 90%

Check:
df -h

Identify:
du -sh /*

Action:
Clean old logs

Verify:
df -h

Escalate:
If usage remains > 85%
```

---

# 32. SRE Troubleshooting Approach

When production is down:

```text
1. Check monitoring
2. Identify affected services
3. Check recent deployments
4. Check application logs
5. Check infrastructure metrics
6. Check network connectivity
7. Check dependencies
8. Mitigate impact
9. Restore service
10. Find root cause
11. Prevent recurrence
```

Important principle:

```text
Restore service first.
Investigate deeper afterward.
```

---

# 33. Common SRE Tools

### CI/CD

```text
Jenkins
GitLab CI/CD
GitHub Actions
Argo CD
Tekton
```

### Containers

```text
Docker
Kubernetes
Helm
```

### Infrastructure

```text
Terraform
Ansible
Packer
```

### Monitoring

```text
Prometheus
Grafana
CloudWatch
Datadog
```

### Logging

```text
Elasticsearch
Logstash
Kibana
Loki
```

### Tracing

```text
OpenTelemetry
Jaeger
Tempo
```

### Cloud

```text
AWS
Azure
GCP
OCI
```

---

# 34. Important SRE Interview Scenario

### Scenario

Users report that an application is slow.

Troubleshooting:

```text
Check latency metrics
      ↓
Check request/error rate
      ↓
Check CPU/memory
      ↓
Check application logs
      ↓
Check database latency
      ↓
Check external dependencies
      ↓
Check recent deployments
      ↓
Check network latency
      ↓
Identify bottleneck
```

Never immediately assume CPU or RAM is the problem.

---

# 35. Another Important Scenario

Production deployment caused failures.

Immediate actions:

```text
Check deployment status
↓
Measure user impact
↓
Stop further rollout
↓
Rollback if required
↓
Verify recovery
↓
Investigate logs
↓
Perform RCA
```

---

# 36. SRE Principles to Remember

```text
Automate repetitive work.

Measure reliability.

Define SLOs.

Use error budgets.

Monitor user-impacting signals.

Design for failure.

Reduce single points of failure.

Prefer automation over manual intervention.

Create actionable alerts.

Perform blameless postmortems.

Build scalable systems.

Make recovery fast.
```

---

# 37. Important Interview Topics

For an SRE role, prepare these areas strongly:

```text
Linux
Networking
Python/Bash
Cloud
Docker
Kubernetes
CI/CD
Terraform
Monitoring
Prometheus
Grafana
Logging
Incident Management
SLI/SLO/SLA
Error Budgets
High Availability
Disaster Recovery
System Design
Troubleshooting
```

---

# 38. Quick SRE Revision Sheet

Remember:

```text
SLI = Measurement
SLO = Reliability target
SLA = Customer agreement

Error Budget = 100% - SLO

Golden Signals:
Latency
Traffic
Errors
Saturation

Observability:
Metrics
Logs
Traces

MTTD = Detect problem
MTTR = Recover from problem
MTBF = Time between failures

RTO = How quickly to recover
RPO = How much data loss is acceptable

SRE =
Reliability
+
Automation
+
Software Engineering
+
Operations
```

---

# 39. SRE Mindset

A strong SRE does not only ask:

> How do I fix this failure?

They also ask:

> Why did this failure occur?

> Why didn't monitoring detect it earlier?

> Can we automatically detect it next time?

> Can we automatically recover from it?

> How can we prevent it from happening again?

That mindset is one of the biggest differences between simply operating infrastructure and practicing Site Reliability Engineering.

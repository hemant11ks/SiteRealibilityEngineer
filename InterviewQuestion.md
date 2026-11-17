# Site Reliability Engineering (SRE) — Interview Questions & Answers

> **GitHub-ready study guide | Beginner → Intermediate → Advanced | Updated: 17 November 2026**  
> Focus: **SRE / DevOps / Platform Engineer interviews**, especially hands-on troubleshooting, on-call decisions, observability, Kubernetes, cloud infrastructure, and automation.

This guide combines **interview-ready answers**, useful commands, trade-offs, and realistic production incidents. The examples use Linux, Kubernetes, Prometheus, Grafana, AWS, Terraform, Jenkins, and Python; the concepts apply across cloud providers. **Example numbers, targets, and incident stories are illustrative**, not records of any particular production system.

## How to use this guide

1. Read a question and answer it out loud in **60–90 seconds**, before expanding the answer.
2. For technical questions, explain **what you would check first**, **how you would confirm the cause**, and **how you would limit user impact**.
3. Reproduce the commands in a **lab**; do not run disruptive commands in production without authorization.
4. For incident questions, use: **Impact → Evidence → Mitigation → Root cause → Prevention**.
5. In interviews, distinguish **tools used hands-on** from tools you've only studied or tested.

## Table of contents

- [1. SRE fundamentals and reliability](#1-sre-fundamentals-and-reliability)
- [2. SLIs, SLOs, SLAs, and error budgets](#2-slis-slos-slas-and-error-budgets)
- [3. Linux and operating systems](#3-linux-and-operating-systems)
- [4. Networking and HTTP](#4-networking-and-http)
- [5. Kubernetes and containers](#5-kubernetes-and-containers)
- [6. Monitoring, logging, and tracing](#6-monitoring-logging-and-tracing)
- [7. Incident management and on-call](#7-incident-management-and-on-call)
- [8. AWS, cloud, and disaster recovery](#8-aws-cloud-and-disaster-recovery)
- [9. CI/CD, Terraform, and GitOps](#9-cicd-terraform-and-gitops)
- [10. Databases, storage, and distributed systems](#10-databases-storage-and-distributed-systems)
- [11. Security, capacity, and automation](#11-security-capacity-and-automation)
- [12. Real-world production scenarios](#12-real-world-production-scenarios)
- [13. PromQL and command cheat sheets](#13-promql-and-command-cheat-sheets)
- [14. Behavioral and experience-based questions](#14-behavioral-and-experience-based-questions)
- [15. Practical labs and revision checklist](#15-practical-labs-and-revision-checklist)
- [Official references](#official-references)

---

## 1. SRE fundamentals and reliability

### Q001. What is Site Reliability Engineering (SRE)?
**Answer:** SRE is an approach to operating software systems using engineering practices. It applies software development, automation, measurement, and incident management to achieve a **defined level of reliability**. A good SRE balances customer experience and release velocity rather than pursuing zero failures at any cost.

**Interview angle:** Mention monitoring, SLIs/SLOs, reducing toil, capacity planning, safe deployments, incident response, and learning from failures.

### Q002. How is SRE different from DevOps?
**Answer:** DevOps is a broader culture and set of practices emphasizing collaboration, continuous delivery, and shared ownership. SRE is a concrete way to implement many of these ideas, with explicit reliability targets, error budgets, operational engineering, and on-call practices. Teams may use both.

### Q003. What is reliability, availability, and durability?
**Answer:** **Reliability** is the ability of a system to perform its intended function over time. **Availability** is the proportion of time or requests when the service is usable, measured against a defined criterion. **Durability** is the ability to preserve data once it has been successfully stored. A service can be available while losing data, or durable while temporarily unavailable.

### Q004. What is toil? How do you reduce it?
**Answer:** Toil is repetitive, manual, tactical operational work that scales with service size and provides little enduring value. Examples: rerunning flaky jobs, manually restarting services, and copying release artifacts. First measure frequency and time spent; then address root causes, write safe automation, improve self-service, and eliminate unnecessary work. Automation is not enough if the underlying failure keeps recurring.

### Q005. What are the four golden signals?
**Answer:** **Latency**, **Traffic**, **Errors**, and **Saturation**. For a web API: p95 response time; requests/second; fraction of failed requests; and utilization/queue pressure on constrained resources. These provide a starting point, not a replacement for service-specific business SLIs.

### Q006. What is the RED method and when is it useful?
**Answer:** **Rate, Errors, Duration** is a service-centric monitoring method. It is useful for APIs, RPC services, and requests crossing a service boundary. Rate tracks throughput; Errors captures unsuccessful operations; Duration tracks distributions of response times. It overlaps with the golden signals but does not explicitly measure resource saturation.

### Q007. What is the USE method?
**Answer:** **Utilization, Saturation, Errors** measures infrastructure resources such as CPUs, disks, networks, and storage. Examples: CPU busy percentage, run-queue depth, and disk I/O errors. Pair USE with RED to connect resource issues to user-facing symptoms.

### Q008. What is a single point of failure (SPOF)?
**Answer:** A component whose failure can take down the service. Examples include a lone database node, one region without failover, or one manually managed credentials server. Mitigate by redundancy, automated failover, backups, isolation, and testing the recovery path. Redundancy alone is not sufficient if failures are correlated.

### Q009. What is a dependency map and why does an SRE need it?
**Answer:** A map of services and their dependencies—databases, queues, identity, DNS, third-party APIs, and network paths. It helps identify blast radius, locate the owner of a failing component, and make informed incident decisions. Keep it updated through service catalogs and telemetry rather than treating it as a one-time diagram.

### Q010. What is the difference between fault tolerance and high availability?
**Answer:** **High availability** aims to minimize interruption through redundancy and recovery. **Fault tolerance** allows a system to continue operating despite certain component failures, often without a visible interruption. The concepts overlap, but fault tolerance is typically the stricter property; both depend on the failure assumptions.

### Q011. Explain graceful degradation and load shedding.
**Answer:** **Graceful degradation** preserves critical functionality when some features or dependencies fail (for example, serving cached product listings without recommendations). **Load shedding** deliberately rejects or drops excess work to protect the service from total overload. Both need explicit prioritization and measurement of user impact.

### Q012. What are blast radius and failure domains?
**Answer:** **Blast radius** is the scope of impact of a failure or bad change. A **failure domain** is a group of resources that may fail together (node, rack, availability zone, region, shared dependency). Reduce blast radius using canary releases, quotas, cell-based architecture, independent zones, and controlled rollouts.

---

## 2. SLIs, SLOs, SLAs, and error budgets

### Q013. Explain SLI vs SLO vs SLA with an example.
**Answer:** A **Service Level Indicator (SLI)** is a measurement (for example, percentage of successful checkout requests). A **Service Level Objective (SLO)** is the target (99.9% successful over 30 days). A **Service Level Agreement (SLA)** is a contractual commitment that may include consequences when conditions are not met. An SLO is an engineering decision; an SLA is a business agreement.

### Q014. How do you calculate availability from requests?
**Answer:** For a request-based SLI, `availability = good_eligible_requests / all_eligible_requests × 100`. The key is defining **eligible** and **good** consistently—for example, whether user-caused 4xx responses count as failures. Time-based uptime and request-based availability are different indicators.

### Q015. What is an error budget?
**Answer:** The allowable unreliability within an SLO window. If the target is **99.9%**, then **0.1%** of eligible requests can fail in the measurement window (using a request-based SLI). Teams use remaining budget to balance reliability investments and the risk of releases, based on an agreed policy.

### Q016. How much downtime does 99.9% availability allow in a 30-day month?
**Answer:** Under a **time-based availability model**, 30 days = 43,200 minutes. At 99.9%, the downtime allowance is `43,200 × 0.001 = 43.2 minutes`. For request-based SLOs, there is no direct fixed downtime allowance because traffic varies over time.

| Time-based target | Approx. downtime in 30 days |
|---|---:|
| 99% | 7 hours 12 minutes |
| 99.9% | 43 minutes 12 seconds |
| 99.95% | 21 minutes 36 seconds |
| 99.99% | 4 minutes 19 seconds |

### Q017. What is error-budget burn rate?
**Answer:** The speed at which the service spends its permitted failure budget relative to the sustainable rate. For a 99.9% SLO, the permitted error fraction is `0.001`; if the measured error fraction is `0.0144` (1.44%), the burn rate is `0.0144 / 0.001 = 14.4×`. A high burn rate justifies faster action than a slow one.

### Q018. What is multiwindow, multi-burn-rate alerting?
**Answer:** An SLO alerting approach that compares error budget burn across both **long and short windows**, reducing delayed recovery and unnecessary pages. Google’s SRE Workbook illustrates a 99.9% SLO with paging conditions such as **14.4× over 1 hour AND 5 minutes**, or **6× over 6 hours AND 30 minutes**. These are example starting points, not universal thresholds.

### Q019. Why is a 100% uptime SLO usually a poor target?
**Answer:** Most real services have dependencies and changing environments. A 100% target leaves no error budget and can discourage safe iteration. Define an ambitious but achievable target based on user needs, reliability economics, and realistic failure modes; extremely critical systems may warrant much stricter targets and architecture.

### Q020. How do you choose a useful SLI for an API?
**Answer:** Start with a user journey (such as successful payment). Define the eligible population and what constitutes success. Capture availability, latency (for example, proportion of requests under a threshold), and correctness where relevant. Measure at a meaningful boundary; infrastructure uptime alone may not reflect what customers experience.

### Q021. What is the difference between p50, p95, and p99 latency?
**Answer:** They are percentiles of request duration: p50 is the median, p95 means 95% of observed requests completed at or below that latency, and p99 captures a slower tail. Tail latency matters because a small fraction of slow requests can harm many users at scale. Always specify the time window, population, and aggregation method.

### Q022. How would you respond when an error budget is exhausted?
**Answer:** Validate the SLI and incident impact; prioritize risk reduction, pause or tighten risky rollouts if consistent with team policy, mitigate recurring faults, and allocate engineering effort to reliability improvements. Communicate trade-offs with product owners. It is a **decision policy**, not an automatic blanket release freeze in every organization.

---

## 3. Linux and operating systems

### Q023. A Linux server has high CPU utilization. What do you check?
**Answer:** Confirm whether the issue affects users, then correlate CPU with traffic and deploys. Inspect process utilization, run queue, system/user/steal time, container quotas, and throttling. Identify the busy thread or process, inspect logs/profiles, and mitigate via rollback, scaling, or load limiting as appropriate.

```bash
uptime
top -H
ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu | head
vmstat 1 5
```

### Q024. What does Linux load average mean?
**Answer:** Load average estimates the average number of tasks runnable or waiting in uninterruptible sleep over roughly **1, 5, and 15 minutes**. It is not CPU percentage. Compare it with available CPUs and inspect I/O waits and process state before concluding CPU is overloaded.

### Q025. The server reports low free memory. Is that automatically a problem?
**Answer:** No. Linux uses memory for filesystem cache. Check `MemAvailable`, swapping, major page faults, memory pressure, and application latency. On a container, also check cgroup limits and OOM events. Do not interpret a small `free` column as proof of memory exhaustion.

```bash
free -h
cat /proc/meminfo | grep -E 'MemAvailable|SwapFree'
vmstat 1 5
journalctl -k --no-pager | grep -i 'out of memory\|oom' | tail
```

### Q026. Disk utilization is 100%. How do you investigate?
**Answer:** Distinguish disk **capacity** from disk **I/O utilization**. For capacity, inspect filesystems, inode counts, and large directories; for I/O, inspect latency, queue pressure, and processes. Clean up only after establishing ownership, retention requirements, and safe deletion options.

```bash
df -h
df -i
du -xhd1 /var 2>/dev/null | sort -h
lsblk
iostat -xz 1 3   # requires sysstat
```

### Q027. Why can disk space remain full after deleting a file?
**Answer:** A process may still hold the deleted file open, preventing storage from being reclaimed until its file descriptor closes. Find deleted-but-open files using `lsof +L1`; then arrange a safe log rotation or controlled process restart rather than indiscriminately killing processes.

### Q028. What is an inode and how can inode exhaustion occur?
**Answer:** An inode stores filesystem metadata for a file or directory. Millions of tiny files can exhaust available inodes even if bytes remain. Check `df -i`, identify directories with excessive file counts, and resolve the generating process or cleanup policy.

### Q029. How do you debug a systemd service that fails to start?
**Answer:** Inspect unit status, recent logs, dependencies, environment, permissions, and executable path. Validate configuration, recent changes, resource availability, and exit codes. Fix the underlying cause and verify a clean restart.

```bash
systemctl status my-service --no-pager
journalctl -u my-service -n 100 --no-pager
systemctl cat my-service
systemctl show my-service -p ExecStart -p Environment
```

### Q030. What is the difference between SIGTERM and SIGKILL?
**Answer:** `SIGTERM` (15) requests graceful termination and can be handled; `SIGKILL` (9) is enforced by the kernel and cannot be caught or ignored. Prefer graceful shutdown so the application can drain requests and release resources. `SIGKILL` can cause interrupted work or unflushed state.

### Q031. How do you identify which process owns a port?
**Answer:** Use `ss -lntp` for listening TCP sockets and associated processes (permissions may restrict process visibility). `lsof -i :PORT` is another option. Verify the expected process and bind address before changing a service.

```bash
ss -lntp
sudo lsof -nP -iTCP:8080 -sTCP:LISTEN
```

### Q032. What is the difference between a process and a thread?
**Answer:** A process has its own address space and resource context; threads within a process share most memory and resources but have independent execution state. A single busy thread can cause latency despite moderate process-level CPU. `top -H` or a profiler helps identify hot threads.

### Q033. What are file descriptors and what happens when they run out?
**Answer:** File descriptors identify open files, sockets, and other kernel objects. Exhausting the per-process or system-wide limit can cause errors such as `Too many open files`, preventing new connections or file opens. Inspect limits and application descriptor usage; investigate leaks before merely raising limits.

```bash
ulimit -n
cat /proc/$(pgrep -o my-service)/limits   # replace process name
sudo lsof -p <PID> | wc -l
```

### Q034. What does `set -euo pipefail` do in a shell script?
**Answer:** `-e` usually stops execution for unhandled nonzero commands, subject to shell-context exceptions; `-u` treats unset variables as errors; `pipefail` causes pipelines to report the rightmost failing command instead of only the final command. It improves safety but requires deliberate handling of expected failures.

```bash
#!/usr/bin/env bash
set -euo pipefail
```

---

## 4. Networking and HTTP

### Q035. Describe what happens when a user opens an HTTPS URL.
**Answer:** The client resolves DNS, connects to the destination (often TCP; HTTP/3 may use QUIC/UDP), performs TLS negotiation, sends an HTTP request, and receives a response. Proxies, CDNs, load balancers, caches, authentication, and backend services may add hops. Troubleshoot each layer separately.

### Q036. How do you troubleshoot an application that cannot resolve a hostname?
**Answer:** Verify the name, resolver configuration, DNS reachability, and whether the failure is local or widespread. Compare application DNS behavior with tools such as `dig`/`getent`. In Kubernetes, also inspect CoreDNS and service search domains; distinguish NXDOMAIN from timeouts.

```bash
getent hosts api.example.com
dig api.example.com
dig +short api.example.com
cat /etc/resolv.conf
```

### Q037. What is the difference between TCP and UDP?
**Answer:** TCP offers ordered, reliable byte-stream delivery with connection setup and congestion control. UDP is message-oriented and does not provide reliability/order by itself. Applications can add these features on top of UDP—QUIC is an example—so it is not correct to say all UDP-based application protocols are unreliable.

### Q038. Explain HTTP 502, 503, and 504.
**Answer:** `502 Bad Gateway`: a gateway/proxy got an invalid upstream response. `503 Service Unavailable`: the server is temporarily unable to handle the request, often due to overload or maintenance. `504 Gateway Timeout`: the gateway/proxy did not receive a timely upstream response. Codes provide clues, not definitive root causes.

### Q039. What are TCP connection timeouts versus read timeouts?
**Answer:** A **connect timeout** means a connection could not be established in time; a **read timeout** means the connection succeeded but a response or data was not received in time. Check DNS, firewall, SYN handling, TLS, server queues, downstream dependency latency, and client timeout settings.

### Q040. How do you debug an endpoint that is slow or unreachable?
**Answer:** Scope the impact, compare client regions, and test DNS, TCP/TLS, HTTP status and timing. Check load balancer health, app readiness, downstream calls, and saturation. Collect timestamps and traces instead of relying on ping alone (ICMP may be blocked).

```bash
curl -v --connect-timeout 3 --max-time 10 https://api.example.com/health
curl -sS -o /dev/null -w 'dns=%{time_namelookup} connect=%{time_connect} tls=%{time_appconnect} ttfb=%{time_starttransfer} total=%{time_total}\n' https://api.example.com/health
```

### Q041. What is the difference between a load balancer and reverse proxy?
**Answer:** Both can sit in front of applications. A load balancer primarily distributes traffic across backends; a reverse proxy accepts client requests and forwards them, often adding TLS termination, caching, routing, auth, or header handling. Real products frequently perform both roles.

### Q042. What is connection pooling and why can it fail?
**Answer:** Pools reuse database or upstream connections to reduce setup overhead. Under high load, exhausted pools can increase wait time and trigger timeouts. Check active/idle counts, pool wait, server connection limits, leaked connections, and downstream latency; increasing pool size without understanding bottlenecks can worsen overload.

### Q043. What is DNS TTL and why does it matter during failover?
**Answer:** A DNS record's TTL guides how long compliant resolvers may cache its result. A shorter TTL can shorten some failover delays, but clients, resolvers, connection reuse, and network caches can still prolong cutover. Test failover rather than assuming the TTL guarantees recovery time.

### Q044. What are TLS certificates and common causes of TLS failure?
**Answer:** Certificates bind public keys to identities under a trust chain. Common failures include expiry, hostname mismatch, untrusted CA, incomplete chain, incompatible protocols/ciphers, and clock skew. Confirm server identity and chain rather than disabling certificate verification.

```bash
openssl s_client -connect api.example.com:443 -servername api.example.com </dev/null
```

---

## 5. Kubernetes and containers

### Q045. What are the core Kubernetes control-plane components?
**Answer:** **API server** exposes the cluster API; **etcd** stores cluster state; **scheduler** places unscheduled Pods; **controller manager** runs reconciliation loops. Nodes run **kubelet**, a container runtime, and networking components such as a CNI plugin. Understand the distinction between desired and observed state.

### Q046. What is the difference between a Pod, Deployment, StatefulSet, and DaemonSet?
**Answer:** A **Pod** is the smallest schedulable unit; a **Deployment** manages stateless replica updates via ReplicaSets; a **StatefulSet** provides stable identities and ordered operations for workloads needing them; a **DaemonSet** runs Pods on eligible nodes, typically for agents. Persistent storage is configured separately and depends on the workload.

### Q047. What is the difference between readiness, liveness, and startup probes?
**Answer:** **Readiness** controls whether the Pod is eligible to receive traffic through matching Services. **Liveness** can restart a container that becomes unhealthy. **Startup** allows slow-starting containers extra initialization time and suppresses liveness/readiness probes until it succeeds. Misconfigured probes can turn a slowdown into a restart storm.

### Q048. A Pod is in `CrashLoopBackOff`. How do you debug it?
**Answer:** Inspect container exit reason, current and previous logs, startup command, probe failures, configuration, secrets, volume mounts, OOM events, and recent rollouts. `CrashLoopBackOff` is a backoff state indicating repeated failures, not the root cause.

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
```

### Q049. A Pod is stuck in `Pending`. What do you check?
**Answer:** Read the scheduling events. Common causes: insufficient requested CPU/memory, taints without tolerations, node selectors or affinity that match no nodes, quota limits, unbound PVCs, or topology constraints. Differentiate unschedulable Pods from image pull or container-start failures.

### Q050. What causes `ImagePullBackOff`?
**Answer:** Kubernetes cannot pull the image and backs off retries. Possible causes: invalid registry/tag, missing image, registry auth or `imagePullSecret`, rate limits, DNS/network failures, and certificate problems. Start with Pod events and then test registry access from the right context.

### Q051. What is `OOMKilled` in Kubernetes?
**Answer:** The container was killed for exceeding its memory limit or due to memory pressure. Inspect the previous container state, memory working set, limits, growth trends, and application behavior. Fix leaks or appropriately tune limits/requests; merely increasing limits may defer the next incident.

### Q052. Explain resource requests and limits.
**Answer:** **Requests** influence scheduling and resource guarantees; **limits** constrain usage where the resource supports enforcement. CPU limits can cause throttling. Memory limits can lead to OOM kills. Determine values from measured workload behavior and capacity headroom.

### Q053. Explain Kubernetes HPA and its limitations.
**Answer:** The **Horizontal Pod Autoscaler** adjusts replica counts using configured metrics and a controller loop. It needs appropriate metrics and available node capacity, may lag traffic spikes, and cannot fix slow dependencies or an incorrectly sized database. Pair it with capacity planning and cluster autoscaling where relevant.

### Q054. What is a Kubernetes Service and what are common types?
**Answer:** A **Service** provides stable discovery and routing to selected backend Pods. Common types include `ClusterIP`, `NodePort`, and `LoadBalancer`. **Ingress** or Gateway API resources can provide HTTP routing through a suitable controller. Services select endpoints; they do not automatically make unhealthy applications healthy.

### Q055. How do ConfigMaps and Secrets differ?
**Answer:** Both provide configuration data. **Secrets** are intended for sensitive values but should not be assumed encrypted or access-controlled by default in every cluster; configure RBAC, encryption at rest, rotation, and external secret-management integration as appropriate. Never commit real secrets to Git.

### Q056. What is a PersistentVolume (PV) and PersistentVolumeClaim (PVC)?
**Answer:** A **PV** represents storage available to the cluster; a **PVC** is a workload's request for storage. A StorageClass may dynamically provision volumes. Storage binding, access modes, zone topology, and reclaim policy can affect scheduling and recovery.

### Q057. What is a PodDisruptionBudget (PDB)?
**Answer:** A PDB controls how many matching Pods may become unavailable during certain **voluntary disruptions** (for example, eviction during node drain). It does not prevent node crashes or guarantee availability. Misconfigured PDBs can delay maintenance and drains.

### Q058. How do rolling updates, blue-green, and canary deployments compare?
**Answer:** **Rolling** replaces instances gradually; **blue-green** switches traffic between separate environments; **canary** exposes a small portion of traffic to the new version before expansion. Choose based on risk, traffic control, rollback speed, data compatibility, and observability.

### Q059. How do you troubleshoot a failed rollout?
**Answer:** Inspect deployment rollout status, ReplicaSets, Pod events and images, readiness failures, resource constraints, and last successful revision. Correlate failures with metrics and traces. Pause/roll back only when safe for schema and state changes; verify after mitigation.

```bash
kubectl rollout status deploy/my-app -n <namespace>
kubectl describe deploy/my-app -n <namespace>
kubectl get rs,pods -n <namespace> -o wide
kubectl rollout history deploy/my-app -n <namespace>
# Authorized mitigation if rollback is safe:
kubectl rollout undo deploy/my-app -n <namespace>
```

### Q060. What is a NetworkPolicy and what does it depend on?
**Answer:** NetworkPolicy defines allowed Pod ingress/egress flows when enforced by a compatible network plugin. A policy object alone does not guarantee isolation; the CNI must support enforcement. Test DNS, egress dependencies, and namespace selectors carefully before restricting traffic.

### Q061. How would you diagnose a Kubernetes Service returning no traffic?
**Answer:** Confirm DNS name/namespace, Service selector, EndpointSlices, target port mapping, Pod readiness, network policies, ingress controller routing, and underlying application listener. A Service with no ready endpoints often points to selectors or Pod readiness, not a load-balancer defect.

```bash
kubectl get svc -n <namespace>
kubectl get endpointslice -n <namespace> -l kubernetes.io/service-name=<service>
kubectl describe svc <service> -n <namespace>
```

---
## 6. Monitoring, logging, and tracing

### Q062. What is the difference between monitoring and observability?
**Answer:** **Monitoring** detects known conditions using measurements, dashboards, and alerts. **Observability** is the ability to understand internal system behavior from external signals, including unfamiliar failure modes. Metrics, logs, traces, profiles, and context-rich telemetry work together. Instrumentation quality matters as much as the monitoring tool.

### Q063. How do Prometheus, Grafana, and Alertmanager work together?
**Answer:** **Prometheus** scrapes or receives supported metric data, stores time series, evaluates queries and alert rules. **Grafana** queries data sources to visualize metrics and related signals. **Alertmanager** groups, deduplicates, routes, silences, and inhibits alerts from Prometheus-compatible alerting rules. Grafana also offers its own alerting system; choose an intentional routing design to prevent duplicates.

### Q064. What are Prometheus counters, gauges, histograms, and summaries?
**Answer:** A **counter** is cumulative and normally increases until restart/reset; a **gauge** rises and falls; a **histogram** records bucket counts plus sum/count for observations; a **summary** calculates quantiles client-side (depending on configuration). Histograms are generally suitable for cross-instance aggregation using correctly constructed queries; precomputed summary quantiles cannot generally be aggregated safely.

### Q065. What does `rate()` do in PromQL?
**Answer:** `rate(counter[5m])` estimates the per-second average increase of a counter over five minutes, with counter-reset handling. Use it for request throughput and errors. Aggregate **after** applying `rate()` to each time series; avoid directly applying it to gauges.

```promql
sum by (service) (rate(http_requests_total[5m]))
```

### Q066. How would you calculate an HTTP 5xx error rate in Prometheus?
**Answer:** Divide the rate of failed eligible requests by all eligible requests for the same service and window. Match labels and define failure semantics; this example assumes `http_requests_total{status=...}` and excludes no traffic categories.

```promql
sum(rate(http_requests_total{job="api",status=~"5.."}[5m]))
/
sum(rate(http_requests_total{job="api"}[5m]))
```

**Caution:** A zero-traffic denominator needs an explicit policy; expression behavior and alerting for low-traffic services require care.

### Q067. How would you calculate p95 request latency from a Prometheus histogram?
**Answer:** Use `histogram_quantile` over the per-second **bucket rates** (for classic histograms) and preserve the `le` bucket label. The result is an estimate based on bucket resolution.

```promql
histogram_quantile(
  0.95,
  sum by (le, service) (rate(http_request_duration_seconds_bucket[5m]))
)
```

**Note:** Prometheus also supports native histograms, whose query shape differs. Verify the application’s metric names and histogram type.

### Q068. What are labels, and why is high cardinality dangerous?
**Answer:** Labels create distinct metric series by attribute. Labels with unbounded values (user ID, request ID, raw URL, timestamp) can explode active series count, memory, storage, and query cost. Prefer bounded labels such as route templates and HTTP method; put request-level detail in logs/traces.

### Q069. What is a Prometheus exporter?
**Answer:** An exporter translates metrics from an application or system into an endpoint Prometheus can scrape. Examples include node exporter for Linux host metrics and blackbox exporter for probe results. Exporters are instrumentation integrations, not substitutes for service-level SLIs.

### Q070. How do you make alerts actionable?
**Answer:** Page for meaningful, urgent user impact or imminent risk. Define an owner, severity, threshold, duration, runbook, dashboard, and mitigation. Route warnings to tickets or lower-urgency channels. Remove duplicate/noisy alerts and review page volume against actual incidents.

### Q071. Explain alert grouping, silencing, and inhibition.
**Answer:** **Grouping** bundles related alerts into fewer notifications. **Silencing** temporarily suppresses notifications matching a condition (for example, during approved maintenance). **Inhibition** suppresses lower-priority alerts when a related higher-level alert is active. These mechanisms reduce noise but must not mask real, independent incidents.

### Q072. What is the difference between logs, metrics, and traces?
**Answer:** **Metrics** summarize numeric trends cheaply; **logs** record discrete events and details; **traces** follow work across service boundaries using spans. For high latency: metrics detect p99 regression, traces reveal the slow dependency, and logs explain a specific failure. Correlation IDs tie evidence together.

### Q073. What does OpenTelemetry provide?
**Answer:** OpenTelemetry provides vendor-neutral APIs, SDKs, instrumentation, and collectors for generating, processing, and exporting telemetry such as traces, metrics, and logs. It is not itself a dashboard or long-term storage database. Common backends include Jaeger/Tempo for traces and Prometheus-compatible storage for metrics.

### Q074. What is distributed tracing and a span?
**Answer:** A trace represents the journey of a request through components. A **span** represents a unit of work with timing, attributes, status, and relationships to other spans. It helps pinpoint latency in fan-out calls, queue waits, and dependency failures. Propagate trace context consistently across services.

### Q075. What is synthetic monitoring versus real-user monitoring (RUM)?
**Answer:** **Synthetic monitoring** runs scheduled probes or scripted journeys from controlled locations, detecting failures even when no real customer uses the service. **RUM** observes actual client experience, including geography, browser, device, and network differences. Use both where customer-facing reliability is critical.

### Q076. What is a recording rule and why use one?
**Answer:** A Prometheus recording rule precomputes a query into a new time series at evaluation intervals. It can reduce expensive repeated calculations and standardize shared service metrics. It also introduces extra stored series and requires correct naming, evaluation, and aggregation semantics.

### Q077. Why might Prometheus show `up == 1` while the service is broken?
**Answer:** `up` indicates whether Prometheus successfully scraped the target, not whether real customer workflows work. The exporter may respond while downstream requests fail. Add user-facing SLIs, endpoint probes, dependency-aware metrics, and correctness checks.

---

## 7. Incident management and on-call

### Q078. Walk through your process when a production alert fires.
**Answer:** (1) **Acknowledge** and assess customer impact and severity. (2) **Stabilize** using a safe reversible mitigation (traffic shift, rollback, scale, feature disable). (3) **Investigate** via metrics, logs, traces, changes, and dependency status. (4) **Communicate** updates and ownership. (5) **Verify recovery** against SLIs. (6) Record a timeline, root-cause investigation, and follow-up actions.

### Q079. What is the difference between an incident and a problem?
**Answer:** An **incident** is an unplanned degradation or interruption requiring response. A **problem** is the underlying cause or class of recurring incidents that needs deeper investigation and permanent remediation. Incident mitigation often precedes root-cause analysis.

### Q080. What are severity levels (SEV1, SEV2, SEV3)?
**Answer:** Severity ranks impact and urgency. **Example only:** SEV1 = critical widespread customer outage; SEV2 = major partial degradation; SEV3 = smaller/localized impact. Exact mappings, response-time expectations, escalation paths, and customer communications are organization-specific and must be documented.

### Q081. Explain MTTD, MTTA, MTTR, and MTBF.
**Answer:** **MTTD**: mean time to detect; **MTTA**: mean time to acknowledge; **MTTR**: mean time to restore/repair (define which); **MTBF**: mean time between failures for applicable systems. These aggregates can hide wide distributions, so track incident severity and percentiles too.

### Q082. When should you roll back versus continue debugging?
**Answer:** Prefer fast, reversible mitigation when a recent change correlates strongly with the impact and rollback is safe. Continue debugging if rollback would corrupt data or worsen compatibility, or if the change is not responsible. Assess customer harm, blast radius, dependency state, and schema compatibility before acting.

### Q083. What is the difference between a runbook and a playbook?
**Answer:** A **runbook** contains repeatable operational procedures (checks, commands, expected outcomes, rollback). A **playbook** provides decision-making guidance for a broader incident class (roles, prioritization, escalation, communications). Terms overlap across organizations; clear ownership and tested procedures matter more than the label.

### Q084. What is a blameless postmortem?
**Answer:** A structured review that examines contributing technical and organizational conditions without framing individual mistakes as the sole cause. Capture impact, timeline, detection, mitigation, contributing factors, what worked, and corrective actions with owners and due dates. It should improve the system and learning culture.

### Q085. What information belongs in a stakeholder incident update?
**Answer:** The affected service and user impact, start time/time zone, current status, mitigation underway, next update cadence, and known workarounds. Clearly distinguish **verified facts** from hypotheses. Avoid publishing secrets, speculation, or unapproved customer details.

### Q086. How do you prevent on-call burnout?
**Answer:** Reduce nonactionable pages, share sustainable rotations, automate repetitive remediation, document runbooks, establish backup escalation, and allocate time to remove root causes. Review on-call load and out-of-hours interruptions. Reliability work should decrease operator burden, not merely distribute it.

---

## 8. AWS, cloud, and disaster recovery

### Q087. What is the AWS shared responsibility model?
**Answer:** AWS generally secures the **underlying cloud infrastructure**, while customers are responsible for items such as their data, identities, workload configurations, application security, and many network controls. Responsibility details depend on whether you use EC2, a managed service, or a SaaS offering.

### Q088. How does CloudWatch compare with Prometheus?
**Answer:** **CloudWatch** is AWS's managed platform for metrics, logs, alarms, and related telemetry. **Prometheus** is an open-source metrics and alerting system with PromQL; it often uses exporters and a separate visualization stack. You can integrate them or use AWS-managed Prometheus-compatible services depending on scale, operating model, and cost.

### Q089. How would you design a highly available AWS web service?
**Answer:** Deploy stateless app instances across multiple **Availability Zones**, front them with an appropriate load balancer, automate instance health/replacement, make the data layer resilient, and monitor user-facing SLIs. Eliminate single-zone dependencies, test failure scenarios, and use backup/DR policies suited to business requirements.

### Q090. What is the difference between an ALB and NLB?
**Answer:** An **Application Load Balancer (ALB)** provides layer-7 HTTP/HTTPS capabilities such as host/path-based routing. A **Network Load Balancer (NLB)** handles layer-4 TCP/UDP/TLS traffic with different performance and networking characteristics. Selection depends on protocol, routing, IP/client-preservation needs, and application behavior.

### Q091. How does Auto Scaling help reliability?
**Answer:** Auto Scaling adjusts compute capacity to changing demand or health conditions. It helps absorb traffic and replace unhealthy instances but is limited by scaling lag, quotas, availability of capacity, and downstream bottlenecks. Load-test and set sensible minimum capacity and alarms.

### Q092. What are RPO and RTO?
**Answer:** **Recovery Point Objective (RPO)** is the maximum acceptable data loss measured in time; **Recovery Time Objective (RTO)** is the target time to restore service after disruption. A database with a 15-minute backup interval may still fail its RPO due to incomplete backup/restore design. Validate both in recovery drills.

### Q093. Compare backup/restore, pilot light, warm standby, and active-active DR.
**Answer:** **Backup/restore** is lower running cost but often slower recovery. **Pilot light** keeps critical foundations running. **Warm standby** keeps a scaled-down working environment. **Active-active** serves traffic from multiple locations and can reduce recovery time but adds complexity, state-consistency, and cost challenges. Choose based on proven RTO/RPO needs.

### Q094. Your AWS region becomes unavailable. What is the response?
**Answer:** Confirm customer impact and cloud health signals, activate the documented DR decision process, assess data freshness and consistency, and shift traffic only when the recovery environment is verified. Monitor load, error rate, DNS propagation, and dependencies. Run a post-incident review and test region failure regularly.

### Q095. What is IAM least privilege and why is it important for SRE?
**Answer:** Give identities only the actions and resources required for their duties. Prefer short-lived credentials, scoped roles, MFA where appropriate, auditing, and controlled emergency access. Broad permissions can turn operational incidents into security incidents.

### Q096. What should you monitor for EKS workloads?
**Answer:** Monitor **user SLIs** alongside Pod/node health, CPU and memory requests/limits, throttling/OOM, node pressure, HPA behavior, network/DNS, ingress error rates, deployment events, cluster component health, and cloud resource quotas. EKS control-plane management does not replace application reliability ownership.

### Q097. How do you identify cloud infrastructure capacity constraints?
**Answer:** Look for quotas, instance/volume limits, IP address exhaustion, load balancer targets, autoscaling events, resource requests, and underlying capacity. Tie infrastructure signals to request failure/latency and create headroom for failures and deployment surges.

---

## 9. CI/CD, Terraform, and GitOps

### Q098. How does CI/CD improve reliability?
**Answer:** CI validates changes consistently through builds, tests, security checks, and artifact provenance. CD deploys through repeatable, controlled steps. Reliability improves when pipelines include safe rollout strategies, environment parity, observability gates, approvals as needed, and rollback plans—not simply faster deployment.

### Q099. What are useful deployment quality gates?
**Answer:** Unit/integration tests, vulnerability-policy checks, artifact integrity, configuration validation, smoke tests, readiness checks, canary analysis against SLO metrics, and change authorization for high-risk services. Quality gates must be fast, meaningful, and designed to fail safely rather than block work arbitrarily.

### Q100. What is infrastructure as code (IaC)?
**Answer:** Defining infrastructure in reviewable, version-controlled code instead of ad hoc console operations. Benefits include repeatability, change review, drift detection, and disaster recovery. Risks include secrets exposure, accidentally destructive plans, and poorly controlled state or permissions.

### Q101. What is Terraform state and why must it be protected?
**Answer:** State maps Terraform-managed resources to real infrastructure and stores metadata needed for planning changes. It may contain sensitive values. Use restricted access, encryption, backups, remote state where appropriate, and locking or backend-specific concurrency controls. Never assume the state is safe to publish because the `.tf` files contain no passwords.

### Q102. Explain `terraform plan`, `apply`, `refresh`, and drift.
**Answer:** `plan` previews the intended changes based on configuration, state, and provider observations; `apply` performs approved changes. **Drift** occurs when real infrastructure changes outside Terraform. Refresh updates observed state (normally part of planning); investigate out-of-band changes before blindly applying plans.

### Q103. What is GitOps?
**Answer:** GitOps uses version-controlled desired-state definitions and automated reconciliation agents to converge deployed environments toward that state. Tools such as **Argo CD** or **Flux** are common in Kubernetes. Git becomes an auditable source of intended configuration, but secret handling and drift policies still require design.

### Q104. How do you perform a safe rollback of a release?
**Answer:** Use immutable, versioned artifacts and a rehearsed rollback mechanism. Check database migration compatibility, feature flags, contract changes, queued work, and data shape. After rollback, verify availability/latency/error SLIs rather than assuming a successful deploy command means customer recovery.

### Q105. Blue-green versus canary: when would you choose each?
**Answer:** Use **blue-green** for quick environment-level switchovers when duplicated capacity is affordable. Use **canary** to progressively expose real traffic and compare behavior with baseline. Canary requires trustworthy health metrics and traffic routing; blue-green still needs data compatibility and careful switchovers.

### Q106. What do you do when a Jenkins pipeline suddenly starts failing?
**Answer:** Identify the **first failing stage** and distinguish app changes, pipeline code, credentials, agents, tools, network, artifact repositories, and external dependencies. Compare with the last green run, examine sanitized logs, reproduce in a safe workspace, fix the cause, and add a regression check. Avoid suppressing failures merely to turn the build green.

### Q107. Why use artifact repositories and immutable versioning?
**Answer:** Artifact repositories such as Nexus store build outputs for traceability, reuse, controlled promotion, and rollback. Tag artifacts immutably by digest, commit, build ID, or release identifier; record build inputs and provenance. Avoid silently overwriting a release artifact with different bytes.

### Q108. What is a deployment freeze and when is it warranted?
**Answer:** A temporary restriction on risky changes, often during severe incidents, high-risk business windows, or exhausted error budgets under team policy. Use narrowly scoped exceptions and a clear exit criterion. A freeze is a risk-management measure, not a substitute for safe deployment engineering.

---

## 10. Databases, storage, and distributed systems

### Q109. What happens when database connections are exhausted?
**Answer:** New requests may queue, timeout, or fail because clients cannot obtain a connection. Examine pool usage, long-running queries, leaked transactions, server limits, and dependency latency. Mitigate by reducing contention, fixing leaks, throttling demand, or resizing pools within database capacity.

### Q110. How do you investigate a slow database query?
**Answer:** Find the affected query fingerprint and execution plan, inspect indexes, row estimates, locks, I/O, CPU, and recent schema or traffic changes. Review application timeout and retry behavior. Make changes with safe tests and compare before/after latency and load.

### Q111. What is replication lag and why does it matter?
**Answer:** Replication lag is the delay between writes on a source and their visibility on replicas. It can cause stale reads, failover data loss risk, and incorrect application behavior. Monitor lag in relevant units and test consistency assumptions; replication is not a backup.

### Q112. Explain CAP theorem at an interview level.
**Answer:** In the presence of a **network partition**, a distributed system must make trade-offs between availability and strong consistency for particular operations. It does **not** mean a system can arbitrarily pick only two properties in all situations. Real systems may make different trade-offs for different operations.

### Q113. What are retries, timeouts, and exponential backoff?
**Answer:** Timeouts bound how long a caller waits. Retries handle transient failures, ideally with bounded attempts, exponential backoff, and jitter to avoid synchronized retry storms. Retrying non-idempotent operations can duplicate side effects; use idempotency keys, budgets, or explicit deduplication where appropriate.

### Q114. What is a circuit breaker?
**Answer:** A control that temporarily stops sending calls to a failing dependency after predefined failure conditions, then probes recovery. It reduces cascading failures but can also reject healthy requests if mis-tuned; combine it with timeouts, bulkheads, and sensible fallback behavior.

### Q115. What is a thundering herd problem?
**Answer:** Many clients or workers act simultaneously after an event, overwhelming a shared dependency—for example, cache expiry causing all clients to query the database. Mitigations include TTL jitter, request coalescing, rate limits, caching, and gradual recovery.

### Q116. Why is a cache not a complete reliability strategy?
**Answer:** Caches improve latency and reduce backend load but introduce staleness, invalidation complexity, cold starts, stampedes, and consistency trade-offs. Design safe cache fallbacks, realistic TTLs, observability, and capacity for cache failures.

---

## 11. Security, capacity, and automation

### Q117. What is defense in depth?
**Answer:** Multiple independent layers of protection—identity controls, least privilege, network policies, hardened images, secret rotation, admission checks, logging, and incident response. A single misconfiguration should not automatically compromise the whole system.

### Q118. How should secrets be managed in CI/CD pipelines?
**Answer:** Use a secret manager or tightly controlled CI credentials store, short-lived tokens where possible, least-privilege roles, masked log output, approved rotation, and controlled injection at runtime. Do not hardcode secrets in Jenkinsfiles, images, Terraform state exposed to unauthorized users, or Git repositories.

### Q119. What is capacity planning?
**Answer:** Forecasting infrastructure needs based on traffic growth, resource demand, seasonal patterns, dependency limits, and reliability headroom. Use load tests, capacity models, utilization/queue measurements, and quota review. Account for failure scenarios (for example, remaining zones handling traffic after one AZ fails).

### Q120. What is a saturation signal?
**Answer:** A sign that a resource is nearing or exceeding a limit—queue depth, connection pool wait, CPU throttling, disk latency, or exhausted worker threads. Utilization alone can mislead: a service can be saturated due to one serial bottleneck before total CPU usage looks high.

### Q121. How does a health check differ from an SLI?
**Answer:** A health check is a narrow probe answering whether some functionality works now; an SLI quantifies a defined service property across requests/events/time. A healthy `/health` endpoint can coexist with broken login, checkout, or data consistency. Instrument the customer journey.

### Q122. What safeguards should self-healing automation have?
**Answer:** Scope checks, idempotency, rate limits, change authorization where needed, rollback, audit logs, alerts on automation actions, and verification of recovery. Auto-restarts and node replacements can amplify damage if their trigger is wrong or the dependency is the real cause.

### Q123. How can Python help with SRE operations?
**Answer:** Python can automate API-based checks, inspect logs, collect metrics, validate configurations, generate incident evidence, and enforce release policies. Good automation includes timeouts, retries with backoff, structured logs, secure credentials, tests, and a dry-run mode for risky changes.

### Q124. What is an idempotent operation?
**Answer:** Repeating the same operation has the same intended end state as performing it once. For example, ensuring a target directory exists can be idempotent; blindly appending the same line to a configuration file is not. Idempotency is particularly valuable when jobs retry after partial failure.

### Q125. How do you avoid retry storms in automation?
**Answer:** Apply a timeout budget, bounded retries, exponential backoff with jitter, concurrency controls, and safe handling for rate-limit responses (such as `Retry-After` when applicable). Export metrics for retry attempts and failed operations. Do not retry authentication failures or non-idempotent writes indiscriminately.

### Q126. How do you prioritize reliability versus feature development?
**Answer:** Use SLO performance, error budget, severity of recent incidents, risk of planned changes, and customer/business impact. Express proposed reliability work in measurable outcomes (such as a reduction in pages or failed requests) and make trade-offs explicitly with stakeholders.

---
## 12. Real-world production scenarios

> **Practice answering in this order:** impact → checks → immediate mitigation → verification → prevention. Avoid saying you would restart everything before identifying the failure mode.

### Scenario 1 — API latency jumps from 200 ms to 3 seconds
**Question:** Users report slow checkout requests. What do you do?

**Strong answer:**
1. Check latency p50/p95/p99, error rate, traffic, saturation, and regions; establish whether customers are affected.
2. Compare before/after deployment and dependency changes; inspect traces to identify the slowest span.
3. Check DB query/connection pool time, external API, CPU throttling, queuing, and resource saturation.
4. Mitigate with a verified rollback, traffic shift, rate limit, or scaling **only if** appropriate to the bottleneck.
5. Verify checkout SLI recovery, communicate, and add a regression test/alert or performance budget.

**Follow-up:** What if CPU is normal? Investigate I/O, locks, DNS, connection pooling, serial dependencies, and queue wait.

### Scenario 2 — Error rate rises immediately after a deployment
**Question:** A new release causes 10% HTTP 5xx responses. How do you respond?

**Strong answer:** Declare appropriate severity, identify affected users and versions, stop rollout, compare canary vs baseline, and roll back the bad version if the data/schema is compatible. Check health probes, config, feature flags, and dependencies. Verify error and latency SLIs after rollback; document the release failure and improve deployment gates.

**Follow-up:** A rollback does not fix it—what next? Check schema changes, shared infrastructure, config rollout, stale connections, and upstream dependency failures.

### Scenario 3 — Kubernetes Pod restarts every 30 seconds
**Question:** Pod status is `CrashLoopBackOff`.

**Strong answer:** Inspect `kubectl describe pod`, container exit status, `kubectl logs --previous`, rollout diff, and OOM metrics. Determine whether it is a process crash, invalid config, missing Secret, failed liveness probe, or memory limit. Correct the cause or safely revert the change. Track restart rate and user impact.

### Scenario 4 — Pods are healthy but the Service returns 503
**Question:** Kubernetes app Pods are Running/Ready but clients get 503.

**Strong answer:** Confirm the error's source (Ingress controller, API gateway, or application); inspect Service selectors and EndpointSlices, target port, gateway routes, network policies, and TLS. Test from inside the cluster. Check whether the upstream is overloaded despite readiness passing. A Pod in `Running` does not prove the full request path works.

### Scenario 5 — Node disk fills up overnight
**Question:** The node suddenly reaches 100% filesystem utilization.

**Strong answer:** Identify filesystem and inode pressure via `df -h`, `df -i`; locate growth by directories and timestamps. Look for container logs, image layers, temp files, and deleted-but-open files. Apply **approved** retention/log rotation and, if appropriate, safely drain/recover the node. Prevent recurrence with quotas, log retention, and threshold-based capacity alerts.

### Scenario 6 — A region or availability zone fails
**Question:** Traffic is failing in one AWS AZ.

**Strong answer:** Confirm scope using service SLIs and AWS resource health; check load balancer target health and capacity in remaining AZs. Execute the tested failover/runbook, validate database quorum/replication and traffic routing, ensure the surviving zones can handle load, communicate progress, and review failure-domain assumptions.

### Scenario 7 — Prometheus stopped scraping a critical target
**Question:** `up{job="api"}` is zero or absent.

**Strong answer:** Check target discovery, scrape errors, networking, exporter process, TLS/credentials, port/path, and relabel configuration. Distinguish **target down** from **series missing**. Avoid assuming customer impact until service-level SLIs or synthetic probes confirm it; monitoring outages also deserve appropriate alerts.

### Scenario 8 — Alert fatigue: 300 alerts in an hour
**Question:** How would you fix noisy paging?

**Strong answer:** Group correlated alerts, remove duplicate pages, identify alert rules unrelated to user impact, improve thresholds and `for` windows, use inhibition and maintenance silences, and add ownership/runbooks. Adopt SLO burn-rate alerts for customer-impacting signals where appropriate. Monitor pages per shift and missed true incidents.

### Scenario 9 — Database connections spike and requests time out
**Question:** Application connections reach the database's maximum.

**Strong answer:** Look for pool saturation, leaked sessions, slow queries, lock waits, recent traffic spikes, and deploy changes. Contain load, fix/revert bad connection behavior, or increase capacity when safe. Avoid increasing all pool sizes blindly; it can overload the database further.

### Scenario 10 — Jenkins pipeline succeeds, but production breaks
**Question:** CI shows green while customers see errors after release.

**Strong answer:** Treat pipeline success as evidence of build/deploy execution, not proof of user-facing health. Check deploy metadata, feature config, routes, certificates, database migrations, and actual service SLIs. Use smoke tests and canary health gates; make deployment completion conditional on measured service behavior and a safe rollback path.

### Scenario 11 — Sudden traffic spike causes cascading failures
**Question:** Incoming traffic triples and downstream services collapse.

**Strong answer:** Confirm legitimate traffic and identify the shared bottleneck. Apply rate limiting/load shedding, circuit breakers, timeout budgets, and bounded retries with jitter. Scale only where there is capacity and no deeper constraint. Prioritize critical user journeys, monitor recovery, and load-test the new limit.

### Scenario 12 — A production Terraform plan wants to delete critical resources
**Question:** What do you do before applying?

**Strong answer:** **Stop**. Inspect the plan and recent configuration/state changes, resource addresses, provider behavior, drift, and whether resources were renamed or moved. Confirm the right workspace/account and assess dependencies. Use state migration / `moved` blocks or imports when appropriate and approved. Never apply a destructive plan merely to reconcile state quickly.

### Scenario 13 — The p99 latency is high but average latency is normal
**Question:** Why and how do you diagnose it?

**Strong answer:** A small population of severely slow requests can leave averages relatively stable. Segment by route, tenant, region, and instance; inspect queueing, slow queries, retries, garbage collection, network tails, and dependency spans. Use suitable histogram buckets and traffic-weighted aggregation.

### Scenario 14 — A service passes health checks but login is broken
**Question:** What does that tell you?

**Strong answer:** The health endpoint only verifies part of the system. Inspect authentication provider, session store, identity permissions, client errors, certificates, and login-specific telemetry. Add a journey-level SLI or safe synthetic login probe that reflects user value without exposing credentials or personal data.

### Scenario 15 — Recovery automation causes repeated restarts
**Question:** Your self-healing bot restarts a service every minute, making outages worse.

**Strong answer:** Disable or rate-limit the automation through the approved control path, stabilize the service, and find the triggering signal. Add cooldowns, retry limits, concurrency caps, independent verification, escalation, and auditing. Make the runbook explain when **not** to auto-remediate.

### Scenario 16 — An on-call alert occurs during a planned deployment
**Question:** How do you decide whether the alert is real or expected?

**Strong answer:** Check whether the signal maps to user impact and whether the change is within the approved rollout window. Compare pre-change and canary metrics, correlate with deployment events, and verify the suppression policy. Do not silence a legitimate customer-facing page simply because a deployment is ongoing.

---

## 13. PromQL and command cheat sheets

### PromQL quick reference

> Metric names and labels vary by application and exporter. Replace these samples with your environment's actual names. These examples primarily assume **classic histograms** and standard counters.

| What you want | PromQL expression |
|---|---|
| Target scrape health | `up{job="api"}` |
| Requests per second | `sum(rate(http_requests_total[5m]))` |
| RPS by service | `sum by(service) (rate(http_requests_total[5m]))` |
| 5xx rate | `sum(rate(http_requests_total{status=~"5.."}[5m]))` |
| 5xx fraction | `sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))` |
| Estimated p95 latency | `histogram_quantile(0.95, sum by(le) (rate(http_request_duration_seconds_bucket[5m])))` |
| Host CPU utilization fraction (one example) | `1 - avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m]))` |
| Host available memory fraction | `node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes` |
| Filesystem available fraction | `node_filesystem_avail_bytes / node_filesystem_size_bytes` |
| Restart increase (kube-state-metrics) | `sum by(namespace,pod) (increase(kube_pod_container_status_restarts_total[1h]))` |

**Avoid common mistakes:** do not calculate global latency p95 as an average of each instance's p95; do not apply `rate()` to ordinary gauges; do not sum over histogram buckets without retaining `le`; do not alert on low-traffic ratios without a minimum request-volume or missing-data policy.

### Linux commands

```bash
# CPU, memory, load, and pressure
uptime; top; free -h; vmstat 1 5
cat /proc/pressure/cpu; cat /proc/pressure/memory

# Storage and filesystem
lsblk; df -h; df -i; du -xhd1 /var 2>/dev/null

# Processes, logs, network
ps aux --sort=-%cpu | head
journalctl -u my-service -n 100 --no-pager
ss -lntp
getent hosts api.example.com
curl -I --max-time 10 https://api.example.com/health
```

### Kubernetes commands

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl top nodes                 # metrics-server required
kubectl top pods -A
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
kubectl get svc,endpointslice -n <namespace>
kubectl rollout status deploy/<name> -n <namespace>
kubectl rollout history deploy/<name> -n <namespace>
```

### Docker commands

```bash
docker ps -a
docker logs --tail 100 <container>
docker inspect <container>
docker stats --no-stream
docker exec -it <container> sh   # only if shell exists in image
```

### Safe troubleshooting checklist

- **Scope:** How many users, routes, regions, or tenants are affected?
- **Timeline:** Exactly when did it begin? Any deploy, config, traffic, or dependency event?
- **Signals:** Latency, traffic, errors, saturation, plus meaningful business SLIs.
- **Dependencies:** Database, cache, queue, DNS, identity, cloud services, external APIs.
- **Mitigation:** Is a rollback, traffic shift, or load shedding safer than guessing the root cause?
- **Verification:** Did the customer-facing SLI return to normal?
- **Follow-up:** What will prevent recurrence or improve detection next time?

---

## 14. Behavioral and experience-based questions

> Use **STAR**: Situation → Task → Action → Result. The response structures below are **templates**, not statements of your personal experience. Replace placeholders with events and outcomes you can defend technically.

### Behavioral 1 — Tell me about a production incident you handled.
**Suggested structure:**
- **Situation:** Which service, environment, and symptom? How did customers or delivery get affected?
- **Task:** What was your responsibility and escalation role?
- **Action:** Which dashboards, logs, commands, and changes did you use? What did you rule out?
- **Result:** How did you verify recovery? Quantify impact or time savings **only if measured**.
- **Learning:** What runbook, alert, test, or architectural improvement followed?

### Behavioral 2 — Describe an automation you built to reduce toil.
**Suggested structure:** Explain the manual process and how often it occurred; demonstrate design and safety (error handling, retries, logging, idempotency); describe rollout and feedback; report measurable savings if available.

### Behavioral 3 — What if a developer disagrees with an SRE about shipping a risky change?
**Suggested answer:** Align on impact, current error budget, proposed rollback and observability, and the criticality of the release. Offer a canary, feature flag, staging validation, or a narrow time window rather than a personal veto. Escalate according to policy if the risk remains unacceptable.

### Behavioral 4 — How do you handle simultaneous alerts from different services?
**Suggested answer:** Prioritize user impact, severity, dependencies, and blast radius. Confirm whether one upstream failure explains multiple alerts. Delegate investigation, name an incident commander for major incidents, and communicate ownership so multiple people do not perform conflicting mitigations.

### Behavioral 5 — Tell me about a time you made a mistake.
**Suggested structure:** Clearly own your decision, explain how you detected the effect, what you did to protect users, and which safeguard you added. Avoid blaming colleagues and avoid sharing sensitive organizational details.

### Behavioral 6 — Why move from DevOps to SRE?
**Suggested answer:** "I enjoy building automation and reliable delivery systems, and I want to extend that ownership to measurable production reliability—SLIs, SLOs, observability, incident response, safe change management, and capacity. My CI/CD and scripting experience gives me a strong foundation; I'm deliberately building deeper production troubleshooting expertise."

### Behavioral 7 — What should you say if you have not used a tool in production?
**Suggested answer:** "I haven't operated **[tool]** in production yet. I have used **[related tool]** and built a lab with **[specific scenario]**. I understand its purpose and trade-offs, and here's how I would validate it before production rollout." This is more credible than claiming experience you cannot demonstrate.

### Behavioral 8 — How would you explain an outage to a nontechnical manager?
**Suggested answer:** State what users could not do, the start time, current extent, what is being done to restore service, and when the next update is due. Avoid unverified root causes, tool jargon, or promises of recovery times without evidence.

### Role-specific discussion prompts

1. Explain a CI/CD architecture you designed: branching, approvals, stage separation, artifacts, release IDs, rollback, and ownership.
2. Describe how you integrated static analysis/security checks without causing unacceptable pipeline delays.
3. Walk through a failed Linux build or containerized tool execution and explain your debugging sequence.
4. Show a Grafana dashboard you built and defend why each metric matters.
5. Explain how an infrastructure automation or Python script handles failure, retries, and observability.
6. Describe a reliability improvement whose **before and after** you measured.

---

## 15. Practical labs and revision checklist

### Five hands-on labs

**Lab 1 — Build an observable API**
- Write a simple FastAPI app with `/health`, `/ready`, and a request endpoint.
- Expose request count, error count, and latency histograms via Prometheus instrumentation.
- Generate successful, failed, and delayed requests; inspect p95 latency and RPS.
- **Evidence to show:** GitHub repo, screenshot of dashboard, example PromQL, README.

**Lab 2 — Kubernetes fault injection (in a disposable lab)**
- Deploy the API to minikube, kind, or a permitted test cluster.
- Intentionally configure a bad image, invalid Secret reference, too-small memory limit, and failing readiness probe **one at a time**.
- For each: record Pod status, event, logs, root cause, correction, and verification.
- **Evidence to show:** short incident journal with commands and observations.

**Lab 3 — Set an SLO and test a burn-rate alert**
- Define a request-based 99.9% success SLO over 30 days.
- Generate synthetic traffic and compare good vs total requests.
- Draft multiwindow burn-rate rules and test them in a lab; account for evaluation periods and traffic volume.
- **Evidence to show:** alert YAML and a documented test of firing/resolution.

**Lab 4 — CI/CD rollout and recovery**
- Build and test the API in Jenkins or GitHub Actions; push a versioned container image.
- Deploy to a test cluster, run smoke checks, and intentionally introduce a faulty release.
- Demonstrate a safe rollback and verify recovery using service metrics.
- **Evidence to show:** pipeline definition, artifact version, deployment timeline, and postmortem summary.

**Lab 5 — Infrastructure drift and recovery**
- Use Terraform against **nonproduction disposable resources**.
- Produce a plan, make an authorized out-of-band change, inspect drift, and reconcile safely.
- Practice remote-state access controls and explain what state files can expose.
- **Evidence to show:** sanitized plans and an explanation of why changes are safe.

### 14-day revision plan

| Day | Focus | Deliverable |
|---|---|---|
| 1 | SRE, golden signals, RED/USE | Explain SRE vs DevOps out loud |
| 2 | SLI/SLO/SLA, error budgets | Calculate 99.9% allowance and burn rate |
| 3 | Linux CPU/memory/IO | Debug one synthetic host bottleneck |
| 4 | Networking, DNS, TLS, HTTP | Diagnose a slow HTTPS request |
| 5 | Kubernetes basics | Explain Pod → Deployment → Service |
| 6 | K8s incident debugging | Resolve a CrashLoopBackOff |
| 7 | Prometheus/Grafana | Create an RPS + p95 + error dashboard |
| 8 | Logs/traces/OpenTelemetry | Trace one request across components |
| 9 | Alerting and on-call | Design a useful alert + runbook |
| 10 | AWS HA and recovery | Sketch multi-AZ design + RTO/RPO |
| 11 | Terraform/GitOps/CI-CD | Walk through plan → deploy → rollback |
| 12 | Databases/distributed systems | Explain timeout, retry, idempotency |
| 13 | Production scenarios | Timed answers to scenarios 1–16 |
| 14 | Behavioral and mock interview | 45-minute interview + gap review |

### Final interview checklist

- [ ] I can explain SLI, SLO, SLA, and error-budget math without notes.
- [ ] I know the four golden signals and RED/USE monitoring methods.
- [ ] I can write PromQL for RPS, error ratio, and latency percentiles.
- [ ] I can troubleshoot CPU, memory, disk, DNS, TLS, and systemd failures.
- [ ] I can debug CrashLoopBackOff, Pending, ImagePullBackOff, and OOMKilled.
- [ ] I understand probes, HPA, PDB, Deployments, Services, and EndpointSlices.
- [ ] I can describe a safe release, rollback, and database migration strategy.
- [ ] I can design monitoring, alert routing, and an on-call runbook.
- [ ] I can explain RPO/RTO, multi-AZ architecture, and disaster recovery testing.
- [ ] I can discuss Terraform state, drift, least privilege, and secret management.
- [ ] I can walk through three production scenarios systematically.
- [ ] I have two verified, honest STAR stories backed by work or lab evidence.

---

## Official references

- [Google — Site Reliability Engineering](https://sre.google/sre-book/table-of-contents/)
- [Google — The Site Reliability Workbook](https://sre.google/workbook/table-of-contents/)
- [Google — Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
- [Prometheus — Querying basics](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Prometheus — Functions](https://prometheus.io/docs/prometheus/latest/querying/functions/)
- [Prometheus — Alerting rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
- [Prometheus Alertmanager — Overview](https://prometheus.io/docs/alerting/latest/alertmanager/)
- [Kubernetes — Debug Running Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/)
- [Kubernetes — Probes](https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/)
- [Kubernetes — Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [OpenTelemetry — Signals](https://opentelemetry.io/docs/concepts/signals/)
- [AWS — Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)
- [HashiCorp Terraform — State](https://developer.hashicorp.com/terraform/language/state)

---

**Study tip:** In an SRE interview, the strongest answer is usually not the longest command list. It is the answer that connects a technical signal to **customer impact**, proposes a **safe mitigation**, and verifies **recovery**.

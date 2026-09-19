# SiteRealibilityEngineer

SRE Best Practices:

Ensuring reliability - getting systems back to steady-state as quickly as possible
Eliminating toil - automating wherever possible
Blameless postmortems - driving better cross-team collaboration
Observing what matters - gaining full visibility into system health
Being proactive - living and breathing SLOs to identify and remediate issues before SLAs are violated
Architecting for resiliency - Informing architectural design decisions to build more reliable systems


SRE Different From DevOps or Platform Engineer ?

DevOps Engineer:
Focus: Software delivery and automation
Builds and manages CI/CD pipelines
Automates build, test, deployment, and infrastructure
Works with Docker, Kubernetes, Terraform, Jenkins, GitLab CI/CD
Goal: Ship software faster and more safely
Common metrics: Deployment frequency, lead time, change failure rate

SRE — Site Reliability Engineer:
Focus: Reliability, availability, and production stability
Defines SLIs, SLOs, and error budgets
Handles monitoring, alerting, incidents, and on-call
Performs root-cause analysis and postmortems
Reduces repetitive operational work, or toil
Goal: Keep systems reliable while allowing teams to release quickly
Common metrics: Availability, latency, error rate, MTTR

Platform Engineer:
Focus: Making infrastructure easy for developers to use
Builds reusable internal platforms and tooling
Creates CI/CD templates, infrastructure modules, and deployment standards
Provides self-service environments for development teams
Often works with Kubernetes, Helm, Terraform, GitOps, Argo CD, Backstage
Goal: Improve developer productivity and reduce infrastructure complexity

Simple Difference
DevOps: How do we build and deploy faster?

SRE: How do we keep production reliable?

Platform Engineering: How do we make DevOps/infrastructure easy for developers to consume?

Example
Creating a Jenkins pipeline → DevOps
Monitoring pipeline/service reliability and reducing MTTR → SRE
Creating a reusable Jenkins/Kubernetes platform for multiple teams → Platform Engineering

Overlap
These roles are not completely separate
One engineer may work across all three
Smaller companies often combine them into one role
Larger companies are more likely to have dedicated DevOps, SRE, and Platform teams

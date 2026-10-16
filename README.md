# 🚀 DevOps, Cloud, SRE & Platform Engineering — Interview Preparation Hub

> A structured, hands-on knowledge base for preparing for **DevOps Engineer, Cloud Engineer, Site Reliability Engineer (SRE), Platform Engineer, and DevSecOps** interviews.

![Focus](https://img.shields.io/badge/Focus-Interview%20Preparation-0A66C2) ![Cloud](https://img.shields.io/badge/Cloud-AWS-orange) ![IaC](https://img.shields.io/badge/IaC-Terraform-purple) ![Containers](https://img.shields.io/badge/Containers-Kubernetes-blue) ![Automation](https://img.shields.io/badge/Automation-Python%20%7C%20Bash-green)

**Maintained by:** [Hemant Kumar Singh](https://www.linkedin.com/in/hemant11ks/)  
**Background:** DevOps engineering, CI/CD, AWS and on-prem infrastructure, Python automation, observability, Android/AOSP build and release engineering.

This repository combines **theory + commands + real interview scenarios + practical projects + revision plans**. It is a learning resource, not a claim that every technology listed below has been used professionally.

---

## 📚 Table of Contents

1. [How to Use This Repository](#-how-to-use-this-repository)
2. [Role-Based Preparation Tracks](#-role-based-preparation-tracks)
3. [Learning Roadmap](#-learning-roadmap)
4. [Core Skills and Interview Topics](#-core-skills-and-interview-topics)
5. [The 20 Engineering Rules](#-the-20-engineering-rules)
6. [Scenario-Based Interview Questions](#-scenario-based-interview-questions)
7. [Hands-On Labs and Portfolio Projects](#-hands-on-labs-and-portfolio-projects)
8. [Architecture and Design Questions](#-architecture-and-design-questions)
9. [Experience-Based Interview Preparation](#-experience-based-interview-preparation)
10. [Important Commands Cheat Sheet](#-important-commands-cheat-sheet)
11. [Official Documentation and Learning Resources](#-official-documentation-and-learning-resources)
12. [30-Day Study Plan](#-30-day-study-plan)
13. [Self-Assessment Checklist](#-self-assessment-checklist)
14. [Suggested Repository Structure](#-suggested-repository-structure)
15. [Contribution and Security Guidelines](#-contribution-and-security-guidelines)

---

## 🎯 How to Use This Repository

For each skill, complete this loop:

1. **Understand:** Explain what the tool solves, its architecture, and trade-offs without notes.
2. **Practice:** Build or repair something yourself; record exact commands and results.
3. **Troubleshoot:** Deliberately cause a failure and identify the evidence that proves the root cause.
4. **Communicate:** Answer *what, why, how, alternatives, security, reliability, and cost*.
5. **Document:** Save sanitized configuration, diagrams, runbooks, and lessons learned.

**Strong interview answer format:** *Problem → Context → Choice → Implementation → Validation → Impact → Trade-offs.*

> **Important:** Differentiate **hands-on production experience**, **personal lab experience**, and **topics currently studying**. Never present a lab as a production deployment or estimate impact metrics you did not measure.

## 🧭 Role-Based Preparation Tracks

| Role | Highest-priority competencies | Interview emphasis |
|---|---|---|
| **DevOps Engineer** | Linux, Git, CI/CD, Docker, Terraform, scripting, artifacts, security | Pipeline design, deployment failures, release automation |
| **Cloud Engineer (AWS)** | IAM, VPC, EC2, ALB, ASG, S3, RDS, EKS, monitoring, cost | Network architecture, availability, security, IaC |
| **SRE** | SLIs/SLOs, observability, incident response, capacity, toil reduction | Debugging, incident leadership, reliability trade-offs |
| **Platform Engineer** | Kubernetes, GitOps, Helm, reusable IaC, developer portals, RBAC | Self-service golden paths and multi-team operations |
| **DevSecOps** | Least privilege, secrets, SAST, dependency and image scanning, supply-chain security | Secure-by-default delivery and risk-based controls |

### Priority for this repository

**Tier 1 — Master:** Linux, TCP/IP and DNS, Git, Bash/Python, Jenkins, AWS IAM/VPC/EC2/S3, Docker, Kubernetes, Terraform.  
**Tier 2 — Deepen:** Prometheus/Grafana, GitLab CI, Helm, Argo CD, Ansible, Nexus/JFrog, troubleshooting, SRE principles.  
**Tier 3 — Differentiate:** System design, security, performance/cost, incident management, AI-assisted log analysis, AOSP/Gerrit/Repo, behavioral storytelling.

## 🗺️ Learning Roadmap

| Phase | Topics | Practical outcome |
|---|---|---|
| Foundation | Linux, processes, filesystems, networking, Git, Python/Bash | Diagnose a broken Linux service and network path |
| Build & release | Maven/Gradle, Jenkins, GitLab, Nexus/JFrog, testing | Pipeline with quality gates and artifact promotion |
| Cloud & IaC | AWS networking/IAM/compute/storage, Terraform, Ansible | Repeatable, securely provisioned environment |
| Containers & orchestration | Docker, Kubernetes, Helm, Argo CD | Deploy, expose, scale, debug, and roll back an app |
| Reliability | Prometheus, Grafana, alerting, logs, SLOs | Dashboard, SLO calculation, actionable alert, runbook |
| Architecture | HA/DR, scaling, cost, zero-trust access, platform APIs | Explain alternatives and failure-domain trade-offs |

## 🧰 Core Skills and Interview Topics

### 1. Linux & Operating Systems

- Boot process and `systemd`; processes, threads, signals, exit codes, `ulimit`, OOM killer.
- Permissions (`rwx`, ACLs, `sudo`), ownership, users/groups, SSH key permissions.
- CPU/load vs utilization; memory/swap; disk usage vs inode usage; filesystem mounts.
- `journalctl`, `systemctl`, `ps`, `top`, `free`, `df`, `du`, `ss`, `lsof`, `strace` (where permitted).
- Cron vs systemd timers; log rotation; package management; SELinux/AppArmor fundamentals.
- **Practice:** A service fails after reboot. Trace status, unit file, dependency, logs, permissions, and port bind.

### 2. Networking, DNS & TLS

- OSI/TCP-IP, TCP handshake, UDP, TCP resets, ports, CIDR and subnetting.
- DNS resolver, recursive lookup, TTL, A/AAAA/CNAME records; HTTP methods/status codes.
- Routing tables, NAT, security groups vs NACLs, load balancing and reverse proxies.
- TLS handshake and certificate chain, mTLS, proxies, firewalls, packet loss and latency.
- **Practice:** Debug `DNS works but HTTPS fails`; distinguish name resolution, route, TLS, backend and application failures.

### 3. Git, GitHub, GitLab & Gerrit

- Merge vs rebase, fast-forward, cherry-pick, revert vs reset, conflict resolution.
- Branch protection, PR/MR reviews, signed commits/tags, semantic versions, release tags.
- Gerrit patch sets and Change-Ids; Android `repo` manifests and multi-repository sync.
- **Practice:** Recover a bad commit safely without rewriting shared production branch history.

### 4. Scripting & Programming

- **Bash:** quoting, exit codes, `set -euo pipefail` caveats, traps, arrays, streams, `find`, `awk`, `sed`, `jq`.
- **Python:** data structures, exceptions, context managers, virtual environments, argparse, pathlib, subprocess, JSON, requests/HTTP client patterns, retries/timeouts, logging, pytest.
- **Groovy:** Jenkins scripted vs declarative pipeline, shared libraries, sandbox, credentials handling.
- **Practice:** Read a JSON log, group failures by error type, generate a report, and exit nonzero on invalid inputs.

### 5. CI/CD & Artifact Management

- CI vs continuous delivery vs continuous deployment, runners/agents, build reproducibility.
- Jenkins controller/agents, credentials, parameters, triggers, `post` conditions, parallel stages, timeouts, retry, workspace cleanup.
- GitLab CI stages, rules, artifacts, caches, runners, protected variables; GitHub Actions workflow permissions.
- Artifact repositories: immutable versions, metadata, checksums, retention, provenance, promotion and rollback.
- Maven and Gradle build lifecycles; unit/integration tests, SAST, code coverage, SBOM and image scan gates.
- **Practice:** Implement *checkout → test → scan → build → publish → stage deploy → smoke test → approval → production deploy → health check*.

### 6. AWS & Cloud Architecture

- **IAM:** users, roles, trust vs permission policies, least privilege, STS, instance profiles, service credentials.
- **VPC:** public/private subnets, routes, IGW, NAT gateway, VPC endpoints, security groups, NACLs, DNS.
- **Compute:** EC2, launch templates, ASGs, load balancers, health checks, Lambda, EKS.
- **Data:** S3 versioning/encryption/lifecycle, RDS backup and Multi-AZ, EBS vs EFS.
- **Visibility:** CloudWatch metrics/logs/alarms, CloudTrail API audit events.
- **Design:** availability zones, recovery objectives (RTO/RPO), cost controls, infrastructure tagging.
- **Practice:** Give a private EC2 instance S3 access with a correctly routed/configured gateway endpoint and least-privilege IAM.

### 7. Infrastructure as Code & Configuration Management

- Terraform providers, state, plan/apply, dependencies, modules, variables, outputs, lifecycle and drift.
- Remote state, state locking where supported by backend, protected state access, workspace vs environment separation.
- Ansible inventory, idempotent modules, roles, handlers, templates, Vault, check mode.
- **Practice:** Review a Terraform plan in CI; promote changes with a reviewed approval step and no long-lived cloud access keys.

### 8. Docker & Containers

- Image layers, build context, cache, multi-stage builds, `.dockerignore`, tags vs digests.
- Process isolation (namespaces/cgroups), networking, bind mounts vs volumes, health checks.
- Rootless/non-root containers, minimal images, resource limits, image signing/scanning.
- **Practice:** Shrink an image and debug a container that exits immediately.

### 9. Kubernetes, Helm & GitOps

- Control plane, API server, etcd, scheduler, controllers, kubelet, CRI and CNI.
- Pods, Deployments, ReplicaSets, StatefulSets, DaemonSets, Jobs/CronJobs.
- Services, DNS, Ingress/Gateway API, NetworkPolicies, requests/limits, probes.
- PV/PVC/StorageClasses, ConfigMaps, Secrets, service accounts, RBAC, HPA, PDB.
- Scheduling, taints/tolerations, affinities, rolling updates, rollback, CrashLoopBackOff and Pending diagnostics.
- Helm charts, templates, values, release revisions; Argo CD desired vs live state, sync, drift, health.
- **Practice:** Break a deployment (bad probe, missing secret, insufficient resources), collect events/logs, and fix it.

### 10. Monitoring, Logging, SRE & Incident Response

- Metrics vs logs vs traces, golden signals (latency, traffic, errors, saturation).
- Prometheus scrape model, exporters, PromQL, Alertmanager routing/inhibition, Grafana dashboards.
- Elasticsearch/ELK indexing and query basics; CloudWatch dashboards and alerts.
- **SLI:** measurable service performance indicator. **SLO:** target for that indicator. **SLA:** contractual commitment (often with remedies).
- Error budgets, burn rates, incident severity, escalation, on-call, runbooks, MTTR, postmortems without blame.
- **Practice:** Define a request-success SLI, calculate the error budget, and create an alert that is actionable rather than noisy.

### 11. Security & DevSecOps

- Secrets managers vs environment variables; short-lived identities over hard-coded secrets.
- SAST, DAST, SCA, SBOM, provenance and dependency pinning; Coverity and defect triage.
- IAM/RBAC boundaries, TLS, secure SSH, audit logs, vulnerability response, patching.
- Kubernetes pod security, restricted service accounts and network policy.
- **Practice:** Find and remove an exposed credential; rotate it and review relevant access logs.

### 12. Databases & Distributed Systems

- SQL fundamentals, indexes, joins, transactions, ACID, isolation, locks, connection pooling.
- Backups vs replicas, recovery testing, eventual consistency, caches, queues, retries, idempotency.
- **Practice:** Explain a database connection spike following a deployment and how to mitigate it.

### 13. AI-Assisted Operations (Differentiator)

- RAG: ingestion → parsing → chunking → indexing → retrieval → generation → evaluation.
- Search and semantic retrieval, Elastic/Elasticsearch, log normalization and timestamps.
- Failure grouping, traceability, hallucination checks, data redaction, least-privilege log access.
- **Practice:** Show a sanitized log query, retrieved evidence and a human-verifiable diagnosis; measure retrieval quality and investigation time.

### 14. AOSP / Automotive Build & Release (Specialization)

- Android `repo` and manifest synchronization, Gerrit reviews and patch sets.
- Build dependencies, toolchains, artifacts, caches, reproducible release handoffs.
- Jenkins integration, Coverity scans, binary promotion and on-prem agent troubleshooting.
- **Practice:** Explain a failed `repo sync` caused by a missing remote ref, stale lock, or inconsistent manifest — and safe recovery steps.

## 📏 The 20 Engineering Rules

1. **Automate repeated, deterministic tasks**; maintain a clear manual recovery path.
2. **Build once, promote the same artifact** across environments; do not rebuild different binaries per environment.
3. **Keep infrastructure and pipeline configuration version-controlled** with reviewable changes.
4. **Never commit secrets**; use short-lived identity and managed secret storage where possible.
5. **Grant least privilege** to humans, services, runners and workloads.
6. **Treat production as immutable where feasible**; avoid manual drift.
7. **Require evidence before root-cause conclusions**: logs, events, metrics, traces, timestamps.
8. **Use timeouts, bounded retries and backoff**; avoid retry storms.
9. **Design operations to be idempotent**, especially deployment and repair scripts.
10. **Validate inputs and quote shell variables**; make destructive steps explicit.
11. **Prefer staged rollout, health checks and rollback** over all-at-once releases.
12. **Make releases traceable** to source revision, build, tests and artifacts.
13. **Set requests, limits, probes and disruption controls** according to real workload behavior.
14. **Measure user impact** rather than alerting solely on machine CPU thresholds.
15. **Define SLIs and SLOs before treating uptime as a success metric**.
16. **Prioritize restoration during incidents**, then perform root-cause analysis.
17. **Test backups and disaster recovery**, not merely their existence.
18. **Document runbooks and incident timelines** so others can act independently.
19. **Watch cloud cost and resource lifecycle**; use budgets and clean up test resources.
20. **Know the failure domain and trade-offs** for every architecture decision.

## 🧩 Scenario-Based Interview Questions

### Scenario 1 — Private EC2 cannot access S3, with no internet

**Approach:** Check EC2 IAM role permissions, DNS resolution, the VPC route table and security controls. For ordinary S3 access from a private subnet, use an **S3 gateway VPC endpoint** with the appropriate route-table association and policies. Verify region compatibility and test a permitted S3 operation. Distinguish bucket, endpoint and IAM policy denials from connectivity failures.

### Scenario 2 — A 6 PM production deployment breaks traffic

**Approach:** Declare/triage incident severity; stop further rollout, assess blast radius, use metrics/logs/traces to establish timeline, and roll back or mitigate via the fastest safe path. Communicate status, validate recovery with user-facing indicators, then hold a blameless postmortem.

### Scenario 3 — Jenkins pipeline fails only on one agent

**Approach:** Compare agent labels, OS/tool versions, workspace permissions, disk/inodes, Java/runtime versions, network access, credentials scope and environment variables. Inspect agent logs and the *first causal error*; avoid restarting the controller as an unexplained permanent fix.

### Scenario 4 — Pod is `CrashLoopBackOff`

**Approach:** `kubectl describe pod`, `kubectl logs --previous`, inspect exit code/OOMKilled, startup config, secret mounts, probe settings and dependencies. Correct underlying issue; do not blindly increase replicas.

### Scenario 5 — Pod is `Pending`

**Approach:** Read scheduler events. Check node availability, CPU/memory requests, taints/tolerations, affinity, quotas, PVC bindings and autoscaler behavior.

### Scenario 6 — Terraform detects unexpected production changes

**Approach:** Review plan/state and audit history; identify manual changes or version drift. Decide whether to codify intended changes or safely reconcile infrastructure. Never force-apply a destructive plan without review and rollback strategy.

### Scenario 7 — 5xx errors rise but CPU remains normal

**Approach:** Split by ingress/service/route, inspect error rates and latency, upstream dependencies, connection pools, recent changes, timeouts and load balancer target health. CPU is only one signal.

### Scenario 8 — Prometheus alert storm during a dependency outage

**Approach:** Identify root dependency, route page-worthy symptoms, group alerts in Alertmanager, use inhibition judiciously, tune alert windows, and preserve the actionable signal.

### Scenario 9 — `git` checkout fails during Android `repo sync`

**Approach:** Verify referenced commit/branch exists in the configured remote, review manifest revision and repository permissions, check for concurrent Git processes before handling stale lock files, and resync only the affected project first. Do not delete repository data before verifying a recoverable source.

### Scenario 10 — Deploy a database schema change without downtime

**Approach:** Use expand-and-contract migrations; ship backward-compatible application changes, backfill safely, observe, switch reads/writes, then remove deprecated fields in a later release. Plan rollback separately for irreversible migrations.

### More practice prompts

- Why is an application reachable from one subnet but not another?
- How would you diagnose TLS `certificate verify failed`?
- What happens when a Kubernetes worker node dies?
- When should you choose StatefulSet instead of Deployment?
- How do you prevent a compromised CI runner from accessing production secrets?
- Explain blue/green vs canary vs rolling deployments.
- A Jenkins downstream stage fails; how do you avoid duplicate notifications?
- How do you choose between S3, EBS and EFS?
- What would you monitor for a business-critical checkout API?
- How do you distinguish a deployment regression from a traffic surge?
- How can you validate whether a RAG-based incident explanation is grounded?
- How do you design a multi-AZ application to meet specific RTO/RPO targets?

## 🛠️ Hands-On Labs and Portfolio Projects

| Project | Deliverables | Skills demonstrated |
|---|---|---|
| **01 — Linux incident lab** | Failure runbook, commands, evidence, fix and prevention | Linux, network, troubleshooting |
| **02 — CI/CD pipeline** | Jenkinsfile, test reports, versioned artifact, rollback instructions | Jenkins, Git, Maven, Nexus |
| **03 — AWS Terraform stack** | VPC, private/public tiers, IAM, backend, README architecture | AWS, Terraform, least privilege |
| **04 — Container delivery** | Multi-stage Dockerfile, K8s YAML/Helm, probes, rollbacks | Docker, Kubernetes, Helm |
| **05 — GitOps delivery** | Argo CD app, Git-sourced configuration, sync policies | GitOps, reconciliation |
| **06 — SRE observability** | Dashboard, recording/alert rules, SLO and on-call runbook | Prometheus, Grafana, SRE |
| **07 — Python ops CLI** | Tested CLI for API/log processing, retries, structured logs | Python, testing, automation |
| **08 — Secure supply chain** | SAST/SCA/image scans, SBOM, permissions model | DevSecOps |
| **09 — Log analysis assistant** | Sanitized sample dataset, retrieval/evaluation demo | Elasticsearch, RAG, LLMs |
| **10 — Developer golden path** | Template repo, CI, self-service deploy documentation | Platform engineering |

For each project, document **architecture, prerequisites, setup, validation, cleanup, known risks, screenshots/log snippets without secrets, and lessons learned**. Set a cloud budget before launching AWS resources.

## 🏗️ Architecture and Design Questions

When asked to design a system, explain:

1. **Requirements:** expected traffic, availability, latency, data retention, budget, compliance.
2. **Architecture:** clients → DNS/CDN → LB/API → services → storage/queue/database.
3. **Network/security:** VPC, segmented subnets, IAM, authentication, encryption, secrets.
4. **Reliability:** health checks, multi-AZ, failover, backups, failure domains and RTO/RPO.
5. **Delivery:** IaC, pipeline, artifact promotion, rollout, rollback.
6. **Operations:** logs, metrics, tracing, on-call alerts, SLO, runbooks.
7. **Trade-offs:** cost, complexity, scaling limits, operational burden.

**Design practice:** A secure CI/CD platform for 20 development teams; a highly available web API on AWS; a private EKS cluster; an internal developer deployment platform; an observable batch pipeline; a disaster recovery strategy.

## 🗣️ Experience-Based Interview Preparation

The examples below are **story prompts grounded in the supplied resume**, not fully scripted claims. Fill in actual scale, constraints, decisions and validation before interviews.

| Story | Stated evidence from experience | Be ready to explain |
|---|---|---|
| **Pipeline optimization** | Jenkins/GitLab/Groovy, ~20% reduction in deployment time | Baseline, bottleneck, optimizations, measurement, failure handling |
| **IaC provisioning** | Terraform and Ansible; setup from hours to minutes | Modules, remote state, identity, configuration drift, approvals |
| **Observability** | Prometheus/Grafana, ~25% lower average MTTR | Important signals, alert noise, before/after methodology |
| **Security gates** | Coverity C++/Java/Android, ~30% lower manual review turnaround | CI integration, defect triage, false positives, exception process |
| **Artifacts and promotion** | Nexus/JFrog integration and rollback practices | Version naming, immutability, checksum, provenance |
| **Source and triggers** | Jenkins with GitLab/Gerrit branch-specific workflows | Webhooks/triggers, patch sets, concurrency, merge safeguards |
| **Incident handling** | On-call/night support, runbooks, SLIs/SLOs | One anonymized incident with detection → mitigation → follow-up |
| **AI log analysis project** | Python, Elasticsearch, RAG/LLMs, ~30% lower RCA investigation time | Corpus, retrieval method, evaluation, hallucination protection |
| **Cross-team collaboration** | Developer self-service and automotive release workflows | Stakeholders, customer constraints, feedback and adoption |

### STAR story template

```text
Situation: What failed or needed improvement? What was the scope?
Task: What were you directly responsible for?
Action: What did YOU design, change, debug, or automate? Why?
Result: What improved? How was it measured or validated?
Reflection: What trade-off, risk, or improvement would you address now?
```

**Behavioral prompts:** Tell me about a failed deployment; a difficult incident; a disagreement with a developer; a manual task you automated; a change that reduced MTTR; a security concern you escalated; an incident you could not solve alone; a time you mentored or supported others.

**Honesty check:** Say “I worked with” for collaborative efforts, not “I architected” unless you led the design. Clarify whether a tool was used in production, a POC, or a learning project. Keep proprietary customer data out of public interview materials.

## ⌨️ Important Commands Cheat Sheet

> Illustrative commands. Replace placeholders with your own environment; review effects before executing.

### Linux and networking

```bash
uname -a
uptime
free -h
df -h
df -i
ps aux --sort=-%cpu | head
systemctl status <service>
journalctl -u <service> -n 100 --no-pager
ss -lntp
dig example.com
curl -Iv https://example.com
ip route
```

### Git

```bash
git status
git log --oneline --graph -10
git diff
git switch -c feature/example
git fetch --prune
git revert <commit-sha>          # safer than rewriting shared history
```

### Docker

```bash
docker build -t demo:local .
docker images
docker ps -a
docker logs <container>
docker inspect <container>
docker exec -it <container> sh
```

### Kubernetes

```bash
kubectl get pods -A
kubectl get events -A --sort-by=.lastTimestamp
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl get deploy,svc,ingress -n <namespace>
kubectl rollout status deployment/<name> -n <namespace>
kubectl rollout history deployment/<name> -n <namespace>
```

### Terraform

```bash
terraform fmt -check
terraform init
terraform validate
terraform plan -out=tfplan
terraform show tfplan            # review before applying
# terraform apply tfplan         # only after approval
```

### AWS CLI

```bash
aws sts get-caller-identity
aws ec2 describe-instances --region <region>
aws s3 ls s3://<bucket>
aws cloudwatch describe-alarms --region <region>
```

### Helm and Argo CD

```bash
helm lint ./chart
helm template demo ./chart
helm list -A
argocd app list
argocd app get <app>
```

### Jenkins / APIs

```bash
curl --fail-with-body --silent --show-error https://example.com/health
jq '.status' response.json
```

## 🔗 Official Documentation and Learning Resources

Official documentation is the primary source; blog posts, YouTube tutorials and AI-generated explanations are supplements. Check version-specific docs for the technology running in your environment.

| Subject | Authoritative references |
|---|---|
| Linux | [Linux kernel documentation](https://docs.kernel.org/), [GNU Bash manual](https://www.gnu.org/software/bash/manual/), [systemd](https://www.freedesktop.org/software/systemd/man/) |
| Git / GitHub / Gerrit | [Pro Git](https://git-scm.com/book/en/v2), [GitHub Docs](https://docs.github.com/), [Gerrit Documentation](https://gerrit-review.googlesource.com/Documentation/) |
| Python | [Python docs](https://docs.python.org/3/), [pytest](https://docs.pytest.org/) |
| Networking | [Cloudflare Learning Center](https://www.cloudflare.com/learning/), [MDN HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP) |
| AWS | [AWS Documentation](https://docs.aws.amazon.com/), [AWS Well-Architected](https://docs.aws.amazon.com/wellarchitected/), [AWS Security Best Practices](https://docs.aws.amazon.com/security/) |
| Terraform | [Terraform docs](https://developer.hashicorp.com/terraform/docs), [Terraform language](https://developer.hashicorp.com/terraform/language) |
| Ansible | [Ansible Documentation](https://docs.ansible.com/) |
| Docker | [Docker Docs](https://docs.docker.com/) |
| Kubernetes | [Kubernetes Concepts](https://kubernetes.io/docs/concepts/), [Tasks](https://kubernetes.io/docs/tasks/), [kubectl Reference](https://kubernetes.io/docs/reference/kubectl/) |
| Helm / Argo CD | [Helm Docs](https://helm.sh/docs/), [Argo CD Docs](https://argo-cd.readthedocs.io/en/stable/) |
| Jenkins | [Jenkins User Handbook](https://www.jenkins.io/doc/), [Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/) |
| GitLab / Actions | [GitLab CI/CD](https://docs.gitlab.com/ci/), [GitHub Actions](https://docs.github.com/en/actions) |
| Maven / Gradle | [Maven Guides](https://maven.apache.org/guides/), [Gradle User Manual](https://docs.gradle.org/current/userguide/userguide.html) |
| Artifact tooling | [Sonatype Nexus Help](https://help.sonatype.com/), [JFrog Documentation](https://jfrog.com/help/r/jfrog-platform-documentation) |
| Observability | [Prometheus Docs](https://prometheus.io/docs/), [Grafana Docs](https://grafana.com/docs/), [Elastic Docs](https://www.elastic.co/docs) |
| SRE | [Google SRE Books](https://sre.google/books/), [SLO Resources](https://sre.google/resources/book-update/slos/) |
| Security | [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/), [OWASP CI/CD Top 10](https://owasp.org/www-project-top-10-ci-cd-security-risks/), [NIST CSF](https://www.nist.gov/cyberframework) |
| OpenTelemetry | [OpenTelemetry Docs](https://opentelemetry.io/docs/) |
| Android/AOSP | [Android Open Source Project](https://source.android.com/), [Android Debug Bridge](https://developer.android.com/tools/adb) |

### Recommended references to read deeply

- **AWS Well-Architected Framework:** Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization and Sustainability.
- **Google SRE Book / Workbook:** SLI, SLO, error budgets, monitoring and incident practices.
- **Kubernetes Concepts:** Controllers, networking, scheduling, security and storage.
- **Terraform Language and State:** Modules, plans, dependency graph, remote backends and drift.
- **Jenkins Pipeline Handbook:** Pipeline-as-Code patterns and credential-safe build stages.

## 📅 30-Day Study Plan

| Days | Focus | Proof of learning |
|---|---|---|
| 1–3 | Linux + SSH + systemd + troubleshooting | Reproduce and debug a service failure |
| 4–5 | Networking + DNS + HTTP/TLS | Explain a broken request path |
| 6–7 | Git + Bash + Python | Automate a safe, tested operation |
| 8–10 | Jenkins + GitLab CI + artifacts | Working pipeline with tests and notifications |
| 11–14 | AWS IAM, VPC, EC2, S3, ALB | Network diagram + least-privilege architecture |
| 15–17 | Terraform + Ansible | Repeatable provision/configuration lab |
| 18–20 | Docker + Kubernetes | Deploy, break, debug, recover app |
| 21–22 | Helm + Argo CD | GitOps-based release |
| 23–25 | Prometheus + Grafana + SRE | SLO dashboard and incident runbook |
| 26–27 | Security + supply chain + cost | Pipeline security review |
| 28 | System design | Two whiteboard architecture exercises |
| 29 | Resume stories | Six evidence-based STAR answers |
| 30 | Mock interview | 45-minute technical + 15-minute behavioral round |

**Daily structure (suggested):** 30 minutes concepts, 60 minutes lab, 20 minutes scenario questions, 10 minutes revision. Adjust to available time and weak areas.

## ✅ Self-Assessment Checklist

- [ ] I can troubleshoot Linux services, CPU, memory, disk, permissions and processes.
- [ ] I can explain DNS, TCP, TLS, subnetting, NAT and VPC endpoint routing.
- [ ] I can write a robust Bash/Python automation with error handling and tests.
- [ ] I can compare CI/CD tools and explain real pipeline failures.
- [ ] I can show how immutable artifacts move from build to production.
- [ ] I can design AWS least-privilege networking and IAM.
- [ ] I can explain Terraform state, modules, drift and controlled rollouts.
- [ ] I can package an app using secure Docker practices.
- [ ] I can investigate `Pending`, `CrashLoopBackOff`, probe and connectivity failures in Kubernetes.
- [ ] I can explain Kubernetes RBAC, service accounts, scheduling and storage.
- [ ] I can deploy and roll back through Helm/Argo CD.
- [ ] I can define an SLI/SLO and alert on actionable symptoms.
- [ ] I can walk through incident detection, mitigation, communication and postmortem.
- [ ] I can explain CI/CD security gates and secret management.
- [ ] I can sketch a highly available system and articulate cost/reliability trade-offs.
- [ ] I have at least 5 tested, evidence-based STAR stories from my actual work.

## 🗂️ Suggested Repository Structure

> Folders below are **suggested**; they are not assumed to exist. Create them as you add notes, labs and interview answers. Link them from this README once populated.

```text
DevOps-Interview-Preparation/
├── README.md
├── 00-Roadmap/
│   └── 30-Day-Plan.md
├── 01-Linux-and-Shell/
├── 02-Networking-and-SSH/
├── 03-Git-GitHub-Gerrit/
├── 04-Python-and-Automation/
├── 05-CI-CD/
│   ├── Jenkins/
│   ├── GitLab-CI/
│   └── GitHub-Actions/
├── 06-AWS-and-Cloud/
├── 07-Terraform-and-Ansible/
├── 08-Docker-and-Kubernetes/
├── 09-Helm-and-GitOps/
├── 10-Observability-and-SRE/
├── 11-Security-and-DevSecOps/
├── 12-Artifact-and-Build-Tools/
├── 13-AOSP-and-Android/
├── 14-System-Design/
├── 15-Interview-Questions/
│   ├── Scenario-Based.md
│   ├── Behavioral-STAR.md
│   └── Quick-Revision.md
├── 16-Hands-On-Projects/
├── 17-AI-for-Operations/
└── assets/
```

**Note template for each topic:** `What is it?` → `Why use it?` → `Architecture` → `Core commands` → `Sample config` → `Common failures` → `Interview Q&A` → `Lab` → `Official links`.

## 🤝 Contribution and Security Guidelines

- Open an issue or pull request with corrections, safer practices, documentation updates or reproducible labs.
- Keep commands version-aware and clearly label any destructive operations.
- Do not publish employer/customer code, internal IPs/hostnames, Jenkins configurations, proprietary build manifests, tokens, access keys or private incident details.
- Use anonymized fixtures and synthetic logs for examples.
- Verify commands in an isolated environment and link to upstream docs when possible.
- Treat AI output as a draft: test commands, validate facts and review security impact.
- Keep personal identifiers limited to information you intentionally want public on GitHub.

---

### Final Interview Reminder

**Knowing commands is useful. Explaining why an architecture was chosen, how it fails, how you detect it, how you recover, and how you measure improvement is what differentiates a strong engineer.**

*Study. Build. Break. Debug. Document. Repeat.*

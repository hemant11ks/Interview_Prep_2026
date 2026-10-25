# AWS End-to-End Interview Notes | DevOps, SRE, Cloud & Platform Engineering

> **Audience:** DevOps / Cloud / SRE / Platform Engineer interviews (roughly 1–5 years of experience).  
> **Goal:** Explain AWS concepts, make design decisions, operate production workloads, troubleshoot incidents, and answer scenario-based interviews.  
> **Last reviewed:** October 2026. AWS services, quotas, prices, instance families and recommendations change; verify links in [Official Documentation](#34-official-documentation-and-further-study) before production use.

[![AWS](https://img.shields.io/badge/AWS-Interview%20Notes-orange)](https://aws.amazon.com/) [![Focus](https://img.shields.io/badge/Focus-DevOps%20%7C%20SRE%20%7C%20Platform-blue)](#) [![Format](https://img.shields.io/badge/Format-Notes%20%2B%20Q%26A%20%2B%20Labs-green)](#)

**Navigation:** [Quick Revision](#1-aws-at-a-glance) · [Core Services](#4-compute--amazon-ec2) · [Networking](#6-vpc-networking-deep-dive) · [Security](#3-identity-security-and-governance) · [Containers](#11-containers-ecr-ecs-eks-and-fargate) · [IaC & CI/CD](#13-infrastructure-as-code-and-configuration-management) · [Troubleshooting](#23-production-troubleshooting-playbook) · [Interview Q&A](#25-interview-questions-and-model-answers) · [Labs](#30-hands-on-labs) · [Cheat Sheet](#31-aws-cli-cheat-sheet)

## Table of Contents

1. [AWS at a Glance](#1-aws-at-a-glance)
2. [Cloud Foundations and Global Infrastructure](#2-cloud-foundations-and-global-infrastructure)
3. [Identity, Security and Governance](#3-identity-security-and-governance)
4. [Compute — Amazon EC2](#4-compute--amazon-ec2)
5. [Load Balancing and Auto Scaling](#5-load-balancing-and-auto-scaling)
6. [VPC Networking Deep Dive](#6-vpc-networking-deep-dive)
7. [Route 53, CloudFront and Edge](#7-route-53-cloudfront-and-edge)
8. [Amazon S3 and Object Storage](#8-amazon-s3-and-object-storage)
9. [EBS, EFS and FSx](#9-ebs-efs-and-fsx)
10. [Databases and Caching](#10-databases-and-caching)
11. [Containers: ECR, ECS, EKS and Fargate](#11-containers-ecr-ecs-eks-and-fargate)
12. [Serverless and Event-Driven Architecture](#12-serverless-and-event-driven-architecture)
13. [Infrastructure as Code and Configuration Management](#13-infrastructure-as-code-and-configuration-management)
14. [AWS CI/CD and Deployment Strategies](#14-aws-cicd-and-deployment-strategies)
15. [Observability: CloudWatch, CloudTrail and X-Ray](#15-observability-cloudwatch-cloudtrail-and-x-ray)
16. [Security Operations and DevSecOps](#16-security-operations-and-devsecops)
17. [Backup, High Availability and Disaster Recovery](#17-backup-high-availability-and-disaster-recovery)
18. [Cost Optimization and FinOps](#18-cost-optimization-and-finops)
19. [Architecture Patterns and Well-Architected](#19-architecture-patterns-and-well-architected)
20. [Hybrid, Multi-Account and Migration](#20-hybrid-multi-account-and-migration)
21. [Linux, Networking and Automation on AWS](#21-linux-networking-and-automation-on-aws)
22. [Production Design Walkthroughs](#22-production-design-walkthroughs)
23. [Production Troubleshooting Playbook](#23-production-troubleshooting-playbook)
24. [Scenario-Based Interview Rounds](#24-scenario-based-interview-rounds)
25. [Interview Questions and Model Answers](#25-interview-questions-and-model-answers)
26. [AWS vs Other Cloud / Common Comparisons](#26-aws-vs-other-cloud--common-comparisons)
27. [Common Misconceptions and Interview Traps](#27-common-misconceptions-and-interview-traps)
28. [DevOps/SRE Project Story Template](#28-devopssre-project-story-template)
29. [Interview Study Roadmap](#29-interview-study-roadmap)
30. [Hands-on Labs](#30-hands-on-labs)
31. [AWS CLI Cheat Sheet](#31-aws-cli-cheat-sheet)
32. [Flashcards and Rapid-Fire Recap](#32-flashcards-and-rapid-fire-recap)
33. [Self-Assessment Checklist](#33-self-assessment-checklist)
34. [Official Documentation and Further Study](#34-official-documentation-and-further-study)

---

## How to use these notes

- **Round 1 (screening):** Learn service purposes and comparison tables. Practice 30-second answers.
- **Round 2 (technical):** Explain request flow, networking, IAM evaluation, storage, container platforms, CI/CD and failure modes.
- **Round 3 (hands-on):** Practice CLI, incident checklists, IAM role assumptions and VPC reachability reasoning.
- **Round 4 (design/managerial):** Explain trade-offs, RTO/RPO, cost, operational ownership, blast radius and measurable results.
- **Interview answer formula:** **Definition → Why → When → Example → Trade-off / Failure mode**.

> **Safety and cost:** Sample commands are educational; do not run change/delete commands blindly. AWS resources can incur charges (NAT Gateways, public IPv4, EKS control planes, RDS, data transfer, load balancers, logs). Use test accounts, least-privilege roles, unique placeholder values and teardown. Never paste live credentials into code or Git.

---

## 1. AWS at a Glance

**Amazon Web Services (AWS)** provides on-demand compute, storage, networking, databases, identity, analytics, and managed services. You pay according to each service's billing model rather than owning physical infrastructure.

| Need | Typical AWS service | Interview one-liner |
|---|---|---|
| Virtual machines | **EC2** | Full guest OS/application control on AWS-managed physical infrastructure |
| Object storage | **S3** | Durable object storage with bucket/object API, not a POSIX disk |
| Block disks | **EBS** | Persistent block storage for supported EC2 configurations |
| Shared file system | **EFS** | Managed elastic NFS file system |
| Private network | **VPC** | Isolated virtual network with subnets, routes and controls |
| DNS | **Route 53** | Authoritative DNS, routing policies and health checks |
| HTTP reverse proxy | **ALB** | Layer-7 host/path-based request routing |
| TCP/UDP load balancing | **NLB** | Layer-4 load balancing, static IP options |
| Instance scaling | **EC2 Auto Scaling** | Adjusts instance fleet capacity to demand |
| Relational DB | **RDS / Aurora** | Managed relational engines and operations |
| NoSQL key-value | **DynamoDB** | Managed, scalable key-value/document database |
| Cache | **ElastiCache** | Managed low-latency in-memory data store |
| Functions | **Lambda** | Run event-driven code without managing servers |
| Container registry | **ECR** | Private/public container image storage |
| Containers | **ECS / EKS** | Managed orchestration; EKS uses Kubernetes |
| Identity | **IAM / IAM Identity Center** | Authorization and workforce access |
| Encryption keys | **KMS** | Managed cryptographic key service |
| Secrets | **Secrets Manager** | Secret storage, retrieval and rotation support |
| Monitoring | **CloudWatch** | Metrics, logs, alarms and operational telemetry |
| Audit/API activity | **CloudTrail** | Records supported account/API events |
| AWS config history | **AWS Config** | Resource inventory, history and configuration rules |
| Threat detection | **GuardDuty** | Detects suspicious and malicious activity |
| Infrastructure as code | **CloudFormation / Terraform** | Declarative infrastructure provisioning |
| CI/CD | **CodePipeline, CodeBuild, CodeDeploy** | Managed pipeline, build and deployment steps |

**Six Well-Architected pillars:** Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability.

### Common service choices

```text
Need to host a VM?               -> EC2
Need managed Kubernetes?        -> EKS
Need containers without K8s?    -> ECS
Need no server management?      -> Lambda / Fargate (depends on workload)
Need HTTP(S) routing?           -> ALB
Need TCP/UDP traffic?           -> NLB
Need static files and objects?  -> S3 (+ CloudFront for CDN)
Need a relational database?     -> RDS / Aurora
Need simple key-value at scale? -> DynamoDB
Need asynchronous queue?       -> SQS
Need event fan-out?             -> SNS / EventBridge
```

## 2. Cloud Foundations and Global Infrastructure

### 2.1 Service models

| Model | You manage | Provider manages | Example |
|---|---|---|---|
| On-premises | Everything | Nothing | Your datacenter |
| IaaS | OS, patches, app, data | Physical hardware, facilities, virtualization | EC2 |
| PaaS / managed service | Application/data/configuration (varies) | Most platform operations | RDS, Elastic Beanstalk |
| FaaS / serverless | Code, identity, configuration | Servers and runtime fleet | Lambda |
| SaaS | Tenant data, identity and settings | Application and underlying stack | Third-party SaaS |

AWS follows a **shared responsibility model**: AWS is responsible for security **of** the cloud; customers are responsible for security **in** the cloud, with scope differing by service. A managed service is not automatically compliant or securely configured.

### 2.2 Global terminology

- **Region:** Geographic area containing multiple isolated Availability Zones (AZs). Most resource scopes are Regional.
- **Availability Zone:** One or more discrete data centers with redundant power/networking; AZs are connected by low-latency links within a Region.
- **Edge location:** Infrastructure for edge services such as CloudFront / Route 53.
- **Local Zone / Wavelength:** Location or telecom-edge options for selected low-latency use cases.
- **Global service vs Regional service:** Understand service scope; IAM has global behavior, EC2 and VPC are Regional, and CloudFront distributions are global. AWS service availability varies.

**HA vs DR:** Multi-AZ protects against AZ-level failures. Multi-Region is a separate design decision for Regional disasters, regulatory needs, or latency—not automatically enabled by Multi-AZ.

### 2.3 Core mental model

```text
AWS Organization
└── AWS Account (billing/security boundary)
    └── Region (e.g., ap-south-1)
        └── VPC: 10.20.0.0/16
            ├── AZ A: public and private subnets
            └── AZ B: public and private subnets
```

Interview note: **Regions are not interchangeable AZs**, and an AWS account is a strong isolation boundary for blast radius and governance.

## 3. Identity, Security and Governance

### 3.1 IAM components

| Component | Purpose | Example |
|---|---|---|
| Principal | Identity making a request | IAM role session, user, AWS service |
| IAM user | Long-lived identity (avoid for workloads) | Legacy service account |
| IAM role | Assumable identity with temporary credentials | EC2 instance profile, CI/CD deploy role |
| Identity-based policy | Grants/restricts actions for identities | `s3:GetObject` on a resource |
| Resource-based policy | Policy on resource | S3 bucket policy, KMS key policy |
| Trust policy | Who can assume a role | `ec2.amazonaws.com`, OIDC principal |
| Permission boundary | Maximum IAM user/role identity permissions | Guardrails for delegated IAM |
| SCP | Organization-level guardrail on member accounts | Deny disallowed Regions |
| Session policy | Restricts permissions of a role session | Federation session limits |
| IAM Identity Center | Central workforce SSO | Engineers access multiple accounts |

**Policy evaluation (simplified):** Default implicit deny → check for **explicit deny** → require applicable allow. Identity policies, resource policies, permission boundaries, SCPs, resource control policies (where applicable), session policies, and organizational context interact. **Do not** assume an IAM policy Allow alone guarantees access. In cross-account access, evaluate both sides and resource-specific behavior. KMS key policies have additional significance.

**Example least-privilege IAM policy (illustrative):**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListOnlyBuildArtifacts",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::EXAMPLE-BUILD-BUCKET",
      "Condition": {"StringLike": {"s3:prefix": ["releases/*"]}}
    },
    {
      "Sid": "ReadBuildArtifacts",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::EXAMPLE-BUILD-BUCKET/releases/*"
    }
  ]
}
```

**S3 distinction:** `s3:ListBucket` applies to the *bucket ARN*; `s3:GetObject` applies to *object ARNs*. A policy may also depend on KMS permissions, bucket policy, Block Public Access settings, endpoint policy and SCPs.

### 3.2 Temporary AWS credentials

1. A workload or user authenticates to a trusted identity provider/service.
2. **AWS STS** issues scoped, temporary credentials when a role is assumed or credentials are exchanged.
3. AWS evaluates policies and request context for the action.
4. Session expires; refresh is managed by the SDK/credentials provider, if configured.

```bash
# Read-only identity inspection; safe to run in your own authorized account
aws sts get-caller-identity
aws configure list
aws iam get-role --role-name EXAMPLE_ROLE
```

**Never** embed access keys in images, `Dockerfile`, GitHub Actions YAML, EC2 user data or scripts. Prefer IAM roles / OIDC federation.

### 3.3 Encryption and security services

- **KMS:** Create/manage cryptographic keys and authorize cryptographic operations. Envelope encryption commonly uses a data key protected by a KMS key.
- **Secrets Manager:** Credentials, API keys, automatic rotation integration where supported; not just encrypted strings.
- **SSM Parameter Store:** Hierarchical configuration parameters, `SecureString` option, and application configuration. Feature set/pricing differ from Secrets Manager.
- **ACM:** TLS certificates for integrated AWS services (certificate availability and Region rules vary; CloudFront custom HTTPS certificates have specific Region requirements).
- **WAF:** Layer-7 web request filtering (SQL injection patterns, rate rules, etc.).
- **Shield:** DDoS protections; Shield Standard and Advanced differ.
- **GuardDuty:** Threat-detection findings from supported signal sources.
- **Security Hub:** Aggregates/normalizes security posture and findings.
- **Inspector:** Vulnerability/exposure scanning for supported workloads.
- **Macie:** Discovers sensitive data in S3.
- **Organizations:** Multi-account governance, SCPs and billing organization.
- **Control Tower:** Landing-zone governance and account setup controls.

### 3.4 Interview security checklist

- Root user protected; MFA and appropriate recovery controls.
- Human access via Identity Center/federation, short-lived sessions, scoped roles.
- Cross-account deployment roles, OIDC for CI, no standing secrets.
- Central CloudTrail / Config / security alerts with required coverage and retention.
- Encrypt data at rest and in transit; plan KMS key ownership and policies.
- Use private subnets, restrictive SGs, SSM Session Manager instead of open SSH when practical.
- IAM Access Analyzer, policy review, least privilege and rotation/remediation for legacy keys.
- Verify backup *restores*, incident runbooks, data retention and access reviews.

## 4. Compute — Amazon EC2

### 4.1 EC2 basics

**EC2** is on-demand virtual compute. Choose **AMI**, instance type, VPC/subnet, security group, storage, role, tenancy, purchase option and startup configuration.

- **AMI:** Template for boot volumes and OS/software configuration.
- **User data:** Startup bootstrap script or cloud-init content; design it to be idempotent and safe on restart/rebuild.
- **Instance profile:** Mechanism for an EC2 instance to receive role credentials.
- **IMDSv2:** Session-token-based instance metadata access; generally require it where supported.
- **ENI:** Virtual network interface; primary/private IPs, security groups.
- **Instance Store:** Physically attached ephemeral storage; contents not persistent across underlying instance failure or termination.
- **EBS:** Separately managed durable block volume (within its AZ), suitable for instance root and application disks.

**Instance families (examples):** `t` burstable; `m` general purpose; `c` compute optimized; `r` memory optimized; `i` storage optimized; `g/p` accelerator offerings. Check current families/Region support before answering sizing questions.

### 4.2 EC2 lifecycle and troubleshooting

Common states: pending → running → stopping/stopped → starting → running → shutting-down/terminated. A stop/start of an EBS-backed instance can change its automatically assigned public IPv4 address unless an Elastic IP or another stable endpoint is used. Instance-store-backed instances have different lifecycle restrictions.

| Symptom | Check | Typical fix |
|---|---|---|
| SSH times out | Routing, source IP, SG, NACL, host firewall, reachability | Fix route and scoped port 22 access / prefer SSM |
| SSH permission denied | Keypair, correct username, `authorized_keys`, OS policy | Use correct identity and key permissions |
| CPU 100% | CloudWatch, `top`, `ps`, load, bottleneck | Profile process, tune/scale |
| Disk full | `df -h`, `du`, volume, inode usage | Cleanup safely / grow filesystem and volume |
| EC2 status check failed | System vs instance check, console output | Restore OS/host, recover or replace appropriately |
| User data not run | cloud-init logs and permissions | Check script, shell, package mirrors, idempotency |
| SSM offline | Instance profile, agent, networking/endpoints | Restore agent/network/IAM paths |

### 4.3 EC2 pricing choices

- **On-Demand:** Flexible, no long commitment.
- **Savings Plans:** Discounted compute usage for eligible committed spend; evaluate terms and coverage.
- **Reserved Instances:** Capacity/billing benefits depending on RI type; distinctions matter.
- **Spot Instances:** Discounted spare capacity subject to interruption; suitable for tolerant/stateless/batch work.
- **Dedicated Hosts / Instances:** Isolation/licensing or regulatory needs.

**Scenario:** Jenkins ephemeral workers running noncritical builds can use Spot with retries and interruption handling. A database requiring steady availability should use a managed HA service or planned resilient architecture, not a lone Spot instance.

## 5. Load Balancing and Auto Scaling

### 5.1 ELB comparison

| Type | Layer | Key strength | Typical use |
|---|---|---|---|
| ALB | L7 | HTTP/HTTPS, host/path rules, WebSocket, target groups | APIs, websites, microservices |
| NLB | L4 | TCP/UDP/TLS, very high performance, static IP per enabled zone options | Databases/protocols, TCP services |
| GWLB | L3/L4 appliance forwarding | Integrates virtual network appliances | Firewalls / traffic inspection |
| CLB | Legacy L4/L7 | Older ELB generation | Maintain/migrate legacy deployments |

An **ALB listener** receives traffic; **rules** select **target groups**; **health checks** determine healthy targets. TLS may terminate at the load balancer or be passed/reencrypted depending on design.

### 5.2 Scaling concepts

- **Auto Scaling Group (ASG):** Maintains min/desired/max EC2 capacity, replaces unhealthy instances, supports multiple AZs.
- **Launch template:** Reusable configuration of AMI, instance type, user data, role and networking settings.
- **Target tracking:** Maintain target metric such as average CPU or ALB request count per target.
- **Step / simple scaling:** React to alarm thresholds with capacity changes.
- **Scheduled / predictive scaling:** Prepare for known or predicted demand.
- **Warm-up / health check grace:** Prevent premature replacement and noisy metrics.
- **Lifecycle hooks:** Perform custom operations on launch/termination.
- **Connection draining / deregistration delay:** Allow in-flight requests to finish.

**Request path:**

```text
Client -> Route 53 -> ALB (public subnets, multiple AZs)
                       |
                  Target Group
                       |
                 ASG in private subnets
                    |           |
                 EC2-A       EC2-B
                       |
                   RDS Multi-AZ
```

**Troubleshoot ALB 502 vs 503 vs 504:** 502 often invalid/closed backend response; 503 commonly no available targets for the request; 504 indicates timeout to target or response. Use ALB access logs, target health reason codes, application logs and metrics—these codes have multiple possible causes.

## 6. VPC Networking Deep Dive

### 6.1 Subnets, routing and CIDR

**VPC:** A logically isolated IP network in a Region. **Subnet:** A CIDR block in one AZ. **Route table:** Controls destination next hop. **Public subnet** has a route to an **Internet Gateway (IGW)**; **private subnet** has no direct IGW route. An instance must also meet public addressing and security requirements to receive inbound internet connections.

Sample layout:

```text
VPC: 10.20.0.0/16
├── AZ A
│   ├── Public A:  10.20.0.0/24   -> 0.0.0.0/0 via IGW
│   └── Private A: 10.20.10.0/24  -> 0.0.0.0/0 via NAT-A (optional)
└── AZ B
    ├── Public B:  10.20.1.0/24   -> 0.0.0.0/0 via IGW
    └── Private B: 10.20.11.0/24  -> 0.0.0.0/0 via NAT-B (optional)
```

- **IGW:** VPC internet edge; not a NAT replacement for private resources.
- **NAT Gateway:** Allows private IPv4 instances outbound connections to the internet and return traffic, without unsolicited inbound internet connections; an appropriately deployed NAT Gateway uses a public subnet for public NAT egress. Pricing and AZ failure domains matter. For IPv6, use suitable egress-only IGW/other architecture.
- **VPC endpoint:** Private route to supported AWS services. **Gateway endpoints** serve **S3 and DynamoDB** within supported routing scope; **interface endpoints** use private ENIs and AWS PrivateLink, with hourly/data processing costs.
- **Peering:** Connect VPCs; non-transitive, CIDRs must not overlap.
- **Transit Gateway:** Hub for many VPCs / hybrid networks; routing and attachment rules still matter.
- **Site-to-Site VPN:** Encrypted tunnels over public internet.
- **Direct Connect:** Dedicated physical network connectivity; **not encrypted by default**. Encrypt with VPN/MACsec as supported/required.

### 6.2 Security Group vs NACL

| Property | Security Group (SG) | Network ACL (NACL) |
|---|---|---|
| Scope | ENI / attached resource | Subnet |
| Stateful? | **Yes** | **No** |
| Allow / deny | Allow rules, default implicit deny | Ordered allow **and** deny rules |
| Rule evaluation | Set of allowed flows | Numbered rules, first match |
| Return traffic | Automatically allowed for permitted flow | Explicit return/ephemeral port rules needed |
| Common use | Primary workload firewall | Coarse subnet-level guardrail |

**CIDR:** `/24` is 256 total IPv4 addresses in ordinary CIDR math, but AWS reserves **five addresses in each VPC IPv4 subnet** (so 251 typically usable); design headroom for ENIs/pods and scaling.

**Private EC2 cannot read S3 with no internet:** Create and associate an **S3 Gateway VPC Endpoint** with appropriate private route tables; check IAM policy, bucket policy, endpoint policy, DNS and KMS permissions. It needs no internet/NAT path for that access. For different accounts and networks, review endpoint scope and routing limitations.

### 6.3 Packet troubleshooting order

1. Does DNS resolve to the intended private/public endpoint?
2. Is the destination listening? (`ss -lntp`, service logs)
3. Are source/destination addresses and ports correct?
4. Does each subnet route table have a valid path in both directions?
5. Do SGs allow the needed flow? Do NACLs allow request + return ports?
6. Is a NAT/IGW/VPC endpoint, TGW, VPN or peering route involved?
7. Is there a host firewall, proxy, TLS problem, MTU or app error?
8. Use **VPC Flow Logs**, **Reachability Analyzer**, load balancer logs, packet captures where permitted.

## 7. Route 53, CloudFront and Edge

### 7.1 Route 53

- **Public hosted zone:** Internet-facing authoritative DNS records.
- **Private hosted zone:** DNS names resolvable in associated VPCs (and configured hybrid resolvers).
- **Record types:** `A`, `AAAA`, `CNAME`, `MX`, `TXT`, `NS`, `SOA`; **Alias** is a Route 53 feature for supported AWS targets and apex names.
- **Routing policies:** Simple, weighted, latency-based, failover, geolocation, geoproximity, multi-value answer, IP-based (verify service-specific constraints).
- **TTL:** Caching lifetime affects propagation of ordinary DNS record changes; Route 53 alias behavior differs by target.
- **Health checks:** Can influence failover routing; full end-to-end resilience requires multiple healthy endpoints.

**Example:** `api.example.com` → alias to an ALB. Weighted records can gradually direct 10% of clients to a new deployment, but DNS-based shifting is not perfectly instantaneous or per-request deterministic.

### 7.2 CloudFront

CloudFront is a **content delivery network (CDN)** that caches and serves objects/content at edge locations. Common origins: S3 and ALB. Combine **S3 + CloudFront OAC** to keep S3 private while serving a public website; add ACM/TLS, caching policies, WAF and access logs as required.

**CloudFront vs Global Accelerator:** CloudFront primarily accelerates/caches HTTP(S) content; Global Accelerator provides static anycast IP entry points and AWS backbone routing for supported TCP/UDP and application endpoints. Choose based on protocol, failover and caching requirements.

## 8. Amazon S3 and Object Storage

### 8.1 Core concepts

- **Bucket:** Namespace/container; Region selection and naming rules apply.
- **Object:** Key, data, metadata, tags, version ID (if versioning on).
- **Consistency:** Strong read-after-write for S3 object PUT/GET/LIST/DELETE operations; do not repeat the outdated 'eventual consistency for overwrites' answer.
- **Default encryption:** New S3 objects are encrypted at rest by default using SSE-S3, unless a different supported setup is chosen.
- **Block Public Access:** Guardrail against accidental public exposure; use explicit policy/access designs.
- **Object Ownership:** Prefer bucket-owner-enforced/ACL-disabled where appropriate.
- **Versioning:** Recover from accidental overwrites/deletion (with delete markers). Versioning is not a substitute for immutable backup strategy.
- **Lifecycle:** Transition/archive/expire objects based on rules and minimum storage durations.
- **Replication:** Same-Region (SRR) or cross-Region (CRR), subject to configuration/permissions.
- **Pre-signed URL:** Time-limited permission to access an object using signer's authority, subject to underlying policies.
- **Object Lock:** Retention/legal hold for WORM-style requirements, with mode-specific behavior.

### 8.2 Storage classes

| Class | Use case | Trade-off |
|---|---|---|
| S3 Standard | Frequently accessed data | Higher storage rate than cold tiers |
| S3 Intelligent-Tiering | Access patterns change/unknown | Monitoring/automation charges and tier details |
| Standard-IA | Infrequently accessed, rapid retrieval | Retrieval charges and minimum durations/sizes |
| One Zone-IA | Re-creatable infrequent data | One AZ resilience model |
| Glacier Instant Retrieval | Archive with instant retrieval | Minimum duration / retrieval pricing |
| Glacier Flexible Retrieval | Archive with retrieval lead time | Restore delay and charges |
| Glacier Deep Archive | Lowest-cost long-term archive patterns | Long restore delay and constraints |

### 8.3 CLI operations

```bash
aws s3 ls
aws s3 ls s3://EXAMPLE-BUCKET/releases/
aws s3 cp ./artifact.zip s3://EXAMPLE-BUCKET/releases/artifact.zip
aws s3 sync ./reports/ s3://EXAMPLE-BUCKET/reports/ --dryrun
aws s3api head-object --bucket EXAMPLE-BUCKET --key releases/artifact.zip
aws s3api get-bucket-versioning --bucket EXAMPLE-BUCKET
aws s3api get-public-access-block --bucket EXAMPLE-BUCKET
```

**Troubleshoot S3 `AccessDenied`:** Identify caller via STS, verify action and resource ARNs, bucket policy, SCPs, permission boundary, endpoint policy, KMS encryption/permissions, VPC condition keys and object ownership. HTTP 403 is not synonymous with a missing identity-based Allow.

## 9. EBS, EFS and FSx

| Dimension | EBS | EFS | S3 |
|---|---|---|---|
| Interface | Block | NFS file | Object API |
| Typical attachment | EC2 / supported services | Multiple Linux clients | API over network |
| AZ scope | Volume in one AZ, snapshot copied/managed separately | Regional option across AZs, or One Zone variants | Regional storage class behavior varies |
| Typical use | Boot/root, local app DB disks | Shared Linux content and app files | Static assets, artifacts, backup, logs |
| Access | Device/filesystem | Mounted NFS filesystem | SDK/API/CLI |

- **EBS types:** `gp3` general purpose SSD; `io2` for demanding IOPS and latency use cases; HDD options exist for supported workloads. Choose based on IOPS, throughput, volume size and workload.
- **EBS snapshots:** Point-in-time, incremental block-level backups; verify application consistency when needed; test restore.
- **EBS Multi-Attach:** Only supported configurations and instance families; application must manage shared-write correctness.
- **EFS:** Shared file system using NFS; verify mount target per AZ, port 2049 and SGs, throughput/performance modes.
- **FSx:** Managed file systems such as Windows File Server, Lustre, NetApp ONTAP and OpenZFS for specialized protocols/features.

**Scenario:** Pod scheduled in another AZ cannot mount an EBS-backed Kubernetes PV because an EBS volume is AZ-scoped. Use appropriate scheduling/node affinity or choose EFS/another storage design for shared multi-AZ needs.

## 10. Databases and Caching

### 10.1 Service selection

| Service | Type | When to pick |
|---|---|---|
| RDS MySQL/PostgreSQL/etc. | Managed relational | SQL, transactions, familiar engines |
| Aurora | AWS-designed relational engine | Managed cluster features/performance/HA requirements |
| DynamoDB | Key-value/document NoSQL | High-scale predictable key-access workloads |
| ElastiCache | In-memory cache | Read latency reduction, caching, sessions |
| Redshift | Analytical warehouse | Columnar analytics at scale |
| OpenSearch Service | Search/analytics | Indexed search, log analytics |

### 10.2 Critical RDS distinctions

- **Single-AZ:** Limited protection against AZ outage.
- **Multi-AZ DB instance deployment:** One primary plus synchronous standby in another AZ; standby **does not serve reads**. Used for **failover**.
- **Multi-AZ DB cluster deployment:** Writer plus **two readable standby instances** across three AZs for supported engines/configurations. Also supports failover and read traffic.
- **Read replica:** Typically asynchronous replication; **read scaling**, can have lag; not the same as standby HA in a standard Multi-AZ DB instance deployment.
- **PITR:** Point-in-time restore within enabled backup retention; restored DB resources are new/alternate resources, not an automatic rollback on current DB.
- **RDS Proxy:** Connection pooling and improved failover behavior for supported RDS databases/workloads.

### 10.3 DynamoDB concepts

Partition key distributes items; optional sort key orders items with same partition key. Know **GSI**, **LSI**, on-demand vs provisioned capacity, autoscaling, consistency options, streams, TTL and hot partitions. Partition-key choice often matters more than adding raw provisioned capacity.

### 10.4 Troubleshooting RDS

1. Identify whether failures are network, auth, engine capacity, query regressions or failover.
2. Check database status/events, CloudWatch metrics (CPU, connections, free storage, latency), enhanced monitoring / Performance Insights alternatives as available.
3. Validate SG (source app SG → DB port), NACL/route/DNS, secrets and TLS.
4. Examine slow queries, locking, indexes, pool saturation and transaction duration.
5. Use read replicas for suitable read-heavy workload; scale vertically/horizontally where supported.
6. Test failover, reconnect behavior and backup restore.

---
## 11. Containers: ECR, ECS, EKS and Fargate

### 11.1 AWS container services

| Service | Meaning | You manage |
|---|---|---|
| ECR | Elastic Container Registry | Images, access, scanning configuration, lifecycle |
| ECS | AWS-native container orchestrator | Task/service definitions, scheduling settings, IAM, deployment |
| EKS | Managed Kubernetes control plane | Kubernetes resources, workloads, node choices, add-ons, security |
| Fargate | Serverless compute for ECS and supported EKS workloads | Task/Pod resource specifications and application |
| EC2 launch type | Containers on your EC2 fleet | Guest OS, patching, instance and orchestration capacity |

**ECS task definition:** Blueprint specifying image, CPU/memory, ports, logging, IAM roles and runtime settings. **ECS task:** Running instance of a task definition. **ECS service:** Maintains desired task count, integrates with target groups and deployment strategy.

**ECS task role vs execution role:** Task role grants the *application* permissions to access AWS services; execution role grants ECS/Fargate agent capabilities such as pulling ECR images, fetching selected secrets and shipping logs (depending on configuration).

### 11.2 ECR build/push example

```bash
# Replace placeholders, authenticate through an authorized AWS profile/role.
export AWS_REGION=ap-south-1
export ACCOUNT_ID=123456789012
export REPO=interview-demo
export IMAGE_TAG=v1

aws ecr describe-repositories --repository-names "$REPO" --region "$AWS_REGION"
aws ecr get-login-password --region "$AWS_REGION" |
  docker login --username AWS --password-stdin "$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"

docker build -t "$REPO:$IMAGE_TAG" .
docker tag "$REPO:$IMAGE_TAG" "$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO:$IMAGE_TAG"
docker push "$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO:$IMAGE_TAG"
```

Repository must exist beforehand; required IAM permissions, Docker installation, DNS/network access and relevant private-subnet endpoints are prerequisites. For reproducible releases, prefer **immutable image digests** over mutable `latest` tags.

### 11.3 EKS architecture

```text
                 AWS-managed EKS control plane
              +--------------------------------+
              | Kubernetes API / etcd /       |
              | scheduler / controller manager|
              +----------------+---------------+
                               |
                       Kubernetes API
                               |
          VPC worker nodes / managed node groups / Fargate*
          +---------------------------------------------+
          | kubelet | kube-proxy (if used) | VPC CNI    |
          | Pods: application / sidecars / agents       |
          +---------------------------------------------+

* Support differs by deployment mode; verify constraints.
```

- **Control plane:** Managed by AWS but still requires cluster configuration, version lifecycle, IAM/network choices and audit controls.
- **Data plane:** Managed node groups, self-managed EC2 nodes, EKS Auto Mode, or Fargate where supported.
- **VPC CNI:** Common EKS networking plugin integrating Pod addresses with VPC networking. IP allocation, ENI limits, subnet free addresses and routing are important.
- **CoreDNS:** In-cluster service discovery.
- **kube-proxy:** Service traffic implementation in configurations that use it.
- **EBS CSI driver:** Dynamic EBS PersistentVolume provisioning; zone constraints apply.
- **AWS Load Balancer Controller:** Manages AWS ALB/NLB resources for supported K8s Ingress/Service patterns. EKS Auto Mode offers some managed alternatives; know which controller owns a resource.
- **Cluster access:** EKS access entries / Kubernetes RBAC and IAM authentication; distinguish authentication from authorization.
- **Pod IAM:** **EKS Pod Identity** recommended for new supported workloads; **IRSA** remains valid, particularly where Pod Identity is unsupported. Avoid granting broad instance-role credentials to application Pods.
- **Scaling:** HPA scales Pods; Cluster Autoscaler/Karpenter/Auto Mode provision or adjust worker capacity by their mechanisms. HPA itself does not add EC2 nodes.

### 11.4 EKS operations

```bash
aws eks list-clusters --region ap-south-1
aws eks describe-cluster --name EXAMPLE-CLUSTER --region ap-south-1 \
  --query 'cluster.{status:status,version:version,endpoint:endpoint}'

aws eks update-kubeconfig --region ap-south-1 --name EXAMPLE-CLUSTER
kubectl get nodes -o wide
kubectl get pods -A
kubectl get events -A --sort-by=.metadata.creationTimestamp
kubectl describe pod EXAMPLE-POD -n default
kubectl logs deploy/EXAMPLE-DEPLOYMENT -n default --tail=100
kubectl rollout status deploy/EXAMPLE-DEPLOYMENT -n default
kubectl rollout undo deploy/EXAMPLE-DEPLOYMENT -n default
```

**EKS troubleshooting order:** Cluster health/endpoint access → nodes Ready and capacity → CNI IP/subnet availability → Pod scheduling/events → image pull and IAM → health probes → Service selectors/endpoints → load balancer target groups → app logs and telemetry.

**Typical mistakes:** Private-cluster endpoint unreachable from CI agent; Pod Identity association missing; ECR endpoint/NAT absent; subnet IP exhaustion; wrong node selectors/taints; security group mismatch; pending load balancer controller installation; HPA CPU metrics missing.

### 11.5 ECS vs EKS vs Lambda

- Choose **ECS** for a simpler AWS-native orchestration experience when Kubernetes compatibility is not needed.
- Choose **EKS** when Kubernetes portability/ecosystem, Helm, operators, K8s-native APIs and platform-standard tooling are important.
- Choose **Lambda** for event-driven execution and short-lived requests (subject to runtime, quotas and workload constraints).
- Choose **Fargate** for managed container compute when per-task/Pod constraints fit; trade off control, available features and cost.

## 12. Serverless and Event-Driven Architecture

### 12.1 AWS Lambda

A function executes on an event and uses **execution role** permissions. Design for short duration, stateless execution, idempotency, retries, concurrency controls and external persistence. Standard Lambda configuration has a **15-minute maximum timeout**, though some newer managed-instance configurations have different limits; verify workload-specific quota before using a numeric answer.

- **Cold start:** Startup latency when initializing an execution environment; mitigations depend on runtime, workload and provisioned concurrency options.
- **Concurrency:** Number of simultaneous invocations, affected by account/function settings and event source.
- **Timeout/memory:** CPU capacity generally increases with selected memory; tune based on cost/performance observations.
- **Failure handling:** Sync caller handles errors; async invocation/event-source integrations have distinct retry and DLQ/destination semantics.
- **VPC-attached Lambda:** Access to VPC resources does *not automatically* provide internet egress; configure NAT or relevant endpoints when required.

### 12.2 Integration comparison

| Service | Pattern | Key interview consideration |
|---|---|---|
| SQS | Pull-based work queue | Visibility timeout, DLQ, retries, duplicates |
| SNS | Pub/sub push fan-out | Subscription delivery/filtering |
| EventBridge | Event bus and rules | Matching/routing events across apps/accounts |
| Step Functions | Stateful orchestration | Workflow, retries, waits, parallel steps |
| API Gateway | Managed API entry point | Authentication, throttling, mappings, integrations |
| Kinesis | Streaming ingestion | Partition keys/shards and consumer throughput |

**Standard SQS queues** provide at-least-once delivery and best-effort ordering; consumer must be idempotent. **FIFO queues** offer ordering within a message group and deduplication semantics with specific constraints; do not treat this as universal end-to-end exactly-once processing.

**Visibility timeout:** A received message becomes temporarily invisible; if not deleted before timeout, it can be delivered again. Set timeout relative to actual processing and heartbeat/extension strategy.

**Example architecture:**

```text
ALB / API Gateway -> App / Lambda -> SQS queue -> Worker (ECS/EC2/Lambda)
                                             |
                                  failure retries / DLQ
                                             |
                                 Alert + redrive policy
```

**Design rule:** Handle retries with idempotency keys, bounded attempts, DLQ monitoring and explicit replay procedure.

## 13. Infrastructure as Code and Configuration Management

### 13.1 IaC choices

- **CloudFormation:** AWS-native declarative stack management (JSON/YAML), change sets and drift detection.
- **AWS CDK:** Define infrastructure using programming languages; synthesize CloudFormation templates.
- **Terraform:** Provider-based, multi-cloud declarative IaC using HCL; state management, plan/apply lifecycle and modules.
- **Ansible:** Agentless automation/configuration; useful for OS/app provisioning, not a substitute for lifecycle-managed cloud IaC.
- **Packer:** Reproducible golden images, often to reduce user-data runtime.

### 13.2 Terraform lifecycle

```text
Write HCL -> terraform fmt -> validate -> init -> plan -> review -> apply
                                       |
                              Remote state + locking
                                       |
                            CI policy/security controls
```

**Minimal Terraform example:**

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0" # Example constraint; verify compatibility
    }
  }
}

provider "aws" {
  region = var.aws_region
}

variable "aws_region" {
  type    = string
  default = "ap-south-1"
}

variable "bucket_name" {
  type = string  # Must be globally unique in relevant S3 namespace.
}

resource "aws_s3_bucket" "artifact_bucket" {
  bucket = var.bucket_name
  tags = {
    Project     = "aws-interview-lab"
    Environment = "dev"
  }
}

resource "aws_s3_bucket_public_access_block" "private" {
  bucket                  = aws_s3_bucket.artifact_bucket.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

output "bucket_name" {
  value = aws_s3_bucket.artifact_bucket.bucket
}
```

```bash
terraform fmt -check
terraform init
terraform validate
terraform plan -var='bucket_name=YOUR-GLOBALLY-UNIQUE-LAB-BUCKET'
# Apply only after review and with permission / budget guardrails:
# terraform apply -var='bucket_name=YOUR-GLOBALLY-UNIQUE-LAB-BUCKET'
# terraform destroy -var='bucket_name=YOUR-GLOBALLY-UNIQUE-LAB-BUCKET'
```

**Remote state best practices:** S3 backend with controlled IAM access, encryption and state locking supported by the installed Terraform version/backend; keep environments isolated; consider bucket versioning and secure backups. Never commit `terraform.tfstate`, credentials, or secret-bearing plans. Be cautious: `sensitive = true` redacts CLI output but does **not** automatically remove secrets from state.

**Drift:** Real infrastructure differs from the declared source/state. Detect via `terraform plan`/CloudFormation drift tooling and reconcile deliberately. Do not blindly import or overwrite manually changed production resources.

### 13.3 Ansible vs Terraform

| Terraform | Ansible |
|---|---|
| Desired cloud resource graph | Configuration and task automation |
| State-based plan/apply | Playbooks/modules; usually agentless SSH/WinRM |
| VPC, EKS, IAM, EC2 resources | Packages, configs, services, app setup |
| Declarative HCL | Mostly YAML tasks with idempotent modules |

**Good combined design:** Terraform provisions infrastructure → image/instance bootstrap or Ansible configures OS → Jenkins/GitOps deploy application → CloudWatch/Prometheus monitor.

## 14. AWS CI/CD and Deployment Strategies

### 14.1 AWS DevOps service map

| Stage | Common tools |
|---|---|
| SCM | GitHub, GitLab, CodeCommit (availability/onboarding rules vary), Gerrit |
| CI trigger/orchestration | Jenkins, GitHub Actions, GitLab CI, CodePipeline |
| Build/test | CodeBuild, Jenkins agents, Docker, Maven/Gradle |
| SAST/SCA | Coverity, Black Duck, SonarQube, dependency scanning |
| Image/artifact | ECR, S3, CodeArtifact, Nexus / Artifactory |
| Deploy | CodeDeploy, ECS, EKS, Argo CD, CloudFormation, Terraform |
| Release verification | CloudWatch alarms, synthetic checks, smoke tests, dashboards |

### 14.2 Jenkins + AWS (reference pattern)

```text
Git / Gerrit change -> Jenkins Pipeline
     |
     +-> lint, tests, SAST/SCA, secret scan
     +-> docker build -> immutable tag/digest -> ECR
     +-> infrastructure plan (Terraform) -> approved apply
     +-> deploy via ECS service update / Helm / Argo CD
     +-> readiness, smoke tests, alarms, metrics
     +-> promote / rollback
```

**Authentication:** Jenkins in AWS should use scoped EC2 instance-profile or container workload IAM roles; external CI may use OIDC federation to AWS STS. Avoid stored long-lived AWS access keys. Consider separation of `build` vs `deploy` roles and accounts.

**Illustrative Jenkinsfile (EKS deploy, placeholders):**

```groovy
pipeline {
  agent { label 'aws-cli-kubectl-helm-agent' }
  options { timestamps(); disableConcurrentBuilds() }
  environment {
    AWS_REGION = 'ap-south-1'
    EKS_CLUSTER = 'EXAMPLE-CLUSTER'
    NAMESPACE = 'demo'
    RELEASE = 'demo-app'
  }
  stages {
    stage('Verify Identity') {
      steps { sh 'aws sts get-caller-identity' }
    }
    stage('Test') {
      steps { sh 'make test' }
    }
    stage('Deploy') {
      when { branch 'main' }
      steps {
        // Agent identity must be pre-authorized in AWS IAM and EKS access/RBAC.
        sh '''
          set -eu
          aws eks update-kubeconfig --region "$AWS_REGION" --name "$EKS_CLUSTER"
          helm upgrade --install "$RELEASE" ./charts/demo \
            --namespace "$NAMESPACE" --create-namespace --wait --timeout 10m
          kubectl rollout status deployment/demo -n "$NAMESPACE" --timeout=5m
        '''
      }
    }
  }
  post {
    failure { echo 'Deployment failed: inspect pipeline, Kubernetes events and alarms.' }
  }
}
```

> Example assumes `make test`, Helm chart, permissions, agent tools and deployment name already exist; it's not a drop-in production pipeline. Production use should include pinned artifacts, input validation, approval/promotion, auditable change records, concurrency policy and safe rollback.

### 14.3 Deployment strategies

| Strategy | Process | Pros | Risks |
|---|---|---|---|
| Recreate | Stop old then start new | Simple | Downtime |
| Rolling | Replace subset gradually | Lower disruption | Old/new mixed versions |
| Blue/green | Switch between full environments | Rapid rollback | Double capacity / state compatibility |
| Canary | Shift small % to new version | Limits blast radius | Metrics and traffic controls needed |
| Feature flags | Separate release from feature exposure | Fine-grained release control | Flag lifecycle/complexity |

**Rollback is more than redeploying old code:** Schema/data migrations, external side effects, queues and compatibility can make rollback unsafe. Prefer backward-compatible migrations and progressive delivery.

### 14.4 EKS GitOps with Argo CD

1. CI builds and pushes image to ECR.
2. CI updates deployment manifest/image digest in Git (or uses a controlled image updater).
3. Argo CD observes the desired state repository.
4. Argo CD syncs Kubernetes resources and reports health/drift.
5. Alerts and rollout strategy validate production behavior.

**Important:** Separate CI's build permission from CD's cluster permission. Use signed/traceable artifacts, reviewable Git promotion and least-privilege cluster roles.

## 15. Observability: CloudWatch, CloudTrail and X-Ray

### 15.1 The three pillars of observability

- **Metrics:** Numeric time-series (CPU, p95 latency, error rate, queue depth, saturation).
- **Logs:** Events and context (structured logs preferred, correlation IDs, log retention controls).
- **Traces:** Path/timing across dependencies (OpenTelemetry, X-Ray integrations).

**Golden signals:** latency, traffic, errors, saturation. **USE:** Utilization, Saturation, Errors (for resources). **RED:** Rate, Errors, Duration (for services).

### 15.2 AWS tools

| Tool | Why it matters |
|---|---|
| CloudWatch Metrics | AWS service/custom metrics, dashboards |
| CloudWatch Logs | Collection, Logs Insights queries, retention/subscription |
| CloudWatch Alarms | Alert from metric/metric math and supported detectors |
| CloudWatch Synthetics | End-to-end synthetic endpoint checks |
| CloudWatch Container Insights | Container/platform telemetry for supported configurations |
| CloudTrail | Who called what supported AWS API, when and from where |
| AWS Config | What a resource configuration is/was and rule compliance |
| VPC Flow Logs | IP traffic metadata for supported network interfaces |
| X-Ray / OpenTelemetry | Distributed tracing/trace ingestion options |
| Prometheus + Grafana | Scraped metrics and dashboards; managed alternatives exist |

**CloudWatch vs CloudTrail:** CloudWatch observes workload/service behavior; CloudTrail records supported account-level API and activity events. CloudTrail does not replace application logs, and its event types/coverage/latency must be configured appropriately.

### 15.3 Log / alarm CLI examples

```bash
aws cloudwatch describe-alarms --alarm-name-prefix prod-
aws logs describe-log-groups --log-group-name-prefix /aws/eks/
aws cloudtrail describe-trails
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=RunInstances --max-results 10
```

**CloudWatch Logs Insights illustrative query:**

```sql
fields @timestamp, @message, @logStream
| filter @message like /ERROR|Exception|timeout/
| sort @timestamp desc
| limit 100
```

### 15.4 SLI, SLO and error budget

- **SLI:** Measured reliability indicator, such as percentage of successful requests.
- **SLO:** Internal target for an SLI, e.g., 99.9% successful requests in a 30-day period.
- **SLA:** External promise/contract, often with remedies.
- **Error budget:** Allowed unreliability before violating SLO; helps decide risk of changes.

**Illustration:** 99.9% availability over an exactly **30-day** window implies **43.2 minutes** of allowed unavailability, assuming a binary time-based metric (actual SLIs may use request counts). This is arithmetic, not an AWS SLA claim.

### 15.5 Alert design

- Alert on **customer impact** (5xx, latency, queue age) more than raw CPU alone.
- Include severity, owner, links to runbook/dashboards, recent deploy, blast radius.
- Use sustained thresholds and multi-window burn-rate alerting when appropriate.
- Avoid alert storms; deduplicate correlated alarms.
- Test alarms and routing; confirm they trigger a human response.

## 16. Security Operations and DevSecOps

**Shift left + runtime controls:**

```text
Commit -> secret scan -> SAST -> SCA/SBOM -> unit tests
      -> image scan -> artifact signing/provenance
      -> IaC policy check -> deploy approval
      -> WAF/SG/IAM/CloudTrail/GuardDuty runtime detection
      -> patch + incident response + evidence collection
```

### 16.1 Supply-chain questions

- **SAST (e.g., Coverity):** Finds source-code issues without executing the application.
- **SCA (e.g., Black Duck):** Inventories dependencies/licenses and identifies known component risks; distinct from SAST.
- **DAST:** Tests running apps externally for vulnerabilities.
- **Container scanning:** Checks OS/package/container image issues (coverage varies).
- **SBOM:** Bill of materials identifying components; useful for incident response and compliance.
- **Shift-left does not remove production security:** Runtime IAM least privilege, segmentation, patching and detection remain necessary.

### 16.2 IAM-related security incident response

If an access key appears compromised: contain and disable/rotate appropriately, inspect CloudTrail and related findings, identify affected resources, preserve evidence, revoke/expire sessions as supported, remove persistence, redeploy clean identities, close policy gaps and document incident. Coordinate with security; do not destroy evidence indiscriminately.

### 16.3 Common policies

- Deny insecure transport for S3 (`aws:SecureTransport` conditions with correct integration considerations).
- Deny unapproved Regions/actions via SCP (service scope and exception handling matter).
- Enforce EBS/RDS encryption with supported Config/SCP/organizational controls.
- Require IMDSv2 for EC2 launch templates and restrict metadata access.
- Limit security-group 0.0.0.0/0 exposure, especially SSH/RDP and databases.
- Restrict CI deploy permissions to an intended account, role, service and environment.

## 17. Backup, High Availability and Disaster Recovery

### 17.1 HA vs fault tolerance vs DR

- **High availability:** Reduce downtime through redundancy and automatic failover.
- **Fault tolerance:** Continue serving despite certain component failures without service interruption where feasible.
- **Disaster recovery (DR):** Restore service after a large incident, often including Regional loss or accidental data damage.

| DR strategy | General approach | Operational trade-off |
|---|---|---|
| Backup and restore | Rebuild infrastructure and restore backup | Lower steady cost, longer recovery |
| Pilot light | Minimal critical core running in secondary Region | Faster, more orchestration |
| Warm standby | Smaller live duplicate stack | Lower recovery time, higher standby cost |
| Multi-site active-active | Live across Regions | High complexity/cost; consistency and failover challenges |

- **RTO:** Maximum acceptable *time* to restore service.
- **RPO:** Maximum acceptable *data loss measured in time*.
- **Backup ≠ HA**; synchronous standby ≠ immutable backup; replication can replicate accidental deletes/corruption.
- **3-2-1 principle (adapt thoughtfully to cloud):** Multiple copies, diverse fault domains/media, at least one isolated/offsite copy when warranted.

### 17.2 Production DR plan

1. Document dependencies and recovery sequence (IAM, DNS, networking, DB, artifacts, cluster, monitoring).
2. Define RTO/RPO with business owners per service.
3. Select Multi-AZ, cross-Region replication and backups according to requirements.
4. Use infrastructure-as-code for rebuild, versioned/immutable artifacts and secure state.
5. Test restores/Region failover *regularly* and record measured recovery times.
6. Include database write cutover, stale DNS caches, secret/key availability, external integrations and reconciliation.
7. Plan failback and data consistency—not just failover.

## 18. Cost Optimization and FinOps

**Top cost drivers:** Oversized EC2/RDS; idle resources; EKS control planes; NAT Gateway hourly + data processing; inter-AZ/cross-Region transfer; load balancers/public IPv4; log retention and ingestion; EBS unattached volumes and snapshots; backup retention; egress; misconfigured autoscaling.

**Actionable review:**

- Use Cost Explorer, AWS Budgets, Cost Anomaly Detection, Cost and Usage Report (CUR)/Data Exports.
- Apply owner/environment/cost-center tags and allocation strategy.
- Rightsize based on observed p95 utilization and performance, not just average CPU.
- Cover steady usage using right-fit Savings Plans/RIs, not blindly every instance.
- Use Spot for retryable workloads with interruption handling.
- Scale nonproduction environments on schedules where safe.
- Reduce unnecessary NAT traffic using S3/DynamoDB gateway endpoints, private endpoints where economical.
- Apply S3 lifecycle policies; tune CloudWatch log retention/cardinality.
- Monitor cross-AZ data transfer, idle ALBs, EIPs/public IPv4, snapshots and stale images.
- Balance savings with reliability/SLO; a cheaper single AZ may be an unacceptable trade-off.

**Cost math:** `Monthly approx = instance hours × effective rate + EBS + transfer + LB + monitoring + other services`. Do not quote memorized hourly prices as timeless; pricing changes by Region, generation, purchase plan and traffic.

## 19. Architecture Patterns and Well-Architected

### 19.1 Three-tier application architecture

```text
                       Users
                         |
                    Route 53
                         |
                 CloudFront + WAF
                         |
                 Internet-facing ALB
                      (2 AZs)
                         |
                  App tier in ASG
                    Private Subnets
                         |
                   RDS Multi-AZ
                    DB Subnets
                         |
      S3 assets / SQS queue / CloudWatch / KMS
```

**Security flow:** Internet → ALB SG (443) → app SG (only ALB on app port) → DB SG (only app on DB port). DB and apps need no public IP. Use SSM, gateway/interface endpoints and controlled NAT as required.

### 19.2 Six pillars applied

| Pillar | AWS design question |
|---|---|
| Operational Excellence | Can we deploy, observe, troubleshoot, improve via runbooks? |
| Security | Who can access what? Are data/keys/network protected? |
| Reliability | What fails at host, AZ, Region, dependency levels? |
| Performance Efficiency | Does compute/database/cache scale with real load? |
| Cost Optimization | Are we paying only for justified capacity/features? |
| Sustainability | Can we reduce idle compute and unnecessary resource use? |

**Architect interview method:** First ask scale, latency, compliance, data size, consistency, budget, RTO/RPO and skills constraints. Then propose a default, enumerate trade-offs, explain scaling, observability and recovery.

## 20. Hybrid, Multi-Account and Migration

### 20.1 Landing zone and governance

- Use **AWS Organizations** to group accounts into organizational units, e.g., Security, Infrastructure, Sandbox, Development, Production.
- Centralize workforce identity, security logs, guardrails, billing and network services appropriately.
- Use **SCPs** to constrain capabilities in member accounts; they do **not** grant permissions.
- Create cross-account deployment roles that restrict principals, conditions and allowable environments.
- Treat accounts as blast-radius and billing boundaries, not just a naming convention.

### 20.2 Hybrid network design

| Tool | Use |
|---|---|
| Site-to-Site VPN | Encrypted tunnel from on-prem network to AWS |
| Direct Connect | Dedicated network connectivity; encryption is separate concern |
| Transit Gateway | Central routing hub for connected networks |
| Route 53 Resolver | Inbound/outbound hybrid DNS resolution rules |
| PrivateLink | Private endpoint service connectivity with supported architecture |

**Hybrid troubleshooting:** Check overlapping CIDRs, propagated/static routes, TGW attachments, VPN BGP, MTU, firewall/SG/NACL rules, DNS forwarders and return path. Use packet-flow reasoning—not just 'the VPN is green'.

### 20.3 Migration approaches

Common framework: **rehost, replatform, repurchase, refactor/re-architect, retire, retain, relocate** (terminology/models may vary). Typical tools include AWS Application Migration Service, DMS, DataSync, Transfer Family, Storage Gateway and Snow Family; choose based on downtime, data volume, bandwidth, change tolerance and application constraints.

## 21. Linux, Networking and Automation on AWS

### 21.1 OS commands useful on EC2

```bash
# Identity / OS
whoami && id && uname -a && cat /etc/os-release
# CPU / memory / disk
uptime; top -b -n 1 | head -30
free -h; df -h; df -i
lsblk; mount | head
# Networking
ip addr; ip route
ss -tulpn
getent hosts example.com
curl -I --connect-timeout 5 https://example.com
# Services and logs
systemctl --failed
journalctl -u EXAMPLE_SERVICE --since '30 minutes ago' --no-pager
# Container / host issues
docker ps -a
docker logs --tail 100 EXAMPLE_CONTAINER
```

For deeper network diagnosis: `dig`, `traceroute` or `tracepath`, `tcpdump` where permitted, `openssl s_client`, `curl -v`, VPC Flow Logs and Reachability Analyzer. Remember: not all AWS internal network paths reveal meaningful ICMP hops.

### 21.2 Python AWS automation (boto3)

```python
# pip install boto3
import boto3
from botocore.exceptions import ClientError

session = boto3.Session(region_name="ap-south-1")
ec2 = session.client("ec2")

try:
    for page in ec2.get_paginator("describe_instances").paginate(
        Filters=[{"Name": "instance-state-name", "Values": ["running"]}]
    ):
        for reservation in page.get("Reservations", []):
            for instance in reservation.get("Instances", []):
                print(instance["InstanceId"], instance["State"]["Name"])
except ClientError as error:
    print(f"AWS API request failed: {error.response['Error']['Code']}")
    raise
```

Use IAM roles/profile credentials through boto3's provider chain, pagination, exponential backoff/retries, timeouts, tagging, dry-run capability where supported, structured logs, idempotency and safe concurrency.

### 21.3 EC2 user data example (Ubuntu/Debian family)

```bash
#!/bin/bash
set -euxo pipefail
apt-get update
DEBIAN_FRONTEND=noninteractive apt-get install -y nginx
systemctl enable --now nginx
printf 'Hello from EC2\n' > /var/www/html/index.html
```

This requires a compatible image, networking/package repository access, suitable privileges and web SG rules. For production, prefer prebuilt images/config-management and avoid secrets in user data.

## 22. Production Design Walkthroughs

### 22.1 CI/CD build-and-release platform

**Requirements:** Multi-repo code integration, reproducible builds, static analysis, artifact publishing and controlled promotion.

```text
Git/Gerrit -> Jenkins controller -> disposable/managed workers
                        |
          +-------------+-------------+
          |             |             |
       Checkout       Compile       Coverity/SCA
          |             |             |
          +---------- Test ------------+
                        |
              Versioned immutable artifact
                        |
               ECR / S3 / Nexus
                        |
                Release approval
                        |
             ECS / EKS / EC2 rollout
                        |
                Smoke test + SLOs
```

**Interview discussion:** Worker isolation, credentials/OIDC, pipeline locks and unique build IDs, cache invalidation, supply-chain scan outcomes, artifacts/digests, rollbacks, observability, retention and queue times.

### 22.2 Production web application in two AZs

**Baseline:** Route 53 → ALB + WAF → ASG/ECS/EKS in private subnets → RDS Multi-AZ; S3 assets, Secrets Manager, CloudWatch, IAM roles, VPC endpoints/NAT. **Failure handling:** One instance/AZ loss → target health fails → distribute traffic to healthy capacity. Database failover may briefly interrupt connections, so retry/backoff and pooling are necessary.

### 22.3 Event-processing / async architecture

Producer → SQS → workers (Lambda/ECS) → database. Guard against poison messages with DLQ, retries, idempotency, backpressure, queue-age alarms and safe replay. Autoscale workers with backlog and processing-duration metrics.

### 22.4 Logging and observability platform

Applications emit structured logs → CloudWatch Logs / collectors → retention/forwarding → searchable store or log analytics → dashboards/alerts. Traces travel via OpenTelemetry and exporters. Restrict sensitive data, minimize excessive log cardinality and enforce access/retention.

## 23. Production Troubleshooting Playbook

**Universal incident response pattern:**

```text
Detect customer impact -> Assess severity and scope -> Mitigate safely
-> Check recent changes -> Form hypotheses -> Correlate metrics/logs/traces
-> Resolve root cause -> Validate SLOs -> Document follow-ups
```

**Keep operations safe:** Read first; avoid restarting/deleting infrastructure blindly. Communicate current impact, mitigation, owner and next checkpoint.

| Incident | First checks | Likely causes / next actions |
|---|---|---|
| 1. EC2 unreachable via SSH | Public/private IP, route table, SG 22, NACL, SSM | Wrong path/user/key, missing IGW/public IP, host firewall |
| 2. EC2 no internet | Private/public subnet, NAT/IGW route, DNS, SG/NACL | No outbound route, NAT failure, DNS/proxy |
| 3. Private EC2 → S3 fails | `sts get-caller-identity`, S3 endpoint, route tables | IAM/bucket/endpoint/KMS denial or missing endpoint |
| 4. ALB 502 | Target logs, backend listener and protocol, target health | Broken response, backend closes connection, TLS mismatch |
| 5. ALB 503 | Target group health, availability, deploy changes | No available registered healthy targets |
| 6. ALB 504 | Target processing times, app logs, timeouts | Backend slow/hanging, network failure |
| 7. ASG constantly replaces EC2 | Health check grace, boot logs, app readiness | Slow bootstrap, failed health check, failed user data |
| 8. CPU spike | CPU process metrics, traffic rate, deployment timeline | Traffic/load regression, runaway process, size mismatch |
| 9. S3 AccessDenied | Identity, exact action/ARN, bucket/endpoint/SCP/KMS | Missing allows, explicit deny, key permission or ownership |
| 10. RDS connection timeout | DNS, subnet, DB SG, route, DB health | Network unreachable, DB down, exhausted connections |
| 11. RDS CPU 95% | Queries, locks, pool, DB load, connection spikes | Bad query/index, contention, too-small instance |
| 12. Lambda timeout | Duration, downstream calls, VPC routing, memory | Slow dependency, missing NAT/endpoint, CPU shortage |
| 13. SQS queue growing | ApproximateAgeOfOldestMessage, worker errors | Underprovisioned workers, poison message, slow DB |
| 14. ECS tasks keep stopping | `describe-services`, stopped reason, task logs | Invalid image/secret, task CPU/memory, failed probe |
| 15. EKS Pods Pending | `describe pod`, events, nodes, quotas, AZ affinity | Insufficient CPU/IPs, taints, PVC constraints |
| 16. EKS ImagePullBackOff | ECR image/tag, IAM, auth, network endpoints | Repo authorization, missing image, ECR access |
| 17. EKS CrashLoopBackOff | `logs --previous`, events, env/config | App crash, missing secret, OOM, bad liveness probe |
| 18. EKS Pod cannot access S3 | Pod identity/IRSA, DNS, endpoint, KMS | Missing role association, network/permissions |
| 19. EKS Ingress not working | Controller logs, Ingress annotations, target groups | Controller IAM/network/SG or unhealthy targets |
| 20. EBS disk full | `df -h`, inode count, volume size, filesystem type | Logs/data growth, expand volume and filesystem |
| 21. EBS PVC Pending | PVC/storage class, CSI permissions, topology | Wrong storage class/zone or attach quota |
| 22. Terraform locked state | Lock owner/workspace/pipeline and active apply | Concurrent apply or stale lock; never force-unlock casually |
| 23. Unexpected high AWS bill | Cost Explorer, service/Region, usage type, tags | NAT/data egress, logs, orphaned resources, autoscale |
| 24. DNS cutover appears broken | `dig`, TTL, resolver caches, health, target status | Stale cache, wrong hosted zone/alias or backend failure |
| 25. CloudWatch alert without logs | Log group, agent permissions, subscription, timestamps | Shipping/retention/config failure; check source host |

### 23.1 Example: private EC2 cannot access S3 (no internet)

**Symptoms:** `aws s3 ls` times out or responds with `AccessDenied`. **Do not combine them.**

```bash
aws sts get-caller-identity
aws s3api head-bucket --bucket EXAMPLE-BUCKET
aws ec2 describe-vpc-endpoints --filters Name=vpc-id,Values=vpc-EXAMPLE
aws ec2 describe-route-tables --filters Name=vpc-id,Values=vpc-EXAMPLE
```

- **Timeout / DNS issue:** Need S3 gateway endpoint + correct associated route tables, supported DNS/Region path, NACL/SG egress.
- **403:** Route likely reached service; evaluate identity, bucket policy, SCP, endpoint policy, object/KMS constraints and specific action.
- **Fix:** Prefer gateway endpoint for same-VPC S3; for more complex hybrid/cross-network scenarios evaluate S3 interface endpoints and restrictions.

### 23.2 Example: deployment breaks production

1. Declare incident, identify affected service/Region/percentage and last known good version.
2. Compare deployments/config/feature flags, ALB 5xx/p95, Pod logs, database migrations.
3. If safe, stop rollout or route back to known-good version; check DB compatibility before rollback.
4. Validate recovery (error rate, latency, health checks) and customer experience.
5. Capture incident timeline, technical root cause, detection gaps and preventive controls.

## 24. Scenario-Based Interview Rounds

Use this **five-part structure**: clarify constraints → hypothesis/design → verification → mitigation → prevention.

### Scenario A — A private EC2 instance must access S3 without internet

**Answer:** Associate an **S3 gateway VPC endpoint** with the private subnet's route table; keep no default internet route if not needed. Check instance IAM role, bucket/endpoint policies, Region/DNS and KMS key permissions. This avoids requiring an IGW or NAT for supported S3 access.

### Scenario B — Design highly available APIs

**Answer:** ALB across at least two AZs, stateless app targets in Auto Scaling Group/ECS/EKS across AZs, private DB via suitable Multi-AZ mode, S3 for static data, IAM least privilege, health checks, CloudWatch alarms, capacity scaling and verified backups. Clarify SLA, failover behavior and cost.

### Scenario C — Deploy changes without downtime

**Answer:** Readiness/health checks, rolling or blue/green/canary release, automated tests, progressive traffic shift, observability-based verification and rollback. Make DB migrations backward compatible; verify draining and connection timeouts.

### Scenario D — Build a low-cost staging environment

**Answer:** Time-based shutdown, smallest right-sized resources, avoid always-on expensive NAT/LB when alternative is justified, restrictive budget alert, S3 lifecycle, limited log retention. Do not confuse lower cost with weakening essential security.

### Scenario E — CI pipeline assumes role across accounts

**Answer:** Configure a trusted role in target account, scope trust to expected external CI principal (OIDC claims/roles), limit permissions to deployment resources, assume via STS temporary credentials, audit through CloudTrail, avoid long-lived access keys.

### Scenario F — Kubernetes Pods stuck Pending

**Answer:** Inspect Pod events and nodes; check CPU/memory requests, taints/tolerations, node selectors/affinity, EBS PV AZ binding, subnet/IP capacity and autoscaler events. Fix the blocking constraint instead of just restarting Pods.

### Scenario G — Main DB is overloaded by reads

**Answer:** Profile slow queries and connections first, optimize indexes and pool; use supported read replicas/readable cluster readers for read-heavy traffic, cache frequently read data and scale DB appropriately. Explain replica lag and read-after-write constraints.

### Scenario H — Incident after a secret leaked to Git

**Answer:** Revoke/rotate compromised secret immediately, assess exposure and CloudTrail, inspect unauthorized activity, reissue using Secrets Manager/role-based access, eradicate from code/history according to procedures, add scanning and least-privilege controls.

### Scenario I — Streaming/queue processing cannot keep up

**Answer:** Look at backlog age, arrival rate vs throughput, worker concurrency, downstream rate limits, retry storms and poison messages. Scale workers with safe concurrency/backpressure, DLQ, idempotency and observability.

### Scenario J — Region outage response

**Answer:** Trigger documented DR decision, verify impact and integrity, fail over using tested cross-Region resources and DNS/traffic management, enforce single-writer strategy where required, measure RTO/RPO, monitor, and plan controlled failback.

---
## 25. Interview Questions and Model Answers

> **Practice aloud:** Give a 30–60-second answer, then add a relevant example and one trade-off. These are **model answers**—adapt them to work you actually performed; do not claim production experience you do not have.

### 25.1 AWS basics and architecture (Q1–Q15)

**Q1. What is AWS?**  
**A:** A cloud platform providing on-demand infrastructure and managed services, including compute, storage, networking, databases and application platforms, billed according to usage/service pricing models.

**Q2. What is the difference between Region and Availability Zone?**  
**A:** A Region is a geographic AWS deployment area; AZs are isolated failure domains within a Region. Multi-AZ improves within-Region resilience; it is not a multi-Region DR strategy.

**Q3. What is the shared responsibility model?**  
**A:** AWS secures cloud infrastructure; customers secure their workload configuration, identity, data and code. For EC2 you manage OS patches; for managed services AWS takes on more platform operations.

**Q4. IaaS vs PaaS vs serverless?**  
**A:** EC2 gives high OS control; RDS manages much of the database platform; Lambda runs event-driven code without fleet management. Choose based on workload fit and operational control.

**Q5. What are the six AWS Well-Architected pillars?**  
**A:** Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization and Sustainability.

**Q6. What is the difference between scaling vertically and horizontally?**  
**A:** Vertical scaling increases resources for a single machine/DB; horizontal scaling adds independent instances/tasks. Horizontal usually needs stateless services, distribution and shared state planning.

**Q7. What is high availability?**  
**A:** Redundant components, health checks and failover minimize downtime. Example: an ALB and stateless application fleet across two AZs with resilient database deployment.

**Q8. Is AWS automatically highly available?**  
**A:** No. Resilience depends on chosen service, configuration and fault domains. One EC2 instance in one AZ is still a single point of failure.

**Q9. What does elasticity mean?**  
**A:** Capacity expands/contracts as demand changes. Use ASGs, container scaling or serverless concurrency depending on service and SLO.

**Q10. What is an AWS account boundary used for?**  
**A:** Isolation of identity, billing, resources, security operations and failure blast radius; use separate production/nonproduction accounts when appropriate.

**Q11. Explain an ARN.**  
**A:** Amazon Resource Name is an identifier with service, partition/Region/account/resource parts as applicable; for example, `arn:aws:s3:::example-bucket`.

**Q12. What is an AWS service quota?**  
**A:** A limit governing usage or resource counts; some are adjustable, others hard limits. Inspect Service Quotas and regional/service docs instead of assuming identical limits across accounts.

**Q13. What is an AWS edge location?**  
**A:** Edge infrastructure used by services like CloudFront to deliver and route client traffic closer to users.

**Q14. Difference between durability and availability?**  
**A:** Durability is likelihood data remains intact; availability is whether service can be accessed when needed. Design backups and service failover for both.

**Q15. How do you choose an AWS service for a new application?**  
**A:** Identify scale, workload type, state, latency, security, team skills, compliance, RTO/RPO and budget, then compare managed service capabilities and operational trade-offs.

### 25.2 IAM, KMS and access control (Q16–Q35)

**Q16. IAM user vs IAM role?**  
**A:** User is a principal with potentially long-lived credentials; a role is assumed by an authorized principal for temporary credentials. Prefer roles/federated workforce access for workloads and people.

**Q17. What is STS?**  
**A:** AWS Security Token Service issues temporary credentials for supported role assumption/federation flows.

**Q18. Trust policy vs permissions policy?**  
**A:** Trust policy controls who may assume a role; permissions policy governs actions after assumption. Both must align with other authorization controls.

**Q19. What is least privilege?**  
**A:** Grant only actions, resources and conditions necessary to a task; test permissions, review usage, reduce broad wildcards and separate deploy responsibilities.

**Q20. What happens when an Allow and explicit Deny conflict?**  
**A:** Explicit Deny wins. Implicit Deny applies when there is no applicable permission to allow a request.

**Q21. What is an SCP?**  
**A:** Organizations-level guardrail limiting effective permissions for member-account principals. It does not independently grant access.

**Q22. What is a permissions boundary?**  
**A:** A maximum permission boundary for supported IAM users/roles. Identity policy and boundary interactions must be evaluated together.

**Q23. How does EC2 access S3 without AWS keys in code?**  
**A:** Attach an IAM role via instance profile and let the AWS SDK/CLI obtain temporary credentials through the instance metadata credential provider.

**Q24. What is IMDSv2?**  
**A:** Token-based EC2 instance metadata service protocol; requiring it reduces certain credential theft risks compared with permissive IMDSv1 setups.

**Q25. How does cross-account access work?**  
**A:** Create appropriately trusted IAM role in target account, grant restricted permissions, and have authorized source principal use STS AssumeRole; review SCP and resource policies.

**Q26. Explain AWS KMS.**  
**A:** Key management and cryptographic operations service; commonly protects data keys for envelope encryption and enforces key permissions through key/IAM policies and grants.

**Q27. KMS vs Secrets Manager?**  
**A:** KMS handles cryptographic keys/operations; Secrets Manager stores and retrieves secrets, integrates with KMS and supports rotation workflows.

**Q28. Parameter Store vs Secrets Manager?**  
**A:** Parameter Store manages application settings and SecureStrings; Secrets Manager specializes in secret lifecycle/rotation/integrations. Evaluate features, rotation needs and pricing.

**Q29. What is MFA and where should it be enforced?**  
**A:** Multi-factor authentication adds proof beyond a password. Protect root and workforce identities, and apply privileged-action policies/conditional access as appropriate.

**Q30. How does an EKS Pod get permission to S3?**  
**A:** Use EKS Pod Identity for supported new workloads or IRSA where appropriate, mapping a Kubernetes service account to scoped IAM permissions. Check VPC access separately.

**Q31. What is IRSA?**  
**A:** IAM Roles for Service Accounts uses Kubernetes service-account OIDC tokens and IAM trust to provide Pod-specific AWS permissions.

**Q32. What is EKS Pod Identity?**  
**A:** EKS-managed Pod-to-IAM-role association using supported node/agent infrastructure. It reduces per-cluster OIDC management, but verify support/constraints.

**Q33. Why might S3 still deny when the IAM role allows GetObject?**  
**A:** Bucket policy, endpoint policy, SCP, explicit Deny, wrong ARN, object access context or SSE-KMS key permissions can block it.

**Q34. How would you detect suspicious API usage?**  
**A:** Use CloudTrail activity records with GuardDuty/Security Hub findings and alarms; corroborate affected principals, IPs and resource changes.

**Q35. Should credentials be stored in Jenkins credentials?**  
**A:** Use temporary AWS roles/OIDC whenever possible; if legacy long-lived credentials are unavoidable, isolate, scope, audit and rotate them securely, never exposing secrets in logs.

### 25.3 EC2, scaling and load balancing (Q36–Q55)

**Q36. What is an AMI?**  
**A:** A template used to launch EC2 instances, including root volume template and launch metadata.

**Q37. What is EC2 user data?**  
**A:** Startup configuration script/cloud-init content often used for bootstrapping. It should be idempotent and should not contain credentials.

**Q38. What is an instance profile?**  
**A:** IAM container association enabling EC2 to use a role and obtain temporary AWS credentials.

**Q39. EBS vs instance store?**  
**A:** EBS is managed persistent block storage; instance store is ephemeral storage tied to the instance's physical host/lifecycle.

**Q40. What happens to the public IP after EC2 stop/start?**  
**A:** Automatically assigned public IPv4 can change; use an Elastic IP or stable DNS/load balancer endpoint if needed, accounting for IPv4 costs.

**Q41. What are EC2 status checks?**  
**A:** AWS checks underlying system reachability and instance-level connectivity, helping distinguish host infrastructure vs guest OS/network problems.

**Q42. On-Demand vs Spot?**  
**A:** On-Demand prioritizes flexibility with standard pricing; Spot uses spare capacity at discounts but can be interrupted and needs retries/checkpointing.

**Q43. What is an Auto Scaling Group?**  
**A:** A fleet manager that maintains desired EC2 capacity across chosen zones and replaces unhealthy instances according to policies.

**Q44. What is a launch template?**  
**A:** Versioned reusable EC2 launch settings for instance type, image, roles, storage and bootstrap.

**Q45. What are min, desired and max capacity?**  
**A:** Min is floor, desired is current target, max is ceiling. Policies adjust desired within limits and the group reconciles running instance count.

**Q46. What is target tracking scaling?**  
**A:** Automatically scales to keep a chosen metric near target, e.g., ASG average CPU or requests per target.

**Q47. ALB vs NLB?**  
**A:** ALB routes HTTP(S) based on Layer-7 rules; NLB distributes TCP/UDP/TLS flows with Layer-4 behavior and static IP options.

**Q48. What is an ALB target group?**  
**A:** Registered instances/IPs/Lambda targets and health-check settings for listener forwarding; supported target types depend on LB configuration.

**Q49. How do you investigate an unhealthy target?**  
**A:** Review health reason, path, response code, port, SG/NACL, backend listener, TLS, health-check timings and recent deploy.

**Q50. What is connection draining?**  
**A:** Delaying target removal so existing requests/connections can complete, with configured deregistration timeout.

**Q51. Why do all instances get replaced after deployment?**  
**A:** Often app fails health checks, wrong launch template, missing bootstrap dependencies or grace period too short; compare instance and ASG activity logs.

**Q52. Blue/green vs rolling deployment?**  
**A:** Blue/green switches between full environments for easy traffic reversal; rolling replaces subsets with lower extra capacity but mixed-version compatibility requirements.

**Q53. When use an Elastic IP?**  
**A:** When an authorized EC2/NAT design requires a stable public IPv4; prefer DNS/LBs where practical and account for pricing.

**Q54. Is an instance in a public subnet automatically reachable?**  
**A:** No. It needs public addressing, valid routing, SG/NACL allowance, host listener and any upstream policies.

**Q55. What are placement groups?**  
**A:** EC2 placement controls (cluster, spread, partition) to influence latency or correlated failure risk; availability constraints apply.

### 25.4 VPC, DNS and connectivity (Q56–Q80)

**Q56. What is a VPC?**  
**A:** A logically isolated virtual network with CIDR blocks, subnets, routing tables and network controls in a Region.

**Q57. What is a subnet?**  
**A:** A portion of VPC IP space restricted to one AZ. Routes and controls determine public/private access, not just subnet name.

**Q58. Public vs private subnet?**  
**A:** Public has a route to IGW; private does not have direct IGW route. Private egress can use NAT/endpoints.

**Q59. What does an IGW do?**  
**A:** Provides a VPC gateway for supported internet connectivity; instances still need public addresses and permissive end-to-end controls.

**Q60. What does a NAT Gateway do?**  
**A:** Enables private IPv4 workloads to initiate outbound connections using translated addresses without unsolicited public inbound access.

**Q61. Is NAT necessary for private EC2 to reach S3?**  
**A:** No; use an S3 gateway endpoint for suitable same-VPC access, with route table and policy configured correctly.

**Q62. What are gateway VPC endpoints for?**  
**A:** Private routing to Amazon S3 and DynamoDB from supported VPC context, with no gateway-endpoint hourly charge.

**Q63. What is an interface endpoint?**  
**A:** AWS PrivateLink endpoint backed by ENIs that lets VPC resources access supported services privately, with service-specific DNS, SG and billing considerations.

**Q64. SG vs NACL?**  
**A:** SG is stateful, associated with ENIs and supports allow rules. NACL is stateless, associated with subnets and has ordered allow/deny rules.

**Q65. Why do NACLs need ephemeral port rules?**  
**A:** Stateless NACLs evaluate return packets separately; responses often use high-numbered client-side ephemeral ports.

**Q66. What is a route table?**  
**A:** Destination-CIDR to target mapping; longest-prefix matching determines selected route among applicable routes, subject to AWS routing rules.

**Q67. What is CIDR?**  
**A:** Address and prefix notation, e.g., `10.20.0.0/16`. Smaller prefix number means larger address block.

**Q68. How many usable IPv4 addresses in an AWS /24 VPC subnet?**  
**A:** 251 under usual AWS VPC IPv4 reservations (256 total minus 5 reserved), absent additional provisioning constraints.

**Q69. What is VPC peering?**  
**A:** Private routing link between VPCs; does not provide transitive routing and does not support overlapping CIDRs.

**Q70. What is Transit Gateway?**  
**A:** A central routing hub connecting VPCs and hybrid networks; attachments, associations and route propagation control traffic.

**Q71. VPN vs Direct Connect?**  
**A:** VPN uses encrypted tunnels over internet; Direct Connect uses dedicated network connectivity but is not encrypted by default.

**Q72. What is Route 53?**  
**A:** AWS DNS and domain management service with record routing policies, hosted zones and health checks.

**Q73. What is Route 53 alias?**  
**A:** AWS-specific record functionality that points to supported AWS resource targets and can be used at zone apex where standard CNAME cannot.

**Q74. How does weighted DNS routing work?**  
**A:** Returns endpoints according to configured weights at DNS resolution time; caching means requests may not split exactly by percentage.

**Q75. Route 53 latency routing vs failover?**  
**A:** Latency policy answers based on measured routing latency; failover policy uses primary/secondary record health behavior.

**Q76. What is CloudFront?**  
**A:** CDN for content delivery via edge locations, caching and origin fetch, often fronting S3/ALB.

**Q77. CloudFront vs S3 website hosting?**  
**A:** CloudFront adds edge caching, HTTPS/options and WAF integration; a private S3 origin with OAC is a common secure architecture.

**Q78. What are VPC Flow Logs?**  
**A:** Metadata about selected IP traffic flows on supported VPC resources; useful for allow/reject analysis but not full packet payloads.

**Q79. How do you debug 'connection timed out'?**  
**A:** Verify DNS, destination listener, routes in both directions, SG, NACL, NAT/endpoints, host firewall and load balancer health; use Reachability Analyzer/Flow Logs.

**Q80. Does adding a route allow traffic automatically?**  
**A:** No. SG/NACL, target health, DNS, policy and return path must also permit it.

### 25.5 S3 and storage (Q81–Q97)

**Q81. What is S3?**  
**A:** AWS object storage accessed using bucket/object APIs, suitable for artifacts, backups, static assets and data lakes.

**Q82. Is S3 a normal file system?**  
**A:** No. It is object storage, not a POSIX block or shared file system; semantics differ for app locking, rename and random updates.

**Q83. What S3 consistency should you mention in modern interviews?**  
**A:** Strong read-after-write consistency for object PUT/GET/LIST/DELETE. Old claims of eventual overwrite consistency are outdated.

**Q84. Is S3 data encrypted by default?**  
**A:** Yes, S3 encrypts new objects by default using SSE-S3 unless another supported encryption option is configured.

**Q85. SSE-S3 vs SSE-KMS?**  
**A:** SSE-S3 uses S3-managed keys; SSE-KMS integrates KMS key policy, auditing and control, with separate KMS permissions/cost considerations.

**Q86. What is S3 versioning?**  
**A:** Retains versions of objects, enabling recovery after overwrite/delete markers; storage costs and lifecycle rules must be managed.

**Q87. What is S3 lifecycle policy?**  
**A:** Rules for object transition to classes, expiration and version management according to age/filters.

**Q88. What is S3 cross-Region replication?**  
**A:** Rule-based object replication to a bucket in another Region, subject to permissions/configuration and replication timing.

**Q89. What is a presigned URL?**  
**A:** Time-limited URL that permits a supported S3 operation under signing principal's permissions, within expiration and policy constraints.

**Q90. What is S3 Object Lock?**  
**A:** Retention/legal hold features for WORM-like requirements, used with supported bucket/versioning and selected retention modes.

**Q91. EBS vs S3?**  
**A:** EBS is block storage used like a disk; S3 stores immutable-versioned objects through APIs.

**Q92. EFS vs EBS?**  
**A:** EFS offers shared NFS access; EBS is AZ-scoped block storage typically attached to a compute instance at a time, with supported exceptions.

**Q93. How do EBS snapshots work?**  
**A:** Point-in-time backups stored via the EBS snapshot service, incremental after initial snapshot; restore creates a volume for attachment.

**Q94. Can EBS follow a Pod across AZs transparently?**  
**A:** No; EBS volume is zonal. Kubernetes scheduler/storage topology needs to place the consuming workload appropriately.

**Q95. How would you prevent public S3 exposure?**  
**A:** Block Public Access, explicit least-privilege policies, disable ACLs when suitable, access review, Config/security alerts and a private CloudFront OAC pattern for websites.

**Q96. Why is S3 AccessDenied happening with correct role?**  
**A:** Check object vs bucket actions, bucket and endpoint policy, SCP, permission boundary, KMS key, owner context and condition keys.

**Q97. What storage would you use for shared Linux uploads across multiple EC2 instances?**  
**A:** EFS if a shared NFS filesystem is needed; S3 is preferable for object-oriented application design.

### 25.6 Databases, caching and data access (Q98–Q113)

**Q98. What is Amazon RDS?**  
**A:** Managed relational database service that automates selected provisioning, backups, patching and availability operations for supported engines.

**Q99. RDS Multi-AZ vs read replica?**  
**A:** Standard Multi-AZ instance provides failover with a non-readable standby; read replicas provide read scaling and typically asynchronous replication. Multi-AZ DB clusters can have readable standby instances.

**Q100. What is RDS failover?**  
**A:** Managed switch to a standby/reader in supported HA deployments after infrastructure or engine events; app connections may break and need reconnection.

**Q101. What is a read replica lag?**  
**A:** Delay between changes on writer and visibility on replica; route consistency-sensitive reads to writer if needed.

**Q102. What is Aurora?**  
**A:** AWS-managed relational database engine family with distributed storage and cluster features compatible with supported MySQL/PostgreSQL modes.

**Q103. What is RDS Proxy?**  
**A:** Managed database connection pooling/proxy service supporting selected RDS engines and improved connection handling.

**Q104. RDS vs DynamoDB?**  
**A:** RDS is relational SQL with joins/transactions; DynamoDB is managed key-value/document NoSQL designed around access patterns and partition keys.

**Q105. What is a DynamoDB partition key?**  
**A:** Attribute hashed to distribute items/storage/workload; poor key choice can create hot partitions.

**Q106. DynamoDB on-demand vs provisioned?**  
**A:** On-demand adjusts billing to request activity; provisioned specifies throughput capacity and can use auto scaling.

**Q107. What is DynamoDB GSI?**  
**A:** Global secondary index enables alternate query keys with independent capacity/consistency considerations.

**Q108. What is ElastiCache?**  
**A:** Managed in-memory data store service used for caching/low-latency workloads; supported engines and deployment models evolve.

**Q109. Cache-aside pattern?**  
**A:** App checks cache, loads from DB on miss and populates cache; invalidate or set TTL carefully to prevent stale data.

**Q110. How do you resolve RDS connection exhaustion?**  
**A:** Check client pool, slow queries, idle leaks, connection limit, proxy options and DB sizing; do not only increase max connections.

**Q111. When should you add an index?**  
**A:** When query plans show beneficial selectivity/joins/sorts, balanced against write/storage costs.

**Q112. What is PITR?**  
**A:** Restore database to a selected recoverable point within configured backup window, usually producing a new resource; test end-to-end recovery.

**Q113. How do you migrate a DB with minimal downtime?**  
**A:** Assess compatibility and schema, use CDC/replication where suitable, dual-run validation, controlled cutover and rollback/failback plan.

### 25.7 ECS, EKS, containers and serverless (Q114–Q135)

**Q114. What is ECR?**  
**A:** Managed container image registry with IAM access, lifecycle and scanning capabilities.

**Q115. ECS vs EKS?**  
**A:** ECS is AWS-native orchestration; EKS is managed Kubernetes. Choose on ecosystem, portability, team skills and platform needs.

**Q116. What is Fargate?**  
**A:** Serverless compute capacity for supported ECS tasks and EKS Pods without managing EC2 fleet, with runtime/support trade-offs.

**Q117. What is an ECS task definition?**  
**A:** Container/runtime specification describing image, CPU, memory, network, IAM roles and logging configuration.

**Q118. ECS task role vs task execution role?**  
**A:** Task role is for application API access; execution role enables ECS runtime integrations like image pull and log delivery.

**Q119. What is EKS?**  
**A:** Managed Kubernetes control plane integrated with AWS identity, networking and compute options.

**Q120. Who manages worker nodes in EKS?**  
**A:** Depends on compute type: customer manages nodes in self-managed groups, AWS manages parts of node lifecycle in managed node groups/Auto Mode/Fargate.

**Q121. What is the AWS VPC CNI?**  
**A:** Kubernetes network plugin connecting Pod networking to VPC addresses; subnet/ENI IP capacity can constrain scheduling.

**Q122. How do you access an EKS cluster?**  
**A:** IAM-authenticated client using EKS endpoint and kubeconfig, authorized via EKS access management and Kubernetes RBAC where applicable.

**Q123. What is AWS Load Balancer Controller?**  
**A:** K8s controller that reconciles supported Ingress/Service resources into AWS ALB/NLB configurations.

**Q124. Why is an EKS Pod Pending?**  
**A:** Scheduling constraints, resource limits, missing nodes/IPs, taints, selectors, PVC topology or failed provisioning; inspect events first.

**Q125. Why is ImagePullBackOff happening?**  
**A:** Missing image/tag, registry authorization, DNS/private endpoint/NAT or pull-rate issue; inspect Pod events and runtime path.

**Q126. What is HPA?**  
**A:** Horizontal Pod Autoscaler adjusts Pod replica count based on metrics; it does not independently add EC2 nodes.

**Q127. Karpenter vs HPA?**  
**A:** HPA adjusts workload replicas; Karpenter provisions/sizes worker capacity to satisfy unschedulable workloads and consolidation goals.

**Q128. How should EKS Pods access AWS secrets?**  
**A:** Use a scoped workload identity and suitable Secrets Manager/CSI or application integration; control delivery, rotation, RBAC and logs.

**Q129. Lambda vs EC2?**  
**A:** Lambda handles event-driven functions without fleet management; EC2 provides full host/runtime flexibility for sustained or specialized workloads.

**Q130. What is a Lambda cold start?**  
**A:** Additional initialization latency when starting an execution environment; impact depends on runtime and configuration.

**Q131. What is Lambda reserved concurrency?**  
**A:** A function-specific limit/reservation of concurrent executions, useful for protecting downstream systems and allocating shared concurrency.

**Q132. Can Lambda in a VPC reach the internet automatically?**  
**A:** No; VPC-attached Lambda needs an appropriate NAT/endpoint/egress architecture for destinations it cannot reach privately.

**Q133. SQS vs SNS?**  
**A:** SQS queues work for consumers to pull and acknowledge; SNS pub/sub fan-out pushes notifications to configured subscribers.

**Q134. Standard vs FIFO SQS?**  
**A:** Standard emphasizes throughput with at-least-once, best-effort ordering; FIFO uses message-group ordering/dedup semantics with service limits.

**Q135. What is EventBridge used for?**  
**A:** Event matching and routing between AWS services/applications using an event bus and rules, supporting decoupled integrations.

### 25.8 IaC, CI/CD and deployment (Q136–Q153)

**Q136. What is IaC?**  
**A:** Infrastructure defined/reviewed as code for repeatable, auditable provisioning with automated validation.

**Q137. CloudFormation vs Terraform?**  
**A:** CloudFormation is AWS-native stack infrastructure management; Terraform manages provider-backed resources with plan/state and supports multi-cloud.

**Q138. Why does Terraform use state?**  
**A:** Maps managed resource identities and attributes to configuration, enabling change planning and dependency tracking.

**Q139. What is Terraform drift?**  
**A:** Real infrastructure diverges from code/state, often through console/manual edits; detect via plan and reconcile intentionally.

**Q140. Why use remote state locking?**  
**A:** Prevents concurrent writers corrupting/conflicting with infrastructure state; use supported backend/locking capabilities.

**Q141. How should Terraform secrets be protected?**  
**A:** Use role authentication and secure secret stores; restrict state/plan access, encrypt and audit state. Sensitive values can remain present in state.

**Q142. What is a Terraform module?**  
**A:** Reusable resource/configuration package with inputs and outputs, commonly versioned for network, compute and platform stacks.

**Q143. What is CodePipeline?**  
**A:** AWS managed pipeline orchestration for staged source, build, approval and deployment actions.

**Q144. What is CodeBuild?**  
**A:** Managed build/test environment executing steps described in build specification and project settings.

**Q145. What is CodeDeploy?**  
**A:** Managed deployment service supporting selected deployment patterns for EC2/on-premises, Lambda and ECS.

**Q146. Jenkins vs CodePipeline?**  
**A:** Jenkins offers extensible CI orchestration with controller/agent maintenance; CodePipeline is AWS-managed orchestration with supported native integrations.

**Q147. What is artifact immutability?**  
**A:** Build once, identify with immutable version/digest and promote same bytes through environments to reduce unreproducible deployments.

**Q148. What is a canary deployment?**  
**A:** Route a small population/percentage to new version, measure SLOs, then expand or roll back.

**Q149. How do you secure a CI/CD deployment to AWS?**  
**A:** Scoped temporary roles/OIDC, separated environments and approvals, short-lived workers, secret scanning, immutable artifacts, audit logging and minimal IAM permissions.

**Q150. How do you integrate SAST/SCA into a pipeline?**  
**A:** Run source scanning and dependency/license analysis in governed stages, publish versioned reports, enforce severity policy with reviewed exceptions and keep feedback fast.

**Q151. What is a deployment rollback?**  
**A:** Return code/traffic/config to safe previous state; requires compatibility with data schemas and stateful side effects.

**Q152. How do you structure Terraform environments?**  
**A:** Isolate prod/nonprod state/accounts/permissions, reuse versioned modules, protect applies and keep environment configuration reviewable.

**Q153. What is GitOps?**  
**A:** Git contains desired configuration, and a reconciler such as Argo CD continuously compares/applies it to the target environment with auditability.

### 25.9 Observability, SRE, security and cost (Q154–Q173)

**Q154. CloudWatch vs CloudTrail?**  
**A:** CloudWatch is metrics/logs/alarms; CloudTrail tracks supported AWS API and account activity events for auditing/investigation.

**Q155. CloudTrail vs AWS Config?**  
**A:** CloudTrail tracks supported calls/events; Config tracks resource configuration/state history and evaluates defined rules.

**Q156. What is an SLI?**  
**A:** A measurable signal of user-perceived service reliability, such as successful request rate or latency percentile.

**Q157. What is an SLO?**  
**A:** Target value for an SLI over a defined period, used to balance reliability and release velocity.

**Q158. What is an error budget?**  
**A:** Allowed unreliability during the SLO window; high burn rate indicates risk and may trigger slower release cadence.

**Q159. What are the four golden signals?**  
**A:** Latency, traffic, errors and saturation.

**Q160. What is MTTR?**  
**A:** Mean time to restore/repair/recover (define the team's exact convention). Reduce through detection, runbooks, controlled rollback and automation.

**Q161. What should a useful production alarm contain?**  
**A:** User-impact metric, severity, owner, SLO context, relevant dashboards, recent deploy link and actionable runbook.

**Q162. What are VPC Flow Logs used for?**  
**A:** Network-flow metadata for investigating allowed/rejected traffic patterns; cannot show application payload.

**Q163. How would you detect an exposed EC2 vulnerability?**  
**A:** Use Inspector scanning where supported, image/OS patching, Attack Surface reviews, security groups and central findings; prioritize by exploitability.

**Q164. GuardDuty vs WAF?**  
**A:** GuardDuty detects threats; WAF enforces filtering rules for supported HTTP(S) endpoints. They address different layers.

**Q165. What is a DLQ?**  
**A:** Dead-letter queue receives messages/events that failed defined retry policies, enabling monitoring, diagnosis and controlled redrive.

**Q166. What is RTO?**  
**A:** Maximum target recovery duration after disruption, agreed with business/service owner.

**Q167. What is RPO?**  
**A:** Maximum tolerable time window of data loss during disaster recovery.

**Q168. What is the difference between backup and replication?**  
**A:** Backup captures recoverable points; replication maintains another copy but may propagate deletes, corruptions or bad changes.

**Q169. How do you cost-optimize EC2?**  
**A:** Rightsize from observed utilization, use correct purchase options, scale with demand, remove idle instances and analyze data transfer/attached costs.

**Q170. What often causes unexpectedly high AWS networking cost?**  
**A:** NAT Gateway data processing, inter-AZ/cross-Region data transfer, internet egress, public IPv4 and long-lived load balancing, depending on usage.

**Q171. What is AWS Budgets?**  
**A:** Budgeting/alerting for spend or usage against configured thresholds; it is not an automatic spending hard cap.

**Q172. What is an immutable infrastructure approach?**  
**A:** Replace instances/images/deployments rather than hand-patching them to reduce configuration drift and improve reproducibility.

**Q173. How do you run a blameless postmortem?**  
**A:** Build factual incident timeline, quantify impact, identify technical/system causes and detection gaps, assign preventative actions and verify completion without blaming individuals.

### 25.10 Advanced and tricky questions (Q174–Q190)

**Q174. Can a security group explicitly deny traffic?**  
**A:** No, security group rules are allow rules. Other controls such as NACLs can explicitly deny traffic.

**Q175. Does S3 need a NAT Gateway to be reached from a private subnet?**  
**A:** Not when a correctly configured gateway endpoint provides the required S3 path; permissions/policies still apply.

**Q176. Is an RDS read replica a synchronous HA standby?**  
**A:** Normally no. Read replicas usually use asynchronous replication; a standard RDS Multi-AZ DB instance standby is separate and not readable.

**Q177. Will ASG guarantee zero-downtime deployment?**  
**A:** No. Correct health checks, safe rollout, sufficient capacity, draining and app compatibility are required.

**Q178. Does HPA automatically scale EKS EC2 nodes?**  
**A:** No; HPA scales Pods. Separate node provisioning/cluster autoscaling component is required where necessary.

**Q179. Does encrypting S3 guarantee access control?**  
**A:** No. Encryption protects data at rest; IAM, bucket policies, Block Public Access, endpoint and KMS key policies govern access.

**Q180. Are AWS Config and CloudTrail substitutes?**  
**A:** No. CloudTrail records API activity; Config records resource configurations and compliance evaluations.

**Q181. Is Direct Connect encrypted?**  
**A:** A Direct Connect circuit is not encrypted by default; choose encryption overlays/features as required.

**Q182. Is a subnet 'private' because EC2 has no public IP?**  
**A:** No; public/private subnet classification derives from its route to IGW; instance public addressing determines practical inbound internet reachability.

**Q183. Are AWS service managed credentials automatically inherited by all EKS Pods?**  
**A:** No. Configure workload-specific Pod Identity/IRSA, IAM trust/permissions and networking; avoid relying on worker role access.

**Q184. Are all Lambda invocations exactly once?**  
**A:** No. Delivery/retry behavior depends on event source/invocation type; functions should be idempotent where duplicate execution is possible.

**Q185. Can an S3 bucket policy alone override an Organizations explicit Deny?**  
**A:** No. Effective authorization evaluates applicable organization policies and explicit deny constraints.

**Q186. Is `terraform plan` always free of sensitive values?**  
**A:** No; plan/state can contain confidential data. Restrict artifacts, logs, access and retention; output redaction is not storage encryption.

**Q187. Does `aws sts get-caller-identity` prove an S3 action will succeed?**  
**A:** No. It identifies the current principal; a separate authorization evaluation controls `s3:GetObject`/other actions.

**Q188. Do you need Multi-Region for every workload?**  
**A:** No. Multi-Region adds complexity and expense; choose based on business resilience, geography, regulatory and consistency needs.

**Q189. Is a Multi-AZ database automatically protected from a malicious DELETE?**  
**A:** No. Data changes can replicate. Use backups, access controls, audit, PITR and deletion protection as appropriate.

**Q190. Why can a deployment look healthy while customers still see errors?**  
**A:** Health endpoints may test too little; dependencies, permissions, traffic, DNS caching or business flows can fail. Use synthetic tests, tracing and user-centric SLIs.

---
## 26. AWS vs Other Cloud / Common Comparisons

Cross-cloud mappings are **conceptual**, not exact feature equivalence. Always verify service capabilities, pricing, and integrations.

| Concept | AWS | Azure | Google Cloud |
|---|---|---|---|
| Virtual machine | EC2 | Azure Virtual Machines | Compute Engine |
| Virtual network | VPC | Virtual Network (VNet) | VPC |
| Object storage | S3 | Blob Storage | Cloud Storage |
| Managed Kubernetes | EKS | AKS | GKE |
| Container registry | ECR | Azure Container Registry | Artifact Registry |
| Serverless functions | Lambda | Azure Functions | Cloud Run functions |
| Managed relational DB | RDS / Aurora | Azure SQL / managed OSS DB | Cloud SQL / AlloyDB |
| DNS | Route 53 | Azure DNS | Cloud DNS |
| Cloud logging/monitoring | CloudWatch | Azure Monitor / Log Analytics | Cloud Monitoring / Logging |
| Access control | IAM | Azure RBAC / Entra ID | Cloud IAM |
| Key management | KMS | Key Vault | Cloud KMS |
| IaC-native | CloudFormation / CDK | ARM / Bicep | Infrastructure Manager / Terraform |

### Five high-value comparison prompts

1. **EC2 vs Lambda:** VM flexibility/long-running processes vs event-driven managed execution.
2. **ECS vs EKS:** AWS-native simplicity vs Kubernetes ecosystem/control abstractions.
3. **S3 vs EFS vs EBS:** Object vs file vs block interfaces.
4. **ALB vs NLB:** HTTP application routing vs transport-level flows.
5. **CloudWatch vs CloudTrail vs Config:** Service telemetry vs activity audit vs configuration history.

## 27. Common Misconceptions and Interview Traps

| Incorrect shortcut | Better interview answer |
|---|---|
| “S3 is eventually consistent.” | Modern S3 provides strong read-after-write for key object operations. |
| “Security groups block by deny rules.” | SGs are allow-only; NACLs have allow/deny. |
| “Private EC2 always needs NAT.” | Many AWS services can be reached via suitable VPC endpoints. |
| “Any Multi-AZ RDS standby can be queried.” | Standard Multi-AZ DB instance standby cannot; Multi-AZ DB cluster readers can. |
| “A read replica has zero lag.” | Replication lag is possible, especially for asynchronous replicas. |
| “An IAM Allow always grants permission.” | Other policies/explicit denies/key policies/conditions may restrict it. |
| “Cross-account S3 only needs one side's policy.” | Evaluate principal and resource policy requirements and context. |
| “SCP grants access.” | SCP only limits effective access; it cannot grant permissions. |
| “HPA scales the cluster.” | HPA scales Pods; node capacity requires separate scaling. |
| “Kubernetes service is reachable once a Pod is Ready.” | Check Service selector, endpoints, LB configuration, network and DNS. |
| “EC2 stop/start preserves public IP.” | Auto-assigned public IPv4 may change. |
| “Direct Connect means encrypted.” | Dedicated path is distinct from cryptographic encryption. |
| “CloudTrail = application logs.” | CloudTrail covers supported AWS activity, not arbitrary app logs. |
| “Backups are proof of recoverability.” | Only tested restores establish recovery confidence. |
| “S3 default encryption means public objects are secure.” | Encryption does not replace authorization/access controls. |
| “Spot is safe for any stateful job.” | Spot interruption needs retry/checkpoint and suitability analysis. |
| “Terraform sensitive values are absent from state.” | Sensitive display flag does not erase them from state. |
| “All production problems need a restart.” | First preserve evidence and isolate cause; avoid blind changes. |
| “Adding AZs means disaster recovery.” | Regional failures require additional DR design where justified. |
| “Larger instances always fix latency.” | First determine CPU, memory, I/O, network, DB query or lock bottleneck. |

## 28. DevOps/SRE Project Story Template

Interviews increasingly evaluate *evidence*: scope, ownership, decisions, difficulty and measurable outcomes. Structure answers using **STAR** (Situation, Task, Action, Result) and clearly separate production work from personal labs.

### Example answer template: AWS deployment automation

> **Situation:** Our multi-repository build/release workflow needed reliable CI, repeatable integrations, security checks and artifact promotion.  
> **Task:** Improve pipeline consistency, reduce manual steps and make failures diagnosable.  
> **Action:** Designed a Jenkins pipeline with controlled Git integration, reusable scripts, Dockerized build environments, stage-specific error handling, automated static analysis, versioned artifacts, role-based cloud access where applicable and monitoring/alerts.  
> **Result:** Report only the outcomes measured in your environment—e.g., deployment time saved, failed-run diagnostics, change turnaround, MTTR, or number of manual steps eliminated.  
> **Trade-offs:** Discuss worker capacity, queue time, cost, long-running builds, caches, compliance/report retention, recovery and least privilege.

### Interview project stories to prepare

| Story | What to demonstrate | Evidence to bring |
|---|---|---|
| Jenkins + artifacts | Pipeline-as-code, failure handling, reproducible releases | Sanitized pipeline stages, flow diagram, metrics |
| Kubernetes deployment | Helm, resource requests, services, ingress, probes | Manifests, rollout/rollback, failure diagnosis |
| Cloud infrastructure | VPC, IAM, EC2/EKS, Terraform modules | Plan/design, least-privilege roles, safe state |
| Observability | CloudWatch, Prometheus/Grafana, tracing | SLI dashboard, alert/runbook, incident outcome |
| DevSecOps | SAST/SCA, SBOM, reports, release gates | Scope/checker examples, risk acceptance policy |
| Incident response | Hypothesis, mitigation, follow-up | Timeline, cause, prevention and measured recovery |

**Do not overclaim:** If AWS was only a lab or POC, say so; describe production experience in your primary environment and explain how it transfers to AWS. Interviewers value precise ownership more than sweeping service lists.

### Three concise interview scripts

**“Tell me about your AWS experience.”**  
“I use AWS to design and practice secure cloud infrastructure and deployment workflows, focusing on EC2, VPC, IAM, S3, containers, CI/CD and observability. My broader DevOps strengths are automation, Linux, Jenkins, Kubernetes and production troubleshooting. I distinguish directly operated production services from hands-on labs and POCs.”

**“How do you design a secure CI/CD pipeline?”**  
“I separate build and deploy permissions, use temporary IAM roles, pin dependencies and artifacts, add tests/SAST/SCA, approve production promotions, deploy progressively and validate with health checks and error-rate/latency alarms. I make rollback and audit trails part of the design.”

**“Your deployment caused 500s—what next?”**  
“I assess blast radius and customer impact, correlate 5xx with deployment timing and target health, quickly stop or revert the rollout if safe, verify recovery and then use logs/metrics/traces to identify root cause and prevent repeat incidents.”

## 29. Interview Study Roadmap

### 14-day practical plan

| Day | Learning | Hands-on output | Interview questions |
|---|---|---|---|
| 1 | AWS basics, Region/AZ, shared responsibility | Architecture sketch | Q1–Q15 |
| 2 | IAM/STS/roles/trust/SCP | Example least-privilege S3 role policy | Q16–Q35 |
| 3 | EC2 + Linux operations | EC2 bootstrap/diagnostic runbook | Q36–Q46 |
| 4 | ELB, ASG and deployment | ALB request/health-check flow diagram | Q47–Q55 |
| 5 | VPC, endpoints, SG/NACL | Two-AZ VPC drawing | Q56–Q80 |
| 6 | S3/EBS/EFS | Storage decision tree, versioning lab | Q81–Q97 |
| 7 | RDS/Aurora/DynamoDB | HA vs replica design | Q98–Q113 |
| 8 | ECS/EKS/Fargate | Image build/push and K8s YAML | Q114–Q128 |
| 9 | Lambda/SQS/EventBridge | Retry/idempotency design | Q129–Q135 |
| 10 | Terraform / CloudFormation | Terraform `fmt`, `validate`, reviewed `plan` | Q136–Q142 |
| 11 | Jenkins/CodePipeline/GitOps | Pipeline sketch / staged deploy | Q143–Q153 |
| 12 | CloudWatch, SRE, security, FinOps | Alarm/runbook + budget design | Q154–Q173 |
| 13 | Debugging, DR and architecture | Rehearse 10 incidents and 2 designs | Q174–Q190 |
| 14 | Mock interviews | Record 90-second answers, fix weak topics | Entire set |

### Revision priority for DevOps / SRE / Platform Engineer

**Must know:** IAM roles, EC2, VPC, SG/NACL, NAT, endpoints, ALB/ASG, S3, RDS HA, CloudWatch, CloudTrail, Linux troubleshooting, Terraform, CI/CD, containers/EKS, backup and cost.  
**Next level:** Multi-account/STS, Transit Gateway, Pod Identity/IRSA, CloudFront/WAF, SQS, KMS, RTO/RPO, error budgets, SLOs, progressive delivery.  
**Advanced:** Multi-Region replication/failover, large-scale observability, IaC governance, workload identity security and incident response at scale.

## 30. Hands-on Labs

> **Before starting:** Select an authorized sandbox AWS account and budget, verify Region, use a role or profile, set tags, and understand costs. Examples contain placeholders; never run destructive operations against production. For all stateful resources, create *and test* a cleanup plan.

### Lab 1 — AWS CLI identity and inventory (read-only)

**Goal:** Confirm profile/Region and identify live infrastructure without changes.

```bash
aws --version
aws configure list
aws sts get-caller-identity
aws ec2 describe-vpcs --region ap-south-1 --query 'Vpcs[*].[VpcId,CidrBlock]' --output table
aws ec2 describe-instances --region ap-south-1 \
  --query 'Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType]' --output table
aws s3 ls
```

**Interview takeaway:** Credential provider chain, account boundaries, Region scoping, safe discovery and no key disclosure.

### Lab 2 — Design a two-AZ VPC on paper

**Requirements:** `10.20.0.0/16` VPC; two public subnets and two private app subnets; ALB in public, app in private, DB private, optional NAT per AZ and S3 gateway endpoint.

```text
                        INTERNET
                            |
                           IGW
                            |
                   Public ALB (AZ-A/B)
                            |
        +-------------------+-------------------+
        |                                       |
   Private App-A                          Private App-B
        |                                       |
        +------------- RDS DB ------------------+
                            |
                  S3 Gateway Endpoint
```

**Task:** Write security group rules (ALB 443 from trusted clients; app port from ALB SG; DB port from app SG), route tables, DNS path, NAT/endpoint decisions, cross-AZ failure handling and cost trade-offs. **Validation:** Explain each packet's outward and return routes.

### Lab 3 — Secure S3 bucket configuration (concept + commands)

A production-grade bucket should have explicit ownership/access, Block Public Access, encryption configuration, versioning/retention according to use, audit and controlled IAM. Begin with **read-only** checks:

```bash
aws s3api get-public-access-block --bucket EXAMPLE-BUCKET
aws s3api get-bucket-encryption --bucket EXAMPLE-BUCKET
aws s3api get-bucket-versioning --bucket EXAMPLE-BUCKET
aws s3api get-bucket-policy-status --bucket EXAMPLE-BUCKET
```

**Exercise:** Write a least-privilege policy for `releases/*` with `ListBucket` limited to the prefix and `GetObject` on corresponding object ARN. Note permission requirements for SSE-KMS. **Validation:** Confirm access for intended role and denial for unintended role with controlled test identities.

### Lab 4 — EC2 Nginx deployment (chargeable; guided)

1. Create a security-restricted launch template and attach a minimal EC2 IAM role if needed.
2. Launch in a permitted subnet with SSM connectivity; do not expose SSH to the world.
3. Use the user-data script in [Section 21.3](#213-ec2-user-data-example-ubuntudebian-family) or bake an AMI.
4. Verify service from appropriate client/network: `curl -I http://PRIVATE_IP/` and `systemctl status nginx`.
5. Check CloudWatch CPU metrics and OS logs; simulate app service stop *only in lab* and observe detection.
6. Terminate test instance and remove unneeded volumes, public IPv4 allocations and log resources.

**Interview takeaway:** Bootstrap, cloud-init logs, IAM instance profile, network routing and troubleshooting.

### Lab 5 — ALB + ASG architecture and debugging

1. Build target group/health-check architecture for Nginx app instances in two AZs.
2. Configure an ALB listener, an ASG with launch template and appropriate SG references.
3. Demonstrate target health transitions, warm-up and deregistration behavior.
4. Practice read-only checks:

```bash
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names EXAMPLE-ASG
aws elbv2 describe-load-balancers --names EXAMPLE-ALB
aws elbv2 describe-target-health --target-group-arn arn:aws:elasticloadbalancing:REGION:ACCOUNT:targetgroup/EXAMPLE/ID
```

**Cleanup:** Remove test ASG/LB/target groups and verify no unneeded EIPs, volumes or log groups remain.

### Lab 6 — Push a Docker image to ECR

Prerequisites: Docker, IAM permissions and existing ECR repository. Follow [Section 11.2](#112-ecr-buildpush-example). Add immutable tags and scan findings as an exercise. **Validation:** Describe ECR repo, image digest and tag mapping; practice how image pull works from private ECS/EKS subnet and why endpoints/NAT may be needed.

### Lab 7 — EKS Pod diagnosis (use existing authorized cluster)

```bash
aws eks update-kubeconfig --name EXAMPLE-CLUSTER --region ap-south-1
kubectl get nodes -o wide
kubectl get pods -A
kubectl describe pod EXAMPLE-POD -n default
kubectl get events -A --sort-by=.metadata.creationTimestamp | tail -40
kubectl top nodes    # Requires metrics pipeline
kubectl logs EXAMPLE-POD -n default --previous # When previous container exists
```

**Exercise:** Diagnose one Pending/CrashLoopBackOff/ImagePullBackOff example. Identify whether failure is scheduling, app startup, workload identity, registry path, probes or storage. Keep cluster changes explicitly authorized.

### Lab 8 — Terraform S3 bucket with safe review

1. Save [Section 13.2](#132-terraform-lifecycle) into an isolated lab directory as `main.tf`.
2. Configure AWS role/profile and a unique test bucket name.
3. Run `terraform fmt -check`, `terraform init`, `terraform validate`, and `terraform plan`.
4. Inspect every change, IAM permission and cost. Apply only if approved.
5. Inspect drift by reviewing outputs/state commands without exposing sensitive state.
6. Clean up via `terraform destroy` after reviewing what will be removed; verify resource deletion and retention needs.

**Interview takeaway:** Version constraints, dependency graph, public access controls, state security and locking.

### Lab 9 — Serverless queue processor design

**Design only or use a sandbox:** API Gateway → Lambda producer → SQS → Lambda/ECS worker → DynamoDB/RDS, with DLQ. Specify:

- Idempotency key and safe retry behavior.
- Visibility timeout and maximum receive count.
- Queue-age alarm, DLQ alert and redrive runbook.
- Workload role and network controls.
- Backpressure and downstream capacity limit.

**Interview takeaway:** At-least-once delivery, retries, poison-message handling and observable async processing.

### Lab 10 — CloudWatch logs and alerting

**Exercise:** Choose a service with an error-rate/latency metric and create a design for an alarm and incident response route. Write a Logs Insights query to find error messages; choose retention and access controls. Demonstrate read-only inspection of alarms and log groups. Discuss false positives and sustained-threshold alerts.

### Lab 11 — RDS high-availability design (paper lab)

**Design:** Relational app needing high availability and read scaling. Compare standard RDS Multi-AZ DB instance, Multi-AZ DB cluster (supported engines), read replicas, and Aurora. Define endpoint usage, connection recovery, read replica lag, backup/PITR and failover testing. **Expected answer:** Multi-AZ mode is selected for availability; read scaling needs readable replicas/cluster readers, not non-readable standard standby.

### Lab 12 — End-to-end capstone: secure web API delivery

**Requirements:** Deploy an API with predictable releases, private network, resilience across AZs, database state, observability and a documented rollback.

```text
Git -> CI tests/SAST/SCA -> Build OCI image -> ECR
  -> Approval -> Helm/Argo CD -> EKS or ECS service
                                   |
Route 53 -> CloudFront/WAF (optional) -> ALB -> API Pods/Tasks
                                   |
                              RDS + S3 + SQS
                                   |
                 CloudWatch/Prometheus/Tracing
```

**Deliverables:**

- `architecture.md`: subnets, CIDRs, routing, trust boundaries and DR assumptions.
- `terraform/`: validated and reviewed IaC, remotely managed state design.
- `app/`: simple health-enabled API and Dockerfile; non-root container.
- `helm/` (EKS) or ECS task/service spec: probes, CPU/memory, resource policy.
- `.github/workflows/` or `Jenkinsfile`: pinned build, scans, digest promotion, deploy verification.
- `runbooks/`: ALB unhealthy target, Pod Pending, DB timeout, S3 AccessDenied and rollback.
- `observability.md`: p95 latency, 5xx, saturation, queue depth, key dashboards, alerts and SLOs.
- `cost.md`: approximate costs, limits, budget alarm and cleanup.

**Pass criteria:** Can explain every request's network path, IAM permission boundary, critical failure mode, recoverability and safe rollback. Do not provision the entire architecture just for interview practice without a budget and teardown plan.

## 31. AWS CLI Cheat Sheet

> All commands operate with your current AWS credentials. Replace placeholder identifiers. Commands below are predominantly **read-only**; commands shown in other sections for writes/deploys require intentional authorization. Prefer `--profile`, `--region`, `--query` and `--output table/json` appropriately.

### Identity & account

```bash
aws --version
aws configure list
aws sts get-caller-identity
aws iam list-roles --max-items 10
aws iam get-role --role-name EXAMPLE_ROLE
aws organizations describe-organization  # If authorized / organization-enabled
```

### EC2, VPC and routing

```bash
aws ec2 describe-regions --all-regions
aws ec2 describe-availability-zones --region ap-south-1
aws ec2 describe-instances --region ap-south-1
aws ec2 describe-instance-status --include-all-instances --region ap-south-1
aws ec2 describe-volumes --region ap-south-1
aws ec2 describe-snapshots --owner-ids self --region ap-south-1
aws ec2 describe-vpcs --region ap-south-1
aws ec2 describe-subnets --region ap-south-1
aws ec2 describe-route-tables --region ap-south-1
aws ec2 describe-security-groups --region ap-south-1
aws ec2 describe-network-acls --region ap-south-1
aws ec2 describe-nat-gateways --region ap-south-1
aws ec2 describe-vpc-endpoints --region ap-south-1
aws ec2 describe-network-interfaces --region ap-south-1
```

### ALB, ASG and CloudWatch

```bash
aws elbv2 describe-load-balancers --region ap-south-1
aws elbv2 describe-target-groups --region ap-south-1
aws elbv2 describe-target-health --target-group-arn TARGET_GROUP_ARN
aws autoscaling describe-auto-scaling-groups --region ap-south-1
aws autoscaling describe-scaling-activities --auto-scaling-group-name EXAMPLE-ASG
aws cloudwatch list-metrics --namespace AWS/EC2 --region ap-south-1
aws cloudwatch describe-alarms --region ap-south-1
```

### S3 and storage

```bash
aws s3 ls
aws s3 ls s3://EXAMPLE-BUCKET/
aws s3api head-bucket --bucket EXAMPLE-BUCKET
aws s3api get-bucket-location --bucket EXAMPLE-BUCKET
aws s3api get-bucket-versioning --bucket EXAMPLE-BUCKET
aws s3api get-bucket-encryption --bucket EXAMPLE-BUCKET
aws s3api get-public-access-block --bucket EXAMPLE-BUCKET
aws s3api list-object-versions --bucket EXAMPLE-BUCKET --prefix releases/ --max-items 20
aws efs describe-file-systems --region ap-south-1
```

### RDS and DynamoDB

```bash
aws rds describe-db-instances --region ap-south-1
aws rds describe-db-clusters --region ap-south-1
aws rds describe-events --source-type db-instance --region ap-south-1
aws dynamodb list-tables --region ap-south-1
aws dynamodb describe-table --table-name EXAMPLE_TABLE --region ap-south-1
```

### ECS, EKS and ECR

```bash
aws ecr describe-repositories --region ap-south-1
aws ecr describe-images --repository-name EXAMPLE_REPO --region ap-south-1
aws ecs list-clusters --region ap-south-1
aws ecs describe-clusters --clusters EXAMPLE_ECS_CLUSTER --region ap-south-1
aws ecs list-services --cluster EXAMPLE_ECS_CLUSTER --region ap-south-1
aws ecs describe-services --cluster EXAMPLE_ECS_CLUSTER --services EXAMPLE_SERVICE --region ap-south-1
aws eks list-clusters --region ap-south-1
aws eks describe-cluster --name EXAMPLE_EKS_CLUSTER --region ap-south-1
aws eks list-nodegroups --cluster-name EXAMPLE_EKS_CLUSTER --region ap-south-1
```

### Lambda, SQS and security

```bash
aws lambda list-functions --region ap-south-1
aws lambda get-function-configuration --function-name EXAMPLE_FUNCTION --region ap-south-1
aws sqs list-queues --region ap-south-1
aws kms list-keys --region ap-south-1
aws secretsmanager list-secrets --region ap-south-1
aws cloudtrail describe-trails --region ap-south-1
aws guardduty list-detectors --region ap-south-1
aws configservice describe-configuration-recorders --region ap-south-1
```

### AWS CLI tips

- `aws help` and `aws <service> <operation> help` provide current command syntax.
- `--query '...'` uses **JMESPath** filtering and formatting client-side.
- `--output table` for readability; `--output json` for scripting.
- Use pagination APIs in scripts. `--no-paginate` can silently limit results to one page.
- `aws sts get-caller-identity` helps diagnose wrong profile/account.
- Prefer IAM roles, temporary credentials and region-specific configuration.
- Do not print environment variables containing secrets in logs.

## 32. Flashcards and Rapid-Fire Recap

| Prompt | Memorize |
|---|---|
| Stateful vs stateless filtering? | SG stateful, NACL stateless |
| S3 private path without NAT? | Gateway endpoint |
| IAM explicit Deny? | Overrides Allow |
| Human access across accounts? | IAM Identity Center / federation |
| EC2 application credentials? | Instance profile + IAM role |
| EKS Pod AWS IAM? | Pod Identity / IRSA |
| Cross-account temporary credentials? | STS AssumeRole |
| HTTP routing? | ALB |
| TCP/UDP routing? | NLB |
| Auto scale EC2? | ASG |
| Auto scale Kubernetes Pods? | HPA |
| Auto scale EKS nodes? | Karpenter / supported cluster autoscaler / Auto Mode |
| Static content CDN? | CloudFront |
| AWS object storage? | S3 |
| Shared Linux NFS? | EFS |
| EC2 disk? | EBS |
| RDS HA? | Multi-AZ mode |
| RDS read scale? | Read replicas / readable cluster readers |
| Fully managed key-value DB? | DynamoDB |
| AWS API audit? | CloudTrail |
| Metrics/logs/alarms? | CloudWatch |
| AWS resource compliance history? | AWS Config |
| Threat detection? | GuardDuty |
| Secrets with rotation integrations? | Secrets Manager |
| Key management? | KMS |
| Event queue? | SQS |
| Pub/sub? | SNS |
| Event bus? | EventBridge |
| Workflow orchestration? | Step Functions |
| Native AWS infrastructure template? | CloudFormation |
| Multi-provider IaC? | Terraform |
| AWS-native CI/CD? | CodePipeline + CodeBuild + CodeDeploy |
| Container registry? | ECR |
| AWS-native containers? | ECS |
| Managed Kubernetes? | EKS |
| Serverless container compute? | Fargate |
| Recovery time target? | RTO |
| Data loss window target? | RPO |
| SRE golden signals? | Latency, traffic, errors, saturation |

## 33. Self-Assessment Checklist

Rate each section: **0 = unfamiliar**, **1 = theory only**, **2 = can operate/debug**, **3 = can design and explain trade-offs**.

- [ ] Describe shared responsibility, Regions/AZs and the six Well-Architected pillars.
- [ ] Explain IAM trust, policies, SCP, STS, role sessions and least privilege.
- [ ] Design a two-AZ VPC including IGW, NAT, private routes and endpoint alternatives.
- [ ] Debug SG vs NACL, DNS, TLS, 502/503/504 and S3 AccessDenied.
- [ ] Choose EC2 instance types and On-Demand/Spot/Savings Plans at a high level.
- [ ] Explain ALB/NLB and ASG autoscaling/health checks.
- [ ] Compare S3/EBS/EFS and explain S3 versioning/encryption/replication.
- [ ] Explain RDS Multi-AZ instance vs readable cluster vs read replica.
- [ ] Compare ECS, EKS, Fargate and Lambda.
- [ ] Explain EKS VPC CNI, Pod Identity, node scaling and persistent storage.
- [ ] Secure a Jenkins/GitHub CI workflow using temporary IAM credentials.
- [ ] Use Terraform safely with modules, remote state, locking and drift checks.
- [ ] Describe blue/green, canary, rollback and DB migration compatibility.
- [ ] Explain CloudWatch/CloudTrail/Config, metrics/logs/traces, and alerting.
- [ ] Define SLIs, SLOs, error budget, RTO and RPO with examples.
- [ ] Debug 10 production scenarios without guesswork or blind restarts.
- [ ] Present two end-to-end architectures with security, reliability and cost analysis.
- [ ] Explain at least two genuine projects: your decisions, ownership and outcomes.

**Interview-ready threshold:** Consistently score 2 or 3 for all essential topics and deliver clear, defensible incident/design explanations. No need to memorize every service launch or quota.

## 34. Official Documentation and Further Study

**Primary official references:**

- [AWS documentation home](https://docs.aws.amazon.com/)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [AWS shared responsibility model](https://aws.amazon.com/compliance/shared-responsibility-model/)
- [IAM best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM policy evaluation logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
- [AWS VPC user guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [S3/DynamoDB gateway VPC endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html)
- [Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/)
- [EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
- [S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [RDS Multi-AZ concepts](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)
- [EKS Best Practices](https://docs.aws.amazon.com/eks/latest/best-practices/introduction.html)
- [EKS Pod Identity vs IRSA](https://docs.aws.amazon.com/eks/latest/userguide/service-accounts.html)
- [ECS Developer Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [AWS Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
- [CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
- [CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)
- [Terraform AWS provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform S3 backend](https://developer.hashicorp.com/terraform/language/backend/s3)
- [AWS CLI reference](https://docs.aws.amazon.com/cli/latest/)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [AWS Pricing Calculator](https://calculator.aws/)
- [AWS Service Quotas](https://docs.aws.amazon.com/servicequotas/latest/userguide/intro.html)
- [AWS workshops](https://workshops.aws/)

**Useful learning routes:** AWS Skill Builder, AWS Prescriptive Guidance, AWS re:Post knowledge base, official reference architectures, practical AWS Free Tier *eligibility and service charges subject to account/plan rules*. Verify any tutorial against current IAM, network and deployment guidance.

---

## Final Interview Advice

1. **Explain the network packet path**, not just service names.
2. **Distinguish permissions from connectivity**: a reachable S3 endpoint can still return `AccessDenied`.
3. **Show recovery thinking**: detection, rollback, backups, RTO/RPO and postmortem.
4. **Make trade-offs explicit**: cost vs availability, managed vs control, throughput vs consistency.
5. **Support answers with real work** and be precise about production experience vs labs.
6. **Practice CLI and failure scenarios**; many interview rounds test how you diagnose rather than how many services you recognize.

**Suggested GitHub structure:**

```text
DevOps-Interview-Preparation/
├── AWS.md                # This file
├── Kubernetes.md         # Architecture, troubleshooting, kubectl, K8s Q&A
├── Jenkins.md            # CI/CD pipelines and troubleshooting
├── Terraform.md          # IaC patterns and modules
├── Docker.md             # Containers, images and runtime
├── Linux.md              # System administration and diagnosis
├── Networking.md         # TCP/IP, DNS, ports and debugging
└── README.md             # Repository navigation
```

> **Revision tip:** Refresh the guide periodically when AWS release notes, docs, security defaults, lifecycle policies and service limits change. The aim is defensible system reasoning—not memorizing stale quotas.

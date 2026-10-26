# Azure Cloud + Azure DevOps CI/CD — Complete Interview & Hands-on Guide

> **Audience:** DevOps Engineers, SREs, Platform Engineers, Cloud Engineers, and release engineers.  
> **Level:** Beginner → intermediate → production troubleshooting.  
> **Updated:** October 2026. Azure products, SKUs, tasks, and prices evolve: always verify live documentation for your tenant and region.  
> **Recommended repository name:** `Azure.md`. An identical `AWS.md` copy is supplied if you need that exact filename, but the contents cover **Microsoft Azure**, not AWS.

## Contents

1. [Azure and Azure DevOps in 60 seconds](#1-azure-and-azure-devops-in-60-seconds)
2. [Cloud basics, hierarchy, and essential services](#2-cloud-basics-hierarchy-and-essential-services)
3. [Azure networking for DevOps and SRE](#3-azure-networking-for-devops-and-sre)
4. [Identity, security, governance, and secrets](#4-identity-security-governance-and-secrets)
5. [Azure CLI command handbook](#5-azure-cli-command-handbook)
6. [Azure DevOps services and Git workflows](#6-azure-devops-services-and-git-workflows)
7. [Azure Pipelines YAML fundamentals](#7-azure-pipelines-yaml-fundamentals)
8. [Build, test, artifacts, environments, and release controls](#8-build-test-artifacts-environments-and-release-controls)
9. [Hands-on project: Python API → ACR → AKS](#9-hands-on-project-python-api--acr--aks)
10. [GitHub Actions to Azure using OIDC](#10-github-actions-to-azure-using-oidc)
11. [Terraform with Azure](#11-terraform-with-azure)
12. [Bicep and ARM deployments](#12-bicep-and-arm-deployments)
13. [Docker, ACR, AKS, Helm, and GitOps](#13-docker-acr-aks-helm-and-gitops)
14. [App Service, Functions, VMs, and deployment methods](#14-app-service-functions-vms-and-deployment-methods)
15. [Monitoring, logging, KQL, and SRE](#15-monitoring-logging-kql-and-sre)
16. [DevSecOps and production-ready patterns](#16-devsecops-and-production-ready-patterns)
17. [Common production failures and solutions](#17-common-production-failures-and-solutions)
18. [AWS ↔ Azure quick comparison](#18-aws--azure-quick-comparison)
19. [Interview questions with answers](#19-interview-questions-with-answers)
20. [Mock interview scenarios and practice labs](#20-mock-interview-scenarios-and-practice-labs)
21. [Learning roadmap and official documentation](#21-learning-roadmap-and-official-documentation)

---

## 1. Azure and Azure DevOps in 60 seconds

**Microsoft Azure** is a public cloud platform providing compute, networking, databases, identity, storage, security, and observability. **Azure DevOps** is a set of software development lifecycle tools; you can use Azure DevOps to deliver applications to Azure, AWS, on-premises, or other clouds. **Azure Pipelines** is Azure DevOps' CI/CD engine.

### The five Azure DevOps services

| Service | Main purpose | Analogy |
|---|---|---|
| Azure Boards | Backlogs, work items, sprints | Jira |
| Azure Repos | Hosted Git repositories, pull requests | GitHub/GitLab |
| Azure Pipelines | Build, test, package, release | Jenkins/GitLab CI |
| Azure Artifacts | Package feeds: NuGet, npm, Maven, PyPI, etc. | Nexus/Artifactory |
| Azure Test Plans | Manual/exploratory testing | Test management suite |

### Typical software-delivery architecture

```text
Developer → Git push / PR
             |
             v
        Azure Repos or GitHub
             |
             v
      Azure Pipelines (CI)
      ├── Checkout source
      ├── Compile / build
      ├── Unit/integration tests
      ├── SAST, SCA, secret scan
      └── Package artifact or image
             |
       Azure Artifacts / ACR
             |
             v
      Azure Pipelines (CD)
      ├── Dev / Test / Staging
      ├── Approval and policy gates
      ├── Deploy to App Service / AKS / VM
      └── Health check / rollback
             |
             v
 Azure Monitor / Application Insights / Logs
```

**CI:** automatically validate and package changes. **Continuous Delivery:** deployment-ready software with an optional manual production approval. **Continuous Deployment:** validated changes automatically reach production without a manual release decision.

**Example:** A pull request runs tests; a merge to `main` produces a versioned container image; dev deploys automatically; a protected production stage deploys the *same image digest* after checks.

---

## 2. Cloud basics, hierarchy, and essential services

### Azure resource hierarchy

```text
Microsoft Entra tenant (identity boundary)
└── Management groups (governance grouping)
    └── Subscription (billing/quota/governance scope)
        └── Resource group (logical lifecycle grouping)
            ├── Virtual network
            ├── Virtual machine
            ├── Storage account
            ├── Key Vault
            ├── Container registry
            └── AKS cluster
```

- **Tenant:** Microsoft Entra ID identity and directory boundary. An organization can have multiple subscriptions.
- **Management group:** policies/RBAC applied across subscriptions.
- **Subscription:** billing, resource-provider registration, service quotas, and governance.
- **Resource group:** logical container for resources; deleting a group deletes the contained resources.
- **Region:** geographic location such as `centralindia` or `eastus`; service availability/quotas differ.
- **Availability zone:** separate datacenter fault domain within a supported region.
- **Resource ID:** `/subscriptions/<id>/resourceGroups/<rg>/providers/Microsoft.Compute/virtualMachines/<vm>`.
- **Tags:** searchable allocation/governance metadata, e.g. `env=dev`, `owner=platform`, `costCenter=1234`.
- **Azure Resource Manager (ARM):** Azure control plane/API for deploying and managing resources.

### Core service categories

| Category | Services | Real-world DevOps use |
|---|---|---|
| Compute | Virtual Machines, VM Scale Sets, App Service, Functions | Build agents, API hosting, event-driven automation |
| Containers | Azure Container Registry (ACR), Azure Kubernetes Service (AKS), Container Apps | Images, orchestration, scalable microservices |
| Storage | Blob, Files, managed disks, queues | Build archives, Terraform state, backups, shared files |
| Networking | VNet, subnet, NSG, Azure Firewall, VPN Gateway, ExpressRoute | Segmentation, secure hybrid access |
| Traffic | Load Balancer, Application Gateway, Front Door, Traffic Manager | L4/L7 routing, WAF, regional/global failover |
| Identity/security | Microsoft Entra ID, RBAC, managed identities, Key Vault, Defender for Cloud | Passwordless access, least privilege, secrets |
| Database/messaging | Azure SQL, Cosmos DB, PostgreSQL, Service Bus, Event Hubs | State, messaging, streaming |
| Observability | Azure Monitor, Log Analytics, Application Insights, Managed Prometheus, Managed Grafana | Metrics, traces, logs, alerts, SLOs |
| IaC/governance | Terraform, Bicep, ARM, Azure Policy, Cost Management | Reproducible infra, guardrails, budgets |

### Shared responsibility and cost

- Azure secures the underlying cloud; *you* still secure identities, data, workloads, network configuration, dependencies, application code, and access.
- **IaaS:** you manage OS configuration, patches, app, and data on VMs.
- **PaaS:** provider handles more underlying OS/platform responsibilities; you still manage app and identities.
- **SaaS:** you primarily manage data, users, and configurations.
- Watch VM/AKS runtime, outbound data, storage, Log Analytics ingestion, gateways, reserved public IPs, and premium service tiers. Deleting a demo resource group is an important final lab step.

### Compute selection: interview heuristic

- **VM:** customized OS/legacy application/long-lived self-hosted build agent.
- **App Service:** managed HTTP web app with simpler operations.
- **Functions:** event-driven short-running jobs or workflow handlers; choose hosting plan appropriately.
- **Container Apps:** managed container apps without running Kubernetes yourself.
- **AKS:** full Kubernetes API/ecosystem, multi-service networking and deployment control; increased operational complexity.

---

## 3. Azure networking for DevOps and SRE

### Network fundamentals

- **VNet:** private network address space, e.g. `10.10.0.0/16`.
- **Subnet:** segment within VNet, e.g. `10.10.1.0/24` for app nodes.
- **NSG:** stateful allow/deny rules for subnet or network interface; lower priority number wins. Default rules also apply.
- **ASG:** group VM NICs by application role for NSG rules.
- **Route table / UDR:** custom next-hop routes; e.g. outbound through Azure Firewall.
- **VNet peering:** private traffic between VNets; not automatically transitive.
- **Private endpoint:** private IP in your VNet mapped to a supported PaaS resource. Configure private DNS resolution and validate it.
- **Service endpoint:** secure a supported PaaS resource to selected VNet/subnets over Microsoft's backbone but it retains the service's public endpoint model.
- **NAT Gateway:** managed, predictable outbound SNAT for a subnet.
- **Azure Bastion:** browser/portal-based SSH/RDP without exposing VM SSH/RDP publicly.
- **VPN Gateway / ExpressRoute:** encrypted IPsec VPN / dedicated private connectivity from on-premises.

### Inbound request example

```text
Internet
   |
 Azure Front Door (global HTTP routing, optional WAF)
   |
 Application Gateway (regional L7 routing, optional WAF)
   |
 Private/ingress endpoint
   |
 App Service / AKS Ingress → Service → Pod
   |
 Private Endpoint → Azure Database
```

**Load balancing distinctions:**

| Service | Typical scope | Works at | Common use |
|---|---|---|---|
| Azure Load Balancer | Regional or global | Layer 4 | TCP/UDP traffic to endpoints |
| Application Gateway | Regional | Layer 7 HTTP(S), also supported TCP/TLS modes | Regional routing, WAF, TLS |
| Azure Front Door | Global | Layer 7 | Global edge entry, WAF, caching, failover |
| Traffic Manager | Global | DNS-based | DNS traffic steering/failover |

### VNet CLI example

```bash
RG=rg-azdevops-demo
LOCATION=centralindia
az group create -n "$RG" -l "$LOCATION"

az network vnet create \
  -g "$RG" -n vnet-demo -l "$LOCATION" \
  --address-prefixes 10.10.0.0/16 \
  --subnet-name snet-app --subnet-prefixes 10.10.1.0/24

az network nsg create -g "$RG" -n nsg-app
az network nsg rule create -g "$RG" --nsg-name nsg-app \
  -n allow-https --priority 200 --direction Inbound \
  --access Allow --protocol Tcp --destination-port-ranges 443

az network vnet subnet update -g "$RG" --vnet-name vnet-demo \
  -n snet-app --network-security-group nsg-app
```

**Important:** The example illustrates primitives, not a complete production network. Do not expose SSH (`22`) or RDP (`3389`) to `0.0.0.0/0`; constrain source and prefer private access. AKS subnet sizing, CNI mode, routing, and egress must be designed before production deployment.

### Debug a connectivity failure

1. Verify DNS (`nslookup` / `dig`) and private DNS links.
2. Check application binding/port and pod or VM listener (`ss -lntp`, `kubectl get endpoints`).
3. Check NSG rules, effective security rules, route tables, firewalls, and NAT.
4. Check access restrictions / private endpoint status / TLS hostnames.
5. Use Azure Network Watcher connection troubleshooting/flow logs where configured; use app logs and packet capture as permitted.

---

## 4. Identity, security, governance, and secrets

### Microsoft Entra ID vs Azure RBAC

- **Authentication:** Who are you? Microsoft Entra ID authenticates users, apps, and workload identities.
- **Authorization:** What may you do? Azure RBAC assigns a **role** at a **scope** (management group, subscription, resource group, or resource).
- **Principal:** user, group, service principal, or managed identity.
- **Service principal:** application identity in a tenant. Avoid static client secrets when federation is available.
- **System-assigned managed identity:** lifecycle bound to one Azure resource.
- **User-assigned managed identity:** independent resource reusable across eligible workloads.
- **Workload identity federation (OIDC):** trust an external token issuer; get short-lived Azure credentials without storing a long-lived secret.
- **Kubernetes RBAC:** separately controls requests to Kubernetes API; Azure subscription Contributor access does *not* automatically mean cluster-admin privileges.
- **Data-plane RBAC:** storage Blob Data Contributor and Key Vault Secrets User are different from generic management-plane roles.

### RBAC examples

```bash
az account show --query '{subscription:id,tenant:tenantId,user:user.name}' -o json
az role assignment list --assignee <OBJECT_ID> --all -o table
az role assignment create \
  --assignee-object-id <OBJECT_ID> \
  --assignee-principal-type ServicePrincipal \
  --role Reader \
  --scope "/subscriptions/<SUB_ID>/resourceGroups/rg-azdevops-demo"
```

`Owner` can manage resources **and assign roles**; `Contributor` can manage most resources but cannot grant RBAC assignments; `Reader` can read Azure management-plane metadata. Roles such as `AcrPull`, `AcrPush`, `Key Vault Secrets User`, and `Storage Blob Data Contributor` cover specific data-plane permissions. Role semantics and registry authorization modes vary; consult the live docs.

### Secure Azure DevOps service connection

In Azure DevOps: **Project settings → Service connections → New service connection → Azure Resource Manager → Workload identity federation**. Prefer an automatic app registration when permissions allow, or a federated user-assigned managed identity. Give the identity the *narrowest necessary role/scope*; authorize only required pipelines. Use separate connections per application and environment.

```yaml
# AzureCLI@2 uses an Azure Resource Manager service connection
steps:
- task: AzureCLI@2
  inputs:
    azureSubscription: 'sc-azure-dev'  # create this connection in the UI
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az account show --query '{name:name,id:id}' -o table
      az group list -o table
```

**Approvals:** Azure DevOps supports checks on environments, service connections, variable groups, etc. For high-assurance production, centrally enforced branch controls, service-connection approvals, and required templates are particularly valuable because YAML authors must not be able to bypass gates simply by editing their own pipeline. A protected environment also gives useful deployment history.

### Secret-handling rules

- Keep runtime credentials in **Azure Key Vault** (with correct data-plane access); apps should prefer managed identity to read secrets.
- Link Key Vault secrets through Azure Pipelines **Library → Variable groups** or use a supported Key Vault task; access/network prerequisites apply.
- Do not put secrets into Git, Dockerfile `ARG`, build logs, command-line flags, Terraform outputs, or unprotected artifacts.
- Rotate exposed credentials; scope tokens minimally. Pipeline secret masking is **not** a substitute for avoiding secret output.
- Use OIDC between GitHub Actions/Azure DevOps and Azure instead of static service principal JSON/passwords.
- Use SAST (Coverity/SonarQube), dependency/SCA (Black Duck/Dependabot), image scanning, IaC scanning, and artifact provenance in CI.

### Governance

**Azure Policy** evaluates/enforces rules such as allowed locations, required tags, encryption, and restricted public networking. **Management groups** organize inherited policy and RBAC. **Defender for Cloud** supplies cloud-security posture and workload protection capabilities. **Budgets** alert on cloud spend, but are not a hard spending cap.

---

## 5. Azure CLI command handbook

Install Azure CLI, then authenticate interactively (or use Azure Cloud Shell):

```bash
az version
az login
az account list -o table
az account set --subscription '<SUBSCRIPTION_ID_OR_NAME>'
az account show -o table
az group list -o table
```

### Common commands

```bash
# Resource groups
az group create --name rg-demo --location centralindia --tags env=lab owner=devops
az group show -n rg-demo -o json
az resource list -g rg-demo -o table

# Available VM sizes (region restrictions/quotas still apply)
az vm list-sizes --location centralindia -o table

# VM inventory/health
az vm list -g rg-demo -d -o table
az vm get-instance-view -g rg-demo -n my-vm -o json

# Storage inventory
az storage account list -g rg-demo -o table

# AKS/ACR inventory
az aks list -g rg-demo -o table
az acr list -g rg-demo -o table
az acr repository list -n <ACR_NAME> -o table

# Web app diagnostics
az webapp list -g rg-demo -o table
az webapp log tail -g rg-demo -n <WEBAPP_NAME>

# Locate command help
az aks create --help
az find 'az monitor metrics'
```

**Safety:** `az group delete -n rg-demo --yes` destroys all resources in that group; inspect contents first and run only for your own disposable lab.

---

## 6. Azure DevOps services and Git workflows

### Typical branch and release strategy

```text
feature/login ─── PR ───┐
feature/alerts ─── PR ──┤
                       main ── versioned image/artifact ── dev ── stage ── prod
                                      (same immutable version promoted)
```

- Require PR review, mandatory CI checks, no direct push to protected branches, and a clear ownership policy.
- Use trunk-based development for fast integration or release branches if lifecycle/support needs require them.
- Tag releases (`v1.4.2`); prefer image digests or immutable version tags, never `latest` as a production release reference.
- Keep build and deploy concerns logically separated; promote the artifact already tested rather than rebuilding for production.

### Azure Pipelines and Jenkins mapping

| Jenkins concept | Azure Pipelines equivalent |
|---|---|
| Jenkinsfile (declarative) | `azure-pipelines.yml` |
| Stage / steps | Stage / job / step |
| Node label | Agent pool / demands |
| Jenkins credentials | Service connections / Key Vault / secret variables |
| Shared library | YAML templates |
| Build archive | Pipeline artifacts / Azure Artifacts / ACR |
| Lockable resources | Exclusive-lock check on protected resources |
| Manual `input` gate | Approval/check on environment or resource |
| Gerrit Trigger | Repository hooks / external integration; not identical out of box |
| Email notifications | Azure DevOps notifications/service hooks/integrations |

**Agents:** Microsoft-hosted agents are ephemeral, freshly provisioned per job; self-hosted agents provide custom software, private network reachability, and reusable machine-level caches, but require OS/agent hardening, patching, and capacity management. On Azure DevOps Services in October 2026, `ubuntu-latest` resolves to Ubuntu 24.04; `ubuntu-26.04` is also offered explicitly. Avoid relying on the moving label when image reproducibility matters. On-prem Azure DevOps Server uses self-hosted agents.

---

## 7. Azure Pipelines YAML fundamentals

### YAML building blocks

```yaml
trigger:
  branches:
    include: [main]

pool:
  vmImage: 'ubuntu-24.04'

variables:
  buildConfiguration: 'Release'

stages:
- stage: Build
  displayName: Build and unit test
  jobs:
  - job: Test
    steps:
    - checkout: self
    - bash: |
        echo "Commit: $(Build.SourceVersion)"
        echo "Build ID: $(Build.BuildId)"
      displayName: Print metadata

- stage: Deploy
  dependsOn: Build
  condition: succeeded()
  jobs:
  - job: Explain
    steps:
    - bash: echo 'Production deployments need an authorized connection and policy gates.'
```

- **Trigger:** code update, scheduled run, manual run, pipeline completion.
- **Stage:** big lifecycle boundary; stages run sequentially by default.
- **Job:** group of steps running on a specific agent; jobs can be parallel.
- **Step:** script or prebuilt task, e.g. `AzureCLI@2`.
- **Deployment job:** specialized job with environment tracking, deployment strategies, approvals/checks.
- **Task:** versioned reusable action, e.g. `AzureWebApp@1`, `PublishPipelineArtifact@1`.
- **Service connection:** permissions to external service; create it outside YAML.

### Variables, parameters, and expressions

```yaml
parameters:
- name: targetEnv
  type: string
  default: dev
  values: [dev, staging]

variables:
  appName: 'orders-api'

steps:
- bash: |
    echo 'Value from template expansion: ${{ parameters.targetEnv }}'
    echo 'Macro variable at run time: $(appName)'
    echo 'Build reason: $(Build.Reason)'
```

| Syntax | Resolution | Example use |
|---|---|---|
| `${{ }}` | Template/compile time | Parameter-selected stage/template |
| `$(name)` | Runtime macro (before task runs) | Task input, scripted value |
| `$[ ]` | Runtime expression | Stage/job condition, derived variable |
| `condition:` | Runtime check | Branch, status, trigger logic |

**Critical:** YAML `$(var)` is an Azure Pipelines macro. Shell `$(command)` is command substitution. When writing shell scripts, be deliberate about where pipeline substitution applies. Avoid printing secret variables.

### Conditional execution example

```yaml
- stage: Production
  dependsOn: Build
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: Prod
    environment: 'production'  # pre-create and configure protected checks
    strategy:
      runOnce:
        deploy:
          steps:
          - bash: echo 'Replace with a real deployment task.'
```

**PR validation caveat:** YAML `pr:` is supported for GitHub/Bitbucket repositories connected to Azure Pipelines; for **Azure Repos Git**, configure PR validation through **branch policies/build validation**, not the YAML PR trigger.

### Templates for large engineering teams

```yaml
# azure-pipelines.yml
stages:
- template: templates/test.yml
  parameters:
    pythonVersion: '3.12'
```

```yaml
# templates/test.yml
parameters:
- name: pythonVersion
  type: string

stages:
- stage: Test
  jobs:
  - job: UnitTests
    pool:
      vmImage: ubuntu-24.04
    steps:
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '${{ parameters.pythonVersion }}'
    - bash: python -m pip install -r requirements.txt && pytest -q
```

Use centrally governed templates, reviewed task versions, separate deployment credentials, and artifact integrity checks to reduce supply-chain risk.

---

## 8. Build, test, artifacts, environments, and release controls

### Python CI with test report and artifact

```yaml
trigger:
  branches:
    include: [main]

pool:
  vmImage: ubuntu-24.04

steps:
- checkout: self
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.12'
- bash: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
    pytest -q --junitxml=test-results.xml
  displayName: Unit tests
- task: PublishTestResults@2
  condition: succeededOrFailed()
  inputs:
    testResultsFormat: 'JUnit'
    testResultsFiles: 'test-results.xml'
- task: PublishPipelineArtifact@1
  inputs:
    targetPath: '$(Build.SourcesDirectory)'
    artifact: 'source-snapshot'
```

For an actual application, publish the *built distributable* (`dist/`, ZIP, JAR) instead of the whole source directory; the example emphasizes syntax.

### Cross-stage artifact use

```yaml
# Publisher in an earlier job/stage
- publish: '$(Build.ArtifactStagingDirectory)'
  artifact: 'drop'

# Consumer in a later dependent job/stage
- download: current
  artifact: drop
# Files at $(Pipeline.Workspace)/drop
```

- Pipeline artifacts are primarily for **Azure DevOps Services**; Azure DevOps Server may require Build Artifacts.
- Container images should be pushed to ACR and promoted by **immutable version/digest**.
- Keep artifact retention, provenance, SBOMs, and signature policy suitable for your organization.

### Recommended release flow

1. Pull request validation: formatting, lint, unit tests, SAST/SCA, IaC checks.
2. Merge: build once, version image/package, scan, publish artifact.
3. Dev deployment: smoke tests and quick rollback.
4. Staging: realistic integration/performance/security checks.
5. Production: resource-level approval, restricted identity, canary/blue-green or progressive rollout.
6. Post-release: SLI/SLO checks, dashboard inspection, deployment annotations, rollback trigger if needed.

**Deployment slots:** Azure App Service slots can support staging and slot swap on supported plans. Production swap changes which slot serves traffic; review warm-up and app settings marked as slot-specific.

## 9. Hands-on project: Python API → ACR → AKS

**Goal:** Write a small HTTP API, test it, package it into a non-root Docker container, publish it to Azure Container Registry, and deploy an immutable build ID to Azure Kubernetes Service through a multi-stage Azure Pipeline.

**Warning:** AKS worker nodes and other resources incur charges. Make sure you own the Azure subscription, have sufficient role/quotas, and delete the disposable resource group afterwards. The example uses a public `LoadBalancer` service and is for a **lab**, not a fully hardened production application.

### 9.1 Repository layout

```text
azure-devops-demo/
├── app/
│   ├── __init__.py
│   └── main.py
├── tests/
│   └── test_main.py
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── requirements.txt
├── Dockerfile
├── .dockerignore
└── azure-pipelines.yml
```

### 9.2 Application: `app/main.py`

```python
from fastapi import FastAPI

app = FastAPI(title="Azure DevOps Demo")

@app.get("/")
def home():
    return {"message": "Hello from Azure AKS"}

@app.get("/health")
def health():
    return {"status": "ok"}
```

### 9.3 Dependencies: `requirements.txt`

```text
fastapi>=0.116,<1
uvicorn>=0.30,<1
httpx>=0.27,<1
pytest>=8,<10
```

For production, separate runtime from test dependencies, lock transitive dependencies, and update based on security policy.

### 9.4 Unit test: `tests/test_main.py`

```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_health():
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json() == {"status": "ok"}
```

Test locally:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pytest -q
uvicorn app.main:app --reload
# In another shell: curl http://localhost:8000/health
```

### 9.5 `Dockerfile`

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app
RUN useradd --system --uid 10001 appuser

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY app/ ./app/

USER appuser
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**`.dockerignore`**:

```text
.git
.venv
__pycache__
.pytest_cache
*.pyc
.env
```

Build and test locally:

```bash
docker build -t demoapi:local .
docker run --rm -p 8000:8000 demoapi:local
curl http://localhost:8000/health
```

### 9.6 Create ACR and AKS (lab CLI)

```bash
# CHANGE THESE BEFORE EXECUTING
export RG='rg-azdevops-lab'
export LOCATION='centralindia'
export ACR_NAME='replacewithuniqueacrname123' # 5–50 alphanumeric chars, globally unique
export AKS_NAME='aks-azdevops-lab'

az login
az account set --subscription '<YOUR_SUBSCRIPTION_ID>'
az group create -n "$RG" -l "$LOCATION"

# Registry; sku availability/pricing vary
az acr create -g "$RG" -n "$ACR_NAME" --sku Basic

# For demonstration; choose VM size/node count based on regional quota and pricing
az aks create \
  -g "$RG" -n "$AKS_NAME" \
  --node-count 1 \
  --generate-ssh-keys \
  --attach-acr "$ACR_NAME"

# Verify access (Entra-integrated clusters may require kubelogin and RBAC configuration)
az aks get-credentials -g "$RG" -n "$AKS_NAME" --overwrite-existing
kubectl get nodes

# Build/push image server-side in ACR; no local Docker daemon required
az acr build -r "$ACR_NAME" -t demoapi:manual-v1 .

# Verify registry name and push
az acr repository show-tags -n "$ACR_NAME" --repository demoapi -o table
```

**Permissions and caveats:** An administrator must provide your identity and the pipeline identity with required ARM/RBAC permissions. ACR pull rights typically apply to the AKS kubelet identity; on registries using repository-scoped ABAC permissions the role model differs. Network-restricted registries and private AKS clusters require additional network/agent setup. `az aks create` defaults are not a production reference architecture.

### 9.7 Kubernetes manifests

Replace `YOUR_ACR_NAME` with the **same actual registry name** used in CLI and pipeline. The Azure Pipeline deploy task changes the image tag from `bootstrap` to that build's ID.

**`k8s/deployment.yaml`**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demoapi
  namespace: demo
spec:
  replicas: 2
  revisionHistoryLimit: 5
  selector:
    matchLabels:
      app: demoapi
  template:
    metadata:
      labels:
        app: demoapi
    spec:
      containers:
      - name: demoapi
        image: YOUR_ACR_NAME.azurecr.io/demoapi:bootstrap
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8000
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 3
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 10
          periodSeconds: 10
        resources:
          requests:
            cpu: 50m
            memory: 64Mi
          limits:
            cpu: 250m
            memory: 256Mi
        securityContext:
          allowPrivilegeEscalation: false
          runAsNonRoot: true
          runAsUser: 10001
          capabilities:
            drop: ["ALL"]
```

**`k8s/service.yaml`**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: demoapi-svc
  namespace: demo
spec:
  selector:
    app: demoapi
  ports:
  - name: http
    port: 80
    targetPort: 8000
  type: LoadBalancer  # LAB ONLY: provisions a public endpoint, may incur charges
```

Create namespace once (you must have Kubernetes namespace permissions):

```bash
kubectl create namespace demo
# It is normal to receive AlreadyExists when re-running this command.
```

### 9.8 Configure the Azure DevOps project

1. Create Azure DevOps organization/project; push repository to **Azure Repos Git** or connect a GitHub repository.
2. Create an **Azure Resource Manager service connection** called `sc-azure-lab` using **Workload identity federation**; authorize *only this pipeline*. It needs permissions to invoke ACR builds and to deploy to AKS. Define permissions with your Azure admin; do not grant subscription-wide Owner by default.
3. Create an Azure Pipelines **environment** called `demo-dev`. Configure appropriate checks/approvals as desired.
4. Update placeholders in `azure-pipelines.yml` and `k8s/deployment.yaml`. Verify the hosted agent can reach your AKS API; a **private** cluster typically requires a connected self-hosted agent.
5. For Azure Repos, enforce PR validation via **Repos → Branches → main → Branch policies → Build validation**. For GitHub, configure branch protection/rulesets and PR triggers as applicable.
6. Commit and run the pipeline. The pipeline's ACR build and Kubernetes deployment use your service connection rather than embedded secrets.

### 9.9 Full multi-stage `azure-pipelines.yml`

```yaml
name: demoapi-$(Date:yyyyMMdd).$(Rev:r)

trigger:
  branches:
    include: [main]

pool:
  vmImage: 'ubuntu-24.04'

variables:
  acrName: 'YOUR_ACR_NAME'              # same registry as manifests
  aksResourceGroup: 'rg-azdevops-lab'
  aksCluster: 'aks-azdevops-lab'
  serviceConnection: 'sc-azure-lab'     # must exist in project settings
  imageRepository: 'demoapi'

stages:
- stage: Validate
  displayName: Run Python tests
  jobs:
  - job: UnitTests
    steps:
    - checkout: self
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.12'
    - bash: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pytest -q --junitxml=test-results.xml
      displayName: Install and test
    - task: PublishTestResults@2
      condition: succeededOrFailed()
      inputs:
        testResultsFormat: 'JUnit'
        testResultsFiles: 'test-results.xml'
        failTaskOnFailedTests: true

- stage: BuildPush
  displayName: Build and push versioned image
  dependsOn: Validate
  condition: succeeded()
  jobs:
  - job: BuildImage
    steps:
    - checkout: self
    - task: AzureCLI@2
      displayName: ACR cloud build
      inputs:
        azureSubscription: 'sc-azure-lab'
        scriptType: bash
        scriptLocation: inlineScript
        inlineScript: |
          set -euo pipefail
          az acr build \
            --registry '$(acrName)' \
            --image '$(imageRepository):$(Build.BuildId)' \
            --file Dockerfile .

- stage: DeployDev
  displayName: Deploy to AKS dev
  dependsOn: BuildPush
  condition: succeeded()
  jobs:
  - deployment: DeployDemo
    displayName: Deploy versioned image
    environment: 'demo-dev'
    strategy:
      runOnce:
        deploy:
          steps:
          - checkout: self
          - task: KubernetesManifest@1
            displayName: Apply manifests with immutable build tag
            inputs:
              action: 'deploy'
              connectionType: 'azureResourceManager'
              azureSubscriptionConnection: 'sc-azure-lab'
              azureResourceGroup: '$(aksResourceGroup)'
              kubernetesCluster: '$(aksCluster)'
              namespace: 'demo'
              manifests: |
                $(Build.SourcesDirectory)/k8s/deployment.yaml
                $(Build.SourcesDirectory)/k8s/service.yaml
              containers: |
                $(acrName).azurecr.io/$(imageRepository):$(Build.BuildId)
```

**Why the pipeline works:** `Validate` blocks a failed test; `BuildPush` publishes `demoapi:<Build.BuildId>` directly to ACR; the ARM-based Kubernetes task authenticates to AKS and replaces the image tag of the matching repository in the checked-out manifest. `demo-dev` stores deployment history and can be guarded with environment checks. The identity still needs AKS/Kubernetes authorization; neither an ARM service connection nor ACR `AcrPush` automatically gives every permission. Container image path **must match** between the manifest and the `containers` input.

For **production**, add a separate `DeployProd` stage with its own `environment: production`, separate federated connection and AKS cluster/namespace, artifact/image digest promotion, more restrictive rights, automated checks, and approval/policy gates outside the YAML. Avoid using `Build.BuildId` across independent pipeline runs unless the producing run is explicitly passed into the deployment pipeline.

### 9.10 Verify, debug, and rollback

```bash
az aks get-credentials -g "$RG" -n "$AKS_NAME" --overwrite-existing
kubectl -n demo get deploy,pods,svc -o wide
kubectl -n demo rollout status deployment/demoapi --timeout=180s
kubectl -n demo describe deployment demoapi
kubectl -n demo logs deployment/demoapi --tail=100
kubectl -n demo get events --sort-by=.lastTimestamp

# Get public IP after the Azure Load Balancer becomes ready
kubectl -n demo get service demoapi-svc
# curl http://<EXTERNAL_IP>/health

# Roll back to the prior ReplicaSet IF rollout history exists
kubectl -n demo rollout history deployment/demoapi
kubectl -n demo rollout undo deployment/demoapi
```

**Note:** A manual rollback can be overwritten by a subsequent pipeline or GitOps reconciliation. For production, roll back the desired manifest/chart/version in source control, investigate why the release failed, and verify the health metrics.

### 9.11 Lab cleanup

```bash
# Verify group and resource inventory first
az resource list -g "$RG" -o table
# Permanently deletes ALL resources in this disposable resource group:
az group delete -n "$RG" --yes
```

---

## 10. GitHub Actions to Azure using OIDC

GitHub Actions is a separate CI/CD platform. It can deploy to Azure with **GitHub OIDC → Microsoft Entra federated credential → Azure RBAC**, avoiding stored client secrets.

### Identity setup outline

1. Create an Entra app registration/service principal or user-assigned managed identity.
2. Create a **federated credential** trusting the right GitHub organization/repository and branch or GitHub environment subject.
3. Assign the identity only the Azure resource roles it needs.
4. Add `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, and `AZURE_SUBSCRIPTION_ID` as GitHub repository/environment configuration **secrets or variables** (these IDs are identifiers, not client secrets).
5. Protect the GitHub environment and branch; grant workflow `id-token: write`.

### `.github/workflows/azure.yml`

```yaml
name: Azure Cloud Validation

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

jobs:
  validate-azure:
    runs-on: ubuntu-latest
    environment: dev
    steps:
      - uses: actions/checkout@v4
      - name: Log in to Azure without a client secret
        uses: azure/login@v3
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - name: Query resources
        run: |
          az account show --output table
          az group list --output table
```

**Real CI/CD extension:** after tests, run `az acr build -r <registry> -t demoapi:${{ github.sha }} .` and use a restricted Azure/Kubernetes deployment identity to promote that immutable SHA or digest. Pin third-party actions to reviewed commits for sensitive production systems; major version tags can move. OIDC **does not grant** Azure access on its own: the matching federated identity and RBAC roles must exist.

---

## 11. Terraform with Azure

**Terraform** provides declarative, multi-cloud infrastructure provisioning with planning, dependency management, state, modules, and providers. Azure's primary provider is `hashicorp/azurerm`. With the v4 provider family, pass a subscription ID explicitly or through `ARM_SUBSCRIPTION_ID`.

### 11.1 Minimal Terraform project

`main.tf`:

```hcl
terraform {
  required_version = ">= 1.7"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}

variable "subscription_id" {
  description = "Target Azure subscription"
  type        = string
}

variable "location" {
  type    = string
  default = "centralindia"
}

resource "azurerm_resource_group" "demo" {
  name     = "rg-terraform-demo"
  location = var.location
  tags = {
    environment = "lab"
    managed_by  = "terraform"
  }
}

output "resource_group_name" {
  value = azurerm_resource_group.demo.name
}
```

```bash
az login
export TF_VAR_subscription_id="$(az account show --query id -o tsv)"
terraform init
terraform fmt -check
terraform validate
terraform plan -out=tfplan
terraform apply tfplan
terraform output
terraform destroy
```

**Do not** commit `terraform.tfstate`, `.terraform/`, unprotected plan files, or secret `.tfvars`. A saved Terraform plan is tied to the provider/state used when it was produced; don't blindly apply an old one.

### 11.2 Remote Terraform state on Azure Blob Storage

Create an existing dedicated state resource group, storage account, and private container **before** using this backend. Restrict networking and assign the pipeline identity the appropriate Blob data-plane role. Backend authentication and provider authentication are related but separate.

`backend.tf`:

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-tfstate"
    storage_account_name = "uniquetfstateaccount123"
    container_name       = "tfstate"
    key                  = "dev/app.tfstate"
    use_azuread_auth     = true
  }
}
```

Initialize with the target subscription and a compatible federated/service-principal/managed identity authentication method as appropriate:

```bash
terraform init -reconfigure
terraform plan -out=tfplan
terraform apply tfplan
```

- Azure Blob backend supports locking to avoid competing Terraform writers, but serialize other deployment tools separately.
- Split states by application/environment and use least-privilege identities.
- Encrypt storage, restrict Blob read/write, consider backup/versioning/soft delete, and audit state access.
- CI: `fmt` → `validate` → security/policy checks → `plan` → review/approval → `apply` with a controlled identity and protected branch.
- For Azure DevOps OIDC, configure Terraform's supported federation environment variables/backend settings and your federated service connection rather than trying to reuse a static password. Specific variables depend on the task/runner.

### 11.3 Terraform vs Bicep

| Terraform | Bicep |
|---|---|
| Multi-provider, multi-cloud, community modules | Azure-native, compiles to ARM templates |
| State file/backend must be managed | ARM handles deployment state in Azure control plane |
| `plan` and `apply` lifecycle | `what-if` and `az deployment` workflows |
| HCL syntax | Azure-focused declarative syntax |
| Good for cross-cloud platforms | Good for Azure-only infrastructure |

---

## 12. Bicep and ARM deployments

**Bicep** is Azure's declarative domain-specific language for deploying ARM resources. You author `.bicep` rather than verbose ARM JSON. For Azure-only IaC, it integrates naturally with deployment scopes and Azure Policy.

### `main.bicep`: storage-account example

```bicep
param location string = resourceGroup().location
param storageName string

resource storage 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: storageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
  }
}

output storageId string = storage.id
```

```bash
az group create -n rg-bicep-lab -l centralindia
az deployment group what-if \
  -g rg-bicep-lab --template-file main.bicep \
  --parameters storageName='<globally-unique-lowercase-name>'
az deployment group create \
  -g rg-bicep-lab --template-file main.bicep \
  --parameters storageName='<globally-unique-lowercase-name>'
```

Use modules and parameter files, enforce naming/tag standards, and review `what-if` output; remember that destructive changes or `complete` deployment mode semantics may affect existing resources. Guard production writes.

---

## 13. Docker, ACR, AKS, Helm, and GitOps

### ACR commands

```bash
az acr login --name "$ACR_NAME"
docker build -t "$ACR_NAME.azurecr.io/demoapi:v1" .
docker push "$ACR_NAME.azurecr.io/demoapi:v1"
az acr repository list -n "$ACR_NAME" -o table
az acr repository show-tags -n "$ACR_NAME" --repository demoapi -o table

# No local Docker needed; build hosted within ACR
az acr build -r "$ACR_NAME" -t demoapi:v2 .
```

Prefer short-lived identity and RBAC for registry access; avoid ACR admin username/password. Configure pull identities appropriate for your registry's RBAC/ABAC mode.

### AKS architecture

```text
Azure-managed Kubernetes control plane
                  |
           AKS node pool(s)
       ┌──────────┴──────────┐
   Node / kubelet         Node / kubelet
    └─ Pod(s)              └─ Pod(s)
       |
  ClusterIP Service → Ingress/Gateway → External client
       |
     ACR images + Key Vault/CSI + Logs/Metrics
```

Control-plane availability, worker node sizing, CNI, zones, policy, upgrades, node pools, autoscaling, and cost determine production reliability. Managed control plane ≠ zero-operations Kubernetes.

### Useful Kubernetes commands

```bash
kubectl get nodes -o wide
kubectl get pods -A
kubectl -n demo get deploy,rs,pods,svc
kubectl -n demo describe pod <POD_NAME>
kubectl -n demo logs <POD_NAME> --previous
kubectl -n demo top pods  # requires metrics collection
kubectl -n demo rollout status deploy/demoapi
kubectl -n demo rollout history deploy/demoapi
kubectl -n demo rollout undo deploy/demoapi
kubectl -n demo scale deploy/demoapi --replicas=3
```

### Helm

```bash
helm create demo-chart
helm lint ./demo-chart
helm template demo ./demo-chart --namespace demo
helm upgrade --install demo ./demo-chart \
  --namespace demo --create-namespace \
  --set image.tag=v1
helm history demo --namespace demo
helm rollback demo 1 --namespace demo
```

Production Helm release: store values by environment, pin chart versions, test templates, use `--atomic --wait` where suitable, and don't place passwords in plain-text Helm values committed to Git.

### GitOps with Argo CD or Flux

```text
CI pipeline → build/scan/sign → immutable image in ACR
                                    |
                         Update desired manifest/chart in Git
                                    |
                         GitOps controller reconciles AKS
                                    |
                           Drift detection / health status
```

Azure DevOps/Azure Pipelines performs CI; **Argo CD**/**Flux** can manage CD by pulling desired state. Avoid two writers (pipeline `kubectl apply` and GitOps controller) competing over the same deployment. A GitOps rollback normally means reverting the desired Git commit/image digest.

### AKS reliability and security checklist

- Deploy multiple replicas across nodes/zones where the SKU/region supports it.
- Configure readiness, liveness, and startup probes appropriately; distinguish startup from application liveness.
- Set CPU/memory requests/limits; understand HPA and cluster-autoscaler scaling boundaries.
- Define PodDisruptionBudget and graceful termination where needed.
- Use network policies and controlled ingress/egress; consider private AKS clusters.
- Use Workload Identity + Key Vault CSI integration for secret access when appropriate.
- Use non-root images, constrained capabilities, image scanning, signed releases, RBAC, and restrictive admission policy.
- Maintain Kubernetes version compatibility and plan node upgrades and API deprecations.

---

## 14. App Service, Functions, VMs, and deployment methods

### Azure App Service

Use when you want managed web hosting without managing your own K8s cluster. Supports code- and container-based deployment, scaling, authentication integrations, settings, and deployment slots on eligible plans.

Example task for an **existing Linux Web App** and prepared ZIP (the package path must exist):

```yaml
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'sc-azure-dev'
    appType: 'webAppLinux'
    appName: 'my-webapp-name'
    package: '$(Pipeline.Workspace)/drop/webapp.zip'
```

Deployment slots can support a safer release:

```text
CI → Artifact → Deploy to staging slot → warm-up/smoke tests
   → check/approval → swap staging into production → monitor
```

**Rollback:** swap back when appropriate or redeploy a known-good artifact. State changes and database migrations may make rollback more complex than changing binaries.

### Azure Functions

Functions run handlers for HTTP requests, messages, storage events, and timers. Common applications: webhook listener, release-status processing, ticket enrichment, lightweight automation. Consider cold-start constraints, timeouts, retries, idempotency, poison queues, managed identity, and execution/hosting plan.

### VM-based deployment

For legacy applications requiring VMs, use image baking, VM Scale Sets, automated configuration, and rolling deployment. Store installation artifacts in private registry/storage and use a gated release rather than manually SSHing into production. When using self-hosted CI agents on VMs, isolate untrusted PR builds from production deployment workers.

---

## 15. Monitoring, logging, KQL, and SRE

### Tooling map

| Tool | What it observes |
|---|---|
| Azure Monitor Metrics | Resource and platform metrics, alerts |
| Log Analytics Workspace | Central log queries with KQL |
| Application Insights | Application request telemetry, dependencies, exceptions, traces |
| Container Insights | AKS/container health and telemetry |
| Azure Managed Prometheus | Prometheus-compatible metrics collection for AKS and supported integrations |
| Azure Managed Grafana | Dashboard visualization/correlation |
| Azure Activity Log | Subscription control-plane events |
| Network Watcher | Network diagnosis and traffic insights |

### Application Insights KQL examples

**Request latency and failure percentage (workspace-based Application Insights):**

```kusto
AppRequests
| where TimeGenerated > ago(1h)
| summarize
    Requests = sum(ItemCount),
    Failures = sumif(ItemCount, Success == false),
    P95Ms = percentile(DurationMs, 95)
  by bin(TimeGenerated, 5m)
| extend FailurePct = iff(Requests == 0, 0.0, 100.0 * Failures / Requests)
| order by TimeGenerated asc
```

**Top exceptions:**

```kusto
AppExceptions
| where TimeGenerated > ago(24h)
| summarize Events = count() by ProblemId
| top 10 by Events desc
```

**Container diagnostics** (available tables depend on the cluster's configured collection and workspace schema):

```kusto
ContainerLogV2
| where TimeGenerated > ago(1h)
| where PodName contains "demoapi"
| project TimeGenerated, PodName, LogMessage
| order by TimeGenerated desc
| take 100
```

### SLI, SLO, and error budget

- **SLI:** measured service indicator, e.g. successful HTTP request fraction or p95 latency.
- **SLO:** target for an SLI over a time window (e.g. 99.9% success rate in 30 days).
- **Error budget:** tolerated noncompliance within an SLO window; helps inform release pace and incident response.
- **SLA:** contractual agreement with service credits/commitments, not the same as your internal SLO.
- **Golden signals:** latency, traffic, errors, saturation.
- **RED method:** rate, errors, duration; **USE:** utilization, saturation, errors.

### Incident response loop

```text
Alert → Validate customer impact → Declare incident / owner
      → Mitigate (rollback, scale, fail over) → Verify recovery
      → Gather evidence → Root-cause analysis → Prevent recurrence
```

**High-value alerts:** error-budget burn rate, p95/p99 degradation, unexpected traffic, 5xx count, pod restarts/OOMKills, saturation, low storage, missing telemetry, failed releases, certificate expiry. Avoid alert storms and arbitrary no-signal thresholds.

---

## 16. DevSecOps and production-ready patterns

### Security gates through the software-delivery chain

```text
Developer workstation
 └─ Pre-commit / secret scanning
PR pipeline
 ├─ SAST: Coverity/SonarQube
 ├─ SCA/OSS: Black Duck or equivalent
 ├─ Tests and lint
 └─ Terraform/IaC policy validation
Protected main build
 ├─ Reproducible build + SBOM
 ├─ Container scan + immutable digest
 └─ Provenance/signing as supported
Release pipeline
 ├─ Federated service connection (no static password)
 ├─ Artifact/branch checks + approval
 ├─ Controlled deployment + smoke checks
 └─ Azure Monitor / SLO validation
```

### Twelve engineering best practices

1. Define resource naming, environment boundaries, and tagging conventions.
2. Use least-privilege RBAC at resource scopes, not wildcard Contributor everywhere.
3. Prefer short-lived OIDC/federated identities and managed identities.
4. Use protected branches and trustworthy production approval/check resources.
5. Promote immutable versioned images or digests; build only once per version.
6. Make CI jobs deterministic; pin dependency and task/action versions appropriately.
7. Build as non-root; scan SAST/SCA/images/IaC, but triage findings by severity and exploitability.
8. Keep secrets out of Git, logs, plan artifacts, and Docker image layers.
9. Treat Terraform state and build provenance as sensitive assets.
10. Design networking for private data access, controlled egress, and auditability.
11. Observe deployment health with objective SLOs, runbooks, and tested recovery.
12. Implement budgets, TTLs, environment teardown, retention, and right-sized capacity.

### Multi-environment example

| Property | Dev | Staging | Production |
|---|---|---|---|
| Source branch | Main/feature as policy permits | Protected main/release | Protected version/tag |
| Identity | Dev-scoped federated identity | Stage-scoped identity | Prod-scoped identity |
| Resource scope | Dev resource group/subscription | Separate stage | Prefer separate prod subscription |
| Approval | Usually automated | QA/change check | Restricted, auditable gate |
| Rollout | Fast replace | Production-like validation | Progressive/blue-green with metrics |
| Secrets | Dev Key Vault | Stage Key Vault | Prod Key Vault; separate permissions |

---

## 17. Common production failures and solutions

| Symptom | Likely cause | First checks / fix |
|---|---|---|
| Pipeline remains queued | No eligible agent, agent offline, parallel-job limits | Agent pool, queue, demands, concurrency allocation |
| `AuthorizationFailed` | Wrong scope/role/service connection principal | `az account show`, principal object ID, `az role assignment list`; verify correct subscription and propagation |
| AzureCLI task login fails | WIF issuer/subject/audience mismatch, connection not authorized | Inspect service connection, Entra federated credential, pipeline authorization, clock/token expiry |
| ACR push/build denied | Missing push/build permission, firewall restrictions, wrong registry | Registry login server, role model (RBAC/ABAC), build task logs, network |
| Pod `ImagePullBackOff` | Missing pull rights/tag, DNS/network outage, bad image reference | `kubectl describe pod`, `kubectl get events`, ACR tags, kubelet identity permissions |
| Pod `CrashLoopBackOff` | Application crash/exit, wrong command, failed liveness | `kubectl logs --previous`, events, image entrypoint, env/config |
| Pod stuck `Pending` | No scheduling capacity, requests too high, taints, PVC issue | `kubectl describe pod`, node allocatable/capacity, quotas, tolerations |
| App Service returns 502/503 | App not started, wrong startup/port, dependency failure | Deployment logs, app logs, health, startup command, network reachability |
| Private Key Vault timeout | DNS/network/firewall/private endpoint configuration | Private DNS resolution, effective routes, private endpoint, self-hosted agent reachability |
| Terraform state locked | Another apply, crashed writer, stale lease | Verify no active writer, inspect state backend, carefully recover only if safe |
| Terraform changed unexpected resources | Wrong subscription/workspace/state/drift | `terraform plan`, env vars, backend key, Azure activity logs |
| AKS API inaccessible from pipeline | Private endpoint/no path, firewall, broken DNS | Use connected self-hosted agent and resolve private DNS, verify authorized network |
| New release increases 5xx | Bad build/config/schema change | Stop rollout, compare deployment, rollback known-good version and verify DB compatibility |
| Azure cost unexpectedly high | Running AKS nodes, gateways, log ingestion, egress | Cost Analysis, budgets, log retention, idle resource cleanup |

### Realistic diagnosis: `ImagePullBackOff`

```bash
kubectl -n demo get pods
kubectl -n demo describe pod <pod>
az acr repository show-tags -n "$ACR_NAME" --repository demoapi -o table
az aks show -g "$RG" -n "$AKS_NAME" --query identityProfile -o json
```

Distinguish **tag does not exist** from **Unauthorized**, **DNS resolution**, and **registry firewall** errors. Never solve registry authentication by globally opening everything or baking credentials into Docker images.

### Realistic diagnosis: pipeline runs on wrong agent

Check the most specific `pool` declaration at pipeline/stage/job, hosted image label, pool permissions, agent demands and capabilities. For a network-restricted workload, a Microsoft-hosted agent may not reach your private Azure endpoint; move only the necessary jobs to a secure self-hosted pool with explicit authorization.

## 18. AWS ↔ Azure quick comparison

These are *conceptual* equivalents, not perfect feature-for-feature matches.

| AWS | Azure | Note |
|---|---|---|
| AWS Account / Organizations | Subscription / Management Groups | Tenant/identity and billing hierarchy differ |
| IAM / IAM Identity Center | Azure RBAC + Microsoft Entra ID | AuthN vs AuthZ boundaries |
| EC2 | Virtual Machines | VM compute |
| Auto Scaling Groups | Virtual Machine Scale Sets | VM fleet scaling |
| S3 | Blob Storage | Object storage |
| EBS | Managed Disks | Block volumes |
| EFS / FSx | Azure Files / Azure NetApp Files | Managed file services |
| VPC | VNet | Virtual networking |
| Security Groups | Network Security Groups | Stateful traffic rule concepts |
| Route 53 | Azure DNS | DNS management; Traffic Manager provides DNS routing |
| CloudFront | Azure Front Door / Azure CDN offerings | Edge delivery; features differ |
| ELB / ALB / NLB | Application Gateway / Load Balancer | L7 vs L4 |
| Direct Connect | ExpressRoute | Dedicated hybrid connectivity |
| ECR | Azure Container Registry | Container image registry |
| EKS | AKS | Managed Kubernetes |
| ECS / Fargate | Azure Container Apps / AKS and other offerings | Different orchestration models |
| Elastic Beanstalk / App Runner | App Service / Container Apps | PaaS app hosting; not 1:1 |
| Lambda | Azure Functions | Serverless execution |
| RDS | Azure Database for PostgreSQL / MySQL / Azure SQL | Managed database |
| DynamoDB | Cosmos DB | NoSQL models vary |
| CloudWatch / X-Ray | Azure Monitor / Application Insights | Telemetry and tracing |
| CloudTrail | Azure Activity Log and Entra audit logs | Control-plane and identity audit |
| KMS / Secrets Manager | Key Vault | Keys, secrets, certificates |
| CloudFormation / CDK | ARM / Bicep | Native IaC tooling |
| CodePipeline / CodeBuild | Azure Pipelines | CI/CD |
| CodeArtifact | Azure Artifacts | Package feeds |
| Systems Manager | Azure Automation / VM Run Command / Azure Arc | Operations tooling |
| AWS Config / Organizations guardrails | Azure Policy / Management Groups | Governance |

**Important distinction:** Terraform works with both clouds but providers/resources/permission models differ. Similarly, Jenkins, GitLab CI, GitHub Actions, SonarQube, Coverity, Black Duck, Docker, Helm, Argo CD, and Prometheus are portable across Azure and other environments.

---

## 19. Interview questions with answers

> **Interview technique:** Give a definition, identify the trade-off, and then add a practical example. For scenario questions, describe **impact → evidence → mitigation → prevention**. Do not claim to have operated a service in production if your experience is only in a lab.

### A. Azure fundamentals (Q1–Q15)

**Q1. What is Azure? How is it different from Azure DevOps?**  
**A:** Azure hosts compute, networking, identity, storage, and managed services. Azure DevOps manages work tracking, source control, packages, testing, and CI/CD. Azure Pipelines can deploy outside Azure too.

**Q2. What is a resource group?**  
**A:** A logical lifecycle and authorization scope for Azure resources. Deleting it deletes contained resources, but resource dependencies can span groups.

**Q3. Tenant vs subscription vs resource group?**  
**A:** Tenant is an identity directory, subscription is a resource/billing/quota boundary, and resource group is an organizational/deployment scope within a subscription.

**Q4. What is Azure Resource Manager?**  
**A:** The control-plane service/API through which Azure resources are created and managed. Portal, CLI, Bicep, ARM templates, and many Terraform provider operations use ARM.

**Q5. Region vs availability zone?**  
**A:** A region is a geographic cloud location; an availability zone is an isolated datacenter fault domain within a supported region. Zones mitigate some datacenter faults; cross-region DR covers a broader failure domain.

**Q6. What is Azure App Service?**  
**A:** Managed application hosting for web apps/APIs with deployment, configuration, scaling, and observability without directly managing VMs/Kubernetes nodes.

**Q7. When choose AKS instead of App Service?**  
**A:** Choose AKS for full Kubernetes API, complex orchestration, sidecars/operators, advanced networking, or organization-wide Kubernetes platforms. Choose App Service for simpler managed web hosting when those controls are unnecessary.

**Q8. Azure Container Apps vs AKS?**  
**A:** Container Apps abstracts most cluster operations for application workloads; AKS provides a managed Kubernetes control plane with more explicit cluster/node operations and ecosystem flexibility.

**Q9. What is Azure Functions?**  
**A:** Event-triggered serverless compute for HTTP, message, timer, or storage events. Design for retries, idempotence, plan constraints, and downstream failures.

**Q10. What is ACR?**  
**A:** Azure's private artifact/container registry used for OCI images and related artifacts. Pipelines build/push; runtime pull identities retrieve images.

**Q11. What is Azure Storage?**  
**A:** A family including Blob object storage, Azure Files shares, queues, tables, and disks provided through related offerings. Common uses: archives, backup, Terraform state, application content.

**Q12. Blob hot vs cool/archive?**  
**A:** Access tiers optimize different frequency/retention patterns; cooler/archive tiers generally favor lower storage charges but higher access/retrieval constraints. Choose based on access, latency, and lifecycle requirements.

**Q13. IaaS vs PaaS vs SaaS?**  
**A:** IaaS gives virtualized infrastructure with OS administration; PaaS abstracts more platform management; SaaS is a fully delivered application with customer data/configuration responsibilities.

**Q14. What are resource tags?**  
**A:** Metadata such as `owner`, `environment`, `costCenter`, and `application` for inventory/governance/cost attribution; enforce appropriate tags with policy.

**Q15. How would you control Azure costs?**  
**A:** Tag and budget resources, size capacity using telemetry, use autoscaling/schedules, optimize storage/log retention, choose suitable pricing tiers, identify idle resources, and implement lab cleanup.

### B. Networking and security (Q16–Q30)

**Q16. What is VNet and subnet?**  
**A:** VNet is an isolated address space in Azure; a subnet divides it into smaller routed segments for services and security policies.

**Q17. What does NSG do?**  
**A:** Applies stateful traffic filtering rules to subnets or NICs, using priority/order and directional settings. It is not an application-layer WAF.

**Q18. NSG vs Azure Firewall?**  
**A:** NSG offers L3/L4 network flow filtering at NIC/subnet boundaries; Azure Firewall is a centralized managed firewall with additional filtering, logging, and routing patterns.

**Q19. Public endpoint vs private endpoint?**  
**A:** A public endpoint is addressable via a public service address, potentially access-controlled; a private endpoint maps a supported service into a VNet using a private IP. DNS must resolve correctly for private access.

**Q20. Service endpoint vs private endpoint?**  
**A:** Service endpoints extend VNet identity to select public PaaS endpoints over Azure backbone; private endpoints create private IP entry points into the service and work with private DNS.

**Q21. Application Gateway vs Azure Load Balancer?**  
**A:** Application Gateway handles regional application-layer web routing, often with WAF; Load Balancer routes TCP/UDP at L4.

**Q22. Front Door vs Traffic Manager?**  
**A:** Front Door proxies global HTTP(S) edge traffic with L7 routing/security/caching; Traffic Manager is DNS-based traffic steering rather than an inline proxy.

**Q23. What is Microsoft Entra ID?**  
**A:** Azure's identity and access-management directory for users and applications, supporting authentication, conditional access, and identity federation. Azure RBAC separately grants resource permissions.

**Q24. Authentication vs authorization?**  
**A:** Authentication proves identity; authorization decides allowed operations. `az login` authenticates, while RBAC determines which resources/actions are available.

**Q25. Service principal vs managed identity?**  
**A:** Both are workload principals. Service principals can represent apps across systems; managed identities simplify credentials for eligible Azure resources, with Azure managing credentials behind the scenes.

**Q26. What is workload identity federation?**  
**A:** An external trusted OIDC token (such as Azure Pipelines or GitHub Actions) is exchanged for short-lived Azure access, removing the need for a stored long-lived client secret. Trust claims and RBAC must match.

**Q27. How do you grant a pipeline access to Azure?**  
**A:** Create an Azure Resource Manager service connection with workload identity federation, assign limited Azure roles/scopes, and explicitly authorize the pipeline to use the connection.

**Q28. What is Key Vault and how would an app read a secret?**  
**A:** Key Vault stores secrets/keys/certificates. Give the app an appropriate managed identity and Key Vault data-plane role, then retrieve at runtime using an SDK or platform integration.

**Q29. Azure Policy vs Azure RBAC?**  
**A:** RBAC controls *who* can take actions. Azure Policy controls *which resource states/configurations* are permitted or audited, independent of who requests them.

**Q30. A VM in a private subnet cannot reach a storage account. Diagnose.**  
**A:** Validate DNS, NSG rules, effective routes, firewall restrictions, outbound path/NAT, storage network ACLs, private endpoint DNS, and the VM's storage data-plane RBAC. Connectivity and permissions are distinct tests.

### C. Azure DevOps and CI/CD (Q31–Q50)

**Q31. Name the five Azure DevOps services.**  
**A:** Boards, Repos, Pipelines, Artifacts, and Test Plans.

**Q32. Azure Pipelines vs GitHub Actions?**  
**A:** Both automate CI/CD. Pipelines integrates with Azure DevOps Repos/environments/service connections; GitHub Actions lives within GitHub and uses workflows/actions. Both can authenticate securely to Azure.

**Q33. Stage vs job vs step?**  
**A:** Stage groups delivery phases, job groups agent-bound execution, step is a script/task within a job.

**Q34. Microsoft-hosted vs self-hosted agent?**  
**A:** Hosted agents are disposable and maintained by Microsoft; self-hosted agents offer private network access/custom dependencies, but require maintenance, isolation, and patching.

**Q35. How does Azure Repos PR validation work?**  
**A:** Configure branch policy/build validation for protected branches. Azure Repos does not use YAML `pr:` as the trigger for PR validation; GitHub and Bitbucket can use YAML PR triggers.

**Q36. What are pipeline variables and parameters?**  
**A:** Parameters are typed compile-time configuration; variables are mostly strings referenced at runtime or macro expansion. Secret variables require safe handling.

**Q37. `${{ }}` vs `$(var)` vs `$[ ]`?**  
**A:** Template-time expressions, macro variables, and runtime expressions, respectively. Mixing the phases can create unexpected values or conditions.

**Q38. How can multiple jobs share a package?**  
**A:** Publish an artifact from producer job, then download it in a downstream dependent job. For container workloads, publish to ACR and deploy an immutable version/digest.

**Q39. Azure Artifacts vs Pipeline Artifacts vs ACR?**  
**A:** Azure Artifacts hosts language package feeds; Pipeline Artifacts stores build/run files; ACR hosts OCI images/artifacts.

**Q40. How do you configure production approval?**  
**A:** Create a protected environment and/or protected service connection with approvals/checks managed by authorized resource owners. Gate use of prod credentials and restrict branch/template usage; don't depend only on a YAML script prompt.

**Q41. What is a deployment job?**  
**A:** A job designed for deploying to an environment, recording deployment history, and supporting deployment strategies/checks.

**Q42. How do you build once and deploy many times?**  
**A:** Produce and scan a versioned artifact/image in CI; promote its immutable digest through dev/stage/prod, keeping environment-specific runtime settings separate.

**Q43. How do you implement rollback in Azure Pipelines?**  
**A:** Keep known-good version metadata, deploy the prior digest/package or swap App Service slots where valid, verify health, and coordinate database/config compatibility. Rollback is a workflow, not simply `git revert`.

**Q44. What is a service connection?**  
**A:** Azure DevOps' configured/authenticated connection to an external resource such as Azure Resource Manager, registry, or Kubernetes. It is permissioned independently from repository access.

**Q45. What does `AzureCLI@2` do?**  
**A:** Runs Azure CLI shell/PowerShell commands authenticated through an ARM service connection, typically with scoped temporary credentials.

**Q46. What does `KubernetesManifest@1` do?**  
**A:** Applies Kubernetes manifests and supports actions such as deploy, bake, canary-related strategies, and image substitutions. With AKS, ARM-based service connections are commonly preferred to long-lived Kubernetes tokens.

**Q47. How do you pass credentials to a pipeline?**  
**A:** Prefer OIDC federation, managed identity, and Azure Key Vault. Use secret variable groups only when appropriate and never echo credentials. Ensure task/service-connection authorization is restricted.

**Q48. What is a pipeline cache?**  
**A:** Reusable dependency/build cache intended to accelerate jobs; unlike a release artifact, it is not the authoritative distributable and should not be trusted blindly across untrusted sources.

**Q49. Your pipeline succeeded, but deployment is unhealthy. Is CI/CD successful?**  
**A:** A green CI run only proves configured checks passed. The release must include application health, rollout verification, telemetry validation, and rollback criteria; otherwise deployment success is insufficient evidence.

**Q50. How would you migrate Jenkins pipelines to Azure Pipelines?**  
**A:** Inventory Jenkinsfile stages, credentials, agents, plugins, triggers, artifact contracts, and approvals; map to YAML stages/jobs/templates, service connections, pools, artifacts, and checks; pilot one application; validate parity and metrics; gradually cut over.

### D. Containers, Kubernetes, and GitOps (Q51–Q65)

**Q51. Docker image vs running container?**  
**A:** Image is an immutable packaged filesystem/configuration; container is a running instance with isolated process resources built from the image.

**Q52. What is AKS?**  
**A:** Azure Kubernetes Service supplies a managed Kubernetes control plane with worker node pools and integrations; teams still manage workload security, scaling, policy, and updates.

**Q53. What is a Pod?**  
**A:** The smallest schedulable Kubernetes unit, containing one or more tightly coupled containers sharing network namespace and storage volumes.

**Q54. Deployment vs StatefulSet?**  
**A:** Deployments suit stateless replicated workloads with rolling updates; StatefulSets supply stable identities/storage associations for stateful workloads.

**Q55. Service vs Ingress?**  
**A:** Service provides stable network discovery/load balancing for Pods; Ingress expresses HTTP(S) routing rules implemented by a controller. Modern Gateway API can also be used.

**Q56. Readiness vs liveness vs startup probe?**  
**A:** Readiness decides if a Pod receives traffic; liveness restarts unhealthy containers; startup postpones liveness/readiness assessment while slow initialization completes (subject to probe configuration).

**Q57. HPA vs Cluster Autoscaler?**  
**A:** HPA adjusts Pod replica counts based on metrics; Cluster Autoscaler adjusts node capacity to schedule Pods when supported and configured. They solve different capacity layers.

**Q58. What causes `ImagePullBackOff`?**  
**A:** Bad image/tag, unauthorized ACR pull, image missing, registry DNS/network/firewall, or throttling. Inspect Pod events, repo tags, kubelet identity, and network configuration.

**Q59. What causes `CrashLoopBackOff`?**  
**A:** The container repeatedly exits or is killed; check `kubectl logs --previous`, events, exit code, probes, memory limits, config, and dependency availability.

**Q60. Why is `latest` discouraged for production?**  
**A:** It is mutable and hides what was deployed; immutable digests/uniquely versioned tags provide provenance and reliable rollback.

**Q61. What is Helm?**  
**A:** Package/template/release management for Kubernetes applications using charts and values. `helm upgrade --install` applies a release; `helm rollback` returns to a prior Helm revision.

**Q62. What is GitOps?**  
**A:** Desired infrastructure/app state is versioned in Git, and a controller such as Argo CD or Flux continuously reconciles a cluster to that desired state.

**Q63. Argo CD vs Azure Pipelines CD?**  
**A:** Argo CD uses pull-based reconciliation of Git desired state; Azure Pipelines usually runs a push-based deployment job. Combining them is common when CI updates a GitOps repo and Argo handles rollout.

**Q64. A Pod can reach the public internet but not a private Azure database. Why?**  
**A:** Check private endpoint/DNS, VNet routing/peering, network policy, outbound firewall, database ACLs, TCP port, TLS, and database credentials. Internet reachability says little about private routing.

**Q65. How do you deploy with zero or minimal downtime?**  
**A:** Multiple healthy replicas, rolling/blue-green/canary deployment, readiness gates, capacity headroom, safe database migrations, graceful termination, and progressive traffic monitoring.

### E. IaC, monitoring, and SRE (Q66–Q80)

**Q66. What is Terraform state?**  
**A:** The mapping between configuration and real resources, including resource attributes. It is sensitive operational data; protect, lock, back up, and scope access.

**Q67. Terraform `plan` vs `apply`?**  
**A:** `plan` computes proposed changes from configuration/state/live reads; `apply` performs changes. Review plan output and lock down production apply permissions.

**Q68. How do you prevent concurrent Terraform changes?**  
**A:** Use remote state with locking and serialize production apply jobs; ensure separate states/workspaces where appropriate. Never force-unlock without verifying no legitimate writer remains.

**Q69. Terraform vs Bicep?**  
**A:** Terraform is multi-provider/multi-cloud and uses external state; Bicep is Azure-native and deploys through ARM with `what-if` preview.

**Q70. What is drift?**  
**A:** Differences between desired IaC definition, saved state, and actual resources, often caused by manual edits or external automation. Detect via plans and governance, then reconcile deliberately.

**Q71. What does Azure Monitor do?**  
**A:** Brings together metrics/logs/alerts/insights for Azure resources and apps. Application Insights provides app telemetry, while Log Analytics supports KQL log queries.

**Q72. Metrics vs logs vs traces?**  
**A:** Metrics are time-series numeric summaries; logs are event records; distributed traces track a request across services with spans and correlation IDs.

**Q73. What is KQL?**  
**A:** Kusto Query Language, used to query and summarize Azure Monitor Logs/Log Analytics (and other Kusto-backed services). Filtering early and limiting output controls query cost/performance.

**Q74. What are SLI, SLO, and SLA?**  
**A:** SLI is measurement; SLO is the internal target for it; SLA is a contractual reliability commitment. Error budget quantifies permitted miss relative to the SLO.

**Q75. How do you reduce MTTR?**  
**A:** Reliable alerts, correlated logs/traces/metrics, clear runbooks, deployment metadata, rollback automation, incident ownership, and post-incident corrective work.

**Q76. You detect 5xx errors immediately after a production deployment. First action?**  
**A:** Assess customer impact and freeze/stop rollout, restore service quickly via rollback/traffic shift if indicated, verify recovery, then investigate release diff, logs, and dependencies.

**Q77. What is an error budget?**  
**A:** The tolerated amount of service unreliability for an SLO period. Sustained budget burn can justify halting risky releases and prioritizing reliability improvements.

**Q78. How do you monitor AKS?**  
**A:** Collect node/pod/resource metrics and cluster events, application logs/traces, request SLIs, and deployment metadata using Azure Monitor integrations, Managed Prometheus/Grafana, Container Insights, and/or approved observability tooling.

**Q79. How do you investigate a sudden Azure bill increase?**  
**A:** Use Cost Analysis segmented by service, group, region, tags, and time; correlate with changes in AKS node count, premium SKUs, outbound bandwidth, log ingestion, unattached resources, and forecasts.

**Q80. How would you design an enterprise Azure CI/CD platform?**  
**A:** Federated scoped identities; reusable pipeline templates; branch/PR policy; private/self-hosted agents where necessary; approved artifact registries; scan/SBOM/signing; Terraform/Bicep; isolated environments; protected releases; progressive rollouts; SLO monitoring and cost controls.

---

## 20. Mock interview scenarios and practice labs

### Scenario 1 — A green build produced a broken production release

**Question:** CI tests passed; after CD, error rate spikes and p95 latency doubles. What do you do?

**Model answer:** Declare/triage impact; correlate timing with deployed image digest and config changes; pause progressive rollout; shift traffic or deploy the last known-good digest; inspect Azure Monitor/App Insights, container logs, downstream dependencies, and database migrations; verify golden signals recover; communicate incident status; follow up with staged smoke/canary tests and rollout error-budget checks.

### Scenario 2 — Private AKS cluster, hosted agent can't deploy

**Question:** Azure Pipelines `KubernetesManifest@1` fails with a network timeout, but your service connection is valid.

**Model answer:** Separate authorization from reachability: a Microsoft-hosted runner is not automatically routed to a private AKS API. Use a hardened self-hosted agent in an allowed VNet/network, verify private DNS, NSGs and routing, choose the ARM-based service connection for AKS, and grant appropriate Kubernetes RBAC. Confirm access before re-running the deployment.

### Scenario 3 — `terraform apply` says authorization failed

**Question:** A pipeline can list the resource group but cannot create a storage account or assign a role.

**Model answer:** Reader may list; resource creation needs an appropriate management-plane role. Grant a narrow role on the intended group. Role assignments require separate permission such as User Access Administrator/Owner or an appropriately scoped custom role; do not give Owner just to work around an unrelated authorization issue. Confirm provider subscription ID and backend data-plane rights independently.

### Scenario 4 — Images build, but AKS cannot pull

**Question:** Azure Pipelines publishes image `v250`, but Pods show `ImagePullBackOff`.

**Model answer:** Read Pod events, verify full registry hostname/repository/tag, inspect ACR push result, check AKS kubelet identity pull permissions and registry RBAC/ABAC mode, private DNS/network/firewall access, and retry safely after fixing root cause.

### Scenario 5 — The on-prem build pipeline must migrate to Azure

**Question:** You manage Jenkins/Gerrit, large AOSP builds, static analysis, and an artifact repository. What is the Azure strategy?

**Model answer:** Keep large specialized AOSP builds on approved self-hosted agents when hosted agents lack capacity/network access; translate Groovy pipeline phases to Azure YAML templates, establish source triggers/branch policies, use federated Azure service connections, preserve artifact provenance/IDs, integrate Coverity/Black Duck via supported command-line tasks, publish to Azure Artifacts/ACR or retain the existing binary repository where necessary, and roll out incrementally with parity checks.

### Scenario 6 — Interview design problem: multi-region SaaS

**Question:** Design a resilient Azure web API serving customers worldwide.

**Model answer:** Consider Front Door/WAF as global ingress, deploy stateless workloads across at least two regions using App Service/AKS as needed, zone resilience where supported, data replication/consistency strategy, private database access, Key Vault/managed identities, deployment rollout per region, telemetry/SLO and failover drills. Explain RTO/RPO and whether data dependencies can genuinely fail over.

### Scenario 7 — Secrets leak into a CI job log

**Question:** A developer accidentally echoes a service principal password.

**Model answer:** Revoke/rotate credential, assess log/artifact exposure and access, remove sensitive traces under policy, update identity to federation or managed identity, restrict access, and add secret scanning, safe scripting, and incident documentation. A redacted UI does not prove the secret was never exposed.

### Practice lab checklist

- [ ] Explain tenant, subscription, resource group, RBAC and service connection without notes.
- [ ] Use Azure CLI to create and enumerate a disposable resource group.
- [ ] Make a `main`-triggered Azure Pipeline run `pytest` and publish test results.
- [ ] Provision an ACR; build and push a uniquely tagged Docker image.
- [ ] Deploy to AKS using an ARM service connection and observe rollout health.
- [ ] Introduce an intentionally invalid image tag and diagnose `ImagePullBackOff`.
- [ ] Roll back to a known-good image and validate application health.
- [ ] Write a Terraform plan, inspect it, apply to a disposable RG, and destroy safely.
- [ ] Deploy one storage resource using Bicep `what-if` and `create`.
- [ ] Use KQL to compute 5-minute error ratio and p95 latency.
- [ ] Demonstrate workload identity federation without a long-lived secret.
- [ ] Explain how to add protected production releases and immutable promotions.

### 21-day preparation roadmap

| Days | Focus | Evidence you should produce |
|---|---|---|
| 1–3 | Tenant, subscriptions, RBAC, regions, VNet, DNS | Architecture sketch and 20 CLI commands |
| 4–6 | Azure Repos, Boards, pipeline YAML, hosted agents | PR validation + multistage YAML |
| 7–9 | ACR, Docker, App Service and AKS | Versioned image and healthy web endpoint |
| 10–12 | AKS debugging, Helm, deployment strategies | Rolling upgrade + rollback demo |
| 13–15 | Terraform, state backend, Bicep, policies | IaC repo with plan/what-if outputs |
| 16–18 | Key Vault, OIDC, DevSecOps, telemetry/KQL | Safe service connection + logs/alerts |
| 19–21 | Scenarios, mock interviews, cost controls | 10-minute architecture walkthrough and incident answers |

**Portfolio description template (adapt only to work you've genuinely done):**

> Implemented a multi-stage Azure CI/CD lab with Azure Pipelines, unit tests, Azure Container Registry, and AKS. Secured deployments through workload identity federation and scoped service connections, deployed versioned container images, and validated application health and rollback. Supplemented the platform with Terraform/Bicep and Azure Monitor/KQL troubleshooting.

---

## 21. Learning roadmap and official documentation

The links below are official primary references. Validate product versions, availability, permissions, and costs before production use.

### Azure foundations

- [Azure documentation](https://learn.microsoft.com/azure/)
- [Azure CLI reference](https://learn.microsoft.com/cli/azure/)
- [Microsoft Entra ID documentation](https://learn.microsoft.com/entra/identity/)
- [Azure RBAC](https://learn.microsoft.com/azure/role-based-access-control/overview)
- [Azure network load balancing guide](https://learn.microsoft.com/azure/architecture/guide/technology-choices/load-balancing-overview)
- [Azure Well-Architected Framework](https://learn.microsoft.com/azure/well-architected/)
- [Azure Cost Management](https://learn.microsoft.com/azure/cost-management-billing/)

### Azure Pipelines and deployment

- [Azure Pipelines concepts](https://learn.microsoft.com/azure/devops/pipelines/get-started/key-pipelines-concepts?view=azure-devops)
- [Azure Pipelines YAML schema](https://learn.microsoft.com/azure/devops/pipelines/yaml-schema/?view=azure-pipelines)
- [Build YAML pipelines with stages](https://learn.microsoft.com/azure/devops/pipelines/process/run-stages?view=azure-devops)
- [Microsoft-hosted agents and image labels](https://learn.microsoft.com/azure/devops/pipelines/agents/hosted?view=azure-devops)
- [Connect to Azure with a federated service connection](https://learn.microsoft.com/azure/devops/pipelines/library/connect-to-azure?view=azure-devops)
- [Pipeline approvals and checks](https://learn.microsoft.com/azure/devops/pipelines/process/approvals?view=azure-devops)
- [Pipeline environments](https://learn.microsoft.com/azure/devops/pipelines/process/environments?view=azure-devops)
- [AzureCLI@2 task](https://learn.microsoft.com/azure/devops/pipelines/tasks/deploy/azure-cli?view=azure-devops)
- [AzureWebApp@1 task](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/azure-web-app-v1?view=azure-pipelines)
- [KubernetesManifest@1 task](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/kubernetes-manifest-v1?view=azure-pipelines)
- [Pipeline artifacts](https://learn.microsoft.com/azure/devops/pipelines/artifacts/pipeline-artifacts?view=azure-devops)
- [Key Vault linked variable groups](https://learn.microsoft.com/azure/devops/pipelines/library/link-variable-groups-to-key-vaults?view=azure-devops)

### Containers, IaC, security and monitoring

- [ACR build tasks](https://learn.microsoft.com/azure/container-registry/container-registry-tasks-overview)
- [ACR and AKS integration](https://learn.microsoft.com/azure/aks/cluster-container-registry-integration)
- [Deploy AKS with Azure Pipelines](https://learn.microsoft.com/azure/aks/devops-pipeline)
- [AKS documentation](https://learn.microsoft.com/azure/aks/)
- [Terraform Azure provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
- [Terraform provider v4 upgrade notes](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/4.0-upgrade-guide)
- [Bicep documentation](https://learn.microsoft.com/azure/azure-resource-manager/bicep/overview)
- [GitHub Actions + Azure OIDC](https://learn.microsoft.com/azure/developer/github/connect-from-azure-openid-connect)
- [Azure Monitor documentation](https://learn.microsoft.com/azure/azure-monitor/)
- [AppRequests KQL examples](https://learn.microsoft.com/azure/azure-monitor/reference/queries/apprequests)
- [Cloud adoption and DevOps security considerations](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/considerations/security-considerations-overview)

---

## Final interview revision: 15 one-liners

1. Azure is the cloud; Azure DevOps is an ALM/CI/CD platform.
2. Azure Pipelines YAML is the closest direct analog to declarative Jenkinsfiles.
3. Azure Repos PR validation is implemented with branch build-validation policies.
4. A service connection is not a magic Administrator token: scope its Azure identity.
5. Use OIDC/federation rather than static service principal secrets where possible.
6. Resource-group Contributor and Kubernetes RBAC are **not** equivalent.
7. VNet and DNS problems often masquerade as application failures.
8. ACR stores versioned images; AKS orchestrates their Pods.
9. A running Pod is not necessarily ready to serve traffic.
10. Artifact promotion should preserve the same digest across environments.
11. Terraform needs secure remote state and controlled applies.
12. Bicep is Azure-native IaC; Terraform is provider-based, multi-cloud IaC.
13. `ImagePullBackOff` starts with Pod events, tag verification, registry rights, and network.
14. An SLO needs an SLI and a defined measurement window.
15. A production deployment is complete only when health, rollback, and telemetry checks pass.

**End of guide.**

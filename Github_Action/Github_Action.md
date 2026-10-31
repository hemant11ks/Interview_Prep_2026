# GitHub Actions — Complete DevOps / SRE / Cloud Interview Guide

> **Level:** Beginner → Advanced | **Focus:** CI/CD, AWS, Docker, Kubernetes, Terraform, DevSecOps, reliability, troubleshooting, interview scenarios  
> **Last reviewed:** 31 October 2026 | **Practice style:** Concepts + runnable examples + interview-ready answers

**How to use this guide:** Read Sections 1–9 to establish fundamentals; build the practical pipelines in Sections 10–19; study security and operations in Sections 20–26; rehearse questions in Sections 27–30. Commands assume a Linux shell unless indicated otherwise. Replace placeholders such as `YOUR_ORG`, `YOUR_REPO`, account IDs, domains and role names before use.

> **Version note:** Major version tags (`@v7`, etc.) in this guide represent public action releases available when reviewed. **For production, pin third-party and critical actions to a vetted full commit SHA, review upgrades, and check your runner's minimum supported version.** GitHub-hosted runner software and action major versions evolve. Some enterprises use older GHES/runner versions and must select compatible action releases.

---

## Table of Contents

1. [GitHub Actions in 60 Seconds](#1-github-actions-in-60-seconds)
2. [Core Architecture and Execution Lifecycle](#2-core-architecture-and-execution-lifecycle)
3. [Workflow Files and YAML Syntax](#3-workflow-files-and-yaml-syntax)
4. [Events and Triggers](#4-events-and-triggers)
5. [Runners and Infrastructure](#5-runners-and-infrastructure)
6. [Jobs, Steps, Dependencies and Conditions](#6-jobs-steps-dependencies-and-conditions)
7. [Contexts, Expressions, Variables and Secrets](#7-contexts-expressions-variables-and-secrets)
8. [Outputs, Environment Files and Job Communication](#8-outputs-environment-files-and-job-communication)
9. [Matrix, Concurrency, Timeouts and Resilience](#9-matrix-concurrency-timeouts-and-resilience)
10. [Project 1: Python CI Pipeline](#10-project-1-python-ci-pipeline)
11. [Project 2: Java / Maven CI Pipeline](#11-project-2-java--maven-ci-pipeline)
12. [Project 3: Node.js CI with Matrix](#12-project-3-nodejs-ci-with-matrix)
13. [Artifacts vs Caches](#13-artifacts-vs-caches)
14. [Project 4: Docker Build and GHCR Push](#14-project-4-docker-build-and-ghcr-push)
15. [Project 5: AWS OIDC + ECR + EKS](#15-project-5-aws-oidc--ecr--eks)
16. [Project 6: Terraform Plan / Apply](#16-project-6-terraform-plan--apply)
17. [Deployment Environments, Approvals and Rollback](#17-deployment-environments-approvals-and-rollback)
18. [Reusable Workflows and Composite Actions](#18-reusable-workflows-and-composite-actions)
19. [Advanced Patterns and Microservice CI/CD](#19-advanced-patterns-and-microservice-cicd)
20. [Security and DevSecOps Best Practices](#20-security-and-devsecops-best-practices)
21. [Supply Chain, SBOM, SAST, SCA and Attestations](#21-supply-chain-sbom-sast-sca-and-attestations)
22. [Self-hosted Runners, ARC and Scaling](#22-self-hosted-runners-arc-and-scaling)
23. [Monitoring, Debugging and Troubleshooting](#23-monitoring-debugging-and-troubleshooting)
24. [Performance, Cost and Workflow Optimization](#24-performance-cost-and-workflow-optimization)
25. [GitHub CLI, Local Testing and Administration](#25-github-cli-local-testing-and-administration)
26. [GitHub Actions vs Jenkins vs GitLab CI](#26-github-actions-vs-jenkins-vs-gitlab-ci)
27. [Interview Questions: Beginner](#27-interview-questions-beginner)
28. [Interview Questions: Intermediate](#28-interview-questions-intermediate)
29. [Interview Questions: Advanced and Scenarios](#29-interview-questions-advanced-and-scenarios)
30. [Hands-on Interview Assignments and Model Answers](#30-hands-on-interview-assignments-and-model-answers)
31. [Frequently Seen Errors — Quick Fix Table](#31-frequently-seen-errors--quick-fix-table)
32. [Best Practices Checklist](#32-best-practices-checklist)
33. [7-Day Interview Preparation Plan](#33-7-day-interview-preparation-plan)
34. [One-Page Revision Cheat Sheet](#34-one-page-revision-cheat-sheet)
35. [Official Documentation and References](#35-official-documentation-and-references)

---

## 1. GitHub Actions in 60 Seconds

**GitHub Actions** is GitHub's event-driven automation platform for running CI/CD and other tasks directly from repository events. Define a **workflow** using YAML under `.github/workflows/`; an **event** starts the workflow; the workflow has **jobs**; a job executes **steps** on a **runner**; each step can run a shell command or a reusable **action**.

```text
Git push / Pull Request / Schedule / Manual trigger
                      |
                      v
             GitHub Actions event
                      |
                      v
           .github/workflows/ci.yml
                      |
             +--------+---------+
             |                  |
             v                  v
       Job: Lint          Job: Unit Tests
       Runner A           Runner B
             |                  |
             +---------+--------+
                       |
                 needs: [lint, test]
                       v
                  Build image
                       |
                 Deploy staging
                       |
              Approval / gates
                       v
                Deploy production
```

**Interview definition:** “GitHub Actions automates repository-triggered workflows such as builds, tests, security scans, container publishing and deployments using YAML-defined jobs executed on GitHub-hosted or self-hosted runners.”

**CI vs CD:** CI continuously validates changes (lint/build/test); continuous *delivery* keeps a release deployable with approvals; continuous *deployment* automatically releases successful changes to production.

### Vocabulary

| Term | What it means | Interview example |
|---|---|---|
| Workflow | YAML automation definition | `ci.yml` |
| Event | Triggering activity | `push`, `pull_request` |
| Run | One execution of a workflow | Run #125 |
| Job | Set of steps on one runner | `unit-tests` |
| Step | A command or an action | `run: pytest` |
| Action | Reusable automation unit | `actions/checkout` |
| Runner | Machine executing a job | `ubuntu-latest`, self-hosted |
| Marketplace | Catalog of actions | Docker and cloud actions |
| `GITHUB_TOKEN` | Short-lived GitHub API token scoped by permissions | Push to GHCR |
| Environment | Deployment target with policy / secrets | `production` |
| Artifact | Stored build output / evidence | Test report, binary |
| Cache | Reusable dependency data | pip, Maven or npm cache |
| OIDC | Identity federation for short-lived cloud auth | AWS IAM role |

---

## 2. Core Architecture and Execution Lifecycle

1. A repository event matches workflow filters.
2. GitHub queues a workflow run and evaluates jobs/conditions.
3. An eligible GitHub-hosted or self-hosted runner takes each runnable job.
4. Actions and shell steps execute in order **within** a job.
5. Jobs without dependencies may run **in parallel**; `needs:` introduces order.
6. Logs, outputs, artifacts, status checks and deployment records become available.
7. Protection rules and required checks can control merge/deployment eligibility.

**Important distinctions:**

- Jobs generally have isolated runner environments; files are **not automatically shared** between jobs. Use artifacts, caches when appropriate, external storage or job outputs for small values.
- Steps in the **same job** usually share the workspace and runner filesystem.
- A failed step normally fails its job and skips subsequent ordinary steps; `if: always()` is useful for cleanup/reporting (subject to cancellation and execution limitations).
- `needs:` dependencies allow data passing with `needs.job_id.outputs...`.
- An **action** is reusable step-level logic; a **reusable workflow** is reusable job/workflow-level orchestration.

---

## 3. Workflow Files and YAML Syntax

### Folder structure

```text
my-repository/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   ├── docker.yml
│   │   ├── deploy.yml
│   │   └── reusable-test.yml
│   └── dependabot.yml
├── app/
├── tests/
├── Dockerfile
└── README.md
```

### Minimal valid workflow

**File:** `.github/workflows/hello.yml`

```yaml
name: Hello Actions

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  say-hello:
    runs-on: ubuntu-latest
    steps:
      - name: Print details
        run: |
          echo "Hello from GitHub Actions"
          echo "Repository: $GITHUB_REPOSITORY"
          echo "Commit: $GITHUB_SHA"
```

### Main syntax keywords

| Keyword | Function |
|---|---|
| `name:` | Workflow, job or step display name |
| `run-name:` | Dynamic name for a specific workflow run |
| `on:` | Events that trigger the workflow |
| `permissions:` | Least-privilege `GITHUB_TOKEN` permissions |
| `env:` | Environment variables |
| `defaults:` | Default run shell / working directory |
| `jobs:` | Workflow's jobs |
| `runs-on:` | Runner selection |
| `steps:` | Ordered commands/actions in a job |
| `uses:` | Invokes an action or reusable workflow (job-level for workflow) |
| `run:` | Runs a shell command |
| `with:` | Inputs passed to an action or reusable workflow |
| `needs:` | Other jobs that must finish first |
| `if:` | Conditional expression |
| `strategy.matrix:` | Job combinations / parallel variations |
| `environment:` | Deployment environment |
| `concurrency:` | Controls overlapping runs/jobs |
| `timeout-minutes:` | Caps job/step execution duration |
| `services:` | Service containers for tests |
| `container:` | Executes a job inside a container |

**YAML tips:** Use spaces, not tabs; `-` marks list items; `|` is a multiline block. Quote ambiguous strings when needed. Most YAML parsers may interpret `on` unexpectedly under YAML 1.1; use GitHub-compatible tooling to validate workflow files.

---

## 4. Events and Triggers

### Most important events

```yaml
on:
  push:
    branches: [main, 'release/**']
    paths:
      - 'src/**'
      - '.github/workflows/**'
  pull_request:
    branches: [main]
  workflow_dispatch:
    inputs:
      target:
        description: Deployment target
        required: true
        type: choice
        options: [staging, production]
        default: staging
  schedule:
    - cron: '15 2 * * 1'  # 02:15 UTC every Monday
```

The example above combines **OR** trigger alternatives. To require multiple conditions, use event-specific filters and/or job `if:`. A scheduled workflow runs from the default branch; cron is evaluated in **UTC** (and can run late under load).

| Trigger | When to use it | Caution |
|---|---|---|
| `push` | Build on branch/tag changes | Filter branches/tags/paths |
| `pull_request` | PR verification | Fork PRs have restricted secrets/permissions |
| `workflow_dispatch` | Manually initiated run | Inputs can be validated and environment gated |
| `schedule` | Nightly scans, audits | UTC cron; default branch |
| `release` | Publish tagged release | Limit event types (e.g. `published`) |
| `workflow_call` | Reusable workflow | Called by another workflow |
| `workflow_run` | React after another workflow | Privilege-boundary danger with untrusted artifacts |
| `repository_dispatch` | External system triggers | Requires API authentication |
| `issue_comment` | ChatOps requests | Check trusted actor/permissions carefully |
| `pull_request_target` | Base-repo privileged PR metadata automation | **Do not execute untrusted PR code with privileged secrets/token** |

### Branch + tag filters

```yaml
on:
  push:
    branches:
      - main
      - 'release/**'
    # For tag-based workflows, use a separate push event configuration/workflow.
```

Example dedicated release workflow:

```yaml
name: Release Tag
on:
  push:
    tags: ['v*']
jobs:
  verify-tag:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building release $GITHUB_REF_NAME"
```

### Manual inputs and conditional job

```yaml
on:
  workflow_dispatch:
    inputs:
      deploy:
        description: Deploy after tests?
        type: boolean
        required: true
        default: false
jobs:
  decision:
    if: ${{ inputs.deploy }}
    runs-on: ubuntu-latest
    steps:
      - run: echo 'Deployment was requested'
```

### Avoid duplicate CI runs

If a push to a feature branch creates a PR, workflows listening to both `push` and `pull_request` may execute twice. A common policy is: test `push` only for `main`, and test `pull_request` for feature branches.

**Interview trap:** An `if:` condition **is not** an alternative event definition: a workflow must start before job conditions are evaluated.

---

## 5. Runners and Infrastructure

### GitHub-hosted runners

```yaml
runs-on: ubuntu-latest
# Or pin a specific supported image label for reproducibility:
# runs-on: ubuntu-24.04
```

GitHub provisions/manages the execution environment. Good for conventional builds and tests, with OS choices such as Ubuntu, Windows and macOS. Available CPU, disk, network, features and charges depend on plan and runner type.

### Self-hosted runner

```yaml
runs-on: [self-hosted, linux, x64, ci-build]
```

A self-hosted runner is an agent you install/manage on your VM, physical host or Kubernetes-based fleet. It can access private infrastructure but requires patching, isolation, hardened identities, network controls and operational monitoring.

| Comparison | GitHub-hosted | Self-hosted |
|---|---|---|
| Host provisioning | Managed by GitHub | Your responsibility |
| Tooling | Image has preinstalled tools | Custom image/host |
| Network access | Cloud-hosted | Private networks possible |
| Scaling | Mostly managed | You design / ARC can scale |
| Security risk | Mostly ephemeral job environment | Persistent-machine compromise / cross-job contamination |
| Good for | Standard web/app CI | Private builds, specialized HW, AOSP-sized workloads |

**Production advice:** Prefer ephemeral, single-job self-hosted runners for untrusted code; never casually expose a persistent runner in a privileged network to public pull requests.

---

## 6. Jobs, Steps, Dependencies and Conditions

```yaml
name: Job Dependencies
on: [push, workflow_dispatch]
permissions:
  contents: read

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - run: echo 'Linting...'

  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - run: echo 'Testing...'

  package:
    needs: [lint, unit-tests]
    runs-on: ubuntu-latest
    if: ${{ github.ref == 'refs/heads/main' }}
    steps:
      - run: echo 'Both required checks passed on main'
```

- `lint` and `unit-tests` can run in parallel.
- `package` runs only after both succeed *and* the branch condition is true.
- `if: failure()` is useful for failure diagnostics; `if: always()` is useful for publishing reports after failure. Prefer `if: ${{ !cancelled() }}` for some downstream orchestration where cancellation should be respected.
- `continue-on-error: true` on a step allows a failure to be tolerated. Don't use it to suppress critical security/test gates.
- `needs:` does **not** transfer files between jobs.

---

## 7. Contexts, Expressions, Variables and Secrets

### Common contexts

| Expression | Meaning |
|---|---|
| `${{ github.repository }}` | `owner/repository` |
| `${{ github.ref }}` | Fully qualified ref (`refs/heads/main`) |
| `${{ github.ref_name }}` | Short ref name |
| `${{ github.sha }}` | Commit SHA associated with event |
| `${{ github.event_name }}` | Trigger event name |
| `${{ github.actor }}` | User/app triggering run |
| `${{ github.run_id }}` | Unique workflow run ID |
| `${{ github.run_number }}` | Workflow's incrementing run number |
| `${{ matrix.python }}` | Matrix value |
| `${{ needs.build.outputs.image }}` | Output from dependency |
| `${{ steps.meta.outputs.tag }}` | Step output |
| `${{ inputs.environment }}` | Manual/reusable workflow input |
| `${{ vars.AWS_REGION }}` | Non-secret repository/org/environment configuration |
| `${{ secrets.MY_SECRET }}` | Secret value (where available) |
| `${{ runner.os }}` | Runner operating system |

### Variable scope

```yaml
name: Scope Example
on: workflow_dispatch
permissions:
  contents: read
env:
  APP_NAME: payments-api
jobs:
  show-scope:
    runs-on: ubuntu-latest
    env:
      TARGET: staging
    steps:
      - name: Show variables
        env:
          REQUEST_ID: demo-123
        run: |
          echo "App=$APP_NAME Target=$TARGET Request=$REQUEST_ID"
```

`env` configures shell environment; `vars` stores non-secret configuration; `secrets` stores sensitive values. A secret isn't automatically injected into shell: explicitly map it through `env:` or action inputs when required.

### Boolean expressions and operators

```yaml
if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
```

Useful functions: `success()`, `failure()`, `cancelled()`, `always()`, `contains()`, `startsWith()`, `endsWith()`, `format()`, `join()`, `toJSON()`, `fromJSON()`, `hashFiles()`.

**Security warning — expression injection:** Avoid embedding user-controlled context directly into `run` shell commands:

```yaml
# BAD: user-controlled PR title could become shell code
# - run: echo "${{ github.event.pull_request.title }}"

# BETTER: pass it as an environment variable, then quote shell expansion
- name: Print PR title safely
  env:
    PR_TITLE: ${{ github.event.pull_request.title }}
  run: printf '%s\n' "$PR_TITLE"
```

**Security detail:** Even in an environment variable, input is untrusted data. Never use it as unvalidated script, a command, a dangerous pathname, or an unchecked deployment target.

---

## 8. Outputs, Environment Files and Job Communication

GitHub uses **environment files** instead of the old deprecated `::set-output` command.

### Step output → job output → dependent job

```yaml
name: Outputs Demo
on: workflow_dispatch
permissions:
  contents: read
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.value }}
    steps:
      - id: version
        run: echo 'value=1.2.3' >> "$GITHUB_OUTPUT"

  consume:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - env:
          VERSION: ${{ needs.build.outputs.version }}
        run: echo "Deploying version $VERSION"
```

| File | Purpose | Example |
|---|---|---|
| `$GITHUB_OUTPUT` | Step output consumed by downstream steps/jobs | `echo 'image=abc' >> "$GITHUB_OUTPUT"` |
| `$GITHUB_ENV` | Variables for **subsequent** steps in same job | `echo 'MODE=staging' >> "$GITHUB_ENV"` |
| `$GITHUB_PATH` | Add directory to PATH for later steps | `echo "$HOME/bin" >> "$GITHUB_PATH"` |
| `$GITHUB_STEP_SUMMARY` | Rich Markdown workflow summary | `echo '## Tests passed' >> "$GITHUB_STEP_SUMMARY"` |

### Multiline output

```bash
{
  echo 'report<<EOF'
  cat report.txt
  echo 'EOF'
} >> "$GITHUB_OUTPUT"
```

For arbitrary/untrusted content use a **random unique delimiter** to avoid collisions; never put credentials into outputs/summaries. For large data use artifacts, not job outputs.

---

## 9. Matrix, Concurrency, Timeouts and Resilience

### Parallel matrix testing

```yaml
strategy:
  fail-fast: false
  max-parallel: 3
  matrix:
    os: [ubuntu-latest, windows-latest]
    python: ['3.11', '3.12', '3.13']
runs-on: ${{ matrix.os }}
steps:
  - uses: actions/checkout@v7
  - uses: actions/setup-python@v7
    with:
      python-version: ${{ matrix.python }}
  - run: python -m unittest discover
```

This creates **2 × 3 = 6 jobs**. `fail-fast: false` keeps other matrix variations running after one fails; `max-parallel` controls simultaneous matrix jobs.

### Concurrency group

```yaml
concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

Useful for canceling outdated PR/branch CI. **Do not blindly cancel production deployments.** For deployment jobs, often set a fixed environment group and `cancel-in-progress: false` so releases don't interrupt each other. Note: a concurrency group can have pending-run replacement behavior; it is not a durable FIFO deployment queue. GitHub also supports expanded queue options; check current docs for the target environment.

### Timeouts and retries

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - name: API test
        timeout-minutes: 5
        run: ./scripts/api-test.sh
```

GitHub Actions doesn't provide a universal `retry: 3` step keyword. Implement bounded retries **only for transient failures** (e.g. network errors), not code/test failures:

```bash
for attempt in 1 2 3; do
  if curl --fail --retry 2 https://example.com/health; then
    exit 0
  fi
  sleep $((attempt * 3))
done
exit 1
```

---
## 10. Project 1: Python CI Pipeline

**Interview task:** “Every PR must run formatting, linting and unit tests. Store the test report even if tests fail.”

**Sample repository:**

```text
python-service/
├── app.py
├── requirements.txt
├── requirements-dev.txt
├── tests/test_app.py
└── .github/workflows/python-ci.yml
```

**File:** `.github/workflows/python-ci.yml`

```yaml
name: Python CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: python-ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  quality:
    runs-on: ubuntu-24.04
    timeout-minutes: 15
    steps:
      - name: Checkout
        uses: actions/checkout@v7
        with:
          persist-credentials: false

      - name: Configure Python
        uses: actions/setup-python@v7
        with:
          python-version: '3.12'
          cache: pip
          cache-dependency-path: requirements-dev.txt

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements-dev.txt

      - name: Format check
        run: python -m black --check .

      - name: Lint
        run: python -m ruff check .

      - name: Unit tests
        run: |
          mkdir -p reports
          python -m pytest tests -q --junitxml=reports/junit.xml

      - name: Upload test results
        if: ${{ always() }}
        uses: actions/upload-artifact@v7
        with:
          name: python-test-report
          path: reports/
          if-no-files-found: warn
          retention-days: 7
```

**Example `requirements-dev.txt`:**

```text
pytest
black
ruff
```

> **Practical:** Install application dependencies from `requirements.txt` as well if they are required for the test run. For stricter reproducibility, lock/pin dependency versions and use a dependency lockfile. The code above expects the three development tools to be declared in `requirements-dev.txt`.

**What to say in an interview:** “I run fast static checks and unit tests on PRs, use a least-privilege token, cache pip dependencies, upload JUnit output and block merges using required status checks. I don't expose deployment credentials to untrusted PR code.”

---

## 11. Project 2: Java / Maven CI Pipeline

**Use case:** Java microservice built with Maven.

**File:** `.github/workflows/java-ci.yml`

```yaml
name: Java Maven CI
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
permissions:
  contents: read
jobs:
  build-test:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v7
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'
          cache: maven
      - name: Test and package
        run: mvn --batch-mode --no-transfer-progress clean verify
      - name: Publish JAR
        uses: actions/upload-artifact@v7
        with:
          name: service-jar
          path: target/*.jar
          if-no-files-found: error
```

**Interview points:** Maven lifecycle: `validate → compile → test → package → verify → install → deploy`; `mvn verify` can run integration/quality gates as configured by plugins. Use `settings.xml` or an appropriate registry credential for private dependencies; don't print credentials.

---

## 12. Project 3: Node.js CI with Matrix

**File:** `.github/workflows/node-ci.yml`

```yaml
name: Node CI Matrix
on:
  pull_request:
  push:
    branches: [main]
permissions:
  contents: read
jobs:
  test:
    name: Node ${{ matrix.node }} on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-24.04, windows-latest]
        node: [22, 24]
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-node@v7
        with:
          node-version: ${{ matrix.node }}
          cache: npm
      - run: npm ci
      - run: npm test
      - run: npm run build --if-present
```

- `npm ci` is designed for lockfile-based clean installs and fails if the lockfile doesn't match package metadata.
- Matrix creates four test jobs (two operating systems × two Node versions).
- `setup-node` caching speeds dependency installation; the `package-lock.json` file should be committed.

---

## 13. Artifacts vs Caches

| Attribute | Artifact | Cache |
|---|---|---|
| Purpose | Preserve/share build output | Accelerate repeated work |
| Example | JAR, coverage, binary, SBOM | Maven `.m2`, npm cache, pip cache |
| Used by | Other jobs / humans / releases | Later jobs/workflow runs |
| Key | Artifact name / ID | Cache key + version + scope |
| Persistence | Configurable retention and repository policies | Eviction / scoped reuse rules |
| Security | Treat content from untrusted runs as untrusted | Protect against cache poisoning |

### Share a compiled binary between jobs

```yaml
name: Artifact Handoff
on: workflow_dispatch
permissions:
  contents: read
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: |
          mkdir -p dist
          printf 'hello\n' > dist/app.txt
      - uses: actions/upload-artifact@v7
        with:
          name: application
          path: dist/

  inspect:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v8
        with:
          name: application
          path: downloaded
      - run: cat downloaded/app.txt
```

### Dependency cache with explicit key

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: pip-${{ runner.os }}-${{ hashFiles('requirements*.txt') }}
    restore-keys: |
      pip-${{ runner.os }}-
```

**Caution:** The explicit cache action release shown is a familiar syntax example; confirm the approved release and runtime compatibility in your environment. Cache only rebuildable dependencies, not long-lived credentials or sensitive outputs. Never trust cached executable content from an untrusted branch in a privileged job.

**Common question:** “Why did artifacts disappear?” Check expiration/retention, upload path, permissions, artifact version/API compatibility, and whether the artifact exists in the *same run*.

---

## 14. Project 4: Docker Build and GHCR Push

**Scenario:** Verify a container build, then push `ghcr.io/owner/repository:<commit-sha>` to GitHub Container Registry **only on `main`**.

**Prerequisites:** A valid `Dockerfile`, GitHub Packages permissions, and package access settings.

**File:** `.github/workflows/docker-ghcr.yml`

```yaml
name: Docker Build and Push to GHCR

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

permissions:
  contents: read
  packages: write

jobs:
  docker:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v7
        with:
          persist-credentials: false

      - name: Derive lowercase image name
        id: image
        shell: bash
        run: |
          echo "name=ghcr.io/${GITHUB_REPOSITORY,,}" >> "$GITHUB_OUTPUT"

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v4

      - name: Log in to GitHub Container Registry
        if: ${{ github.event_name == 'push' }}
        uses: docker/login-action@v4
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build (and push from main)
        uses: docker/build-push-action@v7
        with:
          context: .
          push: ${{ github.event_name == 'push' }}
          tags: ${{ steps.image.outputs.name }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

**Flow:** PR → build only → validation; merge to `main` → authenticate → build → push immutable SHA tag. Avoid using only `latest` for auditable deployments. When deploying, prefer verified immutable image digests (`@sha256:...`) over mutable tags.

**Security notes:**

- This job grants `packages: write` because it may publish; a stricter design separates PR verification from publishing into jobs with different permissions.
- A fork PR generally receives restricted credentials and cannot publish packages.
- Sign or attest artifacts/images, scan vulnerabilities, and restrict who may change publishing workflows.
- A real application should execute its test suite *before* the publishing job (e.g. `needs: test`).

---

## 15. Project 5: AWS OIDC + ECR + EKS

### Target architecture

```text
Merge to main
    |
    v
Unit tests + SAST + dependency audit
    |
    v
GitHub OIDC token  ------> AWS IAM role (temporary STS credentials)
    |
    v
Build container ---> Push immutable tag to Amazon ECR
    |
    v
EKS credentials ---> kubectl / Helm ---> rollout checks
    |
    v
Monitoring + smoke test + optional rollback
```

### Why OIDC rather than access keys?

OIDC lets the workflow request a short-lived AWS session by assuming a tightly scoped role. You do **not** store long-lived `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` as repository secrets. An IAM trust policy restricts which repository/ref/environment may assume the role.

### 1. AWS prerequisites

1. Configure `https://token.actions.githubusercontent.com` as an AWS IAM OIDC identity provider with audience `sts.amazonaws.com`.
2. Create an IAM role with a **restricted trust policy** and least-privilege ECR/EKS permissions.
3. Grant the IAM principal authorization in the target EKS cluster (prefer an EKS access entry / appropriate RBAC).
4. Create the ECR repository and EKS deployment/namespace in advance.
5. Set GitHub repository variables `AWS_REGION`, `AWS_ROLE_ARN`, `EKS_CLUSTER`, `ECR_REPOSITORY` to correct non-secret values.
6. Create and protect a `production` GitHub Environment; restrict deployment branches and reviewers as supported by your plan/repository settings.

**IAM trust policy — conceptual example for a branch-scoped identity:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/YOUR_REPO:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

> **2026 compatibility warning:** GitHub has introduced **immutable OIDC subject claims** (including repository/owner IDs for repositories created after 15 July 2026 or opted in). The `sub` above is an example of the legacy branch-ref shape and **may not match a new repository**. If a job uses an `environment: production`, the subject commonly takes an environment-oriented form instead of the branch-ref form; custom subject templates may also apply. **Inspect your repository's actual OIDC claims and the current GitHub AWS OIDC reference, then make a narrowly matched AWS trust policy. Never “fix” a subject mismatch by allowing every repository (`repo:*`) or using an unrestricted wildcard.**

### 2. ECR + EKS pipeline

**File:** `.github/workflows/aws-ecr-eks.yml`

```yaml
name: AWS ECR + EKS Deployment

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  build-push:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
      id-token: write
    outputs:
      image: ${{ steps.image.outputs.uri }}
    steps:
      - uses: actions/checkout@v7

      - name: Configure temporary AWS credentials
        uses: aws-actions/configure-aws-credentials@v6
        with:
          role-to-assume: ${{ vars.AWS_ROLE_ARN }}
          aws-region: ${{ vars.AWS_REGION }}

      - name: Log in to Amazon ECR
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push container
        id: image
        env:
          AWS_REGION: ${{ vars.AWS_REGION }}
          ECR_REPOSITORY: ${{ vars.ECR_REPOSITORY }}
        run: |
          ACCOUNT_ID="$(aws sts get-caller-identity --query Account --output text)"
          REGISTRY="${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
          IMAGE="${REGISTRY}/${ECR_REPOSITORY}:${GITHUB_SHA}"
          docker build -t "$IMAGE" .
          docker push "$IMAGE"
          echo "uri=$IMAGE" >> "$GITHUB_OUTPUT"

  deploy:
    needs: build-push
    runs-on: ubuntu-24.04
    environment: production
    concurrency:
      group: production-eks
      cancel-in-progress: false
    permissions:
      contents: read
      id-token: write
    steps:
      - name: Configure temporary AWS credentials
        uses: aws-actions/configure-aws-credentials@v6
        with:
          role-to-assume: ${{ vars.AWS_ROLE_ARN }}
          aws-region: ${{ vars.AWS_REGION }}

      - name: Configure Kubernetes context
        env:
          AWS_REGION: ${{ vars.AWS_REGION }}
          EKS_CLUSTER: ${{ vars.EKS_CLUSTER }}
        run: |
          aws eks update-kubeconfig --region "$AWS_REGION" --name "$EKS_CLUSTER"

      - name: Deploy known image
        env:
          IMAGE: ${{ needs.build-push.outputs.image }}
        run: |
          kubectl -n production set image deployment/payments-api \
            payments-api="$IMAGE"
          kubectl -n production rollout status deployment/payments-api --timeout=5m
```

**Very important deployment considerations:**

- The AWS role assumed by `build-push` and `deploy` may require different **trust subject conditions** and IAM policies. In this teaching example both use one `AWS_ROLE_ARN`; in production create separate least-privilege **build** and **deploy** roles and set their appropriate variables.
- The deploy job uses `environment: production`: its actual OIDC `sub` differs from an ordinary branch-based job in many configurations. Configure AWS trust accordingly.
- `kubectl` must be installed and compatible; confirm the GitHub runner image or explicitly set it up. The EKS API must be reachable from the runner (use private-network runners if cluster endpoint is private).
- `kubectl set image` assumes an existing Deployment `payments-api` and container `payments-api` in namespace `production`.
- Use an image digest or deployment manifest/Helm chart promotion for a stronger immutable deployment model, and keep enough history for rollback.
- `workflow_dispatch` runs from a selected ref; the IAM trust and environment protections must ensure **only authorized refs** can deploy. Restrict the deployment environment to main and enforce approval.

### 3. Helm alternative

```bash
helm upgrade --install payments-api ./helm/payments-api \
  --namespace production --create-namespace \
  --set image.repository="$ECR_IMAGE_REPOSITORY" \
  --set image.tag="$GITHUB_SHA" \
  --wait --atomic --timeout 5m
```

`--atomic` attempts rollback when a Helm upgrade fails; still configure health probes, DB migration strategy and monitoring.

**Interview answer:** “I authenticate GitHub Actions to AWS with OIDC and IAM trust conditions, build and publish a digest-addressable image to ECR, deploy through an approved GitHub environment, check EKS rollout health and keep a rollback path. I separate roles, protect branch/environment rules and avoid static cloud access keys.”

---

## 16. Project 6: Terraform Plan / Apply

**Requirement:** PR → `fmt/validate/plan`; merge to `main` → approval → apply. Use a remote, locking-capable state backend. Don't expose cloud credentials to untrusted PRs.

### Terraform files

```text
infra/
├── main.tf
├── variables.tf
├── outputs.tf
└── backend.tf
```

Example backend structure (configure and provision it securely beforehand):

```hcl
terraform {
  backend "s3" {
    bucket       = "your-secure-tf-state-bucket"
    key          = "platform/dev/terraform.tfstate"
    region       = "ap-south-1"
    use_lockfile = true
    encrypt      = true
  }
}
```

> Backends/locking approaches depend on your Terraform release and AWS account design. Protect your state bucket and versioning, restrict IAM, and verify the state locking method your version supports.

**File:** `.github/workflows/terraform.yml`

```yaml
name: Terraform Infrastructure

on:
  pull_request:
    branches: [main]
    paths: ['infra/**', '.github/workflows/terraform.yml']
  push:
    branches: [main]
    paths: ['infra/**', '.github/workflows/terraform.yml']

permissions:
  contents: read

jobs:
  validate:
    runs-on: ubuntu-24.04
    defaults:
      run:
        working-directory: infra
    steps:
      - uses: actions/checkout@v7
      - uses: hashicorp/setup-terraform@v4
        with:
          terraform_version: '1.14.6'
      - run: terraform fmt -check -recursive
      - run: terraform init -backend=false -input=false
      - run: terraform validate -no-color

  apply:
    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
    needs: validate
    runs-on: ubuntu-24.04
    environment: production
    concurrency:
      group: terraform-prod-state
      cancel-in-progress: false
    permissions:
      contents: read
      id-token: write
    defaults:
      run:
        working-directory: infra
    steps:
      - uses: actions/checkout@v7
      - uses: hashicorp/setup-terraform@v4
        with:
          terraform_version: '1.14.6'
      - uses: aws-actions/configure-aws-credentials@v6
        with:
          aws-region: ${{ vars.AWS_REGION }}
          role-to-assume: ${{ vars.TERRAFORM_APPLY_ROLE_ARN }}
      - run: terraform init -input=false
      - name: Create plan from current protected ref/state
        run: terraform plan -input=false -out=tfplan
      - name: Apply approved job plan
        run: terraform apply -input=false tfplan
```

**Important:** This is a **safe deployment skeleton**: PRs perform validation without cloud access; they do **not** run a cloud-backed plan. An organization that wants a PR plan must design a read-only/plan IAM role, scope OIDC authorization to trusted origins, handle forked PRs safely, and publish plan results securely. The apply job creates a **fresh** plan after environment approval (rather than applying an artifact from an untrusted PR). For higher assurance, review and approve the exact plan artifact/digest in a protected release process.

**Terraform interview checks:**

- Why remote state? Shared state, locking, recovery and auditable configuration.
- Why avoid concurrent applies? State corruption and conflicting changes; use backend locks **and** CI concurrency.
- `plan` vs `apply`? Plan previews changes; apply executes them.
- Drift? Real infrastructure differs from declared state/config; detect with safe periodic plan/audits.
- Separate workspaces/accounts? Prefer strong isolation across staging/production, not just variable changes.
- How to stop secrets leaking into plans? Sensitive variables/outputs, secure plan files, restrict artifact visibility, logs and state access.

---

## 17. Deployment Environments, Approvals and Rollback

A GitHub **Environment** can represent `development`, `staging` or `production`. Configure it in **Repository → Settings → Environments** (available protection features depend on repository/plan). Use it for reviewers, branch restrictions and environment-scoped variables/secrets.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.example.com
    steps:
      - run: echo 'Deployment executes after environment protection passes'
```

**Suggested promotion policy:**

```text
PR checks → merge main → build once → immutable artifact/image digest
                                      |
                                      v
                                  staging deploy
                                      |
                                      v
                                 smoke + health tests
                                      |
                                      v
                             protected production approval
                                      |
                                      v
                              production deployment
                                      |
                        monitor / rollback when required
```

**Rollback patterns:** `kubectl rollout undo deployment/app`, `helm rollback RELEASE REVISION`, redeploy last known-good image digest, switch traffic to a previous blue/green target or revert a canary. A rollback is not always safe for irreversible database migrations; plan forward/backward-compatible schemas.

**Release engineering:** Prefer **build once, promote the same tested artifact** over rebuilding different binaries for every environment. Define safe failure/recovery behavior (health check timeouts, gates, notification, manual intervention).

---

## 18. Reusable Workflows and Composite Actions

### Comparison

| Feature | Reusable workflow | Composite action |
|---|---|---|
| Defined in | `.github/workflows/*.yml` | Any action directory with `action.yml` |
| Invoked through | `jobs.<id>.uses` | `steps[*].uses` |
| Can define jobs/runners | Yes | No independent job/runner |
| Inputs / secrets | `workflow_call` | `inputs` and step env |
| Best use | Common organization CI/CD pipeline | Repeated step sequence |
| Example | Shared secure deploy workflow | Setup + test + packaging action |

### Reusable workflow

**File:** `.github/workflows/reusable-test.yml`

```yaml
name: Reusable Python Test
on:
  workflow_call:
    inputs:
      python-version:
        type: string
        required: false
        default: '3.12'
permissions:
  contents: read
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-python@v7
        with:
          python-version: ${{ inputs.python-version }}
      - run: pip install -r requirements-dev.txt
      - run: pytest -q
```

**Caller file:** `.github/workflows/ci-caller.yml`

```yaml
name: Call Shared Tests
on: [pull_request, workflow_dispatch]
permissions:
  contents: read
jobs:
  test:
    uses: ./.github/workflows/reusable-test.yml
    with:
      python-version: '3.12'
```

**Crucial:** `uses: ./.github/workflows/...` is at **job level**, not in `steps:`. For cross-repository reuse, prefer a trusted tag or **commit SHA** rather than a mutable `@main` ref. Pass only necessary named secrets; use `secrets: inherit` only within a clearly trusted boundary.

### Composite action

**File:** `.github/actions/hello/action.yml`

```yaml
name: 'Greetings'
description: 'Small reusable sequence of steps'
inputs:
  who:
    description: 'Who to greet'
    required: true
runs:
  using: composite
  steps:
    - shell: bash
      env:
        WHO: ${{ inputs.who }}
      run: printf 'Hello %s\n' "$WHO"
```

**Caller:**

```yaml
steps:
  - uses: actions/checkout@v7
  - uses: ./.github/actions/hello
    with:
      who: DevOps Team
```

**Design tip:** Reusable workflows enforce standardized complete jobs, permissions and gates; composite actions share repeated step logic within a caller's job.

---

## 19. Advanced Patterns and Microservice CI/CD

### 19.1 Path-based filtering for a monorepo

```yaml
name: Payments Only
on:
  pull_request:
    paths:
      - 'services/payments/**'
      - '.github/workflows/payments.yml'
jobs:
  test-payments:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - run: make -C services/payments test
```

**Caution:** A required status check can remain pending if its entire workflow is skipped by path filters. Design check requirements and skip behavior accordingly.

### 19.2 PostgreSQL service container for integration testing

```yaml
name: Integration Tests
on: [pull_request]
permissions:
  contents: read
jobs:
  integration:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:17
        env:
          POSTGRES_DB: app_test
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass-only-for-ephemeral-ci
        ports:
          - 5432:5432
        options: >-
          --health-cmd "pg_isready -U testuser -d app_test"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v7
      - name: Execute integration tests
        env:
          DATABASE_URL: postgresql://testuser:testpass-only-for-ephemeral-ci@localhost:5432/app_test
        run: ./scripts/integration-test.sh
```

The sample password is only for a disposable CI database and must **never** be reused in real infrastructure. The script must exist in the repository.

### 19.3 Containerized job

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    container:
      image: python:3.12-slim
    steps:
      - uses: actions/checkout@v7
      - run: python --version
```

Check availability of shells/tools inside the container; not all images include `bash`/Git/Node dependencies expected by particular actions.

### 19.4 Cross-workflow triggering

Use `workflow_run` for trustworthy completion-based orchestration and `repository_dispatch` for authenticated external events. Never assume outputs from a completed **untrusted** PR workflow are safe to execute in a high-privilege follow-up workflow.

### 19.5 Release and deployment strategy choices

- **Blue/green:** Build a second production stack, verify, switch traffic, retain rollback path.
- **Canary:** Route a small percentage of traffic to a new release and compare SLI error rate/latency.
- **Rolling update:** Gradually replace pods while maintaining availability with readiness probes.
- **GitOps:** CI publishes signed images and updates deployment intent; Argo CD/Flux reconciles desired state, ideally with policy/approval control.

### 19.6 Selective test execution and quality gates

Design code → unit → integration → build → security → deployment stages, with failures blocking downstream privileged steps. Use matrix and path filtering to reduce cost without creating blind spots. Keep audit evidence (reports, digests, SBOMs) for releases as required.

---
## 20. Security and DevSecOps Best Practices

### Essential security controls

| Risk | Good control |
|---|---|
| Compromised action | Pin trusted action to full verified commit SHA; audit dependencies and update via review/automation |
| Excess token scope | Set `permissions:` to smallest required scopes per job |
| Leaked cloud credentials | Prefer cloud OIDC short-lived role sessions |
| Malicious fork PR | Use read-only PR workflows; do not run untrusted code with write token/secrets |
| Script injection | Pass untrusted text via environment, validate, quote, avoid `eval` |
| Exposed self-hosted runner | Ephemeral runner, dedicated pool, minimal network and filesystem privilege |
| Cache poisoning | Isolate privileged jobs from untrusted cache content |
| Artifact tampering | Verify provenance, digest, signature; never execute untrusted artifact blindly |
| Unauthorized production push | Protected branch, required checks, environments, reviewers, deployment restrictions |
| Lateral movement | Separate build/deploy roles, accounts, runners, and network access |
| Dependency supply-chain | SBOM + lockfiles + SCA + vulnerability scans + provenance |
| Secret in logs | Mask secrets, remove debug leaks, avoid `set -x` when secrets are in use |

### Least privilege `GITHUB_TOKEN`

```yaml
permissions:
  contents: read

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - run: echo 'This job has explicitly defined content-write permission'
```

Common scopes: `contents: read` for checkout; `contents: write` for release/tag changes; `packages: write` for package publishing; `pull-requests: write` to comment on PRs; `id-token: write` for requesting OIDC tokens; `security-events: write` for uploading compatible security findings; `attestations: write` for attestation tasks. Setting *any* permissions explicitly typically makes unspecified scopes `none`; check the current permission reference.

**`id-token: write` does not authorize AWS access by itself**: it permits requesting a signed GitHub OIDC identity token. The external provider's trust policy and IAM permissions decide whether any cloud resource can be accessed.

### Secure action pinning

```yaml
# Easy-to-read learning example:
- uses: actions/checkout@v7

# Production pattern: replace with the exact full verified SHA from the
# reviewed release (illustrative placeholder, NOT an actual commit):
# - uses: actions/checkout@<FULL_40_CHARACTER_VERIFIED_COMMIT_SHA>
```

Allow Dependabot/Renovate to open reviewed update PRs, and consider organization policy / action allowlists / workflow dependency lockfiles where supported. A version tag can move; a full commit SHA identifies immutable code.

### Never do this

```yaml
# DANGEROUS: privileged workflow executes PR HEAD code
# on: pull_request_target
# steps:
#   - uses: actions/checkout@v7
#     with:
#       ref: ${{ github.event.pull_request.head.sha }}
#   - run: ./scripts/from-pull-request.sh
```

Why dangerous? `pull_request_target` runs in the base repository security context, and untrusted PR code may exfiltrate tokens, secrets, runner data or cloud credentials. Newer checkout versions add safeguards for certain dangerous cases, but **workflow design is still your responsibility**.

### Secrets handling

- Store sensitive values in appropriate secret stores, not workflow files or Git history.
- Environment secrets are gated by environment policy; organization/repository secrets have different scopes.
- Secrets are generally unavailable to workflows triggered by forked pull requests; Dependabot-triggered workflows may have special restrictions.
- GitHub attempts to redact known secrets from logs, but masking isn't a substitute for not logging them.
- Avoid passing secrets through artifact names, CLI arguments visible in process listings, job outputs, or caches.
- Rotate any compromised token and audit workflow logs and cloud access events.

### Threat model interview scenario

**Question:** “A malicious contributor submits a PR updating a shell script. Can it steal AWS secrets?”

**Answer:** “If the PR checks are read-only and have no deployment/cloud credentials, risk is significantly reduced. It becomes dangerous if a privileged `pull_request_target`/`workflow_run` job checks out and executes that PR's code, or if an untrusted artifact/cache crosses into a trusted job. I separate trust boundaries, use pinned actions, least-privilege tokens, gated environments, and ephemeral runners.”

---

## 21. Supply Chain, SBOM, SAST, SCA and Attestations

### Where security fits into CI/CD

```text
Code + dependencies
  |
  +--> Lint + unit tests
  +--> SAST (source-code analysis)
  +--> Secret scan
  +--> SCA (third-party dependencies / licenses)
  +--> Container image scan
  +--> SBOM generation
  +--> Artifact attestation / signing
  |
  v
Approved artifact/image → trusted registry → deployment → runtime monitoring
```

| Tool / concept | Category | Purpose |
|---|---|---|
| CodeQL | SAST | Code analysis for supported languages |
| Coverity | SAST | Static code defects / standards checks |
| Black Duck | SCA | Open-source components, risk and licensing |
| Dependabot | Dependency updates | Automated dependency/action update PRs |
| Trivy | Image/filesystem/IaC scanning | Vulnerability and misconfiguration findings |
| Gitleaks | Secret scanning | Detect credentials accidentally in files |
| Syft | SBOM | Generate software component inventory |
| Cosign | Signing / verification | Sign and verify container artifacts |
| Artifact attestation | Provenance | Establish origin and build metadata |

**Example: Trivy image scan** (assumes image built locally earlier in this job, and Trivy is installed or set up explicitly):

```bash
trivy image --exit-code 1 --severity CRITICAL,HIGH "${IMAGE}"
```

Add tuning for acceptable findings, fixed/unfixed policy and false positives; avoid “ignore all” overrides. A scan passing does not prove a product is secure.

### Attestation concept

GitHub Actions can generate signed build provenance attestations. Verify the identity of the workflow, source commit and artifact digest before deployment. The precise GitHub attestation action/API, token permissions, artifact format and registry support vary, so use the current official [artifact attestations documentation](https://docs.github.com/en/actions/concepts/security/artifact-attestations).

### Interview-ready explanation

“SAST checks first-party code, SCA checks dependencies and license exposure, container scanning inspects packaged artifacts, SBOM lists components, and signed attestations help verify **who built what from which source**. I apply blocking policies before release and avoid treating a single scanner as complete assurance.”

---

## 22. Self-hosted Runners, ARC and Scaling

### Basic Ubuntu self-hosted runner onboarding

1. Navigate to repository/organization settings → **Actions → Runners → New self-hosted runner**.
2. Select OS and architecture; follow GitHub's **current** installation commands and one-time registration token instructions.
3. Install into a dedicated, hardened location under a dedicated OS account; don't run arbitrary untrusted workflows as `root`.
4. Install required dependencies (JDK, Docker, build tools, kubectl, AWS CLI) and verify network reachability.
5. Configure as a service using GitHub's displayed setup instructions when appropriate.
6. Apply labels/runner groups and restrict which repositories may use the runner.
7. Patch OS/tools/runner, rotate credentials, monitor disk, memory, idle capacity, queue time and logs.
8. Prefer ephemeral runners for sensitive environments or untrusted workloads.

**Job example:**

```yaml
jobs:
  specialized-build:
    runs-on: [self-hosted, linux, x64, big-disk]
    timeout-minutes: 360
    steps:
      - uses: actions/checkout@v7
      - run: df -h && free -h && nproc
      - run: ./scripts/build.sh
```

**Large projects (AOSP/Yocto):** GitHub-hosted standard runners may be too small for 100+ GB workspaces, hour-long builds, special SDKs and large caches. Use controlled self-hosted machines or autoscaling VMs with workspace isolation, quotas, durable external artifacts and cleanup.

### Actions Runner Controller (ARC)

ARC runs self-hosted runner **scale sets in Kubernetes**. It responds to queued demand and can provision ephemeral runner pods. Key moving parts include controller, listener and runner scale sets; details vary with GitHub's ARC version.

**Autoscaling interview answer:** “I use GitHub-hosted runners for routine work and an autoscaling pool of ephemeral self-hosted runners/ARC for specialized workloads. I isolate sensitive jobs, pre-bake tool images, track job queue time and costs, and avoid sharing persistent build hosts across untrusted branches.”

### Operational checks for self-hosted hosts

```bash
uname -a
id
df -h
free -h
nproc
docker info
java -version
which git jq aws kubectl helm
```

Runner compatibility: newer JavaScript actions may require a recent Actions Runner version (for example, Node 24 actions require newer runner releases). Check release notes before upgrading action major versions on corporate/older GHES runners.

---

## 23. Monitoring, Debugging and Troubleshooting

### Debug workflow method

1. Identify whether the workflow **triggered**, was **queued**, **skipped**, **canceled** or **failed**.
2. Inspect the **exact failed job and step**, logs, event/ref and selected runner.
3. Confirm file path, syntax, branch/paths filters, job `if` and `needs` conditions.
4. Validate runner environment: OS, installed tools, network, disk, credentials, architecture.
5. For cloud errors, inspect `GITHUB_TOKEN` permissions, OIDC claims/trust, IAM roles, environment approvals and network endpoints.
6. Reproduce the failing command locally or on an equivalent isolated runner.
7. Make a minimal, safe fix; rerun and add regression checks.

### Useful diagnostics

```yaml
- name: Diagnostic snapshot
  shell: bash
  run: |
    echo "Event=$GITHUB_EVENT_NAME"
    echo "Ref=$GITHUB_REF"
    echo "SHA=$GITHUB_SHA"
    uname -a
    df -h
    free -h
    git --version
```

Avoid blindly printing `toJSON(secrets)`, whole event payloads, raw tokens, full HTTP headers or complete cloud credentials. GitHub has debug-logging options in its docs, but enable them with an understanding of secret exposure and cost.

### Cases to remember

**Workflow did not run:** Wrong `.github/workflows/` path; workflow not present on triggering branch; branch/path filters; disabled Actions; cron on wrong branch/time zone; required event activity missing.

**Job stays queued:** No runner with matching labels/groups; offline/busy self-hosted runner; resource/plan quota; protected environment waiting for approval; concurrency conflicts.

**`Resource not accessible by integration`:** Missing `GITHUB_TOKEN` permission, fork PR restrictions, organization policy, or trying to use token outside repo scope.

**`No credentials could be loaded`:** Missing `id-token: write`, failed cloud trust condition, wrong region/role ARN, or mismatched `sub` after environment/claim changes.

**Docker build fails:** Dockerfile/path mismatch, unsupported build platform, disk space, denied registry auth, missing build secrets, network issues, rate limits.

**EKS deployment fails:** Wrong cluster name/region, missing `eks:DescribeCluster`, missing cluster access/RBAC, private API unreachable, pod image pull failure, probes failing.

**`pytest: command not found`:** Dependencies not installed in current job/container, wrong PATH, wrong Python environment.

**“works locally, fails in CI”:** Dependencies/runtime/OS mismatch, case-sensitive filesystems, hidden state, secrets, environment variables, network assumptions, test flakiness.

### Operational observability

- Track **queue wait**, **job duration**, **failure rate**, **retry count**, **deployment success**, **MTTR** and cache hit rate.
- Measure DORA metrics where practical: deployment frequency, lead time for changes, change failure rate and failed-deployment recovery time.
- Correlate build/run ID, commit SHA, image digest, environment and deployment ID.
- Use platform logs and external telemetry (CloudWatch, Prometheus/Grafana, ELK, Datadog, etc.) as appropriate.

---

## 24. Performance, Cost and Workflow Optimization

### Priority order

1. Remove redundant runs (event filters, `concurrency` for fast PR CI).
2. Execute independent jobs in parallel with sensible `needs` dependencies.
3. Use dependency caching safely and keep lockfiles reproducible.
4. Avoid downloading/rebuilding the same large artifact repeatedly.
5. Use path filters with correct required-status-check design.
6. Optimize Docker layers/build cache; use `.dockerignore` and multi-stage builds.
7. Split small fast PR checks from slower nightly/integration/security suites.
8. Use right-sized runners; benchmark queue plus execution time, not only CPU.
9. Pre-bake ephemeral self-hosted runner images for heavy tools.
10. Retain useful evidence but apply suitable artifact retention policies.

**Measure optimization, don't guess:**

```text
Before: dependency install 5 min + tests 4 min + packaging 3 min = ~12 min
After:  cache restore 1 min + tests 4 min + packaging 3 min = ~8 min
Relative reduction = (12 - 8) / 12 ≈ 33%
```

The numbers above are an *illustrative calculation*, not a claim from a real measured pipeline.

**Costs:** Hosted minutes, storage and premium runner pricing vary by account/plan and change over time. Look up current GitHub billing documentation instead of memorizing prices. Self-hosted may reduce billed runner minutes in some contexts but adds compute, maintenance and security costs.

---

## 25. GitHub CLI, Local Testing and Administration

### Install and authenticate GitHub CLI

```bash
gh --version
gh auth login
gh auth status
```

### Workflows and runs

```bash
gh workflow list
gh workflow view ci.yml
gh workflow run ci.yml --ref main
gh run list --limit 10
gh run view --log
gh run watch
gh run rerun RUN_ID --failed
gh run download RUN_ID
```

Exact CLI flags can change; use `gh <subcommand> --help` for your installed version. Some commands require appropriate repository permissions.

### Secrets/variables from CLI

```bash
gh secret list
gh variable list
gh variable set AWS_REGION --body ap-south-1
```

To set secrets, prefer interactive/approved secure input paths rather than embedding secret literals in shell history. Understand whether the scope is repository, environment or organization.

### Syntax and local testing

- **actionlint**: static validation of workflow YAML, expressions and many mistakes.
- **yamllint**: general YAML style/syntax, with configuration appropriate for GitHub Actions.
- **`act`**: third-party local workflow execution using Docker; useful for quick feedback but not an exact match for GitHub-hosted runners, permissions, OIDC or environments.
- **GitHub test branch:** Most trustworthy integration test for runner/permissions/hosting behavior.

```bash
# Once actionlint is installed:
actionlint .github/workflows/*.yml

# Run workflows in their real hosting context:
git add .github/workflows && git commit -m 'Add CI workflow'
git push origin feature/add-ci
```

**Action pinning automation:** Dependabot can update `github-actions` dependencies. Example `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
```

Review automated update PRs, particularly when actions gain permissions or change trust boundaries.

---

## 26. GitHub Actions vs Jenkins vs GitLab CI

| Topic | GitHub Actions | Jenkins | GitLab CI/CD |
|---|---|---|---|
| Pipeline config | Workflow YAML | Jenkinsfile/Groovy | `.gitlab-ci.yml` |
| Main execution unit | Job/steps | Agent/stages/steps | Job/stages |
| Integrates best with | GitHub repos and APIs | Mixed enterprise tooling | GitLab SCM and platform |
| Runners/agents | GitHub-hosted or self-hosted | Jenkins agents | GitLab runners |
| Shared pipelines | Reusable workflows/actions | Shared Libraries | Includes/components |
| Credentials | Secrets, `GITHUB_TOKEN`, OIDC | Credentials store and plugins | CI variables, identity/OIDC |
| Ops effort | Low for hosted | Higher controller + plugins + agents | Depends on SaaS/self-managed |
| Typical issue | Token permissions, event trigger | Controller/plugin stability | Runner availability, CI permissions |

### Jenkins vs GitHub Actions interview mapping

| Jenkins concept | GitHub Actions analogue |
|---|---|
| Jenkinsfile | `.github/workflows/*.yml` |
| `pipeline { stages { ... } }` | Workflow jobs and ordered steps |
| `agent { label 'linux' }` | `runs-on: [self-hosted, linux]` |
| `when` | `if` expression |
| `post { always {} }` | Step with `if: always()` |
| `archiveArtifacts` | `upload-artifact` |
| `stash/unstash` | Cross-job upload/download artifacts (different semantics) |
| `build job:` | Reusable workflow / dispatch / separate workflow trigger (depends on case) |
| Build parameters | `workflow_dispatch` inputs |
| Shared Library | Reusable workflow / composite or JS action |
| Credential binding | `secrets` / OIDC / secret manager |
| `lock(...)` | Concurrency group (not identical queueing semantics) |
| Controller / agent | GitHub orchestration / runner |

### Good migration answer

“To migrate a multistage Jenkins pipeline, I inventory stages, triggers, agents, tools, credential usage, artifact handoffs and notifications. I map independent stages to parallel jobs, sequence dependent work with `needs`, replace credential bindings with OIDC/secrets, map storage to artifacts/registry, and move persistent custom agents to appropriately isolated self-hosted runners. I validate branch events, permissions, approval gates and rollback before cutover. I don't assume Jenkins locks or build numbers are equivalent to GitHub Actions concurrency or run numbers.”

### Pipeline architecture example for a three-job release

```text
Job A — Prepare/decision: verify repo, changes and parameters
                    |
                    v
Job B — Build/test: compile, package, test, publish immutable artifact
                    |
                    v
Job C — Release: retrieve artifact, validate, deploy, changelog, notify
```

GitHub Actions translation: three jobs linked by `needs`, with small metadata in job outputs and larger build packages in an artifact registry. For strict sequential IDs, use an external controlled release-number service or a rigorously serialized workflow; `github.run_number` is **not** a universal, gap-free business integration ID.

---
## 27. Interview Questions: Beginner

**Q1. What is GitHub Actions?**  
An event-driven automation/CI/CD service built into GitHub. Workflows describe jobs and steps in YAML and execute on runners.

**Q2. What is a workflow?**  
A configurable automated process defined in `.github/workflows/*.yml` or `.yaml`.

**Q3. What is a job?**  
A group of steps executed on a specified runner. Separate jobs usually have isolated filesystems.

**Q4. What is a step?**  
A single action invocation (`uses`) or shell command (`run`) inside a job.

**Q5. What is an action?**  
A reusable unit of logic, often maintained by GitHub, a vendor or an organization.

**Q6. What is a runner?**  
The machine/agent that executes a job. It can be GitHub-hosted or self-hosted.

**Q7. Where is a workflow stored?**  
`.github/workflows/` at the repository root.

**Q8. Which format is used to write workflows?**  
YAML, with `.yml` or `.yaml` extension.

**Q9. Difference between `uses` and `run`?**  
`uses` invokes an action; `run` executes commands in the configured shell.

**Q10. What is `on:`?**  
The collection of triggering GitHub or external events.

**Q11. What is `workflow_dispatch`?**  
A manually dispatchable workflow trigger, optionally with typed inputs.

**Q12. What is `push` vs `pull_request`?**  
`push` responds to changes pushed to matching refs; `pull_request` responds to matching PR activity, commonly used for pre-merge verification.

**Q13. What is `runs-on`?**  
The runner label(s)/group selection for a job, such as `ubuntu-latest` or `[self-hosted, linux]`.

**Q14. Are jobs parallel by default?**  
Yes, if eligible and runner capacity allows. Use `needs:` to serialize dependencies.

**Q15. Are steps parallel?**  
Ordinarily steps in a job run sequentially. Newer GitHub workflow syntax also offers special background/parallel constructs; these are not the default behavior.

**Q16. What is `needs`?**  
A job dependency mechanism and a way to read dependent job outputs.

**Q17. What is `env`?**  
A map of environment variables scoped at workflow, job or step level.

**Q18. What is `GITHUB_TOKEN`?**  
A short-lived token created for workflow access to GitHub APIs, scoped by token permissions and repository policy.

**Q19. What are repository secrets?**  
Encrypted values managed in GitHub and made available to eligible workflows via `secrets` context.

**Q20. What are repository variables?**  
Non-sensitive configuration values available through `vars` context.

**Q21. What is `github.sha`?**  
The event-associated commit SHA. Its exact meaning depends on the event type; don't assume it always equals PR head SHA.

**Q22. What is `github.ref`?**  
The fully qualified event ref, such as `refs/heads/main`.

**Q23. What is `if`?**  
A condition to control whether a job or step runs.

**Q24. What is a matrix?**  
A strategy that creates multiple job variations, e.g. different operating systems and runtime versions.

**Q25. What is an artifact?**  
A stored output such as compiled files, reports or logs, retrievable for later jobs or by authorized users.

**Q26. What is a cache?**  
Reused dependency/compilation data to reduce repeated work; not a reliable artifact distribution mechanism.

**Q27. What is the Actions Marketplace?**  
A catalog of community/vendor actions; review trustworthiness and pin versions before adopting actions.

**Q28. What happens when a step fails?**  
Its job typically fails and following regular steps are skipped; explicit conditions can run cleanup or diagnostics.

**Q29. What is `checkout`?**  
A GitHub-maintained action to fetch repository content into the runner workspace.

**Q30. What is continuous delivery vs continuous deployment?**  
Delivery makes software releasable, often with an approval; continuous deployment automatically releases qualifying changes.

---

## 28. Interview Questions: Intermediate

**Q31. How do you trigger a workflow only on `main`?**  
Use `on: {push: {branches: [main]}}` (written as ordinary YAML for readability).

**Q32. How do you trigger on a scheduled basis?**  
Use `schedule` with cron, understanding UTC scheduling and default-branch behavior.

**Q33. How do you prevent duplicate feature-branch CI?**  
Use `pull_request` for feature branches and `push` only on `main`, with targeted workflow filters.

**Q34. How do you execute jobs sequentially?**  
Add `needs: prior-job` or `needs: [job1, job2]`.

**Q35. How do you share small metadata between jobs?**  
Declare step outputs via `$GITHUB_OUTPUT`, map to `jobs.<id>.outputs`, then read through `needs.<job>.outputs`.

**Q36. How do you share a binary between jobs?**  
Upload a build artifact in producer job and download it in consumer job, ideally verifying digest/provenance.

**Q37. How do you share a variable with later steps?**  
Append `KEY=value` to `$GITHUB_ENV`; it becomes available to subsequent steps in the job.

**Q38. What is `$GITHUB_STEP_SUMMARY`?**  
An environment file where steps append Markdown to create readable run summaries.

**Q39. How do you run tests across multiple Python versions?**  
Use `strategy.matrix.python` and `setup-python` with `${{ matrix.python }}`.

**Q40. What is `fail-fast`?**  
A matrix strategy controlling whether failures cancel other in-progress/queued matrix jobs.

**Q41. What is `max-parallel`?**  
The maximum number of matrix job variations executing simultaneously.

**Q42. How do you control concurrent deployments?**  
Use environment-scoped `concurrency` groups plus deployment protections; don't assume concurrency creates a perfect FIFO queue.

**Q43. What is `timeout-minutes`?**  
A job/step execution time limit to prevent runaway work.

**Q44. Why use `npm ci` instead of `npm install` for CI?**  
It uses the committed lockfile, provides a clean install, and helps reproducibility.

**Q45. How does caching work?**  
Store dependency data under a cache key (usually derived from OS and lockfile hash); future compatible runs may restore it.

**Q46. Why must build outputs not be stored only in cache?**  
Caches can be evicted, scoped or poisoned; artifacts and registries better fit durable release evidence.

**Q47. How do you build Docker images?**  
Use `docker build`/Buildx, or a reviewed Docker action, then push to a registry using an appropriately scoped identity.

**Q48. What is GHCR?**  
GitHub Container Registry: `ghcr.io`, supporting GitHub package permissions and OCI container images.

**Q49. What permissions are needed to push to GHCR using `GITHUB_TOKEN`?**  
Typically `packages: write`, plus repository/package access configuration; `contents: read` for checkout.

**Q50. What is OIDC?**  
OpenID Connect: the workflow obtains a signed identity token which cloud IAM can exchange for temporary credentials if trust conditions match.

**Q51. Why prefer OIDC for AWS?**  
Avoids permanent AWS keys in GitHub; short-lived credentials and IAM subject conditions restrict access.

**Q52. What is `id-token: write`?**  
Permission to request an OIDC token; it does **not** itself grant cloud write access.

**Q53. What is an environment?**  
A deployment target in GitHub with related configuration, secrets and protection policies.

**Q54. How do you add a manual deployment approval?**  
Configure required reviewers on a protected environment and reference that environment in the deployment job.

**Q55. What is the difference between `vars` and `secrets`?**  
`vars` holds non-secret configuration; `secrets` holds sensitive information subject to availability/security restrictions.

**Q56. How do you handle different staging and production configs?**  
Use separate environments, variables, IAM identities, namespaces/accounts and guarded deployment jobs.

**Q57. What is a reusable workflow?**  
A YAML workflow triggered using `workflow_call`, invoked at job level by other workflows.

**Q58. What is a composite action?**  
An action that packages a series of steps to run in the caller's job, defined using `runs: using: composite`.

**Q59. What does `secrets: inherit` do?**  
Passes available caller secrets to a reusable workflow when the organization/enterprise trust and GitHub rules allow it; use sparingly.

**Q60. How do you generate a release automatically?**  
Trigger on validated tags/releases, build immutable artifacts, apply release permissions, sign/attest, publish, and protect the release process.

**Q61. Can GitHub Actions deploy on-premises infrastructure?**  
Yes. Use correctly secured self-hosted runners or a trusted deployment gateway with required network access.

**Q62. How do you integrate a private database in integration tests?**  
For throwaway CI, use a service container; for private staging, use a protected runner/network and short-lived credentials.

**Q63. What is `persist-credentials: false` in checkout?**  
It prevents persisting checkout credentials for subsequent Git commands; useful when later steps should not retain repo auth.

**Q64. How do you diagnose a skipped job?**  
Inspect event filters, branch/path conditions, `if`, `needs` status, environment policy and available run annotations.

**Q65. How do you publish JUnit test results?**  
Generate reports in the test command, upload as artifacts, optionally use a vetted summary/check-reporting action with minimal permissions.

---

## 29. Interview Questions: Advanced and Scenarios

**Q66. How would you prevent a compromised community action from altering releases?**  
Pin to reviewed full SHA; enforce allowlists and least-privilege job permissions; separate trusted release jobs and use environment approvals, provenance and monitoring.

**Q67. Why is `pull_request_target` risky?**  
It has a base-repository privilege context. Checking out and executing fork code in that context can expose write tokens and secrets.

**Q68. What is a “pwn request”?**  
A PR workflow attack exploiting privileged execution of attacker-controlled code or data across a trust boundary.

**Q69. How can untrusted artifacts compromise a privileged workflow?**  
If the privileged job downloads and executes an attacker-produced script/binary, validation can be bypassed. Verify trust/provenance and avoid executing untrusted results.

**Q70. How do you secure a self-hosted runner for public repositories?**  
Prefer ephemeral job-scoped runners, isolate network/credentials, restrict runner groups, patch images, sanitize workspaces and avoid mixing untrusted PR code with sensitive jobs.

**Q71. What is ARC and when is it useful?**  
Actions Runner Controller, a Kubernetes-based controller for scaling ephemeral self-hosted runner pools when job demand varies.

**Q72. How do you keep production deployments serialized?**  
Use a production concurrency group, protected environment and deployment/state locking; ensure pending-run replacement semantics match release-order requirements.

**Q73. Can `github.run_number` be used as a global deployment ID?**  
No: it increments for a specific workflow, may contain gaps from skipped/failed runs or design changes, and isn't a cross-system transactional ID.

**Q74. How do you implement approval + rollback for EKS?**  
Validate artifact, use protected environment approval, deploy immutable digest with Helm/Kubernetes, wait on rollout/readiness, run smoke tests, monitor and revert to known-good digest/revision as needed.

**Q75. How do you prevent Terraform state corruption?**  
Use a remote lock-capable backend, restrictive IAM, unique state per environment, apply serialization, and disaster-recovery/versioning controls.

**Q76. Why should Terraform plans be treated as sensitive?**  
They may contain infrastructure details or sensitive values in machine-readable formats; store and distribute them as controlled artifacts.

**Q77. How would you deploy to a private EKS API endpoint?**  
Run a protected self-hosted runner or secure network path with VPC access, correct routing/DNS and EKS authorization; don't expose the private endpoint solely for CI convenience.

**Q78. OIDC shows `AccessDenied` on `AssumeRoleWithWebIdentity`. What do you inspect?**  
`id-token` permission, AWS OIDC provider/audience, IAM trust `sub`, branch/environment selection, immutable subject claim format, role ARN and token expiry.

**Q79. How do you enable cross-account ECR deployment?**  
Use cross-account IAM role assumption/registry policy as needed, short-lived credentials, narrow trust and least privilege; verify authorization on both ECR and destination runtime.

**Q80. What should you do when a deployment fails after half of the pods are updated?**  
Stop further rollout, inspect readiness/events, assess live SLOs, roll back if safe, preserve logs/evidence, investigate root cause and implement corrective validation.

**Q81. How can you reduce pipeline time by 40%?**  
Measure bottlenecks; cache dependencies and Docker layers safely, run independent tests in parallel, use prebuilt tool images, and remove redundant work; compare total wall-clock time.

**Q82. How do you keep caches safe across untrusted and privileged jobs?**  
Restrict cache sharing and keying, prevent privileged jobs from consuming untrusted executable cache content, and prefer rebuilding/verifying sensitive outputs.

**Q83. What is artifact attestation?**  
Signed build provenance linking artifact identity/digest to trusted build details, enabling verification before downstream use.

**Q84. How do you implement a software supply-chain gate?**  
Combine source review, locked dependencies, SAST/SCA, secret scan, image scan, signed SBOM/provenance, policy verification and restricted deploy identity.

**Q85. How do you design CI/CD for 30 microservices in a monorepo?**  
Use standardized reusable workflows, robust change detection, service-specific build/test matrices, immutable image tags, parallelization with resource limits and centralized policy/observability.

**Q86. How do you avoid deployment of the wrong Git commit?**  
Promote the digest built from the validated SHA, record provenance, pass output/digest directly into deploy jobs, and avoid mutable tags or rebuilding at deploy time.

**Q87. How do you design blue/green deployment?**  
Deploy to inactive pool, validate health and smoke tests, switch routing, monitor, then retire the old pool after a safe rollback window.

**Q88. How do you design canary deployment?**  
Deploy a small traffic slice, monitor latency/errors/business metrics with rollback thresholds, expand only if SLOs hold.

**Q89. How do you handle database migrations in CI/CD?**  
Use backward-compatible expand/migrate/contract patterns, backups, one-time migration coordination and rollback-aware release sequencing.

**Q90. A runner has no internet access. Can the workflow still run?**  
Only if it can reach required GitHub Actions services and any necessary internal mirrors/registries; actions/tools/dependencies must be available or mirrored. Fully isolated execution generally needs a compatible enterprise solution.

**Q91. How would you monitor self-hosted runner health?**  
Monitor service readiness, runner offline count, queue delay, disk/CPU/memory pressure, workspace growth, network failures, job duration and auth/token errors.

**Q92. What happens when two jobs use the same concurrency group?**  
GitHub enforces the group constraint; running and pending replacement/cancellation depend on the `cancel-in-progress` and queue configuration. It isn't equivalent to a traditional transactional lock.

**Q93. How do you handle a failed security scan that is a false positive?**  
Triaged exception with evidence, scope, owner and expiration; do not disable entire scan. Tighten rules/allowlist exact finding and track remediation.

**Q94. How do you protect shared workflow definitions from tampering?**  
Keep in a controlled repo, require code owners/branch protection, pin caller to immutable refs, audit changes and minimize permissions.

**Q95. Jenkins pipeline randomly hangs; would GitHub Actions automatically fix it?**  
No. Identify resource leaks, executor saturation, locks, external service dependencies and agent health. Re-architecting orchestration may help, but underlying workloads and infrastructure still need analysis.

**Q96. A workflow passes on `main` but fails on PRs from forks. Why?**  
Fork PR token/secrets restrictions, environment identity differences, private dependency access or policy. Keep PR validation credential-free and do privileged publishing only on trusted refs.

**Q97. How would you handle an AOSP build lasting many hours?**  
Use dedicated high-disk/CPU isolated self-hosted infrastructure, controlled caching, timeout/disk cleanup, immutable source manifests, verified artifacts and stage-level logging; consider specialized build systems where appropriate.

**Q98. How would you integrate Coverity and Black Duck?**  
Run language/build-aware capture and SAST with licensed tools; run dependency SCA on manifests/SBOMs; enforce agreed severity/standards gates; upload reports to controlled storage and track exceptions.

**Q99. What KPIs prove your CI/CD improvement?**  
Lead time to merge/deploy, job success rate, CI duration, queue time, deployment frequency, change failure rate, MTTR, cache-hit rate and manual effort removed.

**Q100. Design a secure production pipeline in 60 seconds.**  
“PR checks run least-privilege lint, tests and scans. `main` builds a signed/digest-addressed artifact, attests provenance and publishes to registry via OIDC. A separate deploy role gates promotion by protected environment approvals. EKS/Helm deploys the exact artifact, waits for health and SLO checks, and supports fast rollback. Runners are isolated; metrics and audit logs link commit → run → digest → deployment.”

---
## 30. Hands-on Interview Assignments and Model Answers

### Assignment A — Design a PR quality gate

**Prompt:** A Python project must run lint/tests, upload reports and block merges if checks fail.

**Expected design:** `pull_request` event, `contents: read`, `checkout`, `setup-python`, lockfile-based install, lint, tests, `if: always()` report upload, and branch protection requiring the job check. Explain that upload action alone does not make a branch protection rule; configure ruleset/required status checks separately. A full example appears in **Section 10**.

### Assignment B — Deploy Dockerized service to AWS with no static keys

**Prompt:** Build on merge to `main` and deploy to EKS.

**Expected design:** Test → build immutable image → AWS OIDC short-lived IAM role → ECR push → protected environment approval → deploy exact digest to EKS → rollout/smoke test → monitoring/rollback. Explain EKS access entries and GitHub OIDC `sub` trust conditions. A full workflow appears in **Section 15**.

### Assignment C — Build three sequential stages with artifacts

**Prompt:** Stage 1 validates; Stage 2 packages; Stage 3 publishes a release.

```yaml
name: Three Stage Pipeline
on:
  workflow_dispatch:
permissions:
  contents: read
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - run: echo 'Validated'

  build:
    needs: validate
    runs-on: ubuntu-latest
    steps:
      - name: Package
        run: |
          mkdir -p dist
          echo 'package' > dist/bundle.txt
      - uses: actions/upload-artifact@v7
        with:
          name: bundle
          path: dist/

  publish:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v8
        with:
          name: bundle
          path: dist
      - run: cat dist/bundle.txt
```

**Discuss:** The `publish` stage only verifies the handoff here; a real registry publish needs explicit credentials and appropriate trust boundaries.

### Assignment D — Run against three Java versions

**Prompt:** Test against JDK 17, 21 and 25.

```yaml
strategy:
  fail-fast: false
  matrix:
    java: ['17', '21', '25']
steps:
  - uses: actions/checkout@v7
  - uses: actions/setup-java@v4
    with:
      distribution: temurin
      java-version: ${{ matrix.java }}
  - run: mvn --batch-mode verify
```

Place this under a complete job (`runs-on`, `jobs`) in a full workflow. Ensure the project is compatible with each version and the distributions are supported.

### Assignment E — Nightly security scan

**Prompt:** Run dependency audit nightly, with manual dispatch.

```yaml
name: Nightly Audit
on:
  schedule:
    - cron: '20 1 * * *'
  workflow_dispatch:
permissions:
  contents: read
jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - name: Run dependency audit
        run: ./scripts/dependency-audit.sh
```

**Discuss:** Verify scan script, rate limits, alerts and report retention; `01:20` is UTC, not local time.

### Assignment F — Manual input with safe deployment selection

```yaml
name: Manual Staging Deployment
on:
  workflow_dispatch:
    inputs:
      target:
        required: true
        type: choice
        options: [dev, staging]
        default: staging
permissions:
  contents: read
jobs:
  show-target:
    runs-on: ubuntu-latest
    steps:
      - name: Validate selection
        env:
          TARGET: ${{ inputs.target }}
        run: |
          case "$TARGET" in
            dev|staging) echo "Allowed: $TARGET" ;;
            *) echo 'Invalid target' >&2; exit 1 ;;
          esac
```

Never use arbitrary user input to choose a production cloud role, deploy host or shell command.

### Assignment G — Repository path filters

**Prompt:** Only run backend tests for changes under `backend/**`.

**Answer:**

```yaml
on:
  pull_request:
    paths:
      - 'backend/**'
      - '.github/workflows/backend.yml'
```

**Caution:** Consider how skipped required checks affect merge rules.

### Assignment H — Explain failure diagnosis live

**Prompt:** Job `deploy` failed after build succeeds.

**Model response:** “I inspect the deployment job log and failing command, verify dependent artifact/digest, inspect environment approval and role trust, check EKS API reachability/RBAC, inspect pod events and rollout status, then roll back if health is affected. I capture reproducible facts instead of blindly rerunning.”

### Assignment I — Reusable organization pipeline

**Prompt:** Ten repositories duplicate the same Python tests.

**Answer:** Create a reusable `workflow_call` workflow in a controlled shared repository, declare input language version, require bounded permissions, invoke at job level, pin caller to a verified ref/SHA, and protect changes using CODEOWNERS and reviewed PRs. See **Section 18**.

### Assignment J — Separate PR and release privileges

**Prompt:** PRs must never get publish credentials.

**Answer:** Split jobs/workflows. `pull_request` receives only `contents: read` and no production secret/OIDC capability; trusted main-branch release job uses package write or a cloud OIDC role and environment protection. Reject untrusted artifacts/caches at privilege transition.

### Assignment K — Fix runner saturation

**Prompt:** Twenty jobs queue but only two self-hosted agents are available.

**Answer:** Inspect runner labels/groups/offline state and resource pressure; add capacity or ARC ephemeral scaling; prioritize short tests, cap matrix parallelism, provision more runners and monitor queue delay. Ensure scaling doesn't weaken runner isolation.

### Assignment L — Jenkins to GitHub Actions architecture

**Prompt:** Migrate a 3-stage Jenkins pipeline (prepare → AOSP build → Nexus upload).

**Model design:**

```text
GitHub workflow
  prepare (self-hosted, workspace sanity, manifest validation)
     |
     v
  build (high-disk isolated runner, toolchain, AOSP build, checksum)
     |
     v
  upload (separate protected release identity, retrieve verified artifact,
          upload Nexus, changelog + notification)
```

**Discuss:** Job isolation means a 100+ GB package might be better transferred via approved object storage or repository manager than GitHub artifact service; examine bandwidth, retention and integrity. Preserve original manifest revision; use strong versioning, bounded cleanup, and no duplicated failure alerts.

### Assignment M — Test PostgreSQL-based service

**Prompt:** A backend integration test needs PostgreSQL without a real DB account.

**Answer:** Use a disposable `postgres` **service container** with a disposable test database/password, wait for readiness, execute schema setup/tests, and destroy with the job. See **Section 19.2**.

### Assignment N — Incident: Wrong image deployed

**Prompt:** Production shows an older image than the release SHA.

**Model response:** Compare source SHA, image digest, registry manifest, deployment manifest and running pod `imageID`. Look for mutable `latest` tags, race conditions, re-build at deployment time and stale manifests. Redeploy the verified correct digest after approval; introduce attestation checks.

### Assignment O — Incident: Release paused by concurrency

**Prompt:** Production deployments appear skipped/canceled.

**Answer:** Audit `concurrency.group`, `cancel-in-progress`, pending queue semantics, workflow event/ref, required environment approvals and release ordering. Do not treat cancellation behavior as a deployment lock with guaranteed FIFO. Adopt a release queue strategy if strict order matters.

---

## 31. Frequently Seen Errors — Quick Fix Table

| Error / symptom | Likely cause | First check / fix |
|---|---|---|
| Workflow does not appear | YAML not under `.github/workflows` / workflow disabled | File path, branch, repository Actions policy |
| `Invalid workflow file` | YAML indentation, expression/keyword error | `actionlint`, exact error line |
| `Unexpected value 'uses'` | Wrong nesting (job vs step) | Reusable workflow `uses` belongs to job |
| `Resource not accessible by integration` | Missing token scope / fork restrictions | `permissions`, event, repo settings |
| `Permission denied (publickey)` | Wrong Git SSH credentials | Use checkout token/authorized key, repository permission |
| `Repository not found` | Private repo or wrong remote | Token scope and checkout `repository` input |
| `No runner matched labels` | Offline/unavailable runner | `runs-on` labels, group, runner registration |
| Job waiting for review | Environment protection | Check reviewers and allowed branches |
| OIDC `AccessDenied` | Wrong AWS role trust/`sub`/audience | Inspect identity claims and exact IAM condition |
| AWS `Unable to locate credentials` | Missing OIDC auth | `id-token: write`, configure AWS credentials step |
| ECR `no basic auth credentials` | Docker not logged in / wrong account | AWS session, ECR login and repository region |
| ECR `repository does not exist` | Missing repository or wrong name | Provision ECR repo, verify account/region |
| `kubectl: command not found` | CLI not installed | Install/pin `kubectl` version or runner image |
| `Unable to connect to the server` | EKS API unreachable / wrong context | Network/DNS/VPC kubeconfig endpoint |
| Kubernetes `Forbidden` | RBAC / EKS access misconfigured | IAM access entry, cluster role bindings |
| Pod `ImagePullBackOff` | Image missing/registry auth | Image reference, ECR access from nodes, tags |
| `npm ci` fails | Lockfile missing/mismatched | Update and commit `package-lock.json` |
| `pytest: command not found` | Dev deps not installed | Interpreter, venv, install step |
| Cache misses every time | Unstable key / lockfile not found | Key and `hashFiles()` path |
| Artifact not found | Upload skipped / wrong name/run | Prior steps, naming, retention, path |
| `No files were found` | Incorrect artifact path | `pwd`, `ls`, build output layout |
| Works on Ubuntu but fails Windows | Shell/path differences | `shell`, separators, case/path assumptions |
| Workflow runs twice | `push` and `pull_request` both match | Trigger policy and concurrency |
| Workflow canceled unexpectedly | Concurrency group overlap | Group key and cancel behavior |
| Terraform state lock error | Another apply / stale lock | Identify current writer, never blindly force-unlock |
| Docker `no space left on device` | Large layers / runner disk exhausted | Disk usage, cleanup, larger runner |
| Self-hosted action runtime error | Runner too old | Update runner to supported release |
| Cron not at local time | `schedule` is UTC | Convert to UTC; expect occasional delay |
| PR has no secrets | Fork security policy | Redesign PR checks to avoid secrets |
| Production env secret missing | Environment not selected / gated | Job `environment`, reviewers, scope |

---

## 32. Best Practices Checklist

### Workflow design

- [ ] Trigger policy matches repository workflow (PR checks vs trusted release).
- [ ] YAML is in `.github/workflows` and passes `actionlint`.
- [ ] Workflow names, job names, timeout limits and working directories are clear.
- [ ] Jobs use `needs` to express dependencies, not arbitrary sleeps.
- [ ] Independent safe jobs run in parallel; unnecessary duplicates are removed.
- [ ] Critical errors fail the pipeline; `continue-on-error` is justified.
- [ ] Artifact paths are checked, and report uploads occur after failed tests.
- [ ] Release artifacts have immutable SHA/digest identification.

### Security

- [ ] Workflow/job `GITHUB_TOKEN` permissions are least privilege.
- [ ] Trusted actions are verified and pinned to approved full SHAs in production.
- [ ] Action dependency updates go through security review.
- [ ] Fork PR code never executes in privileged event contexts.
- [ ] Untrusted strings are not injected into shell expressions.
- [ ] OIDC trust policies match expected repository, branch and/or environment claims.
- [ ] Production cloud identities use least-privilege IAM and short-lived sessions.
- [ ] Protected environments/rulesets/required reviewers are configured.
- [ ] No secrets are committed, cached, echoed, or placed in artifacts.
- [ ] Self-hosted runners are isolated, hardened and regularly updated.
- [ ] SAST, SCA, container scanning and SBOM/provenance controls fit risk policy.

### Reliability and deployment

- [ ] Tests and scans are mandatory merge/deploy gates.
- [ ] Builds are reproducible: versions and dependencies pinned/locked.
- [ ] Correct job outputs/artifact digests are handed to deploy jobs.
- [ ] Staging and production are logically/credential isolated.
- [ ] Deployments use approved release metadata and guarded concurrency.
- [ ] Readiness, liveness, smoke tests and observability are available.
- [ ] Rollback or safe roll-forward path is documented.
- [ ] Terraform state and deployments have concurrency/locking protection.
- [ ] Logs/metrics link workflow run, source SHA, image digest and deployment.

### Performance

- [ ] Track CI wall time, queue time, failure rate and total cost.
- [ ] Cache dependency data safely using lockfiles.
- [ ] Use optimized Docker layers / `.dockerignore`.
- [ ] Separate fast PR checks from heavyweight scheduled scans.
- [ ] Autoscale ephemeral runners where appropriate.
- [ ] Artifact retention matches organizational policy.

---

## 33. 7-Day Interview Preparation Plan

| Day | Topics | Hands-on deliverable | Interview rehearsal |
|---|---|---|---|
| 1 | Architecture, `on`, YAML, jobs, steps | Hello + Python CI | Q1–Q30 |
| 2 | Contexts, outputs, dependencies, conditions | Three-job artifact handoff | Q31–Q40 |
| 3 | Matrix, caching, service containers, concurrency | Node matrix + PostgreSQL test | Q41–Q49 |
| 4 | Docker, registries, GHCR, artifacts | Build and push image to GHCR | Q50–Q60 |
| 5 | AWS OIDC, ECR, EKS, environment gates | Safe cloud deploy design | Q66–Q80 |
| 6 | Terraform, reusable workflows, security, ARC | Terraform validation + shared test workflow | Q81–Q100 |
| 7 | Troubleshooting, Jenkins comparison, mock interview | Diagnose one intentional pipeline failure | Three full system-design answers |

**Useful practical GitHub portfolio repo:**

```text
github-actions-devops-lab/
├── .github/workflows/
│   ├── python-ci.yml
│   ├── node-matrix.yml
│   ├── docker-ghcr.yml
│   ├── terraform.yml
│   └── reusable-test.yml
├── services/
├── infrastructure/
├── scripts/
├── screenshots/   # Optional sanitized screenshots, no secrets
└── README.md       # Architecture, setup, screenshots, threat model
```

**Resume/interview project statement (adapt to measured reality):**

> “Designed a GitHub Actions CI/CD pipeline with automated tests, Docker Buildx, secure OIDC federation to AWS ECR/EKS, immutable image tagging, protected environment deployments, artifact provenance and deployment health checks. Optimized repeat builds through dependency caching and concurrency controls.”

Don't claim production ownership, savings or measured speedups without evidence.

---

## 34. One-Page Revision Cheat Sheet

```text
Workflow path        .github/workflows/<name>.yml
Trigger              on: push | pull_request | workflow_dispatch | schedule
Executor             jobs.<id>.runs-on
Steps                jobs.<id>.steps
Action               steps[*].uses: owner/repo@SHA
Shell command        steps[*].run
Job dependency       needs: [build, test]
Conditional          if: ${{ github.ref == 'refs/heads/main' }}
Job output           echo "key=value" >> "$GITHUB_OUTPUT"
Step-to-step env     echo "key=value" >> "$GITHUB_ENV"
Read job output      ${{ needs.build.outputs.key }}
Repo variable        ${{ vars.MY_VAR }}
Secret               ${{ secrets.MY_SECRET }}
Matrix value         ${{ matrix.os }}
AWS OIDC             permissions: { id-token: write, contents: read }
GHCR publish         permissions: { packages: write, contents: read }
Prevent overlap      concurrency: { group: deploy-prod, cancel-in-progress: false }
Artifact upload      actions/upload-artifact
Artifact download    actions/download-artifact
Dependency cache     setup-* cache or actions/cache
Approval             environment: production (+ configured protection)
Reusable workflow    on: workflow_call + jobs.<id>.uses
Composite action     action.yml + runs: { using: composite, steps: [...] }
Local lint           actionlint .github/workflows/*.yml
CLI run listing      gh run list
CLI logs             gh run view RUN_ID --log
CI architecture      trigger -> test -> build -> scan -> publish -> deploy
Release identity     source SHA -> signed/digest artifact -> production record
```

### Top 12 must-know interview answers

1. **`run` vs `uses`:** command vs reusable action.
2. **Job vs step:** runner-isolated job vs sequential work inside a job.
3. **`needs`:** job ordering and output dependencies.
4. **Cache vs artifact:** speed vs output/evidence sharing.
5. **`GITHUB_TOKEN`:** scoped GitHub API identity; explicitly minimize permissions.
6. **OIDC:** short-lived cloud credentials based on signed workload identity.
7. **`pull_request_target`:** dangerous if it executes untrusted code with base-repo privileges.
8. **Matrix:** parallel OS/runtime combinations.
9. **Reusable vs composite:** entire shared job flow vs reusable step sequence.
10. **Environment:** protected deployment target and scoped configuration.
11. **Self-hosted runner:** more control but more security/maintenance responsibility.
12. **Release best practice:** build once, scan, attest, promote immutable digest, verify health and retain rollback.

### 45-second senior-level answer

> “My GitHub Actions design starts with fast PR checks using minimal token scopes and isolated runners. On a protected branch, successful tests and security gates produce a versioned immutable image. I use OIDC for AWS role assumption, push it to ECR, and promote the exact digest through environment-gated deployments to EKS. I control concurrency, verify rollout and application health, capture provenance/SBOM evidence, and support rollback. For heavier builds, I use ephemeral self-hosted runners and monitor queue time, reliability and CI cost.”

---

## 35. Official Documentation and References

**Primary GitHub references (reviewed October 2026):**

1. [GitHub Actions overview](https://docs.github.com/en/actions)
2. [Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax)
3. [Workflow concepts](https://docs.github.com/en/actions/concepts/workflows-and-actions/workflows)
4. [Events that trigger workflows](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows)
5. [Contexts reference](https://docs.github.com/en/actions/reference/workflows-and-actions/contexts)
6. [Expressions](https://docs.github.com/en/actions/reference/workflows-and-actions/expressions)
7. [Reusable workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows)
8. [Security for GitHub Actions](https://docs.github.com/en/actions/how-tos/secure-your-work)
9. [GitHub OIDC for AWS](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws)
10. [Actions Runner Controller](https://docs.github.com/en/actions/concepts/runners/actions-runner-controller)
11. [Self-hosted runner reference](https://docs.github.com/en/actions/reference/runners/self-hosted-runners)
12. [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions)
13. [GitHub CLI](https://cli.github.com/manual/)
14. [GitHub Actions examples](https://github.com/actions/starter-workflows)

**Official / vendor-maintained actions used in examples:**

- [actions/checkout](https://github.com/actions/checkout)
- [actions/setup-python](https://github.com/actions/setup-python)
- [actions/setup-node](https://github.com/actions/setup-node)
- [actions/setup-java](https://github.com/actions/setup-java)
- [actions/upload-artifact](https://github.com/actions/upload-artifact)
- [actions/download-artifact](https://github.com/actions/download-artifact)
- [actions/cache](https://github.com/actions/cache)
- [docker/setup-buildx-action](https://github.com/docker/setup-buildx-action)
- [docker/login-action](https://github.com/docker/login-action)
- [docker/build-push-action](https://github.com/docker/build-push-action)
- [aws-actions/configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials)
- [aws-actions/amazon-ecr-login](https://github.com/aws-actions/amazon-ecr-login)
- [hashicorp/setup-terraform](https://github.com/hashicorp/setup-terraform)
- [actionlint](https://github.com/rhysd/actionlint)

> **Maintenance reminder:** Before implementing an example in a real environment, verify current action release, runner compatibility, workflow syntax, AWS/GitHub OIDC claim format, supported Terraform version, organization policy, cloud permissions, release approvals and security controls. Code samples are training templates, not a substitute for reviewing a production-specific threat model.

---

**End of `Github_Action.md` — GitHub Actions Interview Handbook.**

# Jenkins: Complete DevOps & SRE Interview Notes

> **GitHub-ready guide | Fundamentals → Advanced | Hands-on Jenkinsfiles | Troubleshooting | Interview Q&A**  
> Audience: DevOps Engineer, SRE, Platform Engineer, Cloud Engineer and Release Engineer.  
> **Last reviewed:** October 2026. Command and plugin behavior can vary by Jenkins/plugin version.

---

## Table of Contents

1. [What Is Jenkins?](#1-what-is-jenkins)
2. [CI, CD and the SDLC](#2-ci-cd-and-the-sdlc)
3. [Jenkins Architecture](#3-jenkins-architecture)
4. [Install Jenkins on Ubuntu](#4-install-jenkins-on-ubuntu)
5. [Jenkins Dashboard and Job Types](#5-jenkins-dashboard-and-job-types)
6. [Jenkins Pipeline Fundamentals](#6-jenkins-pipeline-fundamentals)
7. [Complete Declarative CI Pipeline](#7-complete-declarative-ci-pipeline)
8. [Declarative vs Scripted Pipeline](#8-declarative-vs-scripted-pipeline)
9. [Pipeline Directives and Steps](#9-pipeline-directives-and-steps)
10. [Parameters, Environment and Credentials](#10-parameters-environment-and-credentials)
11. [Build Triggers, Webhooks and Cron](#11-build-triggers-webhooks-and-cron)
12. [Controller, Agents and SSH Setup](#12-controller-agents-and-ssh-setup)
13. [Parallel Stages, Matrix and Concurrency](#13-parallel-stages-matrix-and-concurrency)
14. [Shared Libraries](#14-shared-libraries)
15. [Multibranch Pipelines and Git Integration](#15-multibranch-pipelines-and-git-integration)
16. [Gerrit and Large Multi-Repo/AOSP Builds](#16-gerrit-and-large-multi-repoaosp-builds)
17. [Docker Integration](#17-docker-integration)
18. [Kubernetes, Helm and GitOps](#18-kubernetes-helm-and-gitops)
19. [Artifacts, Nexus and JFrog](#19-artifacts-nexus-and-jfrog)
20. [Downstream Jobs, Build IDs and Notifications](#20-downstream-jobs-build-ids-and-notifications)
21. [Security and DevSecOps](#21-security-and-devsecops)
22. [Backups, Upgrades, Scaling and Monitoring](#22-backups-upgrades-scaling-and-monitoring)
23. [Linux and Jenkins Command Cheat Sheet](#23-linux-and-jenkins-command-cheat-sheet)
24. [Troubleshooting: Common Production Incidents](#24-troubleshooting-common-production-incidents)
25. [Interview Questions with Short Answers](#25-interview-questions-with-short-answers)
26. [Scenario-Based Interview Questions](#26-scenario-based-interview-questions)
27. [One-Minute Revision](#27-one-minute-revision)
28. [Official Documentation and Learning Path](#28-official-documentation-and-learning-path)

---

## 1. What Is Jenkins?

**Jenkins** is an open-source automation server used to build, test, scan, package and deliver software. It is widely used to implement **Continuous Integration (CI)** and **Continuous Delivery/Deployment (CD)**. It is extensible through plugins and normally uses **Pipeline as Code** through a version-controlled `Jenkinsfile`.

**Simple example:** A developer pushes code → webhook notifies Jenkins → Jenkins checks out code → runs tests → builds image → scans image → publishes artifact → deploys to a test environment → reports status.

**Why Jenkins?**

- Automates repeated engineering tasks and reduces manual errors.
- Supports distributed build agents and heterogeneous Linux/Windows environments.
- Integrates with GitHub, GitLab, Gerrit, Docker, Kubernetes, cloud providers, artifact registries and quality tools.
- Enables approvals, retries, timeouts, credentials, scheduling and notifications.
- Produces auditable logs, test results and build artifacts.

**Jenkins is not:** a source-control system, a general-purpose container orchestrator, or a complete observability platform. It integrates with these systems.

---

## 2. CI, CD and the SDLC

| Concept | Meaning | Example |
|---|---|---|
| CI | Integrate code frequently and validate it automatically | Every pull request runs unit tests |
| Continuous Delivery | Every successful change is deployable; release may require approval | Package pushed to staging, manual production approval |
| Continuous Deployment | Every approved/eligible change deploys to production automatically | Successful main-branch change deploys without manual approval |
| Pipeline as Code | Pipeline definition stored in source control | `Jenkinsfile` committed with application code |
| GitOps | Git defines desired infrastructure/app configuration | Jenkins updates a Helm values repo; Argo CD reconciles |

```text
Developer -> Git Push / Gerrit Patch Set
                    |
               Webhook/Event
                    |
              Jenkins Pipeline
                    |
  Checkout -> Lint -> Test -> Quality/SAST -> Build
                    |
             Artifact Registry
                    |
         Staging -> Approval -> Production
                    |
         Monitoring / Alerting / Rollback
```

**Interview distinction:** CI detects integration problems early. CD automates the reliable promotion and deployment process. They are related but not interchangeable.

---

## 3. Jenkins Architecture

```text
                         USERS / SCM / WEBHOOKS
                                  |
                                  v
                      +-----------------------+
                      |   JENKINS CONTROLLER  |
                      | UI, queue, scheduler, |
                      | config, job orches-   |
                      | tration, credentials  |
                      +-----------------------+
                         /         |         \
                  SSH / inbound agent / Kubernetes cloud
                       /           |           \
            +------------+  +------------+  +------------+
            | Linux Agent|  | Windows    |  | Ephemeral  |
            | Java, Maven|  | Agent      |  | K8s Pod    |
            +------------+  +------------+  +------------+
                   |             |                 |
              Compile/Test   Windows builds   Containerized jobs
                   +-------------+-----------------+
                                 |
                          Artifact Registry
```

**Main components:**

- **Controller:** Schedules and coordinates builds, serves UI and APIs, stores configuration and execution metadata.
- **Agent/Node:** Machine or container that provides executors and a workspace to run builds.
- **Executor:** A build execution slot assigned to a node; setting an agent to 2 executors allows up to 2 builds concurrently on it, subject to other restrictions.
- **Label:** A node selector such as `linux`, `docker`, `high-memory`, `android-builder`.
- **Workspace:** Filesystem location where a job checks out code and produces temporary files.
- **Queue:** Jobs waiting for available compatible executors/resources.
- **`JENKINS_HOME`:** Controller state: job configurations, build records, plugins, credentials metadata and related data; protect it with backups and restricted access.

**Best practice:** Use `0` executors on the built-in/controller node and execute builds on agents. Isolate untrusted workloads and do not give agents direct access to controller secrets or home directory.

**Agent connection methods:** SSH-launched agents, inbound agents (including WebSocket transport), and dynamically provisioned agents (e.g. Kubernetes). Connection path/ports depend on configuration.

---

## 4. Install Jenkins on Ubuntu

> These steps use the official Jenkins **LTS** package repository. Run on a dedicated VM or test machine; confirm your target Jenkins release supports the selected Java version.

### Prerequisites

- Ubuntu/Debian system with sufficient CPU, RAM and disk.
- Java runtime (official guidance currently uses OpenJDK 21).
- Network access to package sources; a browser-accessible Jenkins port.
- `sudo` privileges, and a reverse proxy/TLS for production exposure.

```bash
sudo apt update
sudo apt install -y fontconfig openjdk-21-jre wget
java -version

sudo install -d -m 755 /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo 'deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/' \
  | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install -y jenkins
sudo systemctl enable --now jenkins
sudo systemctl status jenkins
```

Open `http://<server-ip>:8080` **only on a trusted network** for first setup; do not expose a non-TLS admin UI to the public internet.

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword  # Local administrator only
sudo journalctl -u jenkins -n 100 --no-pager
sudo ss -lntp | grep ':8080'
```

**Initial setup:** unlock → install suggested plugins → create admin → configure Jenkins URL → configure security and agents. Set a stable external URL under **Manage Jenkins → System**, preferably behind an HTTPS reverse proxy.

**Important paths (package install):**

| Path | Purpose |
|---|---|
| `/var/lib/jenkins` | Typical `JENKINS_HOME` |
| `/etc/apt/sources.list.d/jenkins.list` | APT repository config |
| `journalctl -u jenkins` | systemd service logs |
| `/var/lib/jenkins/secrets` | Sensitive internal files; do not commit/copy into repo |

**Docker-based lab** (persistent named volume; do not mount Docker socket casually):

```bash
docker volume create jenkins_home
docker run -d --name jenkins-lab --restart unless-stopped \
  -p 8080:8080 -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts-jdk21

docker logs -f jenkins-lab
```

Docker image tags and Jenkins Java requirements change over time: verify them before deploying.

---

## 5. Jenkins Dashboard and Job Types

| Job type | Best use |
|---|---|
| Freestyle | Simple or legacy UI-configured task |
| Pipeline | CI/CD with versioned Groovy Pipeline definition |
| Multibranch Pipeline | Automatically discovers branches/PRs containing a Jenkinsfile |
| Folder | Organize jobs, RBAC scopes, credentials and libraries |
| Organization Folder | Discover repositories/branches across an SCM organization (plugin-dependent) |

**Frequently used menus:**

- **New Item:** create Freestyle/Pipeline/Multibranch job.
- **Manage Jenkins → Plugins:** install/update plugins after compatibility checks.
- **Manage Jenkins → Credentials:** create scoped secret entries.
- **Manage Jenkins → Nodes:** configure agents, executors and labels.
- **Manage Jenkins → System:** general configuration, email and some integration settings.
- **Manage Jenkins → Tools:** configure JDK, Maven and other tool installations.
- **Job → Pipeline Syntax:** Snippet Generator for exact installed plugin syntax.
- **Job → Build with Parameters / Console Output / Pipeline Steps:** execute and diagnose jobs.

**Build statuses:** `SUCCESS`, `FAILURE`, `UNSTABLE`, `ABORTED`, `NOT_BUILT`. `UNSTABLE` often indicates failed tests or quality threshold; it is not always a compile failure.

---

## 6. Jenkins Pipeline Fundamentals

A Pipeline defines **stages** (logical phases) containing **steps** (operations). The preferred approach for most teams is a **Declarative Pipeline** in a `Jenkinsfile` versioned alongside code.

### Minimal Jenkinsfile

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'echo "Building $(date -u)"'
            }
        }
    }

    post {
        success { echo 'Pipeline succeeded' }
        failure { echo 'Pipeline failed' }
        always  { echo "Result: ${currentBuild.currentResult}" }
    }
}
```

**Important:** `checkout scm` assumes the Pipeline was configured with SCM context (e.g., **Pipeline script from SCM** or Multibranch). Otherwise, explicitly configure the repository with `git`/`checkout`.

**Built-in environment variables:**

| Variable | Meaning |
|---|---|
| `env.BUILD_NUMBER` | Jenkins-generated build sequence (string) |
| `env.BUILD_URL` | Link to build |
| `env.JOB_NAME` | Job name/path |
| `env.WORKSPACE` | Current workspace path on agent |
| `env.NODE_NAME` | Node running this agent block |
| `env.BRANCH_NAME` | Multibranch branch name (when provided) |
| `env.CHANGE_ID` | Change/PR identifier (where supported) |
| `env.GIT_COMMIT` | Commit, usually supplied by Git checkout/plugin |

### How Jenkins executes a Pipeline

1. Obtain Jenkinsfile from source control.
2. Parse/validate its syntax and schedule work.
3. Allocate matching agent/executor (or provision an ephemeral agent).
4. Create/use workspace, run stages/steps and gather output.
5. Publish test reports/artifacts, notify and apply retention policy.

---

## 7. Complete Declarative CI Pipeline

**Example:** Java/Maven project, Linux agent, JUnit report and archive. Assumes Jenkins tools named `jdk21` and `maven3` exist under **Manage Jenkins → Tools**; appropriate Pipeline and JUnit plugins are installed; the project builds with Maven.

```groovy
pipeline {
    agent { label 'linux' }

    tools {
        jdk 'jdk21'
        maven 'maven3'
    }

    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '10'))
        skipDefaultCheckout(true)
    }

    parameters {
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests')
        choice(name: 'TARGET_ENV', choices: ['dev', 'qa'], description: 'Target')
    }

    environment {
        APP_NAME = 'demo-service'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git rev-parse --short HEAD'
            }
        }

        stage('Build & Test') {
            steps {
                script {
                    if (params.RUN_TESTS) {
                        sh 'mvn -B -ntp clean verify'
                    } else {
                        sh 'mvn -B -ntp -DskipTests package'
                    }
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Approval for QA') {
            when { expression { params.TARGET_ENV == 'qa' } }
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input message: 'Promote to QA?', ok: 'Approve'
                }
                echo 'Promotion approved; deploy command goes here.'
            }
        }
    }

    post {
        always {
            junit testResults: 'target/surefire-reports/*.xml', allowEmptyResults: true
            echo "Build ${env.BUILD_NUMBER}: ${currentBuild.currentResult}"
        }
        success {
            echo 'All configured CI stages passed.'
        }
        failure {
            echo "Check the failed stage: ${env.BUILD_URL}console"
        }
        cleanup {
            deleteDir()
        }
    }
}
```

**Know the tradeoffs:**

- `disableConcurrentBuilds()` prevents simultaneous builds of **that job**, not all jobs across Jenkins.
- `timeout` stops stuck builds after a limit; `retry` can be used for transient infrastructure failures.
- `junit` publishes report XML and can mark the build unstable for test failures; `allowEmptyResults: true` avoids failure when none exist but can conceal missing tests, so require reports in critical pipelines.
- `archiveArtifacts` copies matching workspace files into Jenkins-managed build archives; use Nexus/Artifactory or object storage for long-term artifact distribution.
- `input` pauses for approval; configure submitter permissions/approvers as required.
- Cleanup occurs after post-steps. Don't delete workspaces containing artifacts still needed by downstream jobs; publish or stash/unstash appropriately.

---

## 8. Declarative vs Scripted Pipeline

| Feature | Declarative | Scripted |
|---|---|---|
| Structure | `pipeline { agent ... stages ... }` | `node { stage(...) ... }` |
| Flexibility | Structured, predictable, easier to enforce standards | Flexible Groovy flow and advanced logic |
| Error handling | `post`, `when`, `options`, plus `script` | Groovy `try/catch/finally` |
| Recommendation | Default for most CI/CD | For dynamic orchestration that warrants extra complexity |

**Scripted Pipeline:**

```groovy
node('linux') {
    try {
        stage('Checkout') { checkout scm }
        stage('Build')    { sh 'make all' }
        currentBuild.result = 'SUCCESS'
    } catch (Exception ex) {
        currentBuild.result = 'FAILURE'
        throw ex
    } finally {
        echo "Finished with ${currentBuild.currentResult}"
    }
}
```

**Interview:** You can embed limited Groovy flow inside Declarative `steps { script { ... } }`, but keep logic small. Prefer Bash/Python scripts for complex processing so Jenkins controller isn't overloaded with Pipeline Groovy computations.

---

## 9. Pipeline Directives and Steps

| Directive/step | Purpose | Example |
|---|---|---|
| `agent` | Pick execution environment | `agent { label 'docker' }` |
| `stages` / `stage` | Group execution flow | `stage('Test')` |
| `steps` | Execute work | `sh 'pytest -q'` |
| `post` | Handle build outcome | `post { failure { echo 'Failed' } }` |
| `environment` | Set environment vars | `FOO = 'bar'` |
| `parameters` | User-provided job inputs | `string(name: 'BRANCH', ...)` |
| `when` | Conditional stage execution | `when { branch 'main' }` |
| `tools` | Inject configured tool installations | `jdk 'jdk21'` |
| `timeout` | Limit time | `timeout(time: 20, unit: 'MINUTES')` |
| `retry` | Repeat on failure | `retry(3) { sh './fetch.sh' }` |
| `parallel` | Run independent stages concurrently | `parallel { stage('Unit') { ... } ... }` |
| `stash` / `unstash` | Transfer selected files among stages/nodes in same run | `stash name: 'pkg', includes: 'dist/**'` |
| `archiveArtifacts` | Persist build output in Jenkins | `archiveArtifacts 'dist/**'` |
| `junit` | Publish JUnit test reports | `junit 'reports/*.xml'` |
| `build` | Start another Jenkins job | `build job: 'deploy', wait: true` |
| `input` | Request manual approval | `input message: 'Deploy?'` |
| `lock` | Share resource serialization (plugin) | `lock(resource: 'test-rig') { ... }` |
| `withCredentials` | Scope secrets to execution | `withCredentials([...]) { ... }` |

### `when` example

```groovy
stage('Release') {
    when {
        allOf {
            branch 'main'
            expression { params.RUN_RELEASE }
        }
    }
    steps {
        echo 'Release is permitted.'
    }
}
```

`branch` generally requires multibranch context. `params.RUN_RELEASE` must be declared as a parameter elsewhere.

### Error handling

```groovy
stage('Integration Test') {
    steps {
        retry(2) {
            timeout(time: 15, unit: 'MINUTES') {
                sh './scripts/integration-test.sh'
            }
        }
    }
}
```

**Use retry only for potentially transient problems** (network/download/ephemeral agent failures), not deterministic compile/test failures; blind retries hide defects and waste capacity.

---

## 10. Parameters, Environment and Credentials

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'VERSION', defaultValue: '1.0.0', description: 'Release version')
        booleanParam(name: 'DRY_RUN', defaultValue: true, description: 'Preview only')
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Environment')
    }
    stages {
        stage('Display Inputs') {
            steps {
                echo "Version: ${params.VERSION}; Target: ${params.ENV}"
            }
        }
    }
}
```

### Secure token handling (Credentials Binding plugin)

1. Add secret as **Manage Jenkins → Credentials** in the correct scope.
2. Use a stable credential ID, such as `artifact-upload-token`.
3. Bind only within the stage that requires the secret.
4. Let the **shell**, not Groovy interpolation, expand the secret.

```groovy
stage('Call API') {
    steps {
        withCredentials([string(credentialsId: 'artifact-upload-token', variable: 'TOKEN')]) {
            sh '''
                set +x
                curl --fail --silent --show-error \
                  -H "Authorization: Bearer $TOKEN" \
                  https://api.example.invalid/health
            '''
        }
    }
}
```

`example.invalid` is an intentional placeholder; replace it with a trusted endpoint.

### SSH private key credential

```groovy
withCredentials([sshUserPrivateKey(
    credentialsId: 'deploy-ssh-key',
    keyFileVariable: 'SSH_KEY',
    usernameVariable: 'SSH_USER'
)]) {
    sh '''
      ssh -i "$SSH_KEY" -o StrictHostKeyChecking=yes \
        "$SSH_USER@deployment.example.com" 'hostname'
    '''
}
```

The agent needs a verified SSH `known_hosts` entry for the target. **Never disable host-key verification in production** to make SSH errors disappear.

**Do not:** commit secrets to Jenkinsfiles, print `env`, use `echo $TOKEN`, pass secrets through Groovy GStrings (`sh "curl ... ${TOKEN}"`), or expose trusted credentials to unreviewed PR code. Masking is not a complete security boundary.

**Cloud credentials best practice:** On AWS, prefer scoped IAM roles / workload identity where possible over long-lived static keys.

---

## 11. Build Triggers, Webhooks and Cron

### Trigger options

- **Manual:** `Build Now` or `Build with Parameters`.
- **Webhook:** SCM sends an event to Jenkins; usually quicker and cheaper than polling.
- **SCM polling:** `pollSCM('H/5 * * * *')` checks whether code changed.
- **Cron:** `cron('H 2 * * *')` schedules a job regardless of new commits.
- **Upstream:** job starts after a designated job reaches an appropriate result.
- **Gerrit events:** patchset created, comment added, ref updated, change merged (requires plugin/server configuration).

### Jenkins cron syntax

```text
MINUTE HOUR DAY_OF_MONTH MONTH DAY_OF_WEEK
 0-59  0-23     1-31       1-12    0-7
```

| Expression | Meaning |
|---|---|
| `30 21 * * *` | Every day at **21:30**, in the configured scheduler time zone |
| `4 22 * * *` | Every day at **22:04** |
| `H H * * *` | Once daily at a stable, hashed time |
| `H/15 * * * *` | Approximately every 15 minutes, hashed minutes |
| `0 9 * * 1-5` | 09:00 Monday–Friday |
| `H 2 * * 0` | Once each Sunday during 02:00 hour |

**Important:** Jenkins uses its configured time zone for scheduling, often the controller JVM/system timezone; never assume IST or UTC. Confirm controller timezone and job configuration. Jenkins cron is **5 fields**, not Linux crontab's optional user/command format; `H` is a stable hash of job identity (not the literal hour).

```groovy
pipeline {
    agent any
    triggers { cron('30 21 * * *') }
    stages {
        stage('Schedule Check') { steps { sh 'date; date -u' } }
    }
}
```

Some Jenkins cron trigger configurations support a leading `TZ=Asia/Kolkata` line; validate with the Jenkins job's **Schedule** preview and installed-version documentation rather than assuming server timezone.

**Webhook checklist:**

1. SCM integration plugin and connection configured.
2. Event endpoint reachable from SCM (HTTPS, firewall and proxy correct).
3. Credentials/token/secret validation set when supported.
4. Repository/project/branch filter matches the event.
5. Job is enabled; Jenkins has free executor; inspect webhook delivery logs and Jenkins system logs.

---

## 12. Controller, Agents and SSH Setup

**Scenario:** Add a new Ubuntu builder to an existing controller.

### On the agent

```bash
# Create a dedicated, least-privileged account if needed.
sudo useradd -m -s /bin/bash jenkins-agent
sudo install -d -m 700 -o jenkins-agent -g jenkins-agent \
  /home/jenkins-agent/.ssh

# Install a Jenkins-compatible Java version (check current requirements).
sudo apt update
sudo apt install -y openjdk-21-jre openssh-server git
java -version
sudo systemctl status ssh

# Prepare agent root directory owned by agent account.
sudo mkdir -p /home/jenkins-agent/agent
sudo chown -R jenkins-agent:jenkins-agent /home/jenkins-agent/agent
```

### Key-based SSH authentication

1. Generate a **dedicated** ED25519 key pair on an approved admin machine; keep private key secure.
2. Append only its public key to the agent account's `~/.ssh/authorized_keys`.
3. Store the **private key** in Jenkins credentials as **SSH Username with private key**.
4. Verify the agent SSH host fingerprint and add it to controller-managed known hosts.

```bash
# Example: on an approved administrator machine
ssh-keygen -t ed25519 -f ~/.ssh/jenkins_agent_ed25519 -C 'jenkins-agent'

# Public key -> add on the agent, under jenkins-agent user
# ~/.ssh/authorized_keys (mode 600, ~/.ssh mode 700)
chmod 600 ~/.ssh/jenkins_agent_ed25519
ssh -i ~/.ssh/jenkins_agent_ed25519 jenkins-agent@AGENT_IP 'java -version'
```

**On Jenkins UI:** Manage Jenkins → Nodes → New Node → Permanent Agent → set Remote root directory, number of executors, labels → **Launch agents via SSH** → host, credential and verified host-key strategy → Save → Launch Agent.

**Security:** `ssh-keyscan` can collect a host key, but does **not** prove the key is authentic; compare fingerprint through a trusted out-of-band channel before trusting it. Never choose non-verifying host-key strategy merely to bypass a mismatch.

**Agent offline / authentication failure checklist:**

```bash
ping -c 2 AGENT_IP                 # ICMP may be blocked; not decisive
nc -vz AGENT_IP 22               # test TCP port
ssh -vvv -i ~/.ssh/jenkins_agent_ed25519 jenkins-agent@AGENT_IP
sudo journalctl -u ssh -n 80 --no-pager
ls -ld ~/.ssh && ls -l ~/.ssh/authorized_keys
java -version
df -h
free -h
```

Check user/key mismatch, permissions, authorized_keys, firewall, DNS, SELinux/AppArmor policy, Java compatibility, launch path and agent logs. Restrict SSH to the controller/approved management network.

### Node selection in Pipeline

```groovy
pipeline {
    agent none
    stages {
        stage('Linux Build') {
            agent { label 'linux && high-memory' }
            steps { sh 'hostname; nproc' }
        }
        stage('Deployment') {
            agent { label 'deploy-agent' }
            steps { sh 'echo deploying' }
        }
    }
}
```

An agent with 1 executor normally handles 1 build at a time. Heavy builds may need dedicated capacity even if CPU count is high. Labels select capabilities; they do not enforce resource isolation by themselves.

---

## 13. Parallel Stages, Matrix and Concurrency

### Parallel tests

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            parallel {
                stage('Unit') {
                    steps { sh './scripts/unit-tests.sh' }
                }
                stage('Integration') {
                    steps { sh './scripts/integration-tests.sh' }
                }
            }
        }
    }
}
```

Parallel stages require adequate capacity and separate/safe resources. Avoid having parallel jobs modify the same files, cache or deployment environment without coordination.

### Mutual exclusion across *different* jobs

Requires the **Lockable Resources** plugin:

```groovy
stage('Test on Hardware') {
    steps {
        lock(resource: 'shared-device-lab') {
            sh './flash-and-test.sh'
        }
    }
}
```

**Compare:**

- `disableConcurrentBuilds()` → prevents overlapping builds of the same job.
- `lock(resource: '...')` → serializes access to a named shared resource across cooperating jobs.
- `milestone` → helps prevent older builds from continuing past designated points after newer builds advance (requires its plugin).
- `parallel` → decreases elapsed time only where tasks are independent and resources are available.

### Matrix builds

Matrix can test combinations such as JDK 21 and 25, or multiple OS/architecture configurations. Use a matrix only when you actually have compatible agents/tools and need combination coverage.

---

## 14. Shared Libraries

Use Shared Libraries to standardize pipeline stages, error handling, alerts, SCM helpers and build/upload workflows across repositories.

```text
jenkins-shared-library/
├── vars/
│   ├── sayHello.groovy
│   └── sayHello.txt
├── src/
│   └── org/example/BuildHelper.groovy
└── resources/
    └── templates/email.txt
```

**`vars/sayHello.groovy`:**

```groovy
def call(String who = 'Team') {
    echo "Hello, ${who}!"
}
```

**Consumer `Jenkinsfile`:**

```groovy
@Library('team-ci-lib@main') _
pipeline {
    agent any
    stages {
        stage('Welcome') {
            steps { sayHello('DevOps') }
        }
    }
}
```

Configure the trusted SCM source under **Manage Jenkins → System → Global Pipeline Libraries** or equivalent folder-level settings. Prefer **version-pinned/reviewed library releases** for production and carefully restrict who can change trusted libraries. Groovy in Pipeline runs on controller resources; move expensive operations to agents/external scripts.

**Interview answer:** A shared library reduces duplicated Jenkinsfile code and enables consistent tooling, approvals and security policy across teams.

---

## 15. Multibranch Pipelines and Git Integration

**Pipeline script from SCM:** One defined job points to a `Jenkinsfile` in a configured repository/branch.

**Multibranch Pipeline:** Jenkins discovers branch/PR heads and automatically creates associated branch jobs where a `Jenkinsfile` is found (behavior depends on SCM plugin and branch-source traits).

**Advantages:** branch-specific pipelines, pull-request validation, branch cleanup, clear per-branch history.

### Git checkout / webhook flow

```text
Pull Request / Push
     |
     v
SCM Webhook -> Jenkins Multibranch Job
     |                |
     |             Checkout
     |                |
     +----< Build status/result reported to SCM
                      |
                   Tests/Scan
```

```groovy
stage('Checkout') {
    steps {
        checkout scm
        sh 'git log -1 --oneline'
    }
}
```

**Checklist:** Git plugin, credentials, webhook, branch discovery, target `Jenkinsfile` path, network access, webhook secret and minimal permissions. Ensure untrusted PRs cannot access deployment secrets.

---

## 16. Gerrit and Large Multi-Repo/AOSP Builds

This section is especially useful in automotive/embedded engineering where AOSP, Yocto, Gerrit and long-running multi-repository integration builds are common.

### Gerrit event flow

```text
Developer uploads change (refs/for/branch)
                   |
              Patch Set Created
                   |
         Gerrit Trigger Plugin
                   |
        Jenkins integration job
                   |
   Fetch patch set -> Apply on correct base
                   |
         Compile -> Static Analysis
                   |
          Report Verified status
```

**Common events:** `patchset-created`, `comment-added`, `change-merged`, `ref-updated`. Gerrit Trigger plugin allows project, branch and event filters. An SSH event stream or other configured Gerrit connection may be used; see plugin documentation.

**Gerrit labels:** `Verified +1` generally indicates CI passed; `Verified -1` failed (labels/ranges are repository-configurable). `Code-Review +1/-1/+2/-2` expresses reviewer approval/blocking depending on project permissions and configuration. CI should not conflate Code-Review with Verified.

### `repo` utility workflow

```bash
# Illustrative placeholders: replace with your approved manifest URL/branch.
repo init -u ssh://gerrit.example.com/platform/manifest -b release-branch
repo sync -j8 --fail-fast
repo forall -c 'git rev-parse --short HEAD'
```

**Typical AOSP/Yocto flow:**

1. Checkout manifest and `repo sync` thousands of Git projects.
2. Detect manifest changes and decide whether the full build is required.
3. Apply selected patchsets/topics to the correct source directories.
4. Execute builds inside a controlled toolchain/container.
5. Collect system images, logs, software bills of materials, static-analysis reports and metadata.
6. Upload packages to Nexus/Artifactory and publish the exact integration ID.
7. Publish email/SCM verification result and retention details.

**Failure traps:** stale `.git/index.lock` after a crashed Git process, missing remote refs, corrupt partial checkout, out-of-disk/inodes, `repo sync` concurrency, invalid manifests, build cache and Bazel cache problems. Only remove lock files after confirming no Git/repo process is running. Do not delete entire source directories without understanding uncommitted work and download cost.

### Example integration build numbering

```groovy
stage('Set Integration Label') {
    steps {
        script {
            // Keep native Jenkins BUILD_NUMBER as the source of truth.
            env.INTEGRATION_ID = "S01-${env.BUILD_NUMBER}"
            currentBuild.displayName = env.INTEGRATION_ID
        }
        echo "Release integration: ${env.INTEGRATION_ID}"
    }
}
```

**Key point:** A display prefix (`S01-`) does not change Jenkins' underlying numeric `BUILD_NUMBER`. If a chain has an orchestrator job and separate build job, choose a single owning job for release/integration IDs. Don't derive release IDs from the first stage if failed preflight runs should not consume integration numbers.

### First build vs incremental build

- **Full build:** clean/reproducible output after significant manifest/toolchain changes.
- **Incremental build:** reuses build state for faster iterations but must be invalidated on relevant input changes.
- **Release build:** traceable source revisions, pinned dependencies, checksums and immutable artifacts are more important than cache reuse.

---

## 17. Docker Integration

**Jenkins can run within a Docker container, use Docker-hosted agents, or execute pipeline stages in Docker containers.** These are different patterns.

### Dockerized build stage

Requires a compatible agent with a Docker Engine/remote Docker service and the **Docker Pipeline** plugin.

```groovy
pipeline {
    agent any
    stages {
        stage('Test in Container') {
            agent {
                docker {
                    image 'python:3.12-slim'
                    reuseNode true
                }
            }
            steps {
                sh 'python --version'
                sh 'python -m compileall -q .'
            }
        }
    }
}
```

### Build and publish a container image

```groovy
stage('Build Container') {
    steps {
        sh 'docker build -t my-service:${BUILD_NUMBER} .'
    }
}
```

Publishing requires an authorized container registry identity; prefer an ephemeral credential and an appropriately isolated builder (for example, a dedicated BuildKit/Kaniko-style workflow that matches your platform). Always scan dependencies and images before promotion.

**Common errors:** `permission denied` to `/var/run/docker.sock`, Docker daemon unavailable, missing `docker` executable, Docker-in-Docker filesystem/network mismatch, insufficient disk, container user permissions. Giving a user access to the Docker socket usually grants root-equivalent host control—do it only on explicitly trusted, dedicated agents.

---

## 18. Kubernetes, Helm and GitOps

**Two separate patterns:**

1. **Kubernetes Jenkins agents:** Jenkins controller provisions short-lived pods to execute jobs using the Kubernetes plugin.
2. **Kubernetes deployments:** Jenkins produces a container image and deploys/updates resources or commits desired state for a GitOps controller.

### Build -> Helm workflow

```text
Jenkins -> Test/Scan -> Build Image -> Push Immutable Tag/Digest
                                              |
                                +-------------+-------------+
                                |                           |
                         Direct Helm                  GitOps (preferred
                         deployment                   in many platforms)
                                |                           |
                     Kubernetes Cluster        Update Git manifest
                                                            |
                                                         Argo CD
                                                            |
                                                    Kubernetes Cluster
```

**Helm example (approved environment only):**

```bash
kubectl config current-context
helm lint ./chart
helm template demo ./chart --set image.tag=123
helm upgrade --install demo ./chart \
  --namespace demo --create-namespace --set image.tag=123 \
  --wait --timeout 10m
kubectl -n demo rollout status deployment/demo --timeout=300s
```

**Production controls:** use scoped kubeconfig/workload identity, RBAC, namespace isolation, environment approvals, signed images and deployment auditability. Jenkins agents should not receive unrestricted cluster-admin access.

**Why GitOps?** Jenkins handles CI/package publication while Argo CD continuously reconciles cluster state from Git, minimizing direct pipeline-to-cluster access and configuration drift.

---

## 19. Artifacts, Nexus and JFrog

**Artifact** = compiled, testable and deployable output such as `.jar`, `.war`, `.zip`, container image, Android image or tarball.

| Tool/feature | Typical use |
|---|---|
| `archiveArtifacts` | Store selected files within Jenkins build history |
| `stash` / `unstash` | Pass small/intermediate files inside a single Pipeline execution |
| Nexus Repository | Long-lived binary/package repository |
| JFrog Artifactory | Artifact repository and promotion workflows |
| Container Registry | Store versioned container images |

**Best practices:** use immutable version/tag or digest, publish SHA-256 checksum, capture commit and build metadata, have retention policies, scan dependencies, restrict write access, and verify artifact after upload.

```bash
sha256sum dist/app.tar.gz > dist/app.tar.gz.sha256
```

### Nexus upload sample (illustrative)

```groovy
stage('Publish') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'nexus-uploader',
            usernameVariable: 'NEXUS_USER',
            passwordVariable: 'NEXUS_PASSWORD'
        )]) {
            sh '''
              set +x
              curl --fail --show-error --silent \
                --user "$NEXUS_USER:$NEXUS_PASSWORD" \
                --upload-file dist/app.tar.gz \
                'https://nexus.example.com/repository/releases/app.tar.gz'
            '''
        }
    }
}
```

**Note:** The server, repository path, auth policy and deployment API are configuration-specific. `--upload-file` is appropriate for a compatible raw repository endpoint, not every repository format. Never hard-code credentials or publish to a mutable `latest` path as the only release record.

---

## 20. Downstream Jobs, Build IDs and Notifications

### Multi-job orchestration

```text
Job A: Preflight / Manifest Decision
               |
               v
Job B: Compile / Package  <-- OWN integration number
               |
               v
Job C: Upload / Changelog / Publication
```

**Recommended ownership model:**

- Job A checks inputs, branch, manifest, capacity and workspace state.
- Job B uses its **native `BUILD_NUMBER`** as integration identity.
- Job C receives integration ID + immutable artifact manifest from Job B.
- Each job reports **its own actual failures**; a downstream failure should not cause duplicate upstream alerts.
- Use immutable metadata files, artifacts or a designated handoff directory where dependencies are clear; ensure cleanup, permissions and concurrency safety.

### Trigger another job

Requires the **Pipeline: Build Step** plugin (typically part of common Pipeline installations):

```groovy
stage('Start Build') {
    steps {
        script {
            def downstream = build(
                job: 'Compile_Job',
                wait: true,
                propagate: false,
                parameters: [
                    string(name: 'SOURCE_BRANCH', value: 'main')
                ]
            )
            echo "Downstream build #${downstream.number}: ${downstream.result}"
            if (downstream.result != 'SUCCESS') {
                // Handle routing/state deliberately; do not blindly send
                // another notification for a failure already reported there.
                error("Downstream returned ${downstream.result}")
            }
        }
    }
}
```

**Gotcha:** The code above intentionally marks upstream as failed. If upstream has `post { failure { emailext(...) } }`, it can trigger a **second alert**. For one alert per root-cause failure, track whether the failed stage is local versus propagated and suppress upstream email for propagated failures (or centralize reporting with deduplication).

### Email notification

The **Email Extension** plugin adds the `emailext` step; SMTP must be configured.

```groovy
post {
    failure {
        emailext(
            to: 'team@example.com',
            subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "Check console: ${env.BUILD_URL}console"
        )
    }
}
```

To show **Stage Detail**, explicitly record the current stage in an environment variable before potentially failing operations; a generic `post` block does not automatically know which earlier stage caused the error in every scenario.

```groovy
stage('Build') {
    steps {
        script { env.ACTIVE_STAGE = 'Build' }
        sh './scripts/build.sh'
    }
}
```

Then include `env.ACTIVE_STAGE` in the error email. For more robust recording, wrap individual stages, capture exception context and retain error metadata across handoffs. If multiple stages run in parallel, use per-branch error tracking rather than a shared mutable `ACTIVE_STAGE` variable.

### Native Build Number vs Display Name

- `env.BUILD_NUMBER`: numeric job-local build number assigned by Jenkins.
- `currentBuild.displayName`: user-visible label; editable.
- `env.BUILD_TAG`: Jenkins-generated identifier from job/build.
- `currentBuild.description`: optional human-readable build context.

To change the next native build number, use an authorized Jenkins administrative mechanism and back up job metadata first; do not casually edit `nextBuildNumber` while builds run. Each job has its own independent counter.

---

## 21. Security and DevSecOps

**Must-follow controls:**

1. Enable authentication (SSO/OIDC/LDAP/other approved identity provider), enforce least-privilege authorization/RBAC.
2. Run work on dedicated agents, not on the controller.
3. Apply Jenkins LTS + tested plugin updates; remove unneeded/vulnerable plugins.
4. Use scoped credentials, short-lived access and workload identities. Rotate secrets.
5. Isolate untrusted PR/branch builds; never expose privileged credentials to attacker-controlled Jenkinsfiles.
6. Restrict inbound ports, SSH, egress and artifact destinations; use HTTPS with managed certificates.
7. Verify SSH host fingerprints rather than disabling verification.
8. Back up `JENKINS_HOME` and protect keys/configuration. Test restores.
9. Validate scripts/parameters and pin build dependencies/images when reproducibility matters.
10. Integrate testing, SAST, SCA, secrets checks, SBOM, container scans and policy checks.

**Typical plugin/tool ecosystem** (install only if needed and compatible):

| Category | Common examples |
|---|---|
| SCM | Git, GitHub Branch Source, GitLab integration, Gerrit Trigger |
| Pipeline | Pipeline suite, Pipeline: Stage View, Shared Libraries |
| Security | Credentials, Credentials Binding, Role-based Authorization Strategy |
| Agent | SSH Build Agents, Kubernetes |
| Containers | Docker Pipeline |
| Quality | JUnit, Coverage, SonarQube Scanner; Coverity integrations/scripts |
| Artifacts | Artifactory/Nexus integration or authenticated CLI/API |
| Notifications | Email Extension, supported chat integrations |
| Scheduling/concurrency | Lockable Resources |
| Configuration | Configuration as Code (JCasC) |

**Note:** A plugin has its own update cadence, transitive dependencies and possible security advisories. Fewer supported plugins generally mean less operational risk.

---

## 22. Backups, Upgrades, Scaling and Monitoring

### Backup strategy

- Back up the controller's persistent data and plugin/config metadata **including secret material required for restore**, in encrypted access-controlled backup storage.
- Protect backup credentials and test complete recovery periodically.
- Coordinate backups with Jenkins state changes or use a consistent snapshot approach; copying live data without consistency controls can be unreliable.
- Store Jenkinsfile/shared libraries in Git and infrastructure configuration as code where possible.
- Externalize large artifacts; do not retain every log/image forever on the controller.

### Upgrades

1. Read Jenkins LTS release notes, minimum Java requirement and security advisories.
2. Test the same core/plugins on a staging copy with representative jobs.
3. Back up and plan downtime/rollback.
4. Update Jenkins core and compatible plugins; upgrade agents as required.
5. Verify authentication, jobs, webhooks, credential access, agent connectivity and notifications.

### Scaling

- Add appropriate **agents**, not unlimited controller executors.
- Prefer ephemeral/isolated agents for variable build load.
- Limit executors for memory-heavy jobs, manage workspaces and disk.
- Reduce unnecessary Pipeline Groovy controller computation and excessive polling.
- Apply per-job log/build retention and artifact offloading.
- Watch queue time, agent occupancy, job duration, flaky tests and executor utilization.

### Observability

Track Jenkins uptime, queue length, executor availability, failure rate, p95 build time, disk space, JVM heap/GC, thread counts and agent connectivity. Metrics integrations may export to **Prometheus/Grafana**. Configure alerts based on operational impact rather than raw noisy failures.

---

## 23. Linux and Jenkins Command Cheat Sheet

### Controller service

```bash
sudo systemctl status jenkins
sudo systemctl restart jenkins      # Planned operational action; disrupts active work
sudo journalctl -u jenkins -f
sudo journalctl -u jenkins --since '1 hour ago'
systemctl cat jenkins
java -version
```

### Process, ports, system capacity

```bash
ps aux | grep '[j]enkins'
ss -lntp | grep ':8080'
curl -I http://127.0.0.1:8080/login
free -h
df -h
df -i
uptime
top
```

### Network and agent

```bash
getent hosts jenkins.example.com
nc -vz AGENT_IP 22
ssh -vvv USER@AGENT_IP
curl -Iv https://git.example.com/
```

### Jenkins REST API (authenticated example)

```bash
# Use an API token, not an admin password; replace URL/user/token.
# Avoid exposing token in shell history/process lists: use a scoped credential
# or secure automation client instead of typing secrets on a shared host.
curl --fail --silent --show-error \
  --user "$JENKINS_USER:$JENKINS_API_TOKEN" \
  'https://jenkins.example.com/api/json?tree=mode,numExecutors'
```

For POST requests that change Jenkins state, follow the configured authentication and CSRF/crumb behavior; use the REST/API client patterns recommended for your Jenkins version.

### Useful Jenkinsfile snippets

```groovy
echo "Build #${env.BUILD_NUMBER}"
sh 'hostname; pwd; whoami'
sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
archiveArtifacts artifacts: 'dist/**', fingerprint: true
stash name: 'workspace-files', includes: 'dist/**'
unstash 'workspace-files'
currentBuild.displayName = "S01-${env.BUILD_NUMBER}"
```

---

## 24. Troubleshooting: Common Production Incidents

### 1. Job stuck in queue: “Waiting for next available executor”

**Cause candidates:** matching agent offline, all executors busy, wrong labels, throttling/locks, insufficient resources.  
**Fix path:** examine queued reason → node labels/status → executor counts → locks → agent logs → capacity policy. Do not blindly increase executors on saturated machines.

### 2. Agent SSH authentication failed

**Causes:** wrong Jenkins credential, incorrect remote user, missing public key, mode/ownership error, revoked key, host-key failure.  
**Diagnose:** agent SSH logs + `ssh -vvv`, permissions and Jenkins connection logs. Verify host fingerprints. Do not bypass identity checks.

### 3. Jenkins controller becomes unresponsive

**Causes:** JVM memory/GC pressure, too many Groovy-heavy Pipeline operations, exhausted disk, slow plugins, queue load, thread contention.  
**Diagnose:** `systemctl`, `journalctl`, heap/thread diagnostics, metrics, disk; consider graceful safe shutdown/restart only after collecting evidence. Identify the root cause before normalizing a restart-based fix.

### 4. Cron did not trigger at expected IST time

**Causes:** scheduler uses another timezone, malformed 5-field expression, disabled job, maintenance/offline state, no executor or job configuration mismatch.  
**Diagnose:** controller JVM timezone + job schedule preview + trigger history + Jenkins logs. For 21:30 local configured time: `30 21 * * *`.

### 5. `Permission denied` while running Docker

**Cause candidates:** agent account lacks Docker socket permission or daemon not running.  
**Check:** `id`, `ls -l /var/run/docker.sock`, `docker info`, `systemctl status docker`.  
**Fix:** prefer a secure dedicated builder; Docker socket grants powerful host access. If changing group membership, start a new login/service process so permissions refresh.

### 6. Git repo sync fails with `.git/index.lock` exists

**Causes:** simultaneous Git operations or stale file after crash.  
**Fix:** verify no active Git/repo process uses the directory; then remove stale lock **only when safe**, retry a constrained sync. Avoid parallel jobs sharing a mutable checkout.

### 7. `repo sync`: cannot find remote ref

**Causes:** manifest references deleted/nonexistent branch or stale Gerrit mirror, wrong remote/path, permissions.  
**Fix:** `git ls-remote` to confirm ref; review manifest/project revision, credentials and mirror. Re-sync or re-clone the affected project only after fixing the underlying ref problem. Do not alter manifest blindly.

### 8. Pipeline reports success, but artifact is missing

**Causes:** wrong workspace/path, packaging not produced, path glob mismatch, cleanup too early, upload failure ignored.  
**Fix:** `find dist target -maxdepth 3 -type f`, assert file exists, capture checksum and upload response status. Treat missing mandatory artifact as failure.

### 9. Downstream job failed, duplicate email sent

**Cause:** downstream sends a failure email; upstream propagates failure and sends another.  
**Fix:** define notification ownership, use `propagate: false` where suitable, record downstream result, suppress the upstream notification for propagated failures and retain overall failure visibility.

### 10. `RejectedAccessException` / Groovy sandbox error

**Cause:** sandbox requires a method/script approval, often from nonstandard Groovy API use.  
**Fix:** refactor toward supported Pipeline steps/approved shared library; do not approve dangerous signatures without a security review.

### 11. `No such DSL method` in Jenkinsfile

**Cause:** typo, plugin missing/incompatible, step used outside valid context.  
**Fix:** check Pipeline Syntax/Installed Plugins, restart only when necessary and verify plugin dependencies.

### 12. Job suddenly fails after plugin or Java upgrade

**Cause:** compatibility/API mismatch between core, plugins, Java and agent runtime.  
**Fix:** review release notes, plugin dependency health and Java versions; restore a tested combination from backup if required.

### 13. Test report exists but Jenkins shows no tests

**Cause:** wrong `junit` glob, wrong working directory, report generated in a container but not preserved in agent workspace.  
**Fix:** `find . -name '*.xml'`, validate report path and report format, publish before workspace cleanup.

### 14. Out of disk / inode exhaustion

**Cause:** huge AOSP workspaces, Docker images, old archives, build logs or retained caches.  
**Check:** `df -h`, `df -i`, `du -sh /path/*`.  
**Fix:** targeted cleanup and retention, archive externally; never mass-delete active build workspaces or controller data.

### 15. Intermittent “connection reset” during long builds

**Cause:** network instability, agent JVM crash/OOM, load-balancer timeout for some transports, host reboot.  
**Fix:** inspect agent/controller logs and memory/network telemetry, stabilize agent connectivity and use reproducible restartable stages.

---

## 25. Interview Questions with Short Answers

**Q1. What is Jenkins?**  
An automation server that orchestrates CI/CD pipelines, builds, tests, scanning, packaging and release workflows.

**Q2. What is the controller-agent architecture?**  
The controller schedules and manages jobs; agents provide isolated executors/workspaces where actual builds run.

**Q3. Controller vs agent vs executor?**  
Controller coordinates; agent is worker machine/container; executor is a concurrent work slot on a node.

**Q4. What is a Jenkinsfile?**  
A version-controlled Groovy Pipeline-as-Code definition.

**Q5. Declarative vs Scripted?**  
Declarative is structured and easier to maintain; Scripted provides flexible Groovy-based control flow.

**Q6. Difference between `stage` and `step`?**  
Stage groups a logical phase; step executes an action such as checkout or shell command.

**Q7. What is `agent any`?**  
Jenkins may choose any compatible agent with an available executor; not necessarily the controller.

**Q8. What happens with `agent none`?**  
No top-level executor is allocated; individual stages needing execution must specify agents.

**Q9. What is a label?**  
Selector identifying agents with capabilities such as `linux && docker`.

**Q10. What is `post`?**  
Defines actions after pipeline/stage outcome, such as publishing results, email and cleanup.

**Q11. What is a webhook?**  
An SCM HTTP event callback that starts or updates Jenkins automation when code changes.

**Q12. `pollSCM` vs `cron`?**  
Polling checks for SCM changes and triggers only when changes exist; cron triggers at scheduled times regardless of changes.

**Q13. What is Jenkins `H` in cron?**  
A stable hash-derived time value that distributes jobs and reduces spikes.

**Q14. What is `BUILD_NUMBER`?**  
The native, job-local Jenkins build sequence counter. Changing display name doesn't change it.

**Q15. How do you save secrets?**  
Jenkins Credentials with appropriate scope, `withCredentials`, least privilege and short lifetime; never in source code.

**Q16. What is an upstream/downstream job?**  
A job starts or coordinates another; pass explicit parameters/artifact references and handle result propagation.

**Q17. What is a Shared Library?**  
Versioned reusable Jenkins Groovy code, often under `vars/`, `src/`, `resources/`.

**Q18. What is a Multibranch Pipeline?**  
A job that discovers SCM branches/PRs and runs branch-specific Jenkinsfiles.

**Q19. How do you run stages concurrently?**  
Use Declarative `parallel` (or matrix) with enough agents/resources and no unsafe shared state.

**Q20. How do you avoid simultaneous use of a lab rig?**  
Use Lockable Resources plugin around that resource; optionally combine with job-level concurrency control.

**Q21. What is `stash` vs `archiveArtifacts`?**  
Stash moves intermediate files between stages in one run; archive retains build files for later retrieval.

**Q22. How does Jenkins work with Docker?**  
Run builds in Docker agents/containers, build and publish images or operate Docker-based deployment workflows.

**Q23. How does Jenkins integrate with Kubernetes?**  
Provision ephemeral K8s agent pods or deploy via kubectl/Helm/GitOps with scoped identities.

**Q24. Jenkins vs GitHub Actions/GitLab CI?**  
Jenkins is a flexible self-hostable automation server with broad plugins; integrated CI services offer tighter hosted SCM workflows. Choose based on governance, legacy integrations, operational capacity and cost.

**Q25. What is a quality gate?**  
A policy stopping promotion if tests, static analysis, security or coverage thresholds fail.

**Q26. Why do pipelines become `UNSTABLE`?**  
Often test/report results or explicit `unstable`/warning steps; inspect console and post actions.

**Q27. How do you troubleshoot Jenkins slowness?**  
Check queue, agent utilization, CPU/RAM/GC, plugin health, disk/inodes, network and controller Groovy load.

**Q28. How do you restart Jenkins safely?**  
Plan or use quiet-down/graceful restart mechanisms when supported, communicate impact, save diagnostics and verify agent reconnects/build recovery.

**Q29. What is Jenkins Configuration as Code?**  
The JCasC plugin defines supported controller configuration in YAML for repeatability and review.

**Q30. Can Jenkins provide zero-downtime deployments itself?**  
Jenkins can orchestrate rollout steps, but availability comes from the target platform's deployment architecture (rolling, blue-green, canary, health checks and rollback).

**Q31. What is the Pipeline Groovy CPS engine?**  
A continuation-passing-style execution mechanism used to persist/resume Pipeline control flow; certain objects aren't serializable and can cause `NotSerializableException`.

**Q32. How do you avoid secret leaks from PR builds?**  
Separate trusted/untrusted execution, block privileged credentials from untrusted code, apply reviews and minimal SCM token scopes.

**Q33. How is Jenkins related to SRE?**  
It automates safe releases and operational workflows; SRE uses reliability objectives, monitoring, rollback, incident response and toil reduction around those processes.

**Q34. Jenkinsfile syntax validator?**  
Use built-in Declarative linter or Pipeline Syntax tooling supported by your Jenkins version, plus code review and test jobs.

**Q35. What belongs in a deployment handoff?**  
Exact artifact digest, source revision, build metadata, environment, approvals, quality status and rollback information.

---

## 26. Scenario-Based Interview Questions

### Scenario A: A 6 PM release breaks production. First action?

**Answer:** Acknowledge the incident, stop further risky rollouts, assess user impact/health metrics, trigger the documented rollback or mitigation, involve on-call stakeholders and keep timeline/evidence. After restoring service, compare deployment diff and complete a blameless RCA. Do not spend the first 30 minutes only reading Jenkins console while users are impacted.

### Scenario B: A private-subnet agent cannot upload to S3; no internet access.

**Answer:** Verify IAM role and DNS, S3 endpoint access and route tables. Prefer an **S3 Gateway VPC endpoint** for in-region S3 traffic from the VPC, with appropriate route tables, endpoint policy and bucket policy. A NAT gateway is not required for this pattern; confirm regional/cross-account constraints.

### Scenario C: Gerrit sends a patchset event, but Jenkins doesn't start.

**Answer:** Confirm Gerrit connection/event stream and plugin status → matching project/branch/event rule → Jenkins job enabled → permissions → queue/agents → Gerrit/plugin logs. Ensure the patch set belongs to the configured project and branch.

### Scenario D: Build jobs run 2 hours and memory spikes.

**Answer:** Use dedicated high-memory agents, control executor count and concurrency, collect RAM/GC/IO telemetry, inspect caching and workspace size, move expensive processing from Pipeline Groovy to scripts, and set timeouts/retention. Don't simply add more parallel executors.

### Scenario E: A patchset build succeeds, but a nightly integration fails.

**Answer:** Compare exact manifest revision, branch, patchsets, dependency pins, toolchain/container image, environment variables, incremental caches and artifacts. Reproduce on clean pinned inputs before attributing cause to Jenkins.

### Scenario F: How would you design a secure 3-job release chain?

**Answer:**

1. **Preflight:** validate inputs, SCM refs, repository freshness and capacity.
2. **Build:** own the integration `BUILD_NUMBER`, build and test on isolated agent, emit immutable metadata and artifact digest.
3. **Publish:** verify digest, upload to registry, generate changelog and promotion record.
4. Guard shared resources with locks; use timeouts/parameter validation; preserve logs and hashes.
5. Avoid duplicate notifications: failure alerts from the owning job, with upstream reporting only the chain state.

### Scenario G: Pipeline hangs because all agents are busy.

**Answer:** Check queue reason, labels, executors, node health and resource locks; prioritize business-critical jobs and scale isolated agents as justified. Raising executor count without RAM/CPU headroom may worsen throughput.

### Scenario H: Docker build works manually but fails from Jenkins.

**Answer:** Compare effective agent user, environment, Docker socket access, daemon endpoint, working directory, proxy, network, mount and PATH. Jenkins often runs as a different user from the developer. Fix permissions securely rather than using root indiscriminately.

### Scenario I: API token accidentally printed in a log.

**Answer:** Revoke/rotate immediately, assess exposure and audit usage, restrict log access, remove sensitive retained output where appropriate, fix the pipeline to avoid interpolation/echo and use short-lived credentials. Masking alone doesn't solve secret disclosure.

### Scenario J: How do you prove your pipeline reduced release time?

**Answer:** Define baseline and timeframe; track median/p95 lead time, manual steps, reruns, queue time, failed deployments, MTTR and operator effort. Show before/after numbers plus controls for workload changes. Quantify reliability, not just pipeline execution speed.

### Answer framework for experience questions (STAR)

- **Situation:** Releases involved multi-repo source, manual checks and long builds.
- **Task:** Make integrations reliable, traceable and repeatable.
- **Action:** Built Gerrit-triggered Jenkins pipelines, added node selection/locks, artifacts, static analysis, log collection and stage-aware failure handling.
- **Result:** Cite **your verified metrics only** (e.g. deployment time and MTTR changes), explain how measured and what you'd improve next.

---

## 27. One-Minute Revision

```text
JENKINS = Automation server / CI-CD orchestration
CONTROLLER = UI + queue + config + scheduler
AGENT = Build worker         EXECUTOR = concurrent build slot
JENKINSFILE = Pipeline as Code (Groovy)
DECLARATIVE = pipeline {}     SCRIPTED = node {}
STAGE = logical phase         STEP = one action
SCM TRIGGERS = webhook / pollSCM / cron / upstream / Gerrit
H IN CRON = stable hash, not a literal hour
BUILD_NUMBER = job's native numeric counter
DISPLAY NAME = changeable label, not underlying build number
CREDENTIALS = scoped secret access; never commit or echo secrets
LOCK = serialize shared resources; disableConcurrentBuilds = one job
STASH = within one run; ARCHIVE = keep build outputs
SHARED LIBRARY = reusable vars/src/resources Groovy
MULTIBRANCH = per-branch/PR pipeline discovery
DOCKER = reproducible isolated build runtime (secure builder needed)
KUBERNETES = ephemeral agents or deployment target
ARTIFACT = immutable version + digest + source commit
SRE FOCUS = low-risk releases, rollback, monitoring, reduced toil
FIRST DEBUG STEPS = console + queue + agent + credentials + disk/logs
```

**Five habits that make you production-ready:**

1. Every build is traceable to exact Git/manifest revisions and artifact digests.
2. Builds run on scoped, isolated agents, not the controller.
3. Failures are visible, attributed to a stage and notified only once.
4. Deployments have approvals/quality checks and a tested rollback path.
5. Automation is measured by reliability and saved engineering time, not just scripts written.

---

## 28. Official Documentation and Learning Path

**Official references (use them to confirm your installed version and plugin options):**

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [Using a Jenkinsfile](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/)
- [Pipeline Best Practices](https://www.jenkins.io/doc/book/pipeline/pipeline-best-practices/)
- [Using Jenkins Agents](https://www.jenkins.io/doc/book/using/using-agents/)
- [Jenkins Security](https://www.jenkins.io/doc/book/security/)
- [Jenkins Controller Isolation](https://www.jenkins.io/doc/book/security/controller-isolation/)
- [Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
- [Docker Pipeline](https://www.jenkins.io/doc/book/pipeline/docker/)
- [Jenkins Ubuntu/Linux Installation](https://www.jenkins.io/doc/book/installing/linux/)
- [Gerrit Trigger Plugin](https://plugins.jenkins.io/gerrit-trigger/)
- [Jenkins Plugins Index](https://plugins.jenkins.io/)

### Suggested hands-on practice (in order)

- [ ] Install a Jenkins LTS lab and configure your first Pipeline from Git.
- [ ] Create a multibranch job with webhook and build result reporting.
- [ ] Add an SSH agent using verified host keys and scoped credentials.
- [ ] Run unit tests, JUnit publishing, archiving and artifacts in a clean pipeline.
- [ ] Implement parallel stages, retry/timeout and a shared lock.
- [ ] Create `vars/` shared-library code with a reusable step.
- [ ] Build/test inside a Docker agent with appropriate security controls.
- [ ] Publish a versioned artifact to a test repository.
- [ ] Implement a three-job build/publish workflow with one authoritative build ID.
- [ ] Add quality gates, notifications, metrics, documented rollback and restore tests.

---

> **Repository tip:** Keep this file as `README.md` in a `Jenkins/` folder, or as the repository root README if this project is Jenkins-focused. Never push production endpoints, credentials, customer code, proprietary manifests, or internal agent hostnames into a public study repository.

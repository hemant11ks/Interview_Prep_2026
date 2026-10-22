# Docker Interview Study Guide

**Level:** beginner to advanced DevOps interviews  
**Reference check:** 22 October 2026

Use this guide to explain concepts aloud, practice commands, and diagnose failures. Examples assume Linux containers and the current `docker compose` command. Docker Desktop, Windows containers, rootless mode, and different Engine versions can behave differently.

Commands are study examples, not commands tested against your infrastructure. Replace sample names and registry addresses. Lab commands create resources; cleanup commands remove only the named lab resources.

## Contents

1. [Core concepts](#1-core-concepts)
2. [Architecture and isolation](#2-architecture-and-isolation)
3. [Images, layers, and registries](#3-images-layers-and-registries)
4. [Container lifecycle and commands](#4-container-lifecycle-and-commands)
5. [Dockerfile interview essentials](#5-dockerfile-interview-essentials)
6. [Complete application example](#6-complete-application-example)
7. [Build caching and multi-stage builds](#7-build-caching-and-multi-stage-builds)
8. [Storage and persistence](#8-storage-and-persistence)
9. [Networking](#9-networking)
10. [Docker Compose](#10-docker-compose)
11. [Security and secrets](#11-security-and-secrets)
12. [Resources, signals, and health](#12-resources-signals-and-health)
13. [CI/CD and production practices](#13-cicd-and-production-practices)
14. [Troubleshooting workflow](#14-troubleshooting-workflow)
15. [Scenario-based interview questions](#15-scenario-based-interview-questions)
16. [Hands-on interview exercises](#16-hands-on-interview-exercises)
17. [Quick revision cheat sheet](#17-quick-revision-cheat-sheet)
18. [Official references](#18-official-references)

## 1. Core concepts

### What is Docker?

Docker is a platform for building, distributing, and running applications as containers. An image packages an application and its userspace dependencies. A container runs processes from that image with configured isolation, storage, and networking.

**Interview answer:** “Docker gives me a repeatable application package. I build an image, publish it to a registry, and run containers from that image with environment-specific configuration.”

### Image vs container

| Image | Container |
|---|---|
| Read-only packaged filesystem layers and configuration | Runtime instance created from an image |
| Can be tagged, pushed, and pulled | Can be started, stopped, inspected, and removed |
| Shared by multiple containers | Has its own writable layer and runtime settings |
| Rebuilt to change the application package | Replaced during an immutable deployment |

Stopping a container preserves its writable layer. Removing the container removes that layer. Data mounted from a volume or bind mount has a separate lifecycle.

### Container vs virtual machine

| Aspect | Container | Virtual machine |
|---|---|---|
| Kernel | Normally shares the container host's kernel | Has a guest kernel |
| Isolation | OS-level isolation | Hypervisor isolation |
| Startup | Usually fast | Often slower |
| Package | App plus userspace dependencies | Guest OS plus application |
| Compatibility | Requires compatible kernel/architecture or emulation | Runs a supported guest OS |

Containers are not miniature VMs and are not automatically a strong boundary for hostile workloads. Linux containers on Docker Desktop commonly run inside a Linux VM.

### Docker vs Kubernetes vs Compose

- **Docker Engine:** builds and runs containers on a host.
- **Compose:** declares and manages a multi-container application, commonly on one Engine.
- **Kubernetes:** orchestrates workloads across nodes with scheduling, reconciliation, and service abstractions.
- **Swarm:** Docker's integrated cluster orchestration mode.

Kubernetes does not require Docker Engine as its runtime. Docker-built OCI-compatible images can run with compatible Kubernetes runtimes.

## 2. Architecture and isolation

```text
Docker CLI / Compose
         |
         | Engine API
         v
Docker daemon (dockerd) <----> Image registry
         |
         v
Container lifecycle services (commonly containerd)
         |
         v
OCI runtime (commonly runc)
         |
         v
Container processes + host kernel
```

The CLI can target a remote daemon. Filesystem paths and published ports then refer to that daemon's host, not necessarily your laptop.

```bash
docker version                 # Client/server versions
docker info                    # Engine configuration and capabilities
docker context ls              # Available daemon contexts
docker context show            # Current context
```

**Namespaces** isolate views of resources such as processes, mounts, networking, hostnames, IPC, and users. **Cgroups** account for and constrain resources. **Capabilities**, **seccomp**, and security modules such as AppArmor/SELinux provide additional restrictions.

**Interview trap:** namespaces isolate; they do not by themselves enforce CPU or memory limits. Cgroups provide resource controls, not complete security isolation.

## 3. Images, layers, and registries

An image consists of filesystem layers plus configuration and metadata. Storage implementations provide a writable container layer over the image filesystem. Exact storage drivers/snapshotters depend on the installation.

```text
Container writable layer       <- runtime file changes
Application filesystem layer
Dependency filesystem layer
Base filesystem layers         <- image content, shared where possible
```

Deleting a file in a later image layer does not remove the original bytes from an earlier layer. This matters for both image size and accidentally embedded secrets.

### Tags and digests

- `team/api:1.0` is a human-readable tag and can be moved to different content.
- `team/api@sha256:...` references content by digest.
- `latest` is an ordinary tag, not a guarantee of freshness or release ordering.
- A multi-platform reference can identify an image index containing platform-specific manifests.

```bash
docker image ls
docker image inspect nginx:alpine
docker history nginx:alpine
docker pull nginx:alpine
docker tag interview-app:1.0 registry.example.com/team/interview-app:1.0
# Publishing example; requires an authorized registry and credentials:
docker push registry.example.com/team/interview-app:1.0
```

A registry stores image content. A repository groups related images. A tag names a version within a repository. Docker Hub, Amazon ECR, and private registries are examples of registry services.

**save vs export:** `docker save` archives images for `docker load`; `docker export` archives a container filesystem for `docker import`, losing normal image history/configuration semantics. Export does not provide a backup of mounted volume data.

**commit vs build:** `docker commit` captures certain container changes into an image, but a reviewed Dockerfile is more reproducible. Mounted volume contents are not included by commit.

## 4. Container lifecycle and commands

```text
image -> create -> created container -> start -> running -> stop -> exited
                                                      ^             |
                                                      +--- start ---+
                  remove -> deletes container metadata/writable layer
```

`docker run` combines creation and start. `docker start` restarts an existing container with its existing configuration. `docker exec` launches an additional process in a running container.

```bash
# LAB: download and run an example web server, published only on host loopback.
docker run -d --name interview-web -p 127.0.0.1:8080:80 nginx:alpine

# Inspect and test.
docker ps
docker ps -a
docker logs --tail 50 interview-web
docker inspect interview-web
docker top interview-web
docker stats --no-stream interview-web
docker port interview-web
curl http://127.0.0.1:8080/

# Interactive process; requires a shell in the image.
docker exec -it interview-web sh

# Stop/start the same container.
docker stop --time 20 interview-web
docker start interview-web

# Cleanup: removes this lab container.
docker stop interview-web
docker rm interview-web
```

Useful flags: `-d` detached, `-i` keep stdin open, `-t` allocate a terminal, `--rm` remove the container after exit, `--name` assign a name, `-e` set an environment variable.

`attach` connects to the main process's streams; `exec` starts a new process. Interrupting an attached foreground application can stop it.

## 5. Dockerfile interview essentials

| Instruction | Purpose |
|---|---|
| `FROM` | Select base image/start a build stage |
| `WORKDIR` | Set working directory |
| `COPY` | Copy files from context or another stage |
| `ADD` | Copy with additional source/extraction capabilities |
| `RUN` | Execute build-time commands |
| `ARG` | Declare build arguments |
| `ENV` | Set persisted image environment defaults |
| `USER` | Set user for subsequent operations/runtime |
| `EXPOSE` | Document intended ports; does not publish them |
| `ENTRYPOINT` | Configure executable behavior |
| `CMD` | Configure default command/arguments |
| `HEALTHCHECK` | Configure a health probe |

Prefer `COPY` for ordinary local files. Use `ADD` when its additional capabilities are intentional. Build arguments are scoped; do not assume an `ARG` declared before `FROM` is available inside every stage without redeclaration. Neither `ARG` nor `ENV` is a secret-storage mechanism. See the [Dockerfile reference](https://docs.docker.com/reference/dockerfile/).

### RUN vs CMD vs ENTRYPOINT

```dockerfile
RUN python -m compileall /app
ENTRYPOINT ["python", "/app/app.py"]
CMD ["--port", "8080"]
```

- `RUN` executes while building.
- The default runtime command above is `python /app/app.py --port 8080`.
- `docker run IMAGE --port 9000` replaces the CMD arguments.
- `docker run --entrypoint sh IMAGE` overrides the entrypoint.

JSON/exec form does not invoke a shell or expand `$VARIABLE` automatically. Shell form invokes a shell and can complicate signal forwarding. If a wrapper script starts the real application, use `exec "$@"` where appropriate so the app receives signals directly.

## 6. Complete application example

Create these files in a disposable directory. This standard-library Python server is a learning example, not a production web-server recommendation.

**app.py**

```python
from http.server import BaseHTTPRequestHandler, HTTPServer

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/health":
            status, body = 200, b"healthy\n"
        elif self.path == "/":
            status, body = 200, b"Docker interview demo\n"
        else:
            status, body = 404, b"not found\n"
        self.send_response(status)
        self.send_header("Content-Type", "text/plain")
        self.send_header("Content-Length", str(len(body)))
        self.end_headers()
        self.wfile.write(body)

HTTPServer(("0.0.0.0", 8080), Handler).serve_forever()
```

**Dockerfile**

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.13-slim
WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
RUN groupadd --gid 10001 app \
    && useradd --uid 10001 --gid app --no-create-home app
COPY --chown=10001:10001 app.py ./app.py
USER 10001:10001
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD ["python", "-c", "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8080/health', timeout=2)"]
CMD ["python", "app.py"]
```

**.dockerignore**

```text
.git
.env
.env.*
.venv
__pycache__
*.pyc
*.pem
*.key
```

**Build and run — LAB:**

```bash
docker build -t interview-app:1.0 .
docker run -d --name interview-app \
  -p 127.0.0.1:8080:8080 \
  --memory=256m --cpus=0.5 \
  --read-only --tmpfs /tmp:rw,noexec,nosuid,size=16m \
  --cap-drop=ALL --security-opt=no-new-privileges \
  interview-app:1.0
curl http://127.0.0.1:8080/health
docker inspect --format '{{.State.Health.Status}}' interview-app
```

Use a reviewed base-image digest for reproducible releases. The floating tag above keeps the tutorial readable. Stop any previous lab using host port 8080 first.

## 7. Build caching and multi-stage builds

Build cache reuse depends on the instruction and its inputs. An early change can invalidate downstream work. Copy dependency manifests before frequently changing application files, and keep unnecessary files out of the build context.

Example pattern for a Node application with a committed lockfile:

```dockerfile
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
```

A cache hit does not mean upstream packages were refreshed. `--no-cache` skips build-cache reuse; `--pull` checks for a newer base image. Rebuild on a controlled schedule. Combine package-manager metadata refresh and installation in one operation when using that package manager. See [build best practices](https://docs.docker.com/build/building/best-practices/).

### Multi-stage example: build a Go binary, ship the binary

This assumes the build context contains a dependency-free `main.go` that starts a server on port 8080.

```dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.26 AS build
WORKDIR /src
COPY main.go .
RUN CGO_ENABLED=0 go build -trimpath -o /out/server main.go

FROM scratch
COPY --from=build /out/server /server
USER 65532:65532
EXPOSE 8080
ENTRYPOINT ["/server"]
```

The final image omits the compiler and source tree. `scratch` is empty: no shell, package manager, CA bundle, or timezone database. Copy any required runtime assets deliberately. `CGO_ENABLED=0` is suitable only for applications that can work without CGO. See [multi-stage builds](https://docs.docker.com/build/building/multi-stage/).

### Multi-platform builds

```bash
# Publishing example: requires a suitable builder and authorized registry.
docker buildx build --platform linux/amd64,linux/arm64 \
  -t registry.example.com/team/app:1.0 --push .
```

Options include native builders, emulation, or cross-compilation. A multi-platform tag references multiple platform-specific images; it does not make one binary architecture-independent. Verify the builder's capabilities and test each platform. See [multi-platform builds](https://docs.docker.com/build/building/multi-platform/).

## 8. Storage and persistence

| Storage | Managed/location | Typical purpose |
|---|---|---|
| Writable layer | Container filesystem | Disposable runtime changes |
| Named volume | Docker-managed | Persistent application/database files |
| Bind mount | Chosen daemon-host path | Source code or host configuration |
| tmpfs | Memory-backed mount | Temporary data; not durable |

Volumes outlive ordinary container removal. A mount can hide existing files at its destination. A new empty volume may be populated from image files at the mount point; bind mounts behave differently. Volumes do not automatically replicate across hosts or provide backups. See [volume documentation](https://docs.docker.com/engine/storage/volumes/).

```bash
# LAB: prove named-volume persistence across disposable containers.
docker volume create interview-data
docker run --rm --mount source=interview-data,target=/data alpine:3.22 \
  sh -c 'echo persistent > /data/message.txt'
docker run --rm --mount source=interview-data,target=/data alpine:3.22 \
  cat /data/message.txt

# Bind-mount example; ./config must exist on the daemon host.
docker run --rm \
  --mount type=bind,src="$(pwd)/config",dst=/config,readonly \
  alpine:3.22 ls /config
```

For permission errors, compare numeric UID/GID, directory permissions, read-only flags, user-namespace mappings, and SELinux labels. Avoid treating `chmod 777` as the universal solution.

For databases, use application-consistent backup procedures or quiesce writes appropriately. Copying live database files is not automatically a valid backup. Verify restoration.

**Destructive cleanup:** `docker volume rm interview-data` permanently deletes this lab volume's data when it is no longer in use.

## 9. Networking

```text
Host 127.0.0.1:8080
         |
   published-port forwarding
         |
Container interface :80 -> application listener
```

| Mode/driver | Main idea |
|---|---|
| User-defined bridge | Single-host isolated network with container-name DNS |
| Host | Shares host networking where supported |
| None | No external network connectivity |
| Overlay | Multi-host network, commonly with Swarm |
| macvlan/ipvlan | Integration with underlying L2/L3 networking |

`localhost` inside a normal container refers to that container's network namespace. To contact a peer on a user-defined network, use its name and container port. Docker Desktop adds networking layers; host-network support and behavior are platform-dependent.

```bash
# LAB
docker network create interview-net
docker run -d --name interview-backend --network interview-net nginx:alpine
docker run --rm --network interview-net alpine:3.22 \
  wget -qO- http://interview-backend:80/
docker network inspect interview-net
```

**EXPOSE vs publish:** `EXPOSE 80` documents a port; `-p 127.0.0.1:8080:80` publishes it on host loopback. `-p 8080:80` normally binds more broadly. Shared-network containers do not need host publication to communicate with an accessible listener.

When connections fail, check network membership, DNS, target port, listener bind address, host forwarding/firewall rules, and the return path. Docker manages firewall rules, so host INPUT/ufw assumptions may not cover forwarded container traffic. See [bridge networking](https://docs.docker.com/engine/network/drivers/bridge/) and [firewall behavior](https://docs.docker.com/engine/network/packet-filtering-firewalls/).

## 10. Docker Compose

Compose describes services, networks, volumes, and related settings in YAML. Modern examples use `compose.yaml` and `docker compose`; a top-level `version` field is unnecessary.

This lab uses the application from section 6 and adds Redis plus a one-shot probe. The Python application is independent of Redis; the probe demonstrates dependency readiness.

```yaml
services:
  web:
    build: .
    ports:
      - "127.0.0.1:8080:8080"
    restart: unless-stopped

  cache:
    image: redis:7.4-alpine
    command: ["redis-server", "--appendonly", "yes"]
    volumes:
      - cache-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

  probe:
    image: redis:7.4-alpine
    depends_on:
      cache:
        condition: service_healthy
    command: ["redis-cli", "-h", "cache", "ping"]
    restart: "no"

volumes:
  cache-data:
```

```bash
docker compose config          # Validate/render; output may contain secrets
docker compose up -d --build
docker compose ps -a
docker compose logs --tail 50
docker compose exec cache redis-cli ping
docker compose logs probe      # Expected application response: PONG
docker compose down            # Remove project containers/network; retain volume
```

Short-form `depends_on` gives ordering, not application readiness. `service_healthy` waits for the dependency's health check during startup. Applications still need runtime retries and reconnect behavior after a dependency fails later. See [Compose startup order](https://docs.docker.com/compose/how-tos/startup-order/).

`docker compose down -v` also removes the project's declared named volumes and attached anonymous volumes; it can destroy persistent data. External volumes are not removed by Compose down. Keep that distinction clear in interviews.

## 11. Security and secrets

Explain security in layers:

1. Use trusted, maintained base images and patch through rebuilds.
2. Scan dependencies/images and review exploitable findings in context.
3. Run as a non-root user where practical.
4. Drop unnecessary capabilities and avoid `--privileged`.
5. Use a read-only root filesystem with explicit writable locations.
6. Restrict network exposure and protect the daemon API/socket.
7. Apply resource limits and appropriate seccomp/AppArmor/SELinux policy.
8. Keep secrets out of images, logs, command lines, and source control.

Access to a rootful Docker daemon or its socket generally gives powerful host control. Mounting `/var/run/docker.sock` into a container is not a harmless convenience.

**Non-root vs rootless:** `USER` changes the container process identity. Rootless mode runs both the daemon and containers without host root privileges using user namespaces. It has prerequisites and operational differences. See [rootless mode](https://docs.docker.com/engine/security/rootless/).

### Build-time secret example

```dockerfile
# syntax=docker/dockerfile:1
RUN --mount=type=secret,id=repo_token \
    some-package-client --token-file /run/secrets/repo_token install
```

```bash
docker build --secret id=repo_token,src=./repo-token.txt -t app:1.0 .
```

`some-package-client` is a placeholder for your actual tool. Exclude `repo-token.txt` from the build context. Secret mounts make the secret available for that build step without intentionally storing it as an image layer; the command can still leak it if it copies or prints it. See [build secrets](https://docs.docker.com/build/building/secrets/).

For runtime secrets, use an appropriate secret manager or mounted secret files. Compose secrets are not automatically equivalent to Swarm's secret storage/security model. Environment variables can be exposed through inspection or diagnostic output.

## 12. Resources, signals, and health

### Resource limits

```bash
docker run -d --name limited-app \
  --memory=256m --cpus=0.5 --pids-limit=100 interview-app:1.0
docker stats --no-stream limited-app
docker inspect --format '{{.State.OOMKilled}} {{.State.ExitCode}}' limited-app
```

Containers do not get explicit CPU/memory caps by default merely because they are containers; host/parent constraints can still apply. CPU quota limits available CPU time; CPU shares express relative weighting under contention. Memory pressure can cause OOM kills. See [resource constraints](https://docs.docker.com/engine/containers/resource_constraints/).

Exit code **137** commonly means SIGKILL, not necessarily OOM. Check `OOMKilled`, events, and host evidence. Exit code **143** commonly means SIGTERM. Interpret application exit codes in context.

### PID 1 and graceful shutdown

A container exits when its main process exits. Run the main service in the foreground. A startup command that backgrounds the service and exits does not keep the container alive.

`docker stop` sends the configured stop signal, normally SIGTERM, then forces termination if the grace period expires. Ensure the app handles shutdown and that wrappers forward signals. `--init` can supply a minimal init process for signal forwarding and child reaping; it does not repair all application lifecycle bugs.

### Health and restart policy

A health check reports whether a running container passes its probe. Docker Engine does not automatically restart a container merely because it becomes unhealthy.

| Policy | Purpose |
|---|---|
| `no` | No automatic restart |
| `on-failure` | Restart on nonzero exit, optionally with a retry limit |
| `always` | Restart after exit; manual stop suppresses it until explicit start or daemon restart |
| `unless-stopped` | Preserve a stopped state across daemon restart |

Restart policies respond to lifecycle events, not a full dependency recovery model. See [restart policy details](https://docs.docker.com/engine/containers/start-containers-automatically/).

## 13. CI/CD and production practices

```text
Commit -> tests -> image build -> scan/SBOM -> registry
                                               |
                                         immutable digest
                                               |
                                   staging -> checks -> production
```

Build once and promote the same tested image digest. Inject environment-specific configuration at deployment. Keep source revision and release metadata associated with the artifact.

A useful pipeline should:

- Validate the Dockerfile and run unit/integration tests.
- Use BuildKit caching without caching secret values into layers.
- Build the required architectures and test them.
- Generate an SBOM and provenance where tooling supports it.
- Scan and verify/sign artifacts according to deployment policy.
- Push with scoped, temporary credentials.
- Deploy by digest, check readiness, and retain a tested rollback image.

Rolling back an image does not reverse database migrations or external side effects. Design schema compatibility and recovery separately.

Send useful application logs to stdout/stderr and configure collection/rotation. `docker logs` visibility depends on the logging driver and configuration; it does not automatically read arbitrary application log files.

For production stateful services, plan backups, restores, storage durability, restart behavior, and failure domains. Compose alone does not provide a multi-node scheduler or automatic rescheduling onto another host.

## 14. Troubleshooting workflow

Start with the exact symptom, timestamp, image digest, container state, and recent change.

```bash
docker context show
docker ps -a
docker inspect CONTAINER
docker logs --tail 100 CONTAINER
docker stats --no-stream CONTAINER
docker events --since 10m --until 0s
```

Inspect output can include environment secrets; redact before sharing.

| Symptom | Investigate |
|---|---|
| Immediate exit | Main command, exit code, missing config, startup logs |
| Restart loop | Application failure, permissions, missing dependency, OOM |
| Port unreachable | Publication, container port, listener bind, network/firewall |
| Name does not resolve | Shared network, DNS settings, service/container name |
| Permission denied | UID/GID, mount permissions, read-only flags, security labels |
| File disappeared | Container replaced without a volume, wrong mount, hidden path |
| Build is slow | Context size, cache invalidation, dependency downloads, emulation |
| No space left | Host bytes/inodes, image/cache/log/volume usage |
| Exec format error | Architecture mismatch, invalid executable, script format |
| Unhealthy but running | Probe command/dependencies/timeout, real application state |

Useful focused commands:

```bash
docker inspect --format '{{json .State}}' CONTAINER
docker inspect --format '{{json .Mounts}}' CONTAINER
docker inspect --format '{{json .NetworkSettings.Networks}}' CONTAINER
docker system df
docker system df -v
docker buildx du
```

Minimal images may lack `sh`, `curl`, or `ss`. Use approved diagnostic tooling or a separate debug container; do not assume installing utilities into production containers is the right fix.

Avoid starting with `docker system prune -a --volumes`: broad cleanup can remove useful images, caches, and eligible data volumes. Identify the actual disk consumer and remove only intended resources.

## 15. Scenario-based interview questions

### 1. Why does a container exit immediately?

Its main process ended. Inspect the exit code and logs; check the command, configuration, and foreground behavior. Do not hide the failure with an infinite sleep.

### 2. The image runs locally but fails in CI. What do you compare?

Image digest, architecture, environment variables, mounted files, user permissions, resource limits, DNS/proxy settings, and available secrets. Confirm CI is running the artifact you tested.

### 3. Port 8080 is published, but the application is unreachable. Why?

The application may listen on another port or only on container loopback. Check `docker port`, application logs, listener address, daemon host, and firewall path.

### 4. Can one container access another using localhost?

Normally no. Use the peer's name on a shared network. Containers explicitly sharing a network namespace are an exception.

### 5. Why does rebuilding an image not update the running container?

A container was created from the previous image. Recreate it using the new image; restarting the old container keeps its existing image/configuration.

### 6. Why did database data disappear after deployment?

It may have lived in the deleted writable layer, a different volume may have been attached, or cleanup removed the volume. Inspect mount identities and recovery backups.

### 7. How do you make an image smaller?

Use an appropriate runtime base, multi-stage builds, a small context, and only required runtime dependencies. Remove temporary files in the same step that creates them where appropriate; verify functionality and security.

### 8. Does Alpine always make the best image?

No. Its musl-based environment can cause compatibility or build differences. Evaluate dependencies, support, debugging needs, and actual size rather than choosing solely by base-image size.

### 9. Why is a build unexpectedly using stale dependencies?

A cache hit can reuse an earlier installation layer. Review dependency lockfiles, build inputs, package indexes, base-image freshness, and deliberate cache invalidation.

### 10. A secret was copied into an image and deleted later. Is it safe?

No. Earlier layers may retain it. Rotate/revoke the secret, rebuild cleanly, investigate distribution, and use secret mounts or runtime secret delivery.

### 11. A container exits with 137. Is it an OOM kill?

Possibly, but SIGKILL has other causes. Check `OOMKilled`, host memory/kernel evidence, manual actions, and stop-timeout behavior.

### 12. The container is unhealthy. Why did Docker not restart it?

Health status alone does not trigger Engine restart policies. Determine whether the process should recover, exit, or be replaced by an orchestrator's health logic.

### 13. Why does Compose start the app before the database is ready?

Short-form dependency ordering does not guarantee readiness. Add a meaningful health check and `service_healthy`, then keep runtime reconnect/retry logic.

### 14. Why does a bind mount hide files copied during build?

The mount overlays that path at runtime. Inspect the destination and the host source; mount a narrower path or change the layout.

### 15. Why does `docker exec -it app bash` fail?

The container may be stopped, or the image may not contain Bash. Try an available shell only if one exists, or use external debugging tools.

### 16. Why is a container slow despite low average CPU usage?

Check CPU throttling, memory pressure, disk/network latency, dependency waits, and peak utilization. Averages can hide short bursts or quota throttling.

### 17. Why do Docker commands target the wrong machine?

Check the active context and environment overrides such as `DOCKER_HOST`. The CLI and daemon need not run on the same machine.

### 18. Is running as non-root enough to secure a container?

No. Also constrain capabilities, mounts, daemon access, network exposure, syscalls, resource usage, and the supply chain. Isolation depends on the host too.

### 19. How would you handle graceful deployments?

Stop new traffic, let in-flight work finish, send the expected shutdown signal, allow sufficient grace time, and verify persistent writes before termination.

### 20. How do you explain Docker's value in a DevOps interview?

“It provides a versioned application package that we can test and promote consistently. I pair it with controlled configuration, secure builds, persistent storage where required, and observable deployment procedures.”

## 16. Hands-on interview exercises

### Exercise A: Build and explain

Use section 6. Explain every Dockerfile instruction, prove `/health` responds, inspect the configured user, and state why `EXPOSE` does not publish a port.

### Exercise B: Persistence

Use section 8. Write from one disposable container and read from another. Explain why the volume survives both containers and when its data would be removed.

### Exercise C: Networking

Use section 9. Demonstrate name-based access between containers without publishing the backend port. Explain how access from the host differs.

### Exercise D: Diagnose a failure

Run the section 6 image with an incorrect CMD override:

```bash
docker run --name interview-broken interview-app:1.0 python missing.py
docker logs interview-broken
docker inspect --format '{{.State.ExitCode}}' interview-broken
```

Explain why this is an application startup failure rather than a Docker network problem.

### Lab cleanup

Run only for the named resources you created:

```bash
docker rm -f interview-app interview-broken limited-app interview-backend
docker network rm interview-net
# From the directory containing the lab compose.yaml:
docker compose down
```

Missing-resource errors simply indicate that a particular lab was not run or was already removed. Retain volumes unless you intentionally want to delete their data.

## 17. Quick revision cheat sheet

| Question | One-line answer |
|---|---|
| Image vs container? | Package vs runtime instance |
| Container vs VM? | Shared host kernel vs guest kernel |
| Namespaces vs cgroups? | Isolation of views vs resource accounting/limits |
| RUN vs CMD? | Build-time execution vs runtime default |
| CMD vs ENTRYPOINT? | Default command/arguments vs configured executable behavior |
| COPY vs ADD? | Ordinary copying vs additional source/extraction capabilities |
| ARG vs ENV? | Build argument vs persisted environment default; neither is a secret store |
| EXPOSE vs `-p`? | Metadata vs host-port publication |
| Volume vs bind mount? | Docker-managed storage vs chosen host path |
| stop vs rm? | Stop execution vs remove container and writable layer |
| run vs start vs exec? | New container vs existing container vs extra running process |
| Tag vs digest? | Mutable name vs content-addressed reference |
| Healthcheck vs restart? | Probe status vs exit/lifecycle policy |
| Compose vs Kubernetes? | Multi-container application management vs cluster orchestration |
| Non-root vs rootless? | Process identity vs daemon and container privilege model |

```bash
docker ps -a                         # State
docker logs --tail 100 NAME           # Recent logs
docker inspect NAME                  # Configuration/runtime details
docker stats --no-stream NAME        # Resource snapshot
docker exec -it NAME sh              # Shell, if present
docker port NAME                     # Published ports
docker network inspect NETWORK      # Network members/settings
docker volume inspect VOLUME        # Volume details
docker build -t app:1.0 .            # Build
docker compose config               # Validate/render Compose
docker compose up -d --build         # Create/update application
docker compose down                 # Remove project containers/network
```

**Remember:** container `localhost` is local to its namespace; mutable tags are not immutable releases; volumes are not backups; port publication is not application readiness; health status does not automatically restart a container; and an exit code is evidence, not a complete diagnosis.

## 18. Official references

- [Docker Engine](https://docs.docker.com/engine/)
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Build best practices](https://docs.docker.com/build/building/best-practices/)
- [Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)
- [Build secrets](https://docs.docker.com/build/building/secrets/)
- [Multi-platform builds](https://docs.docker.com/build/building/multi-platform/)
- [Volumes](https://docs.docker.com/engine/storage/volumes/)
- [Bridge networking](https://docs.docker.com/engine/network/drivers/bridge/)
- [Packet filtering and firewalls](https://docs.docker.com/engine/network/packet-filtering-firewalls/)
- [Compose service reference](https://docs.docker.com/reference/compose-file/services/)
- [Compose startup order](https://docs.docker.com/compose/how-tos/startup-order/)
- [Resource constraints](https://docs.docker.com/engine/containers/resource_constraints/)
- [Restart policies](https://docs.docker.com/engine/containers/start-containers-automatically/)
- [Rootless mode](https://docs.docker.com/engine/security/rootless/)

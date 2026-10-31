 # ☸️ Kubernetes Interview Preparation — DevOps | SRE | Platform Engineering

> **A practical, interview-ready Kubernetes handbook:** architecture, core objects, networking, storage, scheduling, security, troubleshooting, YAML manifests, `kubectl` commands, real-world scenarios, and concise answers.
>
> **Audience:** DevOps Engineers, Site Reliability Engineers (SREs), Platform Engineers, Cloud Engineers, and Kubernetes administrators (roughly 1–5 years of experience).
>
> **Approach:** For each concept, know **what it is → why it exists → how it works → how to configure it → how to debug it**.

**Official reference:** [Kubernetes documentation](https://kubernetes.io/docs/home/) · [kubectl reference](https://kubernetes.io/docs/reference/kubectl/) · [Kubernetes API reference](https://kubernetes.io/docs/reference/kubernetes-api/)

---

## 📑 Table of Contents

1. [Quick Interview Revision](#1-quick-interview-revision)
2. [Kubernetes Basics](#2-kubernetes-basics)
3. [Architecture and Request Flow](#3-architecture-and-request-flow)
4. [Pods and Workload Controllers](#4-pods-and-workload-controllers)
5. [YAML and Declarative Management](#5-yaml-and-declarative-management)
6. [Services, Networking, DNS, Ingress and Gateway API](#6-services-networking-dns-ingress-and-gateway-api)
7. [Scheduling, Taints, Affinity and Availability](#7-scheduling-taints-affinity-and-availability)
8. [CPU, Memory, HPA and Autoscaling](#8-cpu-memory-hpa-and-autoscaling)
9. [Health Probes and Pod Lifecycle](#9-health-probes-and-pod-lifecycle)
10. [Volumes, PV, PVC, StorageClass and CSI](#10-volumes-pv-pvc-storageclass-and-csi)
11. [Security: RBAC, Secrets, NetworkPolicy, Pod Security](#11-security-rbac-secrets-networkpolicy-pod-security)
12. [Helm, Kustomize, GitOps and CI/CD](#12-helm-kustomize-gitops-and-cicd)
13. [Observability, Reliability and SRE Practices](#13-observability-reliability-and-sre-practices)
14. [Hands-on Deployment Lab](#14-hands-on-deployment-lab)
15. [Troubleshooting Playbook](#15-troubleshooting-playbook)
16. [Top Interview Questions and Answers](#16-top-interview-questions-and-answers)
17. [Real-World Scenario-Based Questions](#17-real-world-scenario-based-questions)
18. [kubectl Cheat Sheet](#18-kubectl-cheat-sheet)
19. [Practice Tasks and 7-Day Revision Plan](#19-practice-tasks-and-7-day-revision-plan)
20. [Official Learning Resources](#20-official-learning-resources)

---

## 1. Quick Interview Revision

### Kubernetes in 30 seconds

**Kubernetes (K8s)** is an open-source container orchestration platform. You declare the **desired state** (for example, *three application replicas*), and controllers continuously reconcile actual state toward it. It provides workload scheduling, service discovery, rollout management, self-healing, configuration, policy integration, and extensible APIs.

Kubernetes **does not**, by itself, build application images, implement a complete CI/CD system, or guarantee that an unhealthy application is fixed. It coordinates the resources and automations configured around it.

### Memorize these first

| Concept | Interview-ready description |
|---|---|
| Pod | Smallest deployable Kubernetes unit; one or more containers share network and volumes |
| Deployment | Manages ReplicaSets and rolling updates for usually stateless apps |
| ReplicaSet | Maintains the requested number of matching Pods |
| StatefulSet | Stable Pod identity and ordered behavior for stateful workloads |
| DaemonSet | Runs a matching Pod on eligible nodes, often for node agents |
| Job / CronJob | Run-to-completion workload / scheduled Job creation |
| Service | Stable network abstraction over a changing set of Pods |
| Ingress | HTTP(S) routing rules; requires an Ingress controller |
| Gateway API | More expressive, role-oriented traffic-routing API; requires an implementation |
| ConfigMap / Secret | Non-confidential configuration / confidential data API object |
| PV / PVC / StorageClass | Storage resource / storage request / dynamic provisioning policy |
| HPA | Adjusts workload replica count according to metrics |
| PDB | Limits certain voluntary Pod disruptions |
| RBAC | Authorizes API operations for users, groups, and ServiceAccounts |
| NetworkPolicy | L3/L4 Pod traffic policy, enforced only by compatible network implementations |
| CRD / Operator | Extend the Kubernetes API / manage app lifecycles through controllers |

### A model production request path

```text
User / client
    |
    v
DNS -> external load balancer
    |
    v
Gateway/Ingress implementation
    |
    v
Service (stable name; selects backends)
    |
    v
Pod IP : container port
    |
    v
Application process

Observability: metrics + logs + traces + events
Security: RBAC + admission + network policy + workload identity
```

Actual routing depends on the cluster's cloud provider, CNI, proxy/eBPF dataplane, and traffic controller.

---

## 2. Kubernetes Basics

### Kubernetes vs Docker

| Docker/container tooling | Kubernetes |
|---|---|
| Build and run containers locally or on hosts | Coordinate containerized workloads across a cluster |
| Container image packaging and lifecycle | Declarative deployment, placement, discovery, scaling, recovery |
| Docker Engine is one possible local container environment | Workers typically use a CRI-compatible runtime such as containerd or CRI-O |

**Important:** Modern Kubernetes does not use Docker Engine directly as a built-in CRI runtime. Docker-built OCI-compatible images still work in Kubernetes.

### Cluster terminology

- **Cluster:** Kubernetes control plane plus worker nodes.
- **Node:** VM or physical machine capable of running Pods.
- **Namespace:** Logical scope for namespaced Kubernetes objects; not a security boundary by itself.
- **Label:** Key/value metadata for grouping and selecting objects (`app=api`, `tier=backend`).
- **Selector:** Query matching labels.
- **Annotation:** Non-identifying metadata such as build information and tooling hints.
- **Desired state:** The configuration specified in the Kubernetes API.
- **Reconciliation:** Repeated controller action to reduce differences between actual and desired state.
- **Operator:** A controller, commonly paired with CRDs, encoding application-specific operational knowledge.

### Container networking basics

Normally each Pod has its own IP, and all containers within that Pod share its network namespace, including `localhost` and ports. Containers in the same Pod can communicate over `127.0.0.1`; different Pods use their network addresses or Services. The CNI implementation supplies the actual Pod networking.

### Namespaced vs cluster-scoped objects

| Namespaced | Cluster-scoped |
|---|---|
| Pod, Deployment, Service, ConfigMap, Secret | Node, Namespace, PersistentVolume |
| Role, RoleBinding, PVC, NetworkPolicy | ClusterRole, ClusterRoleBinding, StorageClass, CRD |

```bash
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
kubectl get namespaces
kubectl get pods -A
```

---

## 3. Architecture and Request Flow

```text
                           CONTROL PLANE
+----------------------------------------------------------------+
| kube-apiserver <----> etcd (persistent cluster-state store)     |
|      ^                                                         |
|      |                                                         |
| kube-scheduler        kube-controller-manager                  |
| cloud-controller-manager (where integrated)                     |
+----------------------------+-----------------------------------+
                             |
                     Kubernetes API
                             |
          +------------------+------------------+
          |                                     |
      WORKER NODE 1                         WORKER NODE 2
 +----------------------+             +----------------------+
 | kubelet              |             | kubelet              |
 | CRI runtime          |             | CRI runtime          |
 | Service dataplane*   |             | Service dataplane*   |
 | CNI networking       |             | CNI networking       |
 | Pods                 |             | Pods                 |
 +----------------------+             +----------------------+

* Often kube-proxy; some CNIs use an alternative, e.g., eBPF.
```

### Component-by-component

| Component | Responsibility | Common issue |
|---|---|---|
| `kube-apiserver` | Kubernetes API entry point; authenticates, authorizes, validates, and persists API objects | API unavailable / authorization errors |
| `etcd` | Consistent key-value store for cluster state | Quorum or disk/latency issues |
| `kube-scheduler` | Selects a node for unscheduled Pods | Pods remain `Pending` |
| `kube-controller-manager` | Runs controllers reconciling workload and cluster state | Desired replicas not created |
| `cloud-controller-manager` | Cloud-specific node/route/load-balancer integration | External LB fails to provision |
| `kubelet` | Node agent; ensures assigned Pods' containers run | Pods not starting / node `NotReady` |
| Container runtime | Pulls images and starts/stops containers through CRI | Image/runtime failures |
| CNI plugin | Pod networking and, depending on plugin, NetworkPolicy | Cross-Pod connection failures |
| `kube-proxy` or alternative | Implements Service forwarding | ClusterIP/Service traffic issues |
| CoreDNS | Cluster DNS resolution | Service-name lookup failures |

### What happens when you run `kubectl apply -f deployment.yaml`?

1. `kubectl` sends the manifest to the API server, using credentials and context from kubeconfig.
2. The API server performs authentication, authorization, admission checks, and schema validation; the desired object is persisted.
3. The Deployment controller reconciles a ReplicaSet; the ReplicaSet controller creates Pods.
4. The scheduler assigns eligible unscheduled Pods to nodes.
5. Each selected node's kubelet asks the container runtime to pull/start the containers, coordinating volume and network setup.
6. Kubelet reports status; readiness influences whether Pod IPs appear as ready Service endpoints.
7. Controllers keep reconciling changes and failures afterward.

**Interview nuance:** Controllers and the scheduler normally communicate through the **API server**, not by directly writing to etcd. The API server is the primary Kubernetes state interface.

### How does Kubernetes self-heal?

- A failed container can be restarted by kubelet according to restart policy.
- A failed Pod in a Deployment can be replaced by the ReplicaSet controller.
- Workloads on failed nodes can eventually be rescheduled, subject to timing, policy, and storage constraints.
- Readiness removes unready Pods from ordinary Service traffic routing.

**Limitation:** Kubernetes cannot fix a broken database, a faulty release, a bad liveness probe, or insufficient cluster capacity automatically.

---

## 4. Pods and Workload Controllers

### Pod lifecycle

The **Pod phase** is one of `Pending`, `Running`, `Succeeded`, `Failed`, or `Unknown`. `CrashLoopBackOff` and `ImagePullBackOff` are **container waiting reasons shown in status output**, not Pod phases.

A Pod is ephemeral: replacement Pods usually receive a different name/IP unless managed by a controller with stable identity semantics, such as a StatefulSet.

### Deployment, ReplicaSet, StatefulSet, DaemonSet, Job

| Workload | Use it when | Example |
|---|---|---|
| Deployment | Stateless replicated application and rolling updates | Web/API service |
| StatefulSet | Stable identity and per-Pod storage matter | Database cluster, queue brokers |
| DaemonSet | An agent is needed on each eligible node | Log collector, node exporter |
| Job | Task must finish successfully | One-off data migration |
| CronJob | A Job must be created on a schedule | Nightly reporting or backups |

**Deployment vs StatefulSet:** Deployments manage interchangeable Pods; StatefulSets provide stable Pod names/ordinals and can use per-Pod PVC templates. StatefulSet **does not automatically provide** database replication, backups, or failover.

### Useful commands

```bash
kubectl get pods -A -o wide
kubectl describe pod <pod> -n <namespace>
kubectl get deployments,replicasets -n <namespace>
kubectl get statefulsets,daemonsets,jobs,cronjobs -A
kubectl scale deployment/web --replicas=5 -n demo
kubectl rollout status deployment/web -n demo
kubectl rollout history deployment/web -n demo
kubectl rollout undo deployment/web -n demo
```

### Init containers vs sidecars

- **Init container:** Performs setup before regular application containers start; traditional init containers run to completion.
- **Sidecar:** Helper container, e.g., proxy, log shipper, or config reloader. Native sidecars can be modeled as restartable init containers on supported clusters; regular helper containers are also a common pattern.
- **Multi-container Pod:** Containers share a failure and scheduling unit; use only when their lifecycles and resources should be tightly coupled.

### ReplicaSet vs ReplicationController

ReplicaSet is the modern controller used by Deployments, with set-based label selector support. ReplicationController is an older API that you generally should not choose for new workloads.

---

## 5. YAML and Declarative Management

### Core manifest structure

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: demo
  labels:
    app: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:stable-alpine
          ports:
            - containerPort: 80
```

- `apiVersion`: API group/version (for example, `v1`, `apps/v1`).
- `kind`: Object type.
- `metadata`: Name, namespace, labels, annotations.
- `spec`: Desired configuration.
- `status`: Observed state, generally maintained by controllers.
- **Critical:** `.spec.selector.matchLabels` must match Pod template labels in a Deployment.

### Imperative vs declarative

```bash
# Imperative: execute an action
kubectl create namespace demo
kubectl scale deployment/web --replicas=4 -n demo

# Declarative: reconcile toward files / field ownership
kubectl apply -f app.yaml
kubectl diff -f app.yaml
kubectl apply --server-side -f app.yaml

# Generate a draft manifest; no cluster resource is created
kubectl create deployment web --image=nginx:stable-alpine \
  --dry-run=client -o yaml

# Inspect API fields / validation
kubectl explain deployment.spec.strategy
kubectl apply --dry-run=server -f app.yaml
```

**`create` vs `apply`:** `create` creates a new object and normally fails if it already exists; `apply` declaratively manages changes. **Server-side apply** records field ownership and can identify conflicts between managers.

### `kubectl` contexts

```bash
kubectl config get-contexts
kubectl config current-context
kubectl config use-context <context-name>
kubectl config view --minify
```

**Production safeguard:** Always verify context and namespace before write/delete commands. Treat kubeconfig and client credentials as sensitive.

---

## 6. Services, Networking, DNS, Ingress and Gateway API

### Service types

| Type | Behavior | Typical use |
|---|---|---|
| `ClusterIP` | Stable internal virtual IP (default) | Pod-to-Pod application traffic |
| `NodePort` | Opens a Service port on node addresses | Testing / simple external access |
| `LoadBalancer` | Requests an external load balancer via supported integration | Cloud-facing Service |
| `ExternalName` | DNS CNAME mapping to external name, not a proxy | DNS alias |
| Headless (`clusterIP: None`) | DNS-based discovery of individual backends; no virtual Service IP | Stateful systems |

`NodePort` typically uses a configured port range (commonly `30000–32767`; configurable). `LoadBalancer` requires a controller/provider integration (or something like MetalLB in suitable bare-metal setups); merely setting the type does not guarantee a public IP.

### Service example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
  namespace: demo
spec:
  selector:
    app: web
  ports:
    - name: http
      port: 80       # port clients use on the Service
      targetPort: 80 # port on the selected Pods
  type: ClusterIP
```

- `port` = port exposed by Service.
- `targetPort` = destination Pod port (number or named container port).
- `nodePort` = externally reachable node port, applicable to NodePort-capable Services.
- `selector` = matches Pod labels; ready backends are represented using EndpointSlices.

### DNS and debugging

Assuming the default cluster domain `cluster.local`, a Service resolves as:

```text
web.demo.svc.cluster.local
service.namespace.svc.cluster.local
```

```bash
kubectl get svc,endpointslices -n demo
kubectl describe svc web -n demo
kubectl run dns-debug --rm -it --restart=Never \
  --image=busybox:1.36 -n demo -- nslookup web.demo.svc.cluster.local
kubectl port-forward -n demo service/web 8080:80
# In another terminal: curl http://localhost:8080/
```

> **DNS issue sequence:** Check Pod DNS config → CoreDNS Pods/Service → Service existence/namespace → EndpointSlices → CNI/network policy → DNS forwarding/upstream. `kubectl get endpoints` may still work, but EndpointSlices are the scalable backend resource.

### Ingress vs Gateway API

| Ingress | Gateway API |
|---|---|
| HTTP(S) routing API; ingress controller required | Routing resources such as `GatewayClass`, `Gateway`, `HTTPRoute`; controller implementation required |
| Familiar path/host rules | Richer separation of infrastructure and app-team responsibilities |
| API is **frozen** but remains supported | Kubernetes docs recommend Gateway for new capabilities |

An Ingress object alone does **not** route traffic without a controller. Gateway API objects alone also require an appropriate controller implementation. TLS usually terminates at a configured edge proxy or gateway, depending on architecture.

### Networking layers to distinguish

- **CNI:** Makes Pod network connectivity possible (e.g., Cilium, Calico, Flannel). Not all CNIs enforce NetworkPolicy.
- **CoreDNS:** Resolves Service/discovery names.
- **Service dataplane:** Routes Service VIP/port traffic to backends (kube-proxy or an alternative).
- **Ingress/Gateway controller:** Handles application-layer edge routing.
- **NetworkPolicy:** Controls supported L3/L4 flows; not an HTTP authorization mechanism by itself.

---

## 7. Scheduling, Taints, Affinity and Availability

### Scheduler decision model

Scheduling typically involves **filtering** feasible nodes and **scoring** suitable nodes, considering resource requests, selectors/affinity, topology, taints, volumes, constraints, and other plugins.

### Node selector / node affinity / Pod affinity

- `nodeSelector`: Simple required key/value constraint.
- **Node affinity:** Richer required or preferred node-label matching.
- **Pod affinity:** Co-locate with Pods matching a label selector.
- **Pod anti-affinity:** Keep matching workloads apart for resilience.
- **Topology spread constraints:** Spread Pods across zones/nodes according to topology keys.

```yaml
# Pod template spec excerpt
spec:
  nodeSelector:
    workload: web
  tolerations:
    - key: dedicated
      operator: Equal
      value: web
      effect: NoSchedule
```

**A toleration is permission, not placement.** Use matching node affinity/selector as well if the workload must actually run on the dedicated pool.

### Taints vs tolerations

| Taint effect | Behavior |
|---|---|
| `NoSchedule` | Scheduler does not place non-tolerating Pods here |
| `PreferNoSchedule` | Soft scheduling preference against placement |
| `NoExecute` | Can evict existing non-tolerating Pods and blocks new ones |

```bash
kubectl label node <node> workload=web
kubectl taint node <node> dedicated=web:NoSchedule
kubectl describe node <node>
# Remove this specific taint (change only a node you manage)
kubectl taint node <node> dedicated=web:NoSchedule-
```

### Drain, cordon, uncordon

```bash
kubectl cordon <node>                    # prevent new normal scheduling
kubectl drain <node> --ignore-daemonsets  # evict eligible Pods; respect PDBs
kubectl uncordon <node>                  # allow scheduling again
```

Draining may block on PDBs, unmanaged Pods, local storage, or other constraints. Do **not** casually add `--force` / `--disable-eviction` in production; those options may bypass safeguards or force deletion.

### PodDisruptionBudget (PDB)

PDB limits **certain voluntary disruptions** (for example, evictions used by drain), **not** hardware failures or every deletion path.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
  namespace: demo
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web
```

With 3 healthy Pods, `minAvailable: 2` allows at most 1 eviction at a time via the Eviction API. If fewer than 2 Pods are healthy, maintenance may be blocked until availability recovers.

---

## 8. CPU, Memory, HPA and Autoscaling

### Requests vs limits

| Field | Meaning | Operational impact |
|---|---|---|
| CPU **request** | Reserved scheduling capacity | Scheduler placement; HPA CPU utilization denominator |
| CPU **limit** | Enforced upper CPU allotment | CPU throttling under contention/limit |
| Memory **request** | Requested scheduling capacity | Placement and node-pressure behavior |
| Memory **limit** | Upper memory bound | Container can be OOM-killed when exceeded |

```yaml
resources:
  requests:
    cpu: "100m"     # 0.1 CPU core
    memory: "128Mi"
  limits:
    cpu: "500m"     # 0.5 CPU core
    memory: "256Mi"
```

**Important:** CPU limiting is a latency trade-off; it can throttle even when the host has spare CPU. Memory is not simply throttled: exceeding a memory limit can cause an OOM kill. Set realistic requests and measure with production traffic.

### QoS classes

| QoS | Simplified criteria | Relative behavior under node memory pressure |
|---|---|---|
| `Guaranteed` | Every container has equal CPU request=limit and memory request=limit | Lower eviction priority generally |
| `Burstable` | At least one CPU/memory request or limit, but not Guaranteed | Middle |
| `BestEffort` | No CPU/memory requests or limits | More vulnerable to eviction |

Eviction also considers usage above requests and other signals; QoS is not an absolute eviction order.

### HPA, VPA, Cluster Autoscaler

- **HPA (Horizontal Pod Autoscaler):** Adjusts replicas, usually via resource/custom/external metrics.
- **VPA (Vertical Pod Autoscaler):** Recommends or adjusts resource requests via installed components; coordinate with HPA to avoid overlapping control of the same signal.
- **Cluster Autoscaler:** Adjusts node-group capacity when Pods cannot schedule / nodes are underused, given cloud or node-pool support.
- **KEDA:** Optional event-driven autoscaling add-on, useful for queue depth and external event sources.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
  namespace: demo
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 6
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

**HPA preconditions:** CPU/memory utilization scaling typically requires metrics collection (commonly metrics-server) and usable resource requests. HPA scales **Pods**, not nodes; a separate node autoscaler may be needed if new Pods stay Pending.

```bash
kubectl top nodes
kubectl top pods -n demo
kubectl get hpa -n demo
kubectl describe hpa web-hpa -n demo
```

If `kubectl top` fails, first verify your metrics API/components. `kubectl top` is not guaranteed on a newly created local cluster.

---

## 9. Health Probes and Pod Lifecycle

| Probe | Question | On failure |
|---|---|---|
| **Readiness** | Can the container serve requests? | Marks Pod not ready; usually removed from Service ready endpoints |
| **Liveness** | Should this container be restarted? | Kubelet restarts the failing container after threshold |
| **Startup** | Has the application finished startup? | Defers liveness/readiness until successful; persistent failure restarts container |

Probe modes: HTTP GET, TCP socket, command execution (`exec`), and gRPC health checks.

```yaml
startupProbe:
  httpGet:
    path: /
    port: 80
  failureThreshold: 12
  periodSeconds: 5
readinessProbe:
  httpGet:
    path: /
    port: 80
  periodSeconds: 5
  timeoutSeconds: 2
livenessProbe:
  httpGet:
    path: /
    port: 80
  periodSeconds: 10
  timeoutSeconds: 2
```

**Pro tip:** Don't make liveness depend on every downstream service; otherwise a database outage can restart every frontend Pod simultaneously. Readiness may depend on essential serviceability; liveness should detect a container that genuinely needs restarting.

### Graceful termination

A typical graceful shutdown sequence is: Pod deletion begins → Pod is marked terminating and removed from regular ready traffic → optional `preStop` hook and SIGTERM handling → process drains active work → kubelet sends SIGKILL if it outlasts its termination grace. Timing details and signal order depend on hooks and runtime; test application behavior, don't assume zero dropped connections.

```yaml
spec:
  terminationGracePeriodSeconds: 40
```

---

## 10. Volumes, PV, PVC, StorageClass and CSI

| Term | Role |
|---|---|
| `emptyDir` | Temporary Pod-scoped storage; deleted when Pod is removed from node |
| `PersistentVolume` (PV) | Cluster storage resource representing provisioned capacity |
| `PersistentVolumeClaim` (PVC) | Application request for persistent storage |
| `StorageClass` | Defines how dynamic volumes should be provisioned |
| CSI | Container Storage Interface; storage driver integration |

### Access modes

- `ReadWriteOnce` (**RWO**): Read-write mount by a single **node**, not necessarily a single Pod.
- `ReadOnlyMany` (**ROX**): Read-only mount from many nodes, if supported.
- `ReadWriteMany` (**RWX**): Read-write from many nodes, if supported.
- `ReadWriteOncePod` (**RWOP**): Restricts the PVC to one Pod at a time for supported CSI storage.

Access modes describe mounting capability; they are not a substitute for application data integrity guarantees.

### PVC example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  # storageClassName: <existing-storage-class>  # optional if default exists
```

Then in a **Pod** spec:

```yaml
spec:
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: data-pvc
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: data
          mountPath: /data
```

**Storage troubleshooting:** `PVC Pending` → inspect PVC events, `StorageClass`, CSI controller, capacity, access mode, binding mode, topology, permissions, and cloud quotas. `CrashLoopBackOff` due to permission errors after a mount is a different problem.

### Storage reclaim policy

- `Delete`: Usually deletes dynamically provisioned backing storage when the claim is removed, subject to driver behavior.
- `Retain`: Keeps PV/backing data for deliberate recovery and manual management.

**Careful:** Deleting PVCs, namespaces, or StatefulSets can have durable data implications. Back up and confirm reclaim policies before destructive actions. Backups and restore drills are separate responsibilities.

---

## 11. Security: RBAC, Secrets, NetworkPolicy, Pod Security

### Security layers

1. **Identity:** Users, groups, ServiceAccounts, workload identity.
2. **Authorization:** RBAC Roles/ClusterRoles and bindings.
3. **Admission:** Built-in/custom admission policies or webhooks.
4. **Workload hardening:** Non-root, read-only filesystem when feasible, drop capabilities, seccomp, least privilege.
5. **Isolation:** Namespaces, Pod security controls, NetworkPolicies, node isolation.
6. **Supply chain:** Image scanning/signatures, SBOM, provenance, policy, minimal trusted base images.
7. **Secrets:** Appropriate secret store, access control, encryption at rest and transport, rotation.

### RBAC: Role vs ClusterRole, RoleBinding vs ClusterRoleBinding

- **Role:** Grants permissions in one namespace.
- **ClusterRole:** Defines cluster-scoped rules, or reusable namespaced rules.
- **RoleBinding:** Applies a Role or ClusterRole **within a namespace**.
- **ClusterRoleBinding:** Applies a ClusterRole cluster-wide.

Example: allow a ServiceAccount to read Pods in namespace `demo`.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pod-reader
  namespace: demo
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: demo
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: demo
subjects:
  - kind: ServiceAccount
    name: pod-reader
    namespace: demo
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
```

```bash
kubectl auth can-i get pods -n demo
kubectl auth can-i list secrets -n demo
kubectl auth can-i list pods \
  --as=system:serviceaccount:demo:pod-reader -n demo
```

Impersonation itself requires sufficient permission, so the third command can be denied before it evaluates the intended ServiceAccount.

### Secret vs ConfigMap

ConfigMaps hold non-confidential config. Secrets are designed for sensitive values, **but ordinary Secret values are base64-encoded, not inherently encrypted by base64**. Use RBAC, encryption at rest, restricted node access, and an external secret manager/CSI integration where appropriate. Avoid printing real secrets to logs or committing them to Git.

```bash
# Demo-only placeholder. Do NOT place real passwords in shell history/repositories.
kubectl create secret generic demo-placeholder -n demo \
  --from-literal=EXAMPLE_VALUE=not-a-real-secret
kubectl get secret demo-placeholder -n demo
```

### Pod Security Standards and admission

Three profiles: `Privileged`, `Baseline`, and `Restricted`. Use namespace admission labels to enforce/audit/warn according to cluster policy. Restricted commonly requires non-root, allowed seccomp profile, dropping capabilities, and disallowing privilege escalation for Linux containers.

```bash
# Example only: first audit your existing workloads before enforcement.
kubectl label namespace demo \
  pod-security.kubernetes.io/audit=restricted
```

### NetworkPolicy

By default, Kubernetes NetworkPolicy does not isolate a Pod unless a policy selects it, and **policy objects have no effect unless the CNI/implementation enforces them**.

```yaml
# Deny ingress to Pods in the demo namespace.
# Apply only where a NetworkPolicy-compatible CNI is present.
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: demo
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

**Important:** A deny policy can break the lab's HTTP tests. Explicitly allow only required callers/ports when using real segmentation. Egress-deny rules can break DNS unless DNS traffic is separately allowed.

---

## 12. Helm, Kustomize, GitOps and CI/CD

### Helm

Helm is a Kubernetes package manager: **Chart** = packaged templates/configuration, **release** = an installed chart instance, **values** = configurable inputs.

```bash
helm list -A
helm show values <chart-reference>
helm template demo <chart-reference> -f values.yaml
helm upgrade --install demo <chart-reference> -n demo --create-namespace
helm history demo -n demo
helm rollback demo <revision> -n demo
```

Helm release rollback re-applies prior release manifests; it is not a database rollback and may not reverse external side effects. Choose pinned, trusted charts and review output before deployment.

### Kustomize

Kustomize composes Kubernetes YAML through bases and overlays without needing a chart template language.

```bash
kubectl kustomize ./overlays/prod
kubectl apply -k ./overlays/prod
```

### GitOps: Argo CD / Flux

A GitOps controller continuously compares an approved declarative source (often Git) with cluster state and can synchronize drift. Recommended practices include pull requests, reviews, protected branches, env overlays, secret management, drift detection, and automated rollback/forward-fix policies.

### Jenkins → Kubernetes example pipeline flow

```text
Developer push / Gerrit change
       |
       v
Jenkins CI: checkout -> tests -> build image -> security checks
       |
       v
Push immutable image digest to container registry
       |
       v
Update deployment manifests / Helm values in Git
       |
       v
GitOps controller syncs target cluster
       |
       v
Rollout verification -> smoke test -> metrics/alerts
```

**Build vs deploy separation:** CI should not require broad production-cluster-admin permissions. Prefer immutable image digests, least-privileged deployment identities, approvals, and traceability to commits/builds.

### Rollout approaches

- **Rolling update:** Replace Pods gradually; `maxSurge` and `maxUnavailable` tune availability.
- **Recreate:** Tear down old Pods before replacing them; risks downtime.
- **Blue/green:** Maintain two environments, then switch traffic.
- **Canary:** Gradually shift some traffic to a new version, usually with gateway/service-mesh/rollout tooling.

`kubectl rollout undo` rolls back a Deployment revision if history is available; it does not automatically undo database migrations or external API side effects.

---

## 13. Observability, Reliability and SRE Practices

### What to monitor

| Layer | Signals |
|---|---|
| Control plane | API errors/latency, etcd health, scheduler failures |
| Nodes | CPU/memory/disk pressure, kubelet health, filesystem, network |
| Pods | Restarts, Pending count, OOM kills, readiness, CPU throttling |
| Services / Apps | Request rate, latency, error rate, saturation |
| Delivery | Deployment failures, rollout duration, change failure rate |
| Storage / Network | PVC usage, I/O latency, DNS errors, dropped packets |

### Observability tools

- **Prometheus:** Scrapes/stores numeric time-series metrics.
- **Grafana:** Dashboards and visualizations.
- **Alertmanager:** Routes/deduplicates/group notifications.
- **OpenTelemetry:** Instrumentation and collection standards for metrics, logs, and traces.
- **Loki / Elasticsearch / other log stores:** Centralized logs (depending on platform).
- **Events:** Short-lived Kubernetes operational hints; do not treat Events as long-term log retention.

### SRE language

- **SLI:** Measured indicator, e.g., successful request percentage.
- **SLO:** Reliability target, e.g., 99.9% successful requests in a month.
- **Error budget:** Allowed unreliability relative to an SLO.
- **MTTR:** Mean time to restore/recover service.
- **RTO/RPO:** Recovery time objective / acceptable data-loss window.

### Four golden signals

**Latency**, **traffic**, **errors**, **saturation**. Couple alerts to user impact; a Pod restart alone is often a diagnostic signal, not necessarily a page-worthy outage.

### High availability checklist

- At least 2–3 replicas as appropriate for service criticality and capacity.
- Spread replicas across nodes / failure domains.
- Good readiness/startup probes and tested graceful shutdown.
- Meaningful requests and appropriately measured limits.
- PDBs and capacity headroom for planned maintenance.
- Tested rollback, backups, restores, incident runbooks, and alerts.
- Multiple control-plane instances/etcd quorum where the platform requires HA.

---

## 14. Hands-on Deployment Lab

This lab uses **a local Kubernetes cluster** (minikube/kind) or a disposable nonproduction namespace. It creates a namespace, an NGINX Deployment, and a ClusterIP Service. PDB and HPA are optional add-ons.

> **Safety:** The NGINX image in this exercise is for learning, not a full restricted-security production deployment. Use vetted image versions/digests and your organization's policies in production.

### Prerequisites

```bash
kubectl cluster-info
kubectl get nodes
# For minikube users, start your cluster if it isn't running:
# minikube start
```

### File: `app.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: demo
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: demo
  labels:
    app: web
spec:
  replicas: 3
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      terminationGracePeriodSeconds: 30
      containers:
        - name: nginx
          image: nginx:stable-alpine
          ports:
            - name: http
              containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 500m
              memory: 256Mi
          startupProbe:
            httpGet:
              path: /
              port: http
            periodSeconds: 5
            failureThreshold: 12
          readinessProbe:
            httpGet:
              path: /
              port: http
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /
              port: http
            periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: web
  namespace: demo
spec:
  selector:
    app: web
  type: ClusterIP
  ports:
    - name: http
      port: 80
      targetPort: http
```

### Deploy, inspect, test

```bash
kubectl apply -f app.yaml
kubectl rollout status deployment/web -n demo
kubectl get deployments,pods,svc -n demo -o wide
kubectl get endpointslices -n demo -l kubernetes.io/service-name=web

# Terminal 1: leave port-forward running
kubectl port-forward -n demo svc/web 8080:80

# Terminal 2: test response
curl -i http://localhost:8080/
```

### Scale and roll back

```bash
kubectl scale deployment/web -n demo --replicas=4
kubectl get pods -n demo -w  # Ctrl+C to stop watch

# A rollout is triggered when Pod-template fields change.
# For a reproducible exercise, replace stable-alpine with a tested image tag
# in app.yaml and apply it again, then run:
kubectl rollout status deployment/web -n demo
kubectl rollout history deployment/web -n demo
kubectl rollout undo deployment/web -n demo
```

**Why no `kubectl set image` command with a made-up version?** Use a version/digest known to exist in your registry. An invalid tag is itself a useful ImagePullBackOff exercise, but not a reliable successful-rollout example.

### Optional: apply PDB and HPA

Save this as `availability.yaml`:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
  namespace: demo
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
  namespace: demo
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 6
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

```bash
kubectl apply -f availability.yaml
kubectl get pdb,hpa -n demo
kubectl describe hpa web-hpa -n demo
# An HPA needs metrics; on many local clusters metrics-server requires setup.
```

**HPA and desired replicas:** If you enable HPA, avoid continuously reapplying a fixed `spec.replicas` through your GitOps tool; otherwise it can fight the autoscaler. Coordinate field ownership and omit `replicas` from the managed workload manifest when appropriate.

### Cleanup

```bash
# WARNING: deletes the whole demo namespace and every resource inside it.
kubectl delete namespace demo
```

---

## 15. Troubleshooting Playbook

### Universal triage: observe → isolate → mitigate → prevent

```bash
# 1. Confirm target
kubectl config current-context
kubectl get namespaces

# 2. Look at scope and events
kubectl get pods -A -o wide
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp

# 3. Inspect affected Pod and owner
kubectl describe pod <pod> -n <namespace>
kubectl get pod <pod> -n <namespace> -o yaml
kubectl get deployment <deployment> -n <namespace> -o wide

# 4. Inspect logs (possibly per container)
kubectl logs <pod> -n <namespace> -c <container> --tail=100
kubectl logs <pod> -n <namespace> -c <container> --previous --tail=100

# 5. Inspect node and resources
kubectl get nodes -o wide
kubectl describe node <node>
kubectl top pods -n <namespace>  # requires Metrics API

# 6. Inspect connectivity if relevant
kubectl get svc,endpointslices -n <namespace>
```

### Symptoms → checks → likely solutions

| Symptom | Check first | Typical resolution |
|---|---|---|
| `Pending` | `describe pod` Events; requests, taints, selectors, PVC | Add capacity, fix selector/toleration, PVC provisioning |
| `ImagePullBackOff` | Image/tag, registry auth, rate limits, DNS, pull secret | Correct digest/tag, credentials, network |
| `CrashLoopBackOff` | `logs --previous`, exit code, startup config/probes | Fix app/env/command, dependencies, probe configuration |
| `OOMKilled` | `describe pod`, memory graph, limit, memory leak | Fix leak, right-size memory, load test |
| `CreateContainerConfigError` | `describe pod` missing Secret/ConfigMap/key | Correct referenced configuration |
| `ContainerCreating` stuck | Events, volumes, CNI, image pull, runtime | Fix mount/network/runtime issue |
| Pod Running but not Ready | Readiness results and dependencies | Fix probe path/port or app readiness |
| Service returns 503/timeouts | Service selector, EndpointSlices, readiness, route | Correct labels/ports/routing, restore backends |
| DNS lookup fails | CoreDNS, namespace, resolver config, policies | Fix DNS components/policy/upstream |
| PVC stuck Pending | PVC events, StorageClass/CSI, topology | Fix provisioning/capacity/access modes |
| Pod cannot schedule | Node resources, taints, affinity, PVC topology | Change requirements or add matching nodes |
| Node `NotReady` | kubelet, runtime, disk/memory pressure, network | Recover node safely, drain/fail over if possible |
| HPA `unknown` metrics | `kubectl top`, metrics API, resource requests | Fix metrics-server/adapters/requests |
| Rollout stuck | `rollout status`, new ReplicaSet, Pod events | Fix new version; consider controlled rollback |
| Unauthorized/Forbidden | `kubectl auth can-i`, RoleBindings, identity | Add *minimal* necessary RBAC |
| NetworkPolicy blocked traffic | Selected Pods, direction, DNS, CNI support | Add precise allow rules |

### Key distinctions interviewers test

**`CrashLoopBackOff` vs `ImagePullBackOff`**

- CrashLoop: container starts and repeatedly exits/fails.
- ImagePull: image could not be retrieved; no successful start.

**`Pending` vs `ContainerCreating`**

- Pending phase can include scheduling and setup; `ContainerCreating` is a container waiting reason during setup.
- Review Events to determine whether the blocker is scheduling, volumes, sandbox network creation, registry pull, or something else.

**Pod Running vs application healthy**

- `Running` does not guarantee `Ready`, correct business behavior, or reachable Service traffic.

**`kubectl logs` vs `kubectl describe`**

- `logs`: application stdout/stderr, if emitted by container.
- `describe`: Kubernetes state and Events such as FailedScheduling, FailedMount, probe failures, image-pull errors.

### Practical debugging command examples

```bash
# Watch namespace-specific events (if allowed)
kubectl get events -n demo --watch

# Inspect deployment revision and controller state
kubectl describe deployment web -n demo
kubectl get rs -n demo -l app=web

# Run temporary client Pod; remove after test
kubectl run tmp-client --rm -it --restart=Never \
  --image=busybox:1.36 -n demo -- sh
# Inside: wget -qO- http://web.demo.svc.cluster.local

# Inspect container command / environment (avoid dumping secrets)
kubectl get pod <pod> -n demo -o jsonpath='{.spec.containers[*].name}'

# Debug a running Pod by attaching an ephemeral container
# (requires permission and a cluster/runtime supporting ephemeral containers)
kubectl debug -it pod/<pod> -n demo --image=busybox:1.36 --target=<container> -- sh
```

---

## 16. Top Interview Questions and Answers

### Fundamentals and architecture (Q1–Q15)

**Q1. What is Kubernetes?**  
A declarative orchestration platform that runs and manages containerized workloads using APIs, schedulers, and reconciling controllers.

**Q2. Why is Kubernetes used?**  
To standardize deployment, scaling, discovery, resilience, and policy across machines and environments.

**Q3. What is a Pod?**  
The smallest deployable unit; its containers share an IP/port namespace and can share volumes.

**Q4. Why not deploy only standalone Pods?**  
Controllers provide replacement, scaling, rollout handling, and desired-state enforcement.

**Q5. What is the control plane?**  
The API server, scheduler, controller manager, etcd, and optional cloud-specific components managing the cluster.

**Q6. What does etcd store?**  
Persistent Kubernetes API cluster state, including resource objects; protect its backups and access.

**Q7. What is a kubelet?**  
An agent on each node that ensures the assigned workload containers are running and reports status.

**Q8. What does kube-proxy do?**  
Implements Service traffic forwarding in many clusters. Alternative networking dataplanes can replace it.

**Q9. What is CNI?**  
A plugin interface used by Kubernetes networking implementations to connect Pods.

**Q10. What is CRI?**  
The Container Runtime Interface between kubelet and compatible runtimes such as containerd or CRI-O.

**Q11. What does the scheduler do?**  
Binds unscheduled Pods to nodes using constraints, available capacity, and scoring rules.

**Q12. Is Kubernetes push-based or pull-based?**  
Its controllers reconcile observed state; kubelets watch their assigned Pods through the API, while deployment tooling may push configuration or use GitOps pull reconciliation.

**Q13. What is a namespace?**  
An organizational scope for namespaced objects, quotas, RBAC and some policies—not a complete isolation/security boundary.

**Q14. Labels vs annotations?**  
Labels identify/select objects; annotations store supplemental metadata.

**Q15. What is a CRD?**  
A CustomResourceDefinition extends the Kubernetes API with new resource kinds, often reconciled by an Operator.

### Workloads and rollout (Q16–Q27)

**Q16. Deployment vs StatefulSet?**  
Deployment for interchangeable replicas; StatefulSet for stable identities and per-replica storage semantics.

**Q17. Deployment vs ReplicaSet?**  
A Deployment manages rollout history and ReplicaSets; a ReplicaSet just maintains replica count.

**Q18. DaemonSet use case?**  
One agent Pod per eligible node, like node-level log/metrics collectors.

**Q19. Job vs CronJob?**  
Job executes a finite workload to completion; CronJob creates Jobs from a schedule.

**Q20. Init container purpose?**  
Runs setup steps before regular app containers, e.g., waiting for a prerequisite or preparing files.

**Q21. What is rolling update?**  
Gradually replaces old ReplicaSet Pods with new ones, controlled by `maxSurge`/`maxUnavailable`.

**Q22. Can a rolling update be zero-downtime?**  
It can minimize interruption if enough healthy replicas, capacity, correct readiness and graceful shutdown exist; never assume a guarantee.

**Q23. How do you rollback a Deployment?**  
`kubectl rollout undo deployment/<name> -n <ns>`; inspect revision history and whether schema/data changes also need work.

**Q24. Why is Pod stuck in CrashLoopBackOff?**  
Repeated container failure; inspect prior logs, exit status, app configuration, dependencies, and probes.

**Q25. Is CrashLoopBackOff a Pod phase?**  
No; it's a container waiting reason.

**Q26. What happens if a Deployment Pod is deleted?**  
The controller normally creates a replacement to restore desired replicas.

**Q27. How do you update image version?**  
Update Pod template using manifest/GitOps or `kubectl set image`, verify rollout, and trace image digest to build.

### Services/networking (Q28–Q39)

**Q28. What is a Service?**  
Stable IP/DNS and traffic abstraction for selected backends, despite ephemeral Pods.

**Q29. ClusterIP vs NodePort?**  
ClusterIP is internal; NodePort additionally exposes an address/port on cluster nodes.

**Q30. NodePort vs LoadBalancer?**  
NodePort exposes a node port; LoadBalancer requests an external LB through an implementation.

**Q31. What is an Ingress?**  
HTTP(S) routing resource that requires a working controller.

**Q32. Gateway API vs Ingress?**  
Gateway API uses more expressive, role-separated resource types; Ingress API is frozen and remains supported.

**Q33. How does service discovery work?**  
Cluster DNS, usually CoreDNS, resolves Service names to their associated network addresses.

**Q34. What if a Service has no endpoints?**  
Check Pod labels, selector, readiness, namespace, target ports, and EndpointSlices.

**Q35. What is a headless Service?**  
`clusterIP: None`, enabling DNS discovery of Pod endpoints without a normal Service VIP.

**Q36. What is a NetworkPolicy?**  
A resource to specify permitted L3/L4 Pod ingress/egress traffic, enforced by a supporting implementation.

**Q37. Can Pods share the same IP?**  
Containers inside the same Pod share its IP; distinct normal Pods have distinct Pod addresses within their networking environment.

**Q38. What is `targetPort`?**  
The destination port on backend Pods, separate from the Service's client-facing `port`.

**Q39. How would you debug DNS?**  
Test lookup from a Pod, verify Service/namespace, CoreDNS health, policy, and network path.

### Scheduling, capacity and storage (Q40–Q53)

**Q40. Requests vs limits?**  
Requests influence scheduling; limits constrain runtime usage (CPU throttling; memory OOM behavior).

**Q41. What does `100m` CPU mean?**  
0.1 CPU core.

**Q42. What is HPA?**  
Controller that adjusts replicas based on observed metrics relative to configured targets.

**Q43. Does HPA add nodes?**  
No; a node autoscaler is a separate component.

**Q44. What is VPA?**  
Add-on for recommending/adjusting CPU and memory requests rather than replica count.

**Q45. Why is an HPA not scaling?**  
Check metrics APIs, resource requests, HPA targets, min/max bounds, readiness, and conditions.

**Q46. What are taints and tolerations?**  
Nodes repel non-tolerating Pods; Pods express permission to run despite certain taints.

**Q47. What is node affinity?**  
Scheduling requirement/preference based on node labels.

**Q48. What is Pod anti-affinity?**  
Rules that discourage or prohibit placing certain matching Pods together.

**Q49. What is PDB?**  
Availability guard for supported voluntary Pod evictions; cannot prevent involuntary failures.

**Q50. PV vs PVC?**  
PV is a storage resource; PVC is a workload's request/claim for storage.

**Q51. What is StorageClass?**  
Provisioning parameters and policies for storage, including dynamic provisioning through a driver.

**Q52. RWO vs RWX?**  
RWO generally allows read-write mounts from one node; RWX allows many nodes where supported.

**Q53. What is CSI?**  
Standard interface for storage systems/drivers to integrate with container orchestrators.

### Security, platform and SRE (Q54–Q70)

**Q54. ConfigMap vs Secret?**  
ConfigMap stores non-secret config; Secret is the API object for sensitive values, but base64 alone is not encryption.

**Q55. How do you secure Kubernetes Secrets?**  
Use least-privilege RBAC, encryption at rest, workload identity/external secret stores, rotation, and safe logging.

**Q56. Role vs ClusterRole?**  
Role is namespace-specific; ClusterRole defines rules usable cluster-wide or through a namespace RoleBinding.

**Q57. What does a ServiceAccount represent?**  
An identity for workloads interacting with the API or integrated identity systems.

**Q58. What is Pod Security Admission?**  
Built-in admission mechanism to enforce/warn/audit Pod Security Standards at namespace scope.

**Q59. What is a readiness probe?**  
Indicates readiness to receive requests, without necessarily restarting the container.

**Q60. What is a liveness probe?**  
Detects a container that should be restarted.

**Q61. When use a startup probe?**  
For slow-starting apps, so liveness/readiness don't react prematurely during initialization.

**Q62. Why use Helm?**  
Package, configure, install, upgrade and manage related Kubernetes manifests as releases.

**Q63. Helm vs Kustomize?**  
Helm renders parameterized chart templates; Kustomize patches/composes native YAML.

**Q64. What is GitOps?**  
Reconciliation of cluster configuration against an approved declarative source, usually tracked in Git.

**Q65. What is Argo CD?**  
GitOps controller for declaring and reconciling Kubernetes application resources from a source repository.

**Q66. What metrics matter for an SRE?**  
Request latency, error rate, traffic, saturation, plus resource health, availability, SLOs and error-budget burn.

**Q67. What is OOMKilled?**  
A process/container was terminated by OOM handling; examine cgroups, memory limit, node pressure and application usage.

**Q68. What happens during a node failure?**  
The control plane detects unhealthy/unreachable nodes and eligible controllers can arrange replacement Pods elsewhere; timing and storage constraints matter.

**Q69. How do you deploy safely to production?**  
Use reviewed manifests, immutable images, least privilege, staged/canary rollout, health checks, metrics, rollback plans, and change records.

**Q70. What is your Kubernetes incident-response workflow?**  
Determine blast radius and user impact; inspect deployments, Pods, Events, metrics and dependencies; mitigate safely; establish cause and prevention.

---

## 17. Real-World Scenario-Based Questions

Use this response pattern in interviews: **symptom → scope/impact → targeted commands → root cause hypotheses → safe mitigation → prevention**.

### Scenario 1 — Deployment has 3 replicas, but only 1 is Ready

1. `kubectl get deploy,rs,pods -n <ns> -o wide`.
2. `kubectl describe pod <not-ready-pod> -n <ns>` to inspect failure reasons.
3. Check startup/readiness endpoints, application logs, requests/limits, PVCs, and node health.
4. Fix the broken image/config/probe or missing capacity; validate endpoints and rollout.
5. Add canary tests, meaningful probes, and appropriate alerts.

### Scenario 2 — A production deployment causes 5xx errors

1. Check user impact, error rates and deployment correlation.
2. Inspect rollout status, newly created Pods, and readiness/Service endpoints.
3. Stop rollout and consider rollback/traffic shifting if safe.
4. Investigate app logs, config differences, downstream health, and migrations.
5. Conduct a blameless review and add regression gates/observability.

### Scenario 3 — Node CPU is 100%, Pods are slow

1. Identify node/Pod pressure using dashboards and `kubectl top` where available.
2. Compare CPU throttling, CPU requests/limits, app latency, and contention.
3. Scale workload or cluster, rebalance, and fix hot loops where appropriate.
4. Right-size requests, configure HPA and node autoscaling, load test and alert.

### Scenario 4 — Pod is OOMKilled repeatedly

1. Inspect `kubectl describe pod`, exit reason, memory peak and prior logs.
2. Look for memory leak, large caches, poor request/limit sizing or node pressure.
3. Tune the app and safely adjust memory limit/request after measurement.
4. Validate under load; add alerts and regression checks.

### Scenario 5 — Service resolves but requests time out

1. Verify Service port/`targetPort`, selector, ready EndpointSlices.
2. Test from an in-cluster Pod; compare direct Pod connectivity vs Service.
3. Check NetworkPolicies, CNI dataplane, kube-proxy/eBPF path and app listening address.
4. Fix the component that breaks the path, then test end-to-end.

### Scenario 6 — DNS names fail inside Pods

1. Inspect `/etc/resolv.conf` inside affected Pods.
2. `nslookup kubernetes.default.svc.cluster.local` from a debug Pod (default domain assumption).
3. Check CoreDNS deployment and `kube-dns` Service, policy, upstream DNS, and CNI.
4. Restore DNS connectivity/capacity; add DNS SLI/alerts if critical.

### Scenario 7 — PVC remains Pending

1. `kubectl describe pvc <claim> -n <ns>`; read events.
2. Inspect `StorageClass`, CSI driver, binding mode, topology and storage quotas.
3. Check dynamically provisioned storage and capacity.
4. Fix storage provisioning without deleting valuable existing data.

### Scenario 8 — All Pods schedule to one node

1. Check resource requests, selectors and placement rules.
2. Add **topologySpreadConstraints** or suitable anti-affinity.
3. Check node/zone capacity, scheduler events and tolerance requirements.
4. Roll safely and verify spread; recognize multi-zone storage limitations.

### Scenario 9 — Maintenance drain is blocked

1. Inspect PDB: `kubectl get pdb -A` and `kubectl describe pdb <name> -n <ns>`.
2. Check available/desired healthy Pods and whether more replicas can start.
3. Improve application readiness or capacity; plan maintenance window.
4. Avoid bypassing eviction protection without documented incident-level authorization.

### Scenario 10 — Image pulls work locally but fail in cluster

1. Verify image registry/digest and pull policy.
2. Check Pod Events and node network/DNS/proxy connectivity.
3. Validate imagePullSecrets or workload identity and registry permissions.
4. Fix authenticated pulls; prefer reproducible immutable images.

### Scenario 11 — HPA never scales under traffic

1. Check `kubectl describe hpa` for conditions.
2. Verify `kubectl top`/metrics APIs, CPU requests, target, min/max bounds.
3. Check target metrics represent actual pressure, not just low-CPU I/O waits.
4. Select better external/application metrics where needed.

### Scenario 12 — Two microservices should talk, but all other ingress must be blocked

1. Label workloads/namespaces intentionally.
2. Install/use a NetworkPolicy-capable CNI.
3. Apply a deny-ingress policy to target Pods plus a specific ingress allow policy for trusted source Pods/ports.
4. Test allowed and denied paths and the app's DNS/egress needs.

### Scenario 13 — A stateful Pod restarts on a different node

1. Check PVC status/access mode and storage topology constraints.
2. Distinguish Pod rescheduling from detached/attached storage recovery.
3. Investigate filesystem/data integrity, driver events and attach time.
4. Test backup, recovery, failover and zone-failure behavior separately.

### Scenario 14 — Cluster nodes are `NotReady`

1. Check node conditions and Events.
2. Inspect kubelet, container runtime, disk/memory pressure and network reachability using permitted node-level access.
3. Review disruption/failover; cordon/drain only where safe and possible.
4. Restore node dependencies, then verify scheduling and service health.

### Scenario 15 — One namespace cannot deploy because of policy

1. Read the API error and related Events; distinguish RBAC denial, admission rejection, ResourceQuota, LimitRange and Pod Security enforcement.
2. `kubectl auth can-i ...` for authorization; check namespace policies.
3. Change the workload to satisfy policy or obtain reviewed least-privilege changes.
4. Never “fix” it by assigning `cluster-admin` broadly.

---

## 18. kubectl Cheat Sheet

> Replace `<...>` placeholders. Destructive commands are marked ⚠️; check current context before executing them.

| Intent | Command |
|---|---|
| Cluster info | `kubectl cluster-info` |
| Context | `kubectl config current-context` |
| Nodes | `kubectl get nodes -o wide` |
| All Pods | `kubectl get pods -A -o wide` |
| Namespace workloads | `kubectl get deploy,sts,ds,svc -n <ns>` |
| Object details | `kubectl describe pod <pod> -n <ns>` |
| Events | `kubectl get events -n <ns> --sort-by=.metadata.creationTimestamp` |
| Current logs | `kubectl logs <pod> -n <ns> -c <container> -f` |
| Previous logs | `kubectl logs <pod> -n <ns> -c <container> --previous` |
| Exec shell | `kubectl exec -it <pod> -n <ns> -- sh` |
| Apply file | `kubectl apply -f app.yaml` |
| Validate against server | `kubectl apply --dry-run=server -f app.yaml` |
| Manifest diff | `kubectl diff -f app.yaml` |
| Scale | `kubectl scale deployment/<name> --replicas=3 -n <ns>` |
| Rollout status | `kubectl rollout status deployment/<name> -n <ns>` |
| Rollback ⚠️ | `kubectl rollout undo deployment/<name> -n <ns>` |
| Expose locally | `kubectl port-forward svc/<name> 8080:80 -n <ns>` |
| Resources | `kubectl top pods -A` (metrics API required) |
| View HPA | `kubectl get hpa -A` |
| View PVC/PV | `kubectl get pvc -A && kubectl get pv` |
| RBAC check | `kubectl auth can-i get pods -n <ns>` |
| Service backends | `kubectl get endpointslices -n <ns>` |
| Mark node unschedulable ⚠️ | `kubectl cordon <node>` |
| Evict eligible Pods ⚠️ | `kubectl drain <node> --ignore-daemonsets` |
| Resume scheduling | `kubectl uncordon <node>` |
| Delete a Pod ⚠️ | `kubectl delete pod <pod> -n <ns>` |
| Namespace deletion ⚠️ | `kubectl delete namespace <ns>` |

**Remember:** `kubectl get all` does **not** literally show all Kubernetes resource types; explicitly ask for Secrets, ConfigMaps, PVCs, NetworkPolicies, CRDs, etc., as needed.

### Quick one-liners

```bash
kubectl get pods -n demo -l app=web -o wide
kubectl get pods -A --field-selector=status.phase=Pending
kubectl get deployment web -n demo -o yaml
kubectl explain pod.spec.containers.resources
kubectl get nodes --show-labels
kubectl get storageclasses
kubectl get crds
kubectl api-resources
```

---

## 19. Practice Tasks and 7-Day Revision Plan

### Hands-on checklist

- [ ] Start minikube/kind and verify the context.
- [ ] Deploy a 3-replica NGINX Deployment from YAML.
- [ ] Expose it with ClusterIP and test using port-forward.
- [ ] Test in-cluster DNS from a disposable debug Pod.
- [ ] Scale 3 → 5 replicas and inspect the ReplicaSet.
- [ ] Perform a safe rolling image update and rollback.
- [ ] Intentionally use an invalid image tag; diagnose `ImagePullBackOff`.
- [ ] Create a failing container to practice `CrashLoopBackOff` investigation.
- [ ] Add a ConfigMap, Secret placeholder, and resource requests/limits.
- [ ] Create a PVC using an available StorageClass and inspect binding.
- [ ] Try HPA after installing/verifying metrics collection.
- [ ] Apply a PDB and explain why drain may be blocked.
- [ ] Test RBAC with a limited ServiceAccount and `kubectl auth can-i`.
- [ ] Use labels, taints/tolerations, and spread constraints in an isolated cluster.
- [ ] Deploy a demo via Helm; inspect rendered templates/revision history.
- [ ] Explain a Jenkins → registry → GitOps → Kubernetes delivery pipeline.

### 7-day plan

| Day | Focus | Deliverable |
|---|---|---|
| 1 | Basics, control plane, Pod lifecycle, namespaces | Explain request flow without notes |
| 2 | Deployments, StatefulSets, Jobs, probes | Deploy, roll out and roll back an app |
| 3 | Services, DNS, CNI, Ingress/Gateway | Debug a broken app connection |
| 4 | Scheduling, resource management, HPA, PDB | Explain a Pending Pod and scale bottleneck |
| 5 | PV/PVC/CSI, RBAC, Secrets, NetworkPolicy | Fix a storage or permission failure |
| 6 | Helm, GitOps, CI/CD, monitoring/SLOs | Walk through a full release workflow |
| 7 | 70 questions + 15 scenarios + mock interview | Answer clearly, with commands and trade-offs |

### Interview answer framework (STAR + technical depth)

For experience-based questions, structure your response:

1. **Situation:** What environment and reliability/deployment problem existed?
2. **Task:** What were you accountable for?
3. **Action:** Which Kubernetes objects, automation, validation, and observability did you implement?
4. **Result:** State only metrics you can support with actual evidence.
5. **Trade-offs:** What failed initially, what would you improve, and how did you minimize risk?

**Sample answer shape (adapt it to your real experience; do not claim unperformed work):**

> “I used a local Kubernetes cluster to validate a container deployment, configured a Deployment and Service with resource requests and health probes, verified rollout behavior with `kubectl`, and used Prometheus/Grafana to review workload health. In production, I would additionally account for RBAC, storage/network policies, node capacity, GitOps approval, and a tested rollback strategy.”

---

## 20. Official Learning Resources

These links are maintained by upstream projects; APIs/features can vary by Kubernetes version and provider.

- [Kubernetes Concepts](https://kubernetes.io/docs/concepts/)
- [Cluster Architecture](https://kubernetes.io/docs/concepts/architecture/)
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Liveness, Readiness, Startup Probes](https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/)
- [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Gateway API](https://gateway-api.sigs.k8s.io/)
- [NetworkPolicy](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Scheduling: Taints/Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
- [Resources and Limits](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [Kubernetes Tasks](https://kubernetes.io/docs/tasks/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/quick-reference/)
- [Helm Documentation](https://helm.sh/docs/)
- [Argo CD Documentation](https://argo-cd.readthedocs.io/en/stable/)
- [Prometheus](https://prometheus.io/docs/introduction/overview/)
- [OpenTelemetry](https://opentelemetry.io/docs/)

---

## Final Interview Advice

1. **Explain the control-plane flow**, not just object definitions.
2. **Understand the differences**: Pod vs Deployment, Deployment vs StatefulSet, Service vs Ingress, requests vs limits, readiness vs liveness, HPA vs node autoscaler, PVC vs PV, Role vs ClusterRole.
3. **Know the first 5 debugging commands**: `get`, `describe`, `logs --previous`, `get events`, `get svc/endpointslices`.
4. **Speak in terms of incidents and trade-offs**: availability, user impact, security, blast radius, capacity, rollback, recovery and prevention.
5. **Practice on a real cluster**; command recall becomes much easier after observing actual Pod status and Event messages.

**Goal:** Don't only say *“Kubernetes scales and self-heals applications.”* Be able to explain **which controller acts, which conditions must be true, which commands prove it, and what can still go wrong.**

---

*Prepared as a practical Kubernetes interview study guide for DevOps, SRE and Platform Engineering roles. Review commands and manifests against your cluster version, platform policy and provider before using them outside a lab.*

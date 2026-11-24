# Kubernetes Manifests

These files are for learning and documentation. Values like `myregistry/backend:1.0` and the database password are placeholders, not real.

## The Story: The Restaurant That Never Sleeps

Once upon a time, there was a restaurant called MyApp Bistro. It was popular, sometimes packed, sometimes quiet, and the owner had one rule: *customers should never see an empty kitchen, no matter what happens behind the scenes.*

**You (the Owner)** don't cook or manage shifts personally. You just write down what you want:
> "I want 3 chefs cooking at all times. Use this recipe. If a chef falls sick, replace them immediately."

You hand this paper to the **API Server**, the restaurant's front desk. Every instruction, from anyone, goes through this desk.

The front desk writes everything into **etcd**, a giant notebook that holds the single source of truth: every recipe, every rule, every current status.

The **Scheduler** is the floor manager who decides *which* kitchen station each new chef works at, based on available space.

The **Controller Manager** is a team of tireless shift supervisors constantly asking: *"Does reality match the paper the owner gave us?"* If a chef collapses (a pod crashes), a supervisor calls in a replacement immediately, with no need for you to notice.

Each chef is a **pod**, the actual worker cooking your app's "dish," carrying their own ingredients and tools (the container).

The instruction "always keep 3 chefs" is a **Deployment**. It hands the headcount enforcement to a **ReplicaSet**, a clipboard nailed to the wall that just says "3."

Customers don't know or care which specific chef makes their food. They walk up to the **host stand** (a **Service**), which seats them at whichever station has a free chef.

Each kitchen location is a **Node**, a physical space with its own stove and counter. The whole restaurant chain, with every location, every chef, the front desk, the notebook, and the supervisors all working together, is the **Cluster**.

Some special staff, like fire-alarm inspectors, need to be in *every single room*. That's a **DaemonSet**: one copy, automatically, on every node.

And **kubectl** is the phone or clipboard *you* use to actually talk to the front desk: checking headcounts, submitting new recipes, scaling up for the dinner rush.

**Moral of the story:** You never manage individual chefs. You declare *what you want*, and an army of invisible supervisors makes reality match your wish, forever.

## Cheat Sheet: Concepts

| Term | What it is | Restaurant analogy |
|---|---|---|
| **Cluster** | Whole system: all machines + control components as one | The entire restaurant chain |
| **Node** | A single machine running workloads (has its own OS; pods/containers share that OS, they don't have their own) | One kitchen location |
| **Pod** | Smallest deployable unit, wraps container(s) | A workstation grouping (one or more chefs sharing a station) |
| **Container** | The packaged app/process inside a pod, the thing actually running | The chef doing the cooking |
| **API Server** | Front door, all commands go through it | The front desk |
| **etcd** | Stores the cluster's entire state | The master notebook |
| **Scheduler** | Assigns pods to nodes (automated bin packing) | The floor manager |
| **Controller Manager** | Keeps reality matching desired state | Shift supervisors |
| **Kubelet** | Node agent running containers | On-site station coordinator |
| **Kube-proxy** | Handles node networking | Waiter routing orders |
| **Deployment** | Declares desired app state (one pod template, replicated N times) | Owner's instruction sheet |
| **ReplicaSet** | Enforces exact replica count | Clipboard: "always 3 chefs" |
| **Replica** | A copy of a running pod | Duplicate chef, same job |
| **StatefulSet** | Like Deployment, but for apps needing stable identity + persistent storage (databases) | Chefs with their own personal, permanent locker |
| **Service** | Stable entry point to reach pods | The host stand |
| **ClusterIP** | Internal-only service | Staff-only phone line |
| **NodePort** | Exposes service on every node's port | A side door, fixed number |
| **LoadBalancer** | External cloud load balancer | Main entrance w/ doorman |
| **ExternalName** | Points to external DNS name | Sign pointing elsewhere |
| **Headless Service** | Returns pod IPs directly (`clusterIP: None`) | Direct line to each chef |
| **DaemonSet** | One pod per node, automatically | Fire alarm in every room |
| **HPA (Horizontal Pod Autoscaler)** | Scales the *number* of pods up/down based on metrics | Hiring more chefs during a rush |
| **VPA (Vertical Pod Autoscaler)** | Scales the *resources* (CPU/memory) of existing pods | Giving one chef a bigger station instead of hiring more |
| **ConfigMap** | Stores non-sensitive config data, separate from app code/image | Seasoning chart pinned near the stove |
| **Secret** | Like ConfigMap, but for sensitive data (base64-encoded, not plain text) | The locked safe combination |
| **Service Binding** | Automated mechanism that injects connection details (from an Operator-managed service) into an app, without manual wiring | A standing arrangement where a chef's key card is auto-issued on shift start |
| **Namespace** | Logical separation in a cluster | Separate floors (dev/prod) |
| **kubectl** | CLI to talk to the API Server | The phone/clipboard you use |

**One-liner:** You tell the **API Server** what you want → saved in **etcd** → **Scheduler** places **pods** on **nodes** → **Controller Manager** keeps it all healthy → **Services** route traffic → you run it all through **kubectl**.

## Project structure

```
k8s-manifests/
├── frontend/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── hpa.yaml
├── backend/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── hpa.yaml
└── database/
    ├── secret.yaml
    ├── statefulset.yaml
    └── service.yaml
```

| | Frontend | Backend | Database |
|---|---|---|---|
| Workload | Deployment | Deployment | StatefulSet |
| Service type | `LoadBalancer` (public) | `ClusterIP` (internal) | Headless (`clusterIP: None`) |
| Container port | 3000 | 8080 | 5432 |

## Before you start

- You need a running cluster (minikube, kind or a cloud cluster).
- The HPA needs `metrics-server` in the cluster to read CPU and memory.
- Deploy the database first, then the backend, then the frontend.

## Deploy

Apply one folder (all files inside it):

```bash
kubectl apply -f k8s-manifests/database/
kubectl apply -f k8s-manifests/backend/
kubectl apply -f k8s-manifests/frontend/
```

Apply everything at once:

```bash
kubectl apply -f k8s-manifests/ -R
```

- `apply` creates the object, or updates it if it already exists.
- `-f` is the file or folder. `-R` also reads sub-folders.

## Check status

```bash
kubectl get deployment,statefulset,pods,svc,hpa
```

Shows the main objects in one list.

```bash
kubectl get pods -o wide
```

Shows each pod with its node and IP.

```bash
kubectl get pods -l app=backend
```

Shows only pods with the label `app=backend`.

## Debug

```bash
kubectl describe pod <pod-name>
```

Shows details and recent events. Use it when a pod is not starting (for example `ImagePullBackOff`).

```bash
kubectl logs <pod-name>
```

Shows the output of the container.

```bash
kubectl logs -f <pod-name>
```

Follows the logs live.

```bash
kubectl exec -it <pod-name> -- sh
```

Opens a shell inside the container.

```bash
kubectl port-forward svc/backend-service 8080:8080
```

Opens the internal backend service on `localhost:8080`. Useful because `ClusterIP` is not reachable from outside.

## Scale

```bash
kubectl scale deployment backend --replicas=5
```

Changes the number of pods by hand. If the HPA is active, it can change this number again.

```bash
kubectl get hpa -w
```

Watches the autoscaler live.

## Update

```bash
kubectl set image deployment/backend backend=myregistry/backend:2.0
```

Changes the image and starts a rolling update.

```bash
kubectl rollout status deployment/backend
```

Waits and shows the progress of the update.

```bash
kubectl rollout undo deployment/backend
```

Goes back to the previous version.

## Delete

```bash
kubectl delete -f k8s-manifests/backend/
```

Deletes every object created from the files in that folder.

```bash
kubectl delete pod <pod-name>
```

Deletes one pod. The Deployment creates a new one.

Note: deleting the StatefulSet does not delete its PersistentVolumeClaim. Delete it yourself if you want to remove the data:

```bash
kubectl get pvc
kubectl delete pvc pgdata-postgres-0
```

## Backend to database connection

Add this to the container in `backend/deployment.yaml`:

```yaml
env:
- name: DB_HOST
  value: "postgres-service"
- name: DB_PORT
  value: "5432"
```

`postgres-service` is the DNS name of the database Service.

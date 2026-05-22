# Docker + Kubernetes

---

## Docker

Packages your application and everything it needs (code, runtime, libraries, config) into a **container**.

### Core Concepts

| Term | Definition |
|---|---|
| **Image** | Read-only blueprint: OS, runtime, dependencies, code |
| **Container** | Running instance of an image |
| **Dockerfile** | Instructions to build an image; each instruction = a layer |
| **Registry** | Storage for images (Docker Hub, ECR, etc.) |

Multiple containers can run from the same image — they share the image layers, not copies of them. Each container gets its own thin writable layer on top.

**Images are immutable** — running 100 containers from the same image does not copy the image 100 times.

### Dockerfile

```dockerfile
FROM python:3.11-slim          # base image

ENV PYTHONDONTWRITEBYTECODE=1  # environment variable

WORKDIR /app                   # all subsequent commands run from here

COPY requirements.txt .        # copy just requirements first (cache trick)
RUN pip install -r requirements.txt

COPY . .                       # copy source code (invalidates cache if changed)

EXPOSE 8080                    # documents the port — does NOT publish it

CMD ["python", "app.py"]       # default command when container starts

VOLUME /app/data               # declares a mount point
```

### Build Cache — Most Important Performance Concept

Docker caches each layer. If a layer and all layers before it are unchanged, it reuses the cache — huge time savings. **If a layer changes, all layers after it are invalidated.**

**Order matters:**
```
COPY requirements.txt .      ← changes rarely
RUN pip install              ← cached unless requirements.txt changes
COPY . .                     ← changes on every code edit
```
Always `COPY` requirements → `RUN install` → `COPY source`. If you `COPY . .` first, every code change invalidates the install layer.

### Multi-stage Builds — Slim Production Images

Separate the build environment from the runtime image. The final image contains only what's needed to run:

```dockerfile
# Stage 1: build
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Stage 2: runtime
FROM alpine:3.18
COPY --from=builder /app/myapp /myapp
CMD ["/myapp"]
```

The final image has only the binary — no Go compiler, no source code, no build tools. Smaller images pull faster when spinning up EC2 test environments.

### .dockerignore

Without this, `COPY . .` copies git history, test files, and local secrets into the image:

```
.git
__pycache__
*.pyc
.env
tests/
*.log
```

### Build & Images

```bash
docker build -t myapp:1.0 .                   # build from Dockerfile in current dir
docker build -f path/to/Dockerfile -t myapp . # specify Dockerfile path
docker images                                  # list images
docker rmi myapp:1.0                           # remove image
```

### Run Containers

```bash
docker run myapp:1.0                                  # basic run
docker run -d -p 8080:8080 --name myapp myapp:1.0     # detached, port mapping host:container
docker run -d -p 8080:8080 -v data:/app/data myapp    # with named volume
docker run -it ubuntu bash                            # interactive shell
docker run --rm myapp:1.0                             # remove container when it exits
```

Flag summary: `-d` background, `-p host:container` port map, `--name` name the container, `-v` volume, `-it` interactive+tty, `--rm` auto-remove.

> **EXPOSE is documentation only** — it does nothing without `-p` at `docker run`. A very common interview misconception.

### Manage Containers

```bash
docker ps                    # running containers
docker ps -a                 # all containers (including stopped)
docker stop <cont>           # SIGTERM, then SIGKILL (graceful)
docker kill <cont>           # SIGKILL immediately
docker rm <cont>             # remove stopped container
docker rm -f <cont>          # stop + remove
docker container prune       # remove all stopped containers
docker exec <cont> ls /app   # run command in running container
docker exec -it <cont> bash  # interactive shell in running container
```

`docker start` vs `docker run`: `start` resumes an existing stopped container; `run` creates and starts a new one.

### Volumes

```bash
# Named volume (managed by Docker — best for production data persistence)
docker run -v mydata:/app/data myapp

# Bind mount (you specify exact host path — best for development, live code changes)
docker run -v /home/shash/code:/app myapp
```

Bind mounts let code edits reflect instantly without rebuilding — used in dev workflows.

### Registry

```bash
docker tag myapp:1.0 username/myapp:1.0
docker push username/myapp:1.0
docker pull username/myapp:1.0
```

### Container vs VM

| | Container | VM |
|---|---|---|
| Kernel | Shares host OS kernel | Own kernel |
| Startup | Seconds | Minutes |
| Size | MBs | GBs |
| Isolation | Process-level | Full hardware |
| Use case | Microservices, CI/CD | Strong isolation needed |

Docker virtualizes only the application layer and shares the host OS kernel. VMs virtualize the entire OS including the kernel.

### Key Interview Takeaways

- **Container vs VM** — containers share host kernel, faster/lighter; VMs have own kernel, stronger isolation.
- **Layers and cache order** — `COPY requirements → RUN install → COPY source`. Interviewers love asking why order matters.
- **Volumes vs bind mounts** — named volumes for production persistence, bind mounts for dev.
- **Images are immutable** — each container gets its own writable layer on top; 100 containers don't copy the image 100 times.
- **EXPOSE is documentation only** — need `-p` at runtime to actually publish.
- **Multi-stage builds** — keep production images lean by separating build from runtime.
- **Build cache** — if a layer changes, all subsequent layers are invalidated.

---

## Kubernetes

Container orchestration platform — start/stop/restart containers, distribute them across nodes, scale up/down, route traffic.

### Architecture

#### Control Plane

| Component | Role |
|---|---|
| **kube-apiserver** | Single entry point — every `kubectl` command and every cluster action goes through this. The only component that reads/writes to etcd. |
| **etcd** | Key-value store that holds all cluster state. The source of truth. |
| **kube-scheduler** | Watches for newly created pods with no assigned node, assigns them to a node. |
| **kube-controller-manager** | Runs control loops that continuously reconcile desired state vs actual state (e.g. ReplicaSet controller ensures correct pod count). |

#### Worker Node

| Component | Role |
|---|---|
| **kubelet** | Agent on each node. Talks to the API server. Ensures containers described in pod specs are running. |
| **kube-proxy** | Handles network routing to pods. Implements Service IPs. |
| **container runtime** | Actually runs containers (containerd, CRI-O, etc.). |

### Pods — Smallest Unit

A pod wraps one or more containers that share:
- The same **network namespace** — same IP address, communicate via `localhost`
- The same **storage** — can share volumes
- The same **lifecycle** — start and stop together

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  containers:
    - name: app
      image: myapp:1.0
      ports:
        - containerPort: 8080
      env:
        - name: ENV
          value: "production"
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"
```

Each pod gets **one IP**. When a pod dies, it's gone — a new one is created with a new IP. **You almost never create pods directly** — you create Deployments.

### Deployment

```
Deployment  →  manages  →  ReplicaSet  →  manages  →  Pod  Pod  Pod
```

Deployments give you: self-healing (restart failed pods), rolling updates, rollback.

```bash
kubectl apply -f deployment.yaml    # create or update (declarative, idempotent)
kubectl scale deployment my-app --replicas=5
```

### Services

Pods get new IPs on every restart. Services provide a **stable DNS name and IP** that doesn't change. Services find their pods using **label selectors**.

| Type | Access |
|---|---|
| **ClusterIP** (default) | Stable virtual IP inside the cluster only |
| **NodePort** | Exposes on a static port (30000–32767) on every node's IP — reachable from outside |
| **LoadBalancer** | Creates a cloud provider LB (AWS ELB, GCP LB, Azure LB) with a public IP — production way to expose externally |

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:
    app: my-app          # routes to pods with this label
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

The `selector` on the Service must match the `labels` on the pod template exactly.

### Labels

The mechanism everything uses to find everything else:
- Services find pods via labels
- Deployments manage pods via labels
- Monitoring tools discover pods via labels

### Namespaces

Divide a single cluster into virtual sub-clusters — different namespaces for different teams or environments (dev/staging/prod) on the same cluster.

```bash
kubectl get namespaces
kubectl get pods -n my-namespace
```

### ConfigMaps and Secrets

Decouple configuration from the container image. ConfigMaps for non-sensitive config, Secrets for passwords/tokens (base64 encoded, not encrypted by default — use RBAC to restrict access).

### kubectl Reference

```bash
kubectl get pods                          # list pods
kubectl get pods -o wide                  # more detail (node, IP)
kubectl describe pod <pod-name>           # full event log and spec
kubectl logs <pod-name>                   # stdout of container
kubectl logs <pod-name> -f                # follow (tail -f style)
kubectl exec -it <pod-name> -- bash       # interactive shell
kubectl apply -f manifest.yaml            # create/update
kubectl delete -f manifest.yaml           # delete
kubectl scale deployment my-app --replicas=5
kubectl rollout status deployment/my-app  # watch rolling update
kubectl rollout undo deployment/my-app    # rollback
kubectl get namespaces
```

### Desired State vs Actual State — The Core Concept

You declare what you want. Kubernetes' **control loops** continuously compare actual state to desired state and take action to close the gap. This is why:
- Deleted pods are immediately recreated by the Deployment
- Scaled-down nodes are drained
- Failed containers are restarted

### Key Interview Takeaways

- **Pod vs Container** — Pod is Kubernetes' unit, not the container. A pod wraps one or more containers and gives them a shared network/storage identity. Almost always one container per pod.
- **Deployment vs Pod** — never create pods directly in production. Deployments provide self-healing, rolling updates, and rollback. Delete a pod managed by a Deployment and it immediately creates a new one.
- **Service necessity** — pods get new IPs on every restart. Services provide stable DNS and load balancing. Everything talks to Services, not pods directly.
- **Labels are everything** — the selector on a Service must match the labels on the pod template exactly.
- **`kubectl apply` always** — declarative, idempotent, works for both create and update. Use in all scripts and CI pipelines.
- **Desired state vs actual state** — the core K8s concept. Declare what you want; control loops reconcile continuously.

## Project Overview – Application (Kind + Kustomize)

Repo `app/` ini berfokus ke aplikasi utama:
- Backend (`galihhhp/elysia-backend:latest`)
- Frontend (`galihhhp/react-frontend:latest`)
- PostgreSQL (`postgres:17`)

Semua resources Kubernetes untuk app ada langsung di folder `app/`, bukan di root repo.

## Phase 1: Setup Kind Cluster

### Task 1.1: Install Prerequisites

**Checklist:**
- [ ] Install kind: `brew install kind` (macOS) atau dari https://kind.sigs.k8s.io
- [ ] Install kubectl: `brew install kubectl` atau dari https://kubernetes.io/docs/tasks/tools/
- [ ] Verify Docker is running
- [ ] Test commands: `kind version`, `kubectl version --client`

**Learning:**
- ✅ kind installation
- ✅ kubectl setup
- ✅ Prerequisites verification

---

### Task 1.2: Create Kind Cluster Configuration
**File:** `app/kind-config.yaml`

**Requirements:**
- Create kind cluster config file
- Configure: 1 control-plane node, 2 worker nodes
- Port mappings: 80:80 (HTTP), 443:443 (HTTPS)

- API server port: 6443
- Optional: Local registry on port 5000
- Configure networking

**Learning:**
- ✅ kind configuration
- ✅ Cluster architecture (control-plane + workers)
- ✅ Port mapping
- ✅ Local registry setup (optional)

---

## Phase 2: Tier 1 - Fundamental Concepts

### Task 2.0: Kustomize Setup (Optional but Recommended)
**Files:**
- `app/base/kustomization.yaml`
- `app/env/dev/kustomization.yaml`
- `app/env/prod/kustomization.yaml`

**Requirements:**
- Understand Kustomize concept (base + overlay pattern)
- Create base kustomization.yaml that references all base resources
- Create environment-specific kustomizations that reference base
- Learn how to deploy to different environments: `kubectl apply -k app/env/dev`

**Learning:**
- ✅ Kustomize for multi-environment deployments
- ✅ Base resources vs environment-specific overrides
- ✅ DRY (Don't Repeat Yourself) principle in Kubernetes
- ✅ Production-ready deployment patterns

---

### Task 2.1: Namespace
**File:** `app/env/dev/namespace.yaml`

**Requirements:**
- Create namespace (e.g., `development` for dev environment)
- Add labels: name, environment
- Understand namespace isolation concept
- Understand Kustomize structure (base + env)

**Learning:**
- ✅ Namespace isolation
- ✅ Resource organization
- ✅ Multi-tenant environments
- ✅ YAML structure
- ✅ Environment-specific namespaces (dev, prod)

---

### Task 2.2: ConfigMap
**File:** 
- `app/env/dev/configmap.yaml`

**Requirements:**

**ConfigMap:**
- App environment variables
- Log level, API timeout
- Frontend & backend URLs
- Understand data vs stringData
- Environment-specific configuration (dev vs prod)
- Understand Kustomize pattern for environment-specific configs

**Learning:**
- ✅ ConfigMap for non-sensitive config
- ✅ Environment variable injection
- ✅ Kustomize for environment-specific configurations
- ✅ Base vs environment overlay pattern

---

### Task 2.3: Deployment & Labels
**Files:**
- `app/base/backend/deployment.yaml`
- `app/base/frontend/deployment.yaml`

**Requirements:**

**Backend Deployment:**
- Image: ``galihhhp/elysia-backend:latest`
- 2 replicas
- Port: 8080
- Environment from ConfigMap & Secrets
- Labels: app=backend, tier=api
- Understand Pod template and labels

**Frontend Deployment:**
- Image: `galihhhp/react-frontend:latest`
- 2 replicas
- Port: 80
- Environment: API URL from ConfigMap
- Labels: app=frontend, tier=ui

**Label & Selector Practice:**
- Add multiple labels to Deployment Pod template
- Use label selectors to filter Pods: `kubectl get pods -l app=backend`
- Understand label vs selector relationship
- Understand how Service uses label selectors

**Learning:**
- ✅ Deployment as workload controller
- ✅ Replica management
- ✅ Rolling updates concept
- ✅ Pod template specification
- ✅ Resource tagging with labels
- ✅ Label selectors
- ✅ Service discovery mechanism (via labels)
- ✅ Core K8s pattern

---

### Task 2.4: Service
**Files:**
- `app/base/backend/service.yaml`
- `app/base/frontend/service.yaml`

**Requirements:**

**Backend Service:**
- Type: ClusterIP
- Port: 8080
- Selector: app=backend
- Understand service discovery via label selectors

**Frontend Service:**
- Type: NodePort
- Port: 80
- NodePort: 30080 (accessible via localhost:30080)
- Selector: app=frontend

**Learning:**
- ✅ Service types (ClusterIP vs NodePort)
- ✅ Service discovery via label selectors
- ✅ Load balancing
- ✅ Pod-to-Pod communication
- ✅ External access patterns
- ✅ How Service finds Pods using label selectors

---

### Task 2.5: PostgreSQL Deployment & Service
**Files:**
- `app/base/postgresql/deployment.yaml`
- `app/base/postgresql/service.yaml`

**Requirements:**

**Deployment:**
- Image: `postgres:17`
- 1 replica
- Environment variables: DB name, user, password (from Secret)
- Labels: app=postgres, tier=database
- Port: 5432

**Service:**
- Type: ClusterIP
- Port: 5432
- Selector: app=postgres

**Learning:**
- ✅ Apply all Tier 1 concepts together
- ✅ Database deployment pattern
- ✅ Service for database access

---

## Phase 3: Testing & Validation

### Task 3.1: Manual Testing

**Test Scenarios:**
- [ ] Cluster is running: `kubectl get nodes`
- [ ] All pods are running: `kubectl get pods -n development` (or `-n production`)
- [ ] Services are accessible
- [ ] Frontend accessible via NodePort: `curl http://localhost:30080`
- [ ] Backend health check: `kubectl exec -it <pod> -n development -- curl http://localhost:8080/health`
- [ ] Database connection works
- [ ] ConfigMap values are injected (check environment-specific values)
- [ ] Kustomize deployment works correctly

**Learning:**
- ✅ Manual testing procedures
- ✅ kubectl commands
- ✅ Service connectivity
- ✅ Debugging techniques

---

### Task 3.2: Cleanup & Re-deploy

**Test:**
- [ ] Delete all resources manually - verify all resources deleted
- [ ] Reapply manifests again - verify clean deployment
- [ ] Test rollback scenario (delete one deployment, reapply)

**Learning:**
- ✅ Idempotent operations
- ✅ Cleanup procedures
- ✅ Re-deployment patterns

---

## Bonus Tasks - Tier 2 & 3 (Optional)

**Note:** Fokus dulu ke Tier 1 (Phase 2) sampai benar-benar paham. Tier 2 & 3 bisa dipelajari setelah Tier 1 dikuasai.

---

### Bonus 1: Tier 2 - Probe (Health Checks)
**Files:**
- Update: `app/base/backend/deployment.yaml`
- Update: `app/base/frontend/deployment.yaml`

**Requirements:**
- Add liveness probe to backend: `/health` endpoint
- Add readiness probe to backend: `/ready` endpoint
- Add startup probe (optional)
- Understand difference: liveness vs readiness vs startup
- Test probe failures

**Learning:**
- ✅ Health checks concept
- ✅ Liveness vs Readiness vs Startup
- ✅ Probe configuration
- ✅ Reliability & zero-downtime

---

### Bonus 2: Tier 2 - Resource Management
**Files:**
- Update: `app/base/backend/deployment.yaml`
- Update: `app/base/frontend/deployment.yaml`
- Update: `app/base/postgresql/deployment.yaml`

**Requirements:**
- Add resource requests and limits
- Backend: CPU 500m, Memory 512Mi
- Frontend: CPU 200m, Memory 256Mi
- PostgreSQL: CPU 500m, Memory 1Gi
- Understand requests vs limits

**Learning:**
- ✅ Resource requests & limits
- ✅ CPU & Memory management
- ✅ Resource efficiency
- ✅ Cluster stability

---

### Bonus 3: Tier 2 - Ingress
**File:** `app/base/ingress.yaml`

**Prerequisites:**
- [ ] kind cluster sudah running
- [ ] Install Ingress controller (Nginx Ingress Controller recommended untuk kind)
- [ ] Verify Ingress controller pods running: `kubectl get pods -n ingress-nginx`
- [ ] Verify Ingress controller service: `kubectl get svc -n ingress-nginx`

**Installation Commands:**
```bash
# Install Nginx Ingress Controller untuk kind
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

# Wait for ingress controller to be ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s

# Verify installation
kubectl get pods -n ingress-nginx
```

**Requirements:**
- Install Ingress controller (Nginx or Traefik)
- Create Ingress resource
- Route traffic: `/api` → backend, `/` → frontend
- Test via `http://localhost`
- Understand path-based routing

**Learning:**
- ✅ Ingress controllers
- ✅ HTTP/HTTPS routing
- ✅ Path-based routing
- ✅ SSL/TLS termination (optional)

---

### Bonus 4: Tier 3 - Persistent Volumes
**Files:**
- `app/base/postgresql/pvc.yaml`

**Requirements:**
- Create PersistentVolumeClaim for PostgreSQL
- Update deployment to use PVC
- Understand PV vs PVC concept
- Test data persistence after pod restart

**Learning:**
- ✅ Persistent volumes
- ✅ PV vs PVC
- ✅ Data persistence
- ✅ Storage classes

---

### Bonus 5: Tier 3 - StatefulSet
**Files:**
- `app/base/postgresql/statefulset.yaml` (optional replacement)

**Requirements:**
- Convert PostgreSQL from Deployment to StatefulSet
- Understand ordered deployment
- Understand persistent identity
- Create Headless Service

**Learning:**
- ✅ StatefulSet concept
- ✅ Stateful vs stateless
- ✅ Ordered deployment
- ✅ Persistent identity

---

### Bonus 6: Tier 3 - DaemonSet
**File:** `app/base/daemonset-logging.yaml`

**Requirements:**
- Create DaemonSet for logging agent
- Understand node-level pods
- Test DaemonSet on all nodes

**Learning:**
- ✅ DaemonSet concept
- ✅ Node-level pods
- ✅ Use cases (logging, monitoring)

---

### Bonus 7: Tier 3 - Jobs & CronJobs
**Files:**
- `app/base/job-backup.yaml`
- `app/base/cronjob-cleanup.yaml`

**Requirements:**
- Create Job for one-time task (e.g., database backup)
- Create CronJob for scheduled task (e.g., cleanup)
- Understand job completion vs failure

**Learning:**
- ✅ Jobs concept
- ✅ CronJobs concept
- ✅ One-time vs scheduled tasks
- ✅ Batch processing patterns

---

### Bonus 8: Tier 3 - Horizontal Pod Autoscaler
**File:** `app/base/hpa.yaml`

**Prerequisites:**
- [ ] kind cluster sudah running
- [ ] Install metrics-server (tidak built-in di kind)
- [ ] Verify metrics-server pods running: `kubectl get pods -n kube-system | grep metrics-server`
- [ ] Test metrics API: `kubectl top nodes` (harus return metrics, bukan error)

**Installation Commands:**
```bash
# Install metrics-server untuk kind
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Patch metrics-server untuk kind (disable TLS verification)
kubectl patch deployment metrics-server -n kube-system --type='json' \
  -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'

# Wait for metrics-server to be ready
kubectl wait --namespace kube-system \
  --for=condition=ready pod \
  --selector=k8s-app=metrics-server \
  --timeout=90s

# Verify installation
kubectl top nodes
kubectl top pods -n development
```

**Requirements:**
- Install metrics-server
- Create HPA for backend deployment
- Configure min/max replicas
- Test auto-scaling with load

**Learning:**
- ✅ Auto-scaling
- ✅ Metrics server
- ✅ HPA configuration
- ✅ CPU/Memory threshold

---

## Project Structure

```
kind-project/
├── app/
│   ├── kind-config.yaml
│   ├── base/
│   │   ├── kustomization.yaml
│   │   ├── backend/
│   │   │   ├── deployment.yaml
│   │   │   └── service.yaml
│   │   ├── frontend/
│   │   │   ├── deployment.yaml
│   │   │   └── service.yaml
│   │   └── postgresql/
│   │       ├── deployment.yaml
│   │       └── service.yaml
│   └── env/
│       ├── dev/
│       │   ├── kustomization.yaml
│       │   ├── namespace.yaml
│       │   └── configmap.yaml
│       └── prod/
│           ├── kustomization.yaml
│           ├── namespace.yaml
│           └── configmap.yaml
```

---

## Learning Outcomes

After completing **Tier 1 (Phase 2)**, you'll understand:

**Fundamental Concepts (MUST KNOW):**
- ✅ kind cluster setup and configuration
- ✅ **Namespace** - Resource isolation and organization
- ✅ **ConfigMap** - Configuration management
- ✅ **Deployment** - Workload controller, rolling updates, replicas, Pod template
- ✅ **Label & Selector** - Resource tagging and service discovery (learned via Deployment & Service)
- ✅ **Service** - Pod communication, load balancing, service types, label selectors
- ✅ **Pod** - Understanding Pods created by Deployment (not manual creation)
- ✅ **Kustomize** - Multi-environment deployment pattern (base + overlay)

**Operations:**
- ✅ kubectl commands mastery
- ✅ Health monitoring and debugging

**After Bonus Tasks (Tier 2 & 3 - Optional):**
- ✅ Health checks (liveness & readiness probes)
- ✅ Resource limits and requests
- ✅ Ingress for HTTP routing
- ✅ Persistent Volumes for data persistence
- ✅ StatefulSet for stateful applications
- ✅ Advanced workload controllers (DaemonSet, Jobs, CronJobs)
- ✅ Horizontal Pod Autoscaler

---

## Expected Deliverables

**Setelah menyelesaikan Tier 1 (Phase 2)**, kamu akan memiliki:

### 1. Infrastructure yang Running
- ✅ kind cluster dengan 1 control-plane + 2 worker nodes
- ✅ Namespace `development` (dev) atau `production` (prod) dengan semua resources terorganisir
- ✅ 3 deployments running (PostgreSQL, Backend, Frontend)
- ✅ 3 services exposed (PostgreSQL ClusterIP, Backend ClusterIP, Frontend NodePort)
- ✅ ConfigMap terkonfigurasi dengan benar (environment-specific)
- ✅ Pods dengan labels yang proper
- ✅ Kustomize structure untuk multi-environment deployment

### 2. Akses ke Aplikasi
- ✅ Frontend accessible via: `http://localhost:30080`
- ✅ Backend API accessible via: `backend-service:8080` (internal cluster)
- ✅ PostgreSQL accessible via: `postgres-service:5432` (internal cluster)
- ✅ Health checks working untuk semua services

### 3. Skills yang Dikuasai (Tier 1)
- ✅ Bisa setup kind cluster dari scratch
- ✅ Bisa create Namespace untuk resource isolation
- ✅ Bisa manage ConfigMaps untuk configuration
- ✅ Bisa create Deployment untuk workload management (dengan labels)
- ✅ Paham Label & Selector untuk resource organization dan service discovery
- ✅ Bisa create Service untuk pod communication (menggunakan label selectors)
- ✅ Paham bagaimana Deployment membuat Pods secara otomatis
- ✅ Bisa create Kubernetes manifests (YAML)
- ✅ Bisa deploy multi-tier application
- ✅ Bisa menggunakan Kustomize untuk multi-environment deployments
- ✅ Paham base + overlay pattern (base resources + environment-specific configs)
- ✅ Bisa debug Kubernetes issues dengan kubectl
- ✅ Bisa monitor cluster health

### 4. Working Commands
Setelah selesai, kamu bisa jalankan:
```bash
# Setup cluster
kind create cluster --config app/kind-config.yaml

# Install kustomize (if not already installed)
# macOS: brew install kustomize
# Or download from: https://kubectl.docs.kubernetes.io/installation/kustomize/

# Deploy to dev environment using Kustomize
kubectl apply -k app/env/dev

# Or build and apply manually
kubectl kustomize app/env/dev | kubectl apply -f -

# Check status
kubectl get all -n development

# Access application
curl http://localhost:30080

# View logs
kubectl logs -f deployment/backend -n development

# Deploy to prod environment
kubectl apply -k app/env/prod

# Cleanup
kubectl delete namespace development
# or
kubectl delete namespace production
```

### 5. Production-Ready Patterns (Tier 1)
Setelah selesai Tier 1, kamu akan punya:
- ✅ Multi-replica deployments (high availability)
- ✅ Service discovery (ClusterIP services)
- ✅ External access (NodePort)
- ✅ Configuration management (ConfigMap)
- ✅ Secrets management (Secret)
- ✅ Namespace isolation
- ✅ Proper labeling for resource organization

**Tier 2 & 3 (Bonus) akan menambahkan:**
- ✅ Resource limits & requests
- ✅ Health checks (liveness & readiness)
- ✅ Ingress for HTTP routing
- ✅ Persistent Volumes
- ✅ Advanced workload controllers

### 6. Next Steps (Setelah Tier 1 Selesai)

**Immediate Next Steps:**
- ✅ Complete Bonus Tasks (Tier 2 & 3) untuk advanced concepts
- ✅ Practice dengan kubectl commands
- ✅ Experiment dengan scaling deployments
- ✅ Test rolling updates

**After Mastering Tier 1 & 2:**
- ✅ Deploy ke cloud (GKE, EKS, AKS)
- ✅ Add monitoring (Prometheus + Grafana)
- ✅ Setup CI/CD pipelines
- ✅ Add service mesh (Istio, Linkerd)
- ✅ Learn Helm for package management

---

## Quick Start Commands

```bash
# Setup cluster
kind create cluster --config app/kind-config.yaml

# Deploy to dev environment using Kustomize
kubectl apply -k app/env/dev

# Check status
kubectl get all -n development

# Access frontend
open http://localhost:30080

# Cleanup
kubectl delete namespace development
```

---

## Integration with Observability Stack

Observability stack untuk project ini didesain di `../obervability/README.md`.

Kontrak yang dipakai observability terhadap app:
- Namespace aplikasi: `development`
- Labels utama:
  - Backend: `app=backend`, `tier=api`
  - Frontend: `app=frontend`, `tier=ui`
  - PostgreSQL: `app=postgres`, `tier=database`
- Service names:
  - Backend: `backend-service` (ClusterIP, port 8080)
  - Frontend: `frontend-service` (NodePort 30080 → 80)
  - PostgreSQL: `postgres-service` (ClusterIP, port 5432)

Pastikan kamu mempertahankan pola namespace, labels, dan nama Service ini ketika menghubungkan ke manifests observability yang akan kamu buat mengikuti `obervability/README.md`.


---
## Project Overview – Application (Kind + Kustomize)

Repo `app/` ini berfokus ke aplikasi utama:
- Backend (`galihhhp/elysia-backend:1.6`)
- Frontend (`galihhhp/react-frontend:1.9`)
- PostgreSQL (`postgres:17`)

📐 **Architecture & Structure**: Lihat [`../ARCHITECTURE.md`](../ARCHITECTURE.md) untuk visual diagrams.

---

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
- Port mappings: 80:80 (HTTP), 443:443 (HTTPS), 30080, 30090

**Learning:**
- ✅ kind configuration
- ✅ Cluster architecture (control-plane + workers)
- ✅ Port mapping

---

## Phase 2: Tier 1 - Fundamental Concepts

### Task 2.0: Kustomize Setup
**Files:**
- `app/base/kustomization.yaml`
- `app/env/dev/kustomization.yaml`
- `app/env/prod/kustomization.yaml`

**Requirements:**
- Understand Kustomize concept (base + overlay pattern)
- Create base kustomization.yaml that references all base resources
- Create environment-specific kustomizations that reference base
- Deploy to different environments: `kubectl apply -k app/env/dev`

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

**Learning:**
- ✅ Namespace isolation
- ✅ Resource organization
- ✅ Multi-tenant environments
- ✅ Environment-specific namespaces (dev, prod)

---

### Task 2.2: ConfigMap
**File:** `app/env/dev/configmap.yaml`

**Requirements:**
- App environment variables
- Database connection config
- Feature flags
- Environment-specific configuration (dev vs prod)

**Learning:**
- ✅ ConfigMap for non-sensitive config
- ✅ Environment variable injection
- ✅ Kustomize for environment-specific configurations

---

### Task 2.3: Secret
**File:** `app/base/postgresql/secret.yaml`

**Requirements:**
- Database credentials (username, password)
- Understand Secret vs ConfigMap
- Reference Secret in Deployment via secretRef

**Learning:**
- ✅ Secret for sensitive data
- ✅ Secret types (Opaque)
- ✅ Environment variable injection from Secret

---

### Task 2.4: Deployment & Labels
**Files:**
- `app/base/backend/deployment.yaml`
- `app/base/frontend/deployment.yaml`

**Implementation:**

**Backend Deployment:**
- Image: `galihhhp/elysia-backend:1.6`
- Port: 3000
- Environment from ConfigMap & Secret
- Labels: `app=backend`, `tier=api`
- Resource limits & requests
- Liveness & readiness probes
- Security context (runAsNonRoot)

**Frontend Deployment:**
- Image: `galihhhp/react-frontend:1.9`
- Port: 80
- Environment from ConfigMap
- Labels: `app=frontend`, `tier=ui`
- Resource limits & requests
- Liveness & readiness probes
- Security context (runAsNonRoot)

**Learning:**
- ✅ Deployment as workload controller
- ✅ Pod template specification
- ✅ Resource tagging with labels
- ✅ Label selectors
- ✅ Health probes
- ✅ Resource management
- ✅ Security context

---

### Task 2.5: Service
**Files:**
- `app/base/backend/service.yaml`
- `app/base/frontend/service.yaml`

**Implementation:**

**Backend Service:**
- Type: ClusterIP
- Port: 3000
- Selector: `app=backend`

**Frontend Service:**
- Type: ClusterIP
- Port: 80
- Selector: `app=frontend`

**Learning:**
- ✅ Service types (ClusterIP)
- ✅ Service discovery via label selectors
- ✅ Load balancing
- ✅ Pod-to-Pod communication

---

### Task 2.6: PostgreSQL
**Files:**
- `app/base/postgresql/deployment.yaml`
- `app/base/postgresql/service.yaml`
- `app/base/postgresql/secret.yaml`
- `app/base/postgresql/pvc.yaml`

**Implementation:**

**Deployment:**
- Image: `postgres:17`
- 1 replica
- Environment from ConfigMap & Secret
- Labels: `app=postgres`, `tier=database`
- Port: 5432
- PVC mount for data persistence
- Init script via ConfigMap

**Service:**
- Type: ClusterIP
- Port: 5432
- Selector: `app=postgres`

**Learning:**
- ✅ Database deployment pattern
- ✅ PersistentVolumeClaim
- ✅ Init script via ConfigMap
- ✅ Service for database access

---

## Phase 3: Testing & Validation

### Task 3.1: Manual Testing

**Test Scenarios:**
- [ ] Cluster is running: `kubectl get nodes`
- [ ] All pods are running: `kubectl get pods -n development`
- [ ] Services are accessible: `kubectl get svc -n development`
- [ ] Frontend accessible via port-forward
- [ ] Backend health check working
- [ ] Database connection works
- [ ] ConfigMap values are injected

**Commands:**
```bash
kubectl get nodes

kubectl get pods -n development

kubectl port-forward svc/frontend-service 80:80 -n development

kubectl port-forward svc/backend-service 3000:3000 -n development

kubectl logs -f deployment/task-deployment -n development
```

**Learning:**
- ✅ Manual testing procedures
- ✅ kubectl commands
- ✅ Service connectivity
- ✅ Debugging techniques

---

### Task 3.2: Cleanup & Re-deploy

**Test:**
- [ ] Delete namespace - verify all resources deleted
- [ ] Reapply manifests - verify clean deployment

**Commands:**
```bash
kubectl delete namespace development

kubectl apply -k app/env/dev

kubectl get all -n development
```

**Learning:**
- ✅ Idempotent operations
- ✅ Cleanup procedures
- ✅ Re-deployment patterns

---

## Learning Outcomes

**Fundamental Concepts:**
- ✅ kind cluster setup and configuration
- ✅ **Namespace** - Resource isolation and organization
- ✅ **ConfigMap** - Configuration management
- ✅ **Secret** - Sensitive data management
- ✅ **Deployment** - Workload controller, rolling updates, replicas, Pod template
- ✅ **Label & Selector** - Resource tagging and service discovery
- ✅ **Service** - Pod communication, load balancing, service types
- ✅ **PersistentVolumeClaim** - Data persistence
- ✅ **Kustomize** - Multi-environment deployment pattern (base + overlay)

**Production Patterns:**
- ✅ Health checks (liveness & readiness probes)
- ✅ Resource limits and requests
- ✅ Security context (runAsNonRoot)
- ✅ Secrets for credentials

---

## Expected Deliverables

### 1. Infrastructure Running
- ✅ kind cluster dengan 1 control-plane + 2 worker nodes
- ✅ Namespace `development` dengan semua resources
- ✅ 3 deployments running (PostgreSQL, Backend, Frontend)
- ✅ 3 services exposed
- ✅ ConfigMap & Secret configured
- ✅ PVC untuk PostgreSQL

### 2. Access
- ✅ Frontend via port-forward: `localhost:80`
- ✅ Backend via port-forward: `localhost:3000`
- ✅ PostgreSQL via port-forward: `localhost:5432`

### 3. Skills
- ✅ Setup kind cluster dari scratch
- ✅ Create Namespace, ConfigMap, Secret
- ✅ Create Deployment dengan probes, resources, security context
- ✅ Create Service untuk pod communication
- ✅ Use Kustomize untuk multi-environment deployments
- ✅ Debug Kubernetes issues dengan kubectl

---

## Next Steps

- ⬜ Add Ingress for HTTP routing
- ⬜ Add HPA (Horizontal Pod Autoscaler)
- ⬜ Add NetworkPolicy for security
- ⬜ Convert PostgreSQL to StatefulSet
- ⬜ Add CI/CD pipeline (GitHub Actions)

## DevOps Kind Project – App + Observability

Repo ini menggabungkan dua fokus utama:
- `app/`: Kind cluster + Kustomize untuk aplikasi (backend, frontend, PostgreSQL)
- `observability/`: Full observability stack (Prometheus, Grafana, kube-state-metrics, node-exporter)

📐 **Architecture diagrams**: Lihat [`ARCHITECTURE.md`](ARCHITECTURE.md) untuk visual lengkap.

---

## 1. Quick Start

### Option A: Automated Setup

```bash
./scripts/setup.sh
```

Script ini akan:
1. Check prerequisites (Docker, kind, kubectl)
2. Create Kind cluster
3. Deploy aplikasi ke namespace `development`
4. Deploy observability stack ke namespace `observability`

### Option B: Manual Setup

```bash
kind create cluster --config app/kind-config.yaml

kubectl apply -k app/env/dev

kubectl apply -k observability/env/dev

kubectl get pods -n development
kubectl get pods -n observability
```

---

## 2. Akses Aplikasi & Tools

| Service | Access Method |
|---------|---------------|
| Frontend | `kubectl port-forward svc/frontend-service 80:80 -n development` → http://localhost |
| Backend | `kubectl port-forward svc/backend-service 3000:3000 -n development` |
| PostgreSQL | `kubectl port-forward svc/postgres-service 5432:5432 -n development` |
| Prometheus | `kubectl port-forward svc/prometheus 9090:9090 -n observability` → http://localhost:9090 |
| Grafana | `kubectl port-forward svc/grafana 3000:3000 -n observability` → http://localhost:3000 |

---

## 3. Cleanup

### Option A: Automated Cleanup

```bash
./scripts/cleanup.sh

./scripts/cleanup.sh -f

./scripts/cleanup.sh -f -d
```

### Option B: Manual Cleanup

```bash
kubectl delete namespace development
kubectl delete namespace observability

kind delete cluster --name kind-app-cluster
```

---

## 4. Learning Path

1. **Tier 1 - Kubernetes Fundamentals** (`app/README.md`):
   - Namespace, ConfigMap, Secret
   - Deployment, Service
   - Labels & Selectors
   - Kustomize (base + overlay)

2. **Tier 2 - Production Patterns** (`app/README.md`):
   - Health probes (liveness, readiness)
   - Resource limits & requests
   - Security context
   - PersistentVolumeClaim

3. **Tier 3 - Observability** (`observability/README.md`):
   - Prometheus + scrape configs
   - Grafana dashboards
   - kube-state-metrics
   - node-exporter (DaemonSet)

---

## 5. Project Features

### Application Stack
- ✅ 3-tier architecture (Frontend → Backend → Database)
- ✅ Multi-environment support (dev/prod)
- ✅ ConfigMap for environment-specific config
- ✅ Secret for database credentials
- ✅ PersistentVolumeClaim for PostgreSQL data
- ✅ Health probes on all deployments
- ✅ Resource limits & requests
- ✅ Security context (runAsNonRoot)

### Observability Stack
- ✅ Prometheus (StatefulSet with PVC)
- ✅ Grafana (with PVC for persistence)
- ✅ kube-state-metrics (cluster-level metrics)
- ✅ node-exporter (DaemonSet for node metrics)
- ✅ RBAC (ServiceAccount, Role, ClusterRole)

### Automation
- ✅ `setup.sh` - One-command cluster + deployment
- ✅ `cleanup.sh` - Clean teardown with options

---

## 6. Documentation

| Document | Focus |
|----------|-------|
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | Visual diagrams: architecture, data flow, network, Kustomize structure |
| [`app/README.md`](app/README.md) | Kubernetes fundamentals learning guide |
| [`observability/README.md`](observability/README.md) | Observability stack learning guide |

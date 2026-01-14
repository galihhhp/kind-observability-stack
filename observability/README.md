## Project Overview – Observability Stack

Folder `observability/` ini berfokus ke observability stack untuk memonitor aplikasi di `app/`:

- Prometheus (metrics collection)
- Grafana (dashboards)
- kube-state-metrics (cluster metrics)
- node-exporter (node metrics)

📐 **Architecture & Data Flow**: Lihat [`../ARCHITECTURE.md`](../ARCHITECTURE.md) untuk visual diagrams.

---

## Prerequisites

- Cluster sudah dibuat dan aplikasi sudah running di namespace `development` (lihat `app/README.md`)
- Namespace `development` dengan pods backend, frontend, dan PostgreSQL sudah ada

---

## Phase 1: Metrics Stack

### Task 1.0: Observability Base Structure

**Files:**
- `observability/base/kustomization.yaml`
- `observability/env/dev/namespace.yaml`

**Implementation:**
- Namespace: `observability`
- Kustomization base mereferensikan semua komponen observability

**Learning:**
- ✅ Observability namespace
- ✅ Kustomize untuk stack observability

---

### Task 1.1: Prometheus

**Files:**
- `observability/base/prometheus/serviceaccount.yaml`
- `observability/base/prometheus/role.yaml`
- `observability/base/prometheus/rolebinding.yaml`
- `observability/base/prometheus/configmap.yaml`
- `observability/base/prometheus/service.yaml`
- `observability/base/prometheus/statefulset.yaml`

**Implementation:**
- StatefulSet dengan PVC untuk data persistence
- ConfigMap untuk scrape configuration
- RBAC untuk akses ke Kubernetes API
- Scrape targets: prometheus, node-exporter, kube-state-metrics, backend pods

**Learning:**
- ✅ Prometheus deployment pattern (StatefulSet)
- ✅ Scrape configuration via ConfigMap
- ✅ RBAC untuk Prometheus
- ✅ Kubernetes service discovery

---

### Task 1.2: kube-state-metrics

**Files:**
- `observability/base/kube-state-metrics/serviceaccount.yaml`
- `observability/base/kube-state-metrics/clusterrole.yaml`
- `observability/base/kube-state-metrics/clusterrolebinding.yaml`
- `observability/base/kube-state-metrics/deployment.yaml`
- `observability/base/kube-state-metrics/service.yaml`

**Implementation:**
- Deployment untuk kube-state-metrics
- ClusterRole untuk akses ke semua resources
- Service untuk expose metrics ke Prometheus

**Learning:**
- ✅ kube-state-metrics usage
- ✅ ClusterRole vs Role
- ✅ Kubernetes object metrics

---

### Task 1.3: node-exporter

**Files:**
- `observability/base/node-exporter/serviceaccount.yaml`
- `observability/base/node-exporter/clusterrole.yaml`
- `observability/base/node-exporter/clusterrolebinding.yaml`
- `observability/base/node-exporter/daemonset.yaml`
- `observability/base/node-exporter/service.yaml`

**Implementation:**
- DaemonSet agar berjalan di setiap node
- Host network dan host PID untuk akses node metrics
- Service untuk expose metrics ke Prometheus

**Learning:**
- ✅ DaemonSet pattern
- ✅ Node-level metrics collection
- ✅ Host namespace access

---

## Phase 2: Grafana Dashboards

### Task 2.0: Grafana Deployment

**Files:**
- `observability/base/grafana/deployment.yaml`
- `observability/base/grafana/service.yaml`
- `observability/base/grafana/pvc.yaml`
- `observability/base/grafana/secret.yaml`

**Implementation:**
- Deployment dengan PVC untuk dashboard persistence
- Secret untuk admin credentials
- Security context (runAsNonRoot)

**Learning:**
- ✅ Grafana deployment
- ✅ Secret untuk credentials
- ✅ PVC untuk persistence

---

### Task 2.1: Connect Prometheus as Data Source

**Steps:**
1. Port-forward Grafana: `kubectl port-forward svc/grafana 3000:3000 -n observability`
2. Login ke Grafana (lihat secret untuk credentials)
3. Add Data Source → Prometheus
4. URL: `http://prometheus:9090`
5. Save & Test

**Learning:**
- ✅ Grafana data source configuration
- ✅ Internal service DNS

---

### Task 2.2: Import Dashboards

**Recommended Dashboards:**
- Kubernetes Cluster (ID: 6417)
- Node Exporter Full (ID: 1860)
- Kubernetes Pods (ID: 6336)

**Steps:**
1. Grafana → Dashboards → Import
2. Enter dashboard ID
3. Select Prometheus data source
4. Import

**Learning:**
- ✅ Dashboard import
- ✅ Metrics visualization

---

## Phase 3: Validation

### Task 3.1: Verify Stack

**Checklist:**
- [ ] Pods running: `kubectl get pods -n observability`
- [ ] Services exposed: `kubectl get svc -n observability`
- [ ] Prometheus targets UP: http://localhost:9090/targets (via port-forward)
- [ ] Grafana accessible: http://localhost:3000 (via port-forward)
- [ ] Data source connected: Grafana → Configuration → Data Sources

---

### Task 3.2: Simulate Failures

**Scenarios:**
- Scale down backend ke 0 replicas dan lihat efeknya di metrics (Prometheus/Grafana)
- Paksa pod crash (image salah atau env salah) dan observasi di Grafana

**Commands:**
```bash
kubectl scale deployment/task-deployment --replicas=0 -n development

kubectl scale deployment/task-deployment --replicas=1 -n development
```

**Learning:**
- ✅ Practical troubleshooting dengan metrics
- ✅ Observability-driven debugging

---

## Expected Deliverables

### 1. Infrastructure Running
- ✅ Namespace `observability`
- ✅ Prometheus (StatefulSet with PVC)
- ✅ kube-state-metrics (Deployment)
- ✅ node-exporter (DaemonSet)
- ✅ Grafana (Deployment with PVC)

### 2. Access
- ✅ Prometheus UI via port-forward (localhost:9090)
- ✅ Grafana UI via port-forward (localhost:3000)
- ✅ All Prometheus targets UP

### 3. Skills
- ✅ Deploy observability stack dengan Kustomize
- ✅ Configure Prometheus scrape targets
- ✅ Setup Grafana data sources
- ✅ Import & create dashboards
- ✅ Debug issues menggunakan metrics

---

## Next Steps

- ⬜ Add Grafana datasource provisioning via ConfigMap
- ⬜ Add dashboard provisioning via ConfigMap
- ⬜ Add Alertmanager untuk alerting
- ⬜ Add HPA (Horizontal Pod Autoscaler) berbasis Prometheus metrics
- ⬜ Add Ingress untuk akses Grafana

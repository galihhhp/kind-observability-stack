## DevOps Kind Project – App + Observability Bridge

Repo ini menggabungkan dua fokus utama:
- `app/`: kind + Kustomize untuk aplikasi (backend, frontend, PostgreSQL)
- `obervability/`: desain dan rencana observability full stack (Prometheus, Alertmanager, Grafana, Loki, Promtail, kube-state-metrics, node-exporter)

Gunakan README ini sebagai jembatan workflow end-to-end.

---

## 1. Struktur Project

```text
kind-project/
├── app/
│   ├── kubernetes/
│   │   ├── kind-config.yaml
│   │   ├── base/
│   │   └── env/
│   └── README.md
└── obervability/
    └── README.md
```

- `app/README.md`: fokus ke Tier 1 Kubernetes fundamentals (namespace, ConfigMap, Deployment, Service, Kustomize, dsb).
- `obervability/README.md`: fokus ke observability stack di atas cluster dan aplikasi yang sudah jalan dari `app/`.

---

## 2. Urutan Workflow End-to-End

Semua perintah jalan dari root repo ini: `kind-project/`.

### Langkah 1 – Setup Cluster + Deploy Aplikasi (`app/`)

```bash
cd app

kind create cluster --config kubernetes/kind-config.yaml

kubectl apply -k kubernetes/env/dev

kubectl get all -n development
```

Validasi cepat:
- Namespace `development` ada.
- Backend, frontend, dan PostgreSQL pods running.
- Frontend bisa diakses via `http://localhost:30080`.

Detail lebih lengkap ada di `app/README.md`.

### Langkah 2 – Siapkan Observability Stack (`obervability/`)

[Inference] Manifests observability belum ada di repo ini, tapi desain dan tahapan lengkap sudah dijelaskan di `obervability/README.md`.

Mindset pemakaian:
- Anggap `app/` sebagai “application team”.
- Anggap `obervability/` sebagai “platform/observability team”.

Saat kamu mulai menulis manifests observability, letakkan di bawah `obervability/` dan ikuti struktur yang disarankan di README-nya.

---

## 3. Kontrak Antara App dan Observability

Agar observability bisa “plug in” tanpa mengubah app terus-menerus, gunakan kontrak berikut:

- **Namespace aplikasi**: `development`
- **Labels utama pods**:
  - Backend: `app=backend`, `tier=api`
  - Frontend: `app=frontend`, `tier=ui`
  - PostgreSQL: `app=postgres`, `tier=database`
- **Service names**:
  - Backend: `backend-service` (ClusterIP, port 8080)
  - Frontend: `frontend-service` (NodePort 30080 → 80)
  - PostgreSQL: `postgres-service` (ClusterIP, port 5432)

Observability stack di `obervability/` bisa asumsi:
- Scrape metrics berdasarkan namespace `development` dan label `app`, `tier`.
- Promtail/Loki pakai labels `namespace`, `pod`, `container`, `app` untuk query logs.

---

## 4. Integration Flow Singkat

Ringkasan integrasi ketika observability stack sudah diimplementasikan:

1. Jalankan cluster dan aplikasi (`app/`):
   - `kind create cluster --config app/kubernetes/kind-config.yaml`
   - `kubectl apply -k app/kubernetes/env/dev`
2. Deploy observability (`obervability/`):
   - `kubectl apply -k <path kustomize observability dev>` [Inference]
3. Akses sistem:
   - Frontend: `http://localhost:30080`
   - Grafana: via NodePort/Ingress yang kamu definisikan di observability manifests [Inference]
   - Prometheus UI: via Service/port-forward [Inference]
4. Gunakan Grafana untuk:
   - Lihat metrics cluster dan aplikasi (Prometheus).
   - Query logs aplikasi (Loki/Promtail).
   - Cek alerts (Prometheus + Alertmanager).

Detail step observability ada di `obervability/README.md`.

---

## 5. Next Steps Rekomendasi Belajar

Urutan belajar yang disarankan:

1. Selesaikan Tier 1 di `app/README.md` sampai:
   - Bisa deploy backend, frontend, dan PostgreSQL dengan Kustomize.
   - Paham namespace, ConfigMap, Deployment, Service, labels, dan selectors.
2. Setelah nyaman dengan dasar Kubernetes, pindah ke `obervability/README.md`:
   - Bangun Prometheus + Alertmanager + Grafana.
   - Tambah Loki + Promtail untuk logs.
   - Tambah alert rules sederhana dan coba failure scenarios.
3. Setelah stack observability stabil:
   - Tambah HPA berbasis metrics.
   - Tambah Ingress + TLS untuk akses app dan Grafana.



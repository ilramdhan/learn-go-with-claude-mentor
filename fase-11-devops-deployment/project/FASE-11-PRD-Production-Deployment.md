# 📋 PRD — Production Deployment untuk E-Commerce System

> **Fase:** 11 — DevOps & Deployment (Fase Terakhir)
> **Tipe Project:** Full Production Setup untuk E-Commerce System
> **Estimasi Pengerjaan:** 10–14 hari
> **Konsep yang Diuji:** GitHub Actions, Docker Registry, Helm, ArgoCD, Zero-Downtime Deployment, Secret Management, Disaster Recovery

---

## 🎯 Tujuan Project

Membawa E-Commerce System dari Fase 7–10 ke **production-grade deployment**. Setelah project ini, sistem berjalan di Kubernetes dengan:
- CI/CD pipeline yang otomatis build, test, dan deploy setiap commit
- Zero-downtime deployment strategy
- Secret management yang aman
- Backup dan disaster recovery yang teruji
- Infrastructure as Code (semua repeatable)

---

## ✅ Features yang Harus Diimplementasikan

### F-01: GitHub Actions CI/CD Pipeline
- **Trigger:** push ke main, PR ke main, manual trigger
- **Jobs yang berjalan paralel:** lint, unit test, integration test, security scan
- **Build:** Docker image untuk semua service dengan tagging: SHA + semver + latest
- **Push:** ke GHCR (GitHub Container Registry)
- **Deploy Staging:** otomatis setelah build berhasil
- **Smoke Test:** setelah deploy staging
- **Deploy Production:** hanya main branch, butuh approval manual (GitHub Environment)
- **Notification:** Slack/Discord saat deploy berhasil atau gagal

### F-02: Helm Charts untuk Semua Service
Buat Helm chart untuk: `auth-service`, `product-service`, `order-service`, `api-gateway`, `notification-service`

Setiap chart wajib include:
- Deployment dengan rolling update strategy
- Service (ClusterIP)
- ConfigMap untuk non-sensitive config
- HPA (min 2, max 10 replicas)
- PodDisruptionBudget (minAvailable: 1)
- Liveness + Readiness probes

Buat parent chart `ecommerce` yang include semua service sebagai dependencies.

### F-03: ArgoCD GitOps
- ArgoCD terinstall di cluster
- App of Apps pattern: satu root app yang mengelola semua service apps
- Auto-sync dari Git repository
- Self-healing: jika ada perubahan manual di cluster, ArgoCD revert ke Git state
- Sync policy: automated dengan prune dan selfHeal

### F-04: Zero-Downtime Deployment
Implementasikan salah satu strategy (pilih sesuai service):
- **Auth Service:** Blue-Green deployment
- **Product Service:** Canary deployment dengan Nginx Ingress (mulai 10%, naik bertahap)
- **Order Service:** Rolling update standar (sudah cukup untuk API service)

Test: jalankan `watch -n 0.5 "curl -s http://localhost/health/live"` selama deployment. Tidak boleh ada gap (downtime).

### F-05: Secret Management
- **Development:** Kubernetes Secrets dari file `.env.local` (tidak di-commit ke git)
- **Staging/Production:** Vault di Kubernetes
- ExternalSecret Operator yang sync Vault secrets ke K8s Secrets
- Secret rotation: update Vault value → K8s Secret otomatis update dalam 1 jam
- Verifikasi: tidak ada plaintext secrets di Git history, pod logs, atau env vars yang ter-log

### F-06: Database Migration Automation
- golang-migrate untuk semua service databases
- Migration dijalankan sebagai Kubernetes Job sebelum deployment (Helm pre-upgrade hook)
- Migration rollback capability (down migration)
- Test: zero-downtime schema change (tambah kolom nullable dulu, baru NOT NULL)

### F-07: Backup & Disaster Recovery
- CronJob backup PostgreSQL setiap hari jam 2 pagi
- Backup disimpan minimal 7 hari
- DR Runbook untuk skenario: database failure, service pod crash, secret corruption
- DR Drill: simulasikan dan dokumentasikan actual RTO

---

## 📁 Struktur File yang Dibutuhkan

```
ecommerce-microservices/
├── .github/
│   └── workflows/
│       ├── ci-cd.yml              ← main pipeline
│       ├── cleanup-registry.yml   ← weekly image cleanup
│       └── security-scan.yml      ← scheduled scan
│
├── charts/
│   ├── ecommerce/                 ← parent chart (App of Apps)
│   │   ├── Chart.yaml
│   │   └── values.yaml
│   ├── auth-service/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   ├── values-staging.yaml
│   │   ├── values-production.yaml
│   │   └── templates/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       ├── configmap.yaml
│   │       ├── hpa.yaml
│   │       ├── pdb.yaml
│   │       └── migrate-job.yaml
│   └── ... (service lain sama strukturnya)
│
├── argocd/
│   ├── projects/
│   │   └── ecommerce-project.yaml
│   └── applications/
│       ├── ecommerce-app-of-apps.yaml
│       ├── auth-service.yaml
│       ├── product-service.yaml
│       └── ...
│
├── k8s/
│   ├── namespaces.yaml
│   ├── rbac/
│   ├── networkpolicies/
│   └── cronjobs/
│       └── backup.yaml
│
├── terraform/
│   ├── modules/
│   │   └── rds-postgres/
│   └── environments/
│       ├── staging/
│       └── production/
│
├── migrations/
│   ├── auth-service/
│   ├── product-service/
│   └── order-service/
│
├── vault/
│   └── policies/
│       └── ecommerce-policy.hcl
│
├── docs/
│   ├── runbooks/
│   │   ├── database-failure.md
│   │   ├── service-crash.md
│   │   └── high-error-rate.md
│   └── architecture/
│       ├── deployment-architecture.md
│       └── ci-cd-flow.md
│
└── Makefile                       ← semua production commands
```

---

## 🧪 Acceptance Criteria

### Minimum (Wajib)
- [ ] GitHub Actions CI/CD pipeline berhasil: lint → test → build → push → deploy staging
- [ ] Helm chart tersedia untuk minimal 2 service (auth + gateway)
- [ ] Deploy ke minikube dengan satu command: `make deploy-staging`
- [ ] Secrets tidak ada di Git repository (use `.gitignore` + Kubernetes Secrets)
- [ ] Database migration berjalan otomatis saat deployment

### Good
- [ ] ArgoCD GitOps setup dan auto-sync berjalan
- [ ] Zero-downtime deployment untuk Auth Service (blue-green atau canary)
- [ ] Backup CronJob berjalan dan restore di-test
- [ ] DR Runbook tersedia untuk 3 skenario

### Excellent
- [ ] Full pipeline termasuk canary monitoring dan auto-rollback
- [ ] Vault + External Secrets Operator berjalan
- [ ] Terraform untuk at least 2 infrastructure resources
- [ ] KEDA untuk event-driven autoscaling Notification Service
- [ ] Production launch checklist 100% complete
- [ ] DR drill selesai dengan documented actual RTO

---

## 📖 Panduan Pengerjaan

### Minggu 1: Foundation

**Hari 1–2:** GitHub Actions
- Setup CI pipeline: lint + unit test + integration test
- Build dan push ke GHCR dengan proper tagging
- Verify: buat PR dan lihat semua checks pass

**Hari 3–4:** Helm Charts
- Buat chart untuk auth-service dan api-gateway
- Test dengan `helm install --dry-run`
- Deploy ke minikube

**Hari 5:** Database Migrations
- Setup golang-migrate
- Buat migration files untuk semua service
- Test run di minikube

### Minggu 2: GitOps & Security

**Hari 6–7:** ArgoCD GitOps
- Install ArgoCD di minikube
- Setup App of Apps
- Test auto-sync

**Hari 8:** Secret Management
- Setup Kubernetes Secrets dari .env
- Test bahwa secrets tidak ter-log

**Hari 9:** Zero-Downtime Deployment
- Implementasikan blue-green untuk auth-service
- Test: no downtime selama switch

**Hari 10:** Backup & DR
- Setup backup CronJob
- Buat DR runbook
- Test backup dan restore

### Minggu 3 (Excellent): Advanced

**Hari 11–12:** Canary + Monitoring
- Canary deployment untuk product-service
- Auto-rollback berdasarkan metrics

**Hari 13–14:** Final polish
- Makefile lengkap
- Documentation
- Production launch checklist review

---

## 📊 Deliverables

1. **GitHub repository** dengan semua kode, charts, dan workflows
2. **`Makefile`** dengan commands: `deploy-staging`, `deploy-production`, `rollback`, `status`, `health`, `backup`, `migrate`
3. **Helm charts** untuk semua service dengan values per environment
4. **`docs/runbooks/`** untuk 3 failure scenarios
5. **Production Launch Checklist** yang sudah di-review

---

## 🎓 Setelah Project Ini Selesai

Kamu sudah menyelesaikan **seluruh kurikulum Go Engineer dari Beginner ke Senior**!

Portfolio kamu mencakup:
- Sistem auth + product + order dengan Clean Architecture dan DDD
- Microservices dengan gRPC, Kafka, dan API Gateway
- Full observability (Prometheus, Grafana, Jaeger)
- Comprehensive test suite (unit, integration, contract, load, fuzz)
- Production deployment dengan CI/CD, GitOps, dan zero-downtime

**Kamu siap untuk posisi Senior Go Engineer!** 🚀

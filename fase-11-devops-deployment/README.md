# 📘 Fase 11: DevOps & Deployment (Fase Terakhir!)

> **Level:** 🔴 Advanced / Senior
> **Durasi Estimasi:** 3–4 minggu
> **Prasyarat:** ✅ Fase 7 (Microservices) + Fase 9 (Observability) + Fase 10 (Testing)

---

## 🎯 Tujuan Fase Ini

- ✅ Memahami DORA metrics dan apa yang membedakan tim elite
- ✅ Membangun GitHub Actions CI/CD pipeline yang production-ready
- ✅ Mengelola Docker images dengan proper tagging dan vulnerability scanning
- ✅ Setup Kubernetes production dengan RBAC, NetworkPolicy, ResourceQuota
- ✅ Menulis Helm charts untuk semua service
- ✅ Menerapkan GitOps dengan ArgoCD
- ✅ Zero-downtime deployment: Blue-Green dan Canary
- ✅ Database migration yang aman di production
- ✅ Secret management yang proper (Vault, External Secrets)
- ✅ Infrastructure as Code dengan Terraform
- ✅ Backup, disaster recovery, dan DR drill

---

## 🗂️ Isi Folder

```
fase-11-devops-deployment/
├── README.md
├── materi/
│   └── FASE-11-DevOps-Deployment.md    ← 13 modul (80KB, 2573 baris)
└── project/
    └── FASE-11-PRD-Production-Deployment.md
```

---

## 📚 Daftar Modul

| # | Modul | Konsep Utama |
|---|-------|-------------|
| 11.1 | DevOps Fundamentals | DORA metrics, CI/CD philosophy, Git branching |
| 11.2 | GitHub Actions CI/CD | Full pipeline: lint→test→build→push→deploy |
| 11.3 | Docker Registry | GHCR, tagging strategy, Trivy scan, multi-platform |
| 11.4 | Kubernetes Production | RBAC, NetworkPolicy, PDB, ResourceQuota |
| 11.5 | Helm Charts | Templates, values per env, hooks, App of Apps |
| 11.6 | GitOps ArgoCD | GitOps principles, auto-sync, self-healing |
| 11.7 | Blue-Green & Canary | Zero-downtime, traffic splitting, Flagger |
| 11.8 | Database Migration | golang-migrate, zero-downtime schema changes |
| 11.9 | Secret Management | K8s Secrets, Vault, External Secrets Operator |
| 11.10 | Terraform IaC | Modules, RDS, state management |
| 11.11 | Cost Optimization | VPA, KEDA, Spot instances |
| 11.12 | Disaster Recovery | Backup CronJob, DR runbook, DR drill |
| 11.13 | Complete Production | Full architecture, launch checklist |

---

## 🛠️ Tools yang Dibutuhkan

```bash
# Kubernetes
minikube start --cpus=4 --memory=8192

# Helm
brew install helm

# ArgoCD
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Terraform
brew install terraform

# stern (multi-pod log streaming)
brew install stern
```

---

## ✅ Checklist Kelulusan

- [ ] GitHub Actions CI pipeline green (lint + test + build + push)
- [ ] Helm chart deploy ke minikube dengan satu command
- [ ] ArgoCD auto-sync dari Git
- [ ] Zero-downtime deployment di-demo (no 5xx saat deploy)
- [ ] Secrets tidak ada di Git repository
- [ ] Database migration berjalan otomatis sebagai K8s Job
- [ ] Backup CronJob berjalan dan restore di-test
- [ ] DR Runbook tersedia untuk minimal 3 skenario
- [ ] **Menyelesaikan project Production Deployment**

---

## 🏆 Selamat Menyelesaikan Seluruh Kurikulum!

```
11 Fase × 13 Modul = 143 Modul Total
143 Latihan Hands-on
11 Project PRD yang realistic

Kamu sudah menguasai:
  ✅ Go language (beginner → advanced)
  ✅ REST API + gRPC
  ✅ Clean Architecture + DDD
  ✅ Microservices + Event-Driven Architecture
  ✅ Full Observability Stack
  ✅ Testing Mastery (unit → load → fuzz)
  ✅ Production DevOps (CI/CD → GitOps → IaC)

Selamat — kamu siap untuk posisi Senior Go Engineer! 🚀
```

---

## ⬅️ Kembali ke Awal

[Lihat Master Roadmap →](../00-MASTER-ROADMAP.md)

# 📘 Fase 7: Microservices Architecture

> **Level:** 🔴 Advanced
> **Durasi Estimasi:** 4–6 minggu
> **Prasyarat:** ✅ Selesaikan Fase 4 (Clean Architecture) + Fase 5 (gRPC) + Fase 6 (DDD)

---

## 🎯 Tujuan Fase Ini

- ✅ Memahami kapan microservices tepat digunakan dan kapan monolith lebih baik
- ✅ Membuat Dockerfile multi-stage yang menghasilkan image production-ready (< 20MB)
- ✅ Menjalankan multi-service system dengan Docker Compose
- ✅ Mengimplementasikan API Gateway pattern
- ✅ Inter-service communication yang resilient (gRPC + circuit breaker)
- ✅ Saga pattern untuk distributed transactions
- ✅ Deploy ke Kubernetes dengan zero-downtime rolling update

---

## 🗂️ Isi Folder

```
fase-07-microservices/
├── README.md                              ← Kamu di sini
├── materi/
│   └── FASE-7-Microservices.md           ← 13 modul (102KB)
└── project/
    └── FASE-7-PRD-Ecommerce-Microservices.md
```

---

## 📚 Daftar Modul

| # | Modul | Konsep Utama |
|---|-------|-------------|
| 7.1 | Apa itu Microservices? | Monolith vs MS, Conway's Law, anti-patterns |
| 7.2 | Docker untuk Go | Multi-stage build, distroless, .dockerignore |
| 7.3 | Docker Compose | Multi-service, health checks, networking |
| 7.4 | Communication Patterns | Sync vs async, gRPC client, deadline propagation |
| 7.5 | API Gateway | Reverse proxy, JWT validation, rate limiting |
| 7.6 | Service Discovery | ENV-based, DNS, health-aware load balancer |
| 7.7 | Inter-Service gRPC | Connection pool, metadata propagation, LB |
| 7.8 | Saga Pattern | Choreography vs orchestration, compensating transactions |
| 7.9 | Circuit Breaker | gobreaker, retry backoff, bulkhead, fallback |
| 7.10 | Configuration | Viper multi-env, secrets, startup validation |
| 7.11 | Health Checks | Liveness, readiness, dependency checks |
| 7.12 | Kubernetes | Pod, Deployment, Service, Ingress, HPA |
| 7.13 | Putting It Together | Architecture diagram, production checklist |

---

## 🛠️ Tools yang Dibutuhkan

```bash
# Wajib
docker --version && docker compose version

# Untuk Kubernetes (modul 7.12)
minikube version    # atau k3d/kind

# Opsional tapi berguna
dive                # analisa Docker image layers
hey                 # load testing
k9s                 # Kubernetes dashboard terminal
```

---

## 🎯 Project Akhir: E-Commerce Microservices System

File PRD: `project/FASE-7-PRD-Ecommerce-Microservices.md`

System dengan 5 services yang saling berkomunikasi:
- **Auth Service** (dari Fase 4)
- **Product Service** (dari Fase 5)
- **Order Service** (dari Fase 6)
- **API Gateway** (baru)
- **Notification Service** (sederhana)

**Estimasi pengerjaan:** 14–21 hari

---

## ✅ Checklist Kelulusan

- [ ] Semua service berjalan dengan `docker compose up -d`
- [ ] API Gateway routing + JWT validation berfungsi
- [ ] Order Service call Product Service via gRPC
- [ ] Checkout saga (happy path + failure compensation) berjalan
- [ ] Circuit breaker terbuka saat Product Service down
- [ ] Health checks tersedia di semua service
- [ ] Rolling update di Kubernetes tanpa downtime
- [ ] **Menyelesaikan project E-Commerce Microservices**

---

## ➡️ Selanjutnya

Setelah selesai Fase 7 → [Fase 8: Message Broker & Event-Driven](../fase-08-message-broker/)

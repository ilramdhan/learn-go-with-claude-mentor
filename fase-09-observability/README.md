# 📘 Fase 9: Observability & Production

> **Level:** 🔴 Advanced
> **Durasi Estimasi:** 2–3 minggu
> **Prasyarat:** ✅ Selesaikan Fase 7 (Microservices) + Fase 8 (Message Broker)

---

## 🎯 Tujuan Fase Ini

- ✅ Memahami Three Pillars of Observability (Logs, Metrics, Traces)
- ✅ Implementasi structured logging dengan Zap
- ✅ Instrumentasi Prometheus metrics (HTTP, gRPC, database, business)
- ✅ Build Grafana dashboards yang actionable
- ✅ Distributed tracing dengan OpenTelemetry + Jaeger
- ✅ SLO/SLI tracking dan alerting
- ✅ Production readiness checklist

---

## 🗂️ Isi Folder

```
fase-09-observability/
├── README.md
├── materi/
│   └── FASE-9-Observability.md    ← 13 modul (66KB)
└── project/
    └── FASE-9-PRD-Observability-Stack.md
```

---

## 📚 Daftar Modul

| # | Modul | Konsep Utama |
|---|-------|-------------|
| 9.1 | Three Pillars | Logs vs Metrics vs Traces, kapan pakai apa |
| 9.2 | Structured Logging | Zap setup, fields, output format |
| 9.3 | Log Best Practices | Sampling, correlation ID, data masking |
| 9.4 | Prometheus Metrics | Counter, Gauge, Histogram, setup |
| 9.5 | Custom Metrics | HTTP/gRPC/DB/business metrics |
| 9.6 | Grafana Dashboard | Setup, panels, cardinality |
| 9.7 | OpenTelemetry | Spans, attributes, SDK setup |
| 9.8 | Jaeger Integration | Export, UI, searching traces |
| 9.9 | Trace Propagation | W3C Trace Context, baggage |
| 9.10 | Alerting & SLO | Error budget, burn rate, alerts |
| 9.11 | pprof Profiling | CPU, memory, goroutine profiling |
| 9.12 | Production Checklist | Comprehensive readiness checklist |
| 9.13 | Full Stack | Prometheus + Grafana + Jaeger + Loki |

---

## 🛠️ Tools yang Dibutuhkan

```bash
# Go libraries
go get go.uber.org/zap
go get github.com/prometheus/client_golang/prometheus
go get go.opentelemetry.io/otel

# Docker services
docker compose -f docker-compose.observability.yml up -d
# Grafana: http://localhost:3000
# Prometheus: http://localhost:9090
# Jaeger: http://localhost:16686
```

---

## ✅ Checklist Kelulusan

- [ ] Zap JSON logging berjalan, request_id propagasi
- [ ] Prometheus metrics di /metrics semua service
- [ ] Grafana dashboard menampilkan RED metrics
- [ ] Traces muncul di Jaeger dengan trace ID konsisten
- [ ] Alerting rules terkonfigurasi
- [ ] SLO dashboard menampilkan error budget
- [ ] **Menyelesaikan project Full Observability Stack**

---

## ➡️ Selanjutnya

Setelah selesai Fase 9 → [Fase 10: Testing Mastery](../fase-10-testing-mastery/)

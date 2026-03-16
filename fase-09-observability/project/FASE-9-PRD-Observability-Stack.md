# 📋 PRD — Full Observability Stack

> **Fase:** 9 — Observability & Production
> **Tipe Project:** Infrastruktur observability untuk E-Commerce System
> **Estimasi Pengerjaan:** 7–10 hari
> **Konsep yang Diuji:** Zap logging, Prometheus metrics, Grafana, OpenTelemetry, Jaeger, SLO

---

## 🎯 Tujuan Project

Menambahkan **full observability stack** ke E-Commerce System dari Fase 7-8. Setelah project ini, tim engineering bisa menjawab pertanyaan-pertanyaan seperti:
- "Kenapa checkout lambat sejak 2 jam lalu?"
- "Berapa error rate Order Service saat ini?"
- "Request #abc123 ini lewat service apa saja dan butuh waktu berapa di tiap service?"
- "Apakah kita memenuhi SLO 99.9% bulan ini?"

---

## ✅ Features yang Harus Diimplementasikan

### F-01: Structured Logging (Semua Service)
- Ganti semua `fmt.Println` / `log.Printf` dengan Zap
- Output JSON di production, console di development
- Setiap request punya `request_id` yang konsisten di semua log
- Tidak ada sensitive data di logs (password, token, credit card)
- Log levels: DEBUG (dev), INFO/WARN/ERROR (production)

### F-02: Prometheus Metrics (Semua Service)
- `/metrics` endpoint tersedia di setiap service
- **HTTP metrics:** request rate, error rate, latency (per endpoint)
- **gRPC metrics:** request rate, error rate, latency (per method)
- **Database metrics:** query duration, connection pool usage
- **Business metrics:** orders created, revenue, active orders
- **Runtime metrics:** goroutines count, memory usage

### F-03: Grafana Dashboards
- **Service Overview Dashboard:** request rate, error rate, p99 latency untuk setiap service
- **Order Service Dashboard:** business metrics (orders, revenue, conversion rate)
- **Infrastructure Dashboard:** memory, CPU, goroutines, DB connections
- **SLO Dashboard:** error budget, burn rate, SLO compliance %

### F-04: Distributed Tracing (OpenTelemetry + Jaeger)
- Traces dari semua service dikirim ke Jaeger
- Trace propagation berfungsi antar semua service
- Gin dan gRPC sudah auto-instrumented
- Business operations (confirm order, check stock) punya manual spans
- Trace exemplars terhubung ke Prometheus metrics

### F-05: Alerting
- Alert saat error rate > 5% selama 2 menit
- Alert saat p99 latency > 1 detik selama 5 menit
- Alert saat service down > 1 menit
- Alert saat no orders created > 1 jam (anomaly)
- Alertmanager configured (minimal: log to file atau webhook)

### F-06: SLO Tracking
- SLO untuk Order Service: 99.9% good requests (< 200ms, non-5xx)
- Dashboard menampilkan error budget remaining
- Alert saat error budget burn rate > 5x normal

---

## 🏗️ Infrastruktur yang Dibutuhkan

```yaml
# docker-compose.observability.yml
services:
  prometheus:  # :9090
  grafana:     # :3000
  jaeger:      # :16686 UI, :4317 OTLP
  loki:        # :3100
  promtail:    # log collector
  alertmanager: # :9093
```

---

## 📋 Acceptance Criteria

### Minimum
- [ ] Semua service menggunakan Zap (JSON output)
- [ ] `request_id` muncul di semua logs dalam satu HTTP request
- [ ] `/metrics` endpoint ada di semua service dan Prometheus scraping berhasil
- [ ] Grafana dashboard menampilkan request rate, error rate, latency
- [ ] Traces muncul di Jaeger untuk setiap HTTP request ke order service

### Good
- [ ] Trace ID konsisten antar minimal 3 service
- [ ] Business metrics (orders, revenue) tersedia
- [ ] Alerting rules terkonfigurasi
- [ ] SLO dashboard menampilkan error budget

### Excellent
- [ ] Grafana Loki menampilkan logs yang bisa di-filter
- [ ] Trace exemplars terhubung ke metrics
- [ ] Runbook untuk 3 failure scenarios dengan screenshot Grafana
- [ ] pprof profiling endpoint dengan basic auth

---

## 🧪 Test Scenarios

### Scenario 1: Request Tracing
```bash
# Kirim request ke order service
curl -X POST localhost:8080/api/v1/orders \
  -H "Authorization: Bearer TOKEN" \
  -d '{"items":[{"product_id":1,"quantity":2}]}'

# 1. Ambil request_id dari response header X-Request-ID
# 2. Cari di Grafana Loki: {service="order-service"} |= "REQUEST_ID"
# 3. Cari di Jaeger: service=order-service, tag=request_id=REQUEST_ID
# Verifikasi: log dan trace untuk request yang sama bisa ditemukan
```

### Scenario 2: SLO Violation Simulation
```bash
# Inject latency ke database (simulate slow queries)
# Verifikasi: p99 latency naik di Grafana
# Verifikasi: SLO compliance turun
# Verifikasi: error budget terkikis
# Verifikasi: alert firing di Prometheus
```

---

## 📁 File Yang Perlu Dibuat/Dimodifikasi

```
ecommerce-microservices/
├── monitoring/
│   ├── prometheus/
│   │   ├── prometheus.yml
│   │   └── alerts.yml
│   ├── grafana/
│   │   ├── provisioning/
│   │   │   ├── datasources/
│   │   │   └── dashboards/
│   │   └── dashboards/
│   │       ├── service-overview.json
│   │       ├── order-business.json
│   │       └── slo-tracking.json
│   ├── loki/
│   │   └── local-config.yaml
│   ├── promtail/
│   │   └── config.yml
│   └── alertmanager/
│       └── alertmanager.yml
│
├── pkg/
│   ├── logger/
│   │   ├── logger.go       ← Zap setup
│   │   ├── context.go      ← logger in context
│   │   └── middleware.go   ← Gin middleware
│   ├── metrics/
│   │   ├── metrics.go      ← all metric definitions
│   │   ├── middleware.go   ← Gin metrics middleware
│   │   └── grpc.go         ← gRPC interceptor
│   └── tracing/
│       ├── tracing.go      ← OTel setup
│       └── middleware.go   ← helper functions
│
├── docker-compose.observability.yml
└── services/
    └── */                  ← semua service di-update dengan logger, metrics, tracing
```

---

*Setelah fase ini, sistem kamu bisa di-observe di production secara profesional!*

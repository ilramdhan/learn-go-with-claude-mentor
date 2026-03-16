# 📋 Observability Quick Reference

## Three Pillars Decision Matrix

| Question | Use |
|----------|-----|
| "Berapa request/detik sekarang?" | Metrics (Prometheus) |
| "Error rate service X?" | Metrics (Prometheus) |
| "Kenapa request #abc lambat?" | Traces (Jaeger) |
| "Apa yang terjadi saat error ini?" | Logs (Loki) |
| "Siapa yang call siapa?" | Traces |
| "Kapan masalah mulai?" | Metrics + Alerts |
| "Apa pesan error lengkapnya?" | Logs |

## PromQL Cheat Sheet

```promql
# Request rate (requests/second) dalam 5 menit
rate(http_requests_total[5m])

# Error rate per service
sum(rate(http_requests_total{status_code=~"5.."}[5m])) by (service)
/
sum(rate(http_requests_total[5m])) by (service)

# Latency p50/p95/p99
histogram_quantile(0.99,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (service, le)
) * 1000

# Active goroutines
go_goroutines{service="order-service"}

# Memory usage (MB)
go_memstats_alloc_bytes / 1024 / 1024

# Kafka consumer lag
kafka_consumer_lag > 1000

# SLO: good requests percentage (last 30 days)
sum(increase(slo_good_requests_total[30d]))
/
sum(increase(slo_total_requests_total[30d])) * 100
```

## LogQL Cheat Sheet

```logql
# All error logs from a service
{service="order-service"} | json | level="error"

# Find slow requests (> 1 second)
{service="api-gateway"} | json | latency_ms > 1000

# Correlate by request ID
{service=~".*"} | json | request_id="abc-123-def"

# Error rate (log-based)
sum(rate({service="auth-service"} | json | level="error" [5m]))

# Parse and filter
{service="order-service"}
  | json
  | line_format "{{.timestamp}} [{{.level}}] {{.msg}} order={{.order_id}}"
```

## Zap Logger Quick Reference

```go
// Always use typed fields for structured logging
logger.Info("order confirmed",
    zap.Uint64("order_id", 123),
    zap.String("user_email", "alice@test.com"),
    zap.Float64("amount", 250000),
    zap.Duration("processing_time", 45*time.Millisecond),
)

// Log levels guide:
// DEBUG - detailed dev info (disabled in prod)
// INFO  - business events, service lifecycle
// WARN  - degraded but functional (high latency, retry)
// ERROR - request failed, needs investigation
// FATAL - service cannot start, os.Exit(1)
```

## Alert Severity Guidelines

| Severity | Response Time | Examples |
|----------|--------------|---------|
| **critical** | < 15 min (page on-call) | Service down, error rate > 10% |
| **warning** | < 4 hours | High latency, consumer lag |
| **info** | Next business day | Scheduled maintenance, low traffic |

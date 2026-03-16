# 📘 FASE 9: Observability & Production

> **Prasyarat:** Selesaikan Fase 7 (Microservices) + Fase 8 (Message Broker)
> **Durasi:** 2–3 minggu
> **Project Akhir:** Full Observability Stack untuk E-Commerce System
> **Tujuan:** Membuat sistem yang dapat di-observe — tahu apa yang terjadi di production tanpa harus SSH ke server

---

## 🗂️ Daftar Modul

| # | Modul | Topik |
|---|-------|-------|
| 9.1 | Three Pillars of Observability | Logs, Metrics, Traces — kapan pakai apa |
| 9.2 | Structured Logging dengan Zap | Setup, fields, log levels, output format |
| 9.3 | Log Best Practices | Sampling, correlation ID, sensitive data |
| 9.4 | Metrics dengan Prometheus | Counter, Gauge, Histogram, Summary |
| 9.5 | Custom Metrics & Instrumentasi | HTTP, gRPC, database, business metrics |
| 9.6 | Grafana Dashboard | Setup, panels, queries, alerting |
| 9.7 | Distributed Tracing dengan OpenTelemetry | Spans, context propagation, SDK |
| 9.8 | Jaeger Integration | Export traces, UI, searching |
| 9.9 | Trace Propagation di Microservices | W3C Trace Context, baggage |
| 9.10 | Alerting & SLO/SLI/SLA | Error budgets, burn rate alerts |
| 9.11 | Profiling dengan pprof | CPU, memory, goroutine profiling |
| 9.12 | Production Readiness Checklist | Checklist lengkap sebelum go-live |
| 9.13 | Full Observability Stack | Semua komponen terintegrasi |

---

## 📦 Modul 9.1 — The Three Pillars of Observability

### Observability vs Monitoring

```
MONITORING (tradisional):
  "Apakah sistem berjalan?" — jawab: ya/tidak
  Cek predefined metrics: CPU, memory, uptime
  Baik untuk: sistem yang dikenal baik

OBSERVABILITY (modern):
  "MENGAPA sistem berperilaku demikian?" — jawab: tidak tahu sebelum lihat data
  Bisa menjawab pertanyaan yang BELUM pernah dipikirkan sebelumnya
  Baik untuk: distributed systems yang kompleks

Quote dari Charity Majors (CTO Honeycomb):
"You cannot monitor your way out of microservices complexity.
 You need to be able to ask arbitrary questions of your system."
```

### The Three Pillars

```
┌─────────────────────────────────────────────────────────┐
│                    OBSERVABILITY                         │
│                                                         │
│   LOGS          METRICS          TRACES                 │
│   ─────         ───────          ──────                 │
│   "What        "How many?"      "Where did              │
│    happened?"  "How fast?"       time go?"              │
│                "How much?"                              │
│                                                         │
│   Discrete      Aggregated       Causality &            │
│   events        numbers          relationships          │
│                                                         │
│   Per-request   Over time        Per-request flow       │
│   details       summary          across services        │
└─────────────────────────────────────────────────────────┘
```

### Kapan Pakai Apa?

```
LOGS — pakai untuk:
  ✅ Debug masalah yang spesifik ("apa yang terjadi di request ini?")
  ✅ Audit trail (siapa login kapan, apa yang diubah)
  ✅ Error details (stack trace, context lengkap)
  ❌ Bukan untuk: trending, capacity planning, alerting

METRICS — pakai untuk:
  ✅ Dashboard real-time (berapa request/detik sekarang?)
  ✅ Alerting (error rate > 5% → kirim alert)
  ✅ Capacity planning (CPU trend naik → perlu scale up)
  ✅ SLO tracking (berapa % request < 200ms?)
  ❌ Bukan untuk: debug masalah spesifik per request

TRACES — pakai untuk:
  ✅ Debugging latency ("kenapa request ini butuh 3 detik?")
  ✅ Memahami alur di microservices ("request ini lewat service apa saja?")
  ✅ Dependency analysis (siapa yang call siapa?)
  ❌ Bukan untuk: storage sangat mahal jika 100% sample
```

### Observability dalam Konteks Microservices

```
Request dari client:
  API Gateway (50ms)
      │
      ├── Auth Service (10ms)
      │
      ├── Product Service (20ms)
      │       └── Database query (15ms)
      │
      └── Order Service (30ms)
              ├── gRPC call Product Service (8ms)
              └── Database write (12ms)

Tanpa observability: "Request lambat" — tidak tahu kenapa
Dengan tracing: "Order Service DB write lambat (12ms vs normal 2ms)"
Dengan metrics: "DB write p99 naik sejak 2 jam lalu"
Dengan logs: "Deadlock pada tabel orders saat concurrent writes"
```

### Tools yang Akan Dipelajari

```
LOGS:    Zap (uber-go/zap) — structured, fast, zero-allocation
METRICS: Prometheus — pull-based metrics + Grafana untuk visualisasi
TRACES:  OpenTelemetry SDK + Jaeger untuk storage dan UI

Stack ini adalah INDUSTRY STANDARD di Go ecosystem.
Dipakai oleh: Google, Uber, Netflix, Gojek, dan hampir semua Go shops.
```

### 🏋️ Latihan 9.1

1. Buat **observability requirements doc** untuk Blog API dari Fase 3: tentukan (a) 5 metric penting yang perlu di-track, (b) 3 log event yang harus selalu ada, (c) 2 trace yang critical untuk debug. Jelaskan mengapa masing-masing.
2. Analisis **failure scenario**: "User report: checkout lambat sejak 1 jam lalu". Bagaimana kamu investigate jika ada (a) hanya logs, (b) logs + metrics, (c) logs + metrics + traces? Tulis langkah diagnosis untuk setiap kasus.
3. Buat **SLO draft** untuk Order Service: tentukan target untuk (a) availability, (b) latency p99, (c) error rate. Jelaskan bagaimana mengukurnya.

---

## 📦 Modul 9.2 — Structured Logging dengan Zap

### Mengapa Zap?

```
MASALAH dengan fmt.Println atau log.Printf:
  fmt.Println("User logged in: " + username)
  → Output: "User logged in: alice"
  → Tidak bisa di-query: "berikan semua login dari user alice"
  → Tidak ada struktur, tidak bisa di-parse otomatis

STRUCTURED LOGGING:
  logger.Info("user logged in",
    zap.String("username", "alice"),
    zap.Uint64("user_id", 123),
    zap.String("ip", "192.168.1.1"),
  )
  → Output: {"level":"info","msg":"user logged in","username":"alice","user_id":123,"ip":"192.168.1.1"}
  → Bisa di-query: level:info AND username:alice
  → Bisa di-filter, di-aggregate, di-alert

ZAP vs zerolog vs logrus:
  zap     → fastest, zero allocation di hot path, production favorite
  zerolog → sedikit lebih cepat, chainable API
  logrus  → paling populer dulu, tapi lebih lambat, deprecated
```

### Setup Zap

```bash
go get go.uber.org/zap
go get go.uber.org/zap/zapcore
```

```go
// pkg/logger/logger.go
package logger

import (
    "os"
    "time"

    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

// New membuat logger yang sudah dikonfigurasi sesuai environment
func New(level, format, serviceName, version string) (*zap.Logger, error) {
    // Parse log level
    var zapLevel zapcore.Level
    if err := zapLevel.UnmarshalText([]byte(level)); err != nil {
        zapLevel = zapcore.InfoLevel // default ke info jika invalid
    }

    // Encoder config
    encoderCfg := zapcore.EncoderConfig{
        TimeKey:        "timestamp",
        LevelKey:       "level",
        NameKey:        "logger",
        CallerKey:      "caller",
        FunctionKey:    zapcore.OmitKey,
        MessageKey:     "msg",
        StacktraceKey:  "stacktrace",
        LineEnding:     zapcore.DefaultLineEnding,
        EncodeLevel:    zapcore.LowercaseLevelEncoder,
        EncodeTime:     zapcore.ISO8601TimeEncoder,
        EncodeDuration: zapcore.MillisDurationEncoder,
        EncodeCaller:   zapcore.ShortCallerEncoder,
    }

    // Pilih encoder berdasarkan format
    var encoder zapcore.Encoder
    switch format {
    case "console":
        encoderCfg.EncodeLevel = zapcore.CapitalColorLevelEncoder // warna di terminal
        encoderCfg.EncodeTime = func(t time.Time, enc zapcore.PrimitiveArrayEncoder) {
            enc.AppendString(t.Format("15:04:05.000"))
        }
        encoder = zapcore.NewConsoleEncoder(encoderCfg)
    default: // "json" — untuk production
        encoder = zapcore.NewJSONEncoder(encoderCfg)
    }

    // Core: writer ke stdout
    core := zapcore.NewCore(
        encoder,
        zapcore.AddSync(os.Stdout),
        zapLevel,
    )

    // Buat logger dengan service info yang selalu ada
    logger := zap.New(core,
        zap.AddCaller(),                    // tambahkan file:line info
        zap.AddCallerSkip(0),
        zap.AddStacktrace(zap.ErrorLevel), // stack trace untuk error ke atas
    ).With(
        // Fields yang selalu ada di setiap log entry
        zap.String("service", serviceName),
        zap.String("version", version),
    )

    return logger, nil
}

// MustNew panik jika gagal — cocok untuk main()
func MustNew(level, format, serviceName, version string) *zap.Logger {
    logger, err := New(level, format, serviceName, version)
    if err != nil {
        panic("failed to initialize logger: " + err.Error())
    }
    return logger
}
```

### Penggunaan Dasar

```go
// main.go
logger := logger.MustNew("debug", "console", "auth-service", "1.0.0")
defer logger.Sync() // flush buffer sebelum exit

// Log levels
logger.Debug("debug detail", zap.String("key", "value"))
logger.Info("service started", zap.Int("port", 8080))
logger.Warn("high memory usage", zap.Float64("percent", 85.5))
logger.Error("database error", zap.Error(err), zap.String("query", "SELECT ..."))
logger.Fatal("cannot start", zap.Error(err)) // log + os.Exit(1)

// Structured fields — SELALU pakai typed fields!
logger.Info("user action",
    zap.Uint64("user_id", 123),
    zap.String("action", "login"),
    zap.String("ip", "192.168.1.1"),
    zap.Duration("latency", 45*time.Millisecond),
    zap.Bool("success", true),
    zap.Strings("roles", []string{"admin", "user"}),
    zap.Any("metadata", map[string]string{"key": "value"}),
)

// Sugar logger — lebih mudah ditulis, sedikit lebih lambat
sugar := logger.Sugar()
sugar.Infof("processing order %d for user %s", orderID, username)
sugar.Warnw("slow query", "duration_ms", 350, "query", "SELECT *")
```

### Logger dengan Request Context

```go
// pkg/logger/context.go
package logger

import (
    "context"
    "go.uber.org/zap"
)

type contextKey string
const loggerKey contextKey = "logger"

// FromContext mengambil logger dari context
// Jika tidak ada, return global logger
func FromContext(ctx context.Context) *zap.Logger {
    if logger, ok := ctx.Value(loggerKey).(*zap.Logger); ok {
        return logger
    }
    return zap.L() // global logger
}

// WithContext menyimpan logger ke context
func WithContext(ctx context.Context, logger *zap.Logger) context.Context {
    return context.WithValue(ctx, loggerKey, logger)
}

// WithFields menambahkan fields ke logger dan simpan ke context baru
// Pattern ini memungkinkan semua log dalam satu request punya fields yang sama
func WithFields(ctx context.Context, fields ...zap.Field) context.Context {
    logger := FromContext(ctx).With(fields...)
    return WithContext(ctx, logger)
}
```

```go
// Gin middleware yang inject request-scoped logger
func RequestLoggerMiddleware(baseLogger *zap.Logger) gin.HandlerFunc {
    return func(c *gin.Context) {
        requestID := c.GetHeader("X-Request-ID")
        if requestID == "" {
            requestID = uuid.New().String()
        }
        c.Header("X-Request-ID", requestID)

        // Buat logger dengan fields request-specific
        reqLogger := baseLogger.With(
            zap.String("request_id", requestID),
            zap.String("method", c.Request.Method),
            zap.String("path", c.Request.URL.Path),
            zap.String("ip", c.ClientIP()),
            zap.String("user_agent", c.Request.UserAgent()),
        )

        // Simpan ke context — semua handler dalam request ini bisa pakai
        ctx := logger.WithContext(c.Request.Context(), reqLogger)
        c.Request = c.Request.WithContext(ctx)

        start := time.Now()
        c.Next()

        // Log setelah handler selesai
        reqLogger.Info("request completed",
            zap.Int("status", c.Writer.Status()),
            zap.Duration("latency", time.Since(start)),
            zap.Int("bytes_written", c.Writer.Size()),
        )
    }
}

// Penggunaan di handler
func (h *UserHandler) GetProfile(c *gin.Context) {
    log := logger.FromContext(c.Request.Context())

    // Log sudah punya request_id, method, path, dll otomatis
    log.Info("fetching user profile", zap.Uint64("user_id", userID))

    user, err := h.useCase.Execute(c.Request.Context(), userID)
    if err != nil {
        log.Error("failed to fetch profile", zap.Error(err))
        // handle error
        return
    }

    log.Info("profile fetched successfully")
    response.OK(c, user)
}
```

### 🏋️ Latihan 9.2

1. Setup Zap di Auth Service dari Fase 4. Buat logger yang output JSON di production dan console di development. Tambahkan field tetap: `service`, `version`, `env`. Verifikasi output JSON bisa di-parse dengan `jq`.
2. Buat `RequestLoggerMiddleware` untuk Gin yang log setiap request dengan: request_id, method, path, status, latency, user_id (jika ada). Test bahwa request_id konsisten antara request log dan semua log di handler.
3. Implementasikan **log rotation**: konfigurasi Zap untuk menulis ke file dengan rotation menggunakan `gopkg.in/natefinish/lumberjack.v2`. File baru dibuat setiap hari, simpan 7 hari terakhir, max size 100MB per file.

---

## 📦 Modul 9.3 — Log Best Practices

### Do's dan Don'ts

```go
// ❌ JANGAN: log informasi sensitif
logger.Info("user authenticated",
    zap.String("password", req.Password),  // BAHAYA!
    zap.String("credit_card", "4111..."),  // BAHAYA!
    zap.String("jwt_token", token),        // BAHAYA! (bisa dicuri dari logs)
)

// ✅ LAKUKAN: mask data sensitif
logger.Info("user authenticated",
    zap.Uint64("user_id", user.ID),  // ID saja, bukan password
    zap.String("email_domain", extractDomain(email)), // domain saja, bukan full email
)

// ❌ JANGAN: log di setiap baris (terlalu verbose, mahal)
for _, item := range items {
    logger.Debug("processing item", zap.Any("item", item)) // spam!
}

// ✅ LAKUKAN: log summary
logger.Debug("processing items",
    zap.Int("count", len(items)),
    zap.Duration("estimated_time", estimateTime(items)),
)

// ❌ JANGAN: log tanpa context
logger.Error("database error") // error apa? query apa? user mana?

// ✅ LAKUKAN: log dengan context yang cukup untuk debug
logger.Error("database query failed",
    zap.Error(err),
    zap.String("operation", "FindUserByEmail"),
    zap.String("table", "users"),
    zap.Duration("duration", time.Since(start)),
)

// ❌ JANGAN: log sesuatu yang tidak akan pernah dibaca
logger.Debug("entering function GetUser")
logger.Debug("got user from database")
logger.Debug("returning user")

// ✅ LAKUKAN: log meaningful events
logger.Info("user profile retrieved",
    zap.Uint64("user_id", userID),
    zap.Bool("from_cache", fromCache),
    zap.Duration("latency", time.Since(start)),
)
```

### Log Sampling untuk High-Traffic

```go
// pkg/logger/sampling.go
// Sampling: tidak log SEMUA request, hanya sample
// Mengurangi biaya storage dan meningkatkan performa

import (
    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

func NewSampledLogger(base *zap.Logger) *zap.Logger {
    // Sampling config:
    // - 100 pertama per detik: log semua
    // - Setelah itu: log 1 dari 100
    sampledCore := zapcore.NewSamplerWithOptions(
        base.Core(),
        time.Second,  // interval
        100,          // first N messages per interval
        100,          // setelahnya, log 1 dari N
    )

    return zap.New(sampledCore, zap.AddCaller())
}

// Penggunaan:
// logger hanya untuk high-volume logs seperti per-request access logs
sampledLogger := NewSampledLogger(baseLogger)
sampledLogger.Info("request processed") // akan di-sample 1%
```

### Correlation ID untuk Distributed Tracing

```go
// Correlation ID menghubungkan logs antar service
// Jika request melewati 5 service, semua log punya ID yang sama

// Di API Gateway — inject correlation ID
func correlationMiddleware(c *gin.Context) {
    corrID := c.GetHeader("X-Correlation-ID")
    if corrID == "" {
        corrID = uuid.New().String()
    }
    c.Header("X-Correlation-ID", corrID)

    // Tambahkan ke logger context
    ctx := logger.WithFields(c.Request.Context(),
        zap.String("correlation_id", corrID),
    )
    c.Request = c.Request.WithContext(ctx)
    c.Next()
}

// Di setiap service — teruskan correlation ID
func propagateCorrelationID(outgoingReq *http.Request, ctx context.Context) {
    log := logger.FromContext(ctx)
    if corrID := extractCorrelationID(ctx); corrID != "" {
        outgoingReq.Header.Set("X-Correlation-ID", corrID)
        log.Debug("propagating correlation ID", zap.String("correlation_id", corrID))
    }
}
```

### Log Level Strategy

```
PRODUCTION:
  DEBUG → OFF (terlalu verbose, biaya mahal)
  INFO  → ON  (normal operations, business events)
  WARN  → ON  (sesuatu tidak optimal tapi tidak error)
  ERROR → ON  (sesuatu gagal, perlu investigasi)
  FATAL → ON  (service tidak bisa start, exit)

STAGING:
  DEBUG → ON untuk komponen tertentu saja
  INFO/WARN/ERROR/FATAL → ON semua

DEVELOPMENT:
  DEBUG → ON semua
  Format: console (berwarna, mudah dibaca manusia)

WHAT TO LOG AT EACH LEVEL:
  DEBUG: masuk/keluar function, intermediate values (hanya dev)
  INFO:  user actions, service started/stopped, config loaded
  WARN:  deprecated API dipakai, retry attempts, high latency
  ERROR: request failed, database error, downstream service error
  FATAL: failed to connect DB at startup, invalid config
```

### 🏋️ Latihan 9.3

1. Audit semua log di Auth Service dan kategorikan setiap log statement ke level yang tepat (DEBUG/INFO/WARN/ERROR). Perbaiki log yang kurang informatif dan hapus log yang tidak berguna.
2. Buat `SensitiveDataFilter` — Zap Core wrapper yang otomatis mask field tertentu (password, credit_card, ssn, token) dengan `***REDACTED***`. Test bahwa data sensitif tidak muncul di output.
3. Implementasikan **structured error logging helper**: fungsi `LogError(ctx, msg, err, fields...)` yang otomatis extract: error type, stack trace (jika error di-wrap), dan HTTP status code (jika AppError). Pastikan format konsisten di semua service.

---

## 📦 Modul 9.4 — Metrics dengan Prometheus

### Apa itu Prometheus?

```
Prometheus adalah sistem monitoring dan alerting:
  - Scrape metrics dari services secara periodic (pull model)
  - Simpan sebagai time-series database
  - Query dengan PromQL
  - Alert jika kondisi terpenuhi

PULL MODEL (Prometheus) vs PUSH MODEL (StatsD, Datadog):

Pull:  Prometheus ──scrape──> Service :8080/metrics (setiap 15 detik)
Push:  Service ──push──> StatsD Server ──> Prometheus

Pull lebih reliable: 
  - Service tidak perlu tahu alamat monitoring server
  - Prometheus tahu jika service down (tidak bisa scrape)
```

### Empat Metric Types

```
COUNTER: Nilai yang hanya naik (tidak pernah turun)
  - Total HTTP requests
  - Total errors
  - Total bytes transferred
  Gunakan untuk: "berapa total X selama ini?"
  
GAUGE: Nilai yang bisa naik dan turun
  - Current active connections
  - Memory usage saat ini
  - Queue size saat ini
  Gunakan untuk: "berapa X sekarang?"

HISTOGRAM: Distribusi nilai (dalam buckets)
  - Request latency: 0-10ms, 10-50ms, 50-100ms, 100-500ms, >500ms
  - Request size
  Gunakan untuk: "berapa % request < 100ms?"
  Output: _bucket, _count, _sum

SUMMARY: Mirip histogram, tapi hitung quantile di sisi client
  - p50, p95, p99 latency
  Kurang fleksibel dari histogram (tidak bisa aggregate)
  Jarang dipakai di Go — gunakan Histogram sebagai gantinya
```

### Setup Prometheus

```bash
go get github.com/prometheus/client_golang/prometheus
go get github.com/prometheus/client_golang/prometheus/promauto
go get github.com/prometheus/client_golang/prometheus/promhttp
```

### Metric Dasar

```go
// pkg/metrics/metrics.go
package metrics

import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
)

// Semua metric didefinisikan di satu tempat
// promauto otomatis register ke default registry

var (
    // COUNTER: jumlah HTTP request
    HTTPRequestsTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Namespace: "ecommerce",
            Subsystem: "http",
            Name:      "requests_total",
            Help:      "Total number of HTTP requests",
        },
        []string{"service", "method", "path", "status_code"},
    )

    // HISTOGRAM: latency HTTP request
    HTTPRequestDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Namespace: "ecommerce",
            Subsystem: "http",
            Name:      "request_duration_seconds",
            Help:      "HTTP request latencies in seconds",
            // Buckets: dari 1ms sampai 10 detik
            Buckets: []float64{.001, .005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5, 10},
        },
        []string{"service", "method", "path"},
    )

    // GAUGE: aktif goroutines
    ActiveGoroutines = promauto.NewGaugeVec(
        prometheus.GaugeOpts{
            Namespace: "ecommerce",
            Subsystem: "runtime",
            Name:      "goroutines_active",
            Help:      "Number of active goroutines",
        },
        []string{"service"},
    )

    // COUNTER: database errors
    DatabaseErrorsTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Namespace: "ecommerce",
            Subsystem: "database",
            Name:      "errors_total",
            Help:      "Total database errors",
        },
        []string{"service", "operation", "error_type"},
    )

    // HISTOGRAM: database query duration
    DatabaseQueryDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Namespace: "ecommerce",
            Subsystem: "database",
            Name:      "query_duration_seconds",
            Help:      "Database query duration in seconds",
            Buckets:   prometheus.DefBuckets, // 0.005, 0.01, 0.025, ..., 10
        },
        []string{"service", "operation"},
    )
)
```

### Gin Middleware untuk HTTP Metrics

```go
// pkg/metrics/gin_middleware.go
package metrics

import (
    "strconv"
    "time"

    "github.com/gin-gonic/gin"
)

// PrometheusMiddleware mencatat HTTP request metrics secara otomatis
func PrometheusMiddleware(serviceName string) gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()

        // Jalankan handler
        c.Next()

        // Catat setelah handler selesai
        duration := time.Since(start).Seconds()
        statusCode := strconv.Itoa(c.Writer.Status())

        // Normalisasi path untuk menghindari high cardinality
        // /users/123 → /users/:id
        path := normalizePath(c.FullPath())
        if path == "" {
            path = "unknown"
        }

        HTTPRequestsTotal.WithLabelValues(
            serviceName,
            c.Request.Method,
            path,
            statusCode,
        ).Inc()

        HTTPRequestDuration.WithLabelValues(
            serviceName,
            c.Request.Method,
            path,
        ).Observe(duration)
    }
}

// normalizePath mengganti ID dengan :id untuk reduce cardinality
// /users/123 → /users/:id
// /orders/456/items → /orders/:id/items
func normalizePath(path string) string {
    // Gin sudah memberikan path template seperti /users/:id
    // Jadi ini biasanya tidak perlu, tapi berguna sebagai fallback
    return path
}
```

### Expose Metrics Endpoint

```go
// main.go — tambahkan /metrics endpoint
import "github.com/prometheus/client_golang/prometheus/promhttp"

func setupRouter(/* ... */) *gin.Engine {
    r := gin.New()
    r.Use(metrics.PrometheusMiddleware("auth-service"))

    // Metrics endpoint — diakses oleh Prometheus scraper
    // PENTING: jangan expose ke publik! Gunakan middleware atau port terpisah
    r.GET("/metrics", gin.WrapH(promhttp.Handler()))

    // ... routes lainnya ...
    return r
}
```

### docker-compose untuk Prometheus

```yaml
# docker-compose.monitoring.yml
services:
  prometheus:
    image: prom/prometheus:v2.48.0
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=15d'
      - '--web.enable-lifecycle'  # enable hot reload
    networks:
      - monitoring

volumes:
  prometheus-data:

networks:
  monitoring:
    driver: bridge
```

```yaml
# monitoring/prometheus.yml
global:
  scrape_interval: 15s       # scrape setiap 15 detik
  evaluation_interval: 15s   # evaluate rules setiap 15 detik

scrape_configs:
  # Prometheus sendiri
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Auth Service
  - job_name: 'auth-service'
    static_configs:
      - targets: ['auth-service:8080']
    metrics_path: '/metrics'

  # Product Service
  - job_name: 'product-service'
    static_configs:
      - targets: ['product-service:8080']

  # Order Service
  - job_name: 'order-service'
    static_configs:
      - targets: ['order-service:8080']

  # API Gateway
  - job_name: 'api-gateway'
    static_configs:
      - targets: ['api-gateway:8080']
```

### 🏋️ Latihan 9.4

1. Tambahkan Prometheus endpoint `/metrics` ke Auth Service. Start Prometheus dengan docker-compose. Verifikasi metric `ecommerce_http_requests_total` muncul di `http://localhost:9090`. Buat request beberapa kali dan lihat counter naik.
2. Buat query PromQL untuk: (a) request rate per menit per service, (b) error rate (5xx) per service, (c) p95 latency per endpoint. Test di Prometheus UI.
3. Tambahkan metric `ecommerce_database_query_duration_seconds` ke GORM callbacks. Setiap database operation (Find, Create, Update, Delete) otomatis tercatat durasinya. Verifikasi dengan slow query simulation.

---

## 📦 Modul 9.5 — Custom Metrics & Instrumentasi

### Business Metrics

```go
// pkg/metrics/business.go
// Business metrics = metrics yang bermakna dari sudut pandang bisnis
// Berbeda dari technical metrics (latency, error rate)

var (
    // Jumlah order yang dibuat
    OrdersCreatedTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Namespace: "ecommerce",
            Subsystem: "business",
            Name:      "orders_created_total",
            Help:      "Total orders created",
        },
        []string{"status"}, // success, failed
    )

    // Revenue total (dalam rupiah)
    RevenueTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Namespace: "ecommerce",
            Subsystem: "business",
            Name:      "revenue_total_idr",
            Help:      "Total revenue in IDR",
        },
        []string{"currency"},
    )

    // Active orders saat ini (gauge)
    ActiveOrders = promauto.NewGauge(
        prometheus.GaugeOpts{
            Namespace: "ecommerce",
            Subsystem: "business",
            Name:      "active_orders",
            Help:      "Number of orders currently in progress",
        },
    )

    // Cart abandonment
    CartAbandonedTotal = promauto.NewCounter(
        prometheus.CounterOpts{
            Namespace: "ecommerce",
            Subsystem: "business",
            Name:      "cart_abandoned_total",
            Help:      "Total abandoned carts",
        },
    )

    // User registration
    UsersRegisteredTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Namespace: "ecommerce",
            Subsystem: "business",
            Name:      "users_registered_total",
            Help:      "Total users registered",
        },
        []string{"source"}, // web, mobile, api
    )
)

// Penggunaan di use case
func (uc *confirmOrderUseCase) Execute(ctx context.Context, input Input) error {
    err := uc.doConfirm(ctx, input)

    if err != nil {
        metrics.OrdersCreatedTotal.WithLabelValues("failed").Inc()
        return err
    }

    // Record business metrics
    metrics.OrdersCreatedTotal.WithLabelValues("success").Inc()
    metrics.RevenueTotal.WithLabelValues("IDR").Add(order.TotalAmount)
    metrics.ActiveOrders.Inc()

    return nil
}
```

### gRPC Metrics

```go
// pkg/metrics/grpc_middleware.go
package metrics

import (
    "context"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
)

var (
    GRPCRequestsTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Namespace: "ecommerce",
            Subsystem: "grpc",
            Name:      "requests_total",
            Help:      "Total gRPC requests",
        },
        []string{"service", "method", "code"},
    )

    GRPCRequestDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Namespace: "ecommerce",
            Subsystem: "grpc",
            Name:      "request_duration_seconds",
            Help:      "gRPC request duration",
            Buckets:   prometheus.DefBuckets,
        },
        []string{"service", "method"},
    )
)

// UnaryServerMetricsInterceptor mencatat metrics untuk setiap gRPC call
func UnaryServerMetricsInterceptor(serviceName string) grpc.UnaryServerInterceptor {
    return func(
        ctx context.Context,
        req interface{},
        info *grpc.UnaryServerInfo,
        handler grpc.UnaryHandler,
    ) (interface{}, error) {
        start := time.Now()

        resp, err := handler(ctx, req)

        code := codes.OK
        if err != nil {
            code = status.Code(err)
        }

        GRPCRequestsTotal.WithLabelValues(
            serviceName,
            info.FullMethod,
            code.String(),
        ).Inc()

        GRPCRequestDuration.WithLabelValues(
            serviceName,
            info.FullMethod,
        ).Observe(time.Since(start).Seconds())

        return resp, err
    }
}
```

### Kafka Consumer Metrics

```go
// pkg/metrics/kafka_metrics.go

var (
    KafkaMessagesConsumed = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Namespace: "ecommerce",
            Subsystem: "kafka",
            Name:      "messages_consumed_total",
            Help:      "Total Kafka messages consumed",
        },
        []string{"topic", "group_id", "status"}, // status: success, error, dlq
    )

    KafkaConsumerLag = promauto.NewGaugeVec(
        prometheus.GaugeOpts{
            Namespace: "ecommerce",
            Subsystem: "kafka",
            Name:      "consumer_lag",
            Help:      "Kafka consumer lag (messages behind)",
        },
        []string{"topic", "partition", "group_id"},
    )

    KafkaMessageProcessingDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Namespace: "ecommerce",
            Subsystem: "kafka",
            Name:      "message_processing_duration_seconds",
            Help:      "Time to process a Kafka message",
            Buckets:   prometheus.DefBuckets,
        },
        []string{"topic", "event_type"},
    )
)
```

### Go Runtime Metrics — Built-in dari prometheus/client_golang

```go
// pkg/metrics/runtime.go
// prometheus/client_golang sudah otomatis expose runtime metrics
// Cukup import:
import _ "github.com/prometheus/client_golang/prometheus/collectors"

// Atau register manual:
func RegisterRuntimeMetrics() {
    prometheus.MustRegister(
        collectors.NewGoCollector(
            collectors.WithGoCollectorRuntimeMetrics(
                collectors.GoRuntimeMetricsRule{Matcher: regexp.MustCompile(`/.*`)},
            ),
        ),
        collectors.NewProcessCollector(collectors.ProcessCollectorOpts{}),
    )
}

// Metrics yang otomatis tersedia setelah register:
// go_goroutines              → jumlah goroutine aktif
// go_gc_duration_seconds     → histogram durasi GC pause
// go_memstats_alloc_bytes    → heap bytes yang sedang dipakai
// go_memstats_sys_bytes      → total memori dari OS
// go_memstats_gc_cpu_fraction → % CPU untuk GC
// process_resident_memory_bytes → RSS memory
// process_cpu_seconds_total  → total CPU seconds

// PromQL queries berguna:
// Goroutine count per service:
//   go_goroutines{service="order-service"}
//
// GC pause p99 (ms):
//   histogram_quantile(0.99, rate(go_gc_duration_seconds_bucket[5m])) * 1000
//
// Heap usage (MB):
//   go_memstats_alloc_bytes{service="auth-service"} / 1024 / 1024
//
// Memory growth rate (deteksi memory leak):
//   rate(go_memstats_alloc_bytes_total[5m])
```

### Alerting untuk Runtime Issues

```yaml
# monitoring/prometheus/alerts-runtime.yml
groups:
  - name: go_runtime
    rules:
      # Goroutine leak
      - alert: GoRoutineLeak
        expr: go_goroutines > 1000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Possible goroutine leak in {{ $labels.service }}"
          description: "Goroutine count is {{ $value }}, expected < 1000"

      # GC pressure
      - alert: HighGCPressure
        expr: go_memstats_gc_cpu_fraction > 0.05
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High GC pressure in {{ $labels.service }}"
          description: "GC using {{ $value | humanizePercentage }} of CPU"

      # Memory usage high
      - alert: HighHeapUsage
        expr: |
          go_memstats_alloc_bytes / go_memstats_sys_bytes > 0.8
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High heap usage in {{ $labels.service }}"
```


### 🏋️ Latihan 9.5

1. Tambahkan **business metrics** ke Order Service: `orders_created_total` (per status), `revenue_total_idr`, `active_orders` (gauge). Verifikasi metrics muncul di Prometheus setelah beberapa order dibuat.
2. Buat **gRPC metrics interceptor** dan pasang ke Product Service. Buat query PromQL: "berapa % gRPC calls ke Product Service yang succeed?"
3. Implementasikan **cache metrics**: tambahkan `cache_hits_total` dan `cache_misses_total` ke Redis cache layer. Buat alert rule: "kirim alert jika cache hit rate < 70% selama 5 menit".

---

## 📦 Modul 9.6 — Grafana Dashboard

### Setup Grafana

```yaml
# docker-compose.monitoring.yml — tambahkan Grafana
services:
  grafana:
    image: grafana/grafana:10.2.0
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_USER: admin
      GF_SECURITY_ADMIN_PASSWORD: adminpassword
      GF_USERS_ALLOW_SIGN_UP: "false"
    volumes:
      - grafana-data:/var/lib/grafana
      - ./monitoring/grafana/provisioning:/etc/grafana/provisioning
      - ./monitoring/grafana/dashboards:/var/lib/grafana/dashboards
    depends_on:
      - prometheus
    networks:
      - monitoring

volumes:
  grafana-data:
```

### Datasource Provisioning

```yaml
# monitoring/grafana/provisioning/datasources/prometheus.yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
```

### Dashboard as Code

```json
// monitoring/grafana/dashboards/ecommerce-overview.json
{
  "title": "E-Commerce Overview",
  "panels": [
    {
      "title": "Request Rate",
      "type": "stat",
      "targets": [{
        "expr": "sum(rate(ecommerce_http_requests_total[5m])) by (service)"
      }]
    },
    {
      "title": "Error Rate (%)",
      "type": "gauge",
      "targets": [{
        "expr": "sum(rate(ecommerce_http_requests_total{status_code=~\"5..\"}[5m])) / sum(rate(ecommerce_http_requests_total[5m])) * 100"
      }]
    },
    {
      "title": "p99 Latency (ms)",
      "type": "graph",
      "targets": [{
        "expr": "histogram_quantile(0.99, sum(rate(ecommerce_http_request_duration_seconds_bucket[5m])) by (service, le)) * 1000"
      }]
    },
    {
      "title": "Active Orders",
      "type": "stat",
      "targets": [{
        "expr": "ecommerce_business_active_orders"
      }]
    }
  ]
}
```

### Penting: Cardinality

```
CARDINALITY = jumlah kombinasi label unik
Cardinality tinggi = memori besar = Prometheus lambat atau crash!

❌ SALAH: user_id sebagai label (jutaan user → jutaan series)
counter.WithLabelValues(userID, path).Inc()

✅ BENAR: gunakan label yang cardinalitynya rendah
counter.WithLabelValues(role, path).Inc()  // role: user/admin = 2 values

Aturan: label harus punya max ~50 values unik
Jangan pakai: user_id, email, IP address, UUID sebagai labels
Boleh pakai: service, method, status_code, endpoint (normalized)
```

### 🏋️ Latihan 9.6

1. Setup Grafana dengan Prometheus datasource. Buat dashboard dengan 4 panel: (a) Request rate per service, (b) Error rate %, (c) p50/p95/p99 latency, (d) Active orders gauge. Export dashboard sebagai JSON.
2. Buat **RED Method dashboard** (Rate, Errors, Duration) untuk setiap service. Buat annotation yang menampilkan deployment events (kapan service di-redeploy).
3. Buat **SLO Dashboard**: panel yang menampilkan error budget burn rate. Jika error budget habis > 5% dalam 1 jam, tampilkan warning.

---

## 📦 Modul 9.7 — Distributed Tracing dengan OpenTelemetry

### Apa itu Distributed Tracing?

```
Bayangkan request dari client melewati 5 service:
  Client → Gateway → Auth → Order → Product → DB

Tanpa tracing: 
  "Order request butuh 2 detik" — tidak tahu di mana lambatnya

Dengan tracing:
  Gateway:   50ms
  Auth:      30ms (validasi JWT)
  Order:     1200ms  ← ini yang lambat!
    ├── DB query: 1150ms  ← ini masalahnya
    └── gRPC ke Product: 20ms
  Product:   20ms

TERMINOLOGI:
  Trace  = satu request end-to-end (punya trace_id unik)
  Span   = satu operasi dalam trace (punya span_id, parent_id)
  Baggage = data yang dibawa sepanjang trace

Visualisasi (Gantt chart):
  Gateway    [===================================================] 2000ms
  Auth         [=========]                                        30ms
  Order                   [=====================================] 1200ms
  DB query                  [===================================] 1150ms
  Product              [====]                                     20ms
```

### Setup OpenTelemetry

```bash
go get go.opentelemetry.io/otel
go get go.opentelemetry.io/otel/trace
go get go.opentelemetry.io/otel/sdk/trace
go get go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc
go get go.opentelemetry.io/contrib/instrumentation/github.com/gin-gonic/gin/otelgin
go get go.opentelemetry.io/contrib/instrumentation/google.golang.org/grpc/otelgrpc
```

```go
// pkg/tracing/tracing.go
package tracing

import (
    "context"
    "fmt"

    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc"
    "go.opentelemetry.io/otel/propagation"
    "go.opentelemetry.io/otel/sdk/resource"
    sdktrace "go.opentelemetry.io/otel/sdk/trace"
    semconv "go.opentelemetry.io/otel/semconv/v1.21.0"
    "go.opentelemetry.io/otel/trace"
)

// InitTracer menginisialisasi OpenTelemetry tracer
// Mengirim traces ke collector (Jaeger, OTLP endpoint)
func InitTracer(ctx context.Context, serviceName, version, collectorEndpoint string) (func(), error) {
    // Exporter: kirim traces ke OTLP collector
    exporter, err := otlptracegrpc.New(ctx,
        otlptracegrpc.WithEndpoint(collectorEndpoint),
        otlptracegrpc.WithInsecure(), // tanpa TLS untuk dev
    )
    if err != nil {
        return nil, fmt.Errorf("create OTLP exporter: %w", err)
    }

    // Resource: metadata tentang service ini
    res, err := resource.New(ctx,
        resource.WithAttributes(
            semconv.ServiceName(serviceName),
            semconv.ServiceVersion(version),
        ),
    )
    if err != nil {
        return nil, fmt.Errorf("create resource: %w", err)
    }

    // Trace Provider
    tp := sdktrace.NewTracerProvider(
        sdktrace.WithBatcher(exporter),  // batch export untuk performa
        sdktrace.WithResource(res),
        // Sample rate: 100% untuk dev, 10% untuk production
        sdktrace.WithSampler(sdktrace.TraceIDRatioBased(1.0)),
    )

    // Set sebagai global provider
    otel.SetTracerProvider(tp)

    // Set propagator untuk context propagation antar service
    // W3C Trace Context adalah standar industri
    otel.SetTextMapPropagator(propagation.NewCompositeTextMapPropagator(
        propagation.TraceContext{},
        propagation.Baggage{},
    ))

    // Cleanup function — panggil saat shutdown
    cleanup := func() {
        if err := tp.Shutdown(context.Background()); err != nil {
            fmt.Printf("tracing shutdown error: %v\n", err)
        }
    }

    return cleanup, nil
}

// Tracer mendapatkan tracer untuk service
func Tracer(name string) trace.Tracer {
    return otel.Tracer(name)
}
```

### Instrumentasi Manual

```go
// Cara membuat spans secara manual
import (
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/codes"
)

func (uc *confirmOrderUseCase) Execute(ctx context.Context, input Input) error {
    // Buat span untuk use case ini
    tracer := otel.Tracer("order-service")
    ctx, span := tracer.Start(ctx, "ConfirmOrderUseCase.Execute")
    defer span.End()

    // Tambahkan attributes yang berguna untuk debug
    span.SetAttributes(
        attribute.Int64("order.id", int64(input.OrderID)),
        attribute.String("payment.id", input.PaymentID),
    )

    // Step 1: Load order
    ctx, dbSpan := tracer.Start(ctx, "OrderRepository.FindByID")
    order, err := uc.orderRepo.FindByID(ctx, input.OrderID)
    dbSpan.End()

    if err != nil {
        // Record error ke span
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
        return err
    }

    // Step 2: gRPC call ke Product Service
    ctx, grpcSpan := tracer.Start(ctx, "ProductClient.UpdateStock")
    err = uc.productClient.UpdateStock(ctx, /* ... */)
    grpcSpan.End()
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
        return err
    }

    // Step 3: Confirm order
    if err := order.Confirm(input.PaymentID); err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
        return err
    }

    span.SetStatus(codes.Ok, "order confirmed")
    return uc.orderRepo.Save(ctx, order)
}
```

### 🏋️ Latihan 9.7

1. Setup OpenTelemetry di Auth Service. Buat span untuk setiap use case (Register, Login) dengan attributes yang relevan. Verifikasi span muncul di terminal log (gunakan stdout exporter dulu).
2. Instrumentasi semua repository methods (FindByID, Save, dll) dengan spans. Tambahkan attribute `db.operation`, `db.table`, `db.statement` (tanpa value untuk security). Verifikasi database spans muncul sebagai child dari use case span.
3. Buat helper `StartSpan(ctx, operationName, attrs...) (context.Context, trace.Span)` yang menyederhanakan pembuatan span. Gunakan di semua service.

---

## 📦 Modul 9.8 — Jaeger Integration

### Setup Jaeger

```yaml
# docker-compose.monitoring.yml — tambahkan Jaeger
services:
  jaeger:
    image: jaegertracing/all-in-one:1.51
    container_name: jaeger
    ports:
      - "16686:16686"  # Jaeger UI
      - "4317:4317"    # OTLP gRPC collector
      - "4318:4318"    # OTLP HTTP collector
    environment:
      COLLECTOR_OTLP_ENABLED: "true"
      SPAN_STORAGE_TYPE: memory  # untuk dev; pakai elasticsearch/cassandra di prod
    networks:
      - monitoring
```

```go
// Konfigurasi exporter ke Jaeger
// Di main.go atau config:
collectorEndpoint := "jaeger:4317"  // untuk container
// collectorEndpoint := "localhost:4317"  // untuk local dev

cleanup, err := tracing.InitTracer(ctx,
    "order-service",
    "1.0.0",
    collectorEndpoint,
)
if err != nil {
    log.Fatal("init tracer", zap.Error(err))
}
defer cleanup()
```

### Otomatis Instrumentasi dengan Middleware

```go
// Gin auto-instrumentation (tidak perlu buat span manual per handler)
import "go.opentelemetry.io/contrib/instrumentation/github.com/gin-gonic/gin/otelgin"

r := gin.New()
r.Use(otelgin.Middleware("order-service"))
// Sekarang setiap request otomatis punya span!

// gRPC auto-instrumentation
import "go.opentelemetry.io/contrib/instrumentation/google.golang.org/grpc/otelgrpc"

// Server
grpcServer := grpc.NewServer(
    grpc.StatsHandler(otelgrpc.NewServerHandler()),
)

// Client
conn, err := grpc.Dial(addr,
    grpc.WithStatsHandler(otelgrpc.NewClientHandler()),
)
```

### Searching dan Filtering di Jaeger UI

```
Jaeger UI (http://localhost:16686):

1. Search traces:
   Service: order-service
   Operation: POST /api/v1/orders
   Min Duration: 1s  ← cari yang lambat
   
2. Trace detail view:
   - Timeline view (Gantt chart)
   - Setiap span dengan attributes
   - Error spans highlighted merah

3. Compare traces:
   - Bandingkan trace yang lambat vs normal
   - Identifikasi span mana yang berbeda

Useful PromQL untuk find traces:
   - Error rate tinggi → temukan trace yang error
   - Latency spike → temukan trace yang lambat
```

### 🏋️ Latihan 9.8

1. Setup Jaeger dengan docker-compose. Pastikan traces dari Auth Service (login endpoint) muncul di Jaeger UI. Buat request 10x dan verify semua traces ter-capture.
2. Buat **slow query simulation**: tambahkan `time.Sleep(2*time.Second)` di salah satu repository method. Cari trace yang lambat di Jaeger UI. Identifikasi span mana yang menjadi bottleneck.
3. Buat **error trace**: tambahkan artificial error di Product Service gRPC handler. Cari traces dengan error di Jaeger UI. Verifikasi error message dan stack trace muncul di span attributes.

---

## 📦 Modul 9.9 — Trace Propagation di Microservices

### W3C Trace Context Standard

```
Trace Context dikirim via HTTP headers:
  traceparent: 00-{trace-id}-{parent-span-id}-{flags}
  tracestate:  vendor-specific info

Contoh:
  traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
               │  └──────── trace ID (128 bit) ──────┘ └─span ID─┘ │
               version                                            sampling flag

Setiap service:
  1. Baca traceparent dari incoming request
  2. Buat child span dengan trace ID yang sama
  3. Inject traceparent ke outgoing request/metadata
```

### HTTP Trace Propagation

```go
// Otomatis dengan otelgin — tidak perlu kode tambahan!
// otelgin.Middleware sudah handle extract dan inject

// Untuk HTTP client yang keluar:
import (
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/propagation"
)

func makeHTTPRequestWithTrace(ctx context.Context, url string) (*http.Response, error) {
    req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)

    // Inject trace context ke HTTP headers
    otel.GetTextMapPropagator().Inject(ctx, propagation.HeaderCarrier(req.Header))
    // Header traceparent otomatis ditambahkan!

    return http.DefaultClient.Do(req)
}
```

### gRPC Trace Propagation

```go
// Otomatis dengan otelgrpc — sudah handle extract dan inject
// Pastikan pasang di SEMUA service (server dan client)

// Di server:
grpc.NewServer(grpc.StatsHandler(otelgrpc.NewServerHandler()))

// Di client:
grpc.Dial(addr, grpc.WithStatsHandler(otelgrpc.NewClientHandler()))

// PENTING: context harus di-pass dengan benar antar function!
// Jika context hilang, trace chain terputus

// ❌ SALAH: buat context baru
func processOrder(ctx context.Context) {
    // ctx punya trace info
    go func() {
        newCtx := context.Background() // ← trace hilang!
        callProductService(newCtx)
    }()
}

// ✅ BENAR: teruskan context asli
func processOrder(ctx context.Context) {
    go func() {
        callProductService(ctx) // ← trace tersambung
    }()
}
```

### Baggage — Data yang Dibawa Sepanjang Trace

```go
// Baggage berguna untuk meneruskan info seperti user_id tanpa harus
// kirim ulang di setiap request

import (
    "go.opentelemetry.io/otel/baggage"
)

// Set baggage di gateway
func setUserBaggage(ctx context.Context, userID string) context.Context {
    userMember, _ := baggage.NewMember("user_id", userID)
    b, _ := baggage.New(userMember)
    return baggage.ContextWithBaggage(ctx, b)
}

// Get baggage di downstream service
func getUserFromBaggage(ctx context.Context) string {
    b := baggage.FromContext(ctx)
    return b.Member("user_id").Value()
}
```

### 🏋️ Latihan 9.9

1. Verifikasi bahwa trace_id konsisten saat request melewati API Gateway → Auth Service → Order Service. Buka Jaeger UI dan cari trace yang span-nya melewati minimal 3 service.
2. Implementasikan **baggage propagation**: kirim `user_id` sebagai baggage dari API Gateway. Di setiap downstream service, extract baggage dan tambahkan ke span attributes. Verifikasi di Jaeger bahwa user_id muncul di semua spans dalam trace yang sama.
3. Buat **trace-based alert**: tambahkan custom attribute `order.amount` ke span. Gunakan Jaeger query untuk mencari semua traces dengan `order.amount > 1000000`. Buat script yang periodic query Jaeger dan alert jika ada high-value order dengan error.

---

## 📦 Modul 9.10 — Alerting & SLO/SLI/SLA

### SLA, SLO, SLI

```
SLA (Service Level Agreement):
  Perjanjian dengan customer/stakeholder
  "99.9% uptime per bulan" → 43.8 menit downtime diizinkan
  Melanggar SLA = penalty (uang, reputasi)

SLO (Service Level Objective):
  Target internal yang lebih ketat dari SLA
  "99.95% uptime" → buffer sebelum langgar SLA
  Biasanya 10-20% lebih ketat dari SLA

SLI (Service Level Indicator):
  Metric aktual yang diukur
  "Uptime selama 30 hari terakhir: 99.97%"
  Yang diukur, bukan yang ditargetkan

ERROR BUDGET:
  Berapa banyak "downtime" yang masih diizinkan sebelum langgar SLO
  SLO 99.9% = 0.1% error budget = 43.8 menit/bulan
  Jika error budget habis → stop deployment, fokus reliability
```

### Prometheus Alerting Rules

```yaml
# monitoring/prometheus/alerts.yml
groups:
  - name: ecommerce.critical
    rules:
      # Alert: Error rate tinggi
      - alert: HighErrorRate
        expr: |
          sum(rate(ecommerce_http_requests_total{status_code=~"5.."}[5m]))
          /
          sum(rate(ecommerce_http_requests_total[5m])) > 0.05
        for: 2m  # harus true selama 2 menit sebelum alert
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }} over the last 5 minutes"

      # Alert: Latency tinggi
      - alert: HighLatency
        expr: |
          histogram_quantile(0.99,
            sum(rate(ecommerce_http_request_duration_seconds_bucket[5m])) by (le, service)
          ) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High p99 latency in {{ $labels.service }}"
          description: "p99 latency is {{ $value | humanizeDuration }}"

      # Alert: Service down
      - alert: ServiceDown
        expr: up{job=~".*-service"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.job }} is down"
          description: "Service has been down for more than 1 minute"

  - name: ecommerce.business
    rules:
      # Alert: Tidak ada order dalam 1 jam (anomaly detection)
      - alert: NoOrdersCreated
        expr: increase(ecommerce_business_orders_created_total[1h]) == 0
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "No orders created in the last hour"
          description: "This might indicate checkout is broken"
```

### SLO Tracking dengan Prometheus

```go
// pkg/metrics/slo.go
// Track SLO compliance secara real-time

var (
    // Request yang "good" = < 200ms dan 2xx/3xx
    SLOGoodRequests = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Namespace: "ecommerce",
            Subsystem: "slo",
            Name:      "good_requests_total",
            Help:      "Requests that meet SLO criteria",
        },
        []string{"service"},
    )

    SLOTotalRequests = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Namespace: "ecommerce",
            Subsystem: "slo",
            Name:      "total_requests_total",
            Help:      "Total requests for SLO calculation",
        },
        []string{"service"},
    )
)

// Middleware yang track SLO compliance
func SLOMiddleware(serviceName string, latencyThreshold time.Duration) gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()
        c.Next()

        duration := time.Since(start)
        statusCode := c.Writer.Status()
        isError := statusCode >= 500

        SLOTotalRequests.WithLabelValues(serviceName).Inc()

        // "Good request" = tidak error DAN latency di bawah threshold
        if !isError && duration < latencyThreshold {
            SLOGoodRequests.WithLabelValues(serviceName).Inc()
        }
    }
}
```

```
PromQL untuk SLO dashboard:
  
# Current SLO compliance (last 30 days)
sum(increase(ecommerce_slo_good_requests_total[30d]))
/
sum(increase(ecommerce_slo_total_requests_total[30d]))

# Error budget remaining
1 - (
  (1 - sum(increase(ecommerce_slo_good_requests_total[30d])) / sum(increase(ecommerce_slo_total_requests_total[30d])))
  /
  (1 - 0.999)  # 99.9% SLO target
)
```

### Alertmanager — Routing & Notification

```yaml
# monitoring/alertmanager/alertmanager.yml
global:
  resolve_timeout: 5m
  # SMTP untuk email
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'alerts@mycompany.com'
  smtp_auth_username: 'alerts@mycompany.com'
  smtp_auth_password: '{{ .SmtpPassword }}'

# Template untuk pesan notifikasi
templates:
  - '/etc/alertmanager/templates/*.tmpl'

# Routing tree: tentukan siapa yang dapat alert apa
route:
  # Default receiver
  receiver: 'ops-team-slack'
  group_by: ['alertname', 'service']
  group_wait: 30s      # tunggu 30s sebelum kirim (untuk grouping)
  group_interval: 5m   # kirim ulang setelah 5 menit jika masih firing
  repeat_interval: 4h  # kirim ulang setiap 4 jam jika masih firing

  routes:
    # Critical alerts → PagerDuty (langsung bangunkan on-call engineer!)
    - match:
        severity: critical
      receiver: pagerduty-critical
      continue: true  # kirim ke receiver berikutnya juga

    # Critical alerts → Slack #incidents
    - match:
        severity: critical
      receiver: slack-incidents

    # Warning alerts → Slack #alerts (tidak bangunkan orang)
    - match:
        severity: warning
      receiver: slack-alerts
      group_wait: 1m  # tunggu lebih lama untuk group

    # Business alerts → berbeda channel
    - match_re:
        alertname: 'NoOrders.*|HighCart.*'
      receiver: business-team-slack

receivers:
  - name: 'ops-team-slack'
    slack_configs:
      - api_url: '{{ .SlackWebhookURL }}'
        channel: '#ops-alerts'
        title: '[{{ .Status | toUpper }}] {{ .CommonLabels.alertname }}'
        text: |
          *Alert:* {{ .CommonLabels.alertname }}
          *Service:* {{ .CommonLabels.service }}
          *Severity:* {{ .CommonLabels.severity }}
          *Summary:* {{ .CommonAnnotations.summary }}
          *Description:* {{ .CommonAnnotations.description }}
          {{ range .Alerts }}
          *Details:*
          {{ range .Labels.SortedPairs }}  • *{{ .Name }}:* `{{ .Value }}`
          {{ end }}
          {{ end }}

  - name: 'slack-incidents'
    slack_configs:
      - api_url: '{{ .SlackWebhookURL }}'
        channel: '#incidents'
        color: '{{ if eq .Status "firing" }}danger{{ else }}good{{ end }}'

  - name: 'pagerduty-critical'
    pagerduty_configs:
      - routing_key: '{{ .PagerDutyIntegrationKey }}'
        description: '{{ .CommonAnnotations.summary }}'
        severity: '{{ .CommonLabels.severity }}'

  - name: 'business-team-slack'
    slack_configs:
      - api_url: '{{ .SlackWebhookURL }}'
        channel: '#business-alerts'

# Silencing: suppress alerts yang sudah diketahui
inhibit_rules:
  # Jika service down (critical), suppress semua alert yang lebih kecil dari service tersebut
  - source_match:
      severity: 'critical'
      alertname: 'ServiceDown'
    target_match:
      severity: 'warning'
    equal: ['service']
```


### 🏋️ Latihan 9.10

1. Buat **Prometheus alerting rules** untuk: error rate > 5% selama 2 menit, p99 latency > 1 detik selama 5 menit, service down selama 1 menit. Test alert dengan stop salah satu service dan verify Prometheus menunjukkan "FIRING".
2. Implementasikan **SLO tracking** untuk Order Service dengan target 99.9% dan latency threshold 200ms. Buat dashboard Grafana yang menampilkan: current SLO compliance %, error budget remaining, burn rate.
3. Setup **Alertmanager**: buat konfigurasi yang kirim notifikasi ke Slack webhook (atau file sink untuk testing) saat alert firing. Verifikasi notifikasi masuk saat service dimatikan.

---

## 📦 Modul 9.11 — Profiling dengan pprof

### Apa itu Profiling?

```
Profiling = mengukur di mana program menghabiskan waktu dan memori

Types:
  CPU Profiling   → function mana yang paling banyak makan CPU?
  Memory Profiling → object mana yang paling banyak allocate?
  Goroutine       → berapa goroutine aktif? ada goroutine leak?
  Block           → goroutine mana yang paling sering blocked?
  Mutex           → contentions di mutex mana?

Kapan profiling:
  - Service lambat tapi tidak tahu kenapa
  - Memory usage terus naik (memory leak)
  - Goroutine terus bertambah (goroutine leak)
  - Sebelum dan sesudah optimization
```

### Setup pprof Endpoint

```go
// HANYA aktifkan di development/staging!
// Jangan expose pprof ke production publik (security risk)

import (
    _ "net/http/pprof" // import untuk side effect: register /debug/pprof endpoints
    "net/http"
)

// Option 1: Server pprof di port terpisah (lebih aman)
func startPprofServer(port int) {
    go func() {
        addr := fmt.Sprintf("localhost:%d", port)
        fmt.Printf("[pprof] listening on http://%s/debug/pprof/\n", addr)
        if err := http.ListenAndServe(addr, nil); err != nil {
            fmt.Printf("[pprof] error: %v\n", err)
        }
    }()
}

// Option 2: Expose via Gin (dengan auth middleware!)
func setupPprofRoutes(r *gin.Engine, authMW gin.HandlerFunc) {
    pprof := r.Group("/debug/pprof")
    pprof.Use(authMW) // WAJIB auth!
    pprof.GET("/", gin.WrapF(http.HandlerFunc(pprof.Index)))
    pprof.GET("/cmdline", gin.WrapF(pprof.Cmdline))
    pprof.GET("/profile", gin.WrapF(pprof.Profile))
    pprof.GET("/symbol", gin.WrapF(pprof.Symbol))
    pprof.GET("/trace", gin.WrapF(pprof.Trace))
}
```

### Menggunakan go tool pprof

```bash
# 1. CPU profiling selama 30 detik
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# 2. Memory profiling
go tool pprof http://localhost:6060/debug/pprof/heap

# 3. Goroutine profiling
go tool pprof http://localhost:6060/debug/pprof/goroutine

# Di dalam pprof interactive mode:
(pprof) top10          # top 10 functions by usage
(pprof) web            # buka flame graph di browser
(pprof) list funcName  # detail untuk satu fungsi
(pprof) svg            # generate SVG call graph

# Flame graph (lebih visual)
go tool pprof -http=:8888 http://localhost:6060/debug/pprof/profile?seconds=30
# Buka http://localhost:8888
```

### Continuous Profiling

```go
// pkg/profiling/profiler.go
// Upload profiles ke storage untuk historical analysis

import (
    "github.com/grafana/pyroscope-go"
)

func StartContinuousProfiling(serverAddr, appName string) error {
    _, err := pyroscope.Start(pyroscope.Config{
        ApplicationName: appName,
        ServerAddress:   serverAddr,
        // Profile semua tipe
        ProfileTypes: []pyroscope.ProfileType{
            pyroscope.ProfileCPU,
            pyroscope.ProfileAllocObjects,
            pyroscope.ProfileAllocSpace,
            pyroscope.ProfileInuseObjects,
            pyroscope.ProfileInuseSpace,
        },
    })
    return err
}
```

### 🏋️ Latihan 9.11

1. Aktifkan pprof di port terpisah (6060) untuk Auth Service. Lakukan load test dengan `hey -n 10000 -c 50 http://localhost:8001/auth/me`. Ambil CPU profile dan identifikasi fungsi yang paling banyak makan CPU.
2. Buat **goroutine leak test**: buat handler yang start goroutine tapi tidak stop. Ambil goroutine profile sebelum dan sesudah 100 requests. Verifikasi goroutine terus bertambah. Fix leak dan verify goroutine count stabil.
3. Buat **memory benchmark**: profile heap sebelum dan sesudah optimasi sederhana (misal: reuse buffer, kurangi allocasi). Gunakan `go tool pprof` untuk compare dua profile dan tunjukkan pengurangan allocasi.

---

## 📦 Modul 9.12 — Production Readiness Checklist

### Checklist Lengkap

```
✅ OBSERVABILITY
  [ ] Structured logging (Zap JSON) di semua service
  [ ] Log levels dikonfigurasi dengan benar (WARN di prod)
  [ ] Request ID dipropagasi di semua logs
  [ ] Sensitive data tidak ada di logs
  [ ] Prometheus metrics di semua service (/metrics endpoint)
  [ ] HTTP, gRPC, database, dan business metrics tersedia
  [ ] Distributed tracing (OpenTelemetry) aktif
  [ ] Jaeger atau Tempo menerima traces
  [ ] Trace propagation bekerja antar service
  [ ] Grafana dashboard untuk setiap service

✅ RELIABILITY
  [ ] Health checks: liveness, readiness, startup
  [ ] Graceful shutdown (handle SIGTERM, drain in-flight requests)
  [ ] Circuit breaker untuk semua downstream calls
  [ ] Retry dengan exponential backoff
  [ ] Timeout di setiap external call
  [ ] Dead Letter Queue untuk message processing
  [ ] Database connection pool configured

✅ SECURITY
  [ ] Service tidak berjalan sebagai root
  [ ] Secrets tidak di-commit, tidak di-log
  [ ] JWT validated di gateway
  [ ] Rate limiting aktif
  [ ] CORS configured benar
  [ ] Dependency vulnerability scan (govulncheck)
  [ ] HTTPS/TLS untuk external traffic

✅ PERFORMANCE
  [ ] Database queries dioptimasi (EXPLAIN ANALYZE)
  [ ] Database indexes untuk semua query yang sering
  [ ] Redis caching untuk data yang sering dibaca
  [ ] Connection pooling dikonfigurasi
  [ ] Response compression aktif (gzip)
  [ ] Static assets served dari CDN

✅ DEPLOYMENT
  [ ] Docker image: multi-stage, non-root, minimal size
  [ ] Health check di Dockerfile / docker-compose
  [ ] Kubernetes resources (requests + limits) dikonfigurasi
  [ ] HPA configured untuk variable load
  [ ] Rolling update strategy (maxUnavailable, maxSurge)
  [ ] Startup probe untuk slow-starting services
  [ ] PodDisruptionBudget untuk HA

✅ CONFIGURATION
  [ ] Semua config via environment variables
  [ ] Secrets via Kubernetes Secrets atau Vault
  [ ] Config validation saat startup
  [ ] Feature flags tersedia untuk gradual rollout

✅ TESTING
  [ ] Unit test coverage > 80% untuk business logic
  [ ] Integration tests untuk critical paths
  [ ] Contract tests untuk event schemas
  [ ] Load test baseline established
  [ ] Chaos engineering experiments documented
```

### Startup Validation Comprehensive

```go
// cmd/api/startup.go
package main

func runStartupChecks(ctx context.Context, cfg *config.Config, db *gorm.DB) error {
    checks := []struct {
        name string
        fn   func() error
    }{
        {
            "database connectivity",
            func() error {
                sqlDB, err := db.DB()
                if err != nil {
                    return err
                }
                return sqlDB.PingContext(ctx)
            },
        },
        {
            "database migrations",
            func() error {
                // Verifikasi schema version sesuai
                var version string
                return db.Raw("SELECT version FROM schema_migrations ORDER BY version DESC LIMIT 1").Scan(&version).Error
            },
        },
        {
            "jwt secret length",
            func() error {
                if len(cfg.JWT.Secret) < 32 {
                    return fmt.Errorf("JWT_SECRET too short: minimum 32 characters")
                }
                return nil
            },
        },
        {
            "downstream services reachable",
            func() error {
                // Cek apakah service yang dibutuhkan bisa dijangkau
                return pingService(ctx, cfg.Services.ProductServiceGRPC)
            },
        },
    }

    for _, check := range checks {
        if err := check.fn(); err != nil {
            return fmt.Errorf("startup check '%s' failed: %w", check.name, err)
        }
        fmt.Printf("✅ Startup check passed: %s\n", check.name)
    }

    return nil
}
```

### 🏋️ Latihan 9.12

1. Lengkapi **Production Readiness Checklist** untuk Auth Service: buat script `scripts/check-production-ready.sh` yang otomatis verifikasi setiap item di checklist (test endpoint health, test metrics endpoint, test /debug/pprof tidak accessible dari luar, dll).
2. Buat **startup validation** yang komprehensif: cek database connectivity, migration version up-to-date, required env vars ada, downstream services reachable. Service harus exit dengan pesan yang jelas jika ada check yang gagal.
3. Dokumentasikan **runbook** untuk 5 failure scenarios paling umum: (a) service OOM killed, (b) database connection pool exhausted, (c) high error rate, (d) slow response time, (e) Kafka consumer lag tinggi.

---

## 📦 Modul 9.13 — Full Observability Stack

### Arsitektur Stack Lengkap

```
┌──────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                      │
│   Auth Service │ Product Service │ Order Service │ Gateway│
│   (Zap + OTel metrics + OTel traces)                     │
└──────────────────────────────────────────────────────────┘
         │ logs (stdout)    │ metrics /metrics    │ traces OTLP
         ▼                  ▼                     ▼
┌──────────────┐   ┌──────────────────┐   ┌────────────────┐
│  Log Stack   │   │  Metrics Stack   │   │  Trace Stack   │
│              │   │                  │   │                │
│  Promtail    │   │  Prometheus      │   │  Jaeger        │
│  (collect)   │   │  (collect+store) │   │  (store+UI)    │
│      │       │   │        │         │   │                │
│  Loki        │   │  Grafana         │   │                │
│  (store)     │   │  (visualize)     │   │                │
│      │       │   │        │         │   │                │
│  Grafana     │   └──────────────────┘   └────────────────┘
│  (query+viz) │                  │               │
└──────────────┘                  ▼               │
                         ┌────────────────────────┘
                         │    GRAFANA (unified)    │
                         │  Logs + Metrics + Traces │
                         └────────────────────────┘
```

### docker-compose.observability.yml Lengkap

```yaml
version: '3.8'

services:
  # === PROMETHEUS (Metrics) ===
  prometheus:
    image: prom/prometheus:v2.48.0
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=15d'
    networks: [monitoring]

  # === GRAFANA (Visualization) ===
  grafana:
    image: grafana/grafana:10.2.0
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: adminpassword
    volumes:
      - grafana-data:/var/lib/grafana
      - ./monitoring/grafana/provisioning:/etc/grafana/provisioning
    depends_on: [prometheus, jaeger, loki]
    networks: [monitoring]

  # === JAEGER (Traces) ===
  jaeger:
    image: jaegertracing/all-in-one:1.51
    ports:
      - "16686:16686"  # UI
      - "4317:4317"    # OTLP gRPC
      - "4318:4318"    # OTLP HTTP
    environment:
      COLLECTOR_OTLP_ENABLED: "true"
    networks: [monitoring]

  # === LOKI (Logs) ===
  loki:
    image: grafana/loki:2.9.0
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
    networks: [monitoring]

  # === PROMTAIL (Log Collector) ===
  promtail:
    image: grafana/promtail:2.9.0
    volumes:
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - ./monitoring/promtail/config.yml:/etc/promtail/config.yml
    command: -config.file=/etc/promtail/config.yml
    depends_on: [loki]
    networks: [monitoring]

  # === ALERTMANAGER (Notifications) ===
  alertmanager:
    image: prom/alertmanager:v0.26.0
    ports:
      - "9093:9093"
    volumes:
      - ./monitoring/alertmanager:/etc/alertmanager
    networks: [monitoring]

volumes:
  prometheus-data:
  grafana-data:

networks:
  monitoring:
    driver: bridge
```

### Promtail Config (Collect Container Logs)

```yaml
# monitoring/promtail/config.yml
server:
  http_listen_port: 9080

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: containers
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s
        filters:
          - name: label
            values: ["logging=true"]  # hanya container dengan label ini

    relabel_configs:
      - source_labels: ['__meta_docker_container_name']
        target_label: 'container'
      - source_labels: ['__meta_docker_container_label_com_docker_compose_service']
        target_label: 'service'

    pipeline_stages:
      - json:
          expressions:
            level: level
            msg: msg
            timestamp: timestamp
      - labels:
          level:
          service:
```

### Grafana Unified Dashboard

```
Di Grafana, kamu bisa buat satu dashboard yang menampilkan:

Row 1: Overview
  - Service request rate (Prometheus)
  - Error rate (Prometheus)
  - p99 latency (Prometheus)
  - Active alerts

Row 2: Logs (Loki panel)
  - Filter: {service="order-service", level="error"}
  - Log stream realtime

Row 3: Traces (Jaeger datasource)
  - Link ke Jaeger untuk traces dengan high latency
  - "Explore traces" button

Cross-linking:
  - Klik error rate spike → lihat logs saat itu → temukan error message
  - Klik log error → explore trace untuk request itu
  - Exemplars: metric data point yang linked ke trace
```

### 🏋️ Latihan 9.13

1. Setup **full observability stack** dengan docker-compose: Prometheus + Grafana + Jaeger + Loki + Promtail + Alertmanager. Verifikasi semua komponen healthy dan terhubung satu sama lain.
2. Buat **Unified Grafana Dashboard** yang menampilkan logs, metrics, dan link ke traces dalam satu view. Buat skenario: trigger error di Order Service → lihat error di logs → lihat error rate naik di metrics → klik ke Jaeger untuk trace detail.
3. Buat **Runbook yang terintegrasi**: dokumen yang berisi: gejala (dengan screenshot Grafana) → query untuk diagnosa → langkah perbaikan → verifikasi sukses. Gunakan untuk skenario "high error rate di checkout".

---

### Loki & LogQL — Query Language untuk Logs

```
LogQL adalah query language untuk Loki, mirip PromQL tapi untuk logs.

BASIC SYNTAX:
  {label="value"}                 → select streams dengan label
  {service="auth-service"}        → semua logs dari auth-service
  {service=~".*-service"}         → regex match
  {service="auth", level="error"} → multiple labels (AND)

LOG FILTER:
  {service="auth"} |= "login"          → mengandung "login"
  {service="auth"} != "debug"          → tidak mengandung "debug"
  {service="auth"} |~ "user.*failed"   → regex match
  {service="auth"} !~ "health.*check"  → tidak regex match

PARSER (untuk structured/JSON logs):
  {service="auth"} | json                        → parse JSON log
  {service="auth"} | json | level="error"        → filter setelah parse
  {service="auth"} | json | duration > 1s        → filter by duration
  {service="auth"} | logfmt                      → parse logfmt format

METRIC QUERIES:
  rate({service="auth"} |= "error" [5m])         → error rate per detik
  count_over_time({service="auth"}[1h])           → total logs dalam 1 jam
  sum(rate({service=~".*-service"}[5m])) by (service)  → per service

USEFUL QUERIES:
  # Semua error logs
  {service="order-service"} | json | level="error"

  # Logs dari request tertentu (correlate dengan trace)
  {service=~".*"} | json | request_id="abc-123"

  # Slow requests (latency > 1 detik)
  {service="api-gateway"} | json | latency_ms > 1000

  # Login failures dalam 5 menit terakhir
  {service="auth-service"} |= "authentication failed" | json
  | line_format "{{.timestamp}} user={{.user_id}} ip={{.ip}}"
```

```yaml
# monitoring/grafana/provisioning/datasources/loki.yaml
apiVersion: 1
datasources:
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    isDefault: false
    jsonData:
      derivedFields:
        # Auto-link dari log ke Jaeger trace
        - matcherRegex: "trace_id=(\\w+)"
          name: TraceID
          url: "http://jaeger:16686/trace/${__value.raw}"
          datasourceUid: jaeger-datasource
```


## 🎯 Review & Checkpoint Fase 9

### Konseptual
- [ ] Apa perbedaan logs, metrics, dan traces — kapan pakai masing-masing?
- [ ] Mengapa structured logging lebih baik dari fmt.Println?
- [ ] Apa itu cardinality dan mengapa berbahaya di Prometheus metrics?
- [ ] Jelaskan perbedaan Counter, Gauge, dan Histogram
- [ ] Apa itu SLO, SLI, SLA, dan error budget?
- [ ] Bagaimana trace propagation bekerja di microservices?
- [ ] Apa yang dimaksud dengan "observability" vs "monitoring"?

### Praktis
- [ ] Zap logger dengan JSON output dan request-scoped context berjalan
- [ ] Prometheus metrics tersedia di /metrics semua service
- [ ] Grafana dashboard menampilkan request rate, error rate, latency
- [ ] OpenTelemetry traces muncul di Jaeger
- [ ] Trace ID konsisten antar 3 service yang berbeda
- [ ] Alerting rule terkonfigurasi dan bisa di-test
- [ ] **Menyelesaikan project Full Observability Stack**

---

## 🎯 Project Akhir Fase 9

Kerjakan project berdasarkan PRD di: **`FASE-9-PRD-Observability-Stack.md`**

---

*Setelah selesai Fase 9, lanjut ke `FASE-10-Testing-Mastery.md`*

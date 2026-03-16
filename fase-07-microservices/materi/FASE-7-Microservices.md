# 📘 FASE 7: Microservices Architecture

> **Prasyarat:** Selesaikan Fase 4 (Clean Architecture) + Fase 5 (gRPC) + Fase 6 (DDD)
> **Durasi:** 4–6 minggu
> **Project Akhir:** E-Commerce Microservices System
> **Tujuan:** Membangun dan menghubungkan beberapa microservice yang saling berkomunikasi menggunakan pola arsitektur industri

---

## 🗂️ Daftar Modul

| # | Modul | Topik |
|---|-------|-------|
| 7.1 | Apa itu Microservices? | Monolith vs Microservices, kapan pakai apa |
| 7.2 | Docker untuk Go | Dockerfile, multi-stage build, optimasi image |
| 7.3 | Docker Compose | Multi-service, networking, volumes, health checks |
| 7.4 | Service Communication Patterns | Sync vs async, REST vs gRPC |
| 7.5 | API Gateway Pattern | Routing, auth, rate limiting, aggregation |
| 7.6 | Service Discovery | Env-based, DNS, health-aware load balancer |
| 7.7 | Inter-Service gRPC | Connection pool, deadline propagation, metadata |
| 7.8 | Saga Pattern | Choreography vs orchestration, compensating transactions |
| 7.9 | Circuit Breaker & Resilience | gobreaker, retry, bulkhead, fallback |
| 7.10 | Distributed Configuration | Viper multi-env, secrets, validation |
| 7.11 | Health Checks & Readiness | Liveness, readiness, dependency checks |
| 7.12 | Kubernetes Fundamentals | Pod, Deployment, Service, Ingress, HPA |
| 7.13 | Putting It All Together | Arsitektur final, checklist produksi |

---

## 📦 Modul 7.1 — Apa itu Microservices?

### Monolith vs Microservices

Sebelum memutuskan pakai microservices, kamu **harus benar-benar memahami trade-off**-nya. Banyak tim yang terburu-buru adopsi microservices dan justru mendapat kompleksitas tanpa manfaat.

```
MONOLITH
────────
Satu binary, satu database, satu deployment.

Keuntungan:
  + Mudah develop di awal
  + Transaction mudah (satu database)
  + Tidak ada network latency antar komponen
  + Simple debugging — semua ada di satu tempat
  + Deploy sederhana

Masalah saat sudah besar:
  - Deploy seluruh app meski ubah 1 baris kode
  - Satu bug crash seluruh sistem
  - Sulit scale komponen tertentu saja
  - Technology lock-in — semua harus satu bahasa/framework
  - Tim besar konflik terus di codebase yang sama


MICROSERVICES
─────────────
Banyak service kecil, masing-masing punya database dan deployment sendiri.

Keuntungan:
  + Deploy independent per service
  + Scale per service sesuai kebutuhan (Product Service lebih besar dari Auth)
  + Isolasi kegagalan — Product Service down tidak crash Order Service
  + Technology flexibility — setiap service bisa beda bahasa
  + Tim bisa bekerja paralel tanpa banyak konflik

Masalah:
  - Network failures harus ditangani
  - Distributed transactions sangat rumit
  - Butuh infrastruktur matang (Docker, K8s, monitoring)
  - Debugging lebih sulit — request bisa melewati 5+ service
  - Lebih banyak overhead: Dockerfile, health checks, config tiap service
```

### Microservices secara Visual

```
Tanpa Microservices (Monolith):

 ┌──────────────────────────────────────────┐
 │  Auth + Product + Order + Notification   │
 │         ──────────────────               │
 │              PostgreSQL                  │
 └──────────────────────────────────────────┘
        Deploy semua sekaligus

Dengan Microservices:

 ┌────────────┐  ┌─────────────┐  ┌──────────────┐
 │Auth Service│  │Prod. Service│  │Order Service │
 │:8001       │  │:8002        │  │:8003         │
 │PostgreSQL  │  │PostgreSQL   │  │PostgreSQL    │
 └──────┬─────┘  └──────┬──────┘  └──────┬───────┘
        │               │               │
        └───────────────┴───────────────┘
                 API Gateway :8080
                       │
                    Client
        Deploy setiap service secara independent
```

### Kapan Pakai Microservices?

```
JANGAN pakai microservices jika:
  - Tim kecil (< 5 developer)
  - Produk masih early stage, belum product-market fit
  - Domain belum dipahami dengan baik
  - "Pakai karena Netflix pakai" — bukan alasan yang valid

PERTIMBANGKAN microservices jika:
  - Tim besar, deploy monolith sudah terlalu lambat dan berisiko
  - Ada domain yang butuh scaling berbeda (search vs checkout)
  - Domain sudah dipahami sangat baik, DDD sudah diterapkan
  - Regulasi mengharuskan isolasi data tertentu
  - Technology requirement berbeda per domain
```

### Conway's Law

> *"Organizations which design systems are constrained to produce designs which are copies of their communication structures."*

Artinya: struktur microservices sebaiknya **mencerminkan struktur tim**.

```
Jangan:  1 tim kelola 10 service — overhead koordinasi besar
Sebaiknya:

  Tim Auth     (3 orang) → Auth Service
  Tim Catalog  (4 orang) → Product + Category Service
  Tim Commerce (5 orang) → Order + Payment Service
  Tim Delivery (3 orang) → Shipping + Notification Service
```

### Anti-Pattern: Distributed Monolith

```
DISTRIBUTED MONOLITH — yang terburuk dari keduanya!

Order Service → Product Service → User Service → Auth Service
  (sync call)    (sync call)      (sync call)

Masalah:
  - Semua service harus running agar satu request bisa selesai
  - Deploy masih saling bergantung (urutan harus benar)
  - Kegagalan satu service menjalar ke semua (cascading failure)
  - Dapat semua kompleksitas microservices, tapi NGGAK dapat benefitnya

Tanda-tanda kamu sudah masuk distributed monolith:
  - Service tidak bisa berdiri sendiri tanpa service lain
  - Shared database schema antar service
  - Deploy harus berurutan
  - Satu perubahan API butuh update banyak service sekaligus
```

### Prinsip Desain Microservices

```
1. Single Responsibility — satu service, satu domain yang jelas
2. Loose Coupling        — kurangi dependencies antar service
3. High Cohesion         — yang berubah bersama, ada di service yang sama
4. Design for Failure    — asumsikan semua external call BISA gagal
5. Decentralize Data     — setiap service punya database sendiri
6. Automate Everything   — CI/CD, testing, deployment semua otomatis
7. Observable            — logging, metrics, tracing dari hari pertama
```

### 🏋️ Latihan 7.1

1. Buat **Architecture Decision Record (ADR)** untuk sistem blog platform dari Fase 3: haruskah di-refactor ke microservices? Format: `## Status`, `## Context`, `## Decision`, `## Consequences`. Minimal 500 kata dengan argumen yang matang dari kedua sisi.
2. Desain **service decomposition** untuk sistem e-commerce dengan 6 domain: User, Product, Order, Payment, Shipping, Notification. Gambar diagram (ASCII art OK) dan tentukan: (a) batas setiap service, (b) tabel/data yang dimiliki masing-masing, (c) mode komunikasi (sync/async) antar service.
3. Identifikasi **3 anti-patterns** dan jelaskan cara memperbaikinya: (a) Order Service yang langsung query database Product Service, (b) Tabel `users` yang diakses oleh 3 service berbeda, (c) Chain sync calls 5 hop: Gateway→Order→Product→Inventory→Warehouse.

---

## 📦 Modul 7.2 — Docker untuk Go

### Mengapa Docker?

```
Tanpa Docker:
  Dev: "Sudah jalan di laptop saya!"
  Server: "Error: binary tidak kompatibel, library versi beda"
  → "works on my machine" syndrome

Dengan Docker:
  Dev:    docker build . && docker run myapp → JALAN
  Server: docker run myapp                  → SAMA PERSIS
  → Lingkungan identik di mana pun
```

### Multi-Stage Build — Cara Profesional

```dockerfile
# ================================================================
# Dockerfile — Production-ready untuk Go service
# Multi-stage: build stage (besar) + final stage (kecil)
# ================================================================

# ---- Stage 1: Builder ----
# Gunakan Go image HANYA untuk compile, tidak dibawa ke production
FROM golang:1.22-alpine AS builder

# git dan ca-certificates dibutuhkan saat build
RUN apk add --no-cache git ca-certificates tzdata

WORKDIR /app

# TRICK PENTING: Copy go.mod DULU, baru download dependencies
# Jika go.mod tidak berubah, Docker cache layer ini → build lebih cepat!
COPY go.mod go.sum ./
RUN go mod download && go mod verify

# Baru copy semua source code
COPY . .

# Build binary dengan optimasi produksi
# CGO_ENABLED=0  → pure Go binary, tidak butuh C library di runtime
# -ldflags "-w -s" → strip debug symbols (30-40% lebih kecil)
# -X → inject build metadata ke dalam binary
ARG VERSION=dev
ARG COMMIT_SHA=unknown
ARG BUILD_TIME

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags="-w -s \
              -X main.Version=${VERSION} \
              -X main.CommitSHA=${COMMIT_SHA} \
              -X main.BuildTime=${BUILD_TIME}" \
    -o /app/server \
    ./cmd/api/main.go

# ---- Stage 2: Final Image ----
# Gunakan minimal image — TANPA Go toolchain!
# distroless = hanya berisi binary + SSL certs + timezone data
# Ukuran: ~5MB vs ~800MB untuk golang:1.22
FROM gcr.io/distroless/static-debian12

# Copy dari builder stage
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /app/server /server

# Jalankan sebagai non-root user untuk keamanan
# UID 65532 = "nonroot" user di distroless
USER nonroot:nonroot

# Document port
EXPOSE 8080

# Health check — Docker akan monitor ini secara berkala
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD ["/server", "-healthcheck"]

ENTRYPOINT ["/server"]
```

### Jika Butuh Alpine (ada shell untuk debug)

```dockerfile
# Alternative: Alpine base image (lebih besar tapi ada sh)
FROM alpine:3.19

RUN apk add --no-cache ca-certificates tzdata

# Buat non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

COPY --from=builder /app/server /server

# Set ownership
RUN chown appuser:appgroup /server

USER appuser
EXPOSE 8080
ENTRYPOINT ["/server"]
```

### .dockerignore — Wajib Ada!

```dockerignore
# Mencegah file tidak perlu masuk ke build context
# Tanpa ini, Docker copy SEMUA file termasuk .git (ratusan MB!)

.git
.gitignore
.github/
*.md
docs/

# Test files (tidak butuh di production image)
*_test.go
test/
testdata/
coverage.out

# Development files
.env
.env.*
*.log
tmp/
air.toml

# Build artifacts
bin/
*.exe

# CI/CD configs
Dockerfile*
docker-compose*
.dockerignore
Makefile
*.sh
```

### Perbandingan Ukuran Image

```bash
# Jalankan ini untuk lihat perbedaan ukuran:
docker build --target builder -t myapp:builder .
docker build -t myapp:final .
docker images | grep myapp

# Expected results:
# golang:1.22 base image   → ~800MB  (JANGAN pakai untuk final!)
# golang:1.22-alpine       → ~250MB  (hanya untuk build stage)
# alpine:3.19 + binary     → ~12MB   (ada shell, bagus untuk debug)
# distroless/static        → ~5MB    (minimal, tanpa shell)
# scratch + binary         → ~2MB    (hanya binary, ekstrem)
```

### Makefile untuk Docker Workflow

```makefile
# Makefile

APP_NAME    = auth-service
VERSION     ?= $(shell git describe --tags --always --dirty 2>/dev/null || echo "dev")
COMMIT_SHA  ?= $(shell git rev-parse --short HEAD 2>/dev/null || echo "unknown")
BUILD_TIME  ?= $(shell date -u +"%Y-%m-%dT%H:%M:%SZ")
REGISTRY    ?= ghcr.io/mycompany

.PHONY: docker-build docker-push docker-run docker-stop docker-logs

# Build image dengan metadata
docker-build:
	docker build \
		--build-arg VERSION=$(VERSION) \
		--build-arg COMMIT_SHA=$(COMMIT_SHA) \
		--build-arg BUILD_TIME=$(BUILD_TIME) \
		-t $(REGISTRY)/$(APP_NAME):$(VERSION) \
		-t $(REGISTRY)/$(APP_NAME):latest \
		.
	@echo "Built: $(REGISTRY)/$(APP_NAME):$(VERSION)"
	@docker images $(REGISTRY)/$(APP_NAME) --format "table {{.Tag}}\t{{.Size}}"

# Push ke container registry
docker-push:
	docker push $(REGISTRY)/$(APP_NAME):$(VERSION)
	docker push $(REGISTRY)/$(APP_NAME):latest

# Jalankan lokal
docker-run:
	docker run -d \
		--name $(APP_NAME) \
		--restart unless-stopped \
		-p 8080:8080 \
		--env-file .env \
		$(REGISTRY)/$(APP_NAME):latest

docker-stop:
	docker stop $(APP_NAME) && docker rm $(APP_NAME)

docker-logs:
	docker logs -f $(APP_NAME)

# Analisa ukuran setiap layer (butuh: go install github.com/wagoodman/dive@latest)
docker-dive:
	dive $(REGISTRY)/$(APP_NAME):latest
```

### Inject Version ke Binary

```go
// main.go — variables yang di-inject oleh -ldflags saat build
package main

import "fmt"

// Diisi saat build dengan: go build -ldflags "-X main.Version=1.0.0"
var (
    Version   = "dev"
    CommitSHA = "unknown"
    BuildTime = "unknown"
)

func main() {
    fmt.Printf("Starting %s v%s (%s) built at %s\n",
        "auth-service", Version, CommitSHA, BuildTime)
    // ... rest of main
}
```

### 🏋️ Latihan 7.2

1. Buat `Dockerfile` multi-stage untuk Auth Service dari Fase 4. Target: ukuran final image **< 20MB**. Verifikasi dengan `docker images`. Gunakan `dive` untuk analisa setiap layer.
2. Implementasikan **non-root user** dengan Alpine base image: buat user `appuser` (UID 1001), set file ownership, verifikasi bahwa container tidak bisa menulis ke `/etc/` dan `/usr/bin/`.
3. Buat `Makefile` lengkap dengan target: `docker-build`, `docker-push`, `docker-run`, `docker-stop`, `docker-logs`, `docker-dive`. Target `docker-build` harus auto-inject `VERSION` dari git tags dan `COMMIT_SHA` dari git revparse.

---

## 📦 Modul 7.3 — Docker Compose

Docker Compose menjalankan **banyak container sekaligus** dengan satu file konfigurasi. Ini adalah alat utama untuk development dan testing microservices di lokal.

### docker-compose.yml Lengkap

```yaml
# docker-compose.yml
# Menjalankan seluruh E-Commerce System di lokal

version: '3.8'

# ============================================================
# SERVICES
# ============================================================
services:

  # --- Databases ---

  auth-db:
    image: postgres:16-alpine
    container_name: auth-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: authuser
      POSTGRES_PASSWORD: ${AUTH_DB_PASSWORD:-authpass}
      POSTGRES_DB: authdb
    ports:
      - "5432:5432"
    volumes:
      - auth-db-data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U authuser -d authdb"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  product-db:
    image: postgres:16-alpine
    container_name: product-db
    environment:
      POSTGRES_USER: productuser
      POSTGRES_PASSWORD: ${PRODUCT_DB_PASSWORD:-productpass}
      POSTGRES_DB: productdb
    ports:
      - "5433:5432"
    volumes:
      - product-db-data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U productuser -d productdb"]
      interval: 10s
      timeout: 5s
      retries: 5

  order-db:
    image: postgres:16-alpine
    container_name: order-db
    environment:
      POSTGRES_USER: orderuser
      POSTGRES_PASSWORD: ${ORDER_DB_PASSWORD:-orderpass}
      POSTGRES_DB: orderdb
    ports:
      - "5434:5432"
    volumes:
      - order-db-data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U orderuser -d orderdb"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    # Aktifkan AOF persistence + password
    command: >
      redis-server
      --appendonly yes
      --requirepass ${REDIS_PASSWORD:-redispass}
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD:-redispass}", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  # --- Application Services ---

  auth-service:
    build:
      context: ./services/auth-service
      dockerfile: Dockerfile
      args:
        VERSION: ${VERSION:-dev}
        COMMIT_SHA: ${COMMIT_SHA:-unknown}
    container_name: auth-service
    restart: unless-stopped
    ports:
      - "8001:8080"
      - "50051:50051"    # gRPC port
    environment:
      APP_ENV: ${APP_ENV:-development}
      LOG_LEVEL: ${LOG_LEVEL:-debug}
      # PENTING: gunakan service name sebagai hostname, BUKAN localhost!
      DATABASE_HOST: auth-db
      DATABASE_PORT: 5432
      DATABASE_NAME: authdb
      DATABASE_USER: authuser
      DATABASE_PASSWORD: ${AUTH_DB_PASSWORD:-authpass}
      REDIS_HOST: redis
      REDIS_PORT: 6379
      REDIS_PASSWORD: ${REDIS_PASSWORD:-redispass}
      JWT_SECRET: ${JWT_SECRET:-dev-secret-minimum-32-chars}
      GRPC_PORT: 50051
    depends_on:
      auth-db:
        condition: service_healthy   # tunggu DB benar-benar siap!
      redis:
        condition: service_healthy
    networks:
      - backend
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8080/health/ready"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s

  product-service:
    build:
      context: ./services/product-service
      dockerfile: Dockerfile
    container_name: product-service
    restart: unless-stopped
    ports:
      - "8002:8080"
      - "50052:50051"
    environment:
      APP_ENV: ${APP_ENV:-development}
      DATABASE_HOST: product-db
      DATABASE_PORT: 5432
      DATABASE_NAME: productdb
      DATABASE_USER: productuser
      DATABASE_PASSWORD: ${PRODUCT_DB_PASSWORD:-productpass}
      GRPC_PORT: 50051
    depends_on:
      product-db:
        condition: service_healthy
    networks:
      - backend
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8080/health/ready"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s

  order-service:
    build:
      context: ./services/order-service
      dockerfile: Dockerfile
    container_name: order-service
    restart: unless-stopped
    ports:
      - "8003:8080"
    environment:
      APP_ENV: ${APP_ENV:-development}
      DATABASE_HOST: order-db
      DATABASE_PORT: 5432
      DATABASE_NAME: orderdb
      DATABASE_USER: orderuser
      DATABASE_PASSWORD: ${ORDER_DB_PASSWORD:-orderpass}
      # Alamat service lain — gunakan service name!
      PRODUCT_SERVICE_GRPC: product-service:50051
      AUTH_SERVICE_HTTP: http://auth-service:8080
    depends_on:
      order-db:
        condition: service_healthy
      product-service:
        condition: service_healthy
      auth-service:
        condition: service_healthy
    networks:
      - backend
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8080/health/ready"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 20s

  api-gateway:
    build:
      context: ./services/api-gateway
      dockerfile: Dockerfile
    container_name: api-gateway
    restart: unless-stopped
    ports:
      - "8080:8080"    # Single entry point untuk semua client!
    environment:
      APP_ENV: ${APP_ENV:-development}
      AUTH_SERVICE_URL: http://auth-service:8080
      PRODUCT_SERVICE_URL: http://product-service:8080
      ORDER_SERVICE_URL: http://order-service:8080
      JWT_SECRET: ${JWT_SECRET:-dev-secret-minimum-32-chars}
      LOG_LEVEL: ${LOG_LEVEL:-debug}
    depends_on:
      auth-service:
        condition: service_healthy
      product-service:
        condition: service_healthy
    networks:
      - backend
      - frontend    # gateway ada di kedua network

# ============================================================
# NETWORKS
# ============================================================
networks:
  backend:
    driver: bridge
    # Semua service internal di sini
  frontend:
    driver: bridge
    # Gateway dan client-facing service di sini

# ============================================================
# VOLUMES (persistent storage)
# ============================================================
volumes:
  auth-db-data:
  product-db-data:
  order-db-data:
  redis-data:
```

### Environment Files

```bash
# .env — JANGAN commit ke git! Tambahkan ke .gitignore
VERSION=1.0.0
APP_ENV=development
LOG_LEVEL=debug

AUTH_DB_PASSWORD=dev_auth_password_123
PRODUCT_DB_PASSWORD=dev_product_password_123
ORDER_DB_PASSWORD=dev_order_password_123
REDIS_PASSWORD=dev_redis_password_123
JWT_SECRET=dev-secret-key-minimum-32-characters-long
```

```bash
# .env.example — COMMIT ini sebagai template untuk tim
VERSION=1.0.0
APP_ENV=development
LOG_LEVEL=debug

AUTH_DB_PASSWORD=CHANGE_ME
PRODUCT_DB_PASSWORD=CHANGE_ME
ORDER_DB_PASSWORD=CHANGE_ME
REDIS_PASSWORD=CHANGE_ME
JWT_SECRET=CHANGE_ME_MINIMUM_32_CHARS
```

### Perintah Docker Compose Penting

```bash
# ---- Start ----
docker compose up -d                    # jalankan semua
docker compose up -d auth-service       # jalankan service tertentu saja
docker compose up -d --build            # rebuild image lalu jalankan
docker compose up -d --scale product-service=3  # 3 instance product

# ---- Monitor ----
docker compose ps                       # status semua service
docker compose logs -f                  # logs semua (realtime)
docker compose logs -f auth-service     # logs service tertentu
docker compose logs --tail=100 order-service  # 100 baris terakhir

# ---- Management ----
docker compose stop                     # stop semua (container tetap)
docker compose down                     # stop + hapus container (volume tetap)
docker compose down -v                  # stop + hapus container + volume (DATA HILANG!)
docker compose restart product-service  # restart satu service
docker compose pull                     # pull image terbaru

# ---- Debug ----
docker compose exec auth-service sh            # masuk ke container
docker compose exec auth-db psql -U authuser -d authdb  # psql ke DB
docker compose exec redis redis-cli -a redispass         # redis CLI
docker compose top                      # proses yang berjalan di container
```

### docker-compose.override.yml untuk Development

```yaml
# docker-compose.override.yml
# Otomatis dipakai saat docker compose up (di dev environment)
# TIDAK perlu disebutkan secara eksplisit

version: '3.8'
services:
  auth-service:
    # Mount source code untuk hot reload
    volumes:
      - ./services/auth-service:/app
    # Override command pakai air untuk hot reload
    command: air -c .air.toml
    environment:
      LOG_LEVEL: debug
      ENABLE_PPROF: "true"   # aktifkan Go profiling

  product-service:
    volumes:
      - ./services/product-service:/app
    command: air -c .air.toml
    environment:
      LOG_LEVEL: debug
```

### Wait-For-It: Pastikan Dependencies Siap

```bash
#!/bin/bash
# scripts/wait-for-it.sh — tunggu TCP port siap sebelum start service

HOST="$1"
PORT="$2"
TIMEOUT="${3:-30}"

echo "Waiting for $HOST:$PORT..."
for i in $(seq 1 $TIMEOUT); do
  if nc -z "$HOST" "$PORT" 2>/dev/null; then
    echo "$HOST:$PORT is ready!"
    exit 0
  fi
  sleep 1
done

echo "Timeout waiting for $HOST:$PORT"
exit 1
```

### 🏋️ Latihan 7.3

1. Buat `docker-compose.yml` yang menjalankan Auth Service + PostgreSQL + Redis. Pastikan: (a) health checks berfungsi, (b) Auth Service hanya start setelah DB healthy, (c) data persisten meski `docker compose down && docker compose up`. Verifikasi dengan: stop semua → start ulang → data masih ada.
2. Buat `docker-compose.override.yml` dengan **hot reload** menggunakan `air`. Mount source code sebagai volume. Test bahwa perubahan kode Go langsung ter-compile ulang tanpa rebuild image.
3. Buat `Makefile` yang wrap perintah docker compose: `make up`, `make down`, `make logs`, `make shell SERVICE=auth-service`, `make psql SERVICE=auth`, `make restart SERVICE=product-service`. Tambahkan target `make status` yang menampilkan health status semua service.

---

## 📦 Modul 7.4 — Service Communication Patterns

### Sync vs Async

```
SYNCHRONOUS (Request-Response):

  Order Service ─── request ──► Product Service
  Order Service ◄── response ── Product Service

  Thread ter-BLOCK selama menunggu response.

  Gunakan saat:
    + Butuh data langsung untuk lanjutkan proses
    + Validasi yang harus selesai sebelum response ke client
    + Data harus konsisten di saat yang sama

  Hindari jika:
    - Side effects (kirim email, update analytics) — gak perlu nunggu
    - Operation lambat yang tidak critical path
    - Service downstream sering down/slow


ASYNCHRONOUS (Event-Driven):

  Order Service ─── OrderConfirmed event ──► Kafka ──► Notification Service
                                                   ──► Inventory Service
                                                   ──► Analytics Service

  Fire and forget — Order Service tidak menunggu.

  Gunakan saat:
    + Side effects (email, SMS, analytics update)
    + Eventual consistency OK
    + Satu event dikonsumsi banyak service
    + Operation yang bisa di-retry jika gagal

  Hindari jika:
    - Butuh response langsung dari downstream
    - Data harus konsisten saat ini juga
```

### gRPC Client dengan Resilience

```go
// pkg/clients/product_client.go
package clients

import (
    "context"
    "fmt"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/credentials/insecure"
    "google.golang.org/grpc/keepalive"
    "google.golang.org/grpc/status"

    productv1 "github.com/kamu/ecommerce/proto/gen/product/v1"
)

// ProductClient adalah type-safe wrapper atas generated gRPC client
type ProductClient struct {
    client  productv1.ProductServiceClient
    conn    *grpc.ClientConn
    timeout time.Duration
}

func NewProductClient(addr string) (*ProductClient, error) {
    conn, err := grpc.Dial(
        addr,
        grpc.WithTransportCredentials(insecure.NewCredentials()),
        // Keepalive: cegah koneksi idle ter-timeout oleh firewall/LB
        grpc.WithKeepaliveParams(keepalive.ClientParameters{
            Time:                10 * time.Second,
            Timeout:             3 * time.Second,
            PermitWithoutStream: true,
        }),
        // Built-in retry policy di gRPC — no need for manual retry!
        grpc.WithDefaultServiceConfig(`{
            "methodConfig": [{
                "name": [{"service": "product.v1.ProductService"}],
                "waitForReady": true,
                "timeout": "10s",
                "retryPolicy": {
                    "maxAttempts": 3,
                    "initialBackoff": "0.5s",
                    "maxBackoff": "5s",
                    "backoffMultiplier": 2.0,
                    "retryableStatusCodes": ["UNAVAILABLE", "DEADLINE_EXCEEDED"]
                }
            }]
        }`),
    )
    if err != nil {
        return nil, fmt.Errorf("dial product service [%s]: %w", addr, err)
    }

    return &ProductClient{
        client:  productv1.NewProductServiceClient(conn),
        conn:    conn,
        timeout: 10 * time.Second,
    }, nil
}

func (c *ProductClient) GetProduct(ctx context.Context, id uint64) (*productv1.Product, error) {
    // Selalu set timeout untuk setiap call!
    ctx, cancel := context.WithTimeout(ctx, c.timeout)
    defer cancel()

    resp, err := c.client.GetProduct(ctx, &productv1.GetProductRequest{Id: id})
    if err != nil {
        // Translate gRPC error ke domain error yang lebih bermakna
        st, _ := status.FromError(err)
        switch st.Code() {
        case codes.NotFound:
            return nil, fmt.Errorf("product %d not found", id)
        case codes.DeadlineExceeded, codes.Canceled:
            return nil, fmt.Errorf("product service timeout after %v", c.timeout)
        case codes.Unavailable:
            return nil, fmt.Errorf("product service is unavailable, please try again")
        default:
            return nil, fmt.Errorf("product service error: %s", st.Message())
        }
    }
    return resp.Product, nil
}

func (c *ProductClient) CheckStock(ctx context.Context, productID uint64, qty uint32) (bool, uint32, error) {
    ctx, cancel := context.WithTimeout(ctx, c.timeout)
    defer cancel()

    resp, err := c.client.CheckStock(ctx, &productv1.CheckStockRequest{
        ProductId: productID,
        Quantity:  qty,
    })
    if err != nil {
        st, _ := status.FromError(err)
        if st.Code() == codes.Unavailable {
            return false, 0, fmt.Errorf("product service unavailable")
        }
        return false, 0, err
    }
    return resp.Available, resp.CurrentStock, nil
}

// Close harus dipanggil saat aplikasi shutdown
func (c *ProductClient) Close() error {
    return c.conn.Close()
}
```

### Deadline Propagation

```go
// Pola yang BENAR: propagate deadline dari incoming request ke outgoing calls
func (h *OrderHandler) CreateOrder(c *gin.Context) {
    // Context dari HTTP request sudah punya deadline/timeout
    ctx := c.Request.Context()

    // Ini otomatis diteruskan ke gRPC call!
    product, err := h.productClient.GetProduct(ctx, productID)
    // Jika HTTP client disconnect atau timeout, gRPC call juga dibatalkan

    // JANGAN buat context baru yang menghilangkan deadline:
    // ctx := context.Background()  // SALAH! kehilangan deadline
}
```

### 🏋️ Latihan 7.4

1. Implementasikan `ProductClient` lengkap dengan method: `GetProduct`, `CheckStock`, `UpdateStock`. Buat unit test menggunakan **bufconn** (in-process gRPC server, tidak pakai network sungguhan).
2. Benchmark latency dengan 3 skenario: (a) direct function call in-process, (b) gRPC loopback `localhost:port`, (c) gRPC over Docker bridge network. Hitung overhead tiap layer.
3. Implementasikan **deadline propagation**: buat middleware di API Gateway yang set `context.WithTimeout(30s)` untuk setiap request, lalu verifikasi bahwa timeout tersebut terpropagasi ke semua downstream gRPC calls. Test: matikan Product Service, pastikan Order Service response dengan timeout (bukan hang selamanya).

---

## 📦 Modul 7.5 — API Gateway Pattern

### Apa itu API Gateway?

```
Tanpa Gateway:
  Client → Auth Service :8001
  Client → Product Service :8002
  Client → Order Service :8003
  Client harus tahu semua alamat service!
  Client harus handle auth untuk setiap service!

Dengan API Gateway:
  Client → API Gateway :8080
             │
             ├─→ Auth Service (internal)
             ├─→ Product Service (internal)
             └─→ Order Service (internal)
  
  Gateway tanggung jawab:
    - Routing ke service yang tepat
    - Validasi JWT SEKALI di gateway (tidak perlu per-service)
    - Rate limiting per IP dan per user
    - Request/response logging
    - SSL termination
    - Request aggregation
```

### Implementasi API Gateway (Gin-based)

```go
// services/api-gateway/internal/gateway/proxy.go
package gateway

import (
    "fmt"
    "net/http"
    "net/http/httputil"
    "net/url"

    "github.com/gin-gonic/gin"
    "go.uber.org/zap"
)

type Upstream struct {
    Name    string
    BaseURL string
}

// Gateway menyimpan reverse proxy per upstream service
type Gateway struct {
    proxies map[string]*httputil.ReverseProxy
    logger  *zap.Logger
}

func New(upstreams []Upstream, logger *zap.Logger) *Gateway {
    g := &Gateway{proxies: make(map[string]*httputil.ReverseProxy), logger: logger}

    for _, up := range upstreams {
        target, _ := url.Parse(up.BaseURL)
        proxy := httputil.NewSingleHostReverseProxy(target)

        // Custom error handler untuk response yang informatif
        upName := up.Name // capture untuk closure
        proxy.ErrorHandler = func(w http.ResponseWriter, r *http.Request, err error) {
            logger.Error("upstream error", zap.String("service", upName), zap.Error(err))
            w.Header().Set("Content-Type", "application/json")
            w.WriteHeader(http.StatusBadGateway)
            fmt.Fprintf(w, `{"success":false,"error":{"code":"SERVICE_UNAVAILABLE","message":"upstream service %s is unavailable"}}`, upName)
        }

        // Modifikasi request sebelum diteruskan ke upstream
        originalDirector := proxy.Director
        proxy.Director = func(req *http.Request) {
            originalDirector(req)
            req.Header.Set("X-Forwarded-By", "api-gateway")
        }

        g.proxies[up.Name] = proxy
    }
    return g
}

// ProxyTo membuat Gin handler yang meneruskan request ke upstream
func (g *Gateway) ProxyTo(upstreamName string) gin.HandlerFunc {
    return func(c *gin.Context) {
        proxy, ok := g.proxies[upstreamName]
        if !ok {
            c.JSON(http.StatusInternalServerError, gin.H{"error": "unknown upstream: " + upstreamName})
            return
        }

        // Teruskan user info (dari JWT yang sudah divalidasi di gateway)
        // ke upstream service via header internal
        if userID, exists := c.Get("user_id"); exists {
            c.Request.Header.Set("X-User-ID", fmt.Sprintf("%v", userID))
            c.Request.Header.Set("X-User-Email", c.GetString("user_email"))
            c.Request.Header.Set("X-User-Role", c.GetString("user_role"))
        }

        // Teruskan request ID untuk distributed tracing
        if reqID := c.GetString("request_id"); reqID != "" {
            c.Request.Header.Set("X-Request-ID", reqID)
        }

        proxy.ServeHTTP(c.Writer, c.Request)
    }
}
```

```go
// services/api-gateway/internal/gateway/router.go
package gateway

import (
    "net/http"
    "time"

    "github.com/gin-gonic/gin"
    "github.com/kamu/api-gateway/internal/middleware"
)

func SetupRouter(gw *Gateway, authMW *middleware.AuthMiddleware, jwtSecret string) *gin.Engine {
    r := gin.New()
    r.Use(
        gin.Recovery(),
        middleware.RequestID(),      // inject X-Request-ID ke setiap request
        middleware.Logger(),         // log setiap request
        middleware.CORS(),           // CORS headers
        middleware.RateLimit(100, time.Minute),  // 100 req/menit per IP
    )

    // Health check untuk gateway itu sendiri
    r.GET("/health", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{"status": "ok", "service": "api-gateway"})
    })

    api := r.Group("/api/v1")

    // === PUBLIC ROUTES (tanpa auth) ===
    auth := api.Group("/auth")
    auth.POST("/register", gw.ProxyTo("auth"))
    auth.POST("/login", gw.ProxyTo("auth"))
    auth.POST("/refresh", gw.ProxyTo("auth"))

    api.GET("/products", gw.ProxyTo("product"))
    api.GET("/products/:id", gw.ProxyTo("product"))
    api.GET("/products/:id/reviews", gw.ProxyTo("product"))

    // === PROTECTED ROUTES (butuh JWT) ===
    protected := api.Group("")
    protected.Use(authMW.Validate())  // validasi JWT SEKALI di gateway
    {
        protected.GET("/auth/me", gw.ProxyTo("auth"))
        protected.POST("/auth/logout", gw.ProxyTo("auth"))
        protected.PUT("/auth/change-password", gw.ProxyTo("auth"))

        protected.POST("/orders", gw.ProxyTo("order"))
        protected.GET("/orders", gw.ProxyTo("order"))
        protected.GET("/orders/:id", gw.ProxyTo("order"))
        protected.POST("/orders/:id/cancel", gw.ProxyTo("order"))
    }

    // === ADMIN ROUTES ===
    admin := api.Group("/admin")
    admin.Use(authMW.Validate(), authMW.RequireRole("admin"))
    {
        admin.GET("/users", gw.ProxyTo("auth"))
        admin.POST("/products", gw.ProxyTo("product"))
        admin.PUT("/products/:id", gw.ProxyTo("product"))
        admin.GET("/orders", gw.ProxyTo("order"))
    }

    return r
}
```

```go
// services/api-gateway/internal/middleware/auth.go
package middleware

import (
    "fmt"
    "net/http"
    "strings"

    "github.com/gin-gonic/gin"
    "github.com/golang-jwt/jwt/v5"
)

type AuthMiddleware struct {
    jwtSecret []byte
}

func NewAuthMiddleware(secret string) *AuthMiddleware {
    return &AuthMiddleware{jwtSecret: []byte(secret)}
}

// Validate memvalidasi JWT — TIDAK call ke Auth Service!
// Token bisa divalidasi secara lokal karena kita tahu secret-nya
func (m *AuthMiddleware) Validate() gin.HandlerFunc {
    return func(c *gin.Context) {
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "success": false,
                "error":   gin.H{"code": "UNAUTHORIZED", "message": "authorization header required"},
            })
            return
        }

        if !strings.HasPrefix(authHeader, "Bearer ") {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "success": false,
                "error":   gin.H{"code": "UNAUTHORIZED", "message": "format: Bearer {token}"},
            })
            return
        }

        tokenStr := strings.TrimPrefix(authHeader, "Bearer ")
        token, err := jwt.Parse(tokenStr, func(t *jwt.Token) (interface{}, error) {
            if _, ok := t.Method.(*jwt.SigningMethodHMAC); !ok {
                return nil, fmt.Errorf("unexpected signing method: %v", t.Header["alg"])
            }
            return m.jwtSecret, nil
        })

        if err != nil || !token.Valid {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "success": false,
                "error":   gin.H{"code": "TOKEN_INVALID", "message": "invalid or expired token"},
            })
            return
        }

        // Simpan user info ke context — bisa dipakai oleh ProxyTo untuk add headers
        claims := token.Claims.(jwt.MapClaims)
        c.Set("user_id", uint(claims["user_id"].(float64)))
        c.Set("user_email", claims["email"].(string))
        c.Set("user_role", claims["role"].(string))

        c.Next()
    }
}

func (m *AuthMiddleware) RequireRole(role string) gin.HandlerFunc {
    return func(c *gin.Context) {
        userRole := c.GetString("user_role")
        if userRole != role {
            c.AbortWithStatusJSON(http.StatusForbidden, gin.H{
                "success": false,
                "error":   gin.H{"code": "FORBIDDEN", "message": "insufficient permissions"},
            })
            return
        }
        c.Next()
    }
}
```

### Request Aggregation

```go
// Endpoint yang menggabungkan data dari beberapa service sekaligus
// GET /api/v1/dashboard — menampilkan profile + recent orders + featured products

func (gw *Gateway) Dashboard(c *gin.Context) {
    ctx := c.Request.Context()
    userID := c.GetString("user_id")
    token := c.GetHeader("Authorization")

    type result struct {
        data interface{}
        err  error
        key  string
    }

    results := make(chan result, 3)

    // Panggil 3 service secara PARALEL menggunakan goroutines
    go func() {
        data, err := gw.fetchUserProfile(ctx, token)
        results <- result{key: "profile", data: data, err: err}
    }()

    go func() {
        data, err := gw.fetchRecentOrders(ctx, token, userID)
        results <- result{key: "recent_orders", data: data, err: err}
    }()

    go func() {
        data, err := gw.fetchFeaturedProducts(ctx)
        results <- result{key: "featured_products", data: data, err: err}
    }()

    // Kumpulkan semua hasil
    response := make(map[string]interface{})
    for i := 0; i < 3; i++ {
        r := <-results
        if r.err != nil {
            // Log error tapi tetap return partial data
            gw.logger.Warn("partial fetch error", zap.String("key", r.key), zap.Error(r.err))
            response[r.key] = nil
        } else {
            response[r.key] = r.data
        }
    }

    c.JSON(http.StatusOK, gin.H{"success": true, "data": response})
}
```

### Rate Limiter dengan Redis (Per User & Per IP)

```go
// pkg/ratelimit/limiter.go
package ratelimit

import (
    "context"
    "fmt"
    "time"

    "github.com/gin-gonic/gin"
    "github.com/redis/go-redis/v9"
)

type RateLimiter struct {
    rdb     *redis.Client
    limit   int           // max requests
    window  time.Duration // time window
}

func NewRateLimiter(rdb *redis.Client, limit int, window time.Duration) *RateLimiter {
    return &RateLimiter{rdb: rdb, limit: limit, window: window}
}

// Middleware menggunakan Sliding Window algorithm di Redis
func (rl *RateLimiter) Middleware(keyFn func(*gin.Context) string) gin.HandlerFunc {
    return func(c *gin.Context) {
        ctx := c.Request.Context()
        key := "ratelimit:" + keyFn(c)
        now := time.Now().UnixMilli()
        windowStart := now - rl.window.Milliseconds()

        // Sliding window dengan Redis sorted set
        pipe := rl.rdb.TxPipeline()
        // Hapus entries di luar window
        pipe.ZRemRangeByScore(ctx, key, "0", fmt.Sprintf("%d", windowStart))
        // Hitung entries dalam window
        countCmd := pipe.ZCard(ctx, key)
        // Tambahkan request baru
        pipe.ZAdd(ctx, key, redis.Z{Score: float64(now), Member: now})
        // Set TTL
        pipe.Expire(ctx, key, rl.window)
        pipe.Exec(ctx)

        count := countCmd.Val()
        remaining := int64(rl.limit) - count

        c.Header("X-RateLimit-Limit", fmt.Sprintf("%d", rl.limit))
        c.Header("X-RateLimit-Remaining", fmt.Sprintf("%d", max(0, remaining)))
        c.Header("X-RateLimit-Reset", fmt.Sprintf("%d", time.Now().Add(rl.window).Unix()))

        if count >= int64(rl.limit) {
            c.AbortWithStatusJSON(429, gin.H{
                "success": false,
                "error": gin.H{
                    "code":    "RATE_LIMIT_EXCEEDED",
                    "message": fmt.Sprintf("Rate limit exceeded. Try again in %v", rl.window),
                },
            })
            return
        }
        c.Next()
    }
}

func max(a, b int64) int64 {
    if a > b { return a }
    return b
}

// Penggunaan di Gateway:
// r.Use(rateLimiter.Middleware(func(c *gin.Context) string {
//     // Rate limit per user jika authenticated, per IP jika tidak
//     if userID := c.GetString("user_id"); userID != "" {
//         return "user:" + userID
//     }
//     return "ip:" + c.ClientIP()
// }))
```


### 🏋️ Latihan 7.5

1. Implementasikan `API Gateway` lengkap dengan reverse proxy ke Auth, Product, dan Order Service. Test setiap route dengan curl atau Postman.
2. Buat endpoint `GET /api/v1/dashboard` yang mengumpulkan data dari 3 service secara **paralel** (profile dari Auth, recent orders dari Order, featured products dari Product). Ukur perbedaan latency vs sequential calls.
3. Implementasikan **rate limiting per user** menggunakan Redis: user terautentikasi mendapat 1000 req/menit, anonymous 60 req/menit. Gunakan sliding window algorithm.

---

## 📦 Modul 7.6 — Service Discovery

### Tiga Pendekatan Discovery

```
1. ENV-BASED (Paling Sederhana):
   Alamat service sudah diketahui saat deploy → simpan di env vars
   
   PRODUCT_SERVICE_GRPC=product-service:50051
   AUTH_SERVICE_HTTP=http://auth-service:8080
   
   Cocok untuk: Docker Compose, simple K8s setup
   Tidak cocok: dynamic scaling, failover otomatis

2. DNS-BASED (Docker Compose & Kubernetes):
   Docker Compose: setiap service punya DNS entry = service name
     product-service:8080 → DNS resolve otomatis
   
   Kubernetes: <service-name>.<namespace>.svc.cluster.local
     auth-service.production.svc.cluster.local:8080
   
   Cocok untuk: hampir semua skenario produksi
   Tidak cocok: multi-cluster, complex routing

3. SERVICE REGISTRY (Consul, etcd, Eureka):
   Service register dirinya saat start, deregister saat stop
   Client query registry untuk dapatkan alamat aktif
   
   Cocok untuk: dynamic scaling, multi-DC, complex scenarios
   Tidak cocok: small teams, simple setups (over-engineering)
```

### Config-Driven Service Locator

```go
// config/services_config.go — pattern yang dipakai di industri
package config

type ServicesConfig struct {
    // Diisi via env vars sesuai environment:
    // Development: localhost:port
    // Docker Compose: service-name:port
    // Kubernetes: service-name.namespace.svc.cluster.local:port
    AuthServiceHTTP    string        `mapstructure:"auth_service_http"    envconfig:"AUTH_SERVICE_HTTP"`
    ProductServiceGRPC string        `mapstructure:"product_service_grpc" envconfig:"PRODUCT_SERVICE_GRPC"`
    OrderServiceHTTP   string        `mapstructure:"order_service_http"   envconfig:"ORDER_SERVICE_HTTP"`
    CallTimeout        time.Duration `mapstructure:"call_timeout"`
}

// Contoh values per environment:

// config/config.development.yaml:
// services:
//   auth_service_http: http://localhost:8001
//   product_service_grpc: localhost:50052
//   order_service_http: http://localhost:8003
//   call_timeout: 10s

// docker-compose.yml environment:
//   AUTH_SERVICE_HTTP=http://auth-service:8080
//   PRODUCT_SERVICE_GRPC=product-service:50051

// kubernetes ConfigMap:
//   AUTH_SERVICE_HTTP=http://auth-service.production.svc.cluster.local:8080
//   PRODUCT_SERVICE_GRPC=product-service.production.svc.cluster.local:50051
```

### Health-Aware Load Balancer

```go
// pkg/discovery/locator.go
// Untuk kasus di mana satu service punya multiple instances
package discovery

import (
    "context"
    "fmt"
    "net/http"
    "sync"
    "sync/atomic"
    "time"
)

// Endpoint merepresentasikan satu instance service
type Endpoint struct {
    Address   string
    healthy   atomic.Bool  // thread-safe tanpa mutex
}

func newEndpoint(address string) *Endpoint {
    e := &Endpoint{Address: address}
    e.healthy.Store(true)
    return e
}

func (e *Endpoint) IsHealthy() bool       { return e.healthy.Load() }
func (e *Endpoint) SetHealthy(h bool)     { e.healthy.Store(h) }

// ServiceLocator dengan round-robin load balancing
type ServiceLocator struct {
    mu        sync.RWMutex
    endpoints map[string][]*Endpoint
    counters  sync.Map  // map[string]*atomic.Uint64 untuk counter per service
}

func NewServiceLocator() *ServiceLocator {
    return &ServiceLocator{
        endpoints: make(map[string][]*Endpoint),
    }
}

// Register menambah endpoint untuk service
func (l *ServiceLocator) Register(serviceName, address string) {
    l.mu.Lock()
    defer l.mu.Unlock()
    l.endpoints[serviceName] = append(l.endpoints[serviceName], newEndpoint(address))
}

// Get mendapatkan satu healthy endpoint dengan round-robin
func (l *ServiceLocator) Get(serviceName string) (string, error) {
    l.mu.RLock()
    eps := l.endpoints[serviceName]
    l.mu.RUnlock()

    var healthy []*Endpoint
    for _, ep := range eps {
        if ep.IsHealthy() {
            healthy = append(healthy, ep)
        }
    }

    if len(healthy) == 0 {
        return "", fmt.Errorf("no healthy endpoints for service: %s", serviceName)
    }

    // Round-robin counter per service
    counter, _ := l.counters.LoadOrStore(serviceName, new(atomic.Uint64))
    idx := counter.(*atomic.Uint64).Add(1) % uint64(len(healthy))
    return healthy[idx].Address, nil
}

// StartHealthMonitor menjalankan periodic health check di background
func (l *ServiceLocator) StartHealthMonitor(ctx context.Context, interval time.Duration) {
    go func() {
        ticker := time.NewTicker(interval)
        defer ticker.Stop()

        for {
            select {
            case <-ctx.Done():
                return
            case <-ticker.C:
                l.mu.RLock()
                for name, eps := range l.endpoints {
                    for _, ep := range eps {
                        wasHealthy := ep.IsHealthy()
                        isHealthy := l.ping(ep.Address)

                        if wasHealthy != isHealthy {
                            ep.SetHealthy(isHealthy)
                            status := "UNHEALTHY"
                            if isHealthy {
                                status = "HEALTHY"
                            }
                            fmt.Printf("[discovery] %s/%s is now %s\n", name, ep.Address, status)
                        }
                    }
                }
                l.mu.RUnlock()
            }
        }
    }()
}

func (l *ServiceLocator) ping(address string) bool {
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()

    req, err := http.NewRequestWithContext(ctx, "GET", "http://"+address+"/health/live", nil)
    if err != nil {
        return false
    }
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return false
    }
    defer resp.Body.Close()
    return resp.StatusCode == http.StatusOK
}
```

### 🏋️ Latihan 7.6

1. Buat `ServiceLocator` dan daftarkan 3 endpoint untuk Product Service. Test bahwa round-robin mendistribusikan request secara merata. Simulasikan satu endpoint mati dan verifikasi bahwa request hanya dikirim ke yang sehat.
2. Buat fungsi `NewClientFromEnv(envKey, defaultAddr string) string` yang membaca alamat service dari env var, dengan fallback ke default untuk development lokal. Gunakan ini di semua service client.
3. Integrasikan `HealthMonitor` ke startup: jalankan health monitor di goroutine, dan buat endpoint `GET /api/v1/admin/discovery` yang menampilkan status semua registered endpoints.

---

## 📦 Modul 7.7 — Inter-Service gRPC Communication

### Connection Pool — Wajib untuk Production

```go
// pkg/grpcpool/pool.go
// Membuat koneksi baru setiap request = overhead BESAR
// Pool reuse koneksi yang sudah ada
package grpcpool

import (
    "fmt"
    "sync"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/connectivity"
    "google.golang.org/grpc/credentials/insecure"
    "google.golang.org/grpc/keepalive"
)

// Pool adalah thread-safe gRPC connection pool
type Pool struct {
    mu    sync.RWMutex
    conns map[string]*grpc.ClientConn
    opts  []grpc.DialOption
}

func NewPool(opts ...grpc.DialOption) *Pool {
    defaultOpts := []grpc.DialOption{
        grpc.WithTransportCredentials(insecure.NewCredentials()),
        grpc.WithKeepaliveParams(keepalive.ClientParameters{
            Time:                30 * time.Second,
            Timeout:             5 * time.Second,
            PermitWithoutStream: true,
        }),
    }
    return &Pool{
        conns: make(map[string]*grpc.ClientConn),
        opts:  append(defaultOpts, opts...),
    }
}

// Get mendapatkan atau membuat koneksi ke addr
// Thread-safe dengan double-check locking pattern
func (p *Pool) Get(addr string) (*grpc.ClientConn, error) {
    // Fast path: check dengan read lock
    p.mu.RLock()
    if conn, ok := p.conns[addr]; ok {
        // Pastikan koneksi masih valid
        state := conn.GetState()
        if state != connectivity.Shutdown && state != connectivity.TransientFailure {
            p.mu.RUnlock()
            return conn, nil
        }
    }
    p.mu.RUnlock()

    // Slow path: buat koneksi baru
    p.mu.Lock()
    defer p.mu.Unlock()

    // Double-check setelah acquire write lock (another goroutine mungkin sudah buat)
    if conn, ok := p.conns[addr]; ok {
        state := conn.GetState()
        if state != connectivity.Shutdown && state != connectivity.TransientFailure {
            return conn, nil
        }
        // Koneksi rusak — tutup dan buat baru
        conn.Close()
    }

    conn, err := grpc.Dial(addr, p.opts...)
    if err != nil {
        return nil, fmt.Errorf("grpc dial [%s]: %w", addr, err)
    }

    p.conns[addr] = conn
    return conn, nil
}

// CloseAll menutup semua koneksi — panggil saat shutdown
func (p *Pool) CloseAll() {
    p.mu.Lock()
    defer p.mu.Unlock()
    for addr, conn := range p.conns {
        conn.Close()
        delete(p.conns, addr)
    }
}
```

### Metadata Propagation

```go
// pkg/grpcmeta/propagate.go
// Propagasi metadata penting dari incoming ke outgoing gRPC calls
// Ini penting untuk distributed tracing dan logging
package grpcmeta

import (
    "context"
    "google.golang.org/grpc/metadata"
)

// Keys yang harus selalu diteruskan antar service
var propagationKeys = []string{
    "x-request-id",   // untuk correlate logs antar service
    "x-trace-id",     // untuk distributed tracing (Fase 9)
    "x-user-id",      // user yang melakukan request
    "x-user-role",    // role untuk auth di downstream service
}

// Forward meneruskan metadata dari incoming context ke outgoing context
// Panggil ini sebelum membuat gRPC call ke service lain
func Forward(ctx context.Context) context.Context {
    inMD, ok := metadata.FromIncomingContext(ctx)
    if !ok {
        return ctx  // tidak ada metadata masuk, skip
    }

    outMD := metadata.New(nil)
    for _, key := range propagationKeys {
        if vals := inMD.Get(key); len(vals) > 0 {
            outMD.Set(key, vals...)
        }
    }

    // Hanya update outgoing context jika ada yang perlu diteruskan
    if len(outMD) > 0 {
        return metadata.NewOutgoingContext(ctx, outMD)
    }
    return ctx
}

// Inject menambahkan nilai ke outgoing metadata
func Inject(ctx context.Context, key, value string) context.Context {
    outMD, ok := metadata.FromOutgoingContext(ctx)
    if !ok {
        outMD = metadata.New(nil)
    }
    outMD.Set(key, value)
    return metadata.NewOutgoingContext(ctx, outMD)
}

// Extract mengambil nilai dari incoming metadata
func Extract(ctx context.Context, key string) string {
    md, ok := metadata.FromIncomingContext(ctx)
    if !ok {
        return ""
    }
    vals := md.Get(key)
    if len(vals) == 0 {
        return ""
    }
    return vals[0]
}
```

```go
// Penggunaan di handler — ini yang membuat distributed tracing bekerja
func (s *OrderServer) CreateOrder(ctx context.Context, req *orderv1.CreateOrderRequest) (*orderv1.CreateOrderResponse, error) {
    // Teruskan metadata (request ID, trace ID, dll) ke semua downstream calls
    ctx = grpcmeta.Forward(ctx)

    // Sekarang semua calls ke service lain membawa metadata yang sama
    product, err := s.productClient.GetProduct(ctx, req.ProductId)
    if err != nil {
        return nil, err
    }

    // Request ID yang sama akan muncul di logs semua service!
    // Request ID dari API Gateway → Order Service → Product Service → logs
    return &orderv1.CreateOrderResponse{/* ... */}, nil
}
```

### Load Balancing di gRPC

```go
// Untuk multiple server addresses dari config
func NewProductClientWithLB(addrs []string) (*ProductClient, error) {
    // Format: "dns:///product-service:50051" untuk DNS-based LB
    // Format untuk multiple addresses:
    target := fmt.Sprintf("ipv4:///%s", strings.Join(addrs, ","))

    conn, err := grpc.Dial(
        target,
        grpc.WithTransportCredentials(insecure.NewCredentials()),
        // Round-robin load balancing built-in
        grpc.WithDefaultServiceConfig(`{
            "loadBalancingConfig": [{"round_robin": {}}]
        }`),
    )
    if err != nil {
        return nil, fmt.Errorf("dial product service: %w", err)
    }

    return &ProductClient{conn: conn, client: productv1.NewProductServiceClient(conn)}, nil
}

// Baca dari env: PRODUCT_SERVICE_GRPC_ADDRS=host1:50051,host2:50051
func newFromEnv() (*ProductClient, error) {
    addrsStr := os.Getenv("PRODUCT_SERVICE_GRPC_ADDRS")
    if addrsStr == "" {
        addrsStr = os.Getenv("PRODUCT_SERVICE_GRPC") // fallback ke single
    }
    if addrsStr == "" {
        addrsStr = "localhost:50052" // dev default
    }
    addrs := strings.Split(addrsStr, ",")
    return NewProductClientWithLB(addrs)
}
```

### 🏋️ Latihan 7.7

1. Implementasikan `ConnectionPool` dan buat benchmark: bandingkan latency P50/P95/P99 dari 1000 request dengan (a) reuse connection dari pool vs (b) buat koneksi baru setiap request.
2. Buat `MetadataPropagatorInterceptor` — server interceptor yang menyimpan metadata ke context, dan client interceptor yang meneruskannya ke outgoing calls. Verifikasi bahwa request ID dari API Gateway muncul di logs Order Service DAN Product Service.
3. Implementasikan load balancing dengan 3 instance Product Service. Gunakan env var `PRODUCT_SERVICE_GRPC_ADDRS` untuk konfigurasi. Verifikasi distribusi request dengan `docker compose up --scale product-service=3`.

---

## 📦 Modul 7.8 — Saga Pattern

### Mengapa Saga Diperlukan?

```
Checkout flow melibatkan 3 database berbeda:

  Order DB: buat order
  Product DB: kurangi stok
  Payment: charge kartu

Tanpa Saga — masalah:
  Bagaimana jika step 2 sukses tapi step 3 gagal?
  Stok sudah berkurang, tapi order tidak terbayar!
  → Data INCONSISTENT

Dengan database tunggal (monolith): pakai SQL transaction, rollback otomatis.
Dengan microservices: tidak bisa! Setiap service punya DB sendiri.
SOLUSI: Saga Pattern
```

### Choreography Saga (Event-Driven)

```
Setiap service mendengarkan events dan bereaksi sendiri.
Tidak ada koordinator terpusat.

Happy path:
  Order Service   → publish: OrderCreated
  Product Service → listen OrderCreated → reserve stock → publish: StockReserved
  Payment Service → listen StockReserved → charge card → publish: PaymentSucceeded
  Order Service   → listen PaymentSucceeded → confirm order → publish: OrderConfirmed

Failure compensation:
  Payment Service → charge gagal → publish: PaymentFailed
  Product Service → listen PaymentFailed → release stock
  Order Service   → listen PaymentFailed → cancel order

Keuntungan: Loose coupling, mudah scale, setiap service independent
Kekurangan: Alur sulit di-trace, logic tersebar di banyak service
```

```go
// services/order-service/internal/saga/checkout_handlers.go
// Handlers untuk choreography saga di Order Service
package saga

import (
    "context"
    "encoding/json"
    "fmt"

    "github.com/kamu/order-service/internal/domain/order/aggregate"
    "github.com/kamu/order-service/internal/domain/order/entity"
    "github.com/kamu/order-service/internal/domain/order/repository"
    "github.com/kamu/order-service/internal/event"
)

type CheckoutSagaHandler struct {
    orderRepo  repository.OrderCommandRepository
    eventPub   event.Publisher
}

func NewCheckoutSagaHandler(repo repository.OrderCommandRepository, pub event.Publisher) *CheckoutSagaHandler {
    return &CheckoutSagaHandler{orderRepo: repo, eventPub: pub}
}

// HandleStockReservationFailed — compensating transaction saat stok tidak tersedia
// Dipanggil saat menerima event "stock.reservation.failed" dari Kafka
func (h *CheckoutSagaHandler) HandleStockReservationFailed(ctx context.Context, msg event.Message) error {
    var payload struct {
        OrderID uint64 `json:"order_id"`
        Reason  string `json:"reason"`
    }
    if err := json.Unmarshal(msg.Body, &payload); err != nil {
        return fmt.Errorf("unmarshal StockReservationFailed: %w", err)
    }

    // Idempotency: cek apakah event sudah diproses
    if msg.IsAlreadyProcessed() {
        return nil
    }

    order, err := h.orderRepo.FindByID(ctx, entity.OrderID(payload.OrderID))
    if err != nil {
        return fmt.Errorf("find order %d: %w", payload.OrderID, err)
    }
    if order == nil {
        return nil // order tidak ada, mungkin sudah dihapus
    }

    // Compensating transaction: cancel order
    if err := order.Cancel("Stock reservation failed: " + payload.Reason); err != nil {
        return fmt.Errorf("cancel order: %w", err)
    }

    return h.orderRepo.Save(ctx, order)
}

// HandlePaymentSucceeded — konfirmasi order setelah pembayaran berhasil
func (h *CheckoutSagaHandler) HandlePaymentSucceeded(ctx context.Context, msg event.Message) error {
    var payload struct {
        OrderID   uint64 `json:"order_id"`
        PaymentID string `json:"payment_id"`
    }
    if err := json.Unmarshal(msg.Body, &payload); err != nil {
        return fmt.Errorf("unmarshal PaymentSucceeded: %w", err)
    }

    if msg.IsAlreadyProcessed() {
        return nil
    }

    order, err := h.orderRepo.FindByID(ctx, entity.OrderID(payload.OrderID))
    if err != nil || order == nil {
        return fmt.Errorf("find order %d: %w", payload.OrderID, err)
    }

    if err := order.Confirm(payload.PaymentID); err != nil {
        return fmt.Errorf("confirm order: %w", err)
    }

    if err := h.orderRepo.Save(ctx, order); err != nil {
        return err
    }

    // Publish OrderConfirmed untuk consumers lain:
    // - Notification Service → kirim email
    // - Inventory Service → update analytics
    // - Warehouse Service → siapkan pengiriman
    return h.eventPub.Publish(ctx, order.DomainEvents()...)
}

// HandlePaymentFailed — compensating transaction setelah payment gagal
func (h *CheckoutSagaHandler) HandlePaymentFailed(ctx context.Context, msg event.Message) error {
    var payload struct {
        OrderID uint64 `json:"order_id"`
        Reason  string `json:"reason"`
    }
    if err := json.Unmarshal(msg.Body, &payload); err != nil {
        return err
    }

    if msg.IsAlreadyProcessed() {
        return nil
    }

    order, err := h.orderRepo.FindByID(ctx, entity.OrderID(payload.OrderID))
    if err != nil || order == nil {
        return fmt.Errorf("find order %d: %w", payload.OrderID, err)
    }

    // Apakah stok sudah dikurangi? Jika ya, perlu di-release
    stockWasReserved := order.Status() == aggregate.StatusConfirmed

    if err := order.Cancel("Payment failed: " + payload.Reason); err != nil {
        return fmt.Errorf("cancel order: %w", err)
    }

    if err := h.orderRepo.Save(ctx, order); err != nil {
        return err
    }

    // Jika stok sudah dikurangi, publish event agar Product Service release
    // Product Service listen "order.cancelled" dan restore stok
    if stockWasReserved {
        return h.eventPub.Publish(ctx, order.DomainEvents()...)
    }
    return nil
}
```

### Orchestration Saga (Centralized Coordinator)

```go
// services/order-service/internal/saga/checkout_orchestrator.go
// Satu coordinator yang mengatur semua langkah
package saga

type CheckoutOrchestrator struct {
    orderRepo     repository.OrderCommandRepository
    productClient *clients.ProductClient
    paymentClient *clients.PaymentClient
    eventPub      event.Publisher
}

// ExecuteCheckout menjalankan saga secara berurutan dengan kompensasi otomatis
func (o *CheckoutOrchestrator) ExecuteCheckout(ctx context.Context, orderID uint64) error {
    order, err := o.orderRepo.FindByIDForUpdate(ctx, entity.OrderID(orderID)) // lock baris
    if err != nil || order == nil {
        return fmt.Errorf("find order: %w", err)
    }

    // Step 1: Reserve stock
    if err := o.reserveStock(ctx, order); err != nil {
        // Compensate: cancel order
        order.Cancel("Failed to reserve stock: " + err.Error())
        o.orderRepo.Save(ctx, order)
        return fmt.Errorf("reserve stock: %w", err)
    }

    // Step 2: Process payment
    paymentID, err := o.processPayment(ctx, order)
    if err != nil {
        // Compensate: release stock DULU, baru cancel order
        o.releaseStock(ctx, order)
        order.Cancel("Payment failed: " + err.Error())
        o.orderRepo.Save(ctx, order)
        return fmt.Errorf("process payment: %w", err)
    }

    // Step 3: Confirm order
    order.Confirm(paymentID)
    if err := o.orderRepo.Save(ctx, order); err != nil {
        return err
    }

    return o.eventPub.Publish(ctx, order.DomainEvents()...)
}

func (o *CheckoutOrchestrator) reserveStock(ctx context.Context, order *aggregate.Order) error {
    for _, item := range order.Items() {
        available, currentStock, err := o.productClient.CheckStock(ctx, item.ProductID(), uint32(item.Quantity()))
        if err != nil {
            return fmt.Errorf("check stock product %d: %w", item.ProductID(), err)
        }
        if !available {
            return fmt.Errorf("insufficient stock for product %d: have %d, need %d",
                item.ProductID(), currentStock, item.Quantity())
        }
    }

    // Kurangi stok semua items
    for _, item := range order.Items() {
        if _, err := o.productClient.UpdateStock(ctx, item.ProductID(), -int32(item.Quantity()), "sale"); err != nil {
            // TODO: rollback yang sudah di-update (idealnya pakai transaction saga state)
            return fmt.Errorf("update stock product %d: %w", item.ProductID(), err)
        }
    }
    return nil
}

func (o *CheckoutOrchestrator) releaseStock(ctx context.Context, order *aggregate.Order) {
    for _, item := range order.Items() {
        // Best effort — log error tapi jangan fail
        if _, err := o.productClient.UpdateStock(ctx, item.ProductID(), int32(item.Quantity()), "return"); err != nil {
            // Log error untuk manual investigation
            fmt.Printf("ERROR: failed to release stock for product %d: %v\n", item.ProductID(), err)
        }
    }
}
```

### Saga State Machine

```go
// Simpan state saga di DB untuk idempotency dan recovery
type CheckoutSagaState struct {
    ID               uint64     `gorm:"primarykey"`
    OrderID          uint64     `gorm:"uniqueIndex;not null"`
    Status           string     `gorm:"not null;default:'RUNNING'"`
    // RUNNING, STOCK_RESERVED, PAYMENT_PROCESSING, COMPLETED, COMPENSATING, FAILED
    StockReserved    bool
    PaymentAttempted bool
    PaymentID        string
    FailureReason    string
    CreatedAt        time.Time
    UpdatedAt        time.Time
}
```

### 🏋️ Latihan 7.8

1. Implementasikan **choreography saga** untuk checkout menggunakan in-memory event bus. Test 3 skenario: (a) happy path → order CONFIRMED, (b) stok tidak cukup → order CANCELLED, (c) payment gagal → stok di-release + order CANCELLED. Verifikasi state di setiap step.
2. Buat `CheckoutSagaStateRepository` dan implementasikan **idempotency**: setiap event handler harus cek `processed_events` table sebelum proses. Jika event sudah diproses, skip tanpa error.
3. Implementasikan **saga timeout**: background goroutine yang setiap menit cek saga yang RUNNING > 10 menit, otomatis cancel dan trigger compensation. Pastikan timeout ini juga idempotent.

---

## 📦 Modul 7.9 — Circuit Breaker & Resilience

### Pola Resiliensi yang Wajib Dipahami

```
1. TIMEOUT
   Setiap external call HARUS punya timeout.
   Tanpa timeout, satu service lambat bisa block semua goroutine.

2. RETRY dengan EXPONENTIAL BACKOFF
   Coba ulang saat gagal sementara (network glitch, brief downtime).
   Jangan retry langsung — tunggu dengan delay yang makin lama.
   Tambahkan jitter untuk hindari thundering herd.

3. CIRCUIT BREAKER
   Jika service sering gagal, stop call sementara (fast fail).
   Beri service waktu untuk recover sebelum coba lagi.

4. BULKHEAD
   Batasi concurrent calls ke satu service.
   Mencegah satu service lambat exhaust semua goroutine.

5. FALLBACK
   Jika service tidak tersedia, return cached/default value.
   Degraded functionality lebih baik dari error total.
```

### Implementasi Circuit Breaker

```bash
go get github.com/sony/gobreaker
```

```go
// pkg/resilience/circuit_breaker.go
package resilience

import (
    "fmt"
    "time"

    "github.com/sony/gobreaker"
)

// CircuitBreaker membungkus sony/gobreaker dengan API yang lebih ergonomis
type CircuitBreaker struct {
    cb   *gobreaker.CircuitBreaker
    name string
}

func NewCircuitBreaker(serviceName string) *CircuitBreaker {
    settings := gobreaker.Settings{
        Name: serviceName,

        // Half-open: izinkan 3 request setelah timeout untuk test recovery
        MaxRequests: 3,

        // Window untuk menghitung failure rate
        Interval: 60 * time.Second,

        // Berapa lama circuit tetap OPEN sebelum coba half-open
        Timeout: 30 * time.Second,

        // Kondisi kapan circuit harus OPEN
        ReadyToTrip: func(counts gobreaker.Counts) bool {
            // Minimal 5 request dulu, lalu open jika failure rate > 60%
            if counts.Requests < 5 {
                return false
            }
            failureRate := float64(counts.TotalFailures) / float64(counts.Requests)
            return failureRate >= 0.6
        },

        // Callback untuk observability — bisa dikirim ke metrics
        OnStateChange: func(name string, from, to gobreaker.State) {
            fmt.Printf("[circuit-breaker] %s: %s → %s\n", name, from, to)
            // TODO Fase 9: recordCircuitBreakerStateChange(name, from.String(), to.String())
        },
    }

    return &CircuitBreaker{
        cb:   gobreaker.NewCircuitBreaker(settings),
        name: serviceName,
    }
}

// Execute menjalankan fn dengan circuit breaker protection
func (cb *CircuitBreaker) Execute(fn func() (interface{}, error)) (interface{}, error) {
    result, err := cb.cb.Execute(fn)
    if err == gobreaker.ErrOpenState {
        return nil, fmt.Errorf("service %s is unavailable (circuit breaker open)", cb.name)
    }
    if err == gobreaker.ErrTooManyRequests {
        return nil, fmt.Errorf("service %s is recovering (circuit breaker half-open, too many requests)", cb.name)
    }
    return result, err
}

// State mengembalikan state saat ini: CLOSED, OPEN, HALF-OPEN
func (cb *CircuitBreaker) State() string {
    return cb.cb.State().String()
}
```

### Retry dengan Exponential Backoff

```go
// pkg/resilience/retry.go
package resilience

import (
    "context"
    "fmt"
    "math"
    "math/rand"
    "time"
)

type RetryConfig struct {
    MaxAttempts  int
    InitialDelay time.Duration
    MaxDelay     time.Duration
    Multiplier   float64
    JitterFactor float64  // 0.25 = ±25% jitter
}

// DefaultRetryConfig adalah setting yang direkomendasikan untuk microservices
var DefaultRetryConfig = RetryConfig{
    MaxAttempts:  3,
    InitialDelay: 500 * time.Millisecond,
    MaxDelay:     10 * time.Second,
    Multiplier:   2.0,
    JitterFactor: 0.25,
}

// Retry menjalankan fn hingga berhasil atau habis attempt
func Retry(ctx context.Context, cfg RetryConfig, fn func() error) error {
    var lastErr error

    for attempt := 0; attempt < cfg.MaxAttempts; attempt++ {
        // Cek context sebelum setiap attempt
        if ctx.Err() != nil {
            return ctx.Err()
        }

        if err := fn(); err == nil {
            return nil  // sukses!
        } else {
            lastErr = err
        }

        // Jangan tunggu setelah attempt terakhir
        if attempt == cfg.MaxAttempts-1 {
            break
        }

        // Exponential backoff: delay * multiplier^attempt
        delay := time.Duration(float64(cfg.InitialDelay) * math.Pow(cfg.Multiplier, float64(attempt)))
        if delay > cfg.MaxDelay {
            delay = cfg.MaxDelay
        }

        // Tambahkan jitter untuk mencegah thundering herd
        // (banyak client retry pada waktu yang sama → DDoS ke service yang baru recover)
        if cfg.JitterFactor > 0 {
            jitter := time.Duration(float64(delay) * cfg.JitterFactor * (rand.Float64()*2 - 1))
            delay += jitter
        }

        select {
        case <-ctx.Done():
            return ctx.Err()
        case <-time.After(delay):
            // tunggu sebelum retry berikutnya
        }
    }

    return fmt.Errorf("operation failed after %d attempts, last error: %w", cfg.MaxAttempts, lastErr)
}
```

### Bulkhead

```go
// pkg/resilience/bulkhead.go
package resilience

import (
    "context"
    "fmt"
)

// Bulkhead membatasi concurrent calls ke satu downstream service
// Mencegah satu service lambat exhausting semua goroutine
type Bulkhead struct {
    sem  chan struct{}
    name string
}

func NewBulkhead(name string, maxConcurrent int) *Bulkhead {
    return &Bulkhead{
        sem:  make(chan struct{}, maxConcurrent),
        name: name,
    }
}

// Execute menjalankan fn dengan batasan concurrency
func (b *Bulkhead) Execute(ctx context.Context, fn func() error) error {
    select {
    case b.sem <- struct{}{}:
        // Dapat slot — jalankan fn
        defer func() { <-b.sem }()
        return fn()
    case <-ctx.Done():
        return ctx.Err()
    default:
        // Tidak ada slot — langsung tolak (non-blocking)
        return fmt.Errorf("bulkhead '%s': max concurrent calls (%d) exceeded", b.name, cap(b.sem))
    }
}

// Available mengembalikan jumlah slot yang tersedia
func (b *Bulkhead) Available() int {
    return cap(b.sem) - len(b.sem)
}
```

### Resilient Client — Semua Pattern Digabung

```go
// pkg/clients/resilient_product_client.go
package clients

import (
    "context"
    "fmt"

    productv1 "github.com/kamu/ecommerce/proto/gen/product/v1"
    "github.com/kamu/ecommerce/pkg/resilience"
)

// ResilientProductClient mengombinasikan semua resilience patterns
type ResilientProductClient struct {
    underlying *ProductClient
    breaker    *resilience.CircuitBreaker
    bulkhead   *resilience.Bulkhead
    retryCfg   resilience.RetryConfig
}

func NewResilientProductClient(addr string) (*ResilientProductClient, error) {
    client, err := NewProductClient(addr)
    if err != nil {
        return nil, err
    }

    return &ResilientProductClient{
        underlying: client,
        breaker:    resilience.NewCircuitBreaker("product-service"),
        bulkhead:   resilience.NewBulkhead("product-service", 20), // max 20 concurrent
        retryCfg:   resilience.DefaultRetryConfig,
    }, nil
}

func (c *ResilientProductClient) GetProduct(ctx context.Context, id uint64) (*productv1.Product, error) {
    var result *productv1.Product

    // Layer 1: Bulkhead — batasi concurrent calls
    err := c.bulkhead.Execute(ctx, func() error {

        // Layer 2: Circuit Breaker — stop jika service terlalu sering gagal
        res, cbErr := c.breaker.Execute(func() (interface{}, error) {

            // Layer 3: Retry — coba ulang saat gagal sementara
            var product *productv1.Product
            retryErr := resilience.Retry(ctx, c.retryCfg, func() error {
                var err error
                product, err = c.underlying.GetProduct(ctx, id)
                return err
            })

            return product, retryErr
        })

        if cbErr != nil {
            return cbErr
        }
        result = res.(*productv1.Product)
        return nil
    })

    return result, err
}

// GetProductWithFallback mencoba get product, fallback ke cache jika gagal
func (c *ResilientProductClient) GetProductWithFallback(
    ctx context.Context,
    id uint64,
    cache ProductCache,
) (*productv1.Product, error) {
    product, err := c.GetProduct(ctx, id)
    if err != nil {
        // Fallback ke cache — stale data lebih baik dari error
        cached, cacheErr := cache.Get(ctx, id)
        if cacheErr == nil && cached != nil {
            // Log bahwa kita serving stale data
            fmt.Printf("WARN: serving stale data for product %d: %v\n", id, err)
            return cached, nil
        }
        // Tidak ada di cache juga — return original error
        return nil, err
    }

    // Update cache dengan data fresh
    cache.Set(ctx, id, product)
    return product, nil
}
```

### 🏋️ Latihan 7.9

1. Test circuit breaker end-to-end: buat mock Product Service yang gagal 70% request. Jalankan 50 request. Verifikasi: (a) circuit terbuka setelah beberapa failure, (b) fast fail saat circuit open (tidak perlu tunggu timeout), (c) circuit menutup kembali setelah recovery period.
2. Buat `ResilientClient` lengkap yang mengombinasikan Circuit Breaker + Retry + Bulkhead. Test dengan 100 concurrent requests ke service yang gagal 30% — semua harus berakhir (entah sukses atau error), tidak ada goroutine yang leak.
3. Implementasikan **fallback berbasis Redis cache**: `GetProductWithFallback` yang return cached value jika service down. Buat test yang verifikasi: (a) cache diupdate saat product berhasil di-fetch, (b) cached value di-return saat service down.

---

## 📦 Modul 7.10 — Distributed Configuration

### Masalah Konfigurasi di Microservices

```
Monolith: 1 app × 1 config = 1 file → mudah
Microservices: 10 service × 4 environment = 40 config sets → ribet!

Tantangan:
  - Konsistensi config antar service (JWT secret harus sama!)
  - Secret management (database password, API keys)
  - Hot reload tanpa restart service
  - Audit: siapa ubah apa, kapan?
```

### Multi-Layer Configuration dengan Viper

```go
// config/config.go — DIPAKAI OLEH SEMUA SERVICE (copy pattern ini)
package config

import (
    "fmt"
    "strings"
    "time"

    "github.com/spf13/viper"
)

type Config struct {
    App      AppConfig
    Database DatabaseConfig
    Redis    RedisConfig
    JWT      JWTConfig
    Services ServicesConfig
    Log      LogConfig
}

type AppConfig struct {
    Name            string        `mapstructure:"name"`
    Env             string        `mapstructure:"env"`   // development, staging, production
    Port            int           `mapstructure:"port"`
    GRPCPort        int           `mapstructure:"grpc_port"`
    ShutdownTimeout time.Duration `mapstructure:"shutdown_timeout"`
    Version         string        // diisi dari build -ldflags, BUKAN dari config file
}

type DatabaseConfig struct {
    Host     string `mapstructure:"host"`
    Port     int    `mapstructure:"port"`
    Name     string `mapstructure:"name"`
    User     string `mapstructure:"user"`
    Password string `mapstructure:"password"` // dari secret/env
    SSLMode  string `mapstructure:"ssl_mode"`
    MaxOpen  int    `mapstructure:"max_open"`
    MaxIdle  int    `mapstructure:"max_idle"`
}

func (d DatabaseConfig) DSN() string {
    return fmt.Sprintf(
        "host=%s port=%d user=%s password=%s dbname=%s sslmode=%s",
        d.Host, d.Port, d.User, d.Password, d.Name, d.SSLMode,
    )
}

type RedisConfig struct {
    Host     string `mapstructure:"host"`
    Port     int    `mapstructure:"port"`
    Password string `mapstructure:"password"`
    DB       int    `mapstructure:"db"`
    PoolSize int    `mapstructure:"pool_size"`
}

func (r RedisConfig) Addr() string { return fmt.Sprintf("%s:%d", r.Host, r.Port) }

type JWTConfig struct {
    Secret        string        `mapstructure:"secret"`
    AccessExpiry  time.Duration `mapstructure:"access_expiry"`
    RefreshExpiry time.Duration `mapstructure:"refresh_expiry"`
}

type ServicesConfig struct {
    AuthServiceHTTP    string        `mapstructure:"auth_service_http"`
    ProductServiceGRPC string        `mapstructure:"product_service_grpc"`
    OrderServiceHTTP   string        `mapstructure:"order_service_http"`
    CallTimeout        time.Duration `mapstructure:"call_timeout"`
}

type LogConfig struct {
    Level  string `mapstructure:"level"`  // debug, info, warn, error
    Format string `mapstructure:"format"` // json, console
}

// Load membaca config dari:
// 1. Default values (hardcoded)
// 2. config.yaml (base config)
// 3. config.<env>.yaml (environment override)
// 4. Environment variables (highest priority, untuk secrets)
func Load(serviceName string) (*Config, error) {
    v := viper.New()
    v.SetConfigType("yaml")

    // Set semua default values
    setDefaults(v, serviceName)

    // Baca config.yaml (base config)
    v.SetConfigName("config")
    v.AddConfigPath("./config")
    v.AddConfigPath(".")
    if err := v.ReadInConfig(); err != nil {
        if _, ok := err.(viper.ConfigFileNotFoundError); !ok {
            return nil, fmt.Errorf("read base config: %w", err)
        }
        // OK jika tidak ada config.yaml — pakai defaults
    }

    // Baca config.<env>.yaml — override untuk environment tertentu
    env := v.GetString("app.env")
    if env != "" && env != "development" {
        v.SetConfigName("config." + env)
        v.MergeInConfig() // merge, bukan replace — preserves base config
    }

    // Environment variables OVERRIDE semua config file
    // Format: DATABASE_HOST   → database.host
    // Format: JWT_SECRET      → jwt.secret
    // Format: REDIS_PASSWORD  → redis.password
    v.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))
    v.AutomaticEnv()

    // Bind secrets secara eksplisit untuk kejelasan
    v.BindEnv("database.password", "DATABASE_PASSWORD")
    v.BindEnv("jwt.secret", "JWT_SECRET")
    v.BindEnv("redis.password", "REDIS_PASSWORD")

    var cfg Config
    if err := v.Unmarshal(&cfg); err != nil {
        return nil, fmt.Errorf("unmarshal config: %w", err)
    }

    if err := validate(&cfg); err != nil {
        return nil, fmt.Errorf("config validation failed: %w", err)
    }

    return &cfg, nil
}

func setDefaults(v *viper.Viper, serviceName string) {
    v.SetDefault("app.name", serviceName)
    v.SetDefault("app.env", "development")
    v.SetDefault("app.port", 8080)
    v.SetDefault("app.grpc_port", 50051)
    v.SetDefault("app.shutdown_timeout", "30s")

    v.SetDefault("database.host", "localhost")
    v.SetDefault("database.port", 5432)
    v.SetDefault("database.ssl_mode", "disable")
    v.SetDefault("database.max_open", 25)
    v.SetDefault("database.max_idle", 5)

    v.SetDefault("redis.host", "localhost")
    v.SetDefault("redis.port", 6379)
    v.SetDefault("redis.db", 0)
    v.SetDefault("redis.pool_size", 10)

    v.SetDefault("jwt.access_expiry", "15m")
    v.SetDefault("jwt.refresh_expiry", "168h")

    v.SetDefault("services.call_timeout", "10s")
    v.SetDefault("log.level", "info")
    v.SetDefault("log.format", "json")
}

// validate memastikan semua required fields ada sebelum service start
// Ini mencegah service start dengan konfigurasi yang tidak lengkap
func validate(cfg *Config) error {
    if cfg.JWT.Secret == "" {
        return fmt.Errorf("JWT_SECRET is required — set via environment variable")
    }
    if cfg.Database.Password == "" {
        return fmt.Errorf("DATABASE_PASSWORD is required — set via environment variable")
    }
    if len(cfg.JWT.Secret) < 32 {
        return fmt.Errorf("JWT_SECRET must be at least 32 characters long")
    }
    if cfg.App.Port < 1 || cfg.App.Port > 65535 {
        return fmt.Errorf("app.port must be between 1 and 65535, got: %d", cfg.App.Port)
    }
    return nil
}
```

### Config Files per Environment

```yaml
# config/config.yaml — Base defaults (commit ke git)
app:
  name: auth-service
  env: development
  port: 8080
  grpc_port: 50051
  shutdown_timeout: 30s

database:
  host: localhost
  port: 5432
  name: authdb
  user: authuser
  ssl_mode: disable
  max_open: 25
  max_idle: 5

redis:
  host: localhost
  port: 6379

services:
  product_service_grpc: localhost:50052
  auth_service_http: http://localhost:8001
  call_timeout: 10s

log:
  level: debug
  format: console
```

```yaml
# config/config.production.yaml — Production overrides (commit ke git)
# Secrets masih via env vars, ini hanya non-sensitive config
database:
  host: postgres-auth.internal
  ssl_mode: require
  max_open: 100
  max_idle: 20

services:
  product_service_grpc: product-service.production.svc.cluster.local:50051
  auth_service_http: http://auth-service.production.svc.cluster.local:8080
  call_timeout: 5s

log:
  level: warn
  format: json
```

### 🏋️ Latihan 7.10

1. Buat sistem config untuk 3 environment: `config.yaml` (dev defaults), `config.staging.yaml` (staging overrides), `config.production.yaml` (prod settings). Test bahwa env-specific file otomatis di-load berdasarkan `APP_ENV` env var.
2. Buat **startup validation yang komprehensif**: cek semua required config ada, URL format valid, port dalam range 1-65535, timeout lebih dari 0. Print pesan error yang jelas tentang env var apa yang kurang.
3. Buat `ConfigWatcher` yang reload config saat file berubah menggunakan `viper.WatchConfig()`. Test dengan mengubah log level dari `debug` ke `warn` tanpa restart service — verifikasi level log berubah live.

---

## 📦 Modul 7.11 — Health Checks & Readiness

### Tiga Jenis Probe

```
LIVENESS PROBE: "Apakah service masih hidup?"
  Kubernetes: jika gagal → restart pod
  Docker Compose: jika gagal → restart container
  
  Implementasi: endpoint sederhana, tidak cek dependencies
  Gagal hanya jika: deadlock, panic loop, tidak bisa serve request sama sekali

READINESS PROBE: "Apakah service siap menerima traffic?"
  Kubernetes: jika gagal → hapus dari Service (tidak terima traffic baru)
  Load Balancer: jika gagal → stop kirim traffic
  
  Implementasi: cek SEMUA dependencies (DB, Redis, downstream services)
  Gagal jika: DB tidak bisa diakses, Redis down, dependency critical down

STARTUP PROBE: "Apakah service sudah selesai startup?"
  Kubernetes: untuk service yang butuh waktu lama startup
  Mencegah liveness probe fail saat service masih loading
```

### Implementasi Health Handler

```go
// internal/delivery/http/handler/health_handler.go
package handler

import (
    "context"
    "fmt"
    "net/http"
    "runtime"
    "time"

    "github.com/gin-gonic/gin"
    "gorm.io/gorm"
)

type HealthStatus string

const (
    Healthy   HealthStatus = "healthy"
    Degraded  HealthStatus = "degraded"
    Unhealthy HealthStatus = "unhealthy"
)

type CheckResult struct {
    Status  HealthStatus `json:"status"`
    Latency string       `json:"latency_ms"`
    Error   string       `json:"error,omitempty"`
    Details interface{}  `json:"details,omitempty"`
}

type HealthHandler struct {
    db        *gorm.DB
    rdb       RedisClient  // interface untuk mockability
    startTime time.Time
    version   string
    service   string
}

type RedisClient interface {
    Ping(ctx context.Context) error
}

func NewHealthHandler(db *gorm.DB, rdb RedisClient, version, service string) *HealthHandler {
    return &HealthHandler{
        db:        db,
        rdb:       rdb,
        startTime: time.Now(),
        version:   version,
        service:   service,
    }
}

// GET /health/live — HANYA cek apakah proses masih berjalan
// Tidak cek DB, Redis, dll — biarkan seringkas mungkin
func (h *HealthHandler) Liveness(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "status":  "alive",
        "service": h.service,
        "version": h.version,
        "uptime":  time.Since(h.startTime).Round(time.Second).String(),
        "time":    time.Now().UTC(),
    })
}

// GET /health/ready — cek semua dependencies
// Dipakai oleh Kubernetes dan Load Balancer untuk routing decision
func (h *HealthHandler) Readiness(c *gin.Context) {
    ctx, cancel := context.WithTimeout(c.Request.Context(), 5*time.Second)
    defer cancel()

    checks := make(map[string]CheckResult)
    overallStatus := Healthy

    // Cek Database
    dbCheck := h.checkDatabase(ctx)
    checks["database"] = dbCheck
    if dbCheck.Status == Unhealthy {
        overallStatus = Unhealthy
    } else if dbCheck.Status == Degraded && overallStatus == Healthy {
        overallStatus = Degraded
    }

    // Cek Redis (jika dikonfigurasi)
    if h.rdb != nil {
        redisCheck := h.checkRedis(ctx)
        checks["redis"] = redisCheck
        if redisCheck.Status == Unhealthy {
            overallStatus = Unhealthy
        }
    }

    statusCode := http.StatusOK
    if overallStatus == Unhealthy {
        statusCode = http.StatusServiceUnavailable // 503
    }

    c.JSON(statusCode, gin.H{
        "status":    overallStatus,
        "service":   h.service,
        "version":   h.version,
        "uptime":    time.Since(h.startTime).Round(time.Second).String(),
        "timestamp": time.Now().UTC(),
        "checks":    checks,
    })
}

// GET /health/info — informasi detail runtime (untuk admin/monitoring tools)
func (h *HealthHandler) Info(c *gin.Context) {
    var memStats runtime.MemStats
    runtime.ReadMemStats(&memStats)

    c.JSON(http.StatusOK, gin.H{
        "status":  "ok",
        "service": h.service,
        "version": h.version,
        "uptime":  time.Since(h.startTime).Round(time.Second).String(),
        "runtime": gin.H{
            "go_version":  runtime.Version(),
            "goroutines":  runtime.NumGoroutine(),
            "alloc_mb":    fmt.Sprintf("%.2f", float64(memStats.Alloc)/1024/1024),
            "total_alloc_mb": fmt.Sprintf("%.2f", float64(memStats.TotalAlloc)/1024/1024),
            "num_gc":      memStats.NumGC,
        },
    })
}

func (h *HealthHandler) checkDatabase(ctx context.Context) CheckResult {
    start := time.Now()

    sqlDB, err := h.db.DB()
    if err != nil {
        return CheckResult{
            Status:  Unhealthy,
            Latency: latencyMs(start),
            Error:   "failed to get sql.DB: " + err.Error(),
        }
    }

    if err := sqlDB.PingContext(ctx); err != nil {
        return CheckResult{
            Status:  Unhealthy,
            Latency: latencyMs(start),
            Error:   "ping failed: " + err.Error(),
        }
    }

    // Cek connection pool health
    stats := sqlDB.Stats()
    status := Healthy
    details := map[string]interface{}{
        "open_connections":   stats.OpenConnections,
        "in_use":            stats.InUse,
        "idle":              stats.Idle,
        "max_open":          stats.MaxOpenConnections,
    }

    // Degraded jika pool utilisasi > 80%
    if stats.MaxOpenConnections > 0 {
        utilization := float64(stats.InUse) / float64(stats.MaxOpenConnections)
        if utilization > 0.8 {
            status = Degraded
            details["warning"] = fmt.Sprintf("high pool utilization: %.0f%%", utilization*100)
        }
    }

    return CheckResult{
        Status:  status,
        Latency: latencyMs(start),
        Details: details,
    }
}

func (h *HealthHandler) checkRedis(ctx context.Context) CheckResult {
    start := time.Now()
    if err := h.rdb.Ping(ctx); err != nil {
        return CheckResult{
            Status:  Unhealthy,
            Latency: latencyMs(start),
            Error:   "ping failed: " + err.Error(),
        }
    }
    return CheckResult{Status: Healthy, Latency: latencyMs(start)}
}

func latencyMs(start time.Time) string {
    return fmt.Sprintf("%d", time.Since(start).Milliseconds())
}

// RegisterRoutes mendaftarkan semua health routes
func (h *HealthHandler) RegisterRoutes(r *gin.Engine) {
    health := r.Group("/health")
    health.GET("/live", h.Liveness)
    health.GET("/ready", h.Readiness)
    health.GET("/info", h.Info)
}
```

### 🏋️ Latihan 7.11

1. Implementasikan `HealthHandler` lengkap untuk semua service. Tambahkan check: (a) goroutine count (warn jika > 1000, error jika > 5000), (b) memory usage (warn jika heap > 80% dari limit), (c) disk space jika service write ke disk.
2. Integrasikan health checks ke `docker-compose.yml` dengan `condition: service_healthy` di `depends_on`. Simulasikan DB down (stop container) dan verifikasi service tidak start sebelum DB ready.
3. Buat **Composite Health Endpoint** di API Gateway: `GET /api/v1/admin/health` yang mengumpulkan readiness dari semua upstream services secara paralel dan tampilkan summary (persentase services yang healthy).

---

## 📦 Modul 7.12 — Kubernetes Fundamentals

### Konsep Inti Kubernetes

```
POD
  Unit terkecil di Kubernetes.
  Berisi 1+ container yang share network dan storage.
  Pods bersifat ephemeral — bisa mati kapan saja!

DEPLOYMENT
  Mengelola N replika Pod yang identik.
  Handle: rolling updates, rollback, scaling.
  "Saya mau 3 instance auth-service, selalu."

SERVICE
  Stable endpoint untuk akses ke Pods.
  Pods bisa mati dan lahir, Service DNS tetap sama.
  Type: ClusterIP (internal), NodePort, LoadBalancer.

INGRESS
  Load balancer eksternal.
  Route traffic berdasarkan hostname/path.
  "api.myapp.com/auth → auth-service"

CONFIGMAP & SECRET
  ConfigMap: konfigurasi non-rahasia (env, file)
  Secret: data rahasia (password, token) — base64 encoded
  Keduanya bisa di-inject sebagai env vars atau volume mounts.

NAMESPACE
  Isolasi logical di dalam satu cluster.
  Gunakan untuk: dev, staging, production.
```

### Manifest Kubernetes Lengkap

```yaml
# k8s/auth-service/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    env: production

---
# k8s/auth-service/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: auth-service-config
  namespace: production
data:
  APP_ENV: "production"
  APP_PORT: "8080"
  LOG_LEVEL: "warn"
  LOG_FORMAT: "json"
  DATABASE_HOST: "postgres-auth.production.svc.cluster.local"
  DATABASE_PORT: "5432"
  DATABASE_NAME: "authdb"
  DATABASE_USER: "authuser"
  DATABASE_SSL_MODE: "require"

---
# k8s/auth-service/secret.yaml
# JANGAN commit secret dengan value asli ke git!
# Gunakan Sealed Secrets, External Secrets, atau Vault untuk produksi
apiVersion: v1
kind: Secret
metadata:
  name: auth-service-secret
  namespace: production
type: Opaque
# nilai HARUS base64 encoded: echo -n "mypassword" | base64
data:
  DATABASE_PASSWORD: bXlkYXRhYmFzZXBhc3M=
  JWT_SECRET: c3VwZXItc2VjcmV0LWtleS1taW5pbXVtLTMyLWNoYXJz
  REDIS_PASSWORD: cmVkaXNwYXNzd29yZA==

---
# k8s/auth-service/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-service
  namespace: production
  labels:
    app: auth-service
    version: "1.0.0"
    managed-by: kubectl
spec:
  replicas: 3

  selector:
    matchLabels:
      app: auth-service

  # Zero-downtime deployment strategy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1   # max 1 pod unavailable saat update
      maxSurge: 1         # max 1 extra pod saat update

  template:
    metadata:
      labels:
        app: auth-service
        version: "1.0.0"

    spec:
      # Security: jangan jalankan sebagai root
      securityContext:
        runAsNonRoot: true
        runAsUser: 65532     # nonroot user di distroless
        fsGroup: 65532

      # Graceful termination: tunggu 30 detik
      terminationGracePeriodSeconds: 30

      containers:
        - name: auth-service
          image: ghcr.io/mycompany/auth-service:1.0.0
          imagePullPolicy: IfNotPresent

          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
            - name: grpc
              containerPort: 50051
              protocol: TCP

          # Config dari ConfigMap
          envFrom:
            - configMapRef:
                name: auth-service-config
            - secretRef:
                name: auth-service-secret

          # Env vars tambahan (pod metadata)
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace

          # Resource limits — WAJIB di production!
          # Tanpa ini, satu pod bisa konsumsi semua resource node
          resources:
            requests:
              cpu: "100m"      # 0.1 CPU — minimum yang diminta
              memory: "128Mi"  # 128MB — minimum yang diminta
            limits:
              cpu: "500m"      # 0.5 CPU — tidak boleh lebih dari ini
              memory: "256Mi"  # 256MB — OOM killed jika lebih dari ini

          # Probes
          livenessProbe:
            httpGet:
              path: /health/live
              port: 8080
            initialDelaySeconds: 10   # tunggu 10 detik sebelum mulai check
            periodSeconds: 30         # check setiap 30 detik
            failureThreshold: 3       # restart setelah 3x gagal berturut
            timeoutSeconds: 5

          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
            failureThreshold: 3
            timeoutSeconds: 5

          startupProbe:
            httpGet:
              path: /health/live
              port: 8080
            failureThreshold: 30  # 30 × 10s = 5 menit startup timeout
            periodSeconds: 10

---
# k8s/auth-service/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: auth-service
  namespace: production
spec:
  # Match pods dengan label ini
  selector:
    app: auth-service

  ports:
    - name: http
      port: 8080
      targetPort: 8080
      protocol: TCP
    - name: grpc
      port: 50051
      targetPort: 50051
      protocol: TCP

  # ClusterIP: hanya accessible dari dalam cluster
  type: ClusterIP

---
# k8s/ingress.yaml — entry point dari internet
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  ingressClassName: nginx
  rules:
    - host: api.myecommerce.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-gateway
                port:
                  number: 8080

---
# k8s/auth-service/hpa.yaml — auto scaling
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: auth-service-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: auth-service

  minReplicas: 2    # minimum selalu 2 untuk HA
  maxReplicas: 10   # maksimum 10 saat peak

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70  # scale up jika CPU > 70%
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

### Perintah kubectl yang Penting

```bash
# ---- Deploy ----
kubectl apply -f k8s/auth-service/           # apply semua file di folder
kubectl apply -f k8s/                        # apply semua recursive

# ---- Monitor ----
kubectl get pods -n production               # list pods
kubectl get pods -n production -w           # watch (realtime update)
kubectl get all -n production               # semua resources
kubectl describe pod <pod-name> -n prod     # detail + events
kubectl logs <pod-name> -n prod             # logs
kubectl logs -f <pod-name> -n prod         # follow logs
kubectl logs -f <pod-name> -c <container>  # logs container spesifik

# ---- Debug ----
kubectl exec -it <pod-name> -n prod -- sh  # masuk ke pod
kubectl port-forward svc/auth-service 8001:8080 -n prod  # akses dari lokal
kubectl top pods -n prod                    # CPU/memory usage

# ---- Scaling ----
kubectl scale deployment auth-service --replicas=5 -n prod
kubectl rollout status deployment/auth-service -n prod  # tunggu selesai

# ---- Update ----
kubectl set image deployment/auth-service \
  auth-service=ghcr.io/mycompany/auth-service:1.1.0 -n prod

kubectl rollout history deployment/auth-service -n prod  # lihat history
kubectl rollout undo deployment/auth-service -n prod     # rollback ke versi sebelumnya
kubectl rollout undo deployment/auth-service \
  --to-revision=2 -n prod                                # rollback ke revision tertentu

# ---- Config ----
kubectl get configmap auth-service-config -n prod -o yaml
kubectl get secret auth-service-secret -n prod -o yaml

# Decode secret (base64)
kubectl get secret auth-service-secret -n prod \
  -o jsonpath='{.data.JWT_SECRET}' | base64 -d
```

### Setup Minikube untuk Development

```bash
# Install minikube
brew install minikube          # macOS
# atau download dari minikube.sigs.k8s.io

# Start cluster
minikube start --cpus=4 --memory=4096

# Enable addons
minikube addons enable ingress
minikube addons enable metrics-server

# Load local image ke minikube (tanpa push ke registry)
minikube image load auth-service:dev

# Akses service
minikube service api-gateway -n production

# Dashboard
minikube dashboard
```

### 🏋️ Latihan 7.12

1. Buat manifest Kubernetes lengkap (Namespace + Deployment + Service + ConfigMap + Secret) untuk Auth Service. Deploy ke **minikube**. Verifikasi dengan: `kubectl get pods`, `kubectl logs`, dan `kubectl port-forward` untuk akses service dari lokal.
2. Implementasikan **HPA** untuk Auth Service dengan metric CPU > 70%. Load test dengan `hey -n 10000 -c 100 http://localhost:8001/health/live`. Verifikasi bahwa pods bertambah otomatis.
3. Lakukan **rolling update** dari v1.0.0 ke v1.1.0 (buat image dengan tag berbeda). Verifikasi **zero downtime** dengan: `while true; do curl -s http://localhost:8001/health/live | jq -r .status; sleep 0.2; done` — semua output harus "alive".

---

## 📦 Modul 7.13 — Putting It All Together

### Arsitektur Final E-Commerce System

```
Internet
    │
    ▼
┌─────────────────────────────────────────────────────┐
│                   API Gateway :8080                  │
│  Auth JWT validation, Rate limiting, Routing         │
└──────────┬──────────────┬──────────────┬────────────┘
           │              │              │
           ▼              ▼              ▼
┌──────────────┐  ┌────────────────┐  ┌──────────────┐
│ Auth Service │  │Product Service │  │Order Service │
│ :8001/:50051 │  │ :8002/:50052   │  │ :8003        │
│ PostgreSQL   │  │ PostgreSQL     │  │ PostgreSQL   │
└──────────────┘  └────────────────┘  └──────┬───────┘
                                             │
                                     ┌───────▼───────┐
                                     │ Message Broker│ ← Fase 8!
                                     │ Kafka/Rabbit  │
                                     └───────┬───────┘
                                             │
                                     ┌───────▼───────┐
                                     │Notification   │
                                     │Service :8004  │
                                     └───────────────┘
Shared Infrastructure:
  Redis  → caching, rate limiting, session
  Jaeger → distributed tracing (Fase 9)
  Prometheus + Grafana → metrics (Fase 9)
```

### Service Communication Matrix

```
Dari → Ke      Auth    Product  Order   Notif   Gateway
Gateway    → HTTP    HTTP     HTTP    —       —
Order      → gRPC   gRPC     —       Kafka   —
Notif      → —      —        Kafka   —       —

Sync (gRPC/HTTP): butuh response langsung
Async (Kafka): side effects, eventual consistency OK
```

### Production Readiness Checklist

```
Infrastructure:
  [ ] Semua service punya Dockerfile multi-stage (< 20MB)
  [ ] docker-compose.yml berjalan dengan docker compose up -d
  [ ] Health checks (liveness + readiness) tersedia semua service
  [ ] Secrets tidak di-hardcode, tidak di-commit ke git

Resilience:
  [ ] Circuit breaker di setiap external call
  [ ] Retry dengan exponential backoff
  [ ] Timeout di setiap HTTP/gRPC call
  [ ] Graceful shutdown (handle SIGTERM)
  [ ] Bulkhead untuk critical external calls

Security:
  [ ] Service tidak berjalan sebagai root (UID != 0)
  [ ] JWT validated di gateway — tidak perlu tiap service
  [ ] Internal service tidak accessible dari luar cluster
  [ ] Secrets di-manage dengan proper secret management

Observability (akan dicover di Fase 9):
  [ ] Structured logging dengan request ID
  [ ] Metrics endpoint /metrics untuk Prometheus
  [ ] Health check endpoints /health/live dan /health/ready
  [ ] Distributed tracing dengan trace ID propagation

Operations:
  [ ] Kubernetes manifests (Deployment + Service + ConfigMap + Secret)
  [ ] HPA untuk service dengan variable load
  [ ] Rolling update strategy configured
  [ ] Runbook untuk common failure scenarios
```

### 🏋️ Latihan 7.13

1. Buat **Architecture Decision Record** untuk 3 keputusan arsitektur utama dalam sistem ini: (a) mengapa pakai gRPC untuk Order-Product communication bukan REST, (b) mengapa choreography saga bukan orchestration, (c) mengapa API Gateway pattern. Format: Context → Options Considered → Decision → Consequences.
2. Buat **end-to-end integration test** yang verifikasi happy path checkout via docker-compose: register → login → list products → create order → simulate payment webhook → verify order status CONFIRMED. Script bash atau Go test file.
3. Buat **runbook** untuk skenario: "Order Service tidak bisa connect ke Product Service". Isi: (a) gejala yang user rasakan, (b) command diagnosis, (c) langkah recovery, (d) cara verifikasi sudah pulih, (e) cara mencegah terulang.

---

### Graceful Shutdown — Wajib di Production

Graceful shutdown memastikan service tidak mati di tengah-tengah request, merusak data, atau meninggalkan koneksi tergantung.

```go
// cmd/api/main.go — Pattern graceful shutdown yang benar
package main

import (
    "context"
    "fmt"
    "net"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"

    "go.uber.org/zap"
    "google.golang.org/grpc"
)

func main() {
    cfg := config.MustLoad("auth-service")
    log := logger.MustNew(cfg.Log.Level, cfg.Log.Format, "auth-service", cfg.App.Version)
    defer log.Sync()

    // Setup HTTP server
    router := setupRouter(cfg, log)
    httpServer := &http.Server{
        Addr:         fmt.Sprintf(":%d", cfg.App.Port),
        Handler:      router,
        ReadTimeout:  15 * time.Second,
        WriteTimeout: 15 * time.Second,
        IdleTimeout:  60 * time.Second,
    }

    // Setup gRPC server
    grpcServer := grpc.NewServer(/* interceptors */)
    grpcListener, err := net.Listen("tcp", fmt.Sprintf(":%d", cfg.App.GRPCPort))
    if err != nil {
        log.Fatal("failed to listen gRPC", zap.Error(err))
    }

    // Start servers di goroutines
    go func() {
        log.Info("HTTP server starting", zap.String("addr", httpServer.Addr))
        if err := httpServer.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatal("HTTP server failed", zap.Error(err))
        }
    }()

    go func() {
        log.Info("gRPC server starting", zap.Int("port", cfg.App.GRPCPort))
        if err := grpcServer.Serve(grpcListener); err != nil {
            log.Fatal("gRPC server failed", zap.Error(err))
        }
    }()

    // ====================================================
    // GRACEFUL SHUTDOWN
    // ====================================================
    // Tunggu signal dari OS: SIGTERM (Kubernetes) atau SIGINT (Ctrl+C)
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)

    sig := <-quit // BLOCK sampai signal diterima
    log.Info("shutdown signal received", zap.String("signal", sig.String()))

    // Buat context dengan timeout untuk shutdown
    // Kubernetes menunggu terminationGracePeriodSeconds (default 30s)
    ctx, cancel := context.WithTimeout(context.Background(), cfg.App.ShutdownTimeout)
    defer cancel()

    // Step 1: Stop terima request baru dari HTTP
    log.Info("shutting down HTTP server...")
    if err := httpServer.Shutdown(ctx); err != nil {
        log.Error("HTTP server shutdown error", zap.Error(err))
    }

    // Step 2: Stop terima request baru dari gRPC, tunggu yang sedang jalan selesai
    log.Info("shutting down gRPC server...")
    grpcServer.GracefulStop()

    // Step 3: Tutup koneksi database
    log.Info("closing database connections...")
    if sqlDB, err := db.DB(); err == nil {
        sqlDB.Close()
    }

    // Step 4: Flush logs (Zap punya buffer)
    log.Info("shutdown complete")
    log.Sync()
}
```

### Kubernetes Termination Flow

```
1. kubectl delete pod / rolling update
   │
   ▼
2. Pod state = Terminating
   Kubernetes STOP kirim traffic baru ke pod ini (remove dari Service)
   │
   ▼
3. SIGTERM dikirim ke container
   (ini yang kita handle dengan signal.Notify)
   │
   ▼
4. Aplikasi menyelesaikan request yang in-flight
   (dengan timeout = terminationGracePeriodSeconds = 30s)
   │
   ▼
5. SIGKILL dikirim jika timeout terlewati
   (paksa kill, request yang belum selesai akan error)

PENTING:
  terminationGracePeriodSeconds di Kubernetes HARUS > ShutdownTimeout di aplikasi!
  Kubernetes (30s) > Aplikasi (25s) → aplikasi selesai sebelum Kubernetes paksa kill
```


## 🎯 Review & Checkpoint Fase 7

### Konseptual
- [ ] Kapan sebaiknya pakai microservices dan kapan monolith lebih baik?
- [ ] Jelaskan Conway's Law dan bagaimana pengaruhnya ke desain service
- [ ] Apa itu Distributed Monolith dan mengapa lebih buruk dari keduanya?
- [ ] Perbedaan Choreography Saga vs Orchestration Saga — kapan pakai masing-masing?
- [ ] Circuit breaker memiliki 3 state — jelaskan transisi antar state
- [ ] Perbedaan Liveness Probe vs Readiness Probe di Kubernetes?
- [ ] Mengapa setiap microservice harus punya database sendiri?
- [ ] Apa itu thundering herd dan bagaimana jitter mengatasi ini?

### Praktis
- [ ] Bisa membuat Dockerfile multi-stage yang menghasilkan image < 20MB
- [ ] Bisa membuat docker-compose.yml multi-service dengan health checks yang benar
- [ ] Bisa mengimplementasikan API Gateway dengan reverse proxy dan JWT validation
- [ ] Bisa mengimplementasikan circuit breaker dengan gobreaker
- [ ] Bisa membuat Kubernetes manifests (Deployment + Service + ConfigMap + Secret + HPA)
- [ ] Bisa melakukan rolling update tanpa downtime
- [ ] Bisa trace request flow melewati multiple services menggunakan request ID
- [ ] **Menyelesaikan project E-Commerce Microservices System**

---

## 🎯 Project Akhir Fase 7

Kerjakan project berdasarkan PRD di: **`FASE-7-PRD-Ecommerce-Microservices.md`**

---

*Setelah selesai Fase 7, lanjut ke `FASE-8-Message-Broker.md`*

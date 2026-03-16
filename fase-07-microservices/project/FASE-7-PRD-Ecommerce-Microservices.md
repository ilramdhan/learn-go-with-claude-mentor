# 📋 PRD — E-Commerce Microservices System

> **Fase:** 7 — Microservices Architecture
> **Tipe Project:** Full Microservices System (4 Services + API Gateway)
> **Estimasi Pengerjaan:** 14–21 hari
> **Konsep yang Diuji:** Docker, Docker Compose, API Gateway, gRPC, Saga, Circuit Breaker, Health Checks, Kubernetes

---

## 🎯 Tujuan Project

Membangun **E-Commerce Microservices System** yang terdiri dari 5 service yang saling berkomunikasi, menggunakan semua pattern yang dipelajari di Fase 7. Ini adalah project portfolio yang menunjukkan kemampuan membangun sistem terdistribusi level industri.

---

## 📌 Sistem Overview

```
Internet
    │
    ▼
API Gateway (port 8080)
    ├── Auth Service (port 8001, gRPC 50051)
    ├── Product Service (port 8002, gRPC 50052)
    └── Order Service (port 8003)
              │
              └── [Kafka] → Notification Service (port 8004)
              
Shared: PostgreSQL (per service), Redis
```

---

## 🏗️ Service Breakdown

### 1. Auth Service (dari Fase 4, di-Dockerize)
**Responsibility:** User authentication dan authorization
- Register, login, refresh token, logout
- JWT token management
- User profile management

**Tech stack:** Go + Gin + PostgreSQL + JWT + Clean Architecture

### 2. Product Service (dari Fase 5, tambahkan HTTP)
**Responsibility:** Product catalog dan inventory
- Product CRUD (admin)
- Stock management
- gRPC untuk inter-service calls (dari Order Service)
- REST untuk client-facing calls (via Gateway)

**Tech stack:** Go + Gin + gRPC + PostgreSQL + Clean Architecture

### 3. Order Service (dari Fase 6, tambahkan komunikasi)
**Responsibility:** Order lifecycle management
- Create, view, cancel order
- Choreography saga untuk checkout
- Komunikasi ke Product Service via gRPC

**Tech stack:** Go + Gin + gRPC client + PostgreSQL + DDD + Saga

### 4. API Gateway (baru di Fase 7)
**Responsibility:** Single entry point, routing, auth
- Reverse proxy ke semua service
- JWT validation (tidak call Auth Service setiap request)
- Rate limiting
- Request logging

**Tech stack:** Go + Gin

### 5. Notification Service (sederhana — untuk belajar event consumer)
**Responsibility:** Send notifications
- Listen event dari Order Service (via in-memory event bus atau Kafka sederhana)
- Log notifikasi (simulasi kirim email/SMS)

**Tech stack:** Go (event consumer, sederhana)

---

## ✅ Functional Requirements

### FR-01: Multi-Service Setup
- Semua service berjalan dari satu perintah: `docker compose up -d`
- Setiap service punya database PostgreSQL sendiri
- Health checks terkonfigurasi dan berjalan
- Service dependencies diatur dengan benar (DB healthy sebelum service start)

### FR-02: API Gateway
- Semua request masuk melalui satu port (8080)
- Public routes tidak butuh auth: `/api/v1/auth/register`, `/api/v1/auth/login`, `GET /api/v1/products`
- Protected routes validate JWT di gateway
- Admin routes validate JWT + role = admin
- Rate limiting 100 req/menit per IP

### FR-03: Inter-Service Communication
- Order Service ↔ Product Service via gRPC (check stock, update stock)
- Connection pool untuk gRPC connections
- Timeout di setiap inter-service call
- Circuit breaker untuk Product Service client di Order Service
- Metadata propagation (request ID, user ID)

### FR-04: Checkout Saga
- Create order → check & reserve stock → process payment (simulasi) → confirm order
- Choreography saga: setiap step publish event
- Compensating transactions:
  - Stock tidak tersedia → cancel order
  - Payment gagal → release stock + cancel order
- Idempotency: event handler cek duplicate

### FR-05: Resilience
- Circuit breaker di Product Client (Order Service)
- Retry dengan exponential backoff untuk transient errors
- Graceful shutdown semua service (handle SIGTERM)
- Health checks: liveness + readiness per service

### FR-06: Observability Dasar
- Request ID di-generate di API Gateway dan di-propagasi ke semua service
- Structured logging (JSON format di production)
- Health endpoint: `GET /health/live`, `GET /health/ready`, `GET /health/info`

---

## 🚫 Non-Functional Requirements

- Semua service response time < 200ms untuk operasi normal
- System harus tetap partial functional jika satu service down
  (Product Service down → bisa login, lihat produk dari cache, tapi tidak bisa checkout)
- Data tidak hilang jika service restart (persistent volumes)
- `docker compose down && docker compose up -d` harus berjalan tanpa masalah

---

## 📁 Struktur Project

```
ecommerce-microservices/
├── services/
│   ├── auth-service/              ← dari Fase 4 (tambahkan Dockerfile)
│   │   ├── Dockerfile
│   │   ├── cmd/api/main.go
│   │   └── config/config.yaml
│   │
│   ├── product-service/           ← dari Fase 5 (tambahkan HTTP handler)
│   │   ├── Dockerfile
│   │   ├── cmd/api/main.go
│   │   └── config/config.yaml
│   │
│   ├── order-service/             ← dari Fase 6 (tambahkan saga + gRPC client)
│   │   ├── Dockerfile
│   │   ├── cmd/api/main.go
│   │   └── config/config.yaml
│   │
│   ├── api-gateway/               ← BARU di Fase 7
│   │   ├── Dockerfile
│   │   ├── cmd/main.go
│   │   ├── internal/
│   │   │   ├── gateway/
│   │   │   │   ├── proxy.go
│   │   │   │   └── router.go
│   │   │   └── middleware/
│   │   │       ├── auth.go
│   │   │       ├── rate_limiter.go
│   │   │       └── logger.go
│   │   └── config/config.yaml
│   │
│   └── notification-service/      ← BARU, sederhana
│       ├── Dockerfile
│       ├── cmd/main.go
│       └── internal/
│           └── handler/
│               └── event_handler.go
│
├── proto/                         ← Shared protobuf definitions
│   └── gen/
│       ├── product/v1/
│       └── order/v1/
│
├── pkg/                           ← Shared packages
│   ├── grpcpool/
│   ├── resilience/
│   │   ├── circuit_breaker.go
│   │   ├── retry.go
│   │   └── bulkhead.go
│   └── metadata/
│       └── propagate.go
│
├── k8s/                           ← Kubernetes manifests
│   ├── auth-service/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── configmap.yaml
│   │   └── secret.yaml
│   ├── product-service/
│   ├── order-service/
│   └── api-gateway/
│
├── scripts/
│   ├── wait-for-it.sh
│   └── test-e2e.sh
│
├── docker-compose.yml
├── docker-compose.override.yml    ← dev overrides (hot reload)
├── .env.example
├── Makefile
└── README.md
```

---

## 🗂️ API Specification

### Auth Service (via Gateway)

```
POST /api/v1/auth/register        → Daftar user baru
POST /api/v1/auth/login           → Login, dapat JWT token
POST /api/v1/auth/refresh         → Refresh access token
GET  /api/v1/auth/me              → Get profile (protected)
POST /api/v1/auth/logout          → Logout (protected)
```

### Product Service (via Gateway)

```
GET  /api/v1/products             → List products (public, paginated)
GET  /api/v1/products/:id         → Get product detail (public)
POST /api/v1/products             → Create product (admin)
PUT  /api/v1/products/:id         → Update product (admin)
DELETE /api/v1/products/:id       → Delete product (admin)

gRPC (internal — Order Service only):
  ProductService.GetProduct(id)
  ProductService.CheckStock(product_id, quantity)
  ProductService.UpdateStock(product_id, delta, reason)
```

### Order Service (via Gateway)

```
POST /api/v1/orders               → Create order (protected)
GET  /api/v1/orders               → List my orders (protected)
GET  /api/v1/orders/:id           → Get order detail (protected)
POST /api/v1/orders/:id/cancel    → Cancel order (protected)
POST /api/v1/orders/:id/submit    → Submit draft order (protected)

Internal webhook:
POST /internal/orders/:id/payment → Simulate payment callback
```

### API Gateway

```
GET  /health                      → Gateway health
GET  /api/v1/dashboard            → Aggregated data (protected)
```

---

## 🔄 Checkout Flow (Saga)

```
1. Client: POST /api/v1/orders (create draft order dengan items)
2. Client: POST /api/v1/orders/:id/submit
3. Order Service:
   a. Validate order (tidak empty)
   b. gRPC call → Product Service: CheckStock (semua items)
   c. gRPC call → Product Service: UpdateStock (kurangi stok)
   d. Order status → PENDING
   e. Publish internal event: OrderSubmitted
4. Simulate payment: POST /internal/orders/:id/payment
   a. Jika payment_result=success:
      - Order status → CONFIRMED
      - Publish: OrderConfirmed
      - Notification Service log: "Order X confirmed, sending email to user Y"
   b. Jika payment_result=failed:
      - gRPC call → Product Service: UpdateStock (kembalikan stok)
      - Order status → CANCELLED
      - Publish: OrderCancelled
```

---

## 🧪 Test Scenarios

### Skenario 1: Happy Path Checkout

```bash
# 1. Register dan login
curl -X POST localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice","email":"alice@test.com","password":"SecurePass123"}'

TOKEN=$(curl -s -X POST localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@test.com","password":"SecurePass123"}' \
  | jq -r '.data.access_token')

# 2. Create product (admin)
ADMIN_TOKEN="<admin_token>"
curl -X POST localhost:8080/api/v1/products \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Laptop","price":15000000,"stock":10,"sku":"ELEC-00001"}'

# 3. Create order
ORDER_ID=$(curl -s -X POST localhost:8080/api/v1/orders \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"items":[{"product_id":1,"quantity":2}]}' \
  | jq -r '.data.id')

# 4. Submit order
curl -X POST localhost:8080/api/v1/orders/$ORDER_ID/submit \
  -H "Authorization: Bearer $TOKEN"

# 5. Simulate payment success
curl -X POST localhost:8080/internal/orders/$ORDER_ID/payment \
  -H "Content-Type: application/json" \
  -d '{"payment_result":"success","payment_id":"PAY-123"}'

# 6. Verify order status = CONFIRMED
curl localhost:8080/api/v1/orders/$ORDER_ID \
  -H "Authorization: Bearer $TOKEN"
# Expected: status = CONFIRMED

# 7. Verify stock berkurang
curl localhost:8080/api/v1/products/1
# Expected: stock = 8 (dari 10 dikurangi 2)
```

### Skenario 2: Payment Failure → Stock Restored

```bash
# ... (sama sampai step 4)

# 5. Simulate payment FAILED
curl -X POST localhost:8080/internal/orders/$ORDER_ID/payment \
  -d '{"payment_result":"failed","reason":"insufficient_funds"}'

# 6. Verify order CANCELLED
curl localhost:8080/api/v1/orders/$ORDER_ID -H "Authorization: Bearer $TOKEN"
# Expected: status = CANCELLED

# 7. Verify stock DIKEMBALIKAN
curl localhost:8080/api/v1/products/1
# Expected: stock = 10 (kembali ke awal)
```

### Skenario 3: Circuit Breaker

```bash
# Stop product service
docker compose stop product-service

# Coba checkout
# Expected: fast fail dengan pesan "product service unavailable"
# Expected: response dalam < 1 detik (tidak timeout penuh)

# Start product service kembali
docker compose start product-service

# Tunggu health check pass
sleep 30

# Coba checkout lagi — harus berhasil
```

---

## 🏆 Kriteria Penilaian

### Minimum (Wajib)
- [ ] Semua 5 service berjalan dengan `docker compose up -d`
- [ ] API Gateway routing berfungsi untuk semua service
- [ ] JWT validation di gateway (protected routes)
- [ ] Order Service bisa call Product Service via gRPC
- [ ] Happy path checkout flow end-to-end berfungsi
- [ ] Health checks tersedia di semua service
- [ ] Graceful shutdown di semua service

### Good
- [ ] Choreography saga dengan compensating transactions
- [ ] Circuit breaker di Product Client
- [ ] Rate limiting di API Gateway
- [ ] Request ID propagation antar service
- [ ] docker-compose.override.yml untuk hot reload

### Excellent
- [ ] Kubernetes manifests (Deployment + Service + ConfigMap + Secret)
- [ ] HPA untuk minimal satu service
- [ ] End-to-end test script (`scripts/test-e2e.sh`)
- [ ] Idempotency di saga event handlers
- [ ] README.md dengan architecture diagram dan cara setup

---

## 📦 Dependencies per Service

```bash
# Auth Service (Fase 4 + Docker)
go get github.com/gin-gonic/gin
go get gorm.io/gorm gorm.io/driver/postgres
go get github.com/golang-jwt/jwt/v5
go get golang.org/x/crypto
go get github.com/spf13/viper

# Product Service (Fase 5 + HTTP handler)
# + semua di atas
go get google.golang.org/grpc
go get google.golang.org/protobuf

# Order Service (Fase 6 + gRPC client + saga)
# + semua di atas
go get github.com/sony/gobreaker    # circuit breaker

# API Gateway (baru)
go get github.com/gin-gonic/gin
go get github.com/golang-jwt/jwt/v5
go get go.uber.org/zap
```

---

## 📖 Panduan Pengerjaan

### Minggu 1: Foundation
**Hari 1–2:** Docker-ize semua service yang sudah ada (Auth, Product, Order)
- Buat Dockerfile multi-stage untuk masing-masing
- Verifikasi image size < 30MB per service

**Hari 3–4:** Docker Compose setup
- Buat docker-compose.yml lengkap
- Pastikan semua health checks berjalan
- Test: `docker compose up -d` → semua service healthy

**Hari 5:** API Gateway
- Implementasikan reverse proxy
- Routing semua endpoints
- JWT validation di gateway

### Minggu 2: Inter-Service Communication
**Hari 6–7:** Order ↔ Product via gRPC
- ProductClient di Order Service
- CheckStock dan UpdateStock
- Connection pool + timeout + retry

**Hari 8–9:** Saga Implementation
- Choreography saga untuk checkout
- Compensating transactions
- Test happy path dan failure cases

**Hari 10:** Circuit Breaker & Resilience
- gobreaker di ProductClient
- Test dengan Product Service down

### Minggu 3: Polish & Kubernetes
**Hari 11–12:** Observability dasar
- Request ID propagation
- Health checks lengkap
- Structured logging

**Hari 13–14:** Kubernetes
- Manifest untuk semua service
- Deploy ke minikube
- Rolling update test

**Hari 15+:** Integration testing dan dokumentasi
- End-to-end test script
- README dengan architecture diagram
- Cleanup dan refactoring

---

## 📝 Deliverable

1. **Source code** di folder `ecommerce-microservices/` dengan struktur yang rapi
2. **docker-compose.yml** yang berjalan dengan satu perintah
3. **k8s/** folder dengan manifest untuk semua service
4. **scripts/test-e2e.sh** yang otomatis test semua skenario
5. **README.md** yang berisi:
   - Architecture diagram
   - Cara setup (prerequisites + steps)
   - API documentation ringkas
   - Cara run tests
   - Known limitations

6. **Push ke GitHub** dengan commit history yang rapi

---

*Ini adalah project paling kompleks sejauh ini — ambil waktumu dan jangan rush. Quality > Speed!*

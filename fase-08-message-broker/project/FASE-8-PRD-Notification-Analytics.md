# 📋 PRD — Notification & Analytics System

> **Fase:** 8 — Message Broker & Event-Driven Architecture
> **Tipe Project:** Event-Driven Microservices (tambahan untuk Fase 7)
> **Estimasi Pengerjaan:** 7–10 hari
> **Konsep yang Diuji:** Kafka Producer/Consumer, Outbox Pattern, DLQ, Event Schema, Consumer Groups

---

## 🎯 Tujuan Project

Mengextend sistem E-Commerce dari Fase 7 dengan menambahkan **event-driven communication** menggunakan Kafka. Mengganti semua sync calls untuk side effects (notifikasi, analytics) dengan async event publishing.

---

## 📌 Arsitektur Target

```
Order Service ──gRPC──> Product Service   (TETAP sync — butuh response)
Order Service ──Kafka──> [orders topic] ──> Notification Service  (async)
                                        ──> Analytics Service     (async)
Payment events ──Kafka──> [payments topic] ──> Notification Service (async)
```

---

## ✅ Features yang Harus Diimplementasikan

### F-01: Order Events (Kafka Producer)
Order Service publish events ke topic `orders` saat:
- Order dibuat: `order.created`
- Order dikonfirmasi: `order.confirmed` (setelah payment success)
- Order dibatalkan: `order.cancelled`
- Order dikirim: `order.dispatched`

**Requirements:**
- Gunakan **Outbox Pattern**: event disimpan ke tabel `order_outbox` dalam transaksi yang sama dengan perubahan order
- OutboxWorker publish ke Kafka setiap 5 detik
- Events menggunakan `EventEnvelope` dengan version, event_id, correlation_id
- Partition key = order_id (semua events satu order di partisi yang sama)

### F-02: Notification Service (Kafka Consumer)
Service baru yang consume topic `orders` dan `payments`:

Event handlers:
- `order.confirmed` → log: "Sending email to {email}: Order #{id} confirmed, total Rp{amount}"
- `order.cancelled` → log: "Sending email to {email}: Order #{id} cancelled, reason: {reason}"
- `order.dispatched` → log: "Sending SMS to {phone}: Your order #{id} is on the way! Tracking: {tracking}"

**Requirements:**
- Consumer group ID: `notification-service-v1`
- Prefetch: 10 messages sekaligus
- Jika handler gagal → retry 3x → kirim ke DLQ topic `orders.dlq`
- Graceful shutdown: selesaikan message yang sedang diproses
- Idempotency: skip jika `event_id` sudah pernah diproses (simpan di Redis Set)

### F-03: Analytics Service (Kafka Consumer)
Service sederhana yang consume events dan update stats:

Event handlers:
- `order.confirmed` → update `daily_stats`:
  - `total_orders` + 1
  - `total_revenue` + amount
  - update `product_sales` per item
- `order.cancelled` → update `daily_stats`:
  - `cancelled_orders` + 1

Expose REST endpoints:
```
GET /analytics/daily?date=2025-01-15
GET /analytics/products/top?limit=10&days=7
GET /analytics/summary
```

**Requirements:**
- Consumer group ID: `analytics-service-v1`
- Gunakan PostgreSQL untuk store stats
- Idempotency: gunakan `event_id` untuk cegah double-counting

### F-04: Dead Letter Queue
- Topic `orders.dlq` untuk messages yang gagal 3x
- Endpoint `GET /api/v1/admin/dlq/orders` — list DLQ messages
- Endpoint `POST /api/v1/admin/dlq/orders/replay` — replay semua DLQ messages
- Endpoint `DELETE /api/v1/admin/dlq/orders/:event_id` — hapus satu DLQ message

### F-05: Schema Versioning
Semua events harus menggunakan `EventEnvelope`:
```json
{
  "event_id": "uuid-v4",
  "event_type": "order.confirmed",
  "version": 1,
  "aggregate_id": "123",
  "aggregate_name": "Order",
  "occurred_at": "2025-01-15T10:30:00Z",
  "source": "order-service",
  "correlation_id": "request-abc123",
  "data": { ... }
}
```

---

## 📁 Struktur Project Tambahan

```
ecommerce-microservices/
├── services/
│   ├── order-service/              ← UPDATE: tambahkan Outbox
│   │   ├── internal/
│   │   │   ├── infrastructure/
│   │   │   │   └── outbox/
│   │   │   │       ├── repository.go
│   │   │   │       └── worker.go
│   │   └── ...
│   │
│   ├── notification-service/       ← BARU
│   │   ├── Dockerfile
│   │   ├── cmd/main.go
│   │   ├── internal/
│   │   │   ├── handler/
│   │   │   │   ├── order_handler.go
│   │   │   │   └── payment_handler.go
│   │   │   └── idempotency/
│   │   │       └── redis_store.go
│   │   └── config/config.yaml
│   │
│   └── analytics-service/          ← BARU
│       ├── Dockerfile
│       ├── cmd/main.go
│       ├── internal/
│       │   ├── handler/
│       │   │   └── order_analytics.go
│       │   ├── repository/
│       │   │   └── stats_repository.go
│       │   └── delivery/http/
│       │       └── analytics_handler.go
│       └── config/config.yaml
│
├── pkg/kafka/                       ← BARU (shared)
│   ├── producer.go
│   ├── consumer.go
│   ├── router.go
│   └── admin.go
│
├── events/                          ← BARU (shared)
│   ├── envelope.go
│   ├── order_events.go
│   └── payment_events.go
│
└── docker-compose.yml               ← UPDATE: tambah Kafka
```

---

## 🗂️ Database Schema Tambahan

```sql
-- Order Service: Outbox table
CREATE TABLE order_outbox (
    id           BIGSERIAL PRIMARY KEY,
    event_id     UUID NOT NULL UNIQUE,
    event_type   VARCHAR(100) NOT NULL,
    aggregate_id VARCHAR(100) NOT NULL,
    payload      JSONB NOT NULL,
    published    BOOLEAN NOT NULL DEFAULT FALSE,
    published_at TIMESTAMPTZ,
    retry_count  INT NOT NULL DEFAULT 0,
    created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_outbox_unpublished ON order_outbox(published, created_at)
    WHERE NOT published AND retry_count < 5;

-- Notification Service: Processed events (idempotency)
-- Simpan di Redis sebagai SET "processed_events"
-- SADD processed_events {event_id}
-- SISMEMBER processed_events {event_id}

-- Analytics Service: Stats tables
CREATE TABLE daily_stats (
    date              DATE PRIMARY KEY,
    total_orders      INT NOT NULL DEFAULT 0,
    cancelled_orders  INT NOT NULL DEFAULT 0,
    total_revenue     DECIMAL(15,2) NOT NULL DEFAULT 0,
    updated_at        TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE product_sales (
    product_id    BIGINT PRIMARY KEY,
    product_name  VARCHAR(255),
    total_sold    INT NOT NULL DEFAULT 0,
    total_revenue DECIMAL(15,2) NOT NULL DEFAULT 0,
    updated_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE processed_events (
    event_id   UUID PRIMARY KEY,
    service    VARCHAR(50) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_processed_events_service ON processed_events(service, created_at);
```

---

## 🧪 Test Scenarios

### Scenario 1: Happy Path

```bash
# Konfirmasi order (trigger event)
curl -X POST localhost:8080/internal/orders/1/payment \
  -d '{"payment_result":"success"}'

# Verifikasi Kafka menerima event
docker exec kafka kafka-console-consumer \
  --topic orders --from-beginning \
  --bootstrap-server localhost:29092 --max-messages 1

# Verifikasi Notification Service log
docker compose logs notification-service | grep "Sending email"
# Expected: "Sending email to alice@test.com: Order #1 confirmed"

# Verifikasi Analytics update
curl localhost:8084/analytics/daily
# Expected: total_orders +1, total_revenue bertambah
```

### Scenario 2: DLQ Test

```bash
# Inject bug: ubah handler notification untuk selalu throw error
# Restart notification service
docker compose restart notification-service

# Konfirmasi beberapa order
curl -X POST localhost:8080/internal/orders/2/payment -d '{"payment_result":"success"}'
curl -X POST localhost:8080/internal/orders/3/payment -d '{"payment_result":"success"}'

# Tunggu 3 retry (lihat logs)
docker compose logs -f notification-service

# Verifikasi events masuk DLQ
curl localhost:8083/api/v1/admin/dlq/orders
# Expected: 2 messages di DLQ

# Fix bug, restart service
# Replay DLQ
curl -X POST localhost:8083/api/v1/admin/dlq/orders/replay
```

### Scenario 3: Idempotency Test

```bash
# Kirim event yang sama 2x (simulasi Kafka redelivery)
EVENT=$(docker exec kafka kafka-console-consumer \
  --topic orders --from-beginning --max-messages 1 \
  --bootstrap-server localhost:29092)

# Produce ulang event yang sama
echo $EVENT | docker exec -i kafka kafka-console-producer \
  --topic orders --bootstrap-server localhost:29092

# Verifikasi analytics TIDAK double-count
curl localhost:8084/analytics/daily
# total_orders harus sama (tidak bertambah 2x)
```

---

## 🏆 Kriteria Penilaian

### Minimum (Wajib)
- [ ] Kafka berjalan di docker-compose
- [ ] Order Service publish events ke Kafka menggunakan Outbox Pattern
- [ ] Notification Service consume dan log events dengan benar
- [ ] Consumer group berfungsi (multiple consumers berbagi load)
- [ ] Graceful shutdown consumer

### Good
- [ ] Analytics Service dengan endpoint statistik
- [ ] Dead Letter Queue untuk failed messages
- [ ] Idempotency di Notification Service
- [ ] EventEnvelope dengan versioning

### Excellent
- [ ] DLQ replay endpoint berfungsi
- [ ] Consumer contract test
- [ ] Kafka UI tersedia di docker-compose
- [ ] End-to-end test script yang cover semua scenarios

---

## 📖 Panduan Pengerjaan

### Hari 1–2: Kafka Setup + Events
- Tambahkan Kafka ke docker-compose.yml
- Definisikan event structs (EventEnvelope, OrderEvents)
- Buat kafka Producer dan Consumer packages
- Test dengan kafka-console-producer/consumer

### Hari 3–4: Outbox Pattern
- Buat tabel order_outbox
- Implementasikan OutboxRepository
- Implementasikan OutboxWorker
- Integrasikan di ConfirmOrder use case
- Test: konfirmasi order, verifikasi event di Kafka

### Hari 5–6: Notification Service
- Buat service baru
- Implementasikan consumer dengan event router
- Tambahkan idempotency dengan Redis
- Tambahkan DLQ handling
- Test happy path + DLQ

### Hari 7: Analytics Service
- Buat service dengan consumer
- Buat tabel stats
- Expose REST endpoints
- Test analytics update

### Hari 8–10: Polish + Testing
- Integrasi penuh di docker-compose
- End-to-end test script
- DLQ replay endpoint
- README update

---

*Setelah fase ini, sistem kamu sudah production-grade event-driven microservices!*

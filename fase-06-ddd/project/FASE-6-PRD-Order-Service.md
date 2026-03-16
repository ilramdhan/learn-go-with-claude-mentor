# 📋 PRD — Order Service (Full DDD Pattern)

> **Fase:** 6 — Domain-Driven Design  
> **Tipe Project:** DDD Microservice dengan Event-Driven  
> **Estimasi Pengerjaan:** 10–14 hari  
> **Konsep yang Diuji:** Aggregate, Value Object, Domain Events, Application Services, CQRS, Bounded Context

---

## 🎯 Tujuan Project

Membangun **Order Service** lengkap menggunakan full DDD pattern. Service ini adalah core domain dari e-commerce system — paling kompleks dan paling kaya business rules.

---

## 📌 Domain Model

### Ubiquitous Language (Order Context)

| Term | Definisi di Order Context |
|------|---------------------------|
| **Order** | Kumpulan item yang dipesan customer dalam satu transaksi |
| **Draft Order** | Order yang belum disubmit, bisa tambah/hapus item |
| **Submit** | Tindakan customer mengkonfirmasi order dan menunggu pembayaran |
| **Confirm** | Order terkonfirmasi setelah pembayaran berhasil |
| **Dispatch** | Order dikirim ke kurir / warehouse |
| **Deliver** | Order diterima oleh customer |
| **Cancel** | Order dibatalkan (dengan alasan) |
| **OrderItem** | Satu baris dalam order (produk + qty + harga snapshot) |
| **PriceSnapshot** | Harga produk pada saat order dibuat (tidak berubah meski harga naik) |

### Order State Machine

```
DRAFT → PENDING → CONFIRMED → DISPATCHED → DELIVERED
  ↓         ↓          ↓             ↓
CANCELLED CANCELLED CANCELLED   (tidak bisa cancel)
```

---

## ✅ Business Rules

### Invariants (harus selalu benar)
1. Order dalam status DRAFT boleh 0 item, status lain minimal 1 item
2. Total amount selalu = sum(quantity × unitPrice) untuk semua items
3. Stock dikurangi hanya saat status berubah ke CONFIRMED
4. Stock dikembalikan hanya saat CANCEL dari status CONFIRMED atau DISPATCHED

### Business Rules per Transisi
| Dari | Ke | Pre-conditions | Side Effects |
|------|----|---------------|-------------|
| NEW | DRAFT | customerID valid, address valid | OrderCreated event |
| DRAFT | PENDING | items > 0, stock tersedia | OrderSubmitted event |
| PENDING | CONFIRMED | payment valid | StockReduced, OrderConfirmed event |
| CONFIRMED | DISPATCHED | trackingNumber ada | OrderDispatched event |
| DISPATCHED | DELIVERED | - | OrderDelivered event |
| DRAFT/PENDING | CANCELLED | reason wajib | OrderCancelled event (no stock) |
| CONFIRMED/DISPATCHED | CANCELLED | reason wajib | OrderCancelled event + StockRestored event |

---

## 📡 API Specification

### HTTP REST (via Gin)
```
POST   /api/v1/orders                     ← Create draft order
GET    /api/v1/orders                     ← List my orders (customer)
GET    /api/v1/orders/:id                 ← Get order detail
PUT    /api/v1/orders/:id/items           ← Add/update item
DELETE /api/v1/orders/:id/items/:productId ← Remove item
POST   /api/v1/orders/:id/submit         ← Submit order
POST   /api/v1/orders/:id/cancel         ← Cancel order
PUT    /api/v1/orders/:id/shipping-address ← Update shipping address

# Internal (dipanggil oleh Payment Service via event)
POST   /internal/orders/:id/confirm      ← Confirm after payment

# Admin
GET    /api/v1/admin/orders              ← List all orders
POST   /api/v1/admin/orders/:id/dispatch ← Mark as dispatched
POST   /api/v1/admin/orders/:id/deliver  ← Mark as delivered
```

---

## 📁 Struktur Project (DDD)

```
order-service/
├── internal/
│   ├── domain/
│   │   └── order/
│   │       ├── aggregate/
│   │       │   └── order.go             ← Order aggregate root (THE core)
│   │       ├── entity/
│   │       │   ├── order_id.go
│   │       │   └── order_item_id.go
│   │       ├── valueobject/
│   │       │   ├── money.go
│   │       │   ├── address.go
│   │       │   └── order_status.go
│   │       ├── event/
│   │       │   ├── domain_event.go      ← Interface
│   │       │   ├── order_created.go
│   │       │   ├── order_submitted.go
│   │       │   ├── order_confirmed.go
│   │       │   ├── order_cancelled.go
│   │       │   ├── order_dispatched.go
│   │       │   └── order_delivered.go
│   │       └── service/
│   │           ├── stock_validation.go  ← Domain service interface
│   │           └── pricing.go           ← Domain service interface
│   │
│   ├── domain/
│   │   └── shared/
│   │       └── apperror/
│   │           └── errors.go
│   │
│   ├── repository/                      ← Interfaces (Write + Read side)
│   │   ├── order_command_repository.go
│   │   └── order_query_repository.go
│   │
│   ├── application/
│   │   └── usecase/
│   │       ├── create_order.go
│   │       ├── add_item.go
│   │       ├── remove_item.go
│   │       ├── submit_order.go
│   │       ├── confirm_order.go
│   │       ├── cancel_order.go
│   │       ├── dispatch_order.go
│   │       └── deliver_order.go
│   │
│   ├── delivery/
│   │   └── http/
│   │       ├── handler/
│   │       ├── middleware/
│   │       ├── request/
│   │       ├── response/
│   │       └── router.go
│   │
│   └── infrastructure/
│       ├── persistence/postgres/
│       │   ├── model/                   ← GORM models (berbeda dari domain!)
│       │   │   └── order_model.go
│       │   ├── mapper/                  ← Konversi domain ↔ model
│       │   │   └── order_mapper.go
│       │   ├── order_command_repo.go
│       │   └── order_query_repo.go
│       └── event/
│           ├── inmemory_publisher.go    ← Untuk testing
│           └── kafka_publisher.go       ← Untuk production (Fase 8)
```

---

## 🗂️ Database Schema

```sql
-- Write model (normalized, untuk GORM)
CREATE TABLE orders (
    id              BIGSERIAL PRIMARY KEY,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at      TIMESTAMPTZ,
    customer_id     BIGINT NOT NULL,
    status          VARCHAR(30) NOT NULL DEFAULT 'DRAFT',
    notes           TEXT,
    shipping_street   VARCHAR(255),
    shipping_city     VARCHAR(100),
    shipping_province VARCHAR(100),
    shipping_postal   VARCHAR(20),
    shipping_country  VARCHAR(100),
    total_amount    DECIMAL(15,2) NOT NULL DEFAULT 0,
    currency        VARCHAR(3) NOT NULL DEFAULT 'IDR',
    confirmed_at    TIMESTAMPTZ,
    cancelled_at    TIMESTAMPTZ,
    cancel_reason   TEXT,
    tracking_number VARCHAR(100)
);

CREATE TABLE order_items (
    id           BIGSERIAL PRIMARY KEY,
    order_id     BIGINT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id   BIGINT NOT NULL,
    product_name VARCHAR(255) NOT NULL,
    quantity     INT NOT NULL CHECK (quantity > 0),
    unit_price   DECIMAL(15,2) NOT NULL,
    currency     VARCHAR(3) NOT NULL DEFAULT 'IDR'
);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);

-- Outbox pattern untuk domain events (Fase 8)
CREATE TABLE order_outbox (
    id           BIGSERIAL PRIMARY KEY,
    event_name   VARCHAR(100) NOT NULL,
    aggregate_id VARCHAR(100) NOT NULL,
    payload      JSONB NOT NULL,
    published    BOOLEAN NOT NULL DEFAULT FALSE,
    created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    published_at TIMESTAMPTZ
);
CREATE INDEX idx_order_outbox_unpublished ON order_outbox(published, created_at) WHERE NOT published;

-- Read model (denormalized, untuk query)
CREATE MATERIALIZED VIEW order_list_view AS
SELECT
    o.id,
    o.customer_id,
    o.status,
    o.total_amount,
    o.currency,
    o.created_at,
    o.confirmed_at,
    COUNT(oi.id) as item_count
FROM orders o
LEFT JOIN order_items oi ON oi.order_id = o.id
WHERE o.deleted_at IS NULL
GROUP BY o.id;

CREATE INDEX ON order_list_view(customer_id, created_at DESC);
```

---

## 🧪 Test Requirements

### Unit Tests (Aggregate — Tanpa Database!)

```go
// Contoh test yang HARUS ada:

func TestOrder_AddItem_Success(t *testing.T) { ... }
func TestOrder_AddItem_NotDraft(t *testing.T) { ... }  // invariant test
func TestOrder_AddItem_ZeroQuantity(t *testing.T) { ... }
func TestOrder_Submit_EmptyOrder(t *testing.T) { ... }
func TestOrder_Submit_Success(t *testing.T) { ... }
func TestOrder_Submit_RaisesDomainEvent(t *testing.T) { ... }
func TestOrder_Confirm_NotPending(t *testing.T) { ... }
func TestOrder_Cancel_Delivered_ShouldFail(t *testing.T) { ... }
func TestOrder_Cancel_Confirmed_ShouldRestoreStock(t *testing.T) { ... }
func TestOrder_TotalAmount_Calculated_Correctly(t *testing.T) { ... }
```

### Integration Tests
- Create order → Add items → Submit → Confirm flow
- Cancel confirmed order → verify stock restore event emitted
- CQRS: write via command repo, read via query repo

---

## 🏆 Kriteria Penilaian

### Minimum
- [ ] Order aggregate dengan semua state transitions
- [ ] Value Objects: Money, Address
- [ ] Domain Events untuk setiap transisi
- [ ] Unit tests untuk aggregate (100% coverage target)
- [ ] Create order + submit + cancel use case

### Good  
- [ ] Full CQRS (command + query repository terpisah)
- [ ] Confirm + dispatch + deliver use case
- [ ] HTTP handler lengkap
- [ ] Integration tests

### Excellent
- [ ] Outbox pattern untuk domain events
- [ ] In-memory event publisher untuk testing
- [ ] Read model (materialized view atau separate read table)
- [ ] Event sourcing sederhana (simpan semua events, reconstruct dari events)

---

## ✅ DDD Checklist

Sebelum submit, verifikasi:
```
Aggregate:
[ ] Semua field private, hanya bisa diubah via method
[ ] Setiap method menjaga invariants
[ ] State transition hanya bisa ke state yang valid
[ ] Domain events di-raise untuk setiap transition yang signifikan
[ ] Ada factory method (NewOrder) dan reconstitution method terpisah

Value Objects:
[ ] Money tidak pernah negatif
[ ] Money operations return Money baru (immutable)
[ ] Address tervalidasi saat dibuat

Domain Events:
[ ] Setiap event punya nama yang jelas (past tense: "order.confirmed")
[ ] Event menyimpan snapshot data yang relevan
[ ] Event tidak menyimpan reference ke aggregate (hanya data)

Application Services:
[ ] Tidak ada business logic di use case
[ ] Use case hanya orkestasi: load → operasi → save → publish event
[ ] Semua dependency di-inject
```

---

*Ini adalah fase yang paling menantang secara konseptual. Take your time!*

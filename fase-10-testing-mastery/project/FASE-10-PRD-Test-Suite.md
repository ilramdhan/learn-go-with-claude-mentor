# 📋 PRD — Comprehensive Test Suite

> **Fase:** 10 — Testing Mastery
> **Tipe Project:** Membangun test suite lengkap untuk E-Commerce System
> **Estimasi Pengerjaan:** 10–14 hari
> **Konsep yang Diuji:** Unit test, Integration test, Contract test, Load test, Fuzz test, CI/CD

---

## 🎯 Tujuan Project

Membangun **Comprehensive Test Suite** untuk E-Commerce System dari Fase 7–9. Setelah project ini selesai, sistem memiliki:
- Keyakinan tinggi bahwa kode berfungsi seperti yang diharapkan
- Safety net untuk refactoring (ubah kode tanpa takut break sesuatu)
- Dokumentasi hidup (test sebagai spec)
- CI pipeline yang otomatis verifikasi kualitas setiap commit

---

## 📋 Scope Testing

### F-01: Unit Tests — Domain Layer (target: 90%+ coverage)

**Auth Service:**
- `User` entity: `NewUser`, `ChangePassword`, `Deactivate`, `UpdateProfile`
- `Email` value object: validasi format, normalisasi lowercase, domain extract
- `Password` value object: validasi complexity (min 8 chars, uppercase, lowercase, digit)
- `RegisterUseCase`: success, duplicate email, weak password
- `LoginUseCase`: success, wrong password, inactive user, user not found
- `RefreshTokenUseCase`: success, expired token, invalid token

**Order Service:**
- `Order` aggregate: `NewOrder`, `AddItem`, `RemoveItem`, `ChangeItemQuantity`, `Submit`, `Confirm`, `Cancel`
- `Money` value object: `Add`, `Subtract`, `Multiply`, `IsZero`, currency mismatch
- `Address` value object: field validation, formatting
- `CreateOrderUseCase`, `SubmitOrderUseCase`, `ConfirmOrderUseCase`, `CancelOrderUseCase`

### F-02: Integration Tests — Repository Layer (target: 70%+ coverage)

Gunakan **testcontainers** untuk semua integration tests:

- `UserRepository`: Save, FindByID, FindByEmail, ExistsByEmail, Update, Delete (soft)
- `OrderRepository`: Save (insert + update), FindByID, FindByIDForUpdate, GetOrderList with filters
- `OutboxRepository`: Save (dalam transaksi), GetUnpublished, MarkPublished, IncrementRetry
- **Transaction test**: verifikasi Order + Outbox save dalam satu transaksi (rollback jika salah satu gagal)

### F-03: HTTP Handler Tests (target: 80%+ coverage)

Test semua endpoints menggunakan `httptest`:

**Auth Handler:**
- `POST /auth/register`: success (201), validation error (400), duplicate email (409)
- `POST /auth/login`: success (200 + tokens), wrong credentials (401), inactive (403)
- `GET /auth/me`: success (200), no token (401), expired token (401)
- `POST /auth/refresh`: success, expired refresh token
- `POST /auth/logout`: success

**Order Handler:**
- `POST /orders`: success (201), validation error (400), unauthenticated (401)
- `GET /orders`: success with pagination, empty result
- `GET /orders/:id`: success, not found (404), unauthorized (403 — access other's order)
- `POST /orders/:id/cancel`: success, already cancelled (400), not owner (403)

### F-04: gRPC Handler Tests

Test semua gRPC endpoints menggunakan `bufconn`:

**Product Service:**
- `GetProduct`: success, not found (NOT_FOUND), invalid ID (INVALID_ARGUMENT)
- `CheckStock`: available, insufficient stock
- `UpdateStock`: success, insufficient stock, product not found
- `CreateProduct`: success, duplicate SKU (ALREADY_EXISTS), validation error

### F-05: Contract Tests

- **gRPC contract**: Order Service ↔ Product Service — verifikasi response format
- **Event contract**: Order Service publish events yang sesuai expected format Notification Service
- Verifikasi backward compatibility: event V1 masih bisa di-parse oleh consumer yang expect V1 fields

### F-06: Load Tests (k6)

Script untuk 3 scenarios:

**Scenario 1: Auth Service Login**
- 50 concurrent users, 2 menit
- Threshold: p95 < 100ms, error rate < 0.5%

**Scenario 2: Product Browse**
- 100 concurrent users (read heavy), 3 menit
- Threshold: p95 < 200ms, error rate < 0.1%

**Scenario 3: Checkout Flow (E2E)**
- 20 concurrent users, 2 menit
- Flow: login → list products → create order → submit order
- Threshold: p95 < 500ms, error rate < 1%

### F-07: Fuzz Tests

- `FuzzNewEmail`: tidak panic dengan input apapun
- `FuzzNewMoney`: tidak panic, amount selalu >= 0 jika success
- `FuzzParseOrderID`: round-trip property (parse → format → parse = sama)
- `FuzzValidatePassword`: tidak panic, consistent result untuk input yang sama

---

## 📁 Struktur File

```
ecommerce-microservices/
├── services/
│   ├── auth-service/
│   │   ├── internal/
│   │   │   ├── domain/
│   │   │   │   └── entity/
│   │   │   │       └── user_test.go
│   │   │   │   └── valueobject/
│   │   │   │       ├── email_test.go
│   │   │   │       ├── email_fuzz_test.go
│   │   │   │       └── password_test.go
│   │   │   ├── usecase/
│   │   │   │   └── auth/
│   │   │   │       ├── register_test.go
│   │   │   │       └── login_test.go
│   │   │   └── delivery/http/handler/
│   │   │       └── auth_handler_test.go
│   │   └── test/
│   │       └── integration/
│   │           └── user_repository_test.go
│   │
│   └── order-service/
│       ├── internal/
│       │   ├── domain/
│       │   │   ├── aggregate/
│       │   │   │   └── order_test.go
│       │   │   └── valueobject/
│       │   │       └── money_test.go
│       │   ├── usecase/
│       │   │   └── order/
│       │   │       ├── create_order_test.go
│       │   │       └── confirm_order_test.go
│       │   └── delivery/
│       │       ├── http/handler/
│       │       │   └── order_handler_test.go
│       │       └── grpc/handler/
│       │           └── product_grpc_test.go
│       └── test/
│           ├── integration/
│           │   ├── setup_test.go
│           │   └── order_repository_test.go
│           └── contract/
│               └── product_service_contract_test.go
│
├── test/
│   ├── fake/
│   │   ├── user_repository.go
│   │   ├── order_repository.go
│   │   └── event_publisher.go (SpyEventPublisher)
│   └── contract/
│       ├── product_service_contract.go
│       └── order_events_contract.go
│
├── load-tests/
│   ├── auth-login.js
│   ├── product-browse.js
│   └── checkout-flow.js
│
└── .github/
    └── workflows/
        └── test.yml
```

---

## 🧪 Test Setup yang Dibutuhkan

### docker-compose.test.yml

```yaml
version: '3.8'
services:
  test-postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: testuser
      POSTGRES_PASSWORD: testpass
      POSTGRES_DB: testdb
    ports:
      - "5433:5432"  # port berbeda agar tidak conflict dengan dev DB

  test-redis:
    image: redis:7-alpine
    ports:
      - "6380:6379"
```

### Environment Variables untuk Test

```bash
# .env.test
TEST_DATABASE_URL=host=localhost port=5433 user=testuser password=testpass dbname=testdb sslmode=disable
TEST_REDIS_URL=redis://localhost:6380
TEST_JWT_SECRET=test-secret-key-minimum-32-characters-long
```

---

## 🏆 Kriteria Penilaian

### Minimum (Wajib)
- [ ] Unit tests domain layer: 90%+ coverage
- [ ] Unit tests use case layer: 85%+ coverage (dengan mocks)
- [ ] Integration tests repository layer berjalan dengan testcontainers
- [ ] HTTP handler tests lengkap (semua endpoint, semua status codes)
- [ ] GitHub Actions CI pipeline green
- [ ] `make test` berhasil dari scratch di mesin baru

### Good
- [ ] gRPC handler tests dengan bufconn
- [ ] Contract tests (gRPC + event schema)
- [ ] k6 load tests dengan thresholds terpenuhi
- [ ] Fuzz tests untuk value objects

### Excellent
- [ ] Mutation testing score > 75% untuk domain layer
- [ ] Golden file tests untuk complex responses
- [ ] Pre-commit hooks untuk test gates
- [ ] Test report dengan coverage badge di README
- [ ] Comprehensive Makefile dengan semua test commands

---

## 📖 Panduan Pengerjaan

### Minggu 1: Foundation

**Hari 1–2:** Setup test infrastructure
- `test/fake/` package: InMemoryUserRepository, InMemoryOrderRepository, SpyEventPublisher
- `test/integration/setup_test.go`: testcontainers helper
- Makefile test commands

**Hari 3–4:** Domain layer unit tests
- User entity + value objects (Email, Password)
- Order aggregate + value objects (Money, Address)
- Target: 90%+ coverage domain layer

**Hari 5:** Use case unit tests
- RegisterUseCase, LoginUseCase (dengan mocks)
- CreateOrderUseCase, ConfirmOrderUseCase (dengan mocks + fakes)

### Minggu 2: Integration & Handler

**Hari 6–7:** Integration tests
- UserRepository + OrderRepository dengan testcontainers
- Transaction tests (Outbox pattern)

**Hari 8:** HTTP handler tests
- Auth endpoints (semua status codes)
- Order endpoints (semua status codes)

**Hari 9:** gRPC + Contract tests
- ProductService gRPC dengan bufconn
- Contract verification functions

### Minggu 3: Advanced Testing

**Hari 10:** Load tests
- k6 scripts untuk 3 scenarios
- Verify thresholds terpenuhi

**Hari 11:** Fuzz + Mutation tests
- Fuzz tests untuk value objects
- Mutation testing report

**Hari 12–14:** CI/CD + Polish
- GitHub Actions workflow
- Pre-commit hooks
- Coverage badges
- Documentation

---

## 📊 Coverage Targets Summary

| Layer | Target | Tool |
|-------|--------|------|
| Domain (entity, VO, aggregate) | 90%+ | unit test |
| Use Case | 85%+ | unit test + mocks |
| Repository | 70%+ | integration test |
| HTTP Handler | 80%+ | httptest |
| gRPC Handler | 75%+ | bufconn |
| Overall | 75%+ | merged coverage |

---

*Test suite yang baik adalah investasi — sekali tulis, manfaatnya seumur project!*

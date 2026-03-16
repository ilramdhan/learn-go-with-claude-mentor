# 📋 PRD — User Auth Service (Clean Architecture)

> **Fase:** 4 — Clean Architecture  
> **Tipe Project:** Production-Ready Auth Microservice  
> **Estimasi Pengerjaan:** 8–12 hari  
> **Konsep yang Diuji:** Domain Layer, Use Case, Repository Interface, DI, Unit Test per Layer, Graceful Shutdown

---

## 🎯 Tujuan Project

Membangun **User Auth Service** yang production-ready menggunakan Clean Architecture. Ini bukan sekadar REST API biasa — strukturnya harus benar-benar mengikuti prinsip Clean Architecture sehingga mudah ditest, di-maintain, dan di-extend.

---

## 📌 Latar Belakang

Platform e-commerce membutuhkan service terpusat untuk autentikasi dan manajemen user. Service ini akan menjadi fondasi untuk semua microservice lainnya (di Fase 7). Setiap microservice akan berkomunikasi dengan Auth Service untuk memvalidasi token.

---

## 🔐 Fitur Utama

### Auth Features
| Feature | Endpoint | Method |
|---------|----------|--------|
| Register | `/api/v1/auth/register` | POST |
| Login | `/api/v1/auth/login` | POST |
| Refresh Token | `/api/v1/auth/refresh` | POST |
| Logout | `/api/v1/auth/logout` | POST |
| Get Profile | `/api/v1/auth/me` | GET |
| Change Password | `/api/v1/auth/change-password` | PUT |
| Forgot Password | `/api/v1/auth/forgot-password` | POST |
| Reset Password | `/api/v1/auth/reset-password` | POST |

### User Management Features
| Feature | Endpoint | Method | Role |
|---------|----------|--------|------|
| List Users | `/api/v1/users` | GET | admin |
| Get User Detail | `/api/v1/users/:id` | GET | admin |
| Update User Profile | `/api/v1/users/profile` | PUT | user |
| Deactivate User | `/api/v1/users/:id/deactivate` | PATCH | admin |
| Activate User | `/api/v1/users/:id/activate` | PATCH | admin |
| Change Role | `/api/v1/users/:id/role` | PUT | admin |
| Delete User | `/api/v1/users/:id` | DELETE | admin |

### Token Validation (untuk microservice lain)
| Feature | Endpoint | Method |
|---------|----------|--------|
| Validate Token | `/api/v1/tokens/validate` | POST |

---

## ✅ Business Rules (Domain Rules)

### User Rules
1. Email harus unik di seluruh sistem
2. Email harus format valid
3. Password minimal 8 karakter, mengandung huruf besar, kecil, dan angka
4. Nama minimal 2 karakter, maksimal 100 karakter
5. User baru otomatis mendapat role `user`
6. User baru otomatis aktif (`is_active = true`)
7. User yang tidak aktif tidak bisa login
8. User tidak bisa mengubah email-nya sendiri (harus via admin)

### Token Rules
1. Access token expire setelah 15 menit
2. Refresh token expire setelah 7 hari
3. Satu user bisa punya maksimal 5 active refresh token (5 device)
4. Logout hanya menghapus refresh token yang bersangkutan
5. Logout all (di semua device) menghapus semua refresh token user
6. Password reset token expire setelah 1 jam dan hanya bisa dipakai sekali

### Security Rules
1. Password reset hanya kirim email, tidak reveal apakah email ada di sistem
2. Login gagal tidak reveal apakah email terdaftar atau password yang salah
3. Brute force protection: lockout setelah 5 kali gagal login dalam 15 menit
4. Semua password di-hash dengan bcrypt cost 14

---

## 📁 Struktur Project yang Diharapkan

```
user-auth-service/
├── cmd/
│   └── api/
│       └── main.go
├── config/
│   ├── config.go
│   └── config.yaml
├── internal/
│   ├── domain/
│   │   ├── entity/
│   │   │   ├── user.go
│   │   │   └── token.go
│   │   ├── valueobject/
│   │   │   ├── email.go
│   │   │   └── password.go
│   │   └── apperror/
│   │       └── errors.go
│   ├── repository/
│   │   ├── user_repository.go     ← INTERFACE
│   │   └── token_repository.go    ← INTERFACE
│   ├── usecase/
│   │   ├── auth/
│   │   │   ├── register.go
│   │   │   ├── login.go
│   │   │   ├── logout.go
│   │   │   ├── refresh.go
│   │   │   ├── change_password.go
│   │   │   └── reset_password.go
│   │   └── user/
│   │       ├── get_profile.go
│   │       ├── update_profile.go
│   │       ├── list_users.go
│   │       └── manage_user.go
│   ├── delivery/
│   │   └── http/
│   │       ├── handler/
│   │       │   ├── auth_handler.go
│   │       │   ├── user_handler.go
│   │       │   └── token_handler.go
│   │       ├── middleware/
│   │       │   ├── auth.go
│   │       │   ├── logger.go
│   │       │   ├── cors.go
│   │       │   └── rate_limiter.go
│   │       ├── request/
│   │       ├── response/
│   │       └── router.go
│   └── infrastructure/
│       └── persistence/
│           └── postgres/
│               ├── db.go
│               ├── user_repository.go  ← IMPLEMENTATION
│               └── token_repository.go ← IMPLEMENTATION
├── pkg/
│   ├── jwt/
│   ├── crypto/
│   ├── validator/
│   └── logger/
├── test/
│   ├── unit/
│   └── integration/
├── docker-compose.yml
├── Makefile
└── go.mod
```

---

## 🗂️ Database Schema

```sql
-- users table
CREATE TABLE users (
    id          BIGSERIAL PRIMARY KEY,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at  TIMESTAMPTZ,
    name        VARCHAR(100) NOT NULL,
    email       VARCHAR(255) NOT NULL,
    password    VARCHAR(255) NOT NULL,
    role        VARCHAR(20) NOT NULL DEFAULT 'user',
    is_active   BOOLEAN NOT NULL DEFAULT TRUE,
    CONSTRAINT users_email_unique UNIQUE (email) WHERE deleted_at IS NULL
);

-- refresh_tokens table
CREATE TABLE refresh_tokens (
    id         BIGSERIAL PRIMARY KEY,
    user_id    BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token      VARCHAR(512) NOT NULL UNIQUE,
    expires_at TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_refresh_tokens_user_id ON refresh_tokens(user_id);
CREATE INDEX idx_refresh_tokens_token ON refresh_tokens(token);
CREATE INDEX idx_refresh_tokens_expires_at ON refresh_tokens(expires_at);

-- password_reset_tokens table
CREATE TABLE password_reset_tokens (
    id         BIGSERIAL PRIMARY KEY,
    user_id    BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token      VARCHAR(512) NOT NULL UNIQUE,
    expires_at TIMESTAMPTZ NOT NULL,
    used_at    TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- login_attempts table (untuk brute force protection)
CREATE TABLE login_attempts (
    id         BIGSERIAL PRIMARY KEY,
    email      VARCHAR(255) NOT NULL,
    ip_address VARCHAR(45) NOT NULL,
    success    BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_login_attempts_email_created ON login_attempts(email, created_at);
```

---

## 📋 Request / Response Specs Lengkap

### POST /auth/register
**Request:**
```json
{
  "name": "Alice Wonderland",
  "email": "alice@example.com",
  "password": "SecurePass123"
}
```
**Response 201:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": 1,
      "name": "Alice Wonderland",
      "email": "alice@example.com",
      "role": "user",
      "is_active": true,
      "created_at": "2025-01-15T10:30:00Z"
    },
    "access_token": "eyJ...",
    "refresh_token": "abc123...",
    "expires_in": 900
  }
}
```
**Error Cases:**
- 400: name terlalu pendek/panjang
- 400: email tidak valid
- 400: password terlalu lemah
- 409: email sudah terdaftar

### POST /auth/login
**Request:**
```json
{
  "email": "alice@example.com",
  "password": "SecurePass123"
}
```
**Response 200:** (sama format dengan register)
**Error Cases:**
- 400: email/password format tidak valid
- 401: email atau password salah (pesan generik!)
- 403: akun dinonaktifkan
- 429: terlalu banyak percobaan login

### PUT /auth/change-password
**Headers:** `Authorization: Bearer <access_token>`
**Request:**
```json
{
  "current_password": "OldPass123",
  "new_password": "NewSecurePass456",
  "confirm_password": "NewSecurePass456"
}
```

### POST /auth/forgot-password
**Request:**
```json
{ "email": "alice@example.com" }
```
**Response 200:** (selalu OK meski email tidak ada — security!)
```json
{
  "success": true,
  "message": "If your email is registered, you will receive a reset link shortly"
}
```

---

## 🧪 Test Requirements

### Unit Tests yang Wajib Ada

| Use Case | Test Cases |
|----------|-----------|
| RegisterUseCase | success, email_exists, invalid_email, weak_password, db_error |
| LoginUseCase | success, wrong_password, user_not_found, user_inactive, db_error |
| LogoutUseCase | success, token_not_found, db_error |
| RefreshTokenUseCase | success, token_expired, token_not_found |
| ChangePasswordUseCase | success, wrong_current_password, same_password |

### Integration Tests
- POST /auth/register → login → get profile flow
- POST /auth/login → refresh → logout flow
- Rate limiting flow

### Minimum Code Coverage
- Domain layer: 90%+
- Use Case layer: 85%+
- Handler layer: 70%+

---

## 🏆 Kriteria Penilaian

### Minimum (Wajib)
- [ ] Struktur folder sesuai Clean Architecture
- [ ] Domain layer tanpa dependency eksternal
- [ ] Use Case layer dengan DI (dependency injection)
- [ ] Repository interface di domain side
- [ ] Register, Login, Logout, Get Profile endpoint
- [ ] JWT access + refresh token
- [ ] Unit tests untuk semua use case

### Good
- [ ] Change password, forgot/reset password
- [ ] User management (admin endpoints)
- [ ] Brute force protection
- [ ] Graceful shutdown
- [ ] Health check endpoint

### Excellent
- [ ] Integration tests
- [ ] Code coverage > 80%
- [ ] Docker Compose lengkap
- [ ] Makefile dengan semua commands
- [ ] Token validation endpoint untuk microservice lain

---

## 🛠️ Infrastructure yang Dibutuhkan

```yaml
# docker-compose.yml
version: '3.8'
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: authuser
      POSTGRES_PASSWORD: authpass
      POSTGRES_DB: authdb
    ports:
      - "5432:5432"

  # Opsional untuk Fase excellent
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

---

## 📖 Panduan Pengerjaan

### Hari 1–2: Domain Layer
- Tulis semua entity (User, Token)
- Tulis semua value object (Email, Password)
- Tulis semua domain errors
- Tulis unit tests untuk domain
- **ATURAN: TIDAK BOLEH import package luar selain standard library**

### Hari 3: Repository & Infrastructure
- Definisikan semua repository interface
- Implementasikan postgres repository
- Setup GORM models dan migration
- Test koneksi database

### Hari 4–5: Use Cases
- RegisterUseCase + test
- LoginUseCase + test
- LogoutUseCase + test
- RefreshTokenUseCase + test

### Hari 6–7: Delivery Layer
- Request/Response DTO
- Auth handler
- Middleware (auth, logger, cors)
- Router setup

### Hari 8: Dependency Injection & Main
- Wire semua komponen di main.go
- Config dengan Viper
- Graceful shutdown
- Health check

### Hari 9–10: Additional Features + Polish
- Change password
- Forgot/reset password
- Brute force protection
- Integration tests

### Hari 11–12: Review & Dokumentasi
- Code review (apakah sudah clean architecture?)
- Checklist: apakah ada dependency yang salah arah?
- README lengkap
- Push ke GitHub

---

## ✅ Checklist Clean Architecture

Sebelum submit, verifikasi:
```
Domain Layer:
[ ] entity/ hanya import standard library
[ ] valueobject/ hanya import standard library
[ ] apperror/ hanya import standard library + net/http

Repository Layer:
[ ] Interface ada di internal/repository/ (bukan infrastructure)
[ ] Interface hanya return domain entity, bukan GORM model

Use Case Layer:
[ ] Tidak ada import gin, echo, atau HTTP framework apapun
[ ] Tidak ada import gorm atau database driver apapun
[ ] Hanya import dari domain/ dan repository/
[ ] Semua dependency diinjeksikan via constructor

Delivery Layer:
[ ] Handler tidak mengandung business logic
[ ] Handler hanya parse request → call use case → format response
[ ] Handler tidak langsung akses database/repository

Infrastructure Layer:
[ ] Implementasi repository ada di infrastructure/
[ ] GORM model boleh berbeda dari domain entity (tapi punya conversion method)
```

---

*Ini adalah fondasi untuk semua fase berikutnya. Pastikan strukturnya benar sebelum lanjut!*

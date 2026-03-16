# 📋 PRD — Blog Platform REST API

> **Fase:** 3 — Gin Framework & REST API  
> **Tipe Project:** Full REST API dengan Database  
> **Estimasi Pengerjaan:** 7–10 hari  
> **Konsep yang Diuji:** Gin routing, GORM, JWT Auth, Middleware, Validation, Repository Pattern, Testing

---

## 🎯 Tujuan Project

Membangun **Blog Platform REST API** yang production-ready dengan fitur autentikasi JWT, manajemen artikel/post, komentar, tag, dan dokumentasi API. Ini adalah project yang bisa langsung dimasukkan ke portfolio.

---

## 📌 Latar Belakang

Sebuah startup membutuhkan backend untuk platform blogging. Mereka ingin API yang clean, aman, dan terdokumentasi dengan baik. API ini akan dikonsumsi oleh frontend (React/Vue) dan mobile app.

---

## 👤 User Roles

| Role | Deskripsi |
|------|-----------|
| `guest` | Tidak login — bisa baca post published |
| `user` | Login — bisa buat post, komentar |
| `admin` | Semua akses + kelola user |

---

## ✅ Functional Requirements

### Auth (FR-01)
- `POST /auth/register` — daftar akun baru
- `POST /auth/login` — login, dapat JWT token
- `POST /auth/refresh` — refresh access token
- `POST /auth/logout` — invalidate token
- `GET /auth/me` — get profil yang sedang login

### Users (FR-02)
- `GET /users` — list semua user (admin only)
- `GET /users/:id` — detail user
- `PUT /users/:id` — update profil sendiri
- `DELETE /users/:id` — hapus user (admin only)
- `PUT /users/:id/role` — ubah role (admin only)

### Posts (FR-03)
- `GET /posts` — list post published (public)
- `GET /posts/:slug` — detail post (public, increment view count)
- `GET /posts/my` — list post milik user yg login
- `POST /posts` — buat post baru (auth required)
- `PUT /posts/:id` — update post (owner/admin)
- `DELETE /posts/:id` — hapus post (owner/admin)
- `PATCH /posts/:id/publish` — publish post (owner/admin)
- `PATCH /posts/:id/unpublish` — unpublish post (owner/admin)

### Comments (FR-04)
- `GET /posts/:id/comments` — list komentar sebuah post
- `POST /posts/:id/comments` — tambah komentar (auth required)
- `DELETE /comments/:id` — hapus komentar (owner/admin)

### Tags (FR-05)
- `GET /tags` — list semua tag
- `GET /tags/:slug/posts` — post dengan tag tertentu
- `POST /tags` — buat tag (admin only)
- `DELETE /tags/:id` — hapus tag (admin only)

### Search (FR-06)
- `GET /search?q=<keyword>` — search post by title/content

---

## 🚫 Non-Functional Requirements

- Response time < 200ms untuk semua endpoint (tanpa data besar)
- Semua endpoint return JSON
- Error response konsisten menggunakan `ErrorInfo` format
- Pagination di semua endpoint list
- JWT access token expire 15 menit, refresh token 7 hari
- Password di-hash dengan bcrypt
- Input validation di semua endpoint yang menerima body
- Rate limiting: 100 req/menit per IP

---

## 📋 API Specification

### Standard Response Format

```json
// Success
{
  "success": true,
  "data": { ... },
  "meta": {
    "page": 1,
    "per_page": 10,
    "total": 100,
    "total_pages": 10
  }
}

// Error
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "validation failed",
    "details": {
      "email": "invalid email format",
      "password": "must be at least 8 characters"
    }
  }
}
```

### Auth Endpoints Detail

#### POST /auth/register
Request:
```json
{
  "name": "Alice Wonderland",
  "email": "alice@example.com",
  "password": "SecurePass123"
}
```
Response 201:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": 1,
      "name": "Alice Wonderland",
      "email": "alice@example.com",
      "role": "user",
      "created_at": "2025-01-15T10:30:00Z"
    },
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 900
  }
}
```

#### POST /auth/login
Request: `{ "email": "...", "password": "..." }`
Response: sama dengan register

#### GET /auth/me
Header: `Authorization: Bearer <token>`
Response:
```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com",
    "role": "user",
    "post_count": 5,
    "created_at": "..."
  }
}
```

### Posts Endpoints Detail

#### GET /posts?page=1&per_page=10&status=published&tag=golang&search=tutorial
Response:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "title": "Getting Started with Go",
      "slug": "getting-started-with-go",
      "excerpt": "Learn Go from the basics...",
      "status": "published",
      "view_count": 1250,
      "author": {
        "id": 1,
        "name": "Alice"
      },
      "tags": [
        {"id": 1, "name": "Go", "slug": "go"}
      ],
      "comment_count": 15,
      "created_at": "2025-01-15T10:30:00Z",
      "published_at": "2025-01-15T11:00:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "per_page": 10,
    "total": 48,
    "total_pages": 5
  }
}
```

#### POST /posts
Header: `Authorization: Bearer <token>`
Request:
```json
{
  "title": "Getting Started with Go",
  "slug": "getting-started-with-go",
  "content": "Go is an open source programming language...",
  "excerpt": "Learn Go from the basics",
  "status": "draft",
  "tag_ids": [1, 2, 3]
}
```

---

## 📁 Struktur Project

```
blog-api/
├── cmd/
│   └── api/
│       └── main.go               ← Entry point
├── config/
│   ├── config.go                 ← Config struct + Viper
│   └── config.yaml               ← Config file
├── internal/
│   ├── handler/                  ← HTTP handlers
│   │   ├── auth_handler.go
│   │   ├── user_handler.go
│   │   ├── post_handler.go
│   │   ├── comment_handler.go
│   │   └── tag_handler.go
│   ├── middleware/               ← Gin middlewares
│   │   ├── auth.go
│   │   ├── rate_limiter.go
│   │   ├── cors.go
│   │   └── logger.go
│   ├── model/                    ← GORM models
│   │   ├── user.go
│   │   ├── post.go
│   │   ├── comment.go
│   │   └── tag.go
│   ├── repository/               ← Database operations
│   │   ├── user_repository.go
│   │   ├── post_repository.go
│   │   ├── comment_repository.go
│   │   └── tag_repository.go
│   ├── service/                  ← Business logic
│   │   ├── auth_service.go
│   │   ├── user_service.go
│   │   ├── post_service.go
│   │   └── tag_service.go
│   ├── request/                  ← Request DTOs
│   │   ├── auth_request.go
│   │   ├── post_request.go
│   │   └── user_request.go
│   └── response/                 ← Response helpers + DTOs
│       ├── response.go
│       └── dto/
│           ├── user_dto.go
│           └── post_dto.go
├── pkg/
│   ├── auth/                     ← JWT + password
│   │   ├── jwt.go
│   │   └── password.go
│   ├── database/                 ← DB connection + migration
│   │   └── db.go
│   └── validator/                ← Custom validators
│       └── validator.go
├── test/
│   ├── integration/
│   └── unit/
├── docker-compose.yml
├── Makefile
├── README.md
└── go.mod
```

---

## 🗂️ Database Schema

```sql
-- users
CREATE TABLE users (
    id          BIGSERIAL PRIMARY KEY,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at  TIMESTAMPTZ,
    name        VARCHAR(100) NOT NULL,
    email       VARCHAR(255) NOT NULL UNIQUE,
    password    VARCHAR(255) NOT NULL,
    role        VARCHAR(20) NOT NULL DEFAULT 'user',
    is_active   BOOLEAN NOT NULL DEFAULT TRUE
);

-- posts
CREATE TABLE posts (
    id           BIGSERIAL PRIMARY KEY,
    created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at   TIMESTAMPTZ,
    title        VARCHAR(255) NOT NULL,
    slug         VARCHAR(255) NOT NULL UNIQUE,
    content      TEXT,
    excerpt      VARCHAR(500),
    status       VARCHAR(20) NOT NULL DEFAULT 'draft',
    view_count   INT NOT NULL DEFAULT 0,
    published_at TIMESTAMPTZ,
    user_id      BIGINT NOT NULL REFERENCES users(id)
);

-- tags
CREATE TABLE tags (
    id   BIGSERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE,
    slug VARCHAR(50) NOT NULL UNIQUE
);

-- post_tags (many2many)
CREATE TABLE post_tags (
    post_id BIGINT REFERENCES posts(id) ON DELETE CASCADE,
    tag_id  BIGINT REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (post_id, tag_id)
);

-- comments
CREATE TABLE comments (
    id         BIGSERIAL PRIMARY KEY,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    content    TEXT NOT NULL,
    post_id    BIGINT NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    user_id    BIGINT NOT NULL REFERENCES users(id)
);

-- refresh_tokens
CREATE TABLE refresh_tokens (
    id         BIGSERIAL PRIMARY KEY,
    user_id    BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token      VARCHAR(500) NOT NULL UNIQUE,
    expires_at TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 🧪 Test Cases (HTTPie / curl)

### Setup environment
```bash
BASE_URL="http://localhost:8080"
TOKEN=""
```

### Auth flow
```bash
# Register
curl -X POST $BASE_URL/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice","email":"alice@test.com","password":"SecurePass123"}'

# Login
curl -X POST $BASE_URL/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@test.com","password":"SecurePass123"}'
# Copy token dari response

TOKEN="eyJhbGci..."

# Get profile
curl $BASE_URL/auth/me \
  -H "Authorization: Bearer $TOKEN"
```

### Posts flow
```bash
# Create post
curl -X POST $BASE_URL/posts \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "My First Post",
    "slug": "my-first-post",
    "content": "This is my first blog post content...",
    "excerpt": "A short description",
    "status": "draft",
    "tag_ids": [1]
  }'

# List my posts
curl "$BASE_URL/posts/my?page=1&per_page=10" \
  -H "Authorization: Bearer $TOKEN"

# Publish post
curl -X PATCH $BASE_URL/posts/1/publish \
  -H "Authorization: Bearer $TOKEN"

# List published posts (public)
curl "$BASE_URL/posts?page=1&per_page=10"

# Get post by slug (public)
curl "$BASE_URL/posts/my-first-post"

# Add comment
curl -X POST $BASE_URL/posts/1/comments \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"Great post!"}'
```

### Error cases
```bash
# Register dengan email duplikat
curl -X POST $BASE_URL/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Bob","email":"alice@test.com","password":"Pass123"}'
# Expected: 409 Conflict

# Access protected route tanpa token
curl $BASE_URL/posts/my
# Expected: 401 Unauthorized

# Update post orang lain
curl -X PUT $BASE_URL/posts/99 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Hacked!"}'
# Expected: 403 Forbidden atau 404 Not Found

# Invalid pagination
curl "$BASE_URL/posts?page=-1&per_page=1000"
# Expected: 400 Bad Request
```

---

## 🏆 Kriteria Penilaian

### Minimum
- [ ] Auth: register, login, get profile
- [ ] Posts: CRUD + publish/unpublish
- [ ] JWT middleware berjalan
- [ ] Validation di semua request body
- [ ] Consistent error response format
- [ ] PostgreSQL dengan GORM

### Good
- [ ] Semua endpoint (comments, tags, search)
- [ ] Pagination di semua list endpoint
- [ ] Repository pattern
- [ ] Refresh token
- [ ] Rate limiting middleware

### Excellent
- [ ] Unit tests untuk handler (dengan mock repository)
- [ ] Integration tests
- [ ] Swagger documentation
- [ ] Graceful shutdown
- [ ] Docker Compose setup

---

## 📦 Dependencies

```go
// go.mod
require (
    github.com/gin-gonic/gin v1.9.1
    gorm.io/gorm v1.25.0
    gorm.io/driver/postgres v1.5.0
    github.com/golang-jwt/jwt/v5 v5.0.0
    golang.org/x/crypto v0.17.0
    github.com/spf13/viper v1.18.0
    github.com/go-playground/validator/v10 v10.16.0
    // Test dependencies
    github.com/stretchr/testify v1.8.4
    github.com/stretchr/mock v1.6.0
)
```

---

## 🛠️ Makefile

```makefile
.PHONY: run build test docker-up docker-down migrate

run:
	go run ./cmd/api/main.go

build:
	go build -o bin/api ./cmd/api/main.go

test:
	go test -v -cover ./...

docker-up:
	docker compose up -d

docker-down:
	docker compose down

migrate:
	go run ./cmd/migrate/main.go

lint:
	golangci-lint run ./...
```

---

## 📖 Panduan Pengerjaan

### Hari 1: Setup & Foundation
- Init project, setup dependencies
- Database model + GORM setup
- Docker Compose PostgreSQL
- Config dengan Viper
- Auto migration

### Hari 2: Auth
- User model + repository
- Password hashing
- JWT service
- Register + Login endpoint

### Hari 3: Post CRUD
- Post model + repository
- Post handler (CRUD)
- Auth middleware integration
- Ownership check

### Hari 4: Relations & Features
- Tags (many2many)
- Comments
- Pagination
- Search

### Hari 5: Polish
- Error handling konsisten
- Validation lengkap
- Rate limiter
- Response DTO

### Hari 6–7: Testing
- Unit tests handler
- Integration tests
- Fix bugs dari test

### Hari 8–9: Optional
- Swagger docs
- Dockerfile
- README lengkap

---

## 📝 Deliverable

1. Source code di `blog-api/`
2. `README.md` dengan:
   - Cara setup & run (docker-compose)
   - API documentation (atau link ke Swagger)
   - Diagram arsitektur sederhana
3. Postman collection atau REST Client file
4. Push ke GitHub dengan commit history yang rapi

---

*Ini adalah project portfolio pertama yang layak ditunjukkan ke recruiter. Pastikan README-nya bagus dan code-nya clean!*

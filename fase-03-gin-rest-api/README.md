# 📘 Fase 3: Gin Framework & REST API

> **Level:** 🟡 Intermediate  
> **Durasi Estimasi:** 3–4 minggu  
> **Prasyarat:** ✅ Selesaikan Fase 1 & 2

---

## 🎯 Tujuan Fase Ini

- ✅ Membangun REST API production-ready dengan Gin
- ✅ Integrasi PostgreSQL dengan GORM
- ✅ Implementasi JWT Authentication
- ✅ Input validation, error response konsisten
- ✅ Unit test dengan httptest dan mock

---

## 🗂️ Isi Folder

```
fase-03-gin-rest-api/
├── README.md
├── materi/
│   └── FASE-3-Gin-REST-API.md       ← 13 modul materi
└── project/
    └── FASE-3-PRD-Blog-API.md       ← PRD project akhir
```

---

## 📚 Daftar Modul

| # | Modul | Konsep Utama |
|---|-------|-------------|
| 3.1 | Gin Basics | routing, handler, group, response helpers |
| 3.2 | Request Handling | path params, query, body binding, headers |
| 3.3 | Middleware | custom middleware, chain, CORS, rate limit |
| 3.4 | Database + GORM | PostgreSQL, CRUD, migration, transaction |
| 3.5 | Repository Pattern | interface abstraksi database |
| 3.6 | JWT Authentication | token generate, validate, refresh |
| 3.7 | Error Handling | consistent error response format |
| 3.8 | Validation | binding tags, custom validators |
| 3.9 | File Upload | multipart form, storage |
| 3.10 | API Versioning | URL versioning best practices |
| 3.11 | Configuration | Viper, env vars, config file |
| 3.12 | Swagger/OpenAPI | auto-generate API docs |
| 3.13 | Testing REST API | httptest, mock repository |

---

## 🛠️ Infrastructure yang Dibutuhkan

```bash
# PostgreSQL via Docker
docker run -d \
  --name postgres-dev \
  -e POSTGRES_USER=devuser \
  -e POSTGRES_PASSWORD=devpass \
  -e POSTGRES_DB=devdb \
  -p 5432:5432 \
  postgres:16-alpine
```

---

## 🎯 Project Akhir: Blog Platform REST API

File: `project/FASE-3-PRD-Blog-API.md`

REST API lengkap untuk platform blogging — cocok untuk **portfolio**!
Fitur: JWT auth, CRUD posts, comments, tags (many2many), search, pagination.

**Estimasi pengerjaan:** 7–10 hari

---

## ✅ Checklist Kelulusan

- [ ] Membuat REST API dengan semua HTTP methods
- [ ] JWT auth berjalan (login → get token → akses protected route)
- [ ] Error response konsisten di semua endpoint
- [ ] Validation berjalan di semua request body
- [ ] Unit test untuk minimal 2 handler
- [ ] **Menyelesaikan project Blog API dan push ke GitHub**

---

## ➡️ Selanjutnya

Setelah selesai Fase 3 → [Fase 4: Clean Architecture](../fase-04-clean-architecture/)

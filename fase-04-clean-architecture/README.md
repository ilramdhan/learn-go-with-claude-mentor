# 📘 Fase 4: Clean Architecture dengan Gin

> **Level:** 🟠 Intermediate  
> **Durasi Estimasi:** 4–5 minggu  
> **Prasyarat:** ✅ Selesaikan Fase 3 — Blog API

---

## 🎯 Tujuan Fase Ini

- ✅ Memahami prinsip Clean Architecture dan SOLID
- ✅ Bisa mengorganisir kode agar mudah ditest dan dimaintain
- ✅ Menerapkan Dependency Injection dengan benar
- ✅ Menulis unit test per layer secara terisolasi
- ✅ Implementasi Graceful Shutdown

---

## 🗂️ Isi Folder

```
fase-04-clean-architecture/
├── README.md
├── materi/
│   └── FASE-4-Clean-Architecture.md  ← 12 modul materi
└── project/
    └── FASE-4-PRD-User-Auth-Service.md
```

---

## 📚 Daftar Modul

| # | Modul | Konsep Utama |
|---|-------|-------------|
| 4.1 | Mengapa Clean Architecture? | Masalah spaghetti code, SOLID |
| 4.2 | Lapisan Clean Architecture | Entity, UseCase, Repository, Delivery |
| 4.3 | Project Structure | Struktur folder industri |
| 4.4 | Domain Layer | Entity, Value Object, Domain Error |
| 4.5 | Repository Layer | Interface di domain side |
| 4.6 | Use Case Layer | Business logic, orchestration |
| 4.7 | Delivery Layer | Thin handler, DTO, error conversion |
| 4.8 | Dependency Injection | Manual DI, wire-up di main.go |
| 4.9 | Unit Testing per Layer | Mock dengan testify |
| 4.10 | Integration Testing | httptest + real database |
| 4.11 | Graceful Shutdown | SIGINT/SIGTERM, timeout |
| 4.12 | Health Check & Readiness | /health, /ready endpoint |

---

## 💡 Konsep Paling Penting

**Dependency Rule**: Panah dependency hanya boleh mengarah ke dalam.
```
Delivery → UseCase → Repository (interface) ← Infrastructure
                ↑
            Domain (tidak bergantung pada siapapun!)
```

---

## 🎯 Project Akhir: User Auth Service

File: `project/FASE-4-PRD-User-Auth-Service.md`

Auth microservice production-ready dengan Clean Architecture.
Fitur: register, login, JWT, refresh token, brute force protection.

**Ini adalah fondasi untuk semua fase berikutnya!**

---

## ✅ Checklist Kelulusan

- [ ] Domain layer tidak import package eksternal apapun
- [ ] Use case layer tidak import Gin atau GORM
- [ ] Handler tipis — tidak ada business logic di handler
- [ ] Unit test use case berjalan tanpa database (menggunakan mock)
- [ ] Semua dependency diinjeksikan via constructor
- [ ] **Menyelesaikan project User Auth Service**

---

## ➡️ Selanjutnya

Setelah selesai Fase 4 → [Fase 5: gRPC + Protobuf](../fase-05-grpc-protobuf/)

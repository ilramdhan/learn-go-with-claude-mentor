# 📘 Fase 6: Domain-Driven Design (DDD)

> **Level:** 🔴 Advanced  
> **Durasi Estimasi:** 3–4 minggu  
> **Prasyarat:** ✅ Selesaikan Fase 4 & 5

---

## 🎯 Tujuan Fase Ini

- ✅ Memahami filosofi DDD dan mengapa diperlukan
- ✅ Membuat Aggregate yang menjaga business invariants
- ✅ Membuat Value Object yang immutable dan self-validating
- ✅ Menerapkan Domain Events untuk event-driven communication
- ✅ Memahami Bounded Context dan CQRS

---

## 🗂️ Isi Folder

```
fase-06-ddd/
├── README.md
├── materi/
│   └── FASE-6-DDD.md                ← 12 modul materi
└── project/
    └── FASE-6-PRD-Order-Service.md  ← PRD project akhir
```

---

## 📚 Daftar Modul

| # | Modul | Konsep Utama |
|---|-------|-------------|
| 6.1 | Apa itu DDD? | Filosofi, tactical vs strategic |
| 6.2 | Ubiquitous Language | Shared vocabulary bisnis & teknis |
| 6.3 | Entity & Value Object | Identitas, immutability |
| 6.4 | Aggregate & Root | Batasan konsistensi, invariants |
| 6.5 | Repository Pattern DDD | Persistensi aggregate |
| 6.6 | Domain Events | Kejadian penting di domain |
| 6.7 | Domain Services | Logic yang tidak milik entity |
| 6.8 | Application Services | Orchestration tanpa business logic |
| 6.9 | Bounded Context | Pemisahan domain yang jelas |
| 6.10 | CQRS Pattern | Pisah read dari write |
| 6.11 | Event Sourcing Intro | State sebagai sequence of events |
| 6.12 | Anti-Corruption Layer | Isolasi dari sistem lain |

---

## 💡 Perbedaan DDD dengan Clean Architecture

```
Clean Architecture = CARA MENGORGANISIR kode
DDD = CARA MEMODELKAN domain bisnis ke dalam kode

Keduanya SALING MELENGKAPI:
- Clean Architecture untuk struktur teknis
- DDD untuk model domain yang ekspresif
```

---

## 🎯 Project Akhir: Order Service (Full DDD)

File: `project/FASE-6-PRD-Order-Service.md`

Order service yang paling kompleks — state machine lengkap, domain events, CQRS.
Core domain dari e-commerce system yang akan dibangun di Fase 7.

---

## ✅ Checklist Kelulusan

- [ ] Order aggregate menjaga semua invariants (tidak bisa submit order kosong, dll)
- [ ] Value Object Money tidak pernah negatif, immutable
- [ ] Domain Events di-raise untuk setiap state transition
- [ ] Unit test aggregate berjalan **tanpa database**
- [ ] CQRS: command repo terpisah dari query repo
- [ ] **Menyelesaikan project Order Service**

---

## ➡️ Selanjutnya

Setelah selesai Fase 6 → [Fase 7: Microservices Architecture](../fase-07-microservices/)

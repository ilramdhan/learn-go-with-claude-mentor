# 📘 Fase 2: Go Intermediate

> **Level:** 🟡 Beginner-Mid  
> **Durasi Estimasi:** 3–4 minggu  
> **Prasyarat:** ✅ Selesaikan Fase 1 + Project CLI Todo Manager

---

## 🎯 Tujuan Fase Ini

Setelah menyelesaikan fase ini, kamu akan:
- ✅ Memahami Interface dan duck typing di Go
- ✅ Bisa menulis program concurrent dengan Goroutines dan Channels
- ✅ Memahami Context untuk timeout dan cancellation
- ✅ Bisa menulis unit test dan benchmark
- ✅ Paham pola-pola idiomatik Go (functional options, middleware chain)

---

## 🗂️ Isi Folder

```
fase-02-go-intermediate/
├── README.md
├── materi/
│   └── FASE-2-Go-Intermediate.md     ← 13 modul materi
└── project/
    └── FASE-2-PRD-File-Processor.md  ← PRD project akhir
```

---

## 📚 Daftar Modul

| # | Modul | Konsep Utama |
|---|-------|-------------|
| 2.1 | Interfaces | duck typing, composition, empty interface |
| 2.2 | Type Assertions & Switches | runtime type checking |
| 2.3 | Goroutines | go keyword, worker pool, closure pitfalls |
| 2.4 | Channels | unbuffered, buffered, directional, pipeline |
| 2.5 | Select Statement | multiplexing, timeout pattern |
| 2.6 | Sync Package | Mutex, RWMutex, WaitGroup, Once, Pool |
| 2.7 | Context | WithCancel, WithTimeout, WithValue, propagation |
| 2.8 | Generics | type parameters, constraints, Map/Filter/Reduce |
| 2.9 | Testing Dasar | table-driven tests, benchmark, coverage |
| 2.10 | File I/O & OS | baca/tulis file, bufio, env vars |
| 2.11 | Working with JSON | marshal, unmarshal, custom marshaling |
| 2.12 | Time & Date | format, parse, timer, ticker |
| 2.13 | Pola Idiomatik Go | functional options, middleware pattern |

---

## ⚠️ Modul Terpenting di Fase Ini

**Modul 2.3–2.7 (Concurrency)** adalah yang paling membedakan Go dari bahasa lain. Habiskan waktu ekstra di bagian ini!

```
Urutan belajar concurrency yang direkomendasikan:
Goroutines → Channels → Select → Sync → Context
```

---

## 🎯 Project Akhir: Concurrent File Processor

File: `project/FASE-2-PRD-File-Processor.md`

CLI tool yang memproses banyak file secara **parallel** menggunakan goroutines.
Fitur: word count, grep-like search, file stats — semua concurrent!

**Estimasi pengerjaan:** 4–6 hari

---

## ✅ Checklist Kelulusan

- [ ] Bisa menjelaskan perbedaan goroutine dan thread OS
- [ ] Bisa membuat pipeline dengan channels
- [ ] Tidak ada race condition di kode concurrent (test dengan `go test -race`)
- [ ] Bisa menggunakan context untuk cancel operasi yang timeout
- [ ] Bisa menulis table-driven tests
- [ ] **Menyelesaikan project File Processor**

---

## ➡️ Selanjutnya

Setelah selesai Fase 2 → [Fase 3: Gin Framework & REST API](../fase-03-gin-rest-api/)

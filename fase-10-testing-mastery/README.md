# 📘 Fase 10: Testing Mastery

> **Level:** 🔴 Advanced
> **Durasi Estimasi:** 2–3 minggu
> **Prasyarat:** ✅ Selesaikan Fase 1–4 (fundamental Go + Clean Architecture)

---

## 🎯 Tujuan Fase Ini

- ✅ Memahami test pyramid dan kapan pakai setiap level test
- ✅ Menulis unit test yang bermakna dengan testify (table-driven, sub-tests, parallel)
- ✅ Menguasai test doubles: Mock, Stub, Fake, Spy
- ✅ Integration testing dengan testcontainers (database nyata, bukan mock)
- ✅ HTTP handler testing dengan httptest + gRPC testing dengan bufconn
- ✅ Contract testing untuk menjamin kompatibilitas antar service
- ✅ Load testing dengan k6 + Go benchmark
- ✅ Fuzz testing untuk menemukan edge cases
- ✅ Mutation testing untuk verifikasi kualitas test
- ✅ CI/CD pipeline dengan GitHub Actions

---

## 🗂️ Isi Folder

```
fase-10-testing-mastery/
├── README.md
├── materi/
│   └── FASE-10-Testing-Mastery.md    ← 13 modul (69KB, 2183 baris)
└── project/
    └── FASE-10-PRD-Test-Suite.md
```

---

## 📚 Daftar Modul

| # | Modul | Konsep Utama |
|---|-------|-------------|
| 10.1 | Testing Philosophy | Test pyramid, ROI, testing strategy |
| 10.2 | Unit Testing Advanced | testify, table-driven, sub-tests, parallel |
| 10.3 | Testing dengan DI | Mock/Stub/Fake/Spy, mockery generation |
| 10.4 | Integration Testing | testcontainers, PostgreSQL nyata, transaction test |
| 10.5 | HTTP Handler Testing | httptest, recorder, golden files |
| 10.6 | gRPC Testing | bufconn, in-process server, streaming test |
| 10.7 | Contract Testing | Consumer-driven contracts, event schema |
| 10.8 | Load & Performance | k6, Go benchmark, benchstat |
| 10.9 | Fuzz Testing | Go 1.18+ fuzzing, corpus, property testing |
| 10.10 | Mutation Testing | go-mutesting, mutation score |
| 10.11 | Coverage Strategy | Coverage tools, meaningful targets |
| 10.12 | Best Practices | Anti-patterns, test naming, assertion quality |
| 10.13 | CI/CD Integration | GitHub Actions, Makefile, pre-commit hooks |

---

## 🛠️ Dependencies

```bash
# Testing libraries
go get github.com/stretchr/testify
go get github.com/vektra/mockery/v2
go get github.com/testcontainers/testcontainers-go
go get github.com/testcontainers/testcontainers-go/modules/postgres

# Load testing
brew install k6

# Mutation testing
go install github.com/zimmski/go-mutesting/cmd/go-mutesting@latest

# Better test output
go install gotest.tools/gotestsum@latest
```

---

## ⚡ Quick Start

```bash
# Unit tests (cepat, tanpa external deps)
make test-unit

# Integration tests (butuh Docker)
make test-integration

# Semua tests + coverage check
make test-all

# Load test
k6 run load-tests/auth-login.js

# Fuzz test
go test -fuzz=FuzzNewEmail -fuzztime=30s ./internal/domain/valueobject/

# Mutation test
go-mutesting ./internal/domain/...
```

---

## ✅ Checklist Kelulusan

- [ ] Domain layer coverage ≥ 90%
- [ ] Use case layer coverage ≥ 85% (dengan mocks)
- [ ] Integration tests repository berjalan dengan testcontainers
- [ ] HTTP handler tests cover semua status codes
- [ ] gRPC tests dengan bufconn berjalan
- [ ] k6 load test thresholds terpenuhi
- [ ] Fuzz tests tidak menemukan panic
- [ ] GitHub Actions CI pipeline green
- [ ] **Menyelesaikan project Comprehensive Test Suite**

---

## 🏆 Perbedaan Developer dengan dan tanpa Testing Mastery

```
Tanpa Testing Mastery:
  "Saya rasa ini sudah benar..." → deploy → production crash → 2 jam debug
  Refactor = takut → kode makin kotor setiap sprint

Dengan Testing Mastery:
  "Test pass, confident deploy" → production stable
  Refactor dengan berani → kode makin bersih setiap sprint
  Bug ditemukan di development, bukan production
```

---

## ➡️ Selanjutnya

Setelah selesai Fase 10 → [Fase 11: DevOps & Deployment](../fase-11-devops-deployment/) — Fase TERAKHIR!

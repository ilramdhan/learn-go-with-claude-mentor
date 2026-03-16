# 🐹 Learn Go with Claude Mentor

<div align="center">

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Go Version](https://img.shields.io/badge/Go-1.22+-00ADD8?logo=go)](https://go.dev)
[![Phases](https://img.shields.io/badge/Phases-11%20Lengkap-brightgreen)](./fase-01-go-fundamentals/)
[![Modules](https://img.shields.io/badge/Total%20Modul-143-blue)](./fase-01-go-fundamentals/)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-orange)](CONTRIBUTING.md)
[![Made with Claude](https://img.shields.io/badge/Made%20with-Claude%20AI-blueviolet?logo=anthropic)](https://claude.ai)

**Kurikulum belajar Go paling komprehensif — dari nol hingga Senior Go Developer**

*Dibuat bersama Claude AI sebagai mentor, untuk komunitas developer Indonesia dan dunia*

[🚀 Mulai Belajar](#-cara-memulai) • [🗺️ Roadmap](#️-roadmap-belajar) • [📋 Semua Fase](#-daftar-fase-lengkap) • [🤝 Kontribusi](CONTRIBUTING.md)

</div>

---

## 📖 Tentang Proyek Ini

Repositori ini berisi **kurikulum belajar Go yang lengkap dan terstruktur** — mulai dari syntax dasar hingga arsitektur microservices production-grade yang dipakai oleh perusahaan tech seperti Google, Netflix, Tokopedia, Gojek, dan Grab.

Setiap materi dilengkapi dengan:
- 📚 **Materi teori** — penjelasan mendalam dengan contoh kode yang bisa langsung dijalankan
- 🏋️ **Latihan per modul** — 3 latihan per modul untuk memperkuat pemahaman
- 🎯 **Project PRD nyata** — Product Requirements Document untuk project akhir setiap fase
- ✅ **Review & Checklist** — self-assessment sebelum lanjut ke fase berikutnya
- 📁 **Aset visual** — diagram, cheatsheet, dan referensi di folder `assets/`

### 🎯 Untuk Siapa?

| Level | Cocok Jika |
|-------|-----------|
| 🟢 Pemula | Punya pengalaman di Python/JS/PHP, ingin mulai Go dari nol |
| 🟡 Junior Dev | Sudah paham Go dasar, ingin level up ke arsitektur yang lebih baik |
| 🟠 Mid Dev | Ingin kuasai microservices, gRPC, DDD, dan sistem terdistribusi |
| 🔴 Senior Dev | Butuh referensi lengkap untuk onboarding tim atau review knowledge |

---

## 🗺️ Roadmap Belajar

```
FASE 1 ──► FASE 2 ──► FASE 3 ──► FASE 4 ──► FASE 5 ──► FASE 6
  Go          Go         Gin        Clean      gRPC +      DDD
 Dasar      Lanjut     REST API    Arch.     Protobuf    Pattern
2-3 mgg    3-4 mgg    3-4 mgg    4-5 mgg    3-4 mgg    3-4 mgg

         ↓ Semua fondasi sudah siap ↓

FASE 7 ──► FASE 8 ──► FASE 9 ──► FASE 10 ─► FASE 11
Micro-     Message   Observ-     Testing    DevOps &
services   Broker    ability     Mastery    Deploy
4-6 mgg   3-4 mgg   2-3 mgg    2-3 mgg    3-4 mgg

Total estimasi: 35–50 minggu (belajar konsisten ~10 jam/minggu)
```

---

## 📋 Daftar Fase Lengkap

| # | Fase | Topik Utama | Level | Durasi | Modul |
|---|------|-------------|-------|--------|-------|
| [01](./fase-01-go-fundamentals/) | **Go Fundamentals** | Variables, Functions, Slices, Maps, Structs, Error Handling, Packages | 🟢 Beginner | 2–3 mgg | 13 |
| [02](./fase-02-go-intermediate/) | **Go Intermediate** | Interfaces, Goroutines, Channels, Context, Generics, Testing | 🟡 Beginner+ | 3–4 mgg | 13 |
| [03](./fase-03-gin-rest-api/) | **Gin & REST API** | Gin framework, GORM, JWT Auth, Pagination, Validation, Swagger | 🟡 Intermediate | 3–4 mgg | 13 |
| [04](./fase-04-clean-architecture/) | **Clean Architecture** | SOLID, Layer separation, DI, Repository, Use Case, Graceful Shutdown | 🟠 Intermediate | 4–5 mgg | 12 |
| [05](./fase-05-grpc-protobuf/) | **gRPC + Protobuf** | Protocol Buffers, 4 RPC types, Interceptors, Streaming, Gateway | 🟠 Intermediate | 3–4 mgg | 13 |
| [06](./fase-06-ddd/) | **Domain-Driven Design** | Aggregate, Value Object, Domain Events, CQRS, Event Sourcing, ACL | 🔴 Advanced | 3–4 mgg | 12 |
| [07](./fase-07-microservices/) | **Microservices** | Docker, K8s, API Gateway, Circuit Breaker, Saga, Service Discovery | 🔴 Advanced | 4–6 mgg | 13 |
| [08](./fase-08-message-broker/) | **Message Broker** | Kafka, RabbitMQ, Consumer Groups, Outbox Pattern, DLQ, Schema | 🔴 Advanced | 3–4 mgg | 13 |
| [09](./fase-09-observability/) | **Observability** | Zap, Prometheus, Grafana, OpenTelemetry, Jaeger, SLO, Alerting | 🔴 Advanced | 2–3 mgg | 13 |
| [10](./fase-10-testing-mastery/) | **Testing Mastery** | Unit, Integration, Contract, Load (k6), Fuzz, Mutation, CI/CD | 🟠 Advanced | 2–3 mgg | 13 |
| [11](./fase-11-devops-deployment/) | **DevOps & Deploy** | GitHub Actions, Helm, ArgoCD, Blue-Green, Vault, Terraform, DR | 🔴 Senior | 3–4 mgg | 13 |

**Total: 11 Fase · 143 Modul · 143 Latihan · 11 Project PRD**

---

## 🚀 Cara Memulai

### Prerequisites

```bash
# 1. Install Go 1.22+
brew install go                    # macOS
sudo apt install golang-go         # Ubuntu/Debian
# atau download dari https://go.dev/dl/

# 2. Verify
go version                         # go version go1.22.x

# 3. Setup VS Code (recommended)
code --install-extension golang.go

# 4. Clone repo
git clone https://github.com/ilramdhan/learn-go-with-claude-mentor.git
cd learn-go-with-claude-mentor
```

### Mulai dari Fase 1

```bash
cd fase-01-go-fundamentals
cat README.md      # baca overview fase ini
cat materi/FASE-1-Go-Fundamentals.md   # mulai belajar!
cat project/FASE-1-PRD-CLI-Todo-Manager.md  # project akhir fase
```

### Struktur Setiap Fase

```
fase-XX-nama/
├── README.md          ← Overview, daftar modul, checklist kelulusan
├── materi/
│   └── FASE-X-Nama.md ← Materi lengkap dengan contoh kode
├── project/
│   └── FASE-X-PRD-*.md ← Project requirements yang harus dikerjakan
└── assets/
    ├── README.md       ← Penjelasan aset visual
    └── *.md / *.png    ← Diagram, cheatsheet, referensi
```

---

## 📚 Resources Tambahan

Folder [`resources/`](./resources/) berisi:

| File | Konten |
|------|--------|
| [`cheatsheets/go-quick-reference.md`](./resources/cheatsheets/go-quick-reference.md) | Syntax Go yang sering dipakai |
| [`tools/SETUP_GUIDE.md`](./resources/tools/SETUP_GUIDE.md) | Setup tools: VS Code, Docker, Kubernetes, dll |
| [`references/LEARNING_RESOURCES.md`](./resources/references/LEARNING_RESOURCES.md) | Buku, video, blog terbaik untuk belajar Go |
| [`CLAUDE_PROMPTS.md`](./resources/CLAUDE_PROMPTS.md) | Prompt untuk berdiskusi dengan Claude AI |

---

## 🏗️ Tech Stack yang Dipelajari

```
Bahasa:    Go 1.22+
Web:       Gin, net/http
Database:  PostgreSQL, Redis, GORM, golang-migrate
API:       REST, gRPC, Protocol Buffers
Messaging: Apache Kafka, RabbitMQ
Container: Docker, Docker Compose
Orchestration: Kubernetes, Helm, ArgoCD
Observability: Zap, Prometheus, Grafana, OpenTelemetry, Jaeger, Loki
Testing:   testify, testcontainers, k6, go-mutesting
CI/CD:     GitHub Actions, Terraform, Vault
```

---

## 📊 Statistik Kurikulum

```
📁 11 Fase
📦 143 Modul
🏋️ 143 Latihan (3 per modul)
🎯 11 Project PRD
📄 ~27,000 baris materi
💾 ~800KB konten teks
⏱️  35–50 minggu belajar
```

---

## 🤝 Cara Berkontribusi

Kami menyambut kontribusi dari komunitas! Lihat [CONTRIBUTING.md](CONTRIBUTING.md) untuk panduan lengkap.

Beberapa cara berkontribusi:
- 🐛 **Report bugs** — typo, kode yang salah, link rusak
- 💡 **Usul konten** — topik yang belum dibahas
- 🎨 **Tambah aset** — diagram, infografis untuk folder `assets/`
- 📝 **Perbaiki materi** — penjelasan yang lebih jelas, contoh tambahan
- 🌍 **Terjemahan** — bantu terjemahkan ke bahasa lain

---

## 📜 Lisensi

Proyek ini dilisensikan di bawah [MIT License](LICENSE) — bebas digunakan untuk keperluan pribadi maupun komersial.

---

## 🙏 Acknowledgments

- [The Go Team](https://go.dev) — atas bahasa pemrograman yang luar biasa
- [Anthropic](https://anthropic.com) — atas Claude AI yang membantu membuat kurikulum ini
- Komunitas developer Indonesia yang terus berkembang 🇮🇩

---

<div align="center">

**Selamat belajar Go! 🚀**

*"The only way to learn a new programming language is by writing programs in it." — Dennis Ritchie*

⭐ Jika kurikulum ini membantu, berikan star di GitHub!

</div>

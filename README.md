# 🐹 Learn Go with Claude Mentor

<div align="center">

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Go Version](https://img.shields.io/badge/Go-1.22+-00ADD8?logo=go)](https://go.dev)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Made with Claude](https://img.shields.io/badge/Made%20with-Claude%20AI-orange?logo=anthropic)](https://claude.ai)
[![GitHub Stars](https://img.shields.io/github/stars/ilramdhan/learn-go-with-claude-mentor?style=social)](https://github.com/ilramdhan/learn-go-with-claude-mentor)

**Kurikulum belajar Go paling komprehensif — dari nol hingga Senior Go Developer**  
*Dibuat bersama Claude AI sebagai mentor, untuk komunitas developer Indonesia dan dunia*

[🚀 Mulai Belajar](#-cara-menggunakan) • [🗺️ Roadmap](#️-roadmap-belajar) • [🤝 Kontribusi](CONTRIBUTING.md) • [💬 Diskusi](https://github.com/ilramdhan/learn-go-with-claude-mentor/discussions)

</div>

---

## 📖 Tentang Proyek Ini

Repositori ini berisi **kurikulum belajar Go yang lengkap** — mulai dari syntax dasar hingga arsitektur microservices production-grade yang dipakai oleh perusahaan tech seperti Google, Netflix, Tokopedia, Gojek, dan Grab.

Kurikulum ini dibuat secara kolaboratif antara **manusia dan AI (Claude by Anthropic)**, kemudian di-review dan disesuaikan untuk kebutuhan komunitas developer Indonesia. Setiap materi dilengkapi dengan:

- 📚 **Materi teori** dengan penjelasan mendalam dan contoh kode lengkap
- 🏋️ **Latihan per modul** untuk memperkuat pemahaman
- 🎯 **Project nyata** dengan PRD (Product Requirements Document) lengkap
- ✅ **Checklist review** untuk memastikan pemahaman sebelum lanjut

### 🎯 Siapa yang Cocok Menggunakan Ini?

| Level | Deskripsi |
|-------|-----------|
| 🟢 Pemula | Punya pengalaman di bahasa lain (Python/JS/PHP), ingin belajar Go |
| 🟡 Junior Dev | Sudah paham Go dasar, ingin level up ke arsitektur yang lebih baik |
| 🟠 Mid Dev | Ingin memahami microservices, gRPC, DDD, dan sistem terdistribusi |
| 🔴 Senior Dev | Ingin referensi lengkap dan terstruktur untuk onboarding tim |

---

## 🗺️ Roadmap Belajar

```
FONDASI                    FRAMEWORK              ARSITEKTUR
┌─────────────────┐        ┌──────────────┐       ┌──────────────────┐
│  FASE 1         │        │  FASE 3      │       │  FASE 4          │
│  Go Fundamentals│──────▶ │  Gin + REST  │──────▶│  Clean Arch      │
│  2-3 minggu     │        │  3-4 minggu  │       │  4-5 minggu      │
└─────────────────┘        └──────────────┘       └────────┬─────────┘
         │                                                  │
┌─────────────────┐                                ┌───────▼──────────┐
│  FASE 2         │                                │  FASE 5          │
│  Go Intermediate│                                │  gRPC + Protobuf │
│  3-4 minggu     │                                │  3-4 minggu      │
└─────────────────┘                                └────────┬─────────┘
                                                            │
                                                   ┌────────▼─────────┐
                                                   │  FASE 6          │
                                                   │  DDD Pattern     │
                                                   │  3-4 minggu      │
                                                   └────────┬─────────┘

PRODUCTION SYSTEMS
┌─────────────────┐   ┌──────────────────┐   ┌──────────────────┐
│  FASE 7         │   │  FASE 8          │   │  FASE 9          │
│  Microservices  │──▶│  Message Broker  │──▶│  Observability   │
│  4-6 minggu     │   │  3-4 minggu      │   │  2-3 minggu      │
└─────────────────┘   └──────────────────┘   └──────────────────┘

QUALITY & DEPLOYMENT
┌─────────────────┐   ┌──────────────────┐
│  FASE 10        │   │  FASE 11         │
│  Testing Mastery│──▶│  DevOps & Deploy │
│  2-3 minggu     │   │  2-3 minggu      │
└─────────────────┘   └──────────────────┘
```

---

## 📋 Daftar Fase

| # | Fase | Level | Durasi | Status |
|---|------|-------|--------|--------|
| [01](./fase-01-go-fundamentals/) | Go Fundamentals | 🟢 Beginner | 2–3 minggu | ✅ Lengkap |
| [02](./fase-02-go-intermediate/) | Go Intermediate | 🟡 Beginner-Mid | 3–4 minggu | ✅ Lengkap |
| [03](./fase-03-gin-rest-api/) | Gin Framework & REST API | 🟡 Intermediate | 3–4 minggu | ✅ Lengkap |
| [04](./fase-04-clean-architecture/) | Clean Architecture | 🟠 Intermediate | 4–5 minggu | ✅ Lengkap |
| [05](./fase-05-grpc-protobuf/) | gRPC + Protocol Buffers | 🟠 Intermediate | 3–4 minggu | ✅ Lengkap |
| [06](./fase-06-ddd/) | Domain-Driven Design (DDD) | 🔴 Advanced | 3–4 minggu | ✅ Lengkap |
| [07](./fase-07-microservices/) | Microservices Architecture | 🔴 Advanced | 4–6 minggu | 🔄 Coming Soon |
| [08](./fase-08-message-broker/) | Message Broker & Event-Driven | 🔴 Advanced | 3–4 minggu | 🔄 Coming Soon |
| [09](./fase-09-observability/) | Observability & Production | 🔴 Advanced | 2–3 minggu | 🔄 Coming Soon |
| [10](./fase-10-testing-mastery/) | Testing Mastery | 🟠 Intermediate | 2–3 minggu | 🔄 Coming Soon |
| [11](./fase-11-devops-deployment/) | DevOps & Deployment | 🔴 Advanced | 2–3 minggu | 🔄 Coming Soon |

**Total estimasi:** 35–50 minggu untuk menyelesaikan semua fase

---

## 🚀 Cara Menggunakan

### 1. Clone Repositori

```bash
git clone https://github.com/ilramdhan/learn-go-with-claude-mentor.git
cd learn-go-with-claude-mentor
```

### 2. Persiapkan Environment

Install tools yang dibutuhkan:
```bash
# Go 1.22+
# Download dari: https://go.dev/dl/

# VS Code + Extension Go (by Google)
# Download dari: https://code.visualstudio.com/

# Verifikasi instalasi
go version
```

### 3. Buka di Obsidian (Direkomendasikan)

Repositori ini dioptimalkan untuk dibuka dengan [Obsidian](https://obsidian.md/) sebagai vault:
1. Buka Obsidian → "Open folder as vault"
2. Pilih folder `learn-go-with-claude-mentor`
3. Nikmati navigasi antar dokumen yang mulus!

Alternatif: buka dengan VS Code biasa.

### 4. Mulai dari Fase 1

Buka [`fase-01-go-fundamentals/README.md`](./fase-01-go-fundamentals/README.md) dan ikuti panduannya.

---

## 📂 Struktur Repositori

```
learn-go-with-claude-mentor/
│
├── README.md                          ← Kamu di sini
├── CONTRIBUTING.md                    ← Cara berkontribusi
├── LICENSE                            ← MIT License
├── CODE_OF_CONDUCT.md                 ← Kode etik komunitas
├── CHANGELOG.md                       ← Riwayat perubahan
│
├── fase-01-go-fundamentals/
│   ├── README.md                      ← Panduan fase
│   ├── materi/                        ← Materi pembelajaran
│   │   └── FASE-1-Go-Fundamentals.md  ← 13 modul lengkap
│   └── project/
│       └── FASE-1-PRD-CLI-Todo-Manager.md
│
├── fase-02-go-intermediate/
│   ├── README.md
│   ├── materi/
│   │   └── FASE-2-Go-Intermediate.md  ← Interface, Goroutines, dll
│   └── project/
│       └── FASE-2-PRD-File-Processor.md
│
├── fase-03-gin-rest-api/
│   ├── README.md
│   ├── materi/
│   │   └── FASE-3-Gin-REST-API.md
│   └── project/
│       └── FASE-3-PRD-Blog-API.md
│
├── fase-04-clean-architecture/
│   ├── README.md
│   ├── materi/
│   │   └── FASE-4-Clean-Architecture.md
│   └── project/
│       └── FASE-4-PRD-User-Auth-Service.md
│
├── fase-05-grpc-protobuf/
│   ├── README.md
│   ├── materi/
│   │   └── FASE-5-gRPC-Protobuf.md
│   └── project/
│       └── FASE-5-PRD-Product-Catalog.md
│
├── fase-06-ddd/
│   ├── README.md
│   ├── materi/
│   │   └── FASE-6-DDD.md
│   └── project/
│       └── FASE-6-PRD-Order-Service.md
│
├── fase-07-microservices/             ← Coming Soon
├── fase-08-message-broker/            ← Coming Soon
├── fase-09-observability/             ← Coming Soon
├── fase-10-testing-mastery/           ← Coming Soon
├── fase-11-devops-deployment/         ← Coming Soon
│
└── resources/
    ├── cheatsheets/                   ← Go cheatsheets
    ├── tools/                         ← Tools & setup guides
    └── references/                    ← Referensi tambahan
```

---

## 🎓 Filosofi Pembelajaran

### Prinsip Utama

1. **Learning by Doing** — setiap fase berakhir dengan project nyata
2. **Progressive Complexity** — setiap fase membangun di atas yang sebelumnya
3. **Industry-Aligned** — pola dan arsitektur yang dipakai di perusahaan tech nyata
4. **No Copy-Paste** — selalu ketik ulang kode untuk muscle memory
5. **Test Everything** — testing adalah bagian integral, bukan opsional

### Cara Belajar yang Direkomendasikan

```
Untuk setiap modul:
1. Baca teori dan pahami konsepnya
2. Ketik ulang (JANGAN copy-paste) semua contoh kode
3. Eksperimen — ubah kode, lihat apa yang terjadi
4. Kerjakan latihan di akhir modul
5. Commit ke Git setiap hari

Untuk setiap project:
1. Baca PRD dengan teliti
2. Buat rencana implementasi dulu (30 menit)
3. Mulai dari bagian paling dasar
4. Test setiap komponen sebelum lanjut
5. Push ke GitHub dengan commit history yang rapi
```

---

## 🤖 Dibuat dengan Claude AI

Kurikulum ini dibuat menggunakan **[Claude](https://claude.ai) by Anthropic** sebagai AI mentor yang membantu:
- Merancang struktur kurikulum yang progresif
- Menulis contoh kode yang idiomatic dan production-ready
- Membuat PRD yang realistis sesuai standar industri
- Cross-checking best practices Go terkini

> *"AI bukan pengganti belajar — AI adalah akselerator belajar."*

Meski dibuat dengan bantuan AI, setiap konten telah:
- Divalidasi terhadap Go official documentation
- Disesuaikan dengan best practices industri
- Dirancang untuk pemahaman manusia, bukan output AI

**Ingin membuat kurikulum serupa?** Lihat [CLAUDE_PROMPTS.md](./resources/CLAUDE_PROMPTS.md) untuk melihat bagaimana kurikulum ini dibuat.

---

## 🤝 Berkontribusi

Proyek ini **open source** dan menyambut kontribusi dari siapapun! Lihat [CONTRIBUTING.md](CONTRIBUTING.md) untuk panduan lengkap.

Cara termudah untuk berkontribusi:
- ⭐ **Star** repositori ini jika bermanfaat
- 🐛 **Laporkan bug** atau kesalahan via [Issues](https://github.com/ilramdhan/learn-go-with-claude-mentor/issues)
- 📝 **Perbaiki typo** atau klarifikasi penjelasan
- 💡 **Usulkan topik** baru via [Discussions](https://github.com/ilramdhan/learn-go-with-claude-mentor/discussions)
- 🌍 **Terjemahkan** ke bahasa lain

---

## 📊 Statistik Kurikulum

| Metrik | Nilai |
|--------|-------|
| Total fase | 11 fase |
| Total modul | 130+ modul |
| Total baris materi | 15,000+ baris |
| Project latihan | 11 project dengan PRD |
| Fase sudah selesai | 6 dari 11 |
| Bahasa | Indonesia (dengan kode dalam bahasa Inggris) |

---

## 📜 Lisensi

Proyek ini dilisensikan di bawah **MIT License** — lihat [LICENSE](LICENSE) untuk detail.

Kamu bebas untuk:
- ✅ Menggunakan untuk belajar pribadi
- ✅ Berbagi ke orang lain
- ✅ Modifikasi untuk kebutuhanmu
- ✅ Menggunakan di kelas/workshop (dengan atribusi)

---

## 🙏 Acknowledgments

- **[Go Team at Google](https://go.dev)** — untuk bahasa yang luar biasa
- **[roadmap.sh](https://roadmap.sh/golang)** — sebagai referensi roadmap
- **[Anthropic](https://anthropic.com)** — untuk Claude AI yang membantu membuat kurikulum ini
- **Komunitas Gopher Indonesia** — untuk inspirasi dan feedback
- **Semua kontributor** — yang telah membantu memperbaiki konten

---

<div align="center">

**⭐ Jika repositori ini bermanfaat, tolong berikan star!**

*Dibuat dengan ❤️ untuk komunitas Go Indonesia dan dunia*

</div>

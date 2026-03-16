# 🤖 Cara Kurikulum Ini Dibuat dengan Claude AI

Dokumen ini menjelaskan transparansi tentang bagaimana kurikulum **Learn Go with Claude Mentor** dibuat menggunakan Claude AI by Anthropic.

---

## 🎯 Tujuan Dokumen Ini

1. **Transparansi** — kami percaya pada keterbukaan tentang penggunaan AI
2. **Reproduksi** — kamu bisa membuat kurikulum serupa untuk bahasa/framework lain
3. **Kontribusi** — memahami proses pembuatan membantu kamu berkontribusi dengan lebih baik

---

## 🔄 Proses Pembuatan

### 1. Initial Conversation

Kurikulum dimulai dari diskusi antara pemilik repo dan Claude:

```
User: "Saya ingin belajar Go dari nol hingga bisa membuat 
       microservices dengan gRPC, DDD, dan message broker. 
       Bisakah kamu menjadi mentor saya?"

Claude: [Membuat kurikulum berstruktur 11 fase]
```

### 2. Customization Phase

Sebelum membuat materi, Claude mengajukan pertanyaan klarifikasi:
- Level pengalaman programming saat ini
- Framework yang ingin dipelajari pertama
- Format belajar yang diinginkan

### 3. Content Generation

Setiap fase dibuat dengan instruksi spesifik:

```
"Buat materi Fase 1 Go Fundamentals yang:
- Mencakup semua modul dari setup hingga defer/panic/recover
- Setiap modul punya contoh kode LENGKAP yang bisa di-compile
- Menggunakan komentar penjelasan yang mengajarkan MENGAPA
- Diakhiri dengan latihan yang menguji pemahaman
- Menggunakan bahasa Indonesia untuk penjelasan
- Konsisten dengan roadmap.sh/golang"
```

### 4. PRD Generation

Untuk setiap project:

```
"Buat PRD untuk project [nama] yang:
- Realistis seperti project yang diberikan ke junior developer
- Mencakup business requirements, bukan hanya technical
- Ada test cases yang bisa dijalankan manual
- Ada kriteria penilaian yang jelas (minimum/good/excellent)
- Panduan pengerjaan per hari"
```

### 5. Cross-check & Validation

Setiap fase di-cross-check untuk:
- Kelengkapan (coverage roadmap.sh)
- Konsistensi (terminologi, format)
- Ketepatan teknis (apakah kode bisa di-compile)
- Progressivitas (apakah level naik dengan tepat)

---

## 📊 Statistik Pembuatan

| Item | Detail |
|------|--------|
| Platform AI | Claude Sonnet (Anthropic) |
| Total sesi | Multiple sessions |
| Total token | ~500,000+ tokens |
| Waktu pembuatan | ~4-5 jam untuk 6 fase |
| Bahasa output | Indonesia + Go code (English) |
| Review manusia | Ya — oleh pemilik repositori |

---

## ⚠️ Disclaimer

Meskipun dibuat dengan AI, semua konten telah:

1. **Divalidasi** — kode dicek bisa di-compile dan konsep sesuai dokumentasi resmi Go
2. **Di-review** — pemilik repositori yang berpengalaman Go memeriksa akurasi
3. **Disesuaikan** — disesuaikan untuk konteks belajar Indonesia

**AI bukan pengganti expertise manusia.** Jika menemukan kesalahan, silakan buka [Issues](https://github.com/ilramdhan/learn-go-with-claude-mentor/issues).

---

## 🔧 Cara Membuat Kurikulum Serupa

Jika ingin membuat kurikulum untuk bahasa/framework lain menggunakan Claude:

### Template Prompt Awal

```
Saya ingin belajar [BAHASA/FRAMEWORK] dari nol.
Goals saya:
1. [Goal 1]
2. [Goal 2]

Background saya: [level pengalaman]

Bisakah kamu menjadi mentor saya dan buat kurikulum lengkap 
dalam format markdown yang mencakup:
- Roadmap terstruktur dengan durasi
- Materi per fase dengan contoh kode lengkap
- Latihan per modul
- Project dengan PRD untuk setiap fase

Format: Multiple markdown files, satu per fase
Bahasa: [bahasa pengantar]
```

### Tips untuk Hasil Terbaik

1. **Spesifik** tentang goals dan level pengalaman
2. **Minta kode yang bisa di-compile** — bukan pseudo-code
3. **Minta penjelasan "mengapa"** bukan hanya "bagaimana"
4. **Iteratif** — review setiap fase sebelum minta fase berikutnya
5. **Cross-check** dengan dokumentasi resmi bahasa tersebut

---

## 📚 Referensi yang Digunakan Claude

- [Go Official Documentation](https://go.dev/doc/)
- [roadmap.sh/golang](https://roadmap.sh/golang)
- [Effective Go](https://go.dev/doc/effective_go)
- [Go by Example](https://gobyexample.com/)
- Clean Architecture by Robert C. Martin
- Domain-Driven Design by Eric Evans
- Building Microservices by Sam Newman

---

*Kurikulum ini merupakan contoh kolaborasi produktif antara manusia dan AI. AI membantu dalam skala dan kecepatan pembuatan konten, sementara manusia memberikan arah, validasi, dan konteks yang tepat.*

# 🤝 Panduan Kontribusi

Terima kasih atas minatmu untuk berkontribusi ke **Learn Go with Claude Mentor**! Setiap kontribusi, sekecil apapun, sangat berarti bagi komunitas.

---

## 📋 Daftar Isi

- [Kode Etik](#kode-etik)
- [Cara Berkontribusi](#cara-berkontribusi)
- [Jenis Kontribusi](#jenis-kontribusi)
- [Setup Development](#setup-development)
- [Standar Penulisan](#standar-penulisan)
- [Proses Pull Request](#proses-pull-request)
- [Pertanyaan & Diskusi](#pertanyaan--diskusi)

---

## 📜 Kode Etik

Proyek ini mengikuti [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). Dengan berpartisipasi, kamu diharapkan menjaga lingkungan yang inklusif, ramah, dan konstruktif.

---

## 🚀 Cara Berkontribusi

### 1. Fork & Clone

```bash
# Fork repositori ini di GitHub, lalu:
git clone https://github.com/USERNAME/learn-go-with-claude-mentor.git
cd learn-go-with-claude-mentor

# Tambahkan upstream
git remote add upstream https://github.com/ilramdhan/learn-go-with-claude-mentor.git
```

### 2. Buat Branch

```bash
# Selalu buat branch baru dari main
git checkout main
git pull upstream main
git checkout -b type/deskripsi-singkat

# Contoh:
git checkout -b fix/typo-fase1-modul3
git checkout -b feat/tambah-latihan-goroutine
git checkout -b docs/update-readme-fase2
```

### 3. Buat Perubahan

Ikuti [standar penulisan](#standar-penulisan) di bawah.

### 4. Commit

```bash
# Gunakan format Conventional Commits
git add .
git commit -m "fix: koreksi typo pada modul 1.3 variables"
git commit -m "feat: tambah latihan untuk goroutines di fase 2"
git commit -m "docs: update README fase 3 dengan contoh output"
```

### 5. Push & Pull Request

```bash
git push origin branch-kamu
# Buka GitHub dan buat Pull Request
```

---

## 🎯 Jenis Kontribusi

### 🐛 Bug / Kesalahan Materi

Temukan kesalahan? Laporkan via [Issues](https://github.com/ilramdhan/learn-go-with-claude-mentor/issues):

- Kesalahan kode (compile error, logic error)
- Kesalahan konsep / penjelasan yang menyesatkan
- Link yang rusak
- Typo atau kesalahan grammar

**Template Issue:**
```
Fase: [contoh: Fase 1, Modul 1.3]
Lokasi: [nama file + nomor baris jika bisa]
Masalah: [jelaskan masalahnya]
Usulan perbaikan: [opsional]
```

### ✨ Perbaikan & Tambahan

- Perjelas penjelasan yang membingungkan
- Tambah contoh kode alternatif
- Tambah latihan baru
- Perbaiki kode yang tidak idiomatic

### 🌍 Terjemahan

Ingin menerjemahkan ke bahasa lain? Buat folder baru:
```
fase-01-go-fundamentals/
  materi/
    FASE-1-Go-Fundamentals.md          ← Bahasa Indonesia (original)
    FASE-1-Go-Fundamentals.en.md       ← English translation
    FASE-1-Go-Fundamentals.ja.md       ← Japanese translation
```

### 📝 Konten Baru

Untuk menambah modul atau fase baru, **diskusikan dulu** via [Issues](https://github.com/ilramdhan/learn-go-with-claude-mentor/issues) sebelum mulai menulis.

---

## ⚙️ Setup Development

Tidak ada dependency khusus. Kamu hanya butuh:

```bash
# Text editor favorit kamu
# Direkomendasikan: VS Code atau Obsidian untuk preview markdown

# Untuk test kode Go yang ada di materi:
go version  # minimal 1.22
```

### Preview Markdown

```bash
# VS Code: install extension "Markdown Preview Enhanced"
# Obsidian: buka folder sebagai vault
# Terminal: npm install -g markdownlint-cli
```

---

## 📏 Standar Penulisan

### Format Dokumen

Setiap file markdown harus mengikuti format ini:

```markdown
# 📘 FASE N: Judul Fase

> **Prasyarat:** ...
> **Durasi:** ...
> **Project Akhir:** ...
> **Tujuan:** ...

---

## 🗂️ Daftar Modul

| # | Modul | Topik |
|---|-------|-------|
...

---

## 📦 Modul N.X — Judul Modul

[konten]

### 🏋️ Latihan N.X

[latihan]
```

### Standar Kode

```go
// ✅ BAIK: Kode harus:
// 1. Bisa di-compile (tidak ada syntax error)
// 2. Idiomatic Go (gunakan go fmt style)
// 3. Menggunakan error handling yang proper
// 4. Punya komentar yang menjelaskan MENGAPA, bukan APA

// ✅ BAIK: Gunakan komentar deskriptif
func validateEmail(email string) error {
    // Trim whitespace karena user sering tidak sengaja tambah spasi
    email = strings.TrimSpace(email)
    if !emailRegex.MatchString(email) {
        return fmt.Errorf("invalid email format: %s", email)
    }
    return nil
}

// ❌ BURUK: Komentar yang tidak berguna
func validateEmail(email string) error {
    // validate email
    email = strings.TrimSpace(email)
    if !emailRegex.MatchString(email) {
        return fmt.Errorf("error")  // error yang tidak informatif
    }
    return nil
}
```

### Bahasa

- **Teks penjelasan**: Bahasa Indonesia
- **Kode Go**: Bahasa Inggris (nama variable, fungsi, komentar dalam kode)
- **Judul section**: Bahasa Indonesia dengan emoji
- **Error messages**: Bahasa Inggris (sesuai konvensi Go)

### Emoji yang Digunakan

| Emoji | Penggunaan |
|-------|-----------|
| 📦 | Modul baru |
| 🏋️ | Latihan |
| ✅ | Contoh yang BENAR |
| ❌ | Contoh yang SALAH / Anti-pattern |
| ⚠️ | Peringatan penting |
| 💡 | Tips tambahan |
| 🎯 | Goal / Checkpoint |
| 📚 | Referensi |

---

## 🔄 Proses Pull Request

### Checklist PR

Sebelum submit PR, pastikan:

- [ ] Kode bisa di-compile (jalankan `go build ./...`)
- [ ] Tidak ada typo pada teks
- [ ] Format konsisten dengan file lain di fase yang sama
- [ ] Branch sudah up-to-date dengan `main`
- [ ] Judul PR jelas dan deskriptif
- [ ] Deskripsi PR menjelaskan apa yang diubah dan mengapa

### Template PR Description

```markdown
## Ringkasan Perubahan
[Jelaskan apa yang diubah dan mengapa]

## Jenis Perubahan
- [ ] Bug fix (perbaikan kesalahan)
- [ ] Tambahan materi baru
- [ ] Perbaikan penjelasan
- [ ] Terjemahan
- [ ] Lainnya: ...

## Fase yang Terpengaruh
- [ ] Fase 1
- [ ] Fase 2
- [ ] ...

## Checklist
- [ ] Kode bisa di-compile
- [ ] Tidak ada typo
- [ ] Format konsisten
```

### Review Process

1. PR akan di-review dalam **3–7 hari kerja**
2. Review berfokus pada: akurasi teknis, kejelasan penjelasan, konsistensi format
3. Mungkin ada request perubahan — ini normal dan konstruktif
4. Setelah approved, PR akan di-merge ke `main`

---

## 💬 Pertanyaan & Diskusi

- **Pertanyaan umum**: [GitHub Discussions](https://github.com/ilramdhan/learn-go-with-claude-mentor/discussions)
- **Bug report**: [GitHub Issues](https://github.com/ilramdhan/learn-go-with-claude-mentor/issues)
- **Ide besar**: Buka Discussion dulu sebelum mulai implementasi

---

## 🏆 Hall of Fame

Semua kontributor akan ditampilkan di [README.md](README.md) dan mendapat badge kontributor! ⭐

---

*Terima kasih telah berkontribusi! Setiap perbaikan, sekecil apapun, membantu ribuan developer belajar Go dengan lebih baik.* 🐹

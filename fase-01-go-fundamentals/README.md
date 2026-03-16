# 📘 Fase 1: Go Fundamentals

> **Level:** 🟢 Beginner  
> **Durasi Estimasi:** 2–3 minggu  
> **Prasyarat:** Pengalaman coding di bahasa apapun (Python/JS/PHP/dll)

---

## 🎯 Tujuan Fase Ini

Setelah menyelesaikan fase ini, kamu akan:
- ✅ Memahami syntax dan type system Go
- ✅ Bisa menulis program Go yang berfungsi
- ✅ Memahami idiom-idiom dasar Go
- ✅ Siap mengerjakan project CLI pertama

---

## 🗂️ Isi Folder

```
fase-01-go-fundamentals/
├── README.md                          ← Kamu di sini
├── materi/
│   └── FASE-1-Go-Fundamentals.md     ← 13 modul materi lengkap
└── project/
    └── FASE-1-PRD-CLI-Todo-Manager.md ← PRD project akhir
```

---

## 📚 Daftar Modul

| # | Modul | Konsep Utama |
|---|-------|-------------|
| 1.1 | Setup & Workspace | Instalasi, go mod, VS Code |
| 1.2 | Hello, Go! | Program pertama, fmt package |
| 1.3 | Variables & Types | var, :=, zero values, type conversion |
| 1.4 | Constants & iota | const, enum pattern dengan iota |
| 1.5 | Control Flow | if/else, switch, for (satu-satunya loop!) |
| 1.6 | Functions | params, multiple return, variadic, closure |
| 1.7 | Arrays & Slices | perbedaan array vs slice, append, copy |
| 1.8 | Maps | CRUD, nil map, map sebagai set |
| 1.9 | Structs & Methods | struct, method, embedded struct |
| 1.10 | Pointers | pointer basics, value vs pointer receiver |
| 1.11 | Error Handling | error interface, custom error, wrapping |
| 1.12 | Packages & Modules | go mod, import, visibility rules |
| 1.13 | Defer, Panic, Recover | cleanup pattern, graceful recovery |

---

## 🚀 Cara Memulai

### Step 1: Install Go

```bash
# Download dari https://go.dev/dl/
# Verifikasi:
go version  # harus 1.22+
```

### Step 2: Setup VS Code

1. Install VS Code
2. Install extension **"Go" by Google**
3. Buka Command Palette (`Ctrl+Shift+P`) → **"Go: Install/Update Tools"**
4. Pilih semua → Install

### Step 3: Buat Folder Kerja

```bash
mkdir ~/go-learning
cd ~/go-learning
mkdir fase-1
cd fase-1
```

### Step 4: Mulai dari Modul 1.1

Buka `materi/FASE-1-Go-Fundamentals.md` dan mulai dari **Modul 1.1 — Setup & Workspace**.

---

## 🏋️ Cara Belajar yang Benar

```
❌ JANGAN lakukan ini:
   - Copy-paste kode langsung
   - Skip latihan di akhir modul
   - Lanjut ke modul berikutnya sebelum paham

✅ LAKUKAN ini:
   - Ketik ulang setiap contoh kode
   - Eksperimen: ubah kode, lihat apa yang terjadi
   - Kerjakan SEMUA latihan
   - Commit ke Git setiap hari, meski kecil
```

---

## 🎯 Project Akhir: CLI Todo Manager

Setelah semua 13 modul selesai, kerjakan project **CLI Todo Manager** berdasarkan PRD di:
`project/FASE-1-PRD-CLI-Todo-Manager.md`

Project ini adalah aplikasi manajemen todo yang berjalan di terminal, dengan fitur:
- Tambah, lihat, update, hapus todo
- Priority system (low/medium/high)
- Simpan data ke JSON
- Multiple packages

**Estimasi pengerjaan:** 3–5 hari

---

## ✅ Checklist Kelulusan

Sebelum lanjut ke Fase 2, pastikan kamu bisa:

- [ ] Membuat program Go dari nol tanpa melihat materi
- [ ] Menjelaskan perbedaan array dan slice
- [ ] Membuat struct dengan method yang tepat
- [ ] Handle error dengan benar (bukan `_` untuk semua error!)
- [ ] Membuat dan mengimport package terpisah
- [ ] **Menyelesaikan project CLI Todo Manager**

---

## ➡️ Selanjutnya

Setelah selesai Fase 1 → [Fase 2: Go Intermediate](../fase-02-go-intermediate/)

---

## 💡 Tips untuk Pemula

> **"Go adalah bahasa yang sengaja dibuat sederhana. Jika kamu merasa perlu cara yang rumit untuk melakukan sesuatu, mungkin ada cara yang lebih Go-idiomatic."**

- Gunakan `gofmt` atau `go fmt ./...` untuk format kode otomatis
- Baca error message dengan teliti — Go error message sangat informatif
- Jika bingung, cek [Go by Example](https://gobyexample.com/) untuk referensi cepat
- Komunitas Go sangat ramah — tidak ada pertanyaan yang terlalu dasar

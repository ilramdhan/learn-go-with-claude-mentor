# 📋 PRD — CLI Todo Manager

> **Fase:** 1 — Go Fundamentals  
> **Tipe Project:** Command Line Interface (CLI)  
> **Estimasi Pengerjaan:** 3–5 hari  
> **Konsep yang Diuji:** Struct, Slice, Map, Error Handling, Packages, File I/O  

---

## 🎯 Tujuan Project

Membangun aplikasi **CLI Todo Manager** yang berjalan di terminal, dapat mengelola daftar tugas (CRUD), menyimpan data ke file JSON, dan diorganisasi dengan baik menggunakan multiple packages.

---

## 📌 Latar Belakang

Seorang developer butuh tools sederhana untuk mencatat tugas-tugasnya langsung dari terminal tanpa harus membuka browser atau aplikasi lain. Aplikasi ini harus ringan, cepat, dan menyimpan data secara persisten.

---

## 👤 Target Pengguna

- Developer yang terbiasa menggunakan terminal
- Pengguna yang ingin manajemen todo sederhana tanpa GUI

---

## ✅ Functional Requirements

### FR-01: Menambah Todo
- User dapat menambah todo baru dengan judul dan deskripsi opsional
- System memberikan ID unik otomatis (incremental integer)
- System mencatat tanggal pembuatan
- Status awal todo adalah "pending"

### FR-02: Melihat Daftar Todo
- User dapat melihat semua todo dalam bentuk tabel
- Tampilkan: ID, Judul, Status, Priority, Tanggal Dibuat
- User dapat filter berdasarkan status (pending/done/all)
- User dapat filter berdasarkan priority (low/medium/high)

### FR-03: Menandai Todo Selesai
- User dapat menandai todo sebagai selesai berdasarkan ID
- System mencatat tanggal penyelesaian
- Todo yang sudah selesai tidak bisa di-done lagi (tampilkan pesan)

### FR-04: Menghapus Todo
- User dapat menghapus todo berdasarkan ID
- System meminta konfirmasi sebelum menghapus (y/N)
- Jika ID tidak ditemukan, tampilkan error yang informatif

### FR-05: Update Todo
- User dapat mengubah judul dan deskripsi todo
- User dapat mengubah priority todo
- ID tidak bisa diubah

### FR-06: Persistensi Data
- Data disimpan ke file `todos.json` di direktori yang sama
- Data di-load otomatis saat program dijalankan
- Data di-save otomatis setiap ada perubahan

### FR-07: Priority System
- Todo memiliki priority: Low, Medium, High
- Default priority adalah Medium

---

## 🚫 Non-Functional Requirements

- **Performance:** Respons setiap command < 100ms
- **Reliability:** Data tidak boleh hilang saat program crash (save sebelum modifikasi)
- **Usability:** Pesan error harus jelas dan actionable
- **Portability:** Berjalan di Linux, macOS, dan Windows

---

## 💻 Spesifikasi CLI

### Command Structure

```
todo <command> [arguments] [flags]
```

### Commands

| Command | Deskripsi | Contoh |
|---------|-----------|--------|
| `add` | Tambah todo baru | `todo add "Belajar Go"` |
| `list` | Tampilkan semua todo | `todo list` |
| `done` | Tandai todo selesai | `todo done 1` |
| `delete` | Hapus todo | `todo delete 1` |
| `update` | Update todo | `todo update 1` |
| `help` | Tampilkan bantuan | `todo help` |

### Flags

| Flag | Command | Deskripsi | Contoh |
|------|---------|-----------|--------|
| `--desc` | add | Deskripsi todo | `todo add "Belajar" --desc "Belajar goroutines"` |
| `--priority` | add, update | Priority (low/medium/high) | `todo add "Task" --priority high` |
| `--status` | list | Filter status | `todo list --status pending` |
| `--priority` | list | Filter priority | `todo list --priority high` |

---

## 🖥️ Contoh Output

### `todo add "Belajar Golang" --priority high --desc "Fokus di goroutines"`
```
✅ Todo berhasil ditambahkan!
┌─────────────────────────────────────────┐
│ ID       : 1                            │
│ Judul    : Belajar Golang               │
│ Deskripsi: Fokus di goroutines          │
│ Priority : HIGH                         │
│ Status   : PENDING                      │
│ Dibuat   : 2025-01-15 10:30:00          │
└─────────────────────────────────────────┘
```

### `todo list`
```
📋 Daftar Todo (Total: 3 | Pending: 2 | Done: 1)
┌────┬──────────────────────┬─────────┬──────────┬─────────────────────┐
│ ID │ Judul                │ Status  │ Priority │ Dibuat              │
├────┼──────────────────────┼─────────┼──────────┼─────────────────────┤
│  1 │ Belajar Golang       │ PENDING │ HIGH     │ 2025-01-15 10:30:00 │
│  2 │ Buat REST API        │ PENDING │ MEDIUM   │ 2025-01-15 11:00:00 │
│  3 │ Setup Docker         │ DONE    │ LOW      │ 2025-01-14 09:00:00 │
└────┴──────────────────────┴─────────┴──────────┴─────────────────────┘
```

### `todo done 1`
```
🎉 Todo #1 berhasil diselesaikan!
"Belajar Golang" → DONE
```

### `todo delete 99`
```
❌ Error: Todo dengan ID 99 tidak ditemukan.
```

### `todo help`
```
CLI Todo Manager — Kelola tugas dari terminal

USAGE:
  todo <command> [arguments] [flags]

COMMANDS:
  add <judul>          Tambah todo baru
  list                 Tampilkan semua todo
  done <id>            Tandai todo selesai
  delete <id>          Hapus todo
  update <id>          Update todo
  help                 Tampilkan bantuan ini

FLAGS:
  --desc <text>        Deskripsi todo (untuk add)
  --priority <level>   Priority: low, medium, high (default: medium)
  --status <status>    Filter status: pending, done, all (untuk list)

EXAMPLES:
  todo add "Belajar Go"
  todo add "Deploy ke server" --priority high --desc "Deploy ke production"
  todo list
  todo list --status pending
  todo list --priority high
  todo done 1
  todo delete 2
  todo update 3
```

---

## 📁 Struktur Project

```
cli-todo/
├── go.mod
├── main.go                  ← Entry point, parse args
├── cmd/
│   ├── add.go               ← Handler command add
│   ├── list.go              ← Handler command list
│   ├── done.go              ← Handler command done
│   ├── delete.go            ← Handler command delete
│   └── update.go            ← Handler command update
├── model/
│   └── todo.go              ← Struct Todo, Priority, Status
├── storage/
│   └── storage.go           ← Load/Save ke file JSON
├── display/
│   └── display.go           ← Format output tabel
└── todos.json               ← File penyimpanan (auto-generated)
```

---

## 🗂️ Data Model

### Todo Struct

```go
type Priority string
type Status string

const (
    PriorityLow    Priority = "LOW"
    PriorityMedium Priority = "MEDIUM"
    PriorityHigh   Priority = "HIGH"
)

const (
    StatusPending Status = "PENDING"
    StatusDone    Status = "DONE"
)

type Todo struct {
    ID          int       `json:"id"`
    Title       string    `json:"title"`
    Description string    `json:"description,omitempty"`
    Status      Status    `json:"status"`
    Priority    Priority  `json:"priority"`
    CreatedAt   time.Time `json:"created_at"`
    CompletedAt *time.Time `json:"completed_at,omitempty"` // pointer karena bisa null
}
```

### Storage Format (todos.json)

```json
{
  "last_id": 3,
  "todos": [
    {
      "id": 1,
      "title": "Belajar Golang",
      "description": "Fokus di goroutines",
      "status": "DONE",
      "priority": "HIGH",
      "created_at": "2025-01-15T10:30:00Z",
      "completed_at": "2025-01-15T18:00:00Z"
    },
    {
      "id": 2,
      "title": "Buat REST API",
      "status": "PENDING",
      "priority": "MEDIUM",
      "created_at": "2025-01-15T11:00:00Z"
    }
  ]
}
```

---

## 🔄 Alur Program

```
main()
  ↓
Parse os.Args (command + flags)
  ↓
Load data dari todos.json
  ↓
Route ke handler yang sesuai
  ↓
Handler: validasi input → jalankan logic → update data
  ↓
Save data ke todos.json
  ↓
Tampilkan output ke user
```

---

## ⚠️ Error Cases yang Harus Dihandle

| Kasus | Pesan Error |
|-------|-------------|
| Command tidak dikenal | `❌ Command tidak dikenal: '<cmd>'. Jalankan 'todo help' untuk bantuan.` |
| ID bukan angka | `❌ ID harus berupa angka bulat positif.` |
| ID tidak ditemukan | `❌ Todo dengan ID <n> tidak ditemukan.` |
| Judul kosong | `❌ Judul todo tidak boleh kosong.` |
| Priority tidak valid | `❌ Priority tidak valid. Gunakan: low, medium, high` |
| Todo sudah done | `⚠️ Todo #<n> sudah selesai sebelumnya.` |
| File JSON corrupt | `❌ Data file corrupt. Apakah ingin reset? (y/N)` |

---

## 🧪 Test Cases Manual

Jalankan semua command berikut dan verifikasi outputnya:

```bash
# 1. Tambah beberapa todo
go run . add "Belajar Go basics"
go run . add "Buat REST API" --priority high --desc "Pakai Gin framework"
go run . add "Setup PostgreSQL" --priority high
go run . add "Belajar Docker" --priority low

# 2. Lihat semua todo
go run . list

# 3. Filter berdasarkan priority
go run . list --priority high

# 4. Selesaikan todo
go run . done 1
go run . done 1  # harusnya tampil warning

# 5. Lihat lagi — pastikan status berubah
go run . list

# 6. Filter berdasarkan status
go run . list --status done
go run . list --status pending

# 7. Update todo
go run . update 2  # interactive: tanya judul baru, desc, priority

# 8. Hapus todo
go run . delete 4  # harus minta konfirmasi
go run . delete 99  # harusnya error

# 9. Error cases
go run . add ""        # judul kosong
go run . done abc      # ID bukan angka
go run . add "Task" --priority super  # priority tidak valid
```

---

## 🏆 Kriteria Penilaian

### Minimum (Wajib)
- [ ] Command `add`, `list`, `done`, `delete` berjalan dengan benar
- [ ] Data persisten di file JSON
- [ ] Error handling yang tepat
- [ ] Struktur package yang rapi

### Good
- [ ] Command `update` berjalan
- [ ] Filter by status dan priority
- [ ] Output tabel yang rapi
- [ ] Semua error cases tertangani

### Excellent
- [ ] Konfirmasi sebelum delete
- [ ] Output dengan emoji dan warna (menggunakan package `github.com/fatih/color`)
- [ ] Sort todo berdasarkan priority atau tanggal
- [ ] Unit test untuk model dan storage package

---

## 📦 Packages yang Boleh Dipakai

### Hanya Standard Library (Direkomendasikan untuk belajar)
- `os` — args, file
- `fmt` — output
- `encoding/json` — JSON encode/decode
- `time` — tanggal waktu
- `strconv` — konversi string-angka
- `strings` — manipulasi string
- `sort` — sorting

### Optional (Untuk Excellent Level)
- `github.com/fatih/color` — warna di terminal

**DILARANG menggunakan framework atau library CLI** (cobra, urfave/cli, dll) — kita belajar dari scratch!

---

## 📖 Panduan Pengerjaan

### Hari 1: Setup & Model
1. Init project dengan `go mod init`
2. Buat folder structure
3. Buat struct `Todo`, constants `Priority`, `Status` di `model/todo.go`
4. Buat fungsi-fungsi di `model/todo.go`: `NewTodo()`, method `MarkDone()`, `IsValid()`
5. Test di playground atau main sementara

### Hari 2: Storage
1. Buat `storage/storage.go`
2. Struct `Storage` dengan field `LastID` dan `Todos []Todo`
3. Fungsi `Load(filepath string) (*Storage, error)`
4. Fungsi `Save(filepath string) error`
5. Fungsi CRUD: `Add()`, `FindByID()`, `Update()`, `Delete()`, `GetAll()`
6. Test storage dengan main sementara

### Hari 3: Commands & Display
1. Buat `display/display.go` untuk format output tabel
2. Implementasi semua command handlers di `cmd/`
3. Parse arguments di `main.go`
4. Connect semua bagian

### Hari 4: Polish & Error Handling
1. Test semua test cases manual
2. Perbaiki semua error cases
3. Tambahkan pesan error yang informatif
4. Cleanup dan refactor

### Hari 5: Optional Features
1. Tambah warna (jika mau)
2. Tambah unit tests
3. Update README

---

## 📝 Deliverable

1. **Source code** di folder `cli-todo/`
2. **README.md** berisi:
   - Cara install dan run
   - Daftar semua command dan flag
   - Screenshot/contoh output
3. **Push ke GitHub**

---

*Good luck! Jika stuck, ingat: google dulu 30 menit, baru tanya mentor 😄*

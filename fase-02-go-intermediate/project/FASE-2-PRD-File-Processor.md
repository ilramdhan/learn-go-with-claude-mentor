# 📋 PRD — Concurrent File Processor CLI

> **Fase:** 2 — Go Intermediate  
> **Tipe Project:** CLI Tool dengan Concurrency  
> **Estimasi Pengerjaan:** 4–6 hari  
> **Konsep yang Diuji:** Goroutines, Channels, WaitGroup, Context, Interface, File I/O, JSON

---

## 🎯 Tujuan Project

Membangun **File Processor CLI** yang memproses banyak file secara **concurrent**, mendukung berbagai operasi (word count, search, stats), dan menghasilkan laporan dalam format JSON atau tabel.

---

## 📌 Latar Belakang

Seorang developer sering perlu menganalisis banyak file teks sekaligus: menghitung kata, mencari pattern, atau menghasilkan statistik. Memproses satu per satu terlalu lambat. Tool ini memproses banyak file secara paralel menggunakan goroutines.

---

## ✅ Functional Requirements

### FR-01: Scan Direktori
- User dapat menentukan direktori yang akan diproses
- System scan semua file di direktori (dan subdirektori dengan flag `--recursive`)
- Support filter extension file (e.g., hanya `.txt`, `.go`, `.md`)
- Tampilkan jumlah file yang ditemukan sebelum diproses

### FR-02: Word Count
- Hitung jumlah kata per file
- Hitung jumlah baris per file
- Hitung jumlah karakter per file
- Tampilkan total keseluruhan

### FR-03: Search (Grep-like)
- Cari string/pattern di semua file secara concurrent
- Tampilkan: nama file, nomor baris, dan isi baris yang cocok
- Support case-insensitive search dengan flag `--ignore-case`
- Tampilkan jumlah total match yang ditemukan

### FR-04: File Stats
- Tampilkan statistik per file: ukuran (bytes), tanggal modifikasi, jumlah baris
- Sort berdasarkan ukuran, nama, atau tanggal (dengan flag)
- Hitung total ukuran semua file

### FR-05: Output Format
- Default: output tabel di terminal
- Flag `--json`: output dalam format JSON
- Flag `--output <file>`: simpan output ke file

### FR-06: Concurrency Control
- Flag `--workers <n>`: jumlah worker goroutine (default: jumlah CPU)
- Tampilkan progress bar atau counter file yang sudah diproses
- Gunakan context untuk timeout (`--timeout <seconds>`)

### FR-07: Error Handling
- File yang tidak bisa dibaca: catat error, lanjutkan proses file lain
- Tampilkan summary error di akhir
- Exit code 1 jika ada error yang terjadi

---

## 💻 Spesifikasi CLI

```
fileproc <command> <directory> [flags]
```

### Commands

| Command | Deskripsi |
|---------|-----------|
| `count` | Word/line/char count semua file |
| `search <pattern>` | Cari pattern di semua file |
| `stats` | Statistik file (ukuran, tanggal, dll) |
| `help` | Bantuan |

### Flags

| Flag | Default | Deskripsi |
|------|---------|-----------|
| `--ext <ext>` | semua | Filter ekstensi (e.g., `.txt,.go`) |
| `--recursive` | false | Proses subdirektori |
| `--workers <n>` | NumCPU | Jumlah worker goroutine |
| `--timeout <n>` | 30 | Timeout dalam detik |
| `--json` | false | Output JSON |
| `--output <file>` | stdout | File output |
| `--ignore-case` | false | Case-insensitive (untuk search) |
| `--sort <field>` | name | Sort by: name, size, lines, date |

---

## 🖥️ Contoh Output

### `fileproc count ./src --ext .go --recursive`

```
📁 Scanning ./src...
   Found 12 files (*.go)

⚙️  Processing with 4 workers...

Progress: [████████████████████] 12/12 files

📊 Word Count Results
┌────────────────────────────────┬────────┬───────┬────────────┐
│ File                           │ Lines  │ Words │ Characters │
├────────────────────────────────┼────────┼───────┼────────────┤
│ src/main.go                    │     45 │   312 │      2,841 │
│ src/handler/user.go            │    120 │   890 │      7,234 │
│ src/model/product.go           │     78 │   543 │      4,102 │
│ ...                            │    ... │   ... │        ... │
├────────────────────────────────┼────────┼───────┼────────────┤
│ TOTAL (12 files)               │    892 │ 6,234 │     52,103 │
└────────────────────────────────┴────────┴───────┴────────────┘

⏱  Completed in 234ms
```

### `fileproc search "TODO" ./src --ext .go --ignore-case`

```
📁 Scanning ./src...
   Found 12 files (*.go)

🔍 Searching for: "TODO" (case-insensitive)

src/handler/user.go:
  Line  23: // TODO: add validation
  Line  87: // todo: handle edge case

src/model/product.go:
  Line  45: // TODO: add caching

✅ Found 3 matches in 2 files
⏱  Completed in 89ms
```

### `fileproc stats ./src --sort size`

```
📁 Scanning ./src...

📈 File Statistics (sorted by size)
┌────────────────────────────┬──────────┬───────┬─────────────────────┐
│ File                       │   Size   │ Lines │ Last Modified       │
├────────────────────────────┼──────────┼───────┼─────────────────────┤
│ src/handler/user.go        │  7.1 KB  │   120 │ 2025-01-15 10:30:00 │
│ src/model/product.go       │  4.0 KB  │    78 │ 2025-01-14 09:00:00 │
│ src/main.go                │  2.8 KB  │    45 │ 2025-01-15 11:00:00 │
├────────────────────────────┼──────────┼───────┼─────────────────────┤
│ TOTAL (12 files)           │ 50.9 KB  │   892 │                     │
└────────────────────────────┴──────────┴───────┴─────────────────────┘
```

### Output JSON (`--json`)

```json
{
  "command": "count",
  "directory": "./src",
  "files_processed": 12,
  "duration_ms": 234,
  "results": [
    {
      "file": "src/main.go",
      "lines": 45,
      "words": 312,
      "characters": 2841
    }
  ],
  "totals": {
    "lines": 892,
    "words": 6234,
    "characters": 52103
  },
  "errors": []
}
```

---

## 📁 Struktur Project

```
file-processor/
├── go.mod
├── main.go                    ← Entry point
├── cmd/
│   ├── root.go                ← Root command, flag parsing
│   ├── count.go               ← Command: count
│   ├── search.go              ← Command: search
│   └── stats.go               ← Command: stats
├── processor/
│   ├── processor.go           ← Interface Processor
│   ├── counter.go             ← WordCounter implementation
│   ├── searcher.go            ← Searcher implementation
│   └── stats.go               ← StatsCollector implementation
├── worker/
│   └── pool.go                ← Worker pool implementation
├── scanner/
│   └── scanner.go             ← Directory scanner
├── output/
│   ├── table.go               ← Table formatter
│   └── json.go                ← JSON formatter
└── model/
    ├── file.go                ← FileInfo struct
    ├── result.go              ← Result structs
    └── config.go              ← Config/options struct
```

---

## 🗂️ Data Models

```go
// model/config.go
type Config struct {
    Directory   string
    Extensions  []string
    Recursive   bool
    Workers     int
    Timeout     int
    OutputJSON  bool
    OutputFile  string
    IgnoreCase  bool
    SortBy      string
}

// model/file.go
type FileInfo struct {
    Path     string
    Name     string
    Size     int64
    ModTime  time.Time
    Lines    int
}

// model/result.go
type CountResult struct {
    File       string `json:"file"`
    Lines      int    `json:"lines"`
    Words      int    `json:"words"`
    Characters int    `json:"characters"`
    Error      string `json:"error,omitempty"`
}

type SearchMatch struct {
    File    string `json:"file"`
    Line    int    `json:"line"`
    Content string `json:"content"`
}

type SearchResult struct {
    File    string        `json:"file"`
    Matches []SearchMatch `json:"matches"`
    Error   string        `json:"error,omitempty"`
}

type StatsResult struct {
    File    string    `json:"file"`
    Size    int64     `json:"size"`
    Lines   int       `json:"lines"`
    ModTime time.Time `json:"mod_time"`
    Error   string    `json:"error,omitempty"`
}

type Report[T any] struct {
    Command          string   `json:"command"`
    Directory        string   `json:"directory"`
    FilesProcessed   int      `json:"files_processed"`
    DurationMs       int64    `json:"duration_ms"`
    Results          []T      `json:"results"`
    Errors           []string `json:"errors"`
}
```

---

## 🔄 Arsitektur Concurrency

```
main()
  ↓
Scanner → scan directory → []FileInfo
  ↓
WorkerPool
  ├── Worker 1 ──┐
  ├── Worker 2 ──┤← jobs channel (FileInfo)
  ├── Worker 3 ──┤
  └── Worker N ──┘
                 ↓
         results channel
                 ↓
         Collector goroutine
                 ↓
         Report + Output
```

### Interface yang Harus Dibuat

```go
// processor/processor.go
type Processor interface {
    Process(ctx context.Context, file FileInfo) (interface{}, error)
    Name() string
}

// Setiap command implement interface ini
type WordCounter struct{}
func (w *WordCounter) Process(ctx context.Context, file FileInfo) (interface{}, error)
func (w *WordCounter) Name() string { return "count" }

type Searcher struct {
    Pattern    string
    IgnoreCase bool
}
func (s *Searcher) Process(ctx context.Context, file FileInfo) (interface{}, error)

type StatsCollector struct{}
func (s *StatsCollector) Process(ctx context.Context, file FileInfo) (interface{}, error)
```

---

## ⚙️ Worker Pool Specification

```go
// worker/pool.go
type Job struct {
    File FileInfo
}

type Result struct {
    File  FileInfo
    Data  interface{}
    Error error
}

type Pool struct {
    workers int
    jobs    chan Job
    results chan Result
}

func NewPool(workers int) *Pool
func (p *Pool) Run(ctx context.Context, processor Processor, files []FileInfo) []Result
```

---

## 🧪 Test Cases Manual

```bash
# Setup: buat test directory dengan beberapa file
mkdir -p testdata/sub
echo "Hello World\nThis is a test\nGo is awesome" > testdata/file1.txt
echo "Another file\nWith some content\nTODO: fix this" > testdata/file2.txt
echo "package main\n\nimport \"fmt\"\n// TODO: add tests\nfunc main() {}" > testdata/sub/main.go

# 1. Count di direktori
go run . count ./testdata

# 2. Count recursive dengan filter
go run . count ./testdata --recursive --ext .go

# 3. Search
go run . search "TODO" ./testdata --recursive

# 4. Search case-insensitive
go run . search "hello" ./testdata --ignore-case

# 5. Stats sorted by size
go run . stats ./testdata --sort size

# 6. Output JSON
go run . count ./testdata --json

# 7. Output ke file
go run . count ./testdata --json --output report.json
cat report.json

# 8. Dengan timeout singkat (simulasi timeout)
go run . count ./testdata --timeout 0

# 9. Custom workers
go run . count ./testdata --workers 1

# 10. Error case: direktori tidak ada
go run . count ./tidak-ada
```

---

## 🏆 Kriteria Penilaian

### Minimum (Wajib)
- [ ] Command `count` berjalan concurrent
- [ ] Worker pool dengan goroutines dan channels
- [ ] Context dengan timeout
- [ ] Error handling per file (tidak crash jika satu file error)

### Good
- [ ] Semua command (`count`, `search`, `stats`)
- [ ] Interface `Processor` diimplementasikan
- [ ] Output JSON dan tabel
- [ ] Progress counter

### Excellent
- [ ] Generics untuk `Report[T any]`
- [ ] Benchmark test (go test -bench)
- [ ] Table-driven unit tests untuk setiap processor
- [ ] Graceful shutdown dengan context cancellation

---

## 📦 Packages yang Boleh Dipakai

### Standard Library
- `os`, `io`, `bufio` — file operations
- `path/filepath` — path manipulation
- `encoding/json` — JSON output
- `sync` — WaitGroup, Mutex
- `context` — timeout, cancellation
- `runtime` — NumCPU
- `strings`, `regexp` — string operations
- `sort` — sorting
- `time` — timing, duration
- `fmt` — output formatting

### Dilarang
- Framework CLI (cobra, urfave/cli)
- Third-party file processing library

---

## 📖 Panduan Pengerjaan

### Hari 1: Foundation
- Setup project structure dan models
- Implement `scanner/scanner.go` untuk directory scanning
- Test scanner secara manual

### Hari 2: Worker Pool
- Implement `worker/pool.go` dengan goroutines + channels
- Test dengan simple task (print filename)

### Hari 3: Processors
- Implement `WordCounter`, `Searcher`, `StatsCollector`
- Test masing-masing secara unit

### Hari 4: Commands & Output
- Connect worker pool dengan processors di setiap command
- Implement table output dan JSON output

### Hari 5: Polish
- Tambah context/timeout
- Error handling lengkap
- Test semua test cases
- Refactor dan cleanup

### Hari 6: Testing
- Tulis unit tests untuk processor
- Tulis benchmark tests
- Fix bugs dari test

---

## 📝 Deliverable

1. Source code di folder `file-processor/`
2. README.md dengan cara pakai + contoh output screenshot
3. Unit tests dengan coverage minimal 60%
4. Push ke GitHub

---

*Tips: Mulai dari scanner → worker pool → satu processor saja → pastikan berjalan → baru tambah fitur lain. Jangan coba implement semuanya sekaligus!*

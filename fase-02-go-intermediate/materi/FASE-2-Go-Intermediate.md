# 📘 FASE 2: Go Intermediate

> **Prasyarat:** Selesaikan Fase 1 + Project CLI Todo Manager  
> **Durasi:** 3–4 minggu  
> **Project Akhir:** File Processor CLI (lihat PRD di file terpisah)  
> **Tujuan:** Menguasai fitur Go yang membedakannya dari bahasa lain: Interface, Goroutines, Channels, dan pola-pola idiomatik Go

---

## 🗂️ Daftar Modul

| # | Modul | Topik |
|---|-------|-------|
| 2.1 | Interfaces | interface, duck typing, composition |
| 2.2 | Type Assertions & Type Switches | runtime type checking |
| 2.3 | Goroutines | concurrency basics, go keyword |
| 2.4 | Channels | komunikasi antar goroutine |
| 2.5 | Select Statement | multiplexing channels |
| 2.6 | Sync Package | Mutex, WaitGroup, Once |
| 2.7 | Context | timeout, cancellation, propagation |
| 2.8 | Generics (Go 1.18+) | type parameters, constraints |
| 2.9 | Testing Dasar | unit test, table-driven test |
| 2.10 | File I/O & OS | baca/tulis file, stdin/stdout |
| 2.11 | Working with JSON | encode, decode, custom marshaling |
| 2.12 | Time & Date | formatting, parsing, duration |
| 2.13 | Pola Idiomatik Go | functional options, builder, middleware chain |

---

## 📦 Modul 2.1 — Interfaces

Interface adalah **kontrak** yang mendefinisikan perilaku. Ini adalah fitur paling powerful di Go.

### Interface Basics

```go
package main

import (
    "fmt"
    "math"
)

// Mendefinisikan interface
type Shape interface {
    Area() float64
    Perimeter() float64
}

// Struct yang "mengimplementasikan" Shape
// (tanpa keyword 'implements' — ini duck typing!)
type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.Radius
}

type Triangle struct {
    A, B, C float64 // panjang sisi
}

func (t Triangle) Area() float64 {
    // Heron's formula
    s := (t.A + t.B + t.C) / 2
    return math.Sqrt(s * (s - t.A) * (s - t.B) * (s - t.C))
}

func (t Triangle) Perimeter() float64 {
    return t.A + t.B + t.C
}

// Fungsi yang menerima interface — polymorphism!
func printShapeInfo(s Shape) {
    fmt.Printf("Area: %.4f, Perimeter: %.4f\n", s.Area(), s.Perimeter())
}

func totalArea(shapes []Shape) float64 {
    total := 0.0
    for _, s := range shapes {
        total += s.Area()
    }
    return total
}

func main() {
    rect := Rectangle{Width: 5, Height: 3}
    circle := Circle{Radius: 4}
    triangle := Triangle{A: 3, B: 4, C: 5}

    // Semua bisa diperlakukan sebagai Shape
    printShapeInfo(rect)
    printShapeInfo(circle)
    printShapeInfo(triangle)

    // Slice of interface
    shapes := []Shape{rect, circle, triangle}
    fmt.Printf("\nTotal area: %.4f\n", totalArea(shapes))
}
```

### Interface Composition

```go
package main

import "fmt"

// Interface kecil-kecil, lalu di-compose
type Reader interface {
    Read() string
}

type Writer interface {
    Write(data string)
}

type Closer interface {
    Close() error
}

// Composition: interface yang embed interface lain
type ReadWriter interface {
    Reader
    Writer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}

// Implementasi
type File struct {
    name    string
    content string
    closed  bool
}

func (f *File) Read() string {
    if f.closed {
        return ""
    }
    return f.content
}

func (f *File) Write(data string) {
    if f.closed {
        return
    }
    f.content += data
}

func (f *File) Close() error {
    f.closed = true
    fmt.Printf("File %s closed\n", f.name)
    return nil
}

// Fungsi menggunakan interface compose
func copyData(src Reader, dst Writer) {
    data := src.Read()
    dst.Write(data)
}

func main() {
    file := &File{name: "test.txt"}
    file.Write("Hello, ")
    file.Write("World!")

    fmt.Println(file.Read()) // Hello, World!

    var rwc ReadWriteCloser = file // File implements all 3
    fmt.Println(rwc.Read())
    rwc.Close()
}
```

### Interface Kosong (Empty Interface)

```go
package main

import "fmt"

// interface{} (atau 'any' di Go 1.18+) — menerima semua tipe
func printAnything(v interface{}) {
    fmt.Printf("Value: %v, Type: %T\n", v, v)
}

// Atau dengan any (alias untuk interface{})
func describe(v any) string {
    return fmt.Sprintf("(%v, %T)", v, v)
}

// Map dengan nilai apapun (seperti JSON object)
func main() {
    printAnything(42)
    printAnything("hello")
    printAnything(true)
    printAnything([]int{1, 2, 3})

    // Map dengan nilai sembarang tipe
    data := map[string]any{
        "name":  "Alice",
        "age":   30,
        "score": 98.5,
        "tags":  []string{"go", "developer"},
    }

    for key, val := range data {
        fmt.Printf("%s: %v (%T)\n", key, val, val)
    }
}
```

### Interface Stringer dan Error

```go
package main

import "fmt"

// fmt.Stringer interface — dipakai saat object di-print
type Stringer interface {
    String() string
}

type Color int

const (
    Red Color = iota
    Green
    Blue
)

// Implementasi Stringer untuk Color
func (c Color) String() string {
    switch c {
    case Red:
        return "Red"
    case Green:
        return "Green"
    case Blue:
        return "Blue"
    default:
        return "Unknown"
    }
}

type Point struct {
    X, Y float64
}

func (p Point) String() string {
    return fmt.Sprintf("(%.2f, %.2f)", p.X, p.Y)
}

func main() {
    c := Green
    fmt.Println(c) // Green (bukan angka 1, karena Stringer diimplementasi)

    p := Point{X: 3.14, Y: 2.72}
    fmt.Println(p)              // (3.14, 2.72)
    fmt.Printf("Point: %v\n", p) // Point: (3.14, 2.72)
}
```

### 🏋️ Latihan 2.1

1. Buat interface `Animal` dengan method `Sound() string` dan `Name() string`
2. Implementasikan untuk `Dog`, `Cat`, `Bird`
3. Buat interface `Sorter` dengan method `Len() int`, `Less(i, j int) bool`, `Swap(i, j int)`
4. Implementasikan untuk `IntSlice` dan gunakan untuk sorting manual

---

## 📦 Modul 2.2 — Type Assertions & Type Switches

```go
package main

import "fmt"

type Animal interface {
    Sound() string
}

type Dog struct{ Name string }
type Cat struct{ Name string }

func (d Dog) Sound() string { return "Woof" }
func (c Cat) Sound() string { return "Meow" }
func (d Dog) Fetch() string { return d.Name + " fetches the ball!" }

func main() {
    var a Animal = Dog{Name: "Rex"}

    // Type assertion: i.(T)
    // Panics jika gagal!
    dog := a.(Dog)
    fmt.Println(dog.Fetch())

    // Safe type assertion — gunakan two-value form
    dog2, ok := a.(Dog)
    if ok {
        fmt.Println("It's a dog:", dog2.Name)
    }

    cat, ok := a.(Cat)
    if !ok {
        fmt.Println("Bukan kucing, cat zero value:", cat)
    }

    // Type switch — lebih idiomatik untuk multiple types
    animals := []Animal{
        Dog{Name: "Buddy"},
        Cat{Name: "Whiskers"},
        Dog{Name: "Max"},
    }

    for _, animal := range animals {
        switch v := animal.(type) {
        case Dog:
            fmt.Printf("Dog %s: %s, can fetch: %s\n", v.Name, v.Sound(), v.Fetch())
        case Cat:
            fmt.Printf("Cat %s: %s\n", v.Name, v.Sound())
        default:
            fmt.Printf("Unknown animal: %T\n", v)
        }
    }
}
```

---


### 🏋️ Latihan 2.2

1. Buat interface `Shape` dengan method `Area() float64` dan `Perimeter() float64`. Implementasikan untuk `Rectangle`, `Circle`, dan `Triangle`. Buat fungsi `LargestShape(shapes []Shape) Shape` yang mengembalikan shape dengan area terbesar.
2. Buat interface `Logger` dengan method `Log(level, message string)`. Implementasikan `ConsoleLogger` (print ke stdout) dan `SilentLogger` (tidak print apapun). Gunakan type switch untuk menangani kedua tipe.
3. Buat fungsi `Describe(i interface{}) string` yang mengembalikan deskripsi nilai berdasarkan tipenya (int, string, bool, slice, map, struct — gunakan type switch).


## 📦 Modul 2.3 — Goroutines

Goroutine adalah **lightweight thread** yang dikelola oleh Go runtime. Ini adalah inti dari concurrency di Go.

```go
package main

import (
    "fmt"
    "time"
)

func sayHello(name string) {
    for i := 0; i < 3; i++ {
        fmt.Printf("Hello from %s (%d)\n", name, i)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    // Jalankan sebagai goroutine dengan keyword 'go'
    go sayHello("Goroutine 1")
    go sayHello("Goroutine 2")

    // Main function juga goroutine (main goroutine)
    // Jika main selesai, semua goroutine lain dihentikan
    sayHello("Main")

    // Tanpa time.Sleep atau mekanisme sync, goroutine bisa tidak sempat jalan
    // Kita akan belajar cara yang benar dengan WaitGroup dan Channel
}
```

### Goroutine dengan Closure

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup

    // MASALAH: loop variable capture
    for i := 0; i < 5; i++ {
        wg.Add(1)
        // BUG: semua goroutine akan print 5!
        // go func() {
        //     defer wg.Done()
        //     fmt.Println(i) // capture variabel i yang berubah
        // }()

        // BENAR: pass sebagai argument
        go func(n int) {
            defer wg.Done()
            fmt.Printf("Worker %d\n", n)
        }(i) // i di-copy saat goroutine dibuat
    }

    wg.Wait() // Tunggu semua goroutine selesai
    fmt.Println("All workers done!")
}
```

### Goroutine dengan Worker Pool Pattern

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
    defer wg.Done()
    for job := range jobs {
        // Simulasi pekerjaan
        fmt.Printf("Worker %d processing job %d\n", id, job)
        time.Sleep(time.Duration(job*100) * time.Millisecond)
        results <- job * job // kirim hasil (job^2)
    }
}

func main() {
    const numJobs = 10
    const numWorkers = 3

    jobs := make(chan int, numJobs)
    results := make(chan int, numJobs)

    var wg sync.WaitGroup

    // Start workers
    for w := 1; w <= numWorkers; w++ {
        wg.Add(1)
        go worker(w, jobs, results, &wg)
    }

    // Kirim jobs
    for j := 1; j <= numJobs; j++ {
        jobs <- j
    }
    close(jobs) // Signal bahwa tidak ada job lagi

    // Tunggu semua worker selesai, lalu close results
    go func() {
        wg.Wait()
        close(results)
    }()

    // Kumpulkan hasil
    total := 0
    for result := range results {
        total += result
    }
    fmt.Printf("Total: %d\n", total)
}
```

---


### 🏋️ Latihan 2.3

1. Buat program yang menjalankan 5 goroutine, masing-masing menghitung kuadrat angka 1–5. Gunakan `sync.WaitGroup` untuk menunggu semua selesai, lalu print hasilnya.
2. Identifikasi dan perbaiki **bug** dalam kode berikut (loop variable capture):
   ```go
   for i := 0; i < 5; i++ {
       go func() { fmt.Println(i) }()
   }
   ```
3. Buat worker pool dengan 3 goroutine yang memproses slice of string (misalnya URL), mensimulasikan HTTP request dengan `time.Sleep` random, dan mengumpulkan hasilnya.


## 📦 Modul 2.4 — Channels

Channel adalah **pipa komunikasi** antar goroutine. "Don't communicate by sharing memory; share memory by communicating."

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Membuat channel
    ch := make(chan int)        // unbuffered channel
    bch := make(chan int, 5)    // buffered channel (kapasitas 5)

    // === UNBUFFERED CHANNEL ===
    // Sender BLOCK sampai receiver siap, dan sebaliknya
    go func() {
        fmt.Println("Goroutine: sending 42")
        ch <- 42 // send — block sampai ada receiver
        fmt.Println("Goroutine: sent!")
    }()

    time.Sleep(1 * time.Second) // simulasi kerja
    val := <-ch                 // receive — block sampai ada sender
    fmt.Println("Main: received", val)

    // === BUFFERED CHANNEL ===
    // Sender tidak block sampai buffer penuh
    bch <- 1
    bch <- 2
    bch <- 3
    // bch tidak block karena buffer belum penuh

    fmt.Println(<-bch) // 1
    fmt.Println(<-bch) // 2
    fmt.Println(<-bch) // 3
}
```

### Directional Channels

```go
package main

import "fmt"

// send-only channel: chan<- T
func produce(ch chan<- int) {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch) // PENTING: sender yang close channel
}

// receive-only channel: <-chan T
func consume(ch <-chan int) {
    // Range pada channel — loop sampai channel di-close
    for val := range ch {
        fmt.Println("Received:", val)
    }
}

func main() {
    ch := make(chan int, 5)

    go produce(ch)
    consume(ch)

    // Pipeline pattern
    naturals := generate(5)
    squares := square(naturals)

    for n := range squares {
        fmt.Println(n)
    }
}

// Pipeline stage 1: generate numbers
func generate(n int) <-chan int {
    out := make(chan int)
    go func() {
        for i := 1; i <= n; i++ {
            out <- i
        }
        close(out)
    }()
    return out
}

// Pipeline stage 2: square them
func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * n
        }
        close(out)
    }()
    return out
}
```

### Done Channel Pattern

```go
package main

import (
    "fmt"
    "time"
)

// Done channel untuk stop goroutine
func generator(done <-chan struct{}, nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            select {
            case out <- n:
            case <-done: // berhenti jika done signal diterima
                return
            }
        }
    }()
    return out
}

func main() {
    done := make(chan struct{})

    out := generator(done, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

    // Ambil hanya 3 pertama, lalu stop
    for i := 0; i < 3; i++ {
        fmt.Println(<-out)
    }
    close(done) // signal semua goroutine untuk berhenti
    time.Sleep(10 * time.Millisecond)
    fmt.Println("Done!")
}
```

### 🏋️ Latihan 2.4

1. Buat pipeline 3 stage: generate angka → filter genap → cetak kuadratnya
2. Buat fungsi `fanOut(in <-chan int, n int) []<-chan int` yang mendistribusikan satu channel ke n channel
3. Buat fungsi `merge(channels ...<-chan int) <-chan int` yang menggabungkan banyak channel jadi satu

---

## 📦 Modul 2.5 — Select Statement

`select` adalah `switch` untuk channels — wait untuk multiple channel operations.

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() {
        time.Sleep(1 * time.Second)
        ch1 <- "one"
    }()

    go func() {
        time.Sleep(2 * time.Second)
        ch2 <- "two"
    }()

    // Tunggu salah satu channel ready
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println("Received from ch1:", msg1)
        case msg2 := <-ch2:
            fmt.Println("Received from ch2:", msg2)
        }
    }

    // Select dengan default (non-blocking)
    ch3 := make(chan int, 1)
    select {
    case val := <-ch3:
        fmt.Println("Received:", val)
    default:
        fmt.Println("No value ready, moving on.") // langsung masuk sini
    }

    // Select dengan timeout
    ch4 := make(chan int)
    select {
    case val := <-ch4:
        fmt.Println("Received:", val)
    case <-time.After(500 * time.Millisecond):
        fmt.Println("Timeout! No value received.")
    }
}
```

### Real-World: Timeout Pattern

```go
package main

import (
    "context"
    "fmt"
    "time"
)

// Simulasi operasi yang lambat
func slowOperation(ctx context.Context) (string, error) {
    select {
    case <-time.After(2 * time.Second): // operasi selesai setelah 2 detik
        return "result", nil
    case <-ctx.Done(): // cancelled atau timeout
        return "", ctx.Err()
    }
}

func main() {
    // Timeout 1 detik
    ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel()

    result, err := slowOperation(ctx)
    if err != nil {
        fmt.Println("Error:", err) // context deadline exceeded
    } else {
        fmt.Println("Result:", result)
    }
}
```

---


### 🏋️ Latihan 2.5

1. Buat fungsi `timeout(ch <-chan int, d time.Duration) (int, bool)` yang mengembalikan nilai dari channel atau `false` jika timeout sebelum ada nilai.
2. Buat program yang memonitor dua channel secara bersamaan menggunakan select. Jika channel pertama menerima data, print "A: <nilai>". Jika channel kedua, print "B: <nilai>". Berhenti setelah total 10 pesan.
3. Implementasikan `nonBlockingSend(ch chan<- int, val int) bool` yang mencoba mengirim ke channel tanpa blocking (return false jika channel penuh).


## 📦 Modul 2.6 — Sync Package

```go
package main

import (
    "fmt"
    "sync"
)

// ===== sync.Mutex =====
// Melindungi shared data dari concurrent access

type SafeCounter struct {
    mu sync.Mutex
    v  map[string]int
}

func (c *SafeCounter) Inc(key string) {
    c.mu.Lock()         // lock sebelum akses data
    defer c.mu.Unlock() // unlock saat fungsi selesai
    c.v[key]++
}

func (c *SafeCounter) Value(key string) int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.v[key]
}

// ===== sync.RWMutex =====
// Read-write lock: banyak reader bisa concurrent, tapi write exclusive

type Cache struct {
    mu   sync.RWMutex
    data map[string]string
}

func (c *Cache) Set(key, value string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.data[key] = value
}

func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock() // Read lock — bisa bersamaan dengan reader lain
    defer c.mu.RUnlock()
    val, ok := c.data[key]
    return val, ok
}

// ===== sync.WaitGroup =====
// Tunggu sekumpulan goroutine selesai

func downloadFiles(urls []string) {
    var wg sync.WaitGroup

    for _, url := range urls {
        wg.Add(1) // tambah counter SEBELUM goroutine start
        go func(u string) {
            defer wg.Done() // kurangi counter saat goroutine selesai
            fmt.Printf("Downloading %s...\n", u)
            // simulasi download
        }(url)
    }

    wg.Wait() // block sampai counter = 0
    fmt.Println("All downloads complete!")
}

// ===== sync.Once =====
// Jalankan fungsi hanya sekali (untuk initialization)

type Singleton struct {
    data string
}

var (
    instance *Singleton
    once     sync.Once
)

func GetInstance() *Singleton {
    once.Do(func() {
        fmt.Println("Creating singleton...")
        instance = &Singleton{data: "initialized"}
    })
    return instance
}

func main() {
    // Counter
    counter := SafeCounter{v: make(map[string]int)}
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            counter.Inc("key")
        }()
    }
    wg.Wait()
    fmt.Println("Counter:", counter.Value("key")) // Harus 1000

    // Cache
    cache := Cache{data: make(map[string]string)}
    cache.Set("name", "Alice")
    if val, ok := cache.Get("name"); ok {
        fmt.Println("Cached:", val)
    }

    // Singleton
    s1 := GetInstance()
    s2 := GetInstance() // tidak akan print "Creating..." lagi
    fmt.Println(s1 == s2) // true — pointer yang sama
}
```

### sync.Pool — Object Reuse

```go
package main

import (
    "bytes"
    "fmt"
    "sync"
)

// Pool untuk buffer reuse — mengurangi GC pressure
var bufPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func processRequest(data string) string {
    // Ambil buffer dari pool
    buf := bufPool.Get().(*bytes.Buffer)
    defer func() {
        buf.Reset()
        bufPool.Put(buf) // kembalikan ke pool
    }()

    buf.WriteString("Processed: ")
    buf.WriteString(data)
    return buf.String()
}

func main() {
    fmt.Println(processRequest("Hello"))
    fmt.Println(processRequest("World"))
}
```

### 🏋️ Latihan 2.6

1. Buat thread-safe `Queue` dengan `sync.Mutex` yang support `Enqueue`, `Dequeue`, `Size`
2. Buat simple in-memory cache dengan TTL menggunakan `sync.RWMutex`
3. Jalankan 10 goroutine yang masing-masing increment counter 1000 kali, pastikan hasilnya 10000

---

## 📦 Modul 2.7 — Context

`context` adalah cara standar Go untuk membawa **deadline, cancellation signal, dan request-scoped values** melintasi batas API.

```go
package main

import (
    "context"
    "fmt"
    "time"
)

// ===== context.Background() dan context.TODO() =====
// Background: root context, tidak pernah cancelled
// TODO: placeholder saat belum tahu context mana yang akan dipakai

// ===== WithCancel =====
func withCancelExample() {
    ctx, cancel := context.WithCancel(context.Background())

    go func() {
        select {
        case <-ctx.Done():
            fmt.Println("goroutine: context cancelled:", ctx.Err())
        }
    }()

    time.Sleep(100 * time.Millisecond)
    cancel() // kirim signal cancellation
    time.Sleep(100 * time.Millisecond)
}

// ===== WithTimeout =====
func fetchData(ctx context.Context, url string) (string, error) {
    // Simulasi HTTP request yang butuh 2 detik
    done := make(chan string, 1)
    go func() {
        time.Sleep(2 * time.Second)
        done <- "data from " + url
    }()

    select {
    case result := <-done:
        return result, nil
    case <-ctx.Done():
        return "", fmt.Errorf("fetchData: %w", ctx.Err())
    }
}

// ===== WithDeadline =====
func withDeadlineExample() {
    deadline := time.Now().Add(500 * time.Millisecond)
    ctx, cancel := context.WithDeadline(context.Background(), deadline)
    defer cancel()

    result, err := fetchData(ctx, "https://api.example.com")
    if err != nil {
        fmt.Println("Error:", err)
    } else {
        fmt.Println("Result:", result)
    }
}

// ===== WithValue =====
// Untuk menyimpan request-scoped values (user ID, trace ID, dll)
type contextKey string

const (
    UserIDKey    contextKey = "userID"
    RequestIDKey contextKey = "requestID"
)

func middleware(ctx context.Context) context.Context {
    // Tambahkan values ke context
    ctx = context.WithValue(ctx, UserIDKey, 42)
    ctx = context.WithValue(ctx, RequestIDKey, "req-abc123")
    return ctx
}

func handler(ctx context.Context) {
    userID := ctx.Value(UserIDKey)
    requestID := ctx.Value(RequestIDKey)
    fmt.Printf("Handling request %v for user %v\n", requestID, userID)
}

// ===== Context propagation — best practice =====
// Context selalu jadi parameter PERTAMA dan tidak di-store di struct
func doWork(ctx context.Context, step string) error {
    select {
    case <-time.After(100 * time.Millisecond):
        fmt.Printf("Step %s done\n", step)
        return nil
    case <-ctx.Done():
        return fmt.Errorf("step %s: %w", step, ctx.Err())
    }
}

func doAllWork(ctx context.Context) error {
    // Context propagated ke setiap step
    if err := doWork(ctx, "1"); err != nil {
        return err
    }
    if err := doWork(ctx, "2"); err != nil {
        return err
    }
    if err := doWork(ctx, "3"); err != nil {
        return err
    }
    return nil
}

func main() {
    fmt.Println("=== WithCancel ===")
    withCancelExample()

    fmt.Println("\n=== WithDeadline ===")
    withDeadlineExample()

    fmt.Println("\n=== WithValue ===")
    ctx := context.Background()
    ctx = middleware(ctx)
    handler(ctx)

    fmt.Println("\n=== Context Propagation ===")
    ctx2, cancel := context.WithTimeout(context.Background(), 500*time.Millisecond)
    defer cancel()
    if err := doAllWork(ctx2); err != nil {
        fmt.Println("Error:", err)
    }
}
```

---


### 🏋️ Latihan 2.7

1. Buat fungsi `fetchWithTimeout(url string, timeout time.Duration) (string, error)` yang mensimulasikan HTTP fetch dan membatalkan jika melebihi timeout.
2. Buat fungsi `pipeline(ctx context.Context, stages ...func(context.Context, int) int) func(int) (int, error)` yang menjalankan serangkaian transformasi dan berhenti jika context dibatalkan.
3. Buat middleware function `withRequestID(ctx context.Context) context.Context` yang menyimpan random request ID ke context, dan `getRequestID(ctx context.Context) string` yang mengambilnya.


## 📦 Modul 2.8 — Generics (Go 1.18+)

Generics memungkinkan kamu menulis fungsi dan tipe yang bekerja dengan berbagai tipe data.

```go
package main

import (
    "fmt"
    "golang.org/x/exp/constraints"
)

// ===== Generic Functions =====

// Tanpa generics — harus tulis untuk setiap tipe
func sumInts(numbers []int) int {
    total := 0
    for _, n := range numbers {
        total += n
    }
    return total
}

// Dengan generics — satu fungsi untuk semua tipe numerik
// T adalah type parameter, constraints.Number adalah constraint-nya
func Sum[T constraints.Number](numbers []T) T {
    var total T
    for _, n := range numbers {
        total += n
    }
    return total
}

// Custom constraint
type Ordered interface {
    constraints.Integer | constraints.Float | ~string
}

func Min[T Ordered](a, b T) T {
    if a < b {
        return a
    }
    return b
}

func Max[T Ordered](a, b T) T {
    if a > b {
        return a
    }
    return b
}

// Generic Map (seperti map() di Python/JS)
func Map[T, U any](slice []T, fn func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// Generic Filter
func Filter[T any](slice []T, predicate func(T) bool) []T {
    var result []T
    for _, v := range slice {
        if predicate(v) {
            result = append(result, v)
        }
    }
    return result
}

// Generic Reduce
func Reduce[T, U any](slice []T, initial U, fn func(U, T) U) U {
    result := initial
    for _, v := range slice {
        result = fn(result, v)
    }
    return result
}

// ===== Generic Types =====

// Generic Stack
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    var zero T
    if len(s.items) == 0 {
        return zero, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}

func (s *Stack[T]) Peek() (T, bool) {
    var zero T
    if len(s.items) == 0 {
        return zero, false
    }
    return s.items[len(s.items)-1], true
}

func (s *Stack[T]) Size() int {
    return len(s.items)
}

// Generic Pair
type Pair[A, B any] struct {
    First  A
    Second B
}

func NewPair[A, B any](first A, second B) Pair[A, B] {
    return Pair[A, B]{First: first, Second: second}
}

func main() {
    // Generic Sum
    ints := []int{1, 2, 3, 4, 5}
    floats := []float64{1.1, 2.2, 3.3}
    fmt.Println(Sum(ints))    // 15
    fmt.Println(Sum(floats))  // 6.6

    // Generic Min/Max
    fmt.Println(Min(3, 7))      // 3
    fmt.Println(Min("abc", "xyz")) // abc
    fmt.Println(Max(3.14, 2.72)) // 3.14

    // Map/Filter/Reduce
    doubled := Map(ints, func(n int) int { return n * 2 })
    fmt.Println(doubled) // [2 4 6 8 10]

    evens := Filter(ints, func(n int) bool { return n%2 == 0 })
    fmt.Println(evens) // [2 4]

    sum := Reduce(ints, 0, func(acc, n int) int { return acc + n })
    fmt.Println(sum) // 15

    words := []string{"hello", "world", "go"}
    lengths := Map(words, func(s string) int { return len(s) })
    fmt.Println(lengths) // [5 5 2]

    // Generic Stack
    stack := &Stack[int]{}
    stack.Push(1)
    stack.Push(2)
    stack.Push(3)

    if top, ok := stack.Pop(); ok {
        fmt.Println("Popped:", top) // 3
    }
    fmt.Println("Size:", stack.Size()) // 2

    // String stack
    sStack := &Stack[string]{}
    sStack.Push("hello")
    sStack.Push("world")
    if top, ok := sStack.Pop(); ok {
        fmt.Println("Popped:", top) // world
    }

    // Generic Pair
    p := NewPair("Alice", 30)
    fmt.Printf("Name: %s, Age: %d\n", p.First, p.Second)
}
```

---


### 🏋️ Latihan 2.8

1. Buat fungsi generik `Contains[T comparable](slice []T, item T) bool` yang mengecek apakah sebuah item ada di dalam slice.
2. Buat generic type `Optional[T any]` (seperti `Option` di Rust / `Optional` di Java) dengan method `IsPresent() bool`, `Get() T`, dan `OrElse(defaultVal T) T`.
3. Buat fungsi generik `GroupBy[T any, K comparable](items []T, keyFn func(T) K) map[K][]T` yang mengelompokkan elemen berdasarkan key yang dikembalikan oleh `keyFn`.


## 📦 Modul 2.9 — Testing Dasar

```go
// File: math/math.go
package math

func Add(a, b int) int {
    return a + b
}

func Divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("division by zero")
    }
    return a / b, nil
}

func IsPrime(n int) bool {
    if n < 2 {
        return false
    }
    for i := 2; i*i <= n; i++ {
        if n%i == 0 {
            return false
        }
    }
    return true
}
```

```go
// File: math/math_test.go
package math

import (
    "testing"
)

// Test function: nama harus dimulai dengan Test
func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Add(2, 3) = %d; want 5", result)
    }
}

// Table-driven tests — idiom yang sangat direkomendasikan di Go
func TestAddTableDriven(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -2, -3, -5},
        {"mixed", -2, 3, 1},
        {"zeros", 0, 0, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, result, tt.expected)
            }
        })
    }
}

func TestDivide(t *testing.T) {
    t.Run("normal division", func(t *testing.T) {
        result, err := Divide(10, 2)
        if err != nil {
            t.Fatalf("unexpected error: %v", err)
        }
        if result != 5.0 {
            t.Errorf("Divide(10, 2) = %f; want 5.0", result)
        }
    })

    t.Run("division by zero", func(t *testing.T) {
        _, err := Divide(10, 0)
        if err == nil {
            t.Error("expected error for division by zero, got nil")
        }
    })
}

func TestIsPrime(t *testing.T) {
    tests := []struct {
        n        int
        expected bool
    }{
        {1, false},
        {2, true},
        {3, true},
        {4, false},
        {5, true},
        {17, true},
        {100, false},
        {97, true},
    }

    for _, tt := range tests {
        t.Run(fmt.Sprintf("IsPrime(%d)", tt.n), func(t *testing.T) {
            result := IsPrime(tt.n)
            if result != tt.expected {
                t.Errorf("IsPrime(%d) = %v; want %v", tt.n, result, tt.expected)
            }
        })
    }
}

// Benchmark test
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(2, 3)
    }
}

func BenchmarkIsPrime(b *testing.B) {
    for i := 0; i < b.N; i++ {
        IsPrime(97)
    }
}
```

Cara menjalankan test:
```bash
# Jalankan semua test
go test ./...

# Jalankan dengan verbose
go test -v ./...

# Jalankan test spesifik
go test -run TestAddTableDriven ./math/

# Jalankan benchmark
go test -bench=. ./math/

# Cek code coverage
go test -cover ./...
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

---


### 🏋️ Latihan 2.9

1. Tulis table-driven tests lengkap untuk fungsi `Fibonacci(n int) int`. Pastikan cover: n=0, n=1, n=2, n=10, n negatif (harus return error).
2. Buat benchmark untuk dua implementasi fungsi yang sama (misalnya mencari elemen di slice dengan linear search vs binary search). Jalankan `go test -bench=. -benchmem` dan bandingkan hasilnya.
3. Tulis test dengan `t.Parallel()` untuk memastikan beberapa test bisa berjalan bersamaan. Amati perbedaan waktu eksekusi.


## 📦 Modul 2.10 — File I/O & OS

```go
package main

import (
    "bufio"
    "fmt"
    "io"
    "os"
    "path/filepath"
    "strings"
)

// ===== Baca File =====

func readEntireFile(filename string) (string, error) {
    data, err := os.ReadFile(filename) // Simple, untuk file kecil
    if err != nil {
        return "", fmt.Errorf("readEntireFile: %w", err)
    }
    return string(data), nil
}

func readFileLineByLine(filename string) ([]string, error) {
    file, err := os.Open(filename)
    if err != nil {
        return nil, err
    }
    defer file.Close()

    var lines []string
    scanner := bufio.NewScanner(file)
    for scanner.Scan() {
        lines = append(lines, scanner.Text())
    }
    return lines, scanner.Err()
}

// ===== Tulis File =====

func writeFile(filename, content string) error {
    // os.WriteFile: create atau overwrite
    return os.WriteFile(filename, []byte(content), 0644)
}

func appendToFile(filename, content string) error {
    file, err := os.OpenFile(filename, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
    if err != nil {
        return err
    }
    defer file.Close()

    _, err = file.WriteString(content)
    return err
}

func writeWithBuffer(filename string, lines []string) error {
    file, err := os.Create(filename)
    if err != nil {
        return err
    }
    defer file.Close()

    writer := bufio.NewWriter(file)
    for _, line := range lines {
        fmt.Fprintln(writer, line)
    }
    return writer.Flush() // PENTING: flush buffer ke disk
}

// ===== File & Directory Operations =====

func fileOps() {
    // Cek apakah file/dir ada
    if _, err := os.Stat("myfile.txt"); os.IsNotExist(err) {
        fmt.Println("File tidak ada")
    }

    // Buat direktori
    os.MkdirAll("path/to/dir", 0755)

    // Rename/Move
    os.Rename("old.txt", "new.txt")

    // Hapus file
    os.Remove("file.txt")

    // Hapus direktori beserta isinya
    os.RemoveAll("path/to/dir")

    // Baca isi direktori
    entries, _ := os.ReadDir(".")
    for _, entry := range entries {
        fmt.Printf("%s (dir: %v)\n", entry.Name(), entry.IsDir())
    }

    // Path operations
    path := "/home/user/documents/file.txt"
    fmt.Println(filepath.Dir(path))      // /home/user/documents
    fmt.Println(filepath.Base(path))     // file.txt
    fmt.Println(filepath.Ext(path))      // .txt
    fmt.Println(filepath.Join("a", "b", "c")) // a/b/c
}

// ===== Environment Variables =====

func envExample() {
    // Get env var
    home := os.Getenv("HOME")
    fmt.Println("HOME:", home)

    // Get dengan default
    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }

    // Set env var
    os.Setenv("MY_VAR", "my_value")

    // Semua env vars
    for _, env := range os.Environ() {
        parts := strings.SplitN(env, "=", 2)
        fmt.Printf("%s = %s\n", parts[0], parts[1])
    }
}

// ===== stdin / stdout =====

func interactiveInput() {
    scanner := bufio.NewScanner(os.Stdin)

    fmt.Print("Masukkan nama: ")
    scanner.Scan()
    name := scanner.Text()

    fmt.Printf("Halo, %s!\n", name)
}

// ===== Copy File =====

func copyFile(src, dst string) error {
    source, err := os.Open(src)
    if err != nil {
        return err
    }
    defer source.Close()

    dest, err := os.Create(dst)
    if err != nil {
        return err
    }
    defer dest.Close()

    _, err = io.Copy(dest, source)
    return err
}

func main() {
    // Write
    writeFile("test.txt", "Hello, World!\nLine 2\nLine 3\n")

    // Read entire
    content, _ := readEntireFile("test.txt")
    fmt.Println(content)

    // Read line by line
    lines, _ := readFileLineByLine("test.txt")
    for i, line := range lines {
        fmt.Printf("%d: %s\n", i+1, line)
    }

    // Append
    appendToFile("test.txt", "Line 4\n")

    // Cleanup
    os.Remove("test.txt")

    fileOps()
}
```

---


### 🏋️ Latihan 2.10

1. Buat fungsi `CountWords(filename string) (map[string]int, error)` yang membaca file teks dan menghitung frekuensi setiap kata (case-insensitive).
2. Buat fungsi `MergeFiles(outputPath string, inputPaths ...string) error` yang menggabungkan isi beberapa file menjadi satu file output.
3. Buat CLI sederhana yang menerima path direktori sebagai argument, lalu menampilkan semua file beserta ukurannya, diurutkan dari terbesar ke terkecil.


## 📦 Modul 2.11 — Working with JSON

JSON sangat penting karena dipakai di REST API, config files, dan storage.

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

// ===== Struct Tags untuk JSON =====
type User struct {
    ID        int       `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    Password  string    `json:"-"`           // selalu di-skip
    Age       int       `json:"age,omitempty"` // skip jika zero value
    CreatedAt time.Time `json:"created_at"`
    Tags      []string  `json:"tags,omitempty"`
}

type APIResponse struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data"`
    Error   string      `json:"error,omitempty"`
    Meta    *Meta       `json:"meta,omitempty"`
}

type Meta struct {
    Total  int `json:"total"`
    Page   int `json:"page"`
    PerPage int `json:"per_page"`
}

func main() {
    // ===== Marshal (Go → JSON) =====
    user := User{
        ID:        1,
        Name:      "Alice",
        Email:     "alice@example.com",
        Password:  "secret",  // akan di-skip
        CreatedAt: time.Now(),
        Tags:      []string{"admin", "moderator"},
    }

    // json.Marshal — menghasilkan []byte
    data, err := json.Marshal(user)
    if err != nil {
        panic(err)
    }
    fmt.Println(string(data))

    // json.MarshalIndent — pretty print
    prettyData, _ := json.MarshalIndent(user, "", "  ")
    fmt.Println(string(prettyData))

    // ===== Unmarshal (JSON → Go) =====
    jsonStr := `{
        "id": 2,
        "name": "Bob",
        "email": "bob@example.com",
        "age": 25,
        "created_at": "2025-01-15T10:30:00Z",
        "tags": ["developer"]
    }`

    var user2 User
    if err := json.Unmarshal([]byte(jsonStr), &user2); err != nil {
        panic(err)
    }
    fmt.Printf("Parsed: %+v\n", user2)

    // ===== Dynamic JSON dengan map =====
    // Ketika struct tidak diketahui di compile time
    var rawData map[string]interface{}
    json.Unmarshal([]byte(jsonStr), &rawData)

    fmt.Println(rawData["name"])          // Bob
    fmt.Println(rawData["age"].(float64)) // 25 (number selalu float64)

    // ===== JSON Array =====
    usersJSON := `[{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]`
    var users []User
    json.Unmarshal([]byte(usersJSON), &users)
    fmt.Println("Users count:", len(users))

    // ===== APIResponse pattern =====
    response := APIResponse{
        Success: true,
        Data:    users,
        Meta: &Meta{
            Total:   100,
            Page:    1,
            PerPage: 10,
        },
    }
    respJSON, _ := json.MarshalIndent(response, "", "  ")
    fmt.Println(string(respJSON))
}
```

### Custom JSON Marshaling

```go
package main

import (
    "encoding/json"
    "fmt"
    "strings"
)

// Custom marshaling untuk tipe yang tidak standard
type Status int

const (
    StatusActive Status = iota + 1
    StatusInactive
    StatusBanned
)

// Custom MarshalJSON
func (s Status) MarshalJSON() ([]byte, error) {
    var str string
    switch s {
    case StatusActive:
        str = "active"
    case StatusInactive:
        str = "inactive"
    case StatusBanned:
        str = "banned"
    default:
        str = "unknown"
    }
    return json.Marshal(str)
}

// Custom UnmarshalJSON
func (s *Status) UnmarshalJSON(data []byte) error {
    var str string
    if err := json.Unmarshal(data, &str); err != nil {
        return err
    }
    switch strings.ToLower(str) {
    case "active":
        *s = StatusActive
    case "inactive":
        *s = StatusInactive
    case "banned":
        *s = StatusBanned
    default:
        return fmt.Errorf("unknown status: %s", str)
    }
    return nil
}

type Member struct {
    Name   string `json:"name"`
    Status Status `json:"status"`
}

func main() {
    m := Member{Name: "Alice", Status: StatusActive}

    data, _ := json.Marshal(m)
    fmt.Println(string(data)) // {"name":"Alice","status":"active"}

    var m2 Member
    json.Unmarshal([]byte(`{"name":"Bob","status":"banned"}`), &m2)
    fmt.Printf("%+v\n", m2) // {Name:Bob Status:3}
}
```

---


### 🏋️ Latihan 2.11

1. Buat struct `Config` dengan berbagai field (string, int, bool, slice, nested struct). Implementasikan `MarshalJSON` dan `UnmarshalJSON` custom sehingga field `Password` selalu di-mask menjadi `"***"` saat marshal.
2. Buat fungsi `MergeJSON(base, override []byte) ([]byte, error)` yang menggabungkan dua JSON object. Field dari `override` menggantikan field yang sama di `base`.
3. Buat fungsi `JSONDiff(a, b []byte) ([]string, error)` yang mengembalikan list field yang berbeda antara dua JSON object.


## 📦 Modul 2.12 — Time & Date

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // ===== Waktu saat ini =====
    now := time.Now()
    fmt.Println(now)             // 2025-01-15 10:30:00.123456 +0700 WIB

    // ===== Komponen waktu =====
    fmt.Println(now.Year())      // 2025
    fmt.Println(now.Month())     // January
    fmt.Println(now.Day())       // 15
    fmt.Println(now.Hour())      // 10
    fmt.Println(now.Minute())    // 30
    fmt.Println(now.Second())    // 0
    fmt.Println(now.Weekday())   // Wednesday

    // ===== Format waktu =====
    // Go menggunakan "reference time": Mon Jan 2 15:04:05 MST 2006
    // (Ini adalah waktu tertentu: 1/2 3:4:5 6 7)
    fmt.Println(now.Format("2006-01-02"))                    // 2025-01-15
    fmt.Println(now.Format("02/01/2006"))                    // 15/01/2025
    fmt.Println(now.Format("2006-01-02 15:04:05"))           // 2025-01-15 10:30:00
    fmt.Println(now.Format(time.RFC3339))                    // 2025-01-15T10:30:00+07:00
    fmt.Println(now.Format("Monday, 02 January 2006"))       // Wednesday, 15 January 2025

    // ===== Parse waktu =====
    dateStr := "2025-01-15 10:30:00"
    t, err := time.Parse("2006-01-02 15:04:05", dateStr)
    if err != nil {
        panic(err)
    }
    fmt.Println(t)

    // Parse dengan timezone
    loc, _ := time.LoadLocation("Asia/Jakarta")
    t2, _ := time.ParseInLocation("2006-01-02 15:04:05", dateStr, loc)
    fmt.Println(t2)

    // ===== Duration =====
    d1 := 5 * time.Second
    d2 := 2 * time.Minute
    d3 := 1 * time.Hour

    fmt.Println(d1)        // 5s
    fmt.Println(d1 + d2)   // 2m5s

    // Tambah/kurangi duration dari time
    tomorrow := now.Add(24 * time.Hour)
    lastWeek := now.Add(-7 * 24 * time.Hour)
    fmt.Println(tomorrow.Format("2006-01-02"))
    fmt.Println(lastWeek.Format("2006-01-02"))

    // Selisih dua waktu
    start := time.Now()
    time.Sleep(100 * time.Millisecond)
    elapsed := time.Since(start)
    fmt.Printf("Elapsed: %v\n", elapsed)

    diff := tomorrow.Sub(now)
    fmt.Printf("Diff: %v (%.0f hours)\n", diff, diff.Hours())

    // Perbandingan waktu
    fmt.Println(tomorrow.After(now))  // true
    fmt.Println(lastWeek.Before(now)) // true

    // Unix timestamp
    fmt.Println(now.Unix())      // detik sejak epoch
    fmt.Println(now.UnixMilli()) // milidetik
    fmt.Println(now.UnixNano())  // nanodetik

    // Dari unix timestamp
    t3 := time.Unix(1705300000, 0)
    fmt.Println(t3)

    // Timer dan Ticker
    timer := time.NewTimer(100 * time.Millisecond)
    <-timer.C // tunggu timer
    fmt.Println("Timer fired!")

    ticker := time.NewTicker(50 * time.Millisecond)
    for i := 0; i < 3; i++ {
        <-ticker.C
        fmt.Println("Tick!")
    }
    ticker.Stop()

    _ = d3
}
```

---


### 🏋️ Latihan 2.12

1. Buat fungsi `BusinessDaysBetween(start, end time.Time) int` yang menghitung jumlah hari kerja (Senin–Jumat) antara dua tanggal.
2. Buat struct `Scheduler` yang bisa menjadwalkan task untuk berjalan pada interval tertentu. Gunakan `time.Ticker` dan tambahkan method `Stop()` untuk menghentikannya.
3. Buat fungsi `ParseFlexibleDate(s string) (time.Time, error)` yang bisa mem-parse berbagai format tanggal: `"2006-01-02"`, `"02/01/2006"`, `"January 2, 2006"`, `"2 Jan 2006"`.


## 📦 Modul 2.13 — Pola Idiomatik Go

### Functional Options Pattern

```go
package main

import (
    "fmt"
    "time"
)

// Server configuration
type Server struct {
    host         string
    port         int
    timeout      time.Duration
    maxConns     int
    enableTLS    bool
}

// Option adalah fungsi yang memodifikasi Server
type Option func(*Server)

// Option constructors
func WithHost(host string) Option {
    return func(s *Server) {
        s.host = host
    }
}

func WithPort(port int) Option {
    return func(s *Server) {
        s.port = port
    }
}

func WithTimeout(d time.Duration) Option {
    return func(s *Server) {
        s.timeout = d
    }
}

func WithMaxConnections(n int) Option {
    return func(s *Server) {
        s.maxConns = n
    }
}

func WithTLS() Option {
    return func(s *Server) {
        s.enableTLS = true
    }
}

// Constructor dengan default values + options
func NewServer(opts ...Option) *Server {
    // Default values
    s := &Server{
        host:     "localhost",
        port:     8080,
        timeout:  30 * time.Second,
        maxConns: 100,
    }

    // Apply semua options
    for _, opt := range opts {
        opt(s)
    }

    return s
}

// ===== Middleware Pattern =====

type HandlerFunc func(string) string

type Middleware func(HandlerFunc) HandlerFunc

func Chain(middlewares ...Middleware) Middleware {
    return func(next HandlerFunc) HandlerFunc {
        for i := len(middlewares) - 1; i >= 0; i-- {
            next = middlewares[i](next)
        }
        return next
    }
}

func LoggingMiddleware(next HandlerFunc) HandlerFunc {
    return func(req string) string {
        fmt.Printf("[LOG] Request: %s\n", req)
        result := next(req)
        fmt.Printf("[LOG] Response: %s\n", result)
        return result
    }
}

func AuthMiddleware(next HandlerFunc) HandlerFunc {
    return func(req string) string {
        fmt.Println("[AUTH] Checking token...")
        return next(req)
    }
}

// ===== Error Handling Idioms =====

// Sentinel errors
var (
    ErrNotFound     = fmt.Errorf("not found")
    ErrUnauthorized = fmt.Errorf("unauthorized")
    ErrInternal     = fmt.Errorf("internal error")
)

// Error type dengan behaviour check
type TemporaryError struct {
    Err       error
    Temporary bool
}

func (e *TemporaryError) Error() string { return e.Err.Error() }
func (e *TemporaryError) IsTemporary() bool { return e.Temporary }

func main() {
    // Functional options
    s := NewServer(
        WithHost("0.0.0.0"),
        WithPort(9090),
        WithTimeout(60*time.Second),
        WithTLS(),
    )
    fmt.Printf("Server: %+v\n", s)

    // Default server
    defaultServer := NewServer()
    fmt.Printf("Default: %+v\n", defaultServer)

    // Middleware chain
    handler := func(req string) string {
        return "Hello, " + req + "!"
    }

    chain := Chain(LoggingMiddleware, AuthMiddleware)
    wrappedHandler := chain(handler)

    result := wrappedHandler("World")
    fmt.Println(result)
}
```

---


### 🏋️ Latihan 2.13

1. Implementasikan **Functional Options Pattern** untuk `Server` struct: buat `WithTimeout(d time.Duration)`, `WithMaxRetries(n int)`, `WithLogger(l *Logger)` options. Test bahwa default values berlaku saat options tidak di-provide.
2. Buat **Middleware Chain** untuk HTTP handler: implementasikan `Chain(handlers ...Middleware) Middleware` yang menerapkan middleware secara berurutan. Test chain: logging → auth → rate-limit → handler, verifikasi urutan eksekusi.
3. Buat **Pipeline Pattern**: function `Pipeline(stages ...func([]int) []int) func([]int) []int` yang compose multiple transformation functions. Gunakan untuk: filter angka genap → kalikan 2 → ambil 10 terbesar. Test dengan slice 100 angka random.


## 🎯 Review & Checkpoint Fase 2

### Konseptual
- [ ] Jelaskan apa itu interface di Go dan mengapa menggunakan duck typing
- [ ] Apa perbedaan goroutine dan thread OS?
- [ ] Apa perbedaan buffered dan unbuffered channel?
- [ ] Kapan menggunakan `sync.Mutex` vs `sync.RWMutex`?
- [ ] Mengapa context selalu jadi parameter pertama?
- [ ] Apa keuntungan functional options pattern?

### Praktis
- [ ] Bisa membuat dan compose interface
- [ ] Bisa membuat goroutine dan sinkronisasi dengan WaitGroup
- [ ] Bisa pakai channel untuk komunikasi antar goroutine
- [ ] Bisa implement context dengan timeout dan cancellation
- [ ] Bisa tulis table-driven tests
- [ ] Bisa encode/decode JSON

---

## 🎯 Project Akhir Fase 2

Kerjakan project **File Processor CLI** berdasarkan PRD di file:

**`FASE-2-PRD-File-Processor.md`**

Project ini akan menguji concurrency, channels, context, interface, dan file I/O secara bersamaan.

---

*Setelah selesai Fase 2, lanjut ke `FASE-3-Gin-REST-API.md`*

# 📘 FASE 1: Go Fundamentals

> **Prasyarat:** Punya pengalaman coding di bahasa lain (Python/JS/PHP)  
> **Durasi:** 2–3 minggu  
> **Project Akhir:** CLI Todo Manager (lihat PRD di file terpisah)  
> **Tujuan:** Memahami syntax, type system, dan idiom dasar Go

---

## 🗂️ Daftar Modul

| # | Modul | Topik |
|---|-------|-------|
| 1.1 | Setup & Workspace | Instalasi, GOPATH, go mod |
| 1.2 | Hello, Go! | Program pertama, struktur file |
| 1.3 | Variables & Types | var, :=, basic types |
| 1.4 | Constants & iota | const, iota, typed constants |
| 1.5 | Control Flow | if/else, switch, for |
| 1.6 | Functions | params, return, variadic, named return |
| 1.7 | Arrays & Slices | perbedaan array vs slice, operasi slice |
| 1.8 | Maps | CRUD map, nil map |
| 1.9 | Structs & Methods | struct, method, embedded struct |
| 1.10 | Pointers | pointer basics, pass by reference |
| 1.11 | Error Handling | error type, custom error, wrapping |
| 1.12 | Packages & Modules | go mod, import, visibility |
| 1.13 | Defer, Panic, Recover | cleanup pattern, recovery |

---

## 📦 Modul 1.1 — Setup & Workspace

### Instalasi Go

1. Download Go dari https://go.dev/dl/ (pilih versi terbaru, minimal 1.22)
2. Ikuti installer sesuai OS kamu
3. Verifikasi instalasi:

```bash
go version
# Output: go version go1.22.x linux/amd64 (atau sesuai OS)
```

### Mengenal GOPATH dan Go Modules

Di era modern Go (1.11+), kita **tidak lagi bergantung pada GOPATH** untuk project kita sendiri. Kita menggunakan **Go Modules**.

```bash
# Cek GOPATH (opsional, tapi baik untuk tahu)
go env GOPATH

# Cek semua environment Go
go env
```

### Membuat Project Pertama

```bash
# Buat folder
mkdir hello-go
cd hello-go

# Inisialisasi Go module
# Format: go mod init <module-name>
# Konvensi: github.com/username/project-name
go mod init github.com/kamu/hello-go
```

Ini akan membuat file `go.mod`:
```
module github.com/kamu/hello-go

go 1.22
```

### Struktur Workspace yang Direkomendasikan

```
~/projects/
├── hello-go/           ← Project fase 1 pertama
│   ├── go.mod
│   └── main.go
│
├── cli-todo/           ← Project akhir fase 1
│   ├── go.mod
│   ├── main.go
│   └── ...
```

### Setup VS Code

Setelah install extension Go (by Google):
1. Buka VS Code
2. `Ctrl+Shift+P` → ketik "Go: Install/Update Tools"
3. Pilih semua → Install
4. Restart VS Code

---

## 📦 Modul 1.2 — Hello, Go!

### Program Pertama

Buat file `main.go`:

```go
package main  // ← Setiap file Go dimulai dengan deklarasi package

import "fmt"  // ← Import package fmt (format, print, scan)

func main() {  // ← Titik masuk program (entry point)
    fmt.Println("Hello, Go!")  // ← Print dengan newline
    fmt.Print("Tanpa newline ")
    fmt.Printf("Nama: %s, Umur: %d\n", "Budi", 25)  // ← Format string
}
```

Jalankan:
```bash
go run main.go
# Hello, Go!
# Tanpa newline Nama: Budi, Umur: 25
```

### Memahami Struktur File Go

```go
package main          // [1] Deklarasi package — WAJIB di setiap file

import (              // [2] Import packages
    "fmt"
    "math"
)

func main() {         // [3] Fungsi main — entry point HANYA di package main
    // kode di sini
}
```

**Aturan penting:**
- Setiap file Go **wajib** punya `package`
- Hanya ada **satu** `func main()` per program
- Import yang tidak dipakai = **error** (Go sangat strict!)
- Nama file bebas, tapi `main.go` adalah konvensi

### Format String di Go

```go
package main

import "fmt"

func main() {
    name := "Andi"
    age := 22
    score := 98.5
    isActive := true

    // %s = string
    fmt.Printf("Nama: %s\n", name)
    
    // %d = integer (decimal)
    fmt.Printf("Umur: %d\n", age)
    
    // %f = float (default 6 desimal)
    fmt.Printf("Nilai: %f\n", score)
    
    // %.2f = float 2 desimal
    fmt.Printf("Nilai: %.2f\n", score)
    
    // %t = boolean
    fmt.Printf("Aktif: %t\n", isActive)
    
    // %v = any value (versatile, paling sering dipakai saat debugging)
    fmt.Printf("Semua: %v %v %v %v\n", name, age, score, isActive)
    
    // %T = tipe data
    fmt.Printf("Tipe age: %T\n", age)  // Output: int
    
    // Sprintf = format ke string (tidak print)
    message := fmt.Sprintf("Halo, %s! Umurmu %d tahun.", name, age)
    fmt.Println(message)
}
```

### 🏋️ Latihan 1.2

Buat program yang menampilkan biodata dirimu sendiri menggunakan:
- `fmt.Println` untuk nama
- `fmt.Printf` untuk umur dan kota
- `fmt.Sprintf` untuk membuat kalimat lengkap lalu print

---

## 📦 Modul 1.3 — Variables & Types

### Cara Deklarasi Variable

```go
package main

import "fmt"

func main() {
    // Cara 1: Deklarasi eksplisit dengan tipe
    var nama string = "Budi"
    var umur int = 25
    
    // Cara 2: Deklarasi eksplisit, tipe diinfer
    var kota = "Jakarta"  // Go tahu ini string
    
    // Cara 3: Short declaration (PALING SERING DIPAKAI dalam fungsi)
    tinggi := 175  // Go tahu ini int
    berat := 70.5  // Go tahu ini float64
    
    // Cara 4: Multiple declaration
    var (
        firstName string = "Budi"
        lastName  string = "Santoso"
        age       int    = 25
    )
    
    // Cara 5: Multiple short declaration
    x, y, z := 1, 2, 3
    
    fmt.Println(nama, umur, kota, tinggi, berat)
    fmt.Println(firstName, lastName, age)
    fmt.Println(x, y, z)
}
```

**Kapan pakai `var` vs `:=`?**
- `:=` hanya bisa di dalam fungsi
- `var` bisa di dalam maupun luar fungsi (package level)
- Gunakan `:=` sebisa mungkin di dalam fungsi

### Zero Values

Di Go, setiap tipe punya **zero value** (nilai default saat tidak diinisialisasi):

```go
package main

import "fmt"

func main() {
    var i int        // zero value: 0
    var f float64    // zero value: 0.0
    var b bool       // zero value: false
    var s string     // zero value: "" (string kosong)
    
    fmt.Printf("int: %v\n", i)        // 0
    fmt.Printf("float64: %v\n", f)    // 0
    fmt.Printf("bool: %v\n", b)       // false
    fmt.Printf("string: %q\n", s)     // "" (%q untuk lihat quotes)
}
```

### Tipe Data Dasar

```go
package main

import "fmt"

func main() {
    // Integer
    var a int8   = 127           // -128 sampai 127
    var b int16  = 32767         // -32768 sampai 32767
    var c int32  = 2147483647    // alias: rune
    var d int64  = 9223372036854775807
    var e int    = 100           // platform-dependent (32 atau 64 bit)
    
    // Unsigned Integer (tidak bisa negatif)
    var ua uint8  = 255          // alias: byte
    var ub uint16 = 65535
    var uc uint   = 100
    
    // Float
    var f32 float32 = 3.14       // 6-7 digit presisi
    var f64 float64 = 3.14159265 // 15-16 digit presisi (DEFAULT)
    
    // Boolean
    var isGo bool = true
    
    // String
    var greeting string = "Hello, Go!"
    
    // Byte dan Rune
    var byteVal byte = 'A'       // uint8, untuk ASCII
    var runeVal rune = '🐹'     // int32, untuk Unicode
    
    fmt.Println(a, b, c, d, e)
    fmt.Println(ua, ub, uc)
    fmt.Println(f32, f64)
    fmt.Println(isGo)
    fmt.Println(greeting)
    fmt.Printf("byte: %c (%d)\n", byteVal, byteVal)  // A (65)
    fmt.Printf("rune: %c (%d)\n", runeVal, runeVal)
}
```

**Tipe yang paling sering dipakai:**
- `int`, `int64` — untuk angka bulat
- `float64` — untuk angka desimal
- `string` — untuk teks
- `bool` — untuk true/false
- `byte` (uint8) — untuk data biner / karakter ASCII
- `rune` (int32) — untuk karakter Unicode

### Type Conversion

Go **tidak** melakukan konversi otomatis. Kamu harus explicit:

```go
package main

import "fmt"

func main() {
    var i int = 42
    var f float64 = float64(i)   // int → float64
    var u uint = uint(f)         // float64 → uint
    
    fmt.Println(i, f, u)
    
    // String conversion
    // int ke string menggunakan strconv, BUKAN string()
    // string(65) = "A" (itu konversi ke karakter, bukan "65")
    
    // Cara yang benar:
    import_note := "gunakan package strconv untuk konversi string-angka"
    fmt.Println(import_note)
    
    // Kita akan bahas strconv di modul selanjutnya
}
```

### 🏋️ Latihan 1.3

1. Deklarasikan variabel untuk data produk: nama (string), harga (float64), stok (int), tersedia (bool)
2. Print semua variabel dengan format yang rapi menggunakan `fmt.Printf`
3. Coba type conversion dari `int` ke `float64` dan sebaliknya

---

## 📦 Modul 1.4 — Constants & iota

### Konstanta Dasar

```go
package main

import "fmt"

// Package-level constants
const Pi = 3.14159
const AppName = "MyApp"
const MaxRetry = 3

// Typed constants
const MaxInt int = 100

func main() {
    // Local constant
    const Greeting = "Halo"
    
    fmt.Println(Pi, AppName, MaxRetry)
    fmt.Println(Greeting)
    
    // Constants tidak bisa diubah
    // Pi = 3.14  // ERROR: cannot assign to Pi
}
```

### iota — Auto-incrementing Constants

`iota` adalah fitur Go untuk membuat konstanta bernilai berurutan (enum):

```go
package main

import "fmt"

// Contoh 1: Status Dasar
type Status int

const (
    StatusPending Status = iota  // 0
    StatusActive                  // 1
    StatusInactive                // 2
    StatusDeleted                 // 3
)

// Contoh 2: Mulai dari 1
type Priority int

const (
    PriorityLow Priority = iota + 1  // 1
    PriorityMedium                     // 2
    PriorityHigh                       // 3
)

// Contoh 3: Bit flags (powers of 2)
type Permission uint

const (
    PermRead    Permission = 1 << iota  // 1  (001)
    PermWrite                            // 2  (010)
    PermExecute                          // 4  (100)
)

// Contoh 4: Lewati nilai dengan _
type Day int

const (
    _        Day = iota  // 0 dilewati
    Monday               // 1
    Tuesday              // 2
    Wednesday            // 3
    Thursday             // 4
    Friday               // 5
    Saturday             // 6
    Sunday               // 7
)

func main() {
    fmt.Println("=== Status ===")
    fmt.Println(StatusPending, StatusActive, StatusInactive, StatusDeleted)
    
    fmt.Println("\n=== Priority ===")
    fmt.Println(PriorityLow, PriorityMedium, PriorityHigh)
    
    fmt.Println("\n=== Permission ===")
    fmt.Println(PermRead, PermWrite, PermExecute)
    
    // Kombinasi permissions menggunakan bitwise OR
    myPerm := PermRead | PermWrite  // 3 (011)
    fmt.Printf("My permissions: %b\n", myPerm)
    
    // Cek permission dengan bitwise AND
    canRead := myPerm & PermRead != 0
    canExecute := myPerm & PermExecute != 0
    fmt.Printf("Can read: %v, Can execute: %v\n", canRead, canExecute)
    
    fmt.Println("\n=== Day ===")
    fmt.Println(Monday, Friday, Sunday)
}
```

### 🏋️ Latihan 1.4

Buat konstanta untuk:
1. Level user: Guest(0), Member(1), Admin(2), SuperAdmin(3) menggunakan iota
2. Ukuran file: KB, MB, GB menggunakan iota dengan perkalian 1024
3. Print semua nilai konstanta tersebut

---

## 📦 Modul 1.5 — Control Flow

### if / else

```go
package main

import "fmt"

func main() {
    age := 20

    // if-else dasar
    if age >= 18 {
        fmt.Println("Dewasa")
    } else {
        fmt.Println("Anak-anak")
    }
    
    // if-else if-else
    score := 85
    if score >= 90 {
        fmt.Println("A")
    } else if score >= 80 {
        fmt.Println("B")
    } else if score >= 70 {
        fmt.Println("C")
    } else {
        fmt.Println("D")
    }
    
    // if dengan initialization statement (sangat idiomatik di Go!)
    // variabel yang dideklarsikan di sini HANYA hidup di dalam if block
    if n := 10; n%2 == 0 {
        fmt.Printf("%d adalah genap\n", n)
    } else {
        fmt.Printf("%d adalah ganjil\n", n)
    }
    // fmt.Println(n)  // ERROR: n tidak ada di sini
}
```

### switch

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Switch dasar
    day := "Monday"
    switch day {
    case "Monday", "Tuesday", "Wednesday", "Thursday", "Friday":
        fmt.Println("Weekday")
    case "Saturday", "Sunday":
        fmt.Println("Weekend")
    default:
        fmt.Println("Unknown day")
    }
    
    // Switch dengan kondisi (seperti if-else if)
    score := 85
    switch {
    case score >= 90:
        fmt.Println("Excellent!")
    case score >= 80:
        fmt.Println("Good!")
    case score >= 70:
        fmt.Println("Average")
    default:
        fmt.Println("Need improvement")
    }
    
    // Switch dengan initialization
    switch t := time.Now().Weekday(); t {
    case time.Saturday, time.Sunday:
        fmt.Println("It's the weekend!")
    default:
        fmt.Println("It's a weekday.")
    }
    
    // Type switch (sangat berguna untuk interface)
    var i interface{} = "hello"
    switch v := i.(type) {
    case int:
        fmt.Printf("int: %v\n", v)
    case string:
        fmt.Printf("string: %v\n", v)
    case bool:
        fmt.Printf("bool: %v\n", v)
    default:
        fmt.Printf("unknown: %T\n", v)
    }
    
    // fallthrough (jarang dipakai, tapi perlu tahu)
    n := 2
    switch n {
    case 1:
        fmt.Println("Satu")
        fallthrough
    case 2:
        fmt.Println("Dua")
        fallthrough
    case 3:
        fmt.Println("Tiga")
    case 4:
        fmt.Println("Empat")  // ini tidak dieksekusi
    }
    // Output: Dua, Tiga
}
```

### for — Satu-satunya Loop di Go

Go hanya punya **satu** kata kunci loop: `for`. Tapi bisa dipakai dalam berbagai bentuk:

```go
package main

import "fmt"

func main() {
    // Bentuk 1: Classic for (seperti C/Java)
    for i := 0; i < 5; i++ {
        fmt.Printf("i = %d\n", i)
    }
    
    // Bentuk 2: While loop (hanya kondisi)
    n := 1
    for n < 100 {
        n *= 2
    }
    fmt.Println(n)  // 128
    
    // Bentuk 3: Infinite loop
    count := 0
    for {
        count++
        if count >= 3 {
            break  // keluar dari loop
        }
        fmt.Println("Loop:", count)
    }
    
    // Bentuk 4: for range — untuk iterasi koleksi
    fruits := []string{"apple", "banana", "cherry"}
    for index, value := range fruits {
        fmt.Printf("[%d] %s\n", index, value)
    }
    
    // Abaikan index dengan _
    for _, fruit := range fruits {
        fmt.Println(fruit)
    }
    
    // Hanya butuh index
    for i := range fruits {
        fmt.Println(i)
    }
    
    // Range pada string (iterasi per rune/karakter)
    for i, ch := range "Hello" {
        fmt.Printf("%d: %c\n", i, ch)
    }
    
    // Range pada map
    ages := map[string]int{"Alice": 30, "Bob": 25, "Charlie": 35}
    for name, age := range ages {
        fmt.Printf("%s: %d\n", name, age)
    }
    
    // continue — skip iterasi saat ini
    for i := 0; i < 10; i++ {
        if i%2 == 0 {
            continue  // skip angka genap
        }
        fmt.Println(i)  // print 1, 3, 5, 7, 9
    }
}
```

### 🏋️ Latihan 1.5

1. Buat program yang cek apakah sebuah angka adalah **bilangan prima**
2. Buat program yang print **FizzBuzz** (1-30: angka kelipatan 3 cetak "Fizz", kelipatan 5 cetak "Buzz", kelipatan 3&5 cetak "FizzBuzz")
3. Buat program yang menghitung **faktorial** sebuah angka menggunakan for loop

---

## 📦 Modul 1.6 — Functions

### Fungsi Dasar

```go
package main

import "fmt"

// Fungsi tanpa return value
func greet(name string) {
    fmt.Printf("Halo, %s!\n", name)
}

// Fungsi dengan satu return value
func add(a, b int) int {
    return a + b
}

// Fungsi dengan multiple return values (FITUR KHAS GO!)
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("tidak bisa bagi dengan nol")
    }
    return a / b, nil  // nil = tidak ada error
}

// Fungsi dengan named return values
func minMax(numbers []int) (min, max int) {
    min = numbers[0]
    max = numbers[0]
    for _, n := range numbers {
        if n < min {
            min = n
        }
        if n > max {
            max = n
        }
    }
    return  // "naked return" — return min dan max secara implisit
}

func main() {
    // Memanggil fungsi
    greet("Andi")
    
    result := add(3, 5)
    fmt.Println("3 + 5 =", result)
    
    // Multiple return values
    quotient, err := divide(10, 3)
    if err != nil {
        fmt.Println("Error:", err)
    } else {
        fmt.Printf("10 / 3 = %.4f\n", quotient)
    }
    
    // Coba bagi dengan nol
    _, err = divide(5, 0)  // _ artinya abaikan nilai pertama
    if err != nil {
        fmt.Println("Error:", err)
    }
    
    // Named return
    nums := []int{3, 1, 4, 1, 5, 9, 2, 6}
    min, max := minMax(nums)
    fmt.Printf("Min: %d, Max: %d\n", min, max)
}
```

### Variadic Functions

```go
package main

import "fmt"

// Variadic: menerima jumlah argumen yang tidak terbatas
func sum(numbers ...int) int {
    total := 0
    for _, n := range numbers {
        total += n
    }
    return total
}

// Mix dengan parameter biasa (variadic harus terakhir)
func logger(level string, messages ...string) {
    fmt.Printf("[%s] ", level)
    for _, msg := range messages {
        fmt.Printf("%s ", msg)
    }
    fmt.Println()
}

func main() {
    fmt.Println(sum(1, 2, 3))           // 6
    fmt.Println(sum(1, 2, 3, 4, 5))    // 15
    
    // Spread operator — expand slice menjadi variadic args
    nums := []int{10, 20, 30, 40}
    fmt.Println(sum(nums...))           // 100
    
    logger("INFO", "Server started")
    logger("ERROR", "Connection failed", "retrying...")
}
```

### Functions sebagai First-Class Citizens

```go
package main

import (
    "fmt"
    "strings"
)

// Fungsi sebagai parameter (Higher-Order Function)
func apply(numbers []int, fn func(int) int) []int {
    result := make([]int, len(numbers))
    for i, n := range numbers {
        result[i] = fn(n)
    }
    return result
}

// Fungsi yang mengembalikan fungsi (Closure)
func multiplier(factor int) func(int) int {
    // 'factor' di-"capture" oleh closure
    return func(n int) int {
        return n * factor
    }
}

// Fungsi yang mengembalikan fungsi dengan state
func counter() func() int {
    count := 0  // state yang di-capture
    return func() int {
        count++
        return count
    }
}

func main() {
    nums := []int{1, 2, 3, 4, 5}
    
    // Anonymous function (lambda)
    doubled := apply(nums, func(n int) int {
        return n * 2
    })
    fmt.Println(doubled)  // [2 4 6 8 10]
    
    // Closure sebagai multiplier
    triple := multiplier(3)
    fmt.Println(triple(5))   // 15
    fmt.Println(triple(10))  // 30
    
    // Counter dengan state
    c := counter()
    fmt.Println(c())  // 1
    fmt.Println(c())  // 2
    fmt.Println(c())  // 3
    
    // Immediately Invoked Function (IIFE)
    result := func(a, b int) int {
        return a + b
    }(10, 20)
    fmt.Println(result)  // 30
    
    // Map/Filter/Reduce style
    words := []string{"hello", "world", "go", "is", "awesome"}
    upper := apply(
        []int{0, 1, 2, 3, 4},
        func(i int) int {
            fmt.Println(strings.ToUpper(words[i]))
            return i
        },
    )
    _ = upper
}
```

### 🏋️ Latihan 1.6

1. Buat fungsi `calculate(a, b float64, op string) (float64, error)` yang menerima dua angka dan operasi ("+", "-", "*", "/") dan mengembalikan hasilnya
2. Buat fungsi `filter(numbers []int, predicate func(int) bool) []int` yang mengembalikan slice berisi angka yang memenuhi kondisi
3. Gunakan fungsi `filter` untuk mendapatkan angka genap dan ganjil dari sebuah slice

---

## 📦 Modul 1.7 — Arrays & Slices

### Array

Array di Go memiliki **ukuran tetap** dan merupakan value type:

```go
package main

import "fmt"

func main() {
    // Deklarasi array
    var scores [5]int  // [0 0 0 0 0]
    
    // Array dengan inisialisasi
    names := [3]string{"Alice", "Bob", "Charlie"}
    
    // Array dengan ... (Go hitung otomatis)
    primes := [...]int{2, 3, 5, 7, 11}
    
    // Akses elemen
    scores[0] = 90
    scores[1] = 85
    fmt.Println(scores)   // [90 85 0 0 0]
    fmt.Println(names[1]) // Bob
    
    // Panjang array
    fmt.Println(len(primes))  // 5
    
    // Array adalah VALUE TYPE — copy saat di-assign
    a := [3]int{1, 2, 3}
    b := a        // b adalah COPY dari a
    b[0] = 999
    fmt.Println(a)  // [1 2 3]  ← tidak berubah!
    fmt.Println(b)  // [999 2 3]
    
    // Multi-dimensional array
    matrix := [2][3]int{
        {1, 2, 3},
        {4, 5, 6},
    }
    fmt.Println(matrix[1][2])  // 6
}
```

### Slices — Yang Paling Sering Dipakai

Slice adalah **view dinamis** ke array. Ini yang mayoritas kamu pakai:

```go
package main

import "fmt"

func main() {
    // Deklarasi slice
    var s []int           // nil slice (panjang 0, kapasitas 0)
    fmt.Println(s == nil) // true
    
    // Membuat slice dengan literal
    fruits := []string{"apple", "banana", "cherry", "date", "elderberry"}
    
    // Membuat slice dengan make(type, length, capacity)
    nums := make([]int, 5)     // [0 0 0 0 0], len=5, cap=5
    nums2 := make([]int, 3, 10) // [0 0 0], len=3, cap=10
    
    fmt.Println(len(nums), cap(nums))    // 5 5
    fmt.Println(len(nums2), cap(nums2))  // 3 10
    
    // Slicing (mengambil bagian dari slice)
    // s[low:high] → dari index low sampai high-1
    fmt.Println(fruits[1:3])   // [banana cherry]
    fmt.Println(fruits[:3])    // [apple banana cherry]  (dari awal)
    fmt.Println(fruits[2:])    // [cherry date elderberry]  (sampai akhir)
    fmt.Println(fruits[:])     // semua elemen
    
    // append — menambah elemen
    s = append(s, 1, 2, 3)
    fmt.Println(s)  // [1 2 3]
    
    // append slice ke slice
    more := []int{4, 5, 6}
    s = append(s, more...)  // spread operator
    fmt.Println(s)  // [1 2 3 4 5 6]
    
    // PENTING: Slice berbagi underlying array!
    original := []int{1, 2, 3, 4, 5}
    slice1 := original[1:4]  // [2 3 4]
    slice1[0] = 999
    fmt.Println(original)  // [1 999 3 4 5] ← BERUBAH!
    fmt.Println(slice1)    // [999 3 4]
    
    // Membuat copy independen
    independent := make([]int, len(slice1))
    copy(independent, slice1)  // copy(dst, src)
    independent[0] = 111
    fmt.Println(slice1)      // tidak berubah
    fmt.Println(independent) // [111 3 4]
}
```

### Operasi Slice Umum

```go
package main

import (
    "fmt"
    "sort"
)

func main() {
    // Menghapus elemen (tidak ada delete bawaan untuk slice)
    s := []int{1, 2, 3, 4, 5}
    
    // Hapus elemen di index 2
    i := 2
    s = append(s[:i], s[i+1:]...)
    fmt.Println(s)  // [1 2 4 5]
    
    // Insert elemen di index tertentu
    s2 := []int{1, 2, 4, 5}
    idx := 2
    s2 = append(s2[:idx], append([]int{3}, s2[idx:]...)...)
    fmt.Println(s2)  // [1 2 3 4 5]
    
    // Sort slice
    unsorted := []int{5, 2, 8, 1, 9, 3}
    sort.Ints(unsorted)
    fmt.Println(unsorted)  // [1 2 3 5 8 9]
    
    strings := []string{"banana", "apple", "cherry"}
    sort.Strings(strings)
    fmt.Println(strings)  // [apple banana cherry]
    
    // Cek apakah slice mengandung elemen (tidak ada built-in contains)
    // Di Go 1.21+ ada slices.Contains
    target := 3
    found := false
    for _, v := range unsorted {
        if v == target {
            found = true
            break
        }
    }
    fmt.Println("Found 3:", found)
    
    // 2D Slice
    matrix := [][]int{
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9},
    }
    fmt.Println(matrix[1][1])  // 5
}
```

### 🏋️ Latihan 1.7

1. Buat fungsi `reverseSlice(s []int) []int` yang membalik urutan elemen
2. Buat fungsi `removeDuplicates(s []int) []int` yang menghapus duplikat
3. Buat fungsi yang menghitung rata-rata dari slice of float64

---

## 📦 Modul 1.8 — Maps

### Map Dasar

```go
package main

import "fmt"

func main() {
    // Deklarasi map
    var m map[string]int  // nil map — TIDAK bisa diisi!
    fmt.Println(m == nil) // true
    
    // Membuat map dengan make
    ages := make(map[string]int)
    
    // Membuat map dengan literal
    capitals := map[string]string{
        "Indonesia": "Jakarta",
        "Japan":     "Tokyo",
        "France":    "Paris",
    }
    
    // Menambah / update elemen
    ages["Alice"] = 30
    ages["Bob"] = 25
    ages["Charlie"] = 35
    
    // Membaca elemen
    fmt.Println(ages["Alice"])   // 30
    fmt.Println(ages["Unknown"]) // 0 (zero value, bukan error!)
    
    // Cek apakah key ada (two-value assignment)
    age, ok := ages["Alice"]
    if ok {
        fmt.Printf("Alice's age: %d\n", age)
    }
    
    age2, ok2 := ages["David"]
    if !ok2 {
        fmt.Printf("David tidak ditemukan (zero value: %d)\n", age2)
    }
    
    // Menghapus elemen
    delete(ages, "Bob")
    fmt.Println(ages)  // map tanpa Bob
    
    // Panjang map
    fmt.Println(len(capitals))  // 3
    
    // Iterasi map (urutan TIDAK dijamin)
    for country, capital := range capitals {
        fmt.Printf("%s: %s\n", country, capital)
    }
    
    // Map adalah REFERENCE TYPE
    m1 := map[string]int{"a": 1, "b": 2}
    m2 := m1        // m2 menunjuk ke map yang SAMA
    m2["a"] = 999
    fmt.Println(m1["a"])  // 999 ← berubah!
    
    // Nested map
    users := map[string]map[string]string{
        "user1": {
            "name":  "Alice",
            "email": "alice@example.com",
        },
        "user2": {
            "name":  "Bob",
            "email": "bob@example.com",
        },
    }
    fmt.Println(users["user1"]["email"])  // alice@example.com
}
```

### Map sebagai Set

```go
package main

import "fmt"

func main() {
    // Go tidak punya tipe Set bawaan
    // Gunakan map[T]struct{} sebagai set (struct{} tidak makan memori)
    
    set := map[string]struct{}{}
    
    // Tambah elemen
    set["apple"] = struct{}{}
    set["banana"] = struct{}{}
    set["apple"] = struct{}{}  // duplicate, diabaikan
    
    // Cek membership
    if _, ok := set["apple"]; ok {
        fmt.Println("apple ada di set")
    }
    
    // Hapus
    delete(set, "banana")
    
    fmt.Println("Set size:", len(set))  // 1
    
    // Iterasi
    for item := range set {
        fmt.Println(item)
    }
}
```

### 🏋️ Latihan 1.8

1. Buat program yang menghitung **frekuensi kata** dalam sebuah kalimat menggunakan map
2. Buat fungsi `groupByLength(words []string) map[int][]string` yang mengelompokkan kata berdasarkan panjangnya
3. Buat fungsi `invertMap(m map[string]string) map[string]string` yang menukar key dan value

---

## 📦 Modul 1.9 — Structs & Methods

### Struct Dasar

```go
package main

import (
    "fmt"
    "math"
)

// Deklarasi struct
type Person struct {
    Name    string
    Age     int
    Email   string
    IsActive bool
}

// Struct dengan tag (penting untuk JSON, DB, validasi)
type Product struct {
    ID    int     `json:"id"`
    Name  string  `json:"name"`
    Price float64 `json:"price,omitempty"`
}

func main() {
    // Membuat instance struct
    p1 := Person{
        Name:     "Alice",
        Age:      30,
        Email:    "alice@example.com",
        IsActive: true,
    }
    
    // Positional initialization (tidak direkomendasikan)
    p2 := Person{"Bob", 25, "bob@example.com", false}
    
    // Zero value struct
    var p3 Person
    
    // Akses field
    fmt.Println(p1.Name)   // Alice
    fmt.Println(p2.Age)    // 25
    fmt.Println(p3.Name)   // "" (zero value)
    
    // Modifikasi field
    p1.Age = 31
    
    // Struct adalah VALUE TYPE
    p4 := p1        // COPY
    p4.Name = "Copy"
    fmt.Println(p1.Name)  // Alice (tidak berubah)
    fmt.Println(p4.Name)  // Copy
    
    // Pointer ke struct
    p5 := &Person{Name: "Eve", Age: 28}
    p5.Age = 29          // otomatis di-dereference
    (*p5).Age = 30       // sama dengan baris di atas
    fmt.Println(p5)
}
```

### Methods

```go
package main

import (
    "fmt"
    "math"
)

type Rectangle struct {
    Width  float64
    Height float64
}

// Method dengan value receiver
// Dipakai saat: tidak perlu modifikasi, struct kecil
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

// Method dengan pointer receiver
// Dipakai saat: perlu modifikasi, atau struct besar
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

func (r Rectangle) String() string {
    return fmt.Sprintf("Rectangle(%.2f x %.2f)", r.Width, r.Height)
}

// ==================

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c Circle) String() string {
    return fmt.Sprintf("Circle(r=%.2f)", c.Radius)
}

// ==================

func main() {
    rect := Rectangle{Width: 5, Height: 3}
    
    fmt.Println(rect)                  // Rectangle(5.00 x 3.00)
    fmt.Printf("Area: %.2f\n", rect.Area())
    fmt.Printf("Perimeter: %.2f\n", rect.Perimeter())
    
    // Memanggil pointer receiver method
    rect.Scale(2)
    fmt.Println(rect)  // Rectangle(10.00 x 6.00)
    
    // Pointer ke struct, panggil method
    pRect := &Rectangle{Width: 4, Height: 7}
    fmt.Printf("Area: %.2f\n", pRect.Area())  // otomatis di-dereference
    
    circle := Circle{Radius: 5}
    fmt.Println(circle)
    fmt.Printf("Area: %.4f\n", circle.Area())
}
```

### Embedded Struct (Komposisi)

```go
package main

import "fmt"

// Base struct
type Animal struct {
    Name string
    Age  int
}

func (a Animal) Describe() string {
    return fmt.Sprintf("I am %s, %d years old", a.Name, a.Age)
}

// Embedded struct (seperti inheritance tapi lebih fleksibel)
type Dog struct {
    Animal       // embed Animal
    Breed string
}

func (d Dog) Bark() string {
    return fmt.Sprintf("%s says: Woof!", d.Name)  // akses field Animal langsung
}

type Cat struct {
    Animal
    IsIndoor bool
}

func main() {
    dog := Dog{
        Animal: Animal{Name: "Rex", Age: 3},
        Breed:  "German Shepherd",
    }
    
    // Akses field embedded langsung
    fmt.Println(dog.Name)         // Rex (dari Animal)
    fmt.Println(dog.Animal.Name)  // Rex (explicit)
    fmt.Println(dog.Breed)        // German Shepherd
    
    // Method promotion — method Animal tersedia di Dog
    fmt.Println(dog.Describe())   // I am Rex, 3 years old
    fmt.Println(dog.Bark())       // Rex says: Woof!
    
    cat := Cat{
        Animal:   Animal{Name: "Whiskers", Age: 5},
        IsIndoor: true,
    }
    fmt.Println(cat.Describe())
}
```

### 🏋️ Latihan 1.9

1. Buat struct `BankAccount` dengan field: `owner`, `balance`, `accountNumber`
2. Tambahkan methods: `Deposit(amount float64)`, `Withdraw(amount float64) error`, `GetBalance() float64`
3. Buat struct `SavingsAccount` yang embed `BankAccount` dengan tambahan field `interestRate`
4. Tambahkan method `AddInterest()` ke `SavingsAccount`

---

## 📦 Modul 1.10 — Pointers

### Konsep Pointer

```go
package main

import "fmt"

func main() {
    // Variabel biasa (value)
    x := 42
    
    // Pointer ke x
    // & = "address of" — mendapatkan alamat memori
    p := &x
    
    fmt.Println(x)   // 42  (nilai x)
    fmt.Println(p)   // 0xc000014080 (alamat memori x)
    
    // * = "dereference" — mengakses nilai di alamat memori
    fmt.Println(*p)  // 42 (nilai di alamat yang ditunjuk p)
    
    // Mengubah nilai melalui pointer
    *p = 100
    fmt.Println(x)   // 100 (x berubah!)
    
    // Pointer ke string
    name := "Alice"
    pName := &name
    *pName = "Bob"
    fmt.Println(name)  // Bob
    
    // Tipe pointer
    var ptr *int        // nil pointer (tidak menunjuk ke mana-mana)
    fmt.Println(ptr)    // <nil>
    fmt.Println(ptr == nil)  // true
    
    // JANGAN dereference nil pointer! → panic
    // fmt.Println(*ptr)  // panic: runtime error: invalid memory address
    
    // Membuat pointer baru dengan new
    p2 := new(int)   // *int, nilai awal 0
    *p2 = 42
    fmt.Println(*p2)  // 42
}
```

### Pass by Value vs Pass by Pointer

```go
package main

import "fmt"

// Pass by value — fungsi dapat COPY
func doubleValue(n int) {
    n *= 2  // hanya mengubah copy lokal
}

// Pass by pointer — fungsi dapat REFERENSI
func doublePointer(n *int) {
    *n *= 2  // mengubah nilai asli
}

type User struct {
    Name  string
    Score int
}

// Struct sebagai value — dapat copy (mahal untuk struct besar)
func updateUserValue(u User) {
    u.Score += 100
}

// Struct sebagai pointer — lebih efisien
func updateUserPointer(u *User) {
    u.Score += 100  // otomatis dereference
}

func main() {
    // Contoh 1: Integer
    num := 10
    doubleValue(num)
    fmt.Println(num)  // 10 — tidak berubah
    
    doublePointer(&num)
    fmt.Println(num)  // 20 — berubah!
    
    // Contoh 2: Struct
    user := User{Name: "Alice", Score: 50}
    
    updateUserValue(user)
    fmt.Println(user.Score)  // 50 — tidak berubah
    
    updateUserPointer(&user)
    fmt.Println(user.Score)  // 150 — berubah!
    
    // Membuat pointer dari struct
    u2 := &User{Name: "Bob", Score: 0}
    updateUserPointer(u2)
    fmt.Println(u2.Score)  // 100
}
```

### 🏋️ Latihan 1.10

1. Buat fungsi `swap(a, b *int)` yang menukar nilai dua integer menggunakan pointer
2. Buat fungsi `increment(n *int, by int)` yang menambah nilai n sebesar by
3. Jelaskan (dalam komentar kode) kapan kamu sebaiknya pakai pointer vs value

---

## 📦 Modul 1.11 — Error Handling

Error handling adalah **salah satu ciri khas terpenting** di Go. Go tidak menggunakan exception; error adalah **nilai biasa** yang dikembalikan.

### Error Basics

```go
package main

import (
    "errors"
    "fmt"
)

// errors.New — membuat error sederhana
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// fmt.Errorf — error dengan format string
func validateAge(age int) error {
    if age < 0 {
        return fmt.Errorf("age %d is invalid: must be non-negative", age)
    }
    if age > 150 {
        return fmt.Errorf("age %d is invalid: too large", age)
    }
    return nil
}

func main() {
    // Pola standar error handling Go
    result, err := divide(10, 2)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Printf("10 / 2 = %.2f\n", result)
    
    _, err = divide(5, 0)
    if err != nil {
        fmt.Println("Error:", err)  // Error: division by zero
    }
    
    // Validate age
    if err := validateAge(-5); err != nil {
        fmt.Println(err)
    }
    if err := validateAge(25); err != nil {
        fmt.Println(err)
    } else {
        fmt.Println("Age is valid")
    }
}
```

### Custom Error Types

```go
package main

import "fmt"

// Custom error type dengan implementasi interface error
type ValidationError struct {
    Field   string
    Message string
}

// Harus implement method Error() string untuk interface error
func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on field '%s': %s", e.Field, e.Message)
}

// Lebih complex custom error
type NotFoundError struct {
    Resource string
    ID       int
}

func (e *NotFoundError) Error() string {
    return fmt.Sprintf("%s with ID %d not found", e.Resource, e.ID)
}

func findUser(id int) (string, error) {
    users := map[int]string{1: "Alice", 2: "Bob"}
    
    if id <= 0 {
        return "", &ValidationError{Field: "id", Message: "must be positive"}
    }
    
    name, ok := users[id]
    if !ok {
        return "", &NotFoundError{Resource: "User", ID: id}
    }
    
    return name, nil
}

func main() {
    // Test dengan berbagai ID
    ids := []int{-1, 1, 99}
    
    for _, id := range ids {
        name, err := findUser(id)
        if err != nil {
            // Type assertion untuk cek jenis error
            var notFound *NotFoundError
            var validation *ValidationError
            
            switch {
            case errors.As(err, &notFound):
                fmt.Printf("[NOT FOUND] %v\n", err)
            case errors.As(err, &validation):
                fmt.Printf("[VALIDATION] %v\n", err)
            default:
                fmt.Printf("[ERROR] %v\n", err)
            }
            continue
        }
        fmt.Printf("Found user: %s\n", name)
    }
}
```

### Error Wrapping (Go 1.13+)

```go
package main

import (
    "errors"
    "fmt"
)

var ErrNotFound = errors.New("not found")
var ErrPermission = errors.New("permission denied")

func getUser(id int) error {
    if id == 0 {
        // Wrap error dengan context tambahan
        return fmt.Errorf("getUser(%d): %w", id, ErrNotFound)
    }
    return nil
}

func processUser(id int) error {
    if err := getUser(id); err != nil {
        // Wrap lagi dengan konteks lebih tinggi
        return fmt.Errorf("processUser: %w", err)
    }
    return nil
}

func main() {
    err := processUser(0)
    if err != nil {
        fmt.Println(err)
        // Output: processUser: getUser(0): not found
        
        // Unwrap untuk cek root error
        if errors.Is(err, ErrNotFound) {
            fmt.Println("Is ErrNotFound: true")
        }
    }
}
```

### 🏋️ Latihan 1.11

1. Buat custom error `InsufficientFundsError` untuk operasi withdraw bank
2. Implementasikan fungsi `Transfer(from, to *BankAccount, amount float64) error`
3. Handle berbagai jenis error dengan type assertion menggunakan `errors.As`

---

## 📦 Modul 1.12 — Packages & Modules

### Struktur Package

```
myapp/
├── go.mod
├── main.go           ← package main
├── math/
│   ├── math.go       ← package math
│   └── math_test.go
├── string/
│   └── string.go     ← package string (custom)
└── model/
    └── user.go       ← package model
```

### Membuat Package

```go
// File: math/math.go
package math  // nama package sesuai folder

// Fungsi EXPORTED: huruf kapital
func Add(a, b int) int {
    return a + b
}

func Subtract(a, b int) int {
    return a - b
}

// Fungsi unexported (private): huruf kecil
func helper(x int) int {
    return x * 2
}

// Variabel dan konstanta exported
var DefaultPrecision = 2
const Pi = 3.14159
```

```go
// File: model/user.go
package model

type User struct {
    ID    int
    Name  string
    Email string
    // unexported field (tidak bisa diakses dari luar package)
    password string
}

// Constructor function (idiom Go)
func NewUser(id int, name, email string) *User {
    return &User{
        ID:    id,
        Name:  name,
        Email: email,
    }
}

func (u *User) SetPassword(pwd string) {
    u.password = pwd  // bisa akses karena dalam package yang sama
}
```

```go
// File: main.go
package main

import (
    "fmt"
    
    // Import package lokal menggunakan module path
    custommath "github.com/kamu/myapp/math"
    "github.com/kamu/myapp/model"
)

func main() {
    sum := custommath.Add(3, 5)
    fmt.Println(sum)
    
    user := model.NewUser(1, "Alice", "alice@example.com")
    fmt.Println(user.Name)
    // fmt.Println(user.password)  // ERROR: unexported
}
```

### go.mod dan Dependency Management

```bash
# Menambah dependency
go get github.com/gin-gonic/gin@latest

# Menambah dependency versi spesifik
go get github.com/gin-gonic/gin@v1.9.1

# Download semua dependency
go mod download

# Bersihkan dependency yang tidak dipakai
go mod tidy

# Lihat semua dependency
go list -m all
```

go.mod setelah menambah dependency:
```
module github.com/kamu/myapp

go 1.22

require (
    github.com/gin-gonic/gin v1.9.1
)
```

### Standard Library yang Wajib Tahu

```go
package main

import (
    "fmt"         // print, format, scan
    "strings"     // manipulasi string
    "strconv"     // konversi string-angka
    "math"        // fungsi matematika
    "sort"        // sorting
    "time"        // tanggal dan waktu
    "os"          // sistem operasi, file, env
    "io"          // input/output
    "bufio"       // buffered I/O
    "encoding/json" // JSON encode/decode
    "net/http"    // HTTP client/server
    "log"         // logging dasar
    "regexp"      // regular expression
    "path/filepath" // path manipulation
    "errors"      // error utilities
    "context"     // context untuk timeout, cancel
)
```

Contoh penggunaan `strings` dan `strconv`:

```go
package main

import (
    "fmt"
    "strconv"
    "strings"
)

func main() {
    // strings package
    s := "Hello, World!"
    
    fmt.Println(strings.ToUpper(s))           // HELLO, WORLD!
    fmt.Println(strings.ToLower(s))           // hello, world!
    fmt.Println(strings.Contains(s, "World")) // true
    fmt.Println(strings.HasPrefix(s, "Hello")) // true
    fmt.Println(strings.HasSuffix(s, "!"))    // true
    fmt.Println(strings.Replace(s, "World", "Go", 1)) // Hello, Go!
    fmt.Println(strings.TrimSpace("  hello  "))  // hello
    fmt.Println(strings.Split("a,b,c", ","))  // [a b c]
    fmt.Println(strings.Join([]string{"a", "b", "c"}, "-"))  // a-b-c
    fmt.Println(strings.Count(s, "l"))        // 3
    fmt.Println(strings.Index(s, "World"))    // 7
    
    // strconv package
    // int ke string
    n := 42
    str := strconv.Itoa(n)  // "42"
    fmt.Printf("%T %v\n", str, str)
    
    // string ke int
    m, err := strconv.Atoi("123")
    if err == nil {
        fmt.Println(m + 1)  // 124
    }
    
    // float ke string
    f := 3.14
    fStr := strconv.FormatFloat(f, 'f', 2, 64)  // "3.14"
    fmt.Println(fStr)
    
    // string ke float
    fVal, _ := strconv.ParseFloat("3.14", 64)
    fmt.Println(fVal * 2)  // 6.28
    
    // bool ke string dan sebaliknya
    bStr := strconv.FormatBool(true)   // "true"
    bVal, _ := strconv.ParseBool("true")  // true
    fmt.Println(bStr, bVal)
}
```

### 🏋️ Latihan 1.12

1. Buat package `calculator` dengan fungsi `Add`, `Subtract`, `Multiply`, `Divide`
2. Buat package `validator` dengan fungsi `ValidateEmail(email string) bool` dan `ValidateAge(age int) error`
3. Import dan gunakan kedua package tersebut di `main.go`

---

## 📦 Modul 1.13 — Defer, Panic, Recover

### defer

`defer` menunda eksekusi fungsi hingga fungsi pemanggil selesai. Sangat berguna untuk **cleanup resources**.

```go
package main

import "fmt"

func main() {
    fmt.Println("Start")
    
    // defer dieksekusi LIFO (Last In, First Out)
    defer fmt.Println("Deferred 1")
    defer fmt.Println("Deferred 2")
    defer fmt.Println("Deferred 3")
    
    fmt.Println("End")
    
    // Output:
    // Start
    // End
    // Deferred 3
    // Deferred 2
    // Deferred 1
}
```

Penggunaan umum defer untuk resource cleanup:

```go
package main

import (
    "fmt"
    "os"
)

func readFile(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()  // ← SELALU close di defer setelah open berhasil
    
    // Lakukan sesuatu dengan file
    // file.Close() dipanggil otomatis saat fungsi selesai
    
    return nil
}

// Simulasi database connection
type DB struct {
    Name string
}

func (db *DB) Close() {
    fmt.Printf("Closing DB: %s\n", db.Name)
}

func processData() {
    db := &DB{Name: "mydb"}
    defer db.Close()  // cleanup dijamin dipanggil
    
    fmt.Println("Processing...")
    // Meski ada return atau panic, db.Close() tetap dipanggil
}

func main() {
    processData()
}
```

### panic dan recover

```go
package main

import "fmt"

// recover harus dipanggil di dalam fungsi yang di-defer
func safeDiv(a, b int) (result int, err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("recovered from panic: %v", r)
        }
    }()
    
    result = a / b  // akan panic jika b == 0
    return result, nil
}

func mustPositive(n int) int {
    if n < 0 {
        panic(fmt.Sprintf("nilai harus positif, dapat: %d", n))
    }
    return n
}

func main() {
    // safeDiv dengan recover
    result, err := safeDiv(10, 2)
    fmt.Println(result, err)  // 5 <nil>
    
    result, err = safeDiv(10, 0)
    fmt.Println(result, err)  // 0 recovered from panic: ...
    
    // panic akan menghentikan program jika tidak di-recover
    // mustPositive(-5)  // ini akan crash program
    
    // Kapan pakai panic?
    // - Kondisi yang benar-benar tidak seharusnya terjadi
    // - Inisialisasi yang gagal (program tidak bisa jalan)
    // - Contoh: parsing template yang hardcoded gagal
}
```

### 🏋️ Latihan 1.13

1. Buat fungsi yang membuka file, membaca isinya, dan menggunakan `defer` untuk close
2. Buat fungsi dengan `recover` yang menangkap panic dan mengembalikan error

---

## 🎯 Review & Checkpoint Fase 1

Sebelum lanjut ke project, pastikan kamu bisa menjawab pertanyaan ini tanpa melihat materi:

### Konseptual
- [ ] Apa perbedaan array dan slice di Go?
- [ ] Kapan pakai pointer receiver vs value receiver?
- [ ] Mengapa error di Go adalah nilai, bukan exception?
- [ ] Apa itu zero value? Sebutkan zero value untuk int, string, bool, slice, map
- [ ] Apa perbedaan `var x int` dan `x := 0`?

### Praktis
- [ ] Cara membuat dan menggunakan package terpisah
- [ ] Cara handle multiple return values
- [ ] Cara deklarasi dan menggunakan struct dengan method
- [ ] Cara iterasi slice dan map dengan `range`
- [ ] Cara menggunakan `defer` untuk resource cleanup

---

## 🎯 Project Akhir Fase 1

Setelah semua modul selesai, kerjakan project **CLI Todo Manager** berdasarkan PRD di file:

**`FASE-1-PRD-CLI-Todo-Manager.md`**

Project ini akan menguji semua konsep yang sudah dipelajari:
- Struct & Methods
- Slices & Maps
- Error handling
- Packages
- File I/O (bonus)

---

## 📚 Resources Tambahan Fase 1

- [A Tour of Go](https://go.dev/tour/) — interaktif, kerjakan semua!
- [Go by Example](https://gobyexample.com/) — referensi cepat
- [Effective Go](https://go.dev/doc/effective_go) — baca setelah selesai fase ini
- [Go Playground](https://go.dev/play/) — untuk eksperimen cepat

---

*Setelah selesai Fase 1, lanjut ke `FASE-2-Go-Intermediate.md`*

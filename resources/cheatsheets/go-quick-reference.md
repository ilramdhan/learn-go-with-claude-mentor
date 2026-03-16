# 🗒️ Go Quick Reference Cheatsheet

> Referensi cepat untuk syntax dan idiom Go yang sering digunakan.
> Gunakan dengan `Ctrl+F` untuk cari syntax yang dibutuhkan.

---

## 📌 Declarations

```go
var x int = 10          // explicit type
var x = 10              // inferred type
x := 10                 // short declaration (hanya di dalam function)
const Pi = 3.14159      // constant
const (                 // iota enum pattern
    Small = iota        // 0
    Medium              // 1
    Large               // 2
)
```

## 📌 Basic Types

```go
bool                    // true, false
string                  // "hello", `raw string`
int, int8, int16, int32, int64
uint, uint8 (byte), uint16, uint32, uint64, uintptr
float32, float64
complex64, complex128
rune                    // alias for int32 (Unicode code point)
```

## 📌 Zero Values

```go
int, float → 0       string → ""
bool → false         pointer, slice, map, channel, func, interface → nil
struct → all fields zero-valued
```

## 📌 Arrays, Slices, Maps

```go
// Array — fixed size
arr := [3]int{1, 2, 3}
arr := [...]int{1, 2, 3}    // size inferred

// Slice — dynamic
s := []int{1, 2, 3}
s = append(s, 4)
s2 := s[1:3]                // [2, 3] — shares underlying array
s3 := make([]int, 0, 10)   // len=0, cap=10

// Copy slice (independent)
dst := make([]int, len(src))
copy(dst, src)

// Map
m := map[string]int{"a": 1}
m := make(map[string]int)
m["key"] = 42
v, ok := m["key"]          // ok=false jika tidak ada
delete(m, "key")
```

## 📌 Structs & Methods

```go
type User struct {
    ID    uint
    Name  string
    Email string
}

// Value receiver (tidak mutasi)
func (u User) String() string { return u.Name }

// Pointer receiver (bisa mutasi)
func (u *User) SetName(name string) { u.Name = name }

// Embedding
type Admin struct {
    User           // embedded — semua method User tersedia
    Permissions []string
}
```

## 📌 Interfaces

```go
type Stringer interface {
    String() string
}

// Interface composition
type ReadWriter interface {
    io.Reader
    io.Writer
}

// Type assertion
if s, ok := v.(Stringer); ok {
    fmt.Println(s.String())
}

// Type switch
switch v := i.(type) {
case int:    fmt.Println("int:", v)
case string: fmt.Println("string:", v)
default:     fmt.Println("unknown")
}

// Empty interface (accept anything)
func Print(v interface{}) { fmt.Println(v) }
func Print(v any) { fmt.Println(v) }  // Go 1.18+
```

## 📌 Goroutines & Channels

```go
// Goroutine
go func() {
    // berjalan secara concurrent
}()

// Channel
ch := make(chan int)        // unbuffered
ch := make(chan int, 10)    // buffered capacity 10

ch <- 42                    // send (blocks jika unbuffered dan tidak ada receiver)
v := <-ch                   // receive
v, ok := <-ch               // ok=false jika channel closed

close(ch)                   // signal: tidak ada lagi yang dikirim

// Range over channel
for v := range ch {         // stop saat channel ditutup
    fmt.Println(v)
}

// Select
select {
case v := <-ch1:    fmt.Println("ch1:", v)
case ch2 <- 42:     fmt.Println("sent to ch2")
case <-time.After(1 * time.Second):
    fmt.Println("timeout")
default:            fmt.Println("no channel ready")
}
```

## 📌 sync Package

```go
// Mutex
var mu sync.Mutex
mu.Lock()
defer mu.Unlock()

// RWMutex (multiple readers, single writer)
var rw sync.RWMutex
rw.RLock(); defer rw.RUnlock()  // read lock
rw.Lock(); defer rw.Unlock()    // write lock

// WaitGroup
var wg sync.WaitGroup
wg.Add(1)
go func() { defer wg.Done(); work() }()
wg.Wait()

// Once (run hanya sekali)
var once sync.Once
once.Do(func() { initialize() })

// Pool (object pooling)
pool := sync.Pool{New: func() interface{} { return &Buffer{} }}
buf := pool.Get().(*Buffer)
defer pool.Put(buf)

// atomic
var counter atomic.Int64
counter.Add(1)
counter.Load()
```

## 📌 Context

```go
// Background dan TODO
ctx := context.Background()   // root context
ctx := context.TODO()         // placeholder

// With cancellation
ctx, cancel := context.WithCancel(parent)
defer cancel()                // ALWAYS defer cancel!

// With timeout
ctx, cancel := context.WithTimeout(parent, 5*time.Second)
defer cancel()

// With deadline
ctx, cancel := context.WithDeadline(parent, time.Now().Add(5*time.Second))
defer cancel()

// With value (gunakan custom key type, bukan string!)
type contextKey string
ctx = context.WithValue(ctx, contextKey("userID"), uint(123))
userID := ctx.Value(contextKey("userID")).(uint)

// Check cancellation
select {
case <-ctx.Done():
    return ctx.Err()  // context.Canceled atau context.DeadlineExceeded
default:
    // proceed
}
```

## 📌 Error Handling

```go
// Basic error
if err != nil {
    return fmt.Errorf("operation failed: %w", err)  // %w = wrappable
}

// Custom error type
type ValidationError struct {
    Field   string
    Message string
}
func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error: %s — %s", e.Field, e.Message)
}

// errors package
errors.Is(err, target)       // check error chain
errors.As(err, &target)      // extract specific error type
errors.Unwrap(err)           // one level unwrap

// Sentinel errors
var ErrNotFound = errors.New("not found")
```

## 📌 Goroutine Patterns

```go
// Fan-out: 1 producer, N workers
func fanOut(input <-chan Work, workers int) []<-chan Result {
    channels := make([]<-chan Result, workers)
    for i := range channels {
        channels[i] = worker(input)
    }
    return channels
}

// Fan-in: N producers, 1 consumer
func merge(channels ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup
    for _, ch := range channels {
        wg.Add(1)
        go func(c <-chan int) {
            defer wg.Done()
            for v := range c { out <- v }
        }(ch)
    }
    go func() { wg.Wait(); close(out) }()
    return out
}

// Worker pool
func workerPool(jobs <-chan Job, numWorkers int) <-chan Result {
    results := make(chan Result)
    var wg sync.WaitGroup
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for job := range jobs {
                results <- process(job)
            }
        }()
    }
    go func() { wg.Wait(); close(results) }()
    return results
}
```

## 📌 Generics (Go 1.18+)

```go
// Generic function
func Map[T, U any](s []T, f func(T) U) []U {
    result := make([]U, len(s))
    for i, v := range s {
        result[i] = f(v)
    }
    return result
}

// Generic type constraint
type Number interface {
    int | int64 | float64
}
func Sum[T Number](nums []T) T {
    var total T
    for _, n := range nums { total += n }
    return total
}

// Generic struct
type Stack[T any] struct { items []T }
func (s *Stack[T]) Push(item T) { s.items = append(s.items, item) }
func (s *Stack[T]) Pop() (T, bool) {
    var zero T
    if len(s.items) == 0 { return zero, false }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}
```

## 📌 Testing

```go
// Unit test
func TestAdd(t *testing.T) {
    got := Add(2, 3)
    want := 5
    if got != want {
        t.Errorf("Add(2,3) = %d, want %d", got, want)
    }
}

// Table-driven test (idiomatic Go)
func TestAdd_TableDriven(t *testing.T) {
    tests := []struct {
        name string; a, b, want int
    }{
        {"positive", 2, 3, 5},
        {"negative", -1, -2, -3},
        {"zero", 0, 0, 0},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Add(tt.a, tt.b); got != tt.want {
                t.Errorf("got %d, want %d", got, tt.want)
            }
        })
    }
}

// Benchmark
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ { Add(2, 3) }
}

// Run: go test -v ./...
// Run: go test -bench=. -benchmem ./...
// Run: go test -coverprofile=coverage.out ./...
```

## 📌 HTTP Server (net/http)

```go
// Basic handler
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Hello, World!")
})

// With context
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    // use ctx for downstream calls
}

// JSON response
json.NewEncoder(w).Encode(data)
w.Header().Set("Content-Type", "application/json")
w.WriteHeader(http.StatusCreated)

// Server with timeout
srv := &http.Server{
    Addr:         ":8080",
    Handler:      mux,
    ReadTimeout:  15 * time.Second,
    WriteTimeout: 15 * time.Second,
    IdleTimeout:  60 * time.Second,
}
srv.ListenAndServe()
```

## 📌 Useful Commands

```bash
go run main.go              # run program
go build -o myapp .         # build binary
go test ./...               # run all tests
go test -v -race ./...      # verbose + race detector
go test -cover ./...        # with coverage
go mod tidy                 # clean up modules
go mod download             # download dependencies
go get package@version      # add/update dependency
go vet ./...                # static analysis
gofmt -w .                  # format code
golangci-lint run           # lint (install separately)
go tool pprof               # profiling
go doc package.Function     # documentation
```

## 📌 Go Proverbs (Idioms to Remember)

```
"Don't communicate by sharing memory; share memory by communicating."
"Errors are values."
"Don't just check errors, handle them gracefully."
"A little copying is better than a little dependency."
"Clear is better than clever."
"Concurrency is not parallelism."
"The bigger the interface, the weaker the abstraction."
```

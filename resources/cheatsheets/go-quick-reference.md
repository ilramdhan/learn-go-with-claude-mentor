# 🗒️ Go Quick Reference Cheatsheet

> Referensi cepat untuk syntax dan idiom Go yang sering digunakan

---

## Declarations

```go
var x int = 10          // explicit type
var x = 10              // inferred type
x := 10                 // short (only inside functions)
const Pi = 3.14         // constant
const Size = 1 << 20    // 1 MB in bytes
```

## Basic Types

```go
bool        // true, false
string      // "hello"
int, int8, int16, int32, int64
uint, uint8 (byte), uint16, uint32, uint64
float32, float64
complex64, complex128
rune        // alias for int32 (Unicode code point)
```

## Composite Types

```go
// Array (fixed size)
var arr [5]int
arr := [3]string{"a", "b", "c"}
arr := [...]int{1, 2, 3}  // size inferred

// Slice (dynamic)
s := []int{1, 2, 3}
s = append(s, 4)
s2 := s[1:3]              // reslice
copy(dst, src)            // copy slice

// Map
m := map[string]int{"a": 1}
m["b"] = 2
v, ok := m["key"]         // check existence
delete(m, "key")

// Struct
type Point struct { X, Y float64 }
p := Point{X: 1, Y: 2}
p.X = 10
```

## Functions

```go
func add(a, b int) int { return a + b }
func swap(a, b int) (int, int) { return b, a }
func sum(nums ...int) int { /* variadic */ }
result, err := divide(10, 2)  // multiple return

// Function value
fn := func(x int) int { return x * 2 }

// Closure
adder := func(n int) func(int) int {
    return func(x int) int { return x + n }
}
```

## Control Flow

```go
// if
if x > 0 { ... } else if x < 0 { ... } else { ... }
if v := f(); v > 0 { ... }  // with init statement

// switch
switch x {
case 1, 2: fmt.Println("1 or 2")
default:   fmt.Println("other")
}

switch {  // no condition = if-else chain
case x > 0: ...
}

// for (the only loop)
for i := 0; i < 10; i++ { ... }
for condition { ... }   // while
for { ... }             // infinite
for i, v := range slice { ... }
for k, v := range mapVar { ... }
```

## Pointers

```go
x := 42
p := &x       // pointer to x
*p = 100      // dereference
p2 := new(int) // allocate, returns *int
```

## Interfaces

```go
type Stringer interface { String() string }

// Implement by having the method:
func (p Point) String() string { return fmt.Sprintf("(%f,%f)", p.X, p.Y) }

// Type assertion
s, ok := i.(string)

// Type switch
switch v := i.(type) {
case int:    fmt.Println("int:", v)
case string: fmt.Println("string:", v)
}
```

## Goroutines & Channels

```go
go f()                    // start goroutine
ch := make(chan int)       // unbuffered
ch := make(chan int, 10)   // buffered
ch <- 42                  // send
v := <-ch                 // receive
close(ch)                 // close (sender only!)
for v := range ch { ... } // receive until closed

// Select
select {
case v := <-ch1: ...
case ch2 <- v:   ...
case <-time.After(1 * time.Second): ...
default:         ...  // non-blocking
}
```

## Error Handling

```go
// Create errors
err := errors.New("something went wrong")
err := fmt.Errorf("operation failed: %w", cause)

// Custom error
type NotFoundError struct { ID int }
func (e *NotFoundError) Error() string { return fmt.Sprintf("id %d not found", e.ID) }

// Check errors
if err != nil { return err }
if errors.Is(err, ErrNotFound) { ... }
var nfe *NotFoundError
if errors.As(err, &nfe) { ... }
```

## Defer, Panic, Recover

```go
defer f.Close()           // runs when function returns
defer fmt.Println("done") // LIFO order

// Panic & recover
defer func() {
    if r := recover(); r != nil {
        fmt.Println("recovered:", r)
    }
}()
panic("something terrible")
```

## Common Patterns

```go
// Functional options
type Option func(*Server)
func WithPort(p int) Option { return func(s *Server) { s.port = p } }
s := NewServer(WithPort(8080), WithTimeout(30*time.Second))

// Context
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

// Error wrapping
return fmt.Errorf("userService.Create: %w", err)

// Initialization
var once sync.Once
once.Do(func() { /* runs once */ })
```

## Useful Standard Library

```go
import (
    "fmt"            // Printf, Sprintf, Println, Errorf
    "strings"        // Contains, Split, Join, ToUpper, Trim
    "strconv"        // Atoi, Itoa, ParseFloat, FormatFloat
    "sort"           // Ints, Strings, Slice
    "time"           // Now, Parse, Format, Sleep, After
    "os"             // Args, Getenv, ReadFile, WriteFile
    "io"             // Reader, Writer, Copy
    "encoding/json"  // Marshal, Unmarshal
    "net/http"       // Get, Post, ListenAndServe
    "sync"           // Mutex, WaitGroup, Once
    "context"        // Background, WithTimeout, WithCancel
    "errors"         // New, Is, As, Unwrap
    "path/filepath"  // Join, Dir, Base, Ext
    "regexp"         // MustCompile, MatchString
)
```

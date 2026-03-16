# 📘 FASE 10: Testing Mastery

> **Prasyarat:** Selesaikan Fase 1–4 (fundamental), Fase 7 (microservices) direkomendasikan
> **Durasi:** 2–3 minggu
> **Project Akhir:** Comprehensive Test Suite untuk E-Commerce System
> **Tujuan:** Menguasai semua level testing dari unit test hingga load test — menghasilkan kode yang benar-benar teruji dan percaya diri di-deploy ke production

---

## 🗂️ Daftar Modul

| # | Modul | Topik |
|---|-------|-------|
| 10.1 | Testing Philosophy & Pyramid | Kenapa testing, test pyramid, ROI |
| 10.2 | Unit Testing Advanced | testify, table-driven, sub-tests, parallel |
| 10.3 | Testing dengan DI & Interface | Mock generation, stub, fake, spy |
| 10.4 | Integration Testing | testcontainers, real database, setup/teardown |
| 10.5 | HTTP Handler Testing | httptest, recorder, golden files |
| 10.6 | gRPC Testing | bufconn, in-process server, mock streams |
| 10.7 | Contract Testing | Consumer-driven contracts, Pact |
| 10.8 | Load & Performance Testing | k6, benchmark, latency targets |
| 10.9 | Fuzz Testing | Go fuzz, corpus, finding edge cases |
| 10.10 | Mutation Testing | go-mutesting, verifikasi kualitas test |
| 10.11 | Test Coverage Strategy | Coverage tools, meaningful coverage |
| 10.12 | Testing Best Practices | Anti-patterns, test doubles taxonomy |
| 10.13 | CI/CD Integration | GitHub Actions, test gates, reporting |

---

## 📦 Modul 10.1 — Testing Philosophy & Test Pyramid

### Mengapa Testing Penting?

```
Tanpa testing:
  - "Semoga jalan di production" → gambling, bukan engineering
  - Bug ditemukan oleh user, bukan developer
  - Refactor = takut → kode makin kotor
  - Onboarding developer baru susah
  - Deploy = stress event

Dengan testing:
  - Refactor dengan percaya diri
  - Bug ditemukan saat development, bukan production
  - Kode sebagai dokumentasi yang selalu up-to-date
  - Deploy dengan tenang
  - Onboarding lebih cepat — test menunjukkan intent kode
```

### Test Pyramid

```
                    /\
                   /  \
                  / E2E\         ← Sedikit, lambat, mahal, brittle
                 /──────\          End-to-end test (browser/API flow)
                /        \
               / Integration\    ← Sedang, test beberapa komponen bersama
              /──────────────\     Real database, real HTTP server
             /                \
            /    Unit Tests    \  ← Banyak, cepat, murah, isolated
           /────────────────────\  Test satu fungsi/method/class
          ────────────────────────

Rasio yang direkomendasikan:
  Unit:        70% → ribuan test, run < 5 detik
  Integration: 20% → ratusan test, run < 2 menit
  E2E:         10% → puluhan test, run < 10 menit

Anti-pattern: Ice Cream Cone (kebalik!)
  Banyak E2E → lambat, flaky, sulit maintain
  Sedikit unit → bug tersembunyi, refactor susah
```

### Return on Investment (ROI) Testing

```
UNIT TESTS:
  Cost:   Rendah (cepat ditulis, cepat dijalankan)
  Value:  Tinggi (isolasi bug, refactor safety)
  Run in: Milliseconds
  Pakai untuk: business logic, algorithms, utilities

INTEGRATION TESTS:
  Cost:   Sedang (butuh setup infra)
  Value:  Tinggi (test actual wiring between components)
  Run in: Seconds
  Pakai untuk: database queries, HTTP handlers, gRPC handlers

E2E TESTS:
  Cost:   Tinggi (setup kompleks, slow, flaky)
  Value:  Tinggi tapi narrow (hanya critical paths)
  Run in: Minutes
  Pakai untuk: critical user journeys (checkout, login flow)
```

### Testing dalam Context Fase-Fase Sebelumnya

```
Fase 1-3: Unit test untuk functions, handlers
Fase 4:   Unit test per layer (Clean Architecture)
         → domain test tanpa DB
         → use case test dengan mock repo
         → handler test dengan mock use case
Fase 5:   gRPC handler test dengan bufconn
Fase 6:   Aggregate test (domain logic tanpa infra)
Fase 7:   Integration test dengan testcontainers
Fase 8:   Consumer contract test untuk Kafka events
Fase 9:   Observability test (metrics emitted, traces created)
Fase 10:  SEMUA di atas + load test + fuzz test
```

### 🏋️ Latihan 10.1

1. Audit codebase Auth Service dari Fase 4: hitung berapa % kode yang sudah ada test-nya. Buat daftar: (a) 5 fungsi yang paling kritis dan belum di-test, (b) 3 bug yang paling mungkin terjadi tanpa testing.
2. Gambar **test pyramid** untuk E-Commerce System: tentukan berapa banyak unit/integration/e2e test yang ideal, dan komponen mana yang di-test di level mana.
3. Buat **testing strategy document**: untuk setiap layer (domain, use case, handler, repository), tentukan: tipe test yang dipakai, dependencies yang di-mock vs nyata, dan target coverage %.

---

## 📦 Modul 10.2 — Unit Testing Advanced

### Setup testify

```bash
go get github.com/stretchr/testify/assert
go get github.com/stretchr/testify/require
go get github.com/stretchr/testify/suite
go get github.com/stretchr/testify/mock
```

### assert vs require

```go
// assert: test terus berjalan meski assertion gagal
// require: test STOP saat assertion gagal (untuk pre-conditions)

func TestCreateUser(t *testing.T) {
    user, err := NewUser("Alice", "alice@test.com", "hashedpwd")

    // require untuk pre-condition — jika err != nil, tidak ada gunanya lanjut
    require.NoError(t, err, "NewUser should not return error")
    require.NotNil(t, user, "user should not be nil")

    // assert untuk validasi nilai — boleh semua dicek meski ada yang gagal
    assert.Equal(t, "Alice", user.Name)
    assert.Equal(t, "alice@test.com", user.Email)
    assert.Equal(t, UserRoleUser, user.Role)
    assert.True(t, user.IsActive)
    assert.False(t, user.CreatedAt.IsZero())
}
```

### Table-Driven Tests — Idiom Terpenting Go

```go
// internal/domain/entity/user_test.go
package entity_test

import (
    "testing"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    "github.com/kamu/app/internal/domain/entity"
)

func TestNewUser(t *testing.T) {
    tests := []struct {
        name        string    // deskripsi test case
        inputName   string
        inputEmail  string
        inputHash   string
        wantErr     bool
        wantErrMsg  string    // substring dari error message
        wantRole    entity.UserRole
        wantActive  bool
    }{
        {
            name:       "valid user creation",
            inputName:  "Alice Wonderland",
            inputEmail: "alice@example.com",
            inputHash:  "$2a$14$hashedpassword",
            wantErr:    false,
            wantRole:   entity.RoleUser,
            wantActive: true,
        },
        {
            name:       "empty name",
            inputName:  "",
            inputEmail: "alice@example.com",
            inputHash:  "$2a$14$hash",
            wantErr:    true,
            wantErrMsg: "name",
        },
        {
            name:       "name too short",
            inputName:  "A",
            inputEmail: "alice@example.com",
            inputHash:  "$2a$14$hash",
            wantErr:    true,
            wantErrMsg: "2 and 100",
        },
        {
            name:       "empty email",
            inputName:  "Alice",
            inputEmail: "",
            inputHash:  "$2a$14$hash",
            wantErr:    true,
            wantErrMsg: "email",
        },
        {
            name:       "empty password hash",
            inputName:  "Alice",
            inputEmail: "alice@example.com",
            inputHash:  "",
            wantErr:    true,
            wantErrMsg: "password",
        },
    }

    for _, tt := range tests {
        // Gunakan tt.name sebagai sub-test name
        t.Run(tt.name, func(t *testing.T) {
            // t.Parallel() — jalankan sub-test secara paralel (opsional)

            got, err := entity.NewUser(tt.inputName, tt.inputEmail, tt.inputHash)

            if tt.wantErr {
                require.Error(t, err, "expected error but got nil")
                if tt.wantErrMsg != "" {
                    assert.Contains(t, err.Error(), tt.wantErrMsg,
                        "error message should contain expected substring")
                }
                assert.Nil(t, got, "user should be nil on error")
                return
            }

            require.NoError(t, err)
            require.NotNil(t, got)
            assert.Equal(t, tt.inputName, got.Name)
            assert.Equal(t, tt.inputEmail, got.Email)
            assert.Equal(t, tt.wantRole, got.Role)
            assert.Equal(t, tt.wantActive, got.IsActive)
        })
    }
}
```

### Sub-tests dan Parallel Testing

```go
func TestOrder_AddItem(t *testing.T) {
    // Helper untuk buat order yang valid — DRY!
    makeValidOrder := func(t *testing.T) *aggregate.Order {
        t.Helper() // error akan menunjuk ke caller, bukan helper ini
        addr, err := valueobject.NewAddress("Jl. Sudirman 1", "Jakarta", "DKI", "12190", "ID")
        require.NoError(t, err)
        order, err := aggregate.NewOrder(123, addr)
        require.NoError(t, err)
        return order
    }

    t.Run("success", func(t *testing.T) {
        t.Parallel() // test ini bisa berjalan bersamaan dengan yang lain

        order := makeValidOrder(t)
        price, _ := valueobject.NewMoney(50000, "IDR")

        err := order.AddItem(1, "Laptop", 2, price)

        require.NoError(t, err)
        assert.Len(t, order.Items(), 1)
        assert.Equal(t, 100000.0, order.TotalAmount().Amount())
    })

    t.Run("duplicate product adds quantity", func(t *testing.T) {
        t.Parallel()

        order := makeValidOrder(t)
        price, _ := valueobject.NewMoney(50000, "IDR")

        _ = order.AddItem(1, "Laptop", 2, price)
        err := order.AddItem(1, "Laptop", 3, price) // tambah lagi

        require.NoError(t, err)
        assert.Len(t, order.Items(), 1, "should still be 1 item")
        assert.Equal(t, 5, order.Items()[0].Quantity(), "quantity should be 5")
        assert.Equal(t, 250000.0, order.TotalAmount().Amount())
    })

    t.Run("cannot add item to non-draft order", func(t *testing.T) {
        t.Parallel()

        order := makeValidOrder(t)
        price, _ := valueobject.NewMoney(50000, "IDR")
        _ = order.AddItem(1, "Laptop", 1, price)
        _ = order.Submit() // pindah dari DRAFT

        err := order.AddItem(2, "Mouse", 1, price)

        require.Error(t, err)
        assert.Contains(t, err.Error(), "DRAFT")
    })

    t.Run("zero quantity rejected", func(t *testing.T) {
        t.Parallel()

        order := makeValidOrder(t)
        price, _ := valueobject.NewMoney(50000, "IDR")

        err := order.AddItem(1, "Laptop", 0, price)

        require.Error(t, err)
        assert.Contains(t, err.Error(), "positive")
    })
}
```

### Test Suite Pattern

```go
// Untuk test yang butuh shared setup/teardown
type UserUseCaseSuite struct {
    suite.Suite
    mockRepo *MockUserRepository
    mockHasher *MockPasswordHasher
    useCase auth.RegisterUseCase
}

// SetupTest dipanggil SEBELUM setiap test method
func (s *UserUseCaseSuite) SetupTest() {
    s.mockRepo = new(MockUserRepository)
    s.mockHasher = new(MockPasswordHasher)
    s.useCase = auth.NewRegisterUseCase(s.mockRepo, s.mockHasher)
}

// TearDownTest dipanggil SETELAH setiap test method
func (s *UserUseCaseSuite) TearDownTest() {
    s.mockRepo.AssertExpectations(s.T())
    s.mockHasher.AssertExpectations(s.T())
}

func (s *UserUseCaseSuite) TestRegister_Success() {
    s.mockRepo.On("ExistsByEmail", mock.Anything, "alice@test.com").Return(false, nil)
    s.mockHasher.On("Hash", "SecurePass123").Return("$2a$14$hash", nil)
    s.mockRepo.On("Save", mock.Anything, mock.MatchedBy(func(u *entity.User) bool {
        return u.Email == "alice@test.com"
    })).Return(nil)

    output, err := s.useCase.Execute(context.Background(), auth.RegisterInput{
        Name:     "Alice",
        Email:    "alice@test.com",
        Password: "SecurePass123",
    })

    s.NoError(err)
    s.NotNil(output)
    s.Equal("Alice", output.User.Name)
}

func (s *UserUseCaseSuite) TestRegister_DuplicateEmail() {
    s.mockRepo.On("ExistsByEmail", mock.Anything, "alice@test.com").Return(true, nil)

    _, err := s.useCase.Execute(context.Background(), auth.RegisterInput{
        Name:     "Alice",
        Email:    "alice@test.com",
        Password: "SecurePass123",
    })

    s.Error(err)
    s.ErrorIs(err, apperror.ErrEmailExists)
}

// Entry point untuk suite
func TestUserUseCaseSuite(t *testing.T) {
    suite.Run(t, new(UserUseCaseSuite))
}
```

### t.Parallel() — Keamanan dan Gotchas

```go
// GOTCHA PALING UMUM dengan t.Parallel(): loop variable capture

// ❌ SALAH: classic loop variable bug
func TestProducts_Bad(t *testing.T) {
    tests := []struct{ id int }{{1}, {2}, {3}}
    for _, tt := range tests {
        t.Run(fmt.Sprintf("id=%d", tt.id), func(t *testing.T) {
            t.Parallel()
            // BUG: semua goroutine capture 'tt' yang sama!
            // Saat goroutine jalan, loop sudah selesai → tt.id = 3 selalu
            result := fetchProduct(tt.id)
            assert.Equal(t, tt.id, result.ID) // FAIL: selalu compare 3
        })
    }
}

// ✅ BENAR: capture loop variable dengan local copy
func TestProducts_Good(t *testing.T) {
    tests := []struct{ id int }{{1}, {2}, {3}}
    for _, tt := range tests {
        tt := tt // ← PENTING: buat copy lokal sebelum goroutine!
        t.Run(fmt.Sprintf("id=%d", tt.id), func(t *testing.T) {
            t.Parallel()
            result := fetchProduct(tt.id)
            assert.Equal(t, tt.id, result.ID) // Sekarang benar
        })
    }
}

// Go 1.22+ fix: loop variable semantics berubah (tidak perlu tt := tt lagi)
// Tapi untuk kompatibilitas, tetap tulis tt := tt

// KAPAN pakai t.Parallel():
// ✅ Test yang independent (tidak share mutable state)
// ✅ Test yang lambat (I/O, sleep, network)
// ❌ Test yang share global state (global variable, singleton)
// ❌ Test yang pakai database rows yang sama

// PASTIKAN test cleanup aman dengan t.Parallel():
func TestWithCleanup(t *testing.T) {
    t.Parallel()

    // Buat resource unik per test (bukan shared!)
    userID := createUniqueUser(t) // ID unik per test run

    t.Cleanup(func() {
        deleteUser(userID) // cleanup hanya resource test ini
    })

    // Test dengan resource yang terisolasi
    result := getUser(userID)
    assert.NotNil(t, result)
}
```


### 🏋️ Latihan 10.2

1. Tulis **table-driven test** lengkap untuk `ValidatePassword` dari Fase 4 value object. Cover: too short, no uppercase, no lowercase, no digit, too common password, valid password. Minimal 8 test cases.
2. Refactor test di Auth Service menggunakan **test suite** (`testify/suite`). Manfaatkan `SetupTest` untuk reset mocks sebelum setiap test, dan `TearDownTest` untuk verify all mock expectations.
3. Buat test untuk `Money.Add()` dengan **parallel sub-tests** yang cover: same currency success, different currency error, zero amount, large amounts, rounding behavior. Verifikasi bahwa t.Parallel() benar-benar jalan paralel.

---

## 📦 Modul 10.3 — Testing dengan DI & Interface

### Test Doubles: Taxonomy

```
TEST DOUBLE = objek pengganti dalam test

DUMMY:    Diisi tapi tidak dipakai (hanya untuk memenuhi parameter)
          var dummy logger = nil

STUB:     Memberikan jawaban yang sudah ditentukan (tidak verify calls)
          stub.FindByEmail returns fixedUser

FAKE:     Implementasi yang benar-benar berjalan tapi tidak cocok untuk prod
          InMemoryRepository (real logic, in-memory storage)

SPY:      Seperti stub tapi record calls untuk bisa di-assert nanti
          spy.Calls() → [[email, "alice@test.com"]]

MOCK:     Seperti spy tapi juga verify expectation (testify/mock)
          mock.On("FindByEmail").Return(user, nil)
          mock.AssertCalled(t, "FindByEmail", ctx, "alice@test.com")
```

### Generating Mocks dengan mockery

```bash
# Install mockery
go install github.com/vektra/mockery/v2@latest

# Generate mock untuk semua interface dalam package
mockery --all --output=mocks --outpkg=mocks

# Generate mock untuk interface tertentu
mockery --name=UserRepository --output=internal/mocks --outpkg=mocks

# Auto-generate saat go generate
# Tambahkan di file interface:
//go:generate mockery --name=UserRepository
```

```go
// internal/repository/user_repository.go
//go:generate mockery --name=UserRepository --output=../mocks --outpkg=mocks
type UserRepository interface {
    Save(ctx context.Context, user *entity.User) error
    FindByID(ctx context.Context, id uint) (*entity.User, error)
    FindByEmail(ctx context.Context, email string) (*entity.User, error)
    ExistsByEmail(ctx context.Context, email string) (bool, error)
    Update(ctx context.Context, user *entity.User) error
    Delete(ctx context.Context, id uint) error
}

// Jalankan: go generate ./...
// Menghasilkan: internal/mocks/UserRepository.go
```

### Fake (Implementasi In-Memory)

```go
// test/fake/user_repository.go
// Fake lebih realistis dari mock — implementasi yang benar tapi in-memory
// Berguna saat logic query cukup kompleks untuk di-mock satu per satu

package fake

import (
    "context"
    "fmt"
    "sync"

    "github.com/kamu/app/internal/domain/entity"
    "github.com/kamu/app/internal/repository"
)

// InMemoryUserRepository mengimplementasikan UserRepository dengan in-memory map
type InMemoryUserRepository struct {
    mu     sync.RWMutex
    users  map[uint]*entity.User
    nextID uint
}

func NewInMemoryUserRepository() *InMemoryUserRepository {
    return &InMemoryUserRepository{
        users:  make(map[uint]*entity.User),
        nextID: 1,
    }
}

func (r *InMemoryUserRepository) Save(ctx context.Context, user *entity.User) error {
    r.mu.Lock()
    defer r.mu.Unlock()

    if user.ID == 0 {
        user.ID = r.nextID // simulate auto-increment
        r.nextID++
    }

    // Deep copy untuk menghindari shared pointer issues
    copied := *user
    r.users[user.ID] = &copied
    return nil
}

func (r *InMemoryUserRepository) FindByID(ctx context.Context, id uint) (*entity.User, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()

    user, ok := r.users[id]
    if !ok {
        return nil, nil
    }
    copied := *user
    return &copied, nil
}

func (r *InMemoryUserRepository) FindByEmail(ctx context.Context, email string) (*entity.User, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()

    for _, u := range r.users {
        if u.Email == email {
            copied := *u
            return &copied, nil
        }
    }
    return nil, nil
}

func (r *InMemoryUserRepository) ExistsByEmail(ctx context.Context, email string) (bool, error) {
    user, err := r.FindByEmail(ctx, email)
    return user != nil, err
}

func (r *InMemoryUserRepository) Update(ctx context.Context, user *entity.User) error {
    r.mu.Lock()
    defer r.mu.Unlock()

    if _, ok := r.users[user.ID]; !ok {
        return fmt.Errorf("user %d not found", user.ID)
    }
    copied := *user
    r.users[user.ID] = &copied
    return nil
}

func (r *InMemoryUserRepository) Delete(ctx context.Context, id uint) error {
    r.mu.Lock()
    defer r.mu.Unlock()
    delete(r.users, id)
    return nil
}

// Helper methods untuk test setup
func (r *InMemoryUserRepository) Seed(users ...*entity.User) {
    for _, u := range users {
        r.Save(context.Background(), u)
    }
}

func (r *InMemoryUserRepository) Count() int {
    r.mu.RLock()
    defer r.mu.RUnlock()
    return len(r.users)
}

func (r *InMemoryUserRepository) Clear() {
    r.mu.Lock()
    defer r.mu.Unlock()
    r.users = make(map[uint]*entity.User)
    r.nextID = 1
}

// Pastikan interface terpenuhi (compile-time check)
var _ repository.UserRepository = (*InMemoryUserRepository)(nil)
```

### Mock dengan Advanced Matchers

```go
// Menggunakan mock.MatchedBy untuk validasi argument yang kompleks
func TestRegisterUseCase_SavesCorrectUser(t *testing.T) {
    mockRepo := new(mocks.UserRepository)
    mockHasher := new(mocks.PasswordHasher)

    // MatchedBy: validasi argument dengan fungsi custom
    mockRepo.On("ExistsByEmail", mock.Anything, mock.AnythingOfType("string")).
        Return(false, nil)

    mockHasher.On("Hash", mock.AnythingOfType("string")).
        Return("$2a$14$hashedpassword", nil)

    // Validasi bahwa user yang di-save punya properties yang benar
    mockRepo.On("Save", mock.Anything,
        mock.MatchedBy(func(user *entity.User) bool {
            // Validasi kondisi spesifik
            return user.Name == "Alice" &&
                user.Email == "alice@test.com" &&
                user.Role == entity.RoleUser &&
                user.IsActive == true &&
                user.PasswordHash != "" && // hashed, bukan plain
                user.PasswordHash != "SecurePass123" // bukan plain text!
        }),
    ).Return(nil)

    uc := auth.NewRegisterUseCase(mockRepo, mockHasher)
    _, err := uc.Execute(context.Background(), auth.RegisterInput{
        Name:     "Alice",
        Email:    "alice@test.com",
        Password: "SecurePass123",
    })

    require.NoError(t, err)
    mockRepo.AssertExpectations(t)
    mockHasher.AssertExpectations(t)
}
```

### testify/mock vs gomock — Kapan Pakai Mana?

```
TESTIFY/MOCK (yang sudah kita pakai):
  Pros:
    + Lebih fleksibel — bisa set expectation kapan saja
    + Error messages lebih readable
    + Tidak butuh codegen untuk setup
    + Cocok untuk test yang lebih behavior-focused
  Cons:
    - Tidak compile-time safe (typo di method name baru ketahuan saat run)
    - Mock setup lebih verbose

GOMOCK (alternatif populer):
  Pros:
    + Compile-time type-safe (typo = compile error)
    + Strict mode: unexpected calls = test fail
    + InOrder untuk verify urutan calls
  Cons:
    - Butuh mockgen untuk generate (lebih banyak setup)
    - API lebih kompleks

REKOMENDASI:
  Fase awal / tim kecil → testify/mock (lebih mudah dipelajari)
  Strict ordering penting → gomock
  Large codebase / type safety penting → gomock
```

```go
// gomock example untuk perbandingan:
// install: go install github.com/golang/mock/mockgen@latest
// generate: mockgen -source=internal/repository/user_repository.go -destination=mocks/user_repository.go

import "github.com/golang/mock/gomock"

func TestLoginUseCase_WithGoMock(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish() // verify all expectations

    mockRepo := mocks.NewMockUserRepository(ctrl)
    mockHasher := mocks.NewMockPasswordHasher(ctrl)

    // Expectation: FindByEmail HARUS dipanggil tepat 1x dengan argument ini
    mockRepo.EXPECT().
        FindByEmail(gomock.Any(), "alice@test.com").
        Return(&entity.User{ID: 1, Email: "alice@test.com"}, nil).
        Times(1)

    // Expectation dengan urutan (InOrder)
    gomock.InOrder(
        mockHasher.EXPECT().Verify("SecurePass123", "hashedpass").Return(true),
        mockRepo.EXPECT().UpdateLastLogin(gomock.Any(), uint(1)).Return(nil),
    )

    uc := auth.NewLoginUseCase(mockRepo, mockHasher)
    _, err := uc.Execute(context.Background(), auth.LoginInput{
        Email:    "alice@test.com",
        Password: "SecurePass123",
    })

    require.NoError(t, err)
    // ctrl.Finish() akan verify semua expectations terpenuhi
}
```


### 🏋️ Latihan 10.3

1. Buat `InMemoryOrderRepository` (fake) yang mengimplementasikan `OrderCommandRepository`. Tambahkan method helper: `Seed(orders...)`, `Count()`, `GetAll()`. Tulis test untuk `CreateOrderUseCase` menggunakan fake repository.
2. Generate mocks untuk semua repository interfaces menggunakan `mockery`. Tulis test `LoginUseCase` dengan mocks yang verifikasi: (a) `FindByEmail` dipanggil dengan email yang tepat, (b) password di-verify, (c) token di-generate, (d) refresh token di-save.
3. Buat `SpyEventPublisher` yang merekam semua events yang dipublish. Gunakan spy ini untuk verifikasi bahwa `ConfirmOrderUseCase` mempublish event `OrderConfirmed` dengan data yang benar setelah order dikonfirmasi.

---

## 📦 Modul 10.4 — Integration Testing

### Mengapa testcontainers?

```
MASALAH dengan shared test database:
  - Test saling mengotori data
  - Urutan test berpengaruh
  - Tidak bisa parallel
  - Environment berbeda beda (local vs CI)

SOLUSI: testcontainers-go
  - Start PostgreSQL container BARU untuk setiap test suite
  - Data fresh, tidak ada state dari test lain
  - Identik di local dan CI
  - Cleanup otomatis setelah test
```

```bash
go get github.com/testcontainers/testcontainers-go
go get github.com/testcontainers/testcontainers-go/modules/postgres
```

### Setup testcontainers

```go
// test/integration/setup_test.go
package integration_test

import (
    "context"
    "fmt"
    "testing"
    "time"

    "github.com/stretchr/testify/require"
    "github.com/testcontainers/testcontainers-go"
    "github.com/testcontainers/testcontainers-go/modules/postgres"
    "github.com/testcontainers/testcontainers-go/wait"
    pgdriver "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

// TestMain adalah entry point untuk integration test package
func TestMain(m *testing.M) {
    // Jalankan tests
    os.Exit(m.Run())
}

// SetupTestDatabase membuat database PostgreSQL sementara untuk testing
// Panggil di awal setiap test suite
func SetupTestDatabase(t *testing.T) *gorm.DB {
    t.Helper()

    ctx := context.Background()

    // Start PostgreSQL container
    pgContainer, err := postgres.RunContainer(ctx,
        testcontainers.WithImage("postgres:16-alpine"),
        postgres.WithDatabase("testdb"),
        postgres.WithUsername("testuser"),
        postgres.WithPassword("testpass"),
        testcontainers.WithWaitStrategy(
            wait.ForLog("database system is ready to accept connections").
                WithOccurrence(2).
                WithStartupTimeout(30*time.Second),
        ),
    )
    require.NoError(t, err, "failed to start postgres container")

    // Cleanup saat test selesai
    t.Cleanup(func() {
        if err := pgContainer.Terminate(ctx); err != nil {
            t.Logf("failed to terminate container: %v", err)
        }
    })

    // Get connection string
    dsn, err := pgContainer.ConnectionString(ctx, "sslmode=disable")
    require.NoError(t, err)

    // Connect dengan GORM
    db, err := gorm.Open(pgdriver.Open(dsn), &gorm.Config{})
    require.NoError(t, err)

    // Run migrations
    err = db.AutoMigrate(
        &model.User{},
        &model.RefreshToken{},
    )
    require.NoError(t, err)

    return db
}

// CleanupTables membersihkan semua data antar test
func CleanupTables(t *testing.T, db *gorm.DB, tables ...string) {
    t.Helper()
    for _, table := range tables {
        db.Exec(fmt.Sprintf("TRUNCATE TABLE %s RESTART IDENTITY CASCADE", table))
    }
}
```

### Integration Test untuk Repository

```go
// test/integration/user_repository_test.go
package integration_test

import (
    "context"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    "github.com/kamu/app/internal/domain/entity"
    "github.com/kamu/app/internal/infrastructure/persistence/postgres"
)

func TestUserRepository_Integration(t *testing.T) {
    // Setup: buat database container SEKALI untuk semua sub-test ini
    db := SetupTestDatabase(t)
    repo := postgres.NewUserRepository(db)
    ctx := context.Background()

    // Cleanup sebelum setiap sub-test
    cleanup := func() { CleanupTables(t, db, "users") }

    t.Run("Save and FindByID", func(t *testing.T) {
        cleanup()

        // Arrange
        user, err := entity.NewUser("Alice", "alice@test.com", "$2a$14$hash")
        require.NoError(t, err)

        // Act: Save
        err = repo.Save(ctx, user)
        require.NoError(t, err)
        assert.NotZero(t, user.ID, "ID should be assigned after Save")

        // Act: FindByID
        found, err := repo.FindByID(ctx, user.ID)
        require.NoError(t, err)
        require.NotNil(t, found)

        // Assert
        assert.Equal(t, user.ID, found.ID)
        assert.Equal(t, "Alice", found.Name)
        assert.Equal(t, "alice@test.com", found.Email)
        assert.Equal(t, entity.RoleUser, found.Role)
        assert.True(t, found.IsActive)
    })

    t.Run("FindByEmail returns nil for non-existent", func(t *testing.T) {
        cleanup()

        found, err := repo.FindByEmail(ctx, "nobody@test.com")

        require.NoError(t, err)
        assert.Nil(t, found, "should return nil, not error, for not found")
    })

    t.Run("ExistsByEmail", func(t *testing.T) {
        cleanup()

        // Arrange: simpan user
        user, _ := entity.NewUser("Bob", "bob@test.com", "$2a$14$hash")
        repo.Save(ctx, user)

        // Assert: email yang ada
        exists, err := repo.ExistsByEmail(ctx, "bob@test.com")
        require.NoError(t, err)
        assert.True(t, exists)

        // Assert: email yang tidak ada
        notExists, err := repo.ExistsByEmail(ctx, "nobody@test.com")
        require.NoError(t, err)
        assert.False(t, notExists)
    })

    t.Run("unique email constraint", func(t *testing.T) {
        cleanup()

        user1, _ := entity.NewUser("Alice", "alice@test.com", "$2a$14$hash")
        err := repo.Save(ctx, user1)
        require.NoError(t, err)

        // Duplikat email → harus error
        user2, _ := entity.NewUser("Alice2", "alice@test.com", "$2a$14$hash")
        err = repo.Save(ctx, user2)
        assert.Error(t, err, "duplicate email should return error")
    })

    t.Run("soft delete", func(t *testing.T) {
        cleanup()

        user, _ := entity.NewUser("Charlie", "charlie@test.com", "$2a$14$hash")
        repo.Save(ctx, user)

        // Delete
        err := repo.Delete(ctx, user.ID)
        require.NoError(t, err)

        // Tidak bisa ditemukan setelah delete (soft delete)
        found, err := repo.FindByID(ctx, user.ID)
        require.NoError(t, err)
        assert.Nil(t, found, "soft deleted user should not be found")
    })
}
```

### 🏋️ Latihan 10.4

1. Buat **integration test** lengkap untuk `OrderRepository`: test Save (new + existing), FindByID, FindByIDForUpdate (cek row lock), GetOrderList dengan filters. Gunakan testcontainers.
2. Buat **database transaction test**: verifikasi bahwa Outbox Pattern benar-benar transaksional — jika save order berhasil tapi outbox gagal (simulasi error), kedua operasi harus rollback.
3. Setup **test database helper** yang reusable: `TestDB(t)` yang start container sekali per test binary (bukan per test), menggunakan `sync.Once` dan `TestMain` untuk efisiensi.

---

## 📦 Modul 10.5 — HTTP Handler Testing

### httptest Package

```go
// internal/delivery/http/handler/auth_handler_test.go
package handler_test

import (
    "bytes"
    "context"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"

    "github.com/gin-gonic/gin"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    "github.com/kamu/app/internal/delivery/http/handler"
    "github.com/kamu/app/internal/mocks"
    authuc "github.com/kamu/app/internal/usecase/auth"
)

func init() {
    gin.SetMode(gin.TestMode) // matikan debug output
}

// setupRouter adalah helper yang buat router dengan mock dependencies
func setupAuthRouter(t *testing.T) (*gin.Engine, *mocks.RegisterUseCase, *mocks.LoginUseCase) {
    t.Helper()

    mockRegister := new(mocks.RegisterUseCase)
    mockLogin := new(mocks.LoginUseCase)

    h := handler.NewAuthHandler(mockRegister, mockLogin, nil, nil)

    r := gin.New()
    r.POST("/auth/register", h.Register)
    r.POST("/auth/login", h.Login)

    return r, mockRegister, mockLogin
}

// httpPost adalah helper untuk buat HTTP POST request
func httpPost(t *testing.T, router *gin.Engine, path string, body interface{}) *httptest.ResponseRecorder {
    t.Helper()

    bodyBytes, err := json.Marshal(body)
    require.NoError(t, err)

    req := httptest.NewRequest(http.MethodPost, path, bytes.NewBuffer(bodyBytes))
    req.Header.Set("Content-Type", "application/json")

    w := httptest.NewRecorder()
    router.ServeHTTP(w, req)

    return w
}

func TestAuthHandler_Register(t *testing.T) {
    t.Run("success returns 201 with user data", func(t *testing.T) {
        router, mockRegister, _ := setupAuthRouter(t)

        // Setup mock expectation
        mockRegister.On("Execute", mock.Anything, authuc.RegisterInput{
            Name:     "Alice",
            Email:    "alice@test.com",
            Password: "SecurePass123",
        }).Return(&authuc.RegisterOutput{
            User: &entity.User{ID: 1, Name: "Alice", Email: "alice@test.com"},
        }, nil)

        // Make request
        w := httpPost(t, router, "/auth/register", map[string]string{
            "name":     "Alice",
            "email":    "alice@test.com",
            "password": "SecurePass123",
        })

        // Assert response
        assert.Equal(t, http.StatusCreated, w.Code)

        var resp map[string]interface{}
        require.NoError(t, json.Unmarshal(w.Body.Bytes(), &resp))
        assert.True(t, resp["success"].(bool))

        data := resp["data"].(map[string]interface{})
        assert.Equal(t, float64(1), data["id"])
        assert.Equal(t, "Alice", data["name"])
        // Password tidak boleh ada di response!
        assert.NotContains(t, data, "password")
        assert.NotContains(t, data, "password_hash")

        mockRegister.AssertExpectations(t)
    })

    t.Run("validation error returns 400", func(t *testing.T) {
        router, _, _ := setupAuthRouter(t)

        w := httpPost(t, router, "/auth/register", map[string]string{
            "name":     "A", // terlalu pendek
            "email":    "not-an-email",
            "password": "123",
        })

        assert.Equal(t, http.StatusBadRequest, w.Code)

        var resp map[string]interface{}
        json.Unmarshal(w.Body.Bytes(), &resp)
        assert.False(t, resp["success"].(bool))

        errObj := resp["error"].(map[string]interface{})
        assert.Equal(t, "VALIDATION_ERROR", errObj["code"])
        // Verifikasi detail validasi per field
        details := errObj["details"].(map[string]interface{})
        assert.Contains(t, details, "name")
        assert.Contains(t, details, "email")
    })

    t.Run("duplicate email returns 409", func(t *testing.T) {
        router, mockRegister, _ := setupAuthRouter(t)

        mockRegister.On("Execute", mock.Anything, mock.Anything).
            Return(nil, apperror.ErrEmailExists)

        w := httpPost(t, router, "/auth/register", map[string]string{
            "name":     "Alice",
            "email":    "alice@test.com",
            "password": "SecurePass123",
        })

        assert.Equal(t, http.StatusConflict, w.Code)

        var resp map[string]interface{}
        json.Unmarshal(w.Body.Bytes(), &resp)
        errObj := resp["error"].(map[string]interface{})
        assert.Equal(t, "EMAIL_EXISTS", errObj["code"])
    })
}
```

### Golden File Testing

```go
// Golden files: simpan expected response di file, compare saat test
// Berguna untuk complex response yang susah di-assert inline

// test/testdata/golden/get_product_response.json
// {
//   "success": true,
//   "data": {
//     "id": 1,
//     "name": "Laptop Gaming",
//     ...
//   }
// }

func TestProductHandler_Get_GoldenFile(t *testing.T) {
    router, _ := setupProductRouter(t)

    w := httptest.NewRecorder()
    req := httptest.NewRequest("GET", "/products/1", nil)
    router.ServeHTTP(w, req)

    require.Equal(t, http.StatusOK, w.Code)

    goldenFile := "testdata/golden/get_product_response.json"

    if os.Getenv("UPDATE_GOLDEN") == "true" {
        // Update golden file dengan response aktual
        os.WriteFile(goldenFile, w.Body.Bytes(), 0644)
        t.Logf("Updated golden file: %s", goldenFile)
        return
    }

    // Compare dengan golden file
    expected, err := os.ReadFile(goldenFile)
    require.NoError(t, err)

    assert.JSONEq(t, string(expected), w.Body.String())
}
```

### 🏋️ Latihan 10.5

1. Buat **complete test suite** untuk Auth Handler: test Register (success, validation error, duplicate email), Login (success, wrong password, inactive user), GetMe (success, invalid token). Pastikan setiap test verifikasi: status code, response body structure, error code.
2. Implementasikan **golden file testing** untuk GET /products/:id dan GET /orders response. Buat script `make update-golden` yang menjalankan test dengan `UPDATE_GOLDEN=true` untuk update golden files saat format response berubah.
3. Buat **request helper library** `testhttp` dengan fungsi: `GET(t, router, path, headers)`, `POST(t, router, path, body, headers)`, `PUT(...)`, `DELETE(...)` yang return `*httptest.ResponseRecorder`. Tambahkan helper `DecodeResponse(t, w, &target)` untuk parse response body.

---

## 📦 Modul 10.6 — gRPC Testing

### bufconn — In-Process gRPC Server

```go
// test/grpc/product_handler_test.go
package grpc_test

import (
    "context"
    "net"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
    "google.golang.org/grpc/test/bufconn"

    productv1 "github.com/kamu/app/proto/gen/product/v1"
    grpchandler "github.com/kamu/app/internal/delivery/grpc/handler"
    "github.com/kamu/app/internal/mocks"
)

const bufSize = 1024 * 1024

// setupGRPCServer membuat in-process gRPC server untuk testing
// Menggunakan bufconn: network yang hanya ada di memori (tidak pakai port nyata)
func setupGRPCServer(t *testing.T) productv1.ProductServiceClient {
    t.Helper()

    // Setup mock use cases
    mockGetProduct := new(mocks.GetProductUseCase)
    mockCreateProduct := new(mocks.CreateProductUseCase)

    // Seed data untuk mock
    mockGetProduct.On("Execute", mock.Anything, productuc.GetProductInput{ID: 1}).
        Return(&productuc.GetProductOutput{
            Product: &entity.Product{
                ID:    1,
                Name:  "Laptop Gaming",
                Price: 15000000,
                Stock: 10,
            },
        }, nil)

    mockGetProduct.On("Execute", mock.Anything, productuc.GetProductInput{ID: 999}).
        Return(nil, apperror.ErrProductNotFound)

    // Buat gRPC server
    server := grpc.NewServer()
    productHandler := grpchandler.NewProductServer(
        mockGetProduct,
        mockCreateProduct,
        nil, nil, nil,
    )
    productv1.RegisterProductServiceServer(server, productHandler)

    // bufconn: listener in-memory (sangat cepat, tidak perlu port)
    lis := bufconn.Listen(bufSize)

    // Start server di goroutine
    go func() {
        if err := server.Serve(lis); err != nil && err != grpc.ErrServerStopped {
            t.Logf("gRPC server error: %v", err)
        }
    }()

    // Cleanup saat test selesai
    t.Cleanup(func() {
        server.GracefulStop()
        lis.Close()
    })

    // Buat client yang terhubung ke bufconn
    conn, err := grpc.DialContext(context.Background(), "bufnet",
        grpc.WithContextDialer(func(ctx context.Context, s string) (net.Conn, error) {
            return lis.Dial()
        }),
        grpc.WithTransportCredentials(insecure.NewCredentials()),
    )
    require.NoError(t, err)

    t.Cleanup(func() { conn.Close() })

    return productv1.NewProductServiceClient(conn)
}

func TestProductService_GetProduct(t *testing.T) {
    client := setupGRPCServer(t)
    ctx := context.Background()

    t.Run("success returns product", func(t *testing.T) {
        resp, err := client.GetProduct(ctx, &productv1.GetProductRequest{Id: 1})

        require.NoError(t, err)
        require.NotNil(t, resp)
        assert.Equal(t, uint64(1), resp.Product.Id)
        assert.Equal(t, "Laptop Gaming", resp.Product.Name)
        assert.Equal(t, float64(15000000), resp.Product.Price)
    })

    t.Run("not found returns NOT_FOUND status", func(t *testing.T) {
        _, err := client.GetProduct(ctx, &productv1.GetProductRequest{Id: 999})

        require.Error(t, err)

        st, ok := status.FromError(err)
        require.True(t, ok)
        assert.Equal(t, codes.NotFound, st.Code())
    })

    t.Run("zero ID returns INVALID_ARGUMENT", func(t *testing.T) {
        _, err := client.GetProduct(ctx, &productv1.GetProductRequest{Id: 0})

        require.Error(t, err)
        st, _ := status.FromError(err)
        assert.Equal(t, codes.InvalidArgument, st.Code())
    })
}
```

### Testing gRPC Streaming

```go
// Test server streaming
func TestProductService_StreamProducts(t *testing.T) {
    client := setupGRPCServer(t)

    // Stream request
    stream, err := client.StreamProductsByCategory(context.Background(),
        &productv1.ListProductsRequest{Category: "electronics"},
    )
    require.NoError(t, err)

    // Collect all streamed products
    var products []*productv1.Product
    for {
        product, err := stream.Recv()
        if err == io.EOF {
            break
        }
        require.NoError(t, err)
        products = append(products, product)
    }

    // Assert
    assert.NotEmpty(t, products)
    for _, p := range products {
        assert.NotEmpty(t, p.Name)
        assert.Positive(t, p.Price)
    }
}
```

### 🏋️ Latihan 10.6

1. Buat **complete gRPC test suite** untuk Product Service: GetProduct (success, not found, invalid ID), CreateProduct (success, validation error, duplicate SKU), UpdateStock (success, insufficient stock). Gunakan bufconn.
2. Test **gRPC interceptors**: buat test yang verifikasi logging interceptor mencatat setiap request, dan auth interceptor menolak request tanpa token dengan `Unauthenticated` status.
3. Test **gRPC streaming**: untuk `BatchCreateProducts` (client streaming), kirim 5 products via stream dan verifikasi semua berhasil dibuat. Test juga saat ada satu item yang invalid — verifikasi partial success di response.

---

## 📦 Modul 10.7 — Contract Testing

### Mengapa Contract Testing?

```
MASALAH di microservices:
  Order Service memanggil Product Service API.
  Jika Product Service ubah response format → Order Service crash!
  
  Integration test tidak cukup: lambat, butuh semua service running.
  Unit test tidak cukup: hanya test service sendiri.

SOLUSI: Consumer-Driven Contract Testing
  1. Consumer (Order Service) mendefinisikan "contract":
     "Saya expect Product Service response punya field: id, name, price"
  2. Provider (Product Service) memverifikasi contract terpenuhi.
  3. Komunikasi lewat contract file (bukan test bersama).
```

### Manual Contract Testing

```go
// Tanpa library Pact — implementasi manual yang sederhana

// contracts/product_service_contract.go
package contracts

// ProductServiceContract mendefinisikan expected behavior Product Service
// Ini adalah "consumer perspective" dari Order Service
type ProductServiceContract struct {
    // Order Service hanya butuh field ini dari Product Service
    // Field lain boleh ada tapi tidak ditest di sini
}

// VerifyGetProductResponse memverifikasi response Product Service
// sesuai dengan yang diharapkan Order Service
func VerifyGetProductResponse(t *testing.T, resp *productv1.GetProductResponse) {
    t.Helper()
    require.NotNil(t, resp, "response must not be nil")
    require.NotNil(t, resp.Product, "product must not be nil")

    // Fields yang WAJIB ada dari sudut pandang Order Service
    assert.Positive(t, resp.Product.Id, "product.id must be positive")
    assert.NotEmpty(t, resp.Product.Name, "product.name must not be empty")
    assert.Positive(t, resp.Product.Price, "product.price must be positive")
    assert.GreaterOrEqual(t, resp.Product.Stock, uint32(0), "product.stock must be non-negative")
}

func VerifyCheckStockResponse(t *testing.T, resp *productv1.CheckStockResponse, requestedQty uint32) {
    t.Helper()
    require.NotNil(t, resp)

    // Contract: response harus bilang available atau tidak
    // Jika available=false, current_stock harus di-return
    if !resp.Available {
        assert.Less(t, resp.CurrentStock, requestedQty,
            "if not available, current_stock should be less than requested")
    }
}
```

```go
// contracts/product_service_contract_test.go
// Di-run sebagai part of Product Service test suite
// Memverifikasi Product Service memenuhi contract yang diharapkan consumer

package contracts_test

func TestProductServiceFulfillsOrderServiceContract(t *testing.T) {
    // Setup real Product Service (atau integration test dengan DB)
    client := setupIntegrationGRPCClient(t)

    // Seed data yang diperlukan
    createTestProduct(t, client)

    t.Run("GetProduct fulfills contract", func(t *testing.T) {
        resp, err := client.GetProduct(ctx, &productv1.GetProductRequest{Id: 1})
        require.NoError(t, err)

        // Verifikasi contract (dari Order Service perspective)
        contracts.VerifyGetProductResponse(t, resp)
    })

    t.Run("CheckStock fulfills contract", func(t *testing.T) {
        resp, err := client.CheckStock(ctx, &productv1.CheckStockRequest{
            ProductId: 1,
            Quantity:  5,
        })
        require.NoError(t, err)

        contracts.VerifyCheckStockResponse(t, resp, 5)
    })
}
```

### Event Contract Testing

```go
// Event schema contract — pastikan producer publish format yang benar

// contracts/order_events_contract.go
func VerifyOrderConfirmedEvent(t *testing.T, eventBytes []byte) {
    t.Helper()

    var event events.OrderConfirmedEvent
    require.NoError(t, json.Unmarshal(eventBytes, &event),
        "event should be valid JSON matching OrderConfirmedEvent schema")

    // Required fields dari consumer perspective
    assert.NotEmpty(t, event.EventID, "event_id required")
    assert.Equal(t, "order.confirmed", event.EventType, "wrong event_type")
    assert.Equal(t, 1, event.Version, "version must be 1")
    assert.NotEmpty(t, event.AggregateID, "aggregate_id required")
    assert.False(t, event.OccurredAt.IsZero(), "occurred_at required")
    assert.Positive(t, event.CustomerID, "customer_id required")
    assert.Positive(t, event.TotalAmount, "total_amount must be positive")
    assert.NotEmpty(t, event.Items, "items must not be empty")

    for i, item := range event.Items {
        assert.Positive(t, item.ProductID, "item[%d].product_id required", i)
        assert.Positive(t, item.Quantity, "item[%d].quantity must be positive", i)
        assert.Positive(t, item.UnitPrice, "item[%d].unit_price must be positive", i)
    }
}
```

### 🏋️ Latihan 10.7

1. Buat **contract file** untuk semua gRPC calls antara Order Service dan Product Service. Tulis `VerifyXxx` functions untuk setiap contract. Jalankan di Product Service test suite untuk verifikasi contract terpenuhi.
2. Buat **event schema contract tests** untuk semua events yang dipublish Order Service (OrderCreated, OrderConfirmed, OrderCancelled). Verifikasi bahwa format event yang di-publish sesuai contract yang diharapkan Notification Service.
3. Implementasikan **contract versioning**: buat `ContractV1` dan `ContractV2` untuk `GetProduct` response. Test bahwa Product Service bisa serve keduanya (untuk saat ada schema evolution yang backward compatible).

---

## 📦 Modul 10.8 — Load & Performance Testing

### Benchmark Test Go

```go
// Benchmark: mengukur performance fungsi secara langsung

// internal/domain/valueobject/money_test.go
func BenchmarkMoney_Add(b *testing.B) {
    m1, _ := valueobject.NewMoney(50000, "IDR")
    m2, _ := valueobject.NewMoney(25000, "IDR")

    b.ResetTimer() // mulai timer setelah setup

    for i := 0; i < b.N; i++ {
        _, _ = m1.Add(m2)
    }
}

func BenchmarkMoney_Add_Parallel(b *testing.B) {
    m1, _ := valueobject.NewMoney(50000, "IDR")
    m2, _ := valueobject.NewMoney(25000, "IDR")

    b.ResetTimer()
    b.RunParallel(func(pb *testing.PB) {
        for pb.Next() {
            _, _ = m1.Add(m2)
        }
    })
}

// Jalankan:
// go test -bench=. -benchmem ./internal/domain/...
//
// Output:
// BenchmarkMoney_Add-8       50000000    24.5 ns/op    0 B/op    0 allocs/op
//                                                                └─ zero allocation!
```

### Benchmark dengan benchstat

```bash
# Jalankan benchmark beberapa kali untuk statistical accuracy
go test -bench=BenchmarkMoney_Add -count=10 -benchmem > before.txt

# Buat optimasi
# ...

# Jalankan lagi
go test -bench=BenchmarkMoney_Add -count=10 -benchmem > after.txt

# Compare
go install golang.org/x/perf/cmd/benchstat@latest
benchstat before.txt after.txt

# Output:
# name         old time/op  new time/op  delta
# Money_Add-8  24.5ns ± 2%  15.2ns ± 1%  -38.0%  (p=0.000)
# name         old allocs   new allocs   delta
# Money_Add-8  0.00 ± 0%    0.00 ± 0%   ~  (all equal)
```

### k6 untuk Load Testing

```bash
# Install k6
brew install k6  # macOS
# atau download dari https://k6.io/docs/getting-started/installation/
```

```javascript
// load-tests/auth-service.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

// Custom metrics
const errorRate = new Rate('error_rate');
const loginDuration = new Trend('login_duration');

// Test configuration
export const options = {
    stages: [
        { duration: '30s', target: 10  },  // warm up: ramp to 10 users
        { duration: '1m',  target: 50  },  // ramp to 50 users
        { duration: '2m',  target: 50  },  // stay at 50 users (steady state)
        { duration: '30s', target: 0   },  // ramp down
    ],
    thresholds: {
        // SLO: 95% request < 200ms
        'http_req_duration': ['p(95)<200'],
        // Error rate < 1%
        'error_rate': ['rate<0.01'],
        // Login khusus: p99 < 500ms
        'login_duration': ['p(99)<500'],
    },
};

const BASE_URL = __ENV.BASE_URL || 'http://localhost:8080';

export function setup() {
    // Buat test user yang akan dipakai untuk login
    const res = http.post(`${BASE_URL}/api/v1/auth/register`, JSON.stringify({
        name: 'Load Test User',
        email: 'loadtest@test.com',
        password: 'LoadTest123',
    }), { headers: { 'Content-Type': 'application/json' } });

    return { email: 'loadtest@test.com', password: 'LoadTest123' };
}

export default function(data) {
    const startTime = new Date();

    // Test: login
    const loginRes = http.post(`${BASE_URL}/api/v1/auth/login`,
        JSON.stringify({ email: data.email, password: data.password }),
        { headers: { 'Content-Type': 'application/json' } }
    );

    loginDuration.add(new Date() - startTime);

    const loginSuccess = check(loginRes, {
        'login status is 200': (r) => r.status === 200,
        'login returns token': (r) => {
            try {
                const body = JSON.parse(r.body);
                return body.data && body.data.access_token;
            } catch {
                return false;
            }
        },
    });

    errorRate.add(!loginSuccess);

    if (!loginSuccess) {
        console.error(`Login failed: ${loginRes.status} ${loginRes.body}`);
        return;
    }

    const token = JSON.parse(loginRes.body).data.access_token;

    // Test: get profile (authenticated endpoint)
    const profileRes = http.get(`${BASE_URL}/api/v1/auth/me`, {
        headers: { 'Authorization': `Bearer ${token}` },
    });

    check(profileRes, {
        'profile status is 200': (r) => r.status === 200,
        'profile returns user data': (r) => {
            const body = JSON.parse(r.body);
            return body.data && body.data.id;
        },
    });

    sleep(1); // Simulasi user think time
}

export function teardown(data) {
    // Cleanup test data jika perlu
}
```

```bash
# Jalankan load test
k6 run load-tests/auth-service.js

# Output summary:
# scenarios: (100.00%) 1 scenario, 50 max VUs
# data_received:     1.2 MB
# data_sent:         800 KB
# http_req_duration: avg=45ms  min=12ms  med=38ms  max=312ms p(90)=89ms p(95)=124ms p(99)=201ms
# http_reqs:         2145 8.9/s
# error_rate:        0.00%
# ✓ p(95)<200: threshold=200ms, actual=124ms PASSED
# ✓ error_rate<0.01: threshold=0.01, actual=0.00 PASSED
```

### 🏋️ Latihan 10.8

1. Buat **benchmark** untuk semua value object methods (`Money.Add`, `Money.Subtract`, `Email.Validate`). Identifikasi yang paling banyak allocate dengan `-benchmem`. Coba optimasi satu fungsi dan ukur improvement dengan `benchstat`.
2. Buat **k6 load test** untuk checkout flow end-to-end: register → login → browse products → create order → submit order. Set thresholds: p95 < 500ms, error rate < 1%. Jalankan dengan 20 concurrent users selama 2 menit.
3. Buat **performance regression test**: jalankan benchmark sebelum dan sesudah perubahan code. Buat CI step yang fail jika ada regression > 20% pada critical functions.

---

## 📦 Modul 10.9 — Fuzz Testing

### Apa itu Fuzz Testing?

```
Fuzz testing = test dengan INPUT RANDOM yang di-generate otomatis
Tujuan: menemukan edge cases yang tidak terpikirkan developer

Go 1.18+ punya built-in fuzzing!

Cocok untuk:
  - Parsing functions (JSON, URL, query string)
  - Validation functions
  - Encoding/decoding
  - String manipulation
  - Mathematical operations

Contoh bug yang sering ditemukan fuzz:
  - Panic saat input kosong
  - Integer overflow dengan nilai ekstrem
  - Infinite loop dengan input tertentu
  - Crash saat Unicode input
```

### Fuzz Test Dasar

```go
// internal/domain/valueobject/email_fuzz_test.go
package valueobject_test

import (
    "testing"
    "github.com/kamu/app/internal/domain/valueobject"
)

// FuzzNewEmail mencoba berbagai input untuk mencari bug
func FuzzNewEmail(f *testing.F) {
    // Seed corpus: contoh input awal
    f.Add("alice@example.com")
    f.Add("user.name+tag@domain.co.id")
    f.Add("")
    f.Add("not-an-email")
    f.Add("@")
    f.Add("a@b")

    // Fuzzer akan generate input random berdasarkan seeds
    f.Fuzz(func(t *testing.T, input string) {
        // Requirement: NewEmail tidak boleh PANIC (tidak boleh crash)
        // Boleh return error, tapi tidak boleh panic
        defer func() {
            if r := recover(); r != nil {
                t.Errorf("NewEmail(%q) panicked: %v", input, r)
            }
        }()

        email, err := valueobject.NewEmail(input)
        if err != nil {
            // Error is OK — tapi email harus nil jika ada error
            if email != (valueobject.Email{}) {
                t.Errorf("NewEmail(%q) returned non-zero Email with error", input)
            }
            return
        }

        // Jika tidak error: email string harus sama dengan input (setelah normalisasi)
        // dan harus valid kembali jika di-parse ulang
        _, err2 := valueobject.NewEmail(email.String())
        if err2 != nil {
            t.Errorf("NewEmail(email.String()) should succeed, input=%q email=%q err=%v",
                input, email.String(), err2)
        }
    })
}

// FuzzMoney mencoba berbagai amount dan currency
func FuzzNewMoney(f *testing.F) {
    f.Add(0.0, "IDR")
    f.Add(100.0, "IDR")
    f.Add(-1.0, "IDR")
    f.Add(1e15, "IDR")
    f.Add(0.001, "IDR")
    f.Add(100.0, "")
    f.Add(100.0, "INVALID_CURRENCY_CODE_TOO_LONG")

    f.Fuzz(func(t *testing.T, amount float64, currency string) {
        defer func() {
            if r := recover(); r != nil {
                t.Errorf("NewMoney(%v, %q) panicked: %v", amount, currency, r)
            }
        }()

        money, err := valueobject.NewMoney(amount, currency)
        if err != nil {
            return // error adalah valid response
        }

        // Jika berhasil: amount harus >= 0
        if money.Amount() < 0 {
            t.Errorf("NewMoney(%v, %q): amount should never be negative, got %v",
                amount, currency, money.Amount())
        }

        // Currency harus sama dengan input (normalized)
        if money.Currency() == "" {
            t.Errorf("NewMoney(%v, %q): currency should not be empty on success",
                amount, currency)
        }
    })
}
```

### Menjalankan Fuzz Test

```bash
# Jalankan normal (hanya seed corpus):
go test ./internal/domain/valueobject/...

# Jalankan fuzzing mode (terus generate input baru):
go test -fuzz=FuzzNewEmail ./internal/domain/valueobject/

# Fuzz selama 30 detik:
go test -fuzz=FuzzNewEmail -fuzztime=30s ./internal/domain/valueobject/

# Jika fuzz menemukan bug, ia menyimpan input yang bermasalah di:
# testdata/fuzz/FuzzNewEmail/

# Re-run dengan specific failing input:
go test -run=FuzzNewEmail/testdata/fuzz/FuzzNewEmail/abc123 ./...
```

### 🏋️ Latihan 10.9

1. Buat fuzz test untuk `ValidatePassword` dari Fase 4: pastikan tidak pernah panic dengan input apapun. Jalankan selama 60 detik. Dokumentasikan 3 edge case menarik yang ditemukan fuzzer.
2. Buat fuzz test untuk JSON parsing di Event Envelope: `FuzzUnmarshalEvent(f *testing.F)` yang verifikasi bahwa parser tidak panic dan selalu return error yang meaningful (bukan nil) untuk invalid JSON.
3. Buat fuzz test untuk `OrderID` parsing: `FuzzParseOrderID(f *testing.F)` yang verifikasi bahwa parse dari string valid dan kemudian format balik ke string menghasilkan nilai yang sama (round-trip property).

---

## 📦 Modul 10.10 — Mutation Testing

### Apa itu Mutation Testing?

```
Mutation testing: UBAH kode secara kecil (mutasi) dan check apakah test GAGAL

Jika test tetap PASS setelah kode dimutasi → test tidak benar-benar mendeteksi bug!

Contoh mutasi:
  Original: if amount >= 0 {
  Mutant 1: if amount > 0 {      ← apakah test detect ini?
  Mutant 2: if amount <= 0 {     ← apakah test detect ini?
  Mutant 3: return true          ← apakah test detect ini?

Mutation score = (mutants killed / total mutants) * 100%
Target: > 80% mutation score
```

```bash
# Install go-mutesting
go install github.com/zimmski/go-mutesting/cmd/go-mutesting@latest
```

```bash
# Jalankan mutation testing pada package
go-mutesting ./internal/domain/valueobject/...

# Output:
# PASS "github.com/kamu/app/internal/domain/valueobject/money.go:45.1" with mutation "@amount >= 0" -> "@amount > 0"
# FAIL "github.com/kamu/app/internal/domain/valueobject/money.go:45.1" with mutation "@amount >= 0" -> "@amount <= 0"
#
# The mutation score is 0.75 (75%)
```

### Meningkatkan Mutation Score

```go
// SEBELUM — test coverage tinggi tapi mutation score rendah
func TestNewMoney_NegativeAmount(t *testing.T) {
    _, err := valueobject.NewMoney(-1, "IDR")
    assert.Error(t, err) // hanya cek ada error, tidak cek apa errornya
}

// SETELAH — test lebih spesifik, mutation score lebih tinggi
func TestNewMoney_NegativeAmount(t *testing.T) {
    _, err := valueobject.NewMoney(-1, "IDR")
    require.Error(t, err)
    assert.Contains(t, err.Error(), "negative",
        "error should mention negative amount")

    // Test boundary: tepat di boundary
    money, err := valueobject.NewMoney(0, "IDR")
    require.NoError(t, err, "zero should be allowed")
    assert.Equal(t, 0.0, money.Amount())

    // Test just above boundary
    money2, err := valueobject.NewMoney(0.01, "IDR")
    require.NoError(t, err, "positive amount should be allowed")
    assert.Positive(t, money2.Amount())
}
```

### Memahami Hasil Mutation Testing

```
SURVIVED MUTANT = bug yang test kamu TIDAK bisa deteksi!
KILLED MUTANT   = bug yang test kamu berhasil deteksi

Contoh kasus nyata:

Original code:
  func (o *Order) CanCancel() bool {
      return o.status == StatusPending || o.status == StatusConfirmed
  }

Mutant 1 (survived ⚠️):
  return o.status == StatusPending && o.status == StatusConfirmed
  // Test tidak detect → test tidak pernah test case HANYA Confirmed!

Mutant 2 (killed ✅):
  return o.status != StatusPending || o.status == StatusConfirmed
  // Test detect → ada test yang expects Pending = cancelable

CARA IMPROVE: Tambah test yang explicit test setiap kondisi:
  - Cancel saat StatusPending → success
  - Cancel saat StatusConfirmed → success
  - Cancel saat StatusCancelled → error (tidak bisa cancel ulang)
  - Cancel saat StatusDelivered → error
```

### Integrasi Mutation Testing ke CI

```yaml
# .github/workflows/mutation-test.yml
# Jalankan mutation testing weekly (terlalu lambat untuk setiap commit)
name: Mutation Testing
on:
  schedule:
    - cron: '0 3 * * 1'  # setiap Senin jam 3 pagi
  workflow_dispatch:

jobs:
  mutation-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.22'

      - name: Install go-mutesting
        run: go install github.com/zimmski/go-mutesting/cmd/go-mutesting@latest

      - name: Run mutation tests
        run: |
          go-mutesting ./internal/domain/... 2>&1 | tee mutation-report.txt
          SCORE=$(tail -1 mutation-report.txt | grep -oP '\d+\.\d+(?= \()')
          echo "Mutation score: $SCORE"

          # Fail jika score di bawah threshold
          python3 -c "
          score = float('$SCORE')
          if score < 0.75:
              print(f'Mutation score {score} below threshold 0.75')
              exit(1)
          print(f'OK: mutation score {score}')
          "

      - name: Upload report
        uses: actions/upload-artifact@v3
        with:
          name: mutation-report
          path: mutation-report.txt
```

### Praktik: Improve Mutation Score Step by Step

```go
// Workflow yang direkomendasikan:
// 1. Jalankan mutation testing
// 2. Lihat surviving mutants
// 3. Untuk setiap surviving mutant: tambah test yang specifically test kondisi tersebut
// 4. Re-run, ulangi sampai score > 80%

// Contoh: surviving mutant di Order.Submit()
// Original:
func (o *Order) Submit() error {
    if len(o.items) == 0 {  // Mutant: if len(o.items) != 0
        return errors.New("cannot submit empty order")
    }
    if o.status != StatusDraft {  // Mutant: if o.status == StatusDraft
        return errors.New("can only submit draft orders")
    }
    o.status = StatusPending
    return nil
}

// Test yang membunuh mutant ini:
func TestOrder_Submit(t *testing.T) {
    t.Run("fails with empty items", func(t *testing.T) {
        order := newOrderWithNoItems()
        err := order.Submit()
        require.Error(t, err)  // Mutant len(o.items) != 0 → menyebabkan error saat ada items
        assert.Contains(t, err.Error(), "empty")
    })

    t.Run("fails when not draft", func(t *testing.T) {
        order := newConfirmedOrder()
        err := order.Submit()
        require.Error(t, err)  // Mutant == StatusDraft → fail saat status IS draft
        assert.Contains(t, err.Error(), "draft")
    })

    t.Run("succeeds when draft with items", func(t *testing.T) {
        order := newDraftOrderWithItems()
        err := order.Submit()
        require.NoError(t, err)
        assert.Equal(t, StatusPending, order.Status())
    })
}
```


### 🏋️ Latihan 10.10

1. Jalankan mutation testing pada `Order` aggregate. Identifikasi 3 mutant yang berhasil survive (test tidak detect). Tulis test tambahan yang membunuh mutant tersebut.
2. Jalankan mutation testing pada `Money` value object. Target mutation score minimal 80%. Perbaiki test yang lemah.
3. Buat **mutation testing report**: script yang jalankan `go-mutesting` pada semua domain packages dan generate laporan: mutation score per package, top 5 surviving mutants yang paling berbahaya.

---

## 📦 Modul 10.11 — Test Coverage Strategy

### Coverage Tools

```bash
# Jalankan test dengan coverage
go test -cover ./...

# Output:
# ok  github.com/kamu/app/internal/domain/entity   coverage: 87.5% of statements
# ok  github.com/kamu/app/internal/usecase/auth    coverage: 91.2% of statements
# ok  github.com/kamu/app/internal/delivery/http   coverage: 72.3% of statements

# Generate coverage profile
go test -coverprofile=coverage.out ./...

# Lihat dalam HTML (interaktif)
go tool cover -html=coverage.out -o coverage.html
open coverage.html

# Summary per function
go tool cover -func=coverage.out | tail -1
# total:   (statements)   82.4%

# Coverage dengan race detector
go test -race -coverprofile=coverage.out ./...
```

### Coverage Targets yang Realistis

```
JANGAN kejar 100% coverage — itu anti-pattern!
Fokus pada MEANINGFUL coverage, bukan number.

Targets yang masuk akal:
  Domain layer (entity, value object, aggregate): 90%+
    → Business logic paling kritis, harus high coverage
  
  Use Case layer: 85%+
    → Application logic, test dengan mocks
  
  Repository layer: 60–70%
    → Integration test dengan real DB (lebih lambat, lebih sedikit)
  
  Delivery layer (handlers): 70–80%
    → HTTP/gRPC handler test
  
  Infrastructure: 40–60%
    → Kode boilerplate, config, wiring — susah di-test
  
  Main/entry point: Skip
    → Wiring code, tidak perlu test

HINDARI:
  - Test yang hanya untuk coverage (test without assertions!)
  - Mock-heavy test yang tidak test logic sama sekali
  - Skip error paths untuk dapat coverage
```

### Meaningful Coverage dengan -coverprofile

```bash
# Exclude generated code dari coverage
go test -coverprofile=coverage.out \
  -coverpkg=./internal/domain/...,./internal/usecase/... \
  ./...

# Cek coverage per function
go tool cover -func=coverage.out | sort -k 3 -n | head -20
# Menampilkan functions dengan coverage terendah → prioritaskan untuk test
```

### Branch Coverage vs Statement Coverage

```go
// Statement coverage: setiap BARIS kode dieksekusi
// Branch coverage:    setiap CABANG (if/else) dieksekusi

func Classify(n int) string {
    if n > 0 {          // branch: n>0 true, n>0 false
        return "positive"
    } else if n < 0 {   // branch: n<0 true, n<0 false
        return "negative"
    }
    return "zero"
}

// Test dengan hanya n=5:
//   Statement coverage: 66% (baris "return negative" dan "return zero" tidak dieksekusi)
//   Branch coverage: 50% (n>0 true dieksekusi, false tidak)

// Test yang benar untuk 100% branch coverage:
func TestClassify(t *testing.T) {
    assert.Equal(t, "positive", Classify(5))   // n>0 = true
    assert.Equal(t, "negative", Classify(-3))  // n<0 = true
    assert.Equal(t, "zero", Classify(0))       // n>0 = false, n<0 = false
}
```

### Coverage untuk Table-Driven Tests

```go
// Table-driven test biasanya memberikan coverage yang lebih baik
// karena mudah menambah edge cases

func TestValidateEmail(t *testing.T) {
    tests := []struct {
        email   string
        wantErr bool
    }{
        // Happy path
        {"alice@example.com", false},
        {"user.name+tag@domain.co.id", false},

        // Error paths (untuk branch coverage)
        {"", true},           // empty string
        {"@", true},          // no local part
        {"a@", true},         // no domain
        {"a@b", true},        // domain too short
        {"a b@c.com", true},  // space in email
        {"a@c.com.", true},   // trailing dot
    }

    for _, tt := range tests {
        t.Run(tt.email, func(t *testing.T) {
            _, err := valueobject.NewEmail(tt.email)
            if tt.wantErr {
                assert.Error(t, err)
            } else {
                assert.NoError(t, err)
            }
        })
    }
}
```

### Mengecualikan Kode dari Coverage

```go
// Beberapa kode memang tidak perlu di-test:
// - generated code
// - main() entrypoint
// - platform-specific code

// Tandai dengan comment pragma:
func generatedFunc() { //nolint
    // kode generated, skip dari coverage analysis
}

// Di .coveragerc atau Makefile:
// Exclude file tertentu dari coverage report
go test -coverprofile=cov.out ./...
grep -v "_gen.go\|_mock.go\|main.go" cov.out > filtered.out
go tool cover -func=filtered.out
```

### Practical Coverage Workflow

```bash
# 1. Generate coverage dengan HTML viewer
go test -coverprofile=coverage.out -covermode=atomic ./...
go tool cover -html=coverage.out -o coverage.html

# 2. Buka dan identifikasi uncovered lines
open coverage.html  # merah = tidak di-cover, hijau = di-cover

# 3. Focus pada high-value uncovered code:
#    - Error handling paths
#    - Edge cases di domain logic
#    - Security checks

# 4. Jangan cover yang tidak perlu:
#    - Logging statements
#    - Simple getters
#    - Platform initialization code

# Per-package summary (sort by coverage)
go tool cover -func=coverage.out | sort -k3 -n | head -20
# Tampilkan 20 function dengan coverage TERENDAH
```


### 🏋️ Latihan 10.11

1. Generate coverage report untuk Auth Service. Identifikasi: (a) 5 fungsi dengan coverage terendah, (b) apakah ada fungsi penting yang 0%? Buat rencana untuk improve coverage pada fungsi yang paling kritis.
2. Buat **coverage gate** di Makefile: `make test-coverage` yang run semua test dan gagal jika total coverage domain layer < 80%. Integrate ke CI pipeline.
3. Analisis perbedaan antara **line coverage** dan **branch coverage** untuk fungsi `Order.Cancel()`. Tulis test cases yang memastikan setiap branch (setiap kondisi if/else) ter-cover, bukan hanya setiap baris.

---

## 📦 Modul 10.12 — Testing Best Practices & Anti-Patterns

### Anti-Patterns yang Harus Dihindari

```go
// ❌ ANTI-PATTERN 1: Test yang tidak assert apapun
func TestCreateUser_Bad(t *testing.T) {
    user, err := NewUser("Alice", "alice@test.com", "hash")
    _ = user
    _ = err
    // Tidak ada assertion! Coverage naik tapi test tidak berguna
}

// ✅ BENAR: Test yang verify behavior
func TestCreateUser_Good(t *testing.T) {
    user, err := NewUser("Alice", "alice@test.com", "hash")
    require.NoError(t, err)
    assert.Equal(t, "Alice", user.Name)
    assert.True(t, user.IsActive)
}

// ❌ ANTI-PATTERN 2: Test yang tergantung urutan
func TestA(t *testing.T) {
    globalDB.Create(&User{Name: "Alice"}) // efek samping global!
}

func TestB(t *testing.T) {
    count := globalDB.Count(&User{}) // bergantung pada TestA sudah berjalan!
    assert.Equal(t, 1, count)
}

// ✅ BENAR: Test yang isolated
func TestB(t *testing.T) {
    db := SetupTestDatabase(t) // fresh DB
    db.Create(&User{Name: "Bob"})
    count := db.Count(&User{})
    assert.Equal(t, 1, count)
}

// ❌ ANTI-PATTERN 3: Magic numbers tanpa explanation
func TestOrder_TotalCalculation(t *testing.T) {
    order.AddItem(1, "Laptop", 2, price)
    assert.Equal(t, 30000, order.Total()) // kenapa 30000?
}

// ✅ BENAR: Explicit calculation
func TestOrder_TotalCalculation(t *testing.T) {
    unitPrice := 15000
    quantity := 2
    expectedTotal := unitPrice * quantity // 30000

    order.AddItem(1, "Laptop", quantity, Price(unitPrice))
    assert.Equal(t, expectedTotal, order.Total(),
        "total should be unit_price * quantity")
}

// ❌ ANTI-PATTERN 4: Test yang terlalu banyak mock
func TestRegister_Overtested(t *testing.T) {
    // Mock semuanya — test ini tidak benar-benar test logic
    mockValidator.On("Validate", ...).Return(nil)
    mockHasher.On("Hash", ...).Return("hash")
    mockRepo.On("ExistsByEmail", ...).Return(false, nil)
    mockRepo.On("Save", ...).Return(nil)
    mockNotifier.On("SendWelcomeEmail", ...).Return(nil)
    mockAuditLog.On("Log", ...).Return(nil)
    // Mocking terlalu banyak → test hanya test wiring, bukan logic
}
```

### Best Practices

```go
// ✅ BEST PRACTICE 1: Test behavior, not implementation
// Jangan test "method ini dipanggil" — test "behavior ini terjadi"

// ❌ Implementation test
mockRepo.AssertCalled(t, "Save", ...) // just checks "Save was called"

// ✅ Behavior test
user, _ := repo.FindByID(ctx, savedUser.ID)
assert.NotNil(t, user, "user should be persisted after Register")

// ✅ BEST PRACTICE 2: One assertion per test (ideally)
// Test yang gagal harus langsung clear kenapa

// ❌ Terlalu banyak assertion
func TestUser(t *testing.T) {
    user, err := NewUser(...)
    assert.NoError(t, err)
    assert.Equal(t, "Alice", user.Name)
    assert.Equal(t, "alice@test.com", user.Email)
    assert.Equal(t, RoleUser, user.Role)
    assert.True(t, user.IsActive)
    assert.False(t, user.CreatedAt.IsZero())
}
// Jika satu gagal, yang lain tetap jalan → noise

// ✅ Lebih baik: sub-tests
func TestUser(t *testing.T) {
    user, err := NewUser("Alice", "alice@test.com", "hash")
    require.NoError(t, err, "NewUser should succeed")
    require.NotNil(t, user)

    t.Run("has correct name", func(t *testing.T) {
        assert.Equal(t, "Alice", user.Name)
    })
    t.Run("defaults to user role", func(t *testing.T) {
        assert.Equal(t, RoleUser, user.Role)
    })
    t.Run("is active by default", func(t *testing.T) {
        assert.True(t, user.IsActive)
    })
}
```

### 🏋️ Latihan 10.12

1. Audit test suite Auth Service: identifikasi semua anti-patterns (test tanpa assertion, test bergantung state global, magic numbers, over-mocking). Fix semua yang ditemukan.
2. Buat **test quality checklist** dan review test suite Order Service dengan checklist tersebut. Dokumentasikan: (a) test yang paling valuable, (b) test yang bisa dihapus karena tidak memberikan value, (c) test yang perlu diperkuat.
3. Buat panduan **"Test Naming Convention"** untuk tim: kapan pakai `TestFunctionName_Condition_ExpectedBehavior` vs `TestFunctionName` dengan sub-tests. Berikan contoh konkret untuk masing-masing.

---

## 📦 Modul 10.13 — CI/CD Integration untuk Testing

### GitHub Actions untuk Testing

```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  GO_VERSION: '1.22'

jobs:
  # Job 1: Unit & Integration Tests
  test:
    name: Tests
    runs-on: ubuntu-latest

    services:
      # PostgreSQL untuk integration tests
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      # Redis untuk integration tests
      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: true  # cache go modules

      - name: Download dependencies
        run: go mod download

      - name: Run linter
        uses: golangci/golangci-lint-action@v3
        with:
          version: latest

      - name: Run unit tests
        run: |
          go test -v -race -short \
            -coverprofile=coverage.out \
            ./internal/domain/... \
            ./internal/usecase/...

      - name: Run integration tests
        env:
          TEST_DATABASE_URL: "host=localhost user=testuser password=testpass dbname=testdb sslmode=disable"
          TEST_REDIS_URL: "redis://localhost:6379"
        run: |
          go test -v -race \
            -run Integration \
            -coverprofile=integration-coverage.out \
            ./test/integration/...

      - name: Merge coverage reports
        run: |
          go install github.com/wadey/gocovmerge@latest
          gocovmerge coverage.out integration-coverage.out > merged-coverage.out

      - name: Check coverage threshold
        run: |
          COVERAGE=$(go tool cover -func=merged-coverage.out | tail -1 | awk '{print $3}')
          THRESHOLD=75.0
          echo "Coverage: $COVERAGE"
          python3 -c "
          cov = float('$COVERAGE'.strip('%'))
          thr = float('$THRESHOLD')
          if cov < thr:
              print(f'Coverage {cov}% is below threshold {thr}%')
              exit(1)
          print(f'Coverage OK: {cov}% >= {thr}%')
          "

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./merged-coverage.out

  # Job 2: Build verification
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
      - name: Build all services
        run: |
          for service in auth-service product-service order-service api-gateway; do
            echo "Building $service..."
            go build -v ./services/$service/cmd/...
          done

  # Job 3: Security scan
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
      - name: Run govulncheck
        run: |
          go install golang.org/x/vuln/cmd/govulncheck@latest
          govulncheck ./...
```

### Makefile Testing Commands

```makefile
# Makefile — semua testing commands

.PHONY: test test-unit test-integration test-all test-coverage \
        test-race test-fuzz bench lint

# Run unit tests only (fast, no external dependencies)
test-unit:
	go test -v -race -short \
		-coverprofile=coverage.unit.out \
		./internal/domain/... \
		./internal/usecase/...
	@echo "Unit test coverage:"
	@go tool cover -func=coverage.unit.out | tail -1

# Run integration tests (requires docker)
test-integration:
	@echo "Starting test containers..."
	docker compose -f docker-compose.test.yml up -d
	@echo "Waiting for services..."
	sleep 5
	go test -v -race \
		-coverprofile=coverage.integration.out \
		./test/integration/...
	@echo "Stopping test containers..."
	docker compose -f docker-compose.test.yml down -v

# Run all tests
test-all: test-unit test-integration
	@echo "Merging coverage reports..."
	gocovmerge coverage.unit.out coverage.integration.out > coverage.merged.out
	go tool cover -func=coverage.merged.out | tail -1

# Check coverage threshold
test-coverage: test-all
	@COVERAGE=$$(go tool cover -func=coverage.merged.out | tail -1 | awk '{print $$3}'); \
	echo "Total coverage: $$COVERAGE"; \
	python3 -c "import sys; cov=float('$$COVERAGE'.strip('%')); \
		sys.exit(1 if cov < 75 else 0)" || \
	(echo "❌ Coverage below 75%!" && exit 1)
	@echo "✅ Coverage OK!"

# Run with race detector
test-race:
	go test -race ./...

# Run fuzz tests (30 seconds per fuzz function)
test-fuzz:
	go test -fuzz=FuzzNewEmail -fuzztime=30s ./internal/domain/valueobject/
	go test -fuzz=FuzzNewMoney -fuzztime=30s ./internal/domain/valueobject/

# Benchmark
bench:
	go test -bench=. -benchmem -count=3 \
		./internal/domain/... \
		./internal/usecase/...

# Lint
lint:
	golangci-lint run ./...

# Full CI pipeline locally
ci: lint test-all test-coverage
	@echo "✅ All CI checks passed!"
```

### Test Report dengan gotestsum

```bash
# Install gotestsum (better output than go test)
go install gotest.tools/gotestsum@latest

# Run with JUnit output (untuk CI)
gotestsum --junitfile test-results.xml -- -v -race ./...

# Run with GitHub Actions output
gotestsum --format github-actions -- -v -race ./...

# Watch mode (re-run tests saat file berubah)
gotestsum --watch ./...
```

### Test Build Tags — Memisahkan Test berdasarkan Kategori

```go
// Build tags memungkinkan kamu jalankan subset test berdasarkan kebutuhan:
// go test ./...                  → hanya test tanpa build tag (unit tests)
// go test -tags integration ./... → test unit + integration
// go test -tags e2e ./...        → semua test

// internal/repository/user_repository_integration_test.go
//go:build integration
// +build integration

package repository_test

// Test ini HANYA dijalankan dengan: go test -tags integration
// Cocok untuk: test yang butuh external dependencies (database, redis, kafka)

import (
    "testing"
    "github.com/stretchr/testify/require"
)

func TestUserRepository_Integration_Save(t *testing.T) {
    // Setup real database connection
    db := setupRealDatabase(t)
    repo := NewUserRepository(db)

    // Test dengan database sungguhan
    user, err := entity.NewUser("Alice", "alice@test.com", "hash")
    require.NoError(t, err)

    err = repo.Save(context.Background(), user)
    require.NoError(t, err)
    require.NotZero(t, user.ID)
}
```

```go
// test/e2e/checkout_test.go
//go:build e2e

package e2e_test

// Test ini HANYA dijalankan dengan: go test -tags e2e
// Butuh semua service running (docker compose up)

func TestCheckoutFlow_E2E(t *testing.T) {
    baseURL := os.Getenv("E2E_BASE_URL")
    if baseURL == "" {
        t.Skip("E2E_BASE_URL not set, skipping e2e tests")
    }

    // 1. Register user
    // 2. Login
    // 3. Create order
    // 4. Submit order
    // 5. Verify order status
}
```

```makefile
# Makefile — run berbagai level tests
test-unit:
    go test -v -race -short ./...

test-integration:
    go test -v -race -tags integration ./...

test-e2e:
    E2E_BASE_URL=http://localhost:8080 go test -v -tags e2e ./test/e2e/...

test-all: test-unit test-integration
```

```yaml
# GitHub Actions — jalankan integration test dengan service containers
test-integration:
  services:
    postgres:
      image: postgres:16-alpine
  steps:
    - name: Run integration tests
      run: go test -v -race -tags integration ./...
```


### 🏋️ Latihan 10.13

1. Setup **GitHub Actions workflow** yang: (a) run linter, (b) run unit tests dengan race detector, (c) run integration tests dengan postgres service container, (d) check coverage > 75%, (e) build semua service. Pastikan semua step berhasil di repository kamu.
2. Buat **test report dashboard**: gunakan `gotestsum --junitfile` untuk generate JUnit XML, lalu tampilkan hasilnya di GitHub Actions summary. Tambahkan badge coverage di README.
3. Implementasikan **test gates**: buat pre-commit hook (`.git/hooks/pre-commit`) yang run unit tests sebelum setiap commit. Jika test gagal, commit dibatalkan. Buat juga `pre-push` hook yang run full test suite.

---

## 🎯 Review & Checkpoint Fase 10

### Konseptual
- [ ] Jelaskan perbedaan antara Mock, Stub, Fake, dan Spy
- [ ] Kapan pakai unit test vs integration test vs e2e test?
- [ ] Apa itu mutation testing dan mengapa coverage 100% tidak cukup?
- [ ] Apa itu consumer-driven contract testing dan masalah apa yang diselesaikannya?
- [ ] Apa itu fuzz testing dan kapan berguna?
- [ ] Mengapa test yang hanya meningkatkan coverage tapi tidak punya assertions berbahaya?

### Praktis
- [ ] Unit tests dengan table-driven, parallel, sub-tests berjalan
- [ ] Mocks di-generate dengan mockery dan digunakan dengan benar
- [ ] Integration tests dengan testcontainers berjalan
- [ ] HTTP handler tests dengan httptest lengkap
- [ ] gRPC tests dengan bufconn berjalan
- [ ] Load test k6 dengan thresholds berhasil
- [ ] Fuzz tests berjalan tanpa panic
- [ ] GitHub Actions CI pipeline green
- [ ] **Menyelesaikan project Comprehensive Test Suite**

---

## 🎯 Project Akhir Fase 10

Kerjakan project berdasarkan PRD di: **`FASE-10-PRD-Test-Suite.md`**

---

*Setelah selesai Fase 10, lanjut ke `FASE-11-DevOps-Deployment.md` — fase terakhir!*

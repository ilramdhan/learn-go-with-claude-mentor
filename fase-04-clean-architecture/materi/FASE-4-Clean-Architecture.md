# 📘 FASE 4: Clean Architecture dengan Gin

> **Prasyarat:** Selesaikan Fase 3 — Blog API  
> **Durasi:** 4–5 minggu  
> **Project Akhir:** User Auth Service dengan Clean Architecture  
> **Tujuan:** Memahami dan mengimplementasikan Clean Architecture agar kode mudah ditest, dimaintain, dan diubah tanpa efek domino

---

## 🗂️ Daftar Modul

| # | Modul | Topik |
|---|-------|-------|
| 4.1 | Mengapa Clean Architecture? | Masalah spaghetti code, prinsip SOLID |
| 4.2 | Lapisan Clean Architecture | Entity, UseCase, Repository, Delivery |
| 4.3 | Project Structure | Struktur folder industri |
| 4.4 | Domain Layer | Entity, Value Object, Domain Error |
| 4.5 | Repository Layer | Interface + implementasi |
| 4.6 | Use Case Layer | Business logic, orchestration |
| 4.7 | Delivery Layer | HTTP handler, request/response DTO |
| 4.8 | Dependency Injection | Manual DI, wire-up di main.go |
| 4.9 | Unit Testing per Layer | Mock, testify, test isolation |
| 4.10 | Integration Testing | Database test, httptest |
| 4.11 | Graceful Shutdown | Signal handling, timeout |
| 4.12 | Health Check & Readiness | /health, /ready endpoint |

---

## 📦 Modul 4.1 — Mengapa Clean Architecture?

### Masalah di Kode Tanpa Arsitektur

Bayangkan kamu menulis handler seperti ini (kode yang **BURUK**):

```go
// ❌ BAD: Semua logic tercampur di handler
func CreateUserHandler(c *gin.Context) {
    var req struct {
        Name     string `json:"name"`
        Email    string `json:"email"`
        Password string `json:"password"`
    }
    c.ShouldBindJSON(&req)

    // Validasi di handler
    if req.Email == "" {
        c.JSON(400, gin.H{"error": "email required"})
        return
    }

    // Hashing password di handler
    hashedPwd, _ := bcrypt.GenerateFromPassword([]byte(req.Password), 14)

    // Query database langsung di handler
    var existingUser User
    db.Where("email = ?", req.Email).First(&existingUser)
    if existingUser.ID != 0 {
        c.JSON(409, gin.H{"error": "email already exists"})
        return
    }

    // Buat user di handler
    user := User{
        Name:     req.Name,
        Email:    req.Email,
        Password: string(hashedPwd),
    }
    db.Create(&user)

    // Kirim email welcome di handler (!)
    sendWelcomeEmail(user.Email, user.Name)

    c.JSON(201, gin.H{"user": user})
}
```

**Masalahnya:**
1. **Tidak bisa ditest** — `db` dan `sendWelcomeEmail` hardcoded, tidak bisa di-mock
2. **Coupling tinggi** — ubah database = ubah handler = ubah test
3. **Business logic tersebar** — logika "email harus unik" ada di HTTP handler, bukan di domain
4. **Tidak bisa reuse** — mau pakai logic yang sama di gRPC? Harus tulis ulang
5. **Sulit dibaca** — satu fungsi 50+ baris yang melakukan segalanya

### Solusi: Clean Architecture

Robert C. Martin (Uncle Bob) memperkenalkan **Clean Architecture** dengan prinsip utama:

```
┌────────────────────────────────────────────┐
│              Delivery Layer                │  ← HTTP, gRPC, CLI, WebSocket
│  ┌──────────────────────────────────────┐  │
│  │           Use Case Layer             │  │  ← Business Logic / Application Rules
│  │  ┌────────────────────────────────┐  │  │
│  │  │       Repository Layer         │  │  │  ← Interface untuk data access
│  │  │  ┌──────────────────────────┐  │  │  │
│  │  │  │      Domain Layer        │  │  │  │  ← Entity, Value Object, Rules
│  │  │  └──────────────────────────┘  │  │  │
│  │  └────────────────────────────────┘  │  │
│  └──────────────────────────────────────┘  │
└────────────────────────────────────────────┘
```

**Aturan Utama: Dependency Rule**
> Panah ketergantungan hanya boleh mengarah ke dalam.
> Layer luar boleh tahu layer dalam, tapi layer dalam TIDAK BOLEH tahu layer luar.

Artinya:
- Domain tidak tahu ada HTTP
- Use Case tidak tahu ada Gin
- Repository interface ada di dalam, implementasinya di luar
- Use Case tidak tahu database apa yang dipakai

### Prinsip SOLID

```
S — Single Responsibility  : Satu class/fungsi, satu tanggung jawab
O — Open/Closed            : Terbuka untuk ekstensi, tertutup untuk modifikasi
L — Liskov Substitution    : Subtipe harus bisa menggantikan supertipenya
I — Interface Segregation  : Interface kecil lebih baik dari interface besar
D — Dependency Inversion   : Depend pada abstraksi (interface), bukan konkret
```

---


### 🏋️ Latihan 4.1

1. Ambil kode handler dari Fase 3 Blog API yang masih "fat handler" (logic di handler). Identifikasi semua pelanggaran SOLID di dalamnya. Tulis daftar masalahnya sebelum refactor.
2. Gambar diagram dependency untuk kode Blog API yang sudah ada. Tandai mana dependency yang mengarah ke arah yang salah (arah keluar bukan ke dalam).
3. Tentukan Bounded Context untuk sistem e-commerce sederhana: User, Product, Order, Payment, Notification. Gambar batasannya dan tentukan aggregate di setiap context.

## 📦 Modul 4.2 — Lapisan Clean Architecture

### Penjelasan Setiap Layer

```
DOMAIN LAYER (paling dalam — tidak bergantung pada siapapun)
├── entity/         ← Objek bisnis inti (User, Product, Order)
├── valueobject/    ← Objek tidak bermutasi (Email, Money, PhoneNumber)
└── errors/         ← Domain-specific errors

REPOSITORY LAYER (interface saja — implementasi di luar)
└── repository/     ← interface UserRepository, PostRepository, dll

USE CASE LAYER (business logic/application rules)
└── usecase/        ← RegisterUser, LoginUser, CreatePost, dll

DELIVERY LAYER (paling luar — tahu framework)
├── http/handler/   ← Gin handlers
├── http/middleware/← Gin middlewares
└── grpc/           ← gRPC handlers (nanti di Fase 5)

INFRASTRUCTURE (implementasi konkret — database, email, cache)
├── repository/postgres/  ← GORM implementation
├── repository/redis/     ← Redis implementation
└── external/email/       ← Email service
```

### Alur Data

```
HTTP Request
    ↓
[Handler] — parse request, validasi format
    ↓
[Use Case] — business logic, orchestrate
    ↓  ↑
[Repository Interface] — data access contract
    ↓
[Repository Implementation] — database/redis
```

---


### 🏋️ Latihan 4.2

1. Untuk sebuah "Expense Tracker" app, identifikasi: (a) domain entities (apa saja yang punya identity?), (b) value objects (apa saja yang tidak punya identity?), (c) use cases (apa saja yang bisa dilakukan user?), (d) delivery options (HTTP? CLI? Keduanya?).
2. Buat diagram lapisan Clean Architecture untuk Blog API dan tandai di lapisan mana setiap file dari Fase 3 seharusnya berada.

## 📦 Modul 4.3 — Project Structure

```
user-auth-service/
├── cmd/
│   └── api/
│       └── main.go                   ← Entry point, wire semua dependencies
│
├── config/
│   ├── config.go
│   └── config.yaml
│
├── internal/
│   │
│   ├── domain/                       ← DOMAIN LAYER (paling dalam)
│   │   ├── entity/
│   │   │   ├── user.go               ← User entity
│   │   │   └── token.go              ← Token entity
│   │   ├── valueobject/
│   │   │   ├── email.go              ← Email value object
│   │   │   └── password.go           ← Password value object
│   │   └── apperror/
│   │       └── errors.go             ← Domain errors
│   │
│   ├── repository/                   ← REPOSITORY INTERFACES
│   │   ├── user_repository.go        ← interface UserRepository
│   │   └── token_repository.go       ← interface TokenRepository
│   │
│   ├── usecase/                      ← USE CASE LAYER
│   │   ├── auth/
│   │   │   ├── register.go           ← RegisterUseCase
│   │   │   ├── login.go              ← LoginUseCase
│   │   │   ├── logout.go             ← LogoutUseCase
│   │   │   └── refresh.go            ← RefreshTokenUseCase
│   │   └── user/
│   │       ├── get_profile.go
│   │       └── update_profile.go
│   │
│   ├── delivery/                     ← DELIVERY LAYER
│   │   └── http/
│   │       ├── handler/
│   │       │   ├── auth_handler.go
│   │       │   └── user_handler.go
│   │       ├── middleware/
│   │       │   ├── auth.go
│   │       │   ├── logger.go
│   │       │   └── cors.go
│   │       ├── request/
│   │       │   ├── auth_request.go
│   │       │   └── user_request.go
│   │       ├── response/
│   │       │   ├── response.go
│   │       │   └── user_response.go
│   │       └── router.go
│   │
│   └── infrastructure/               ← INFRASTRUCTURE (implementasi konkret)
│       ├── persistence/
│       │   ├── postgres/
│       │   │   ├── db.go
│       │   │   ├── user_repository.go    ← implements repository.UserRepository
│       │   │   └── token_repository.go
│       │   └── redis/
│       │       └── token_repository.go   ← alternative implementation
│       └── external/
│           └── email/
│               └── smtp.go
│
├── pkg/                              ← Shared utilities (reusable lintas project)
│   ├── jwt/
│   │   └── jwt.go
│   ├── crypto/
│   │   └── bcrypt.go
│   ├── validator/
│   │   └── validator.go
│   └── logger/
│       └── logger.go
│
├── test/
│   ├── unit/
│   │   ├── usecase/
│   │   └── domain/
│   └── integration/
│       └── handler/
│
├── docker-compose.yml
├── Makefile
└── go.mod
```

---


### 🏋️ Latihan 4.3

1. Buat struktur folder Clean Architecture kosong (hanya folder dan file `.gitkeep`) untuk service baru "Inventory Service". Definisikan dulu domain entities apa yang ada, baru buat strukturnya.
2. Buat `go.mod` dan `go.work` untuk setup multi-module workspace yang berisi `user-service` dan `product-service`. Pastikan kedua service bisa saling import shared package.

## 📦 Modul 4.4 — Domain Layer

Domain layer adalah **inti dari aplikasi**. Tidak boleh import package apapun dari luar (tidak ada Gin, tidak ada GORM, tidak ada JWT di sini).

### Entity

```go
// internal/domain/entity/user.go
package entity

import (
    "time"
    "github.com/kamu/user-auth-service/internal/domain/apperror"
)

// UserRole adalah tipe untuk role user
type UserRole string

const (
    RoleUser  UserRole = "user"
    RoleAdmin UserRole = "admin"
)

// User adalah entity inti — merepresentasikan pengguna sistem
type User struct {
    ID           uint
    Name         string
    Email        string      // sudah validated email
    PasswordHash string      // sudah di-hash, tidak pernah plain text
    Role         UserRole
    IsActive     bool
    CreatedAt    time.Time
    UpdatedAt    time.Time
}

// NewUser adalah factory function — satu-satunya cara buat User yang valid
// Ini memastikan User tidak pernah dalam state yang tidak valid
func NewUser(name, email, passwordHash string) (*User, error) {
    if name == "" {
        return nil, apperror.ErrInvalidName
    }
    if len(name) < 2 || len(name) > 100 {
        return nil, apperror.NewValidationError("name", "must be between 2 and 100 characters")
    }
    if email == "" {
        return nil, apperror.ErrInvalidEmail
    }
    if passwordHash == "" {
        return nil, apperror.NewValidationError("password_hash", "cannot be empty")
    }

    return &User{
        Name:         name,
        Email:        email,
        PasswordHash: passwordHash,
        Role:         RoleUser,
        IsActive:     true,
        CreatedAt:    time.Now(),
        UpdatedAt:    time.Now(),
    }, nil
}

// Domain methods — business rules yang dimiliki User

// Activate mengaktifkan user
func (u *User) Activate() {
    u.IsActive = true
    u.UpdatedAt = time.Now()
}

// Deactivate menonaktifkan user
func (u *User) Deactivate() {
    u.IsActive = false
    u.UpdatedAt = time.Now()
}

// PromoteToAdmin mengubah role user menjadi admin
func (u *User) PromoteToAdmin() error {
    if u.Role == RoleAdmin {
        return apperror.ErrAlreadyAdmin
    }
    u.Role = RoleAdmin
    u.UpdatedAt = time.Now()
    return nil
}

// UpdateProfile mengubah nama user
func (u *User) UpdateProfile(name string) error {
    if name == "" || len(name) < 2 {
        return apperror.NewValidationError("name", "must be at least 2 characters")
    }
    u.Name = name
    u.UpdatedAt = time.Now()
    return nil
}

// IsAdmin mengecek apakah user adalah admin
func (u *User) IsAdmin() bool {
    return u.Role == RoleAdmin
}
```

```go
// internal/domain/entity/token.go
package entity

import "time"

// TokenType membedakan access dan refresh token
type TokenType string

const (
    AccessToken  TokenType = "access"
    RefreshToken TokenType = "refresh"
)

// Token merepresentasikan refresh token yang disimpan di DB
type Token struct {
    ID        uint
    UserID    uint
    Token     string
    Type      TokenType
    ExpiresAt time.Time
    CreatedAt time.Time
}

// NewRefreshToken membuat refresh token baru
func NewRefreshToken(userID uint, token string, expiresAt time.Time) *Token {
    return &Token{
        UserID:    userID,
        Token:     token,
        Type:      RefreshToken,
        ExpiresAt: expiresAt,
        CreatedAt: time.Now(),
    }
}

// IsExpired mengecek apakah token sudah kadaluarsa
func (t *Token) IsExpired() bool {
    return time.Now().After(t.ExpiresAt)
}
```

### Value Objects

Value Object adalah objek yang identitasnya ditentukan oleh **nilainya**, bukan ID-nya. Immutable setelah dibuat.

```go
// internal/domain/valueobject/email.go
package valueobject

import (
    "fmt"
    "regexp"
    "strings"
)

// emailRegex adalah ekspresi reguler untuk validasi email
var emailRegex = regexp.MustCompile(`^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$`)

// Email adalah value object yang menjamin email selalu valid
type Email struct {
    value string
}

// NewEmail membuat Email baru, return error jika format tidak valid
func NewEmail(email string) (Email, error) {
    email = strings.TrimSpace(strings.ToLower(email))
    if !emailRegex.MatchString(email) {
        return Email{}, fmt.Errorf("invalid email format: %s", email)
    }
    return Email{value: email}, nil
}

// String mengembalikan nilai string dari Email
func (e Email) String() string {
    return e.value
}

// Domain mengembalikan domain dari email (bagian setelah @)
func (e Email) Domain() string {
    parts := strings.Split(e.value, "@")
    if len(parts) != 2 {
        return ""
    }
    return parts[1]
}

// Equals mengecek apakah dua Email sama
func (e Email) Equals(other Email) bool {
    return e.value == other.value
}
```

```go
// internal/domain/valueobject/password.go
package valueobject

import (
    "fmt"
    "unicode"
)

// PasswordStrength menunjukkan kekuatan password
type PasswordStrength int

const (
    PasswordWeak   PasswordStrength = iota
    PasswordFair
    PasswordStrong
)

// minPasswordLength adalah panjang minimum password
const minPasswordLength = 8

// ValidatePassword mengecek apakah plain text password memenuhi syarat
// Tidak menyimpan password, hanya memvalidasi
func ValidatePassword(password string) error {
    if len(password) < minPasswordLength {
        return fmt.Errorf("password must be at least %d characters", minPasswordLength)
    }

    var hasUpper, hasLower, hasDigit bool
    for _, c := range password {
        switch {
        case unicode.IsUpper(c):
            hasUpper = true
        case unicode.IsLower(c):
            hasLower = true
        case unicode.IsDigit(c):
            hasDigit = true
        }
    }

    if !hasUpper {
        return fmt.Errorf("password must contain at least one uppercase letter")
    }
    if !hasLower {
        return fmt.Errorf("password must contain at least one lowercase letter")
    }
    if !hasDigit {
        return fmt.Errorf("password must contain at least one digit")
    }

    return nil
}

// CalculateStrength menghitung kekuatan password
func CalculateStrength(password string) PasswordStrength {
    if len(password) < minPasswordLength {
        return PasswordWeak
    }
    var score int
    if len(password) >= 12 {
        score++
    }
    for _, c := range password {
        switch {
        case unicode.IsUpper(c):
            score++
        case unicode.IsLower(c):
            score++
        case unicode.IsDigit(c):
            score++
        case unicode.IsPunct(c) || unicode.IsSymbol(c):
            score += 2
        }
    }
    if score >= 6 {
        return PasswordStrong
    }
    if score >= 4 {
        return PasswordFair
    }
    return PasswordWeak
}
```

### Domain Errors

```go
// internal/domain/apperror/errors.go
package apperror

import (
    "errors"
    "fmt"
    "net/http"
)

// AppError adalah domain error yang membawa HTTP status code
// ini memungkinkan delivery layer tahu status code tanpa tahu Gin
type AppError struct {
    Code       string // machine-readable code
    Message    string // human-readable message
    HTTPStatus int    // HTTP status code yang sesuai
    Err        error  // underlying error (optional)
}

func (e *AppError) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("%s: %v", e.Message, e.Err)
    }
    return e.Message
}

func (e *AppError) Unwrap() error {
    return e.Err
}

// Helper untuk membuat AppError
func New(code, message string, httpStatus int) *AppError {
    return &AppError{
        Code:       code,
        Message:    message,
        HTTPStatus: httpStatus,
    }
}

func NewWithErr(code, message string, httpStatus int, err error) *AppError {
    return &AppError{
        Code:       code,
        Message:    message,
        HTTPStatus: httpStatus,
        Err:        err,
    }
}

// ValidationError untuk field-level validation errors
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error: field '%s' %s", e.Field, e.Message)
}

func NewValidationError(field, message string) *ValidationError {
    return &ValidationError{Field: field, Message: message}
}

// ===== Sentinel Errors =====
// Semua domain error terdefinisi di sini, delivery layer tinggal errors.Is/As

var (
    // User errors
    ErrUserNotFound    = New("USER_NOT_FOUND", "user not found", http.StatusNotFound)
    ErrEmailExists     = New("EMAIL_EXISTS", "email already in use", http.StatusConflict)
    ErrInvalidName     = New("INVALID_NAME", "invalid name", http.StatusBadRequest)
    ErrInvalidEmail    = New("INVALID_EMAIL", "invalid email format", http.StatusBadRequest)
    ErrInvalidPassword = New("INVALID_PASSWORD", "invalid password", http.StatusBadRequest)
    ErrWrongPassword   = New("WRONG_PASSWORD", "email or password is incorrect", http.StatusUnauthorized)
    ErrUserInactive    = New("USER_INACTIVE", "user account is deactivated", http.StatusForbidden)
    ErrAlreadyAdmin    = New("ALREADY_ADMIN", "user is already an admin", http.StatusConflict)
    ErrUnauthorized    = New("UNAUTHORIZED", "authentication required", http.StatusUnauthorized)
    ErrForbidden       = New("FORBIDDEN", "insufficient permissions", http.StatusForbidden)

    // Token errors
    ErrTokenNotFound   = New("TOKEN_NOT_FOUND", "token not found", http.StatusUnauthorized)
    ErrTokenExpired    = New("TOKEN_EXPIRED", "token has expired", http.StatusUnauthorized)
    ErrTokenInvalid    = New("TOKEN_INVALID", "token is invalid", http.StatusUnauthorized)
)

// IsAppError mengecek apakah error adalah AppError
func IsAppError(err error) bool {
    var appErr *AppError
    return errors.As(err, &appErr)
}

// GetHTTPStatus mengambil HTTP status code dari error
// Default 500 jika bukan AppError
func GetHTTPStatus(err error) int {
    var appErr *AppError
    if errors.As(err, &appErr) {
        return appErr.HTTPStatus
    }
    return http.StatusInternalServerError
}

// GetCode mengambil error code
func GetCode(err error) string {
    var appErr *AppError
    if errors.As(err, &appErr) {
        return appErr.Code
    }
    return "INTERNAL_ERROR"
}
```

---


### 🏋️ Latihan 4.4

1. Buat Entity `Product` dengan field: ID, Name, Price, Stock, Category. Buat factory function `NewProduct` yang memvalidasi semua field. Tambahkan methods: `UpdatePrice(newPrice float64) error`, `AddStock(qty int) error`, `DeductStock(qty int) error` dengan validasi yang tepat.
2. Buat Value Object `Money` (Amount + Currency) yang immutable. Implementasikan `Add`, `Subtract`, `Multiply`. Buat unit test lengkap untuk semua operasi termasuk error cases (currency mismatch, negative amount).
3. Buat domain error hierarchy: `AppError` (base), `ValidationError`, `NotFoundError`, `ConflictError`. Setiap error harus punya `Code`, `Message`, dan `HTTPStatus`.

## 📦 Modul 4.5 — Repository Layer (Interface)

```go
// internal/repository/user_repository.go
package repository

import (
    "context"
    "github.com/kamu/user-auth-service/internal/domain/entity"
)

// UserRepository mendefinisikan kontrak untuk operasi data User.
// Interface ini ada di DALAM (domain side), implementasinya di infrastructure.
// Use Case hanya tahu interface ini, BUKAN implementasi konkretnya.
type UserRepository interface {
    // Save menyimpan user baru atau update yang sudah ada
    Save(ctx context.Context, user *entity.User) error

    // FindByID mencari user berdasarkan ID
    // Return (nil, nil) jika tidak ditemukan
    FindByID(ctx context.Context, id uint) (*entity.User, error)

    // FindByEmail mencari user berdasarkan email
    // Return (nil, nil) jika tidak ditemukan
    FindByEmail(ctx context.Context, email string) (*entity.User, error)

    // Update memperbarui data user
    Update(ctx context.Context, user *entity.User) error

    // Delete menghapus user (soft delete)
    Delete(ctx context.Context, id uint) error

    // FindAll mengambil semua user dengan filter dan pagination
    FindAll(ctx context.Context, opts FindAllOptions) ([]*entity.User, int64, error)

    // ExistsByEmail mengecek apakah email sudah terdaftar
    ExistsByEmail(ctx context.Context, email string) (bool, error)
}

// FindAllOptions adalah parameter untuk query FindAll
type FindAllOptions struct {
    Page    int
    PerPage int
    Search  string
    Role    string
    OrderBy string
    OrderDir string // "asc" atau "desc"
}
```

```go
// internal/repository/token_repository.go
package repository

import (
    "context"
    "github.com/kamu/user-auth-service/internal/domain/entity"
)

// TokenRepository mendefinisikan kontrak untuk operasi data Token
type TokenRepository interface {
    // Save menyimpan refresh token baru
    Save(ctx context.Context, token *entity.Token) error

    // FindByToken mencari token berdasarkan string token
    FindByToken(ctx context.Context, token string) (*entity.Token, error)

    // DeleteByToken menghapus token berdasarkan string
    DeleteByToken(ctx context.Context, token string) error

    // DeleteAllByUserID menghapus semua token milik user (logout dari semua device)
    DeleteAllByUserID(ctx context.Context, userID uint) error

    // DeleteExpired membersihkan token yang sudah expired
    DeleteExpired(ctx context.Context) error
}
```

---


### 🏋️ Latihan 4.5

1. Buat `ProductRepository` interface dengan semua CRUD methods + `FindByCategory`, `SearchByName`, `FindLowStock`. Buat juga `InMemoryProductRepository` sebagai implementasi untuk testing.
2. Tambahkan method `FindAllWithFilters(ctx, opts FilterOptions) ([]*Product, int64, error)` di interface. `FilterOptions` harus support: pagination, sorting, filter by category, filter by price range, search keyword.

## 📦 Modul 4.6 — Use Case Layer

Use Case adalah tempat **business logic** hidup. Tidak boleh import Gin, tidak boleh import GORM. Hanya boleh import dari domain dan repository interface.

### RegisterUseCase

```go
// internal/usecase/auth/register.go
package auth

import (
    "context"
    "fmt"

    "github.com/kamu/user-auth-service/internal/domain/apperror"
    "github.com/kamu/user-auth-service/internal/domain/entity"
    "github.com/kamu/user-auth-service/internal/domain/valueobject"
    "github.com/kamu/user-auth-service/internal/repository"
    "github.com/kamu/user-auth-service/pkg/crypto"
)

// RegisterInput adalah input untuk RegisterUseCase
// Ini adalah Plain Data, bukan HTTP request struct
type RegisterInput struct {
    Name     string
    Email    string
    Password string
}

// RegisterOutput adalah output dari RegisterUseCase
type RegisterOutput struct {
    User *entity.User
}

// RegisterUseCase mendefinisikan interface use case untuk registrasi
type RegisterUseCase interface {
    Execute(ctx context.Context, input RegisterInput) (*RegisterOutput, error)
}

// registerUseCase adalah implementasi konkret
type registerUseCase struct {
    userRepo  repository.UserRepository
    hasher    crypto.PasswordHasher
}

// NewRegisterUseCase membuat instance registerUseCase
// Dependency diinjeksikan melalui constructor — bukan hardcoded
func NewRegisterUseCase(
    userRepo repository.UserRepository,
    hasher crypto.PasswordHasher,
) RegisterUseCase {
    return &registerUseCase{
        userRepo:  userRepo,
        hasher:    hasher,
    }
}

// Execute menjalankan business logic registrasi
func (uc *registerUseCase) Execute(ctx context.Context, input RegisterInput) (*RegisterOutput, error) {
    // Step 1: Validasi email format (domain rule)
    email, err := valueobject.NewEmail(input.Email)
    if err != nil {
        return nil, apperror.ErrInvalidEmail
    }

    // Step 2: Validasi password strength (domain rule)
    if err := valueobject.ValidatePassword(input.Password); err != nil {
        return nil, apperror.NewWithErr(
            "INVALID_PASSWORD",
            err.Error(),
            400,
            err,
        )
    }

    // Step 3: Cek apakah email sudah terdaftar (business rule)
    exists, err := uc.userRepo.ExistsByEmail(ctx, email.String())
    if err != nil {
        return nil, fmt.Errorf("register: check email: %w", err)
    }
    if exists {
        return nil, apperror.ErrEmailExists
    }

    // Step 4: Hash password (security rule)
    hashedPassword, err := uc.hasher.Hash(input.Password)
    if err != nil {
        return nil, fmt.Errorf("register: hash password: %w", err)
    }

    // Step 5: Buat entity User baru (domain rule — melalui factory function)
    user, err := entity.NewUser(input.Name, email.String(), hashedPassword)
    if err != nil {
        return nil, err
    }

    // Step 6: Simpan ke repository
    if err := uc.userRepo.Save(ctx, user); err != nil {
        return nil, fmt.Errorf("register: save user: %w", err)
    }

    return &RegisterOutput{User: user}, nil
}
```

### LoginUseCase

```go
// internal/usecase/auth/login.go
package auth

import (
    "context"
    "fmt"
    "time"

    "github.com/kamu/user-auth-service/internal/domain/apperror"
    "github.com/kamu/user-auth-service/internal/domain/entity"
    "github.com/kamu/user-auth-service/internal/domain/valueobject"
    "github.com/kamu/user-auth-service/internal/repository"
    "github.com/kamu/user-auth-service/pkg/crypto"
    "github.com/kamu/user-auth-service/pkg/jwt"
)

type LoginInput struct {
    Email    string
    Password string
}

type LoginOutput struct {
    AccessToken  string
    RefreshToken string
    ExpiresIn    int64  // detik sampai access token expired
    User         *entity.User
}

type LoginUseCase interface {
    Execute(ctx context.Context, input LoginInput) (*LoginOutput, error)
}

type loginUseCase struct {
    userRepo   repository.UserRepository
    tokenRepo  repository.TokenRepository
    hasher     crypto.PasswordHasher
    jwtService jwt.JWTService
}

func NewLoginUseCase(
    userRepo repository.UserRepository,
    tokenRepo repository.TokenRepository,
    hasher crypto.PasswordHasher,
    jwtService jwt.JWTService,
) LoginUseCase {
    return &loginUseCase{
        userRepo:   userRepo,
        tokenRepo:  tokenRepo,
        hasher:     hasher,
        jwtService: jwtService,
    }
}

func (uc *loginUseCase) Execute(ctx context.Context, input LoginInput) (*LoginOutput, error) {
    // Step 1: Validasi format email
    email, err := valueobject.NewEmail(input.Email)
    if err != nil {
        // Jangan reveal apakah email yang salah atau password
        // Security: gunakan pesan generik
        return nil, apperror.ErrWrongPassword
    }

    // Step 2: Cari user berdasarkan email
    user, err := uc.userRepo.FindByEmail(ctx, email.String())
    if err != nil {
        return nil, fmt.Errorf("login: find user: %w", err)
    }
    if user == nil {
        // Tetap gunakan pesan generik, jangan bilang "email tidak ditemukan"
        return nil, apperror.ErrWrongPassword
    }

    // Step 3: Cek apakah user aktif
    if !user.IsActive {
        return nil, apperror.ErrUserInactive
    }

    // Step 4: Verifikasi password
    if !uc.hasher.Verify(input.Password, user.PasswordHash) {
        return nil, apperror.ErrWrongPassword
    }

    // Step 5: Generate access token
    accessTokenExpiry := 15 * time.Minute
    accessToken, err := uc.jwtService.GenerateAccessToken(user.ID, user.Email, string(user.Role))
    if err != nil {
        return nil, fmt.Errorf("login: generate access token: %w", err)
    }

    // Step 6: Generate refresh token
    refreshTokenStr, err := uc.jwtService.GenerateRefreshToken()
    if err != nil {
        return nil, fmt.Errorf("login: generate refresh token: %w", err)
    }

    // Step 7: Simpan refresh token ke repository
    refreshToken := entity.NewRefreshToken(
        user.ID,
        refreshTokenStr,
        time.Now().Add(7*24*time.Hour),
    )
    if err := uc.tokenRepo.Save(ctx, refreshToken); err != nil {
        return nil, fmt.Errorf("login: save refresh token: %w", err)
    }

    return &LoginOutput{
        AccessToken:  accessToken,
        RefreshToken: refreshTokenStr,
        ExpiresIn:    int64(accessTokenExpiry.Seconds()),
        User:         user,
    }, nil
}
```

### GetProfileUseCase

```go
// internal/usecase/user/get_profile.go
package user

import (
    "context"
    "fmt"

    "github.com/kamu/user-auth-service/internal/domain/apperror"
    "github.com/kamu/user-auth-service/internal/domain/entity"
    "github.com/kamu/user-auth-service/internal/repository"
)

type GetProfileInput struct {
    UserID uint
}

type GetProfileOutput struct {
    User *entity.User
}

type GetProfileUseCase interface {
    Execute(ctx context.Context, input GetProfileInput) (*GetProfileOutput, error)
}

type getProfileUseCase struct {
    userRepo repository.UserRepository
}

func NewGetProfileUseCase(userRepo repository.UserRepository) GetProfileUseCase {
    return &getProfileUseCase{userRepo: userRepo}
}

func (uc *getProfileUseCase) Execute(ctx context.Context, input GetProfileInput) (*GetProfileOutput, error) {
    user, err := uc.userRepo.FindByID(ctx, input.UserID)
    if err != nil {
        return nil, fmt.Errorf("get profile: %w", err)
    }
    if user == nil {
        return nil, apperror.ErrUserNotFound
    }
    return &GetProfileOutput{User: user}, nil
}
```

---


### 🏋️ Latihan 4.6

1. Implementasikan `CreateProductUseCase` lengkap: validasi input, cek duplikat SKU via repository, buat entity, simpan. Tulis unit test dengan mock repository untuk: success case, duplicate SKU case, validation error case, repository error case.
2. Implementasikan `SearchProductsUseCase` yang menerima `SearchInput{Keyword, Category, MinPrice, MaxPrice, Page, PerPage}` dan mengembalikan paginated result. Pastikan semua validasi input ada.

## 📦 Modul 4.7 — Delivery Layer (HTTP Handler)

Handler hanya bertugas:
1. Parse HTTP request → input DTO
2. Panggil use case
3. Format output → HTTP response

```go
// internal/delivery/http/handler/auth_handler.go
package handler

import (
    "errors"
    "net/http"

    "github.com/gin-gonic/gin"
    "github.com/kamu/user-auth-service/internal/delivery/http/request"
    "github.com/kamu/user-auth-service/internal/delivery/http/response"
    "github.com/kamu/user-auth-service/internal/domain/apperror"
    authUsecase "github.com/kamu/user-auth-service/internal/usecase/auth"
)

// AuthHandler menangani semua HTTP request terkait autentikasi
type AuthHandler struct {
    registerUC authUsecase.RegisterUseCase
    loginUC    authUsecase.LoginUseCase
    logoutUC   authUsecase.LogoutUseCase
    refreshUC  authUsecase.RefreshTokenUseCase
}

// NewAuthHandler membuat instance AuthHandler dengan dependency injection
func NewAuthHandler(
    registerUC authUsecase.RegisterUseCase,
    loginUC authUsecase.LoginUseCase,
    logoutUC authUsecase.LogoutUseCase,
    refreshUC authUsecase.RefreshTokenUseCase,
) *AuthHandler {
    return &AuthHandler{
        registerUC: registerUC,
        loginUC:    loginUC,
        logoutUC:   logoutUC,
        refreshUC:  refreshUC,
    }
}

// Register godoc
// @Summary Register user baru
// @Tags auth
// @Accept json
// @Produce json
// @Param request body request.RegisterRequest true "Register data"
// @Success 201 {object} response.AuthResponse
// @Failure 400 {object} response.ErrorResponse
// @Failure 409 {object} response.ErrorResponse
// @Router /auth/register [post]
func (h *AuthHandler) Register(c *gin.Context) {
    // Step 1: Parse dan validasi request body
    var req request.RegisterRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        response.BadRequest(c, "validation failed", parseValidationErrors(err))
        return
    }

    // Step 2: Panggil use case dengan input yang sesuai
    output, err := h.registerUC.Execute(c.Request.Context(), authUsecase.RegisterInput{
        Name:     req.Name,
        Email:    req.Email,
        Password: req.Password,
    })

    // Step 3: Handle error dari use case
    if err != nil {
        h.handleError(c, err)
        return
    }

    // Step 4: Format dan kirim response
    response.Created(c, response.ToUserResponse(output.User))
}

func (h *AuthHandler) Login(c *gin.Context) {
    var req request.LoginRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        response.BadRequest(c, "validation failed", parseValidationErrors(err))
        return
    }

    output, err := h.loginUC.Execute(c.Request.Context(), authUsecase.LoginInput{
        Email:    req.Email,
        Password: req.Password,
    })
    if err != nil {
        h.handleError(c, err)
        return
    }

    response.OK(c, gin.H{
        "access_token":  output.AccessToken,
        "refresh_token": output.RefreshToken,
        "expires_in":    output.ExpiresIn,
        "user":          response.ToUserResponse(output.User),
    })
}

func (h *AuthHandler) Logout(c *gin.Context) {
    var req request.LogoutRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        response.BadRequest(c, "refresh_token is required", nil)
        return
    }

    userID := c.GetUint("user_id")
    err := h.logoutUC.Execute(c.Request.Context(), authUsecase.LogoutInput{
        UserID:       userID,
        RefreshToken: req.RefreshToken,
    })
    if err != nil {
        h.handleError(c, err)
        return
    }

    response.NoContent(c)
}

// handleError adalah helper untuk mengkonversi domain error ke HTTP response
// Ini satu-satunya tempat di delivery layer yang tahu tentang AppError
func (h *AuthHandler) handleError(c *gin.Context, err error) {
    var appErr *apperror.AppError
    if errors.As(err, &appErr) {
        c.JSON(appErr.HTTPStatus, response.ErrorResponse{
            Success: false,
            Error: response.ErrorInfo{
                Code:    appErr.Code,
                Message: appErr.Message,
            },
        })
        return
    }

    // Error tidak dikenal — jangan expose detail ke client
    // Log error ini untuk debugging
    c.JSON(http.StatusInternalServerError, response.ErrorResponse{
        Success: false,
        Error: response.ErrorInfo{
            Code:    "INTERNAL_ERROR",
            Message: "an unexpected error occurred",
        },
    })
}
```

### Request & Response DTOs

```go
// internal/delivery/http/request/auth_request.go
package request

// RegisterRequest adalah request body untuk endpoint POST /auth/register
type RegisterRequest struct {
    Name     string `json:"name" binding:"required,min=2,max=100"`
    Email    string `json:"email" binding:"required,email"`
    Password string `json:"password" binding:"required,min=8"`
}

// LoginRequest adalah request body untuk endpoint POST /auth/login
type LoginRequest struct {
    Email    string `json:"email" binding:"required,email"`
    Password string `json:"password" binding:"required"`
}

// LogoutRequest adalah request body untuk endpoint POST /auth/logout
type LogoutRequest struct {
    RefreshToken string `json:"refresh_token" binding:"required"`
}

// RefreshTokenRequest adalah request body untuk endpoint POST /auth/refresh
type RefreshTokenRequest struct {
    RefreshToken string `json:"refresh_token" binding:"required"`
}
```

```go
// internal/delivery/http/response/user_response.go
package response

import (
    "time"
    "github.com/kamu/user-auth-service/internal/domain/entity"
)

// UserResponse adalah DTO untuk menampilkan data user ke client
// Field yang sensitif (password hash) tidak diekspos
type UserResponse struct {
    ID        uint      `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    Role      string    `json:"role"`
    IsActive  bool      `json:"is_active"`
    CreatedAt time.Time `json:"created_at"`
}

// ToUserResponse mengkonversi entity.User ke UserResponse
func ToUserResponse(user *entity.User) *UserResponse {
    return &UserResponse{
        ID:        user.ID,
        Name:      user.Name,
        Email:     user.Email,
        Role:      string(user.Role),
        IsActive:  user.IsActive,
        CreatedAt: user.CreatedAt,
    }
}

// ToUserResponseList mengkonversi slice of entity.User ke slice of UserResponse
func ToUserResponseList(users []*entity.User) []*UserResponse {
    result := make([]*UserResponse, len(users))
    for i, u := range users {
        result[i] = ToUserResponse(u)
    }
    return result
}
```

---


### 🏋️ Latihan 4.7

1. Refactor Blog API handler `CreatePost` menjadi "thin handler" menggunakan Clean Architecture. Handler hanya boleh: (1) parse request, (2) call use case, (3) handle error, (4) return response.
2. Buat `ErrorConverter` yang mengkonversi domain error ke HTTP response. Test bahwa: `NotFoundError` → 404, `ConflictError` → 409, `ValidationError` → 400, unknown error → 500.

## 📦 Modul 4.8 — Dependency Injection

```go
// cmd/api/main.go
package main

import (
    "context"
    "fmt"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/gin-gonic/gin"
    "github.com/kamu/user-auth-service/config"
    "github.com/kamu/user-auth-service/internal/delivery/http/handler"
    "github.com/kamu/user-auth-service/internal/delivery/http/middleware"
    "github.com/kamu/user-auth-service/internal/infrastructure/persistence/postgres"
    authUsecase "github.com/kamu/user-auth-service/internal/usecase/auth"
    userUsecase "github.com/kamu/user-auth-service/internal/usecase/user"
    "github.com/kamu/user-auth-service/pkg/crypto"
    "github.com/kamu/user-auth-service/pkg/jwt"
    "github.com/kamu/user-auth-service/pkg/logger"
)

func main() {
    // ===== 1. Load configuration =====
    cfg, err := config.Load()
    if err != nil {
        log.Fatalf("failed to load config: %v", err)
    }

    // ===== 2. Setup logger =====
    appLogger := logger.New(cfg.Log.Level, cfg.Log.Format)

    // ===== 3. Connect database =====
    db, err := postgres.Connect(cfg.Database.DSN())
    if err != nil {
        appLogger.Fatal("failed to connect database", "error", err)
    }

    // ===== 4. Run migrations =====
    if err := postgres.Migrate(db); err != nil {
        appLogger.Fatal("failed to migrate database", "error", err)
    }

    // ===== 5. Initialize Infrastructure (Repository implementations) =====
    userRepo := postgres.NewUserRepository(db)
    tokenRepo := postgres.NewTokenRepository(db)

    // ===== 6. Initialize shared services / adapters =====
    passwordHasher := crypto.NewBcryptHasher()
    jwtService := jwt.NewJWTService(cfg.JWT.Secret, cfg.JWT.AccessExpiry, cfg.JWT.RefreshExpiry)

    // ===== 7. Initialize Use Cases (wire up dependencies) =====
    // Auth use cases
    registerUC := authUsecase.NewRegisterUseCase(userRepo, passwordHasher)
    loginUC := authUsecase.NewLoginUseCase(userRepo, tokenRepo, passwordHasher, jwtService)
    logoutUC := authUsecase.NewLogoutUseCase(tokenRepo)
    refreshUC := authUsecase.NewRefreshTokenUseCase(tokenRepo, userRepo, jwtService)

    // User use cases
    getProfileUC := userUsecase.NewGetProfileUseCase(userRepo)
    updateProfileUC := userUsecase.NewUpdateProfileUseCase(userRepo)

    // ===== 8. Initialize Handlers =====
    authHandler := handler.NewAuthHandler(registerUC, loginUC, logoutUC, refreshUC)
    userHandler := handler.NewUserHandler(getProfileUC, updateProfileUC)

    // ===== 9. Initialize Middlewares =====
    authMiddleware := middleware.NewAuthMiddleware(jwtService)
    loggerMiddleware := middleware.NewLoggerMiddleware(appLogger)
    corsMiddleware := middleware.NewCORSMiddleware(cfg.App.AllowedOrigins)

    // ===== 10. Setup Router =====
    if cfg.App.Env == "production" {
        gin.SetMode(gin.ReleaseMode)
    }

    router := setupRouter(
        authHandler,
        userHandler,
        authMiddleware,
        loggerMiddleware,
        corsMiddleware,
    )

    // ===== 11. Start server dengan Graceful Shutdown =====
    srv := &http.Server{
        Addr:         fmt.Sprintf(":%d", cfg.App.Port),
        Handler:      router,
        ReadTimeout:  15 * time.Second,
        WriteTimeout: 15 * time.Second,
        IdleTimeout:  60 * time.Second,
    }

    // Start server di goroutine
    go func() {
        appLogger.Info("server started", "port", cfg.App.Port)
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            appLogger.Fatal("server failed", "error", err)
        }
    }()

    // Wait untuk shutdown signal
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    appLogger.Info("shutting down server...")

    // Graceful shutdown: berikan waktu 30 detik untuk request yang sedang berjalan
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    if err := srv.Shutdown(ctx); err != nil {
        appLogger.Fatal("server forced to shutdown", "error", err)
    }

    appLogger.Info("server exited cleanly")
}
```

### Router Setup

```go
// internal/delivery/http/router.go
package http

import (
    "github.com/gin-gonic/gin"
    "github.com/kamu/user-auth-service/internal/delivery/http/handler"
    "github.com/kamu/user-auth-service/internal/delivery/http/middleware"
)

func SetupRouter(
    authHandler *handler.AuthHandler,
    userHandler *handler.UserHandler,
    authMiddleware *middleware.AuthMiddleware,
    loggerMiddleware *middleware.LoggerMiddleware,
    corsMiddleware *middleware.CORSMiddleware,
) *gin.Engine {
    r := gin.New()

    // Global middlewares
    r.Use(gin.Recovery())
    r.Use(loggerMiddleware.Handle())
    r.Use(corsMiddleware.Handle())

    // Health check (tidak perlu auth)
    r.GET("/health", healthCheck)
    r.GET("/ready", readinessCheck)

    // API v1
    v1 := r.Group("/api/v1")
    {
        // Auth routes — public
        auth := v1.Group("/auth")
        {
            auth.POST("/register", authHandler.Register)
            auth.POST("/login", authHandler.Login)
            auth.POST("/refresh", authHandler.RefreshToken)

            // Logout perlu auth
            auth.POST("/logout", authMiddleware.Authenticate(), authHandler.Logout)
            auth.GET("/me", authMiddleware.Authenticate(), authHandler.GetMe)
        }

        // User routes — protected
        users := v1.Group("/users")
        users.Use(authMiddleware.Authenticate())
        {
            users.GET("/profile", userHandler.GetProfile)
            users.PUT("/profile", userHandler.UpdateProfile)

            // Admin only
            admin := users.Group("")
            admin.Use(authMiddleware.RequireRole("admin"))
            {
                admin.GET("", userHandler.GetAll)
                admin.GET("/:id", userHandler.GetByID)
                admin.DELETE("/:id", userHandler.Delete)
                admin.PUT("/:id/role", userHandler.UpdateRole)
            }
        }
    }

    return r
}

func healthCheck(c *gin.Context) {
    c.JSON(200, gin.H{
        "status":  "ok",
        "service": "user-auth-service",
    })
}

func readinessCheck(c *gin.Context) {
    // Cek koneksi database dan dependencies lainnya
    c.JSON(200, gin.H{
        "status": "ready",
        "checks": gin.H{
            "database": "ok",
        },
    })
}
```

---


### 🏋️ Latihan 4.8

1. Buat `main.go` lengkap untuk User Auth Service yang melakukan dependency injection secara manual (tanpa framework DI). Urutkan: config → db → repos → usecases → handlers → router → server.
2. Buat `Container` struct yang menyimpan semua dependencies sebagai field. Buat method `NewContainer(cfg *Config) (*Container, error)` yang menginisialisasi semua. Pastikan teardown bisa dilakukan dengan `container.Close()`.

## 📦 Modul 4.9 — Unit Testing per Layer

### Testing Use Case dengan Mock

```go
// test/unit/usecase/auth/register_test.go
package auth_test

import (
    "context"
    "errors"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
    "github.com/kamu/user-auth-service/internal/domain/apperror"
    "github.com/kamu/user-auth-service/internal/domain/entity"
    authUsecase "github.com/kamu/user-auth-service/internal/usecase/auth"
)

// ===== Mock Repository =====

type MockUserRepository struct {
    mock.Mock
}

func (m *MockUserRepository) Save(ctx context.Context, user *entity.User) error {
    args := m.Called(ctx, user)
    return args.Error(0)
}

func (m *MockUserRepository) FindByEmail(ctx context.Context, email string) (*entity.User, error) {
    args := m.Called(ctx, email)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*entity.User), args.Error(1)
}

func (m *MockUserRepository) ExistsByEmail(ctx context.Context, email string) (bool, error) {
    args := m.Called(ctx, email)
    return args.Bool(0), args.Error(1)
}

// ... implement method lainnya

// ===== Mock Password Hasher =====

type MockPasswordHasher struct {
    mock.Mock
}

func (m *MockPasswordHasher) Hash(password string) (string, error) {
    args := m.Called(password)
    return args.String(0), args.Error(1)
}

func (m *MockPasswordHasher) Verify(password, hash string) bool {
    args := m.Called(password, hash)
    return args.Bool(0)
}

// ===== Tests =====

func TestRegisterUseCase_Execute_Success(t *testing.T) {
    // Arrange
    mockUserRepo := new(MockUserRepository)
    mockHasher := new(MockPasswordHasher)
    uc := authUsecase.NewRegisterUseCase(mockUserRepo, mockHasher)

    input := authUsecase.RegisterInput{
        Name:     "Alice",
        Email:    "alice@example.com",
        Password: "SecurePass123",
    }

    // Setup mock expectations
    mockUserRepo.On("ExistsByEmail", mock.Anything, "alice@example.com").
        Return(false, nil)
    mockHasher.On("Hash", "SecurePass123").
        Return("$2a$14$hashedpassword", nil)
    mockUserRepo.On("Save", mock.Anything, mock.MatchedBy(func(u *entity.User) bool {
        return u.Name == "Alice" && u.Email == "alice@example.com"
    })).Return(nil)

    // Act
    output, err := uc.Execute(context.Background(), input)

    // Assert
    assert.NoError(t, err)
    assert.NotNil(t, output)
    assert.NotNil(t, output.User)
    assert.Equal(t, "Alice", output.User.Name)
    assert.Equal(t, "alice@example.com", output.User.Email)
    assert.Equal(t, entity.RoleUser, output.User.Role)
    assert.True(t, output.User.IsActive)

    // Verify mock calls
    mockUserRepo.AssertExpectations(t)
    mockHasher.AssertExpectations(t)
}

func TestRegisterUseCase_Execute_EmailAlreadyExists(t *testing.T) {
    mockUserRepo := new(MockUserRepository)
    mockHasher := new(MockPasswordHasher)
    uc := authUsecase.NewRegisterUseCase(mockUserRepo, mockHasher)

    mockUserRepo.On("ExistsByEmail", mock.Anything, "alice@example.com").
        Return(true, nil) // email sudah ada!

    _, err := uc.Execute(context.Background(), authUsecase.RegisterInput{
        Name:     "Alice",
        Email:    "alice@example.com",
        Password: "SecurePass123",
    })

    assert.Error(t, err)
    assert.True(t, errors.Is(err, apperror.ErrEmailExists))

    // Hash tidak boleh dipanggil
    mockHasher.AssertNotCalled(t, "Hash")
}

func TestRegisterUseCase_Execute_InvalidEmail(t *testing.T) {
    mockUserRepo := new(MockUserRepository)
    mockHasher := new(MockPasswordHasher)
    uc := authUsecase.NewRegisterUseCase(mockUserRepo, mockHasher)

    _, err := uc.Execute(context.Background(), authUsecase.RegisterInput{
        Name:     "Alice",
        Email:    "not-an-email", // email tidak valid
        Password: "SecurePass123",
    })

    assert.Error(t, err)
    // Repository tidak boleh dipanggil sama sekali
    mockUserRepo.AssertNotCalled(t, "ExistsByEmail")
    mockUserRepo.AssertNotCalled(t, "Save")
}

// Table-driven tests untuk berbagai skenario validasi
func TestRegisterUseCase_Execute_ValidationCases(t *testing.T) {
    tests := []struct {
        name    string
        input   authUsecase.RegisterInput
        wantErr bool
        errType error
    }{
        {
            name: "valid input",
            input: authUsecase.RegisterInput{
                Name: "Alice", Email: "alice@test.com", Password: "SecurePass123",
            },
            wantErr: false,
        },
        {
            name: "empty name",
            input: authUsecase.RegisterInput{
                Name: "", Email: "alice@test.com", Password: "SecurePass123",
            },
            wantErr: true,
        },
        {
            name: "invalid email",
            input: authUsecase.RegisterInput{
                Name: "Alice", Email: "notanemail", Password: "SecurePass123",
            },
            wantErr: true,
            errType: apperror.ErrInvalidEmail,
        },
        {
            name: "weak password — too short",
            input: authUsecase.RegisterInput{
                Name: "Alice", Email: "alice@test.com", Password: "abc",
            },
            wantErr: true,
        },
        {
            name: "weak password — no uppercase",
            input: authUsecase.RegisterInput{
                Name: "Alice", Email: "alice@test.com", Password: "securepass123",
            },
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            mockUserRepo := new(MockUserRepository)
            mockHasher := new(MockPasswordHasher)

            // Setup mock untuk valid case
            if !tt.wantErr {
                mockUserRepo.On("ExistsByEmail", mock.Anything, mock.Anything).Return(false, nil)
                mockHasher.On("Hash", mock.Anything).Return("hashed", nil)
                mockUserRepo.On("Save", mock.Anything, mock.Anything).Return(nil)
            }

            uc := authUsecase.NewRegisterUseCase(mockUserRepo, mockHasher)
            _, err := uc.Execute(context.Background(), tt.input)

            if tt.wantErr {
                assert.Error(t, err)
                if tt.errType != nil {
                    assert.True(t, errors.Is(err, tt.errType))
                }
            } else {
                assert.NoError(t, err)
            }
        })
    }
}
```

---


### 🏋️ Latihan 4.9

1. Tulis unit tests lengkap untuk `LoginUseCase` dengan semua skenario: (a) success, (b) user not found, (c) wrong password, (d) user inactive, (e) token generation error, (f) token save error. Gunakan table-driven tests.
2. Buat `MockEmailService` yang mengimplementasikan `EmailService` interface untuk testing. Tambahkan assertion bahwa email welcome dikirim saat register berhasil.


## 📦 Modul 4.10 — Integration Testing

Integration test memverifikasi bahwa beberapa komponen bekerja bersama dengan benar — termasuk database sungguhan, bukan mock.

### Setup Test Database

```go
// test/integration/setup_test.go
package integration_test

import (
    "context"
    "fmt"
    "os"
    "testing"

    "gorm.io/driver/postgres"
    "gorm.io/gorm"

    "github.com/kamu/user-auth-service/internal/infrastructure/persistence/postgres/model"
)

var testDB *gorm.DB

// TestMain adalah entry point untuk semua integration test di package ini
func TestMain(m *testing.M) {
    // Gunakan database test terpisah!
    dsn := getTestDSN()

    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        fmt.Fprintf(os.Stderr, "failed to connect test db: %v\n", err)
        os.Exit(1)
    }

    // Migrate
    db.AutoMigrate(&model.User{}, &model.RefreshToken{})
    testDB = db

    // Jalankan semua test
    code := m.Run()

    // Cleanup: hapus semua data test (bukan drop table!)
    testDB.Exec("DELETE FROM refresh_tokens")
    testDB.Exec("DELETE FROM users")

    os.Exit(code)
}

func getTestDSN() string {
    // Baca dari environment variable
    dsn := os.Getenv("TEST_DATABASE_URL")
    if dsn == "" {
        // Default untuk development
        dsn = "host=localhost user=testuser password=testpass dbname=testdb_auth sslmode=disable"
    }
    return dsn
}

// cleanDB membersihkan data antar test
func cleanDB(t *testing.T) {
    t.Helper()
    testDB.Exec("DELETE FROM refresh_tokens")
    testDB.Exec("DELETE FROM users")
}
```

### Integration Test untuk Handler

```go
// test/integration/auth_handler_test.go
package integration_test

import (
    "bytes"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"

    "github.com/gin-gonic/gin"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

// setupTestRouter membuat router yang terhubung ke test database
func setupTestRouter(t *testing.T) *gin.Engine {
    t.Helper()
    gin.SetMode(gin.TestMode)

    // Wire semua dependency dengan test database
    userRepo := postgres.NewUserRepository(testDB)
    tokenRepo := postgres.NewTokenRepository(testDB)
    hasher := crypto.NewBcryptHasher()
    jwtService := jwt.NewJWTService("test-secret-key", 15*time.Minute, 7*24*time.Hour)

    registerUC := authUsecase.NewRegisterUseCase(userRepo, hasher)
    loginUC := authUsecase.NewLoginUseCase(userRepo, tokenRepo, hasher, jwtService)
    logoutUC := authUsecase.NewLogoutUseCase(tokenRepo)

    authHandler := handler.NewAuthHandler(registerUC, loginUC, logoutUC, nil)

    r := gin.New()
    r.POST("/auth/register", authHandler.Register)
    r.POST("/auth/login", authHandler.Login)
    r.POST("/auth/logout", middleware.NewAuthMiddleware(jwtService).Authenticate(), authHandler.Logout)

    return r
}

func TestAuthFlow_RegisterLoginLogout(t *testing.T) {
    cleanDB(t)
    r := setupTestRouter(t)

    // ===== Step 1: Register =====
    t.Run("register success", func(t *testing.T) {
        body := map[string]string{
            "name":     "Alice",
            "email":    "alice@test.com",
            "password": "SecurePass123",
        }
        bodyBytes, _ := json.Marshal(body)

        req := httptest.NewRequest(http.MethodPost, "/auth/register", bytes.NewBuffer(bodyBytes))
        req.Header.Set("Content-Type", "application/json")
        w := httptest.NewRecorder()

        r.ServeHTTP(w, req)

        assert.Equal(t, http.StatusCreated, w.Code)

        var resp map[string]interface{}
        err := json.Unmarshal(w.Body.Bytes(), &resp)
        require.NoError(t, err)
        assert.True(t, resp["success"].(bool))

        data := resp["data"].(map[string]interface{})
        assert.NotEmpty(t, data["access_token"])
        assert.NotEmpty(t, data["refresh_token"])
    })

    // ===== Step 2: Register duplikat email =====
    t.Run("register duplicate email", func(t *testing.T) {
        body := map[string]string{
            "name":     "Alice Duplicate",
            "email":    "alice@test.com", // sama!
            "password": "SecurePass123",
        }
        bodyBytes, _ := json.Marshal(body)

        req := httptest.NewRequest(http.MethodPost, "/auth/register", bytes.NewBuffer(bodyBytes))
        req.Header.Set("Content-Type", "application/json")
        w := httptest.NewRecorder()

        r.ServeHTTP(w, req)

        assert.Equal(t, http.StatusConflict, w.Code) // 409
    })

    // ===== Step 3: Login =====
    var accessToken, refreshToken string
    t.Run("login success", func(t *testing.T) {
        body := map[string]string{
            "email":    "alice@test.com",
            "password": "SecurePass123",
        }
        bodyBytes, _ := json.Marshal(body)

        req := httptest.NewRequest(http.MethodPost, "/auth/login", bytes.NewBuffer(bodyBytes))
        req.Header.Set("Content-Type", "application/json")
        w := httptest.NewRecorder()

        r.ServeHTTP(w, req)

        assert.Equal(t, http.StatusOK, w.Code)

        var resp map[string]interface{}
        json.Unmarshal(w.Body.Bytes(), &resp)
        data := resp["data"].(map[string]interface{})
        accessToken = data["access_token"].(string)
        refreshToken = data["refresh_token"].(string)

        assert.NotEmpty(t, accessToken)
        assert.NotEmpty(t, refreshToken)
    })

    // ===== Step 4: Logout =====
    t.Run("logout success", func(t *testing.T) {
        body := map[string]string{"refresh_token": refreshToken}
        bodyBytes, _ := json.Marshal(body)

        req := httptest.NewRequest(http.MethodPost, "/auth/logout", bytes.NewBuffer(bodyBytes))
        req.Header.Set("Content-Type", "application/json")
        req.Header.Set("Authorization", "Bearer "+accessToken)
        w := httptest.NewRecorder()

        r.ServeHTTP(w, req)

        assert.Equal(t, http.StatusNoContent, w.Code)

        // ===== Step 5: Token tidak bisa dipakai lagi =====
        req2 := httptest.NewRequest(http.MethodPost, "/auth/logout", bytes.NewBuffer(bodyBytes))
        req2.Header.Set("Content-Type", "application/json")
        req2.Header.Set("Authorization", "Bearer "+accessToken)
        w2 := httptest.NewRecorder()

        r.ServeHTTP(w2, req2)
        // refresh token sudah dihapus — logout kedua harus tetap OK (idempotent)
    })
}
```

### Cara Menjalankan Integration Test

```bash
# Pastikan test database sudah berjalan
docker run -d   --name postgres-test   -e POSTGRES_USER=testuser   -e POSTGRES_PASSWORD=testpass   -e POSTGRES_DB=testdb_auth   -p 5433:5432   postgres:16-alpine

# Set environment variable
export TEST_DATABASE_URL="host=localhost port=5433 user=testuser password=testpass dbname=testdb_auth sslmode=disable"

# Jalankan integration test saja (bukan unit test)
go test ./test/integration/... -v -count=1

# Atau dengan tag build
go test -tags integration ./... -v

# Jalankan dengan timeout
go test ./test/integration/... -v -timeout 60s
```

### Test Tags untuk Pemisahan

```go
// Tambahkan build tag di integration test
//go:build integration
// +build integration

package integration_test
```

```bash
# Jalankan hanya unit test (cepat, tanpa database)
go test ./...

# Jalankan unit test + integration test
go test -tags integration ./...
```

### 🏋️ Latihan 4.10

1. Buat integration test untuk endpoint `POST /auth/register` yang memverifikasi: (a) user tersimpan di database, (b) password tersimpan sebagai hash (bukan plain text!), (c) role default adalah "user", (d) token yang dikembalikan valid.
2. Buat integration test untuk full flow: register → login → akses protected endpoint → logout → coba akses lagi (harus gagal). Verifikasi state database di setiap step.
3. Buat `TestContainerSetup` menggunakan library `testcontainers-go` untuk otomatis start/stop PostgreSQL Docker container saat test, sehingga tidak perlu database manual.


## 📦 Modul 4.11 — Graceful Shutdown

```go
// pkg/server/server.go
package server

import (
    "context"
    "net/http"
    "time"
    
    "github.com/gin-gonic/gin"
)

type Server struct {
    httpServer *http.Server
}

func New(addr string, handler *gin.Engine) *Server {
    return &Server{
        httpServer: &http.Server{
            Addr:         addr,
            Handler:      handler,
            ReadTimeout:  15 * time.Second,
            WriteTimeout: 15 * time.Second,
            IdleTimeout:  60 * time.Second,
        },
    }
}

func (s *Server) Start() error {
    return s.httpServer.ListenAndServe()
}

func (s *Server) Shutdown(ctx context.Context) error {
    return s.httpServer.Shutdown(ctx)
}
```

```go
// main.go — graceful shutdown pattern yang benar

func main() {
    // ... setup code ...

    srv := server.New(fmt.Sprintf(":%d", cfg.App.Port), router)

    // Start di background goroutine
    go func() {
        if err := srv.Start(); err != nil && err != http.ErrServerClosed {
            log.Fatalf("server error: %v", err)
        }
    }()

    log.Printf("✅ Server running on port %d", cfg.App.Port)

    // Tunggu signal OS
    quit := make(chan os.Signal, 1)
    // SIGINT = Ctrl+C
    // SIGTERM = docker stop / kubernetes pod termination
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    sig := <-quit

    log.Printf("🛑 Received signal: %v. Shutting down...", sig)

    // Beri waktu 30 detik untuk request yang sedang berjalan
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    // Stop menerima request baru, tunggu yang sedang berjalan selesai
    if err := srv.Shutdown(ctx); err != nil {
        log.Fatalf("❌ Server shutdown failed: %v", err)
    }

    // Close database connection
    if sqlDB, err := db.DB(); err == nil {
        sqlDB.Close()
    }

    log.Println("✅ Server exited cleanly")
}
```

---


### 🏋️ Latihan 4.11

1. Tambahkan graceful shutdown ke User Auth Service yang: (a) stop menerima request baru, (b) tunggu request aktif selesai max 30 detik, (c) close database connection, (d) log semua step. Test dengan `kill -SIGTERM <pid>` saat ada request sedang diproses.
2. Buat `StartupCheck` yang berjalan saat startup dan memverifikasi: database terhubung, migrasi sudah up-to-date, env variables wajib ada. Jika gagal, program exit dengan error message yang jelas.

## 📦 Modul 4.12 — Health Check & Readiness

```go
// internal/delivery/http/handler/health_handler.go
package handler

import (
    "net/http"
    "time"

    "github.com/gin-gonic/gin"
    "gorm.io/gorm"
)

type HealthHandler struct {
    db      *gorm.DB
    version string
    startTime time.Time
}

func NewHealthHandler(db *gorm.DB, version string) *HealthHandler {
    return &HealthHandler{
        db:      db,
        version: version,
        startTime: time.Now(),
    }
}

// GET /health — untuk load balancer, tidak cek dependencies
func (h *HealthHandler) Health(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "status":  "ok",
        "version": h.version,
        "uptime":  time.Since(h.startTime).String(),
    })
}

// GET /ready — untuk Kubernetes, cek semua dependencies
func (h *HealthHandler) Ready(c *gin.Context) {
    checks := make(map[string]string)
    allOK := true

    // Cek database
    sqlDB, err := h.db.DB()
    if err != nil || sqlDB.Ping() != nil {
        checks["database"] = "unhealthy"
        allOK = false
    } else {
        checks["database"] = "healthy"
    }

    status := http.StatusOK
    statusStr := "ready"
    if !allOK {
        status = http.StatusServiceUnavailable
        statusStr = "not ready"
    }

    c.JSON(status, gin.H{
        "status": statusStr,
        "checks": checks,
    })
}
```

---


### 🏋️ Latihan 4.12

1. Buat health check endpoint `/health` yang mengembalikan uptime, versi build (dari ldflags), dan memori yang digunakan. Buat readiness endpoint `/ready` yang mengecek koneksi database dan mengembalikan detail per dependency.
2. Buat `ShutdownManager` yang mengumpulkan semua resource yang perlu di-close (`database`, `redis`, `message broker`) dan menutupnya secara graceful dengan timeout yang bisa dikonfigurasi.

## 🎯 Review & Checkpoint Fase 4

### Konseptual
- [ ] Jelaskan Dependency Rule di Clean Architecture
- [ ] Mengapa Domain Layer tidak boleh import package luar?
- [ ] Apa perbedaan Entity dan Value Object?
- [ ] Mengapa Interface Repository ada di dalam (domain side)?
- [ ] Apa keuntungan memisahkan Use Case dari Handler?
- [ ] Mengapa kita perlu Graceful Shutdown?

### Praktis
- [ ] Bisa membuat Entity dengan factory function dan business methods
- [ ] Bisa membuat Value Object yang immutable dan self-validating
- [ ] Bisa membuat Use Case yang testable dengan dependency injection
- [ ] Bisa membuat Handler yang tipis (thin handler)
- [ ] Bisa membuat unit test per layer dengan mock
- [ ] Bisa implementasi Graceful Shutdown

---

## 🎯 Project Akhir Fase 4

Kerjakan project **User Auth Service** berdasarkan PRD di file:

**`FASE-4-PRD-User-Auth-Service.md`**

---

*Setelah selesai Fase 4, lanjut ke `FASE-5-gRPC-Protobuf.md`*

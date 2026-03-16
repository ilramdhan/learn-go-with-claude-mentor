# 📘 FASE 3: Gin Framework & REST API

> **Prasyarat:** Selesaikan Fase 1 + Fase 2  
> **Durasi:** 3–4 minggu  
> **Project Akhir:** REST API Blog Platform (lihat PRD di file terpisah)  
> **Tujuan:** Membangun production-ready REST API menggunakan Gin, PostgreSQL, dan best practices industri

---

## 🗂️ Daftar Modul

| # | Modul | Topik |
|---|-------|-------|
| 3.1 | Gin Basics | routing, handler, response |
| 3.2 | Request Handling | params, query, body, binding |
| 3.3 | Middleware | custom, built-in, CORS, rate limit |
| 3.4 | Database + GORM | PostgreSQL, CRUD, migration, transaction |
| 3.5 | Repository Pattern | interface abstraksi database |
| 3.6 | JWT Authentication | generate, validate, refresh token |
| 3.7 | Consistent Error Response | format error yang seragam |
| 3.8 | Validation | binding tags, custom validators |
| 3.9 | File Upload | single/multiple upload, MIME validation |
| 3.10 | API Versioning | URL path versioning, deprecation |
| 3.11 | Configuration | Viper, env vars, config file |
| 3.12 | Swagger/OpenAPI | swaggo, auto-generate docs |
| 3.13 | Testing REST API | httptest, mock repository |

---







## 📦 Modul 3.1 — Gin Basics

### Setup

```bash
mkdir blog-api
cd blog-api
go mod init github.com/kamu/blog-api
go get github.com/gin-gonic/gin
```

### Hello Gin

```go
// main.go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func main() {
    // Buat router
    r := gin.Default() // includes Logger + Recovery middleware

    // Route sederhana
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{
            "message": "pong",
        })
    })

    // Jalankan server
    r.Run(":8080") // default port 8080
}
```

```bash
go run main.go
# curl http://localhost:8080/ping
# {"message":"pong"}
```

### Routing Lengkap

```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()

    // === HTTP Methods ===
    r.GET("/users", getUsers)
    r.POST("/users", createUser)
    r.PUT("/users/:id", updateUser)
    r.PATCH("/users/:id", patchUser)
    r.DELETE("/users/:id", deleteUser)

    // === Route Groups ===
    v1 := r.Group("/api/v1")
    {
        v1.GET("/posts", getPosts)
        v1.POST("/posts", createPost)

        // Nested group
        posts := v1.Group("/posts")
        {
            posts.GET("/:id", getPost)
            posts.PUT("/:id", updatePost)
            posts.DELETE("/:id", deletePost)

            // Nested lebih dalam
            posts.GET("/:id/comments", getPostComments)
        }
    }

    // === Static Files ===
    r.Static("/assets", "./assets")
    r.StaticFile("/favicon.ico", "./favicon.ico")

    // === 404 Handler ===
    r.NoRoute(func(c *gin.Context) {
        c.JSON(http.StatusNotFound, gin.H{
            "error": "route not found",
        })
    })

    r.Run(":8080")
}

// Handler functions
func getUsers(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"users": []string{"Alice", "Bob"}})
}

func createUser(c *gin.Context) {
    c.JSON(http.StatusCreated, gin.H{"message": "user created"})
}

func updateUser(c *gin.Context) {
    id := c.Param("id")
    c.JSON(http.StatusOK, gin.H{"message": "user " + id + " updated"})
}

func patchUser(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"message": "user partially updated"})
}

func deleteUser(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"message": "user deleted"})
}

func getPosts(c *gin.Context)        { c.JSON(200, gin.H{"posts": []string{}}) }
func createPost(c *gin.Context)      { c.JSON(201, gin.H{"message": "created"}) }
func getPost(c *gin.Context)         { c.JSON(200, gin.H{"post": c.Param("id")}) }
func updatePost(c *gin.Context)      { c.JSON(200, gin.H{"message": "updated"}) }
func deletePost(c *gin.Context)      { c.JSON(200, gin.H{"message": "deleted"}) }
func getPostComments(c *gin.Context) { c.JSON(200, gin.H{"comments": []string{}}) }
```

### Response Helpers

```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()

    // JSON response
    r.GET("/json", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{
            "status": "ok",
            "data": map[string]interface{}{
                "id":   1,
                "name": "Alice",
            },
        })
    })

    // String response
    r.GET("/text", func(c *gin.Context) {
        c.String(http.StatusOK, "Hello, %s!", "World")
    })

    // HTML response
    r.GET("/html", func(c *gin.Context) {
        c.Data(http.StatusOK, "text/html", []byte("<h1>Hello</h1>"))
    })

    // Redirect
    r.GET("/redirect", func(c *gin.Context) {
        c.Redirect(http.StatusMovedPermanently, "/json")
    })

    // Set header
    r.GET("/header", func(c *gin.Context) {
        c.Header("X-Custom-Header", "my-value")
        c.JSON(http.StatusOK, gin.H{"message": "ok"})
    })

    // Abort — stop middleware chain
    r.GET("/abort", func(c *gin.Context) {
        c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
            "error": "unauthorized",
        })
    })

    r.Run(":8080")
}
```

---


### 🏋️ Latihan 3.1

1. Buat Gin server minimal yang: (a) jalankan di port 8080, (b) handle `GET /` yang return JSON `{"message":"Hello, Go!"}`, (c) handle route yang tidak ada dengan 404 JSON response. Verifikasi dengan curl.
2. Tambahkan **logging middleware** custom: log setiap request dengan method, path, status code, dan latency. Format: `2025/01/15 10:30:00 | 200 | 1.23ms | GET /api/users`.
3. Buat router dengan **route grouping**: group `/api/v1/public` (tanpa auth) dan `/api/v1/private` (dengan dummy auth check). Tambahkan minimal 2 endpoint di setiap group.


## 📦 Modul 3.2 — Request Handling

### Path Parameters, Query, Body

```go
package main

import (
    "fmt"
    "net/http"
    "strconv"

    "github.com/gin-gonic/gin"
)

// ===== Path Parameters =====
// GET /users/123
func getUserByID(c *gin.Context) {
    id := c.Param("id")

    // Convert ke int
    userID, err := strconv.Atoi(id)
    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "invalid id"})
        return
    }

    c.JSON(http.StatusOK, gin.H{"user_id": userID})
}

// GET /users/alice/posts/5
func getUserPost(c *gin.Context) {
    username := c.Param("username")
    postID := c.Param("postID")
    c.JSON(http.StatusOK, gin.H{
        "username": username,
        "post_id":  postID,
    })
}

// ===== Query Parameters =====
// GET /posts?page=1&per_page=10&status=published&sort=created_at
func getPosts(c *gin.Context) {
    // Dengan default value
    page := c.DefaultQuery("page", "1")
    perPage := c.DefaultQuery("per_page", "10")
    status := c.Query("status") // "" jika tidak ada
    sort := c.DefaultQuery("sort", "created_at")

    pageInt, _ := strconv.Atoi(page)
    perPageInt, _ := strconv.Atoi(perPage)

    fmt.Printf("page=%d, per_page=%d, status=%s, sort=%s\n",
        pageInt, perPageInt, status, sort)

    c.JSON(http.StatusOK, gin.H{
        "page":     pageInt,
        "per_page": perPageInt,
        "status":   status,
        "sort":     sort,
    })
}

// ===== Request Body Binding =====

type CreateUserRequest struct {
    Name     string `json:"name" binding:"required,min=2,max=50"`
    Email    string `json:"email" binding:"required,email"`
    Password string `json:"password" binding:"required,min=8"`
    Age      int    `json:"age" binding:"omitempty,min=0,max=150"`
}

func createUser(c *gin.Context) {
    var req CreateUserRequest

    // ShouldBindJSON: return error jika binding gagal
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    // Proses request
    c.JSON(http.StatusCreated, gin.H{
        "message": "user created",
        "user": gin.H{
            "name":  req.Name,
            "email": req.Email,
        },
    })
}

// ===== Request Headers =====
func getWithHeaders(c *gin.Context) {
    // Baca header
    contentType := c.GetHeader("Content-Type")
    authHeader := c.GetHeader("Authorization")
    customHeader := c.GetHeader("X-Request-ID")

    fmt.Printf("Content-Type: %s, Auth: %s, RequestID: %s\n",
        contentType, authHeader, customHeader)

    c.JSON(http.StatusOK, gin.H{"status": "ok"})
}

// ===== Binding dari berbagai sumber =====

// Form data
type LoginRequest struct {
    Username string `form:"username" binding:"required"`
    Password string `form:"password" binding:"required"`
}

func loginWithForm(c *gin.Context) {
    var req LoginRequest
    if err := c.ShouldBind(&req); err != nil { // Bind dari form atau JSON
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    c.JSON(http.StatusOK, gin.H{"username": req.Username})
}

// Query string binding
type PaginationRequest struct {
    Page    int    `form:"page" binding:"omitempty,min=1"`
    PerPage int    `form:"per_page" binding:"omitempty,min=1,max=100"`
    Search  string `form:"search"`
}

func getPostsWithBinding(c *gin.Context) {
    var req PaginationRequest
    req.Page = 1    // default
    req.PerPage = 10

    if err := c.ShouldBindQuery(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusOK, req)
}

func main() {
    r := gin.Default()

    r.GET("/users/:id", getUserByID)
    r.GET("/users/:username/posts/:postID", getUserPost)
    r.GET("/posts", getPosts)
    r.GET("/posts/binding", getPostsWithBinding)
    r.POST("/users", createUser)
    r.POST("/login", loginWithForm)
    r.GET("/headers", getWithHeaders)

    r.Run(":8080")
}
```

---


### 🏋️ Latihan 3.2

1. Buat endpoint `POST /users` yang menerima JSON body dengan field: `name`, `email`, `age`. Validasi: name tidak kosong, email format valid, age antara 18-100. Return 400 jika invalid dengan pesan error per field.
2. Buat endpoint `GET /users/:id/profile` yang menerima path param `id` (uint) dan query params `fields=name,email` untuk select fields mana yang di-return. Handle: id bukan angka → 400, id tidak ada → 404.
3. Buat endpoint yang menerima semua HTTP methods (GET, POST, PUT, DELETE) untuk resource yang sama dan return method yang dipakai dalam response.


## 📦 Modul 3.3 — Middleware

Middleware adalah fungsi yang berjalan **sebelum** dan/atau **sesudah** handler.

```go
package main

import (
    "log"
    "net/http"
    "strings"
    "time"

    "github.com/gin-gonic/gin"
)

// ===== Custom Middleware =====

// Logger middleware
func LoggerMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()
        path := c.Request.URL.Path
        method := c.Request.Method

        // Jalankan handler berikutnya
        c.Next()

        // Setelah handler selesai
        duration := time.Since(start)
        status := c.Writer.Status()

        log.Printf("[%d] %s %s %v", status, method, path, duration)
    }
}

// Request ID middleware
func RequestIDMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        requestID := c.GetHeader("X-Request-ID")
        if requestID == "" {
            requestID = generateRequestID()
        }

        // Set ke context — bisa diambil di handler
        c.Set("request_id", requestID)

        // Set ke response header
        c.Header("X-Request-ID", requestID)

        c.Next()
    }
}

func generateRequestID() string {
    return fmt.Sprintf("req-%d", time.Now().UnixNano())
}

// CORS middleware
func CORSMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Header("Access-Control-Allow-Origin", "*")
        c.Header("Access-Control-Allow-Methods", "GET, POST, PUT, PATCH, DELETE, OPTIONS")
        c.Header("Access-Control-Allow-Headers", "Content-Type, Authorization, X-Request-ID")

        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(http.StatusNoContent)
            return
        }

        c.Next()
    }
}

// Rate limiter middleware (simplified)
type RateLimiter struct {
    requests map[string][]time.Time
}

func NewRateLimiter() *RateLimiter {
    return &RateLimiter{requests: make(map[string][]time.Time)}
}

func (rl *RateLimiter) Middleware(maxRequests int, window time.Duration) gin.HandlerFunc {
    return func(c *gin.Context) {
        ip := c.ClientIP()
        now := time.Now()

        // Hapus request yang sudah di luar window
        var recent []time.Time
        for _, t := range rl.requests[ip] {
            if now.Sub(t) < window {
                recent = append(recent, t)
            }
        }

        if len(recent) >= maxRequests {
            c.AbortWithStatusJSON(http.StatusTooManyRequests, gin.H{
                "error": "rate limit exceeded",
            })
            return
        }

        rl.requests[ip] = append(recent, now)
        c.Next()
    }
}

// JWT Auth middleware (preview — detail di modul 3.6)
func AuthRequired() gin.HandlerFunc {
    return func(c *gin.Context) {
        authHeader := c.GetHeader("Authorization")

        if authHeader == "" {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "error": "authorization header required",
            })
            return
        }

        // Format: "Bearer <token>"
        parts := strings.SplitN(authHeader, " ", 2)
        if len(parts) != 2 || parts[0] != "Bearer" {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "error": "invalid authorization format",
            })
            return
        }

        token := parts[1]
        // TODO: validate token (akan dibahas di modul 3.6)

        // Set user info ke context
        c.Set("user_id", 1) // contoh
        c.Set("token", token)

        c.Next()
    }
}

// Recovery middleware dengan custom response
func RecoveryMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        defer func() {
            if err := recover(); err != nil {
                log.Printf("PANIC: %v", err)
                c.AbortWithStatusJSON(http.StatusInternalServerError, gin.H{
                    "error": "internal server error",
                })
            }
        }()
        c.Next()
    }
}

func main() {
    // Mode production: matikan debug logs
    // gin.SetMode(gin.ReleaseMode)

    r := gin.New() // gin.New() tanpa middleware bawaan

    // Global middlewares
    r.Use(RecoveryMiddleware())
    r.Use(LoggerMiddleware())
    r.Use(CORSMiddleware())
    r.Use(RequestIDMiddleware())

    rateLimiter := NewRateLimiter()
    r.Use(rateLimiter.Middleware(100, time.Minute))

    // Public routes (tanpa auth)
    r.POST("/login", loginHandler)
    r.POST("/register", registerHandler)

    // Protected routes (dengan auth middleware)
    protected := r.Group("/api")
    protected.Use(AuthRequired())
    {
        protected.GET("/profile", profileHandler)
        protected.PUT("/profile", updateProfileHandler)
    }

    // Admin routes (auth + admin check)
    admin := r.Group("/admin")
    admin.Use(AuthRequired(), AdminRequired())
    {
        admin.GET("/users", adminGetUsers)
    }

    r.Run(":8080")
}

func AdminRequired() gin.HandlerFunc {
    return func(c *gin.Context) {
        // Check dari context yang di-set oleh AuthRequired
        // userRole := c.GetString("user_role")
        // if userRole != "admin" {
        //     c.AbortWithStatusJSON(403, gin.H{"error": "forbidden"})
        //     return
        // }
        c.Next()
    }
}

func loginHandler(c *gin.Context)         { c.JSON(200, gin.H{"token": "xxx"}) }
func registerHandler(c *gin.Context)      { c.JSON(201, gin.H{"message": "registered"}) }
func profileHandler(c *gin.Context)       { c.JSON(200, gin.H{"profile": "data"}) }
func updateProfileHandler(c *gin.Context) { c.JSON(200, gin.H{"message": "updated"}) }
func adminGetUsers(c *gin.Context)        { c.JSON(200, gin.H{"users": []string{}}) }
```

---


### 🏋️ Latihan 3.3

1. Buat **CORS middleware** yang: allow origin `http://localhost:3000` dan `https://myapp.com`, allow methods GET/POST/PUT/DELETE, allow headers Authorization dan Content-Type. Test dengan browser atau curl dengan `-H "Origin: http://localhost:3000"`.
2. Buat **rate limiter middleware** menggunakan Redis (atau in-memory map dengan sync.Mutex): limit 10 request/menit per IP. Return 429 dengan header `X-RateLimit-Remaining` dan `X-RateLimit-Reset`.
3. Buat **recovery middleware** yang: catch panic, log stack trace, return 500 JSON response. Test dengan endpoint yang sengaja panic.


## 📦 Modul 3.4 — Database + GORM

### Setup PostgreSQL + GORM

```bash
go get gorm.io/gorm
go get gorm.io/driver/postgres
```

Docker Compose untuk PostgreSQL:
```yaml
# docker-compose.yml
version: '3.8'
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: bloguser
      POSTGRES_PASSWORD: blogpassword
      POSTGRES_DB: blogdb
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

```bash
docker compose up -d
```

### Model & Migration

```go
// model/user.go
package model

import (
    "time"
    "gorm.io/gorm"
)

type User struct {
    ID        uint           `gorm:"primarykey" json:"id"`
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
    DeletedAt gorm.DeletedAt `gorm:"index" json:"-"` // soft delete

    Name     string `gorm:"size:100;not null" json:"name"`
    Email    string `gorm:"size:255;uniqueIndex;not null" json:"email"`
    Password string `gorm:"size:255;not null" json:"-"` // jangan expose
    Role     string `gorm:"size:20;default:user" json:"role"`
    IsActive bool   `gorm:"default:true" json:"is_active"`

    Posts []Post `gorm:"foreignKey:UserID" json:"posts,omitempty"`
}

// model/post.go
type Post struct {
    ID        uint           `gorm:"primarykey" json:"id"`
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
    DeletedAt gorm.DeletedAt `gorm:"index" json:"-"`

    Title    string `gorm:"size:255;not null" json:"title"`
    Slug     string `gorm:"size:255;uniqueIndex;not null" json:"slug"`
    Content  string `gorm:"type:text" json:"content"`
    Excerpt  string `gorm:"size:500" json:"excerpt"`
    Status   string `gorm:"size:20;default:draft" json:"status"` // draft, published
    ViewCount int   `gorm:"default:0" json:"view_count"`

    UserID uint `gorm:"not null" json:"user_id"`
    User   User `gorm:"foreignKey:UserID" json:"author,omitempty"`

    Tags     []Tag     `gorm:"many2many:post_tags;" json:"tags,omitempty"`
    Comments []Comment `gorm:"foreignKey:PostID" json:"comments,omitempty"`
}

// model/tag.go
type Tag struct {
    ID   uint   `gorm:"primarykey" json:"id"`
    Name string `gorm:"size:50;uniqueIndex;not null" json:"name"`
    Slug string `gorm:"size:50;uniqueIndex;not null" json:"slug"`

    Posts []Post `gorm:"many2many:post_tags;" json:"-"`
}

// model/comment.go
type Comment struct {
    ID        uint      `gorm:"primarykey" json:"id"`
    CreatedAt time.Time `json:"created_at"`

    Content string `gorm:"type:text;not null" json:"content"`

    PostID uint `gorm:"not null" json:"post_id"`
    UserID uint `gorm:"not null" json:"user_id"`
    User   User `gorm:"foreignKey:UserID" json:"author,omitempty"`
}
```

### Database Connection & Migration

```go
// database/db.go
package database

import (
    "fmt"
    "log"

    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    "gorm.io/gorm/logger"
    
    "github.com/kamu/blog-api/model"
)

var DB *gorm.DB

func Connect(dsn string) (*gorm.DB, error) {
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{
        Logger: logger.Default.LogMode(logger.Info),
    })
    if err != nil {
        return nil, fmt.Errorf("database connection failed: %w", err)
    }

    // Connection pool settings
    sqlDB, err := db.DB()
    if err != nil {
        return nil, err
    }
    sqlDB.SetMaxOpenConns(25)
    sqlDB.SetMaxIdleConns(5)
    sqlDB.SetConnMaxLifetime(5 * time.Minute)

    return db, nil
}

func Migrate(db *gorm.DB) error {
    return db.AutoMigrate(
        &model.User{},
        &model.Post{},
        &model.Tag{},
        &model.Comment{},
    )
}

// DSN format: "host=localhost user=bloguser password=blogpassword dbname=blogdb port=5432 sslmode=disable"
```

### GORM CRUD Operations

```go
// Contoh operasi GORM

// === CREATE ===
user := model.User{
    Name:     "Alice",
    Email:    "alice@example.com",
    Password: "hashed_password",
}
result := db.Create(&user)
// user.ID terisi otomatis setelah Create

// Batch create
users := []model.User{
    {Name: "Bob", Email: "bob@example.com"},
    {Name: "Charlie", Email: "charlie@example.com"},
}
db.Create(&users)

// === READ ===
// Cari berdasarkan primary key
var user model.User
db.First(&user, 1)           // WHERE id = 1
db.First(&user, "id = ?", 1) // sama
db.First(&user)               // ORDER BY id LIMIT 1

// Cari semua
var users []model.User
db.Find(&users)
db.Find(&users, "role = ?", "admin")

// Dengan kondisi
db.Where("name = ?", "Alice").First(&user)
db.Where("age > ? AND role = ?", 18, "user").Find(&users)
db.Where("name LIKE ?", "%Ali%").Find(&users)

// Dengan order, limit, offset
db.Order("created_at desc").Limit(10).Offset(20).Find(&users)

// Select kolom tertentu
db.Select("id", "name", "email").Find(&users)

// Preload (eager loading)
db.Preload("Posts").First(&user, 1) // load Posts juga
db.Preload("Posts.Tags").First(&user, 1) // nested preload
db.Preload("Posts", "status = ?", "published").Find(&users)

// === UPDATE ===
// Update semua field
db.Save(&user) // UPDATE semua kolom

// Update kolom tertentu
db.Model(&user).Update("name", "New Name")
db.Model(&user).Updates(map[string]interface{}{
    "name":  "New Name",
    "email": "new@example.com",
})
db.Model(&user).Updates(model.User{Name: "New Name"}) // hanya update non-zero fields

// Update tanpa model (WHERE clause)
db.Model(&model.User{}).Where("role = ?", "guest").Update("is_active", false)

// === DELETE ===
db.Delete(&user)           // soft delete (set DeletedAt)
db.Unscoped().Delete(&user) // hard delete

// Bulk delete
db.Where("created_at < ?", time.Now().Add(-30*24*time.Hour)).Delete(&model.Post{})

// === COUNT ===
var count int64
db.Model(&model.User{}).Count(&count)
db.Model(&model.Post{}).Where("status = ?", "published").Count(&count)

// === Raw SQL ===
var users []model.User
db.Raw("SELECT * FROM users WHERE age > ? ORDER BY name", 18).Scan(&users)

// Exec (untuk INSERT/UPDATE/DELETE)
db.Exec("UPDATE users SET role = ? WHERE id IN ?", "moderator", []int{1, 2, 3})

// === Transactions ===
err := db.Transaction(func(tx *gorm.DB) error {
    if err := tx.Create(&user).Error; err != nil {
        return err // rollback otomatis
    }
    if err := tx.Create(&post).Error; err != nil {
        return err // rollback
    }
    return nil // commit
})
```

---


### 🏋️ Latihan 3.4

1. Setup GORM dengan PostgreSQL. Buat model `User` dengan: soft delete, auto timestamps (created_at, updated_at). Buat migrasi (AutoMigrate) dan seed 5 users. Verifikasi data muncul di database.
2. Buat GORM callback yang otomatis: (a) set `created_at` dan `updated_at` saat create, (b) update `updated_at` saat update. Buat test yang memverifikasi callback berjalan.
3. Implementasikan **database connection pool**: `MaxOpenConns=25`, `MaxIdleConns=5`, `ConnMaxLifetime=5m`. Buat endpoint `GET /health/db` yang menampilkan stats pool (open, idle, in-use connections).


## 📦 Modul 3.5 — Repository Pattern

Repository memisahkan logika database dari business logic.

```go
// repository/user_repository.go
package repository

import (
    "context"
    "fmt"
    "gorm.io/gorm"
    "github.com/kamu/blog-api/model"
)

// Interface — mudah di-mock untuk testing
type UserRepository interface {
    Create(ctx context.Context, user *model.User) error
    FindByID(ctx context.Context, id uint) (*model.User, error)
    FindByEmail(ctx context.Context, email string) (*model.User, error)
    Update(ctx context.Context, user *model.User) error
    Delete(ctx context.Context, id uint) error
    FindAll(ctx context.Context, opts FindOptions) ([]model.User, int64, error)
}

type FindOptions struct {
    Page    int
    PerPage int
    Search  string
    Role    string
    OrderBy string
}

// Implementasi konkret
type userRepository struct {
    db *gorm.DB
}

func NewUserRepository(db *gorm.DB) UserRepository {
    return &userRepository{db: db}
}

func (r *userRepository) Create(ctx context.Context, user *model.User) error {
    return r.db.WithContext(ctx).Create(user).Error
}

func (r *userRepository) FindByID(ctx context.Context, id uint) (*model.User, error) {
    var user model.User
    result := r.db.WithContext(ctx).First(&user, id)
    if result.Error != nil {
        if errors.Is(result.Error, gorm.ErrRecordNotFound) {
            return nil, fmt.Errorf("user with id %d not found", id)
        }
        return nil, result.Error
    }
    return &user, nil
}

func (r *userRepository) FindByEmail(ctx context.Context, email string) (*model.User, error) {
    var user model.User
    result := r.db.WithContext(ctx).Where("email = ?", email).First(&user)
    if result.Error != nil {
        if errors.Is(result.Error, gorm.ErrRecordNotFound) {
            return nil, nil // nil, nil artinya tidak ditemukan (bukan error)
        }
        return nil, result.Error
    }
    return &user, nil
}

func (r *userRepository) Update(ctx context.Context, user *model.User) error {
    return r.db.WithContext(ctx).Save(user).Error
}

func (r *userRepository) Delete(ctx context.Context, id uint) error {
    return r.db.WithContext(ctx).Delete(&model.User{}, id).Error
}

func (r *userRepository) FindAll(ctx context.Context, opts FindOptions) ([]model.User, int64, error) {
    var users []model.User
    var total int64

    query := r.db.WithContext(ctx).Model(&model.User{})

    // Filter
    if opts.Search != "" {
        query = query.Where("name ILIKE ? OR email ILIKE ?",
            "%"+opts.Search+"%", "%"+opts.Search+"%")
    }
    if opts.Role != "" {
        query = query.Where("role = ?", opts.Role)
    }

    // Count total
    query.Count(&total)

    // Pagination
    if opts.Page > 0 && opts.PerPage > 0 {
        offset := (opts.Page - 1) * opts.PerPage
        query = query.Limit(opts.PerPage).Offset(offset)
    }

    // Order
    orderBy := "created_at DESC"
    if opts.OrderBy != "" {
        orderBy = opts.OrderBy
    }
    query = query.Order(orderBy)

    err := query.Find(&users).Error
    return users, total, err
}
```

---


### Pagination — Wajib untuk List Endpoints

```go
// internal/delivery/http/request/pagination.go
package request

import (
    "math"
    "github.com/gin-gonic/gin"
)

// PaginationRequest adalah parameter standar untuk semua list endpoint
type PaginationRequest struct {
    Page    int    `form:"page"    binding:"min=1"`
    PerPage int    `form:"per_page" binding:"min=1,max=100"`
    SortBy  string `form:"sort_by"`
    SortDir string `form:"sort_dir" binding:"omitempty,oneof=asc desc"`
}

// WithDefaults mengisi nilai default jika tidak di-provide
func (p *PaginationRequest) WithDefaults() *PaginationRequest {
    if p.Page == 0 { p.Page = 1 }
    if p.PerPage == 0 { p.PerPage = 10 }
    if p.SortDir == "" { p.SortDir = "desc" }
    return p
}

// Offset menghitung OFFSET untuk query SQL
func (p *PaginationRequest) Offset() int {
    return (p.Page - 1) * p.PerPage
}

// PaginationResponse adalah struktur response untuk list endpoint
type PaginationResponse[T any] struct {
    Data       []T   `json:"data"`
    Total      int64 `json:"total"`
    Page       int   `json:"page"`
    PerPage    int   `json:"per_page"`
    TotalPages int   `json:"total_pages"`
    HasNext    bool  `json:"has_next"`
    HasPrev    bool  `json:"has_prev"`
}

// NewPagination membuat PaginationResponse dari data dan total
func NewPagination[T any](data []T, total int64, req *PaginationRequest) *PaginationResponse[T] {
    totalPages := int(math.Ceil(float64(total) / float64(req.PerPage)))
    return &PaginationResponse[T]{
        Data:       data,
        Total:      total,
        Page:       req.Page,
        PerPage:    req.PerPage,
        TotalPages: totalPages,
        HasNext:    req.Page < totalPages,
        HasPrev:    req.Page > 1,
    }
}
```

```go
// Penggunaan di handler
func (h *ProductHandler) List(c *gin.Context) {
    var req request.PaginationRequest
    if err := c.ShouldBindQuery(&req); err != nil {
        response.ValidationError(c, err)
        return
    }
    req.WithDefaults()

    products, total, err := h.listProductsUC.Execute(c.Request.Context(), usecase.ListProductsInput{
        Page:    req.Page,
        PerPage: req.PerPage,
        SortBy:  req.SortBy,
        SortDir: req.SortDir,
    })
    if err != nil {
        response.InternalError(c, err)
        return
    }

    response.OK(c, request.NewPagination(products, total, &req))
}

// Repository dengan pagination
func (r *productRepository) FindAll(ctx context.Context, input FindAllInput) ([]*entity.Product, int64, error) {
    var products []model.ProductModel
    var total int64

    q := r.db.WithContext(ctx).Model(&model.ProductModel{})

    // Apply filters
    if input.Category != "" {
        q = q.Where("category = ?", input.Category)
    }
    if input.Search != "" {
        q = q.Where("name ILIKE ? OR description ILIKE ?",
            "%"+input.Search+"%", "%"+input.Search+"%")
    }

    // Count total sebelum pagination
    q.Count(&total)

    // Apply sorting dan pagination
    sortCol := "created_at"
    if input.SortBy != "" { sortCol = input.SortBy }
    q = q.Order(sortCol + " " + input.SortDir).
        Limit(input.PerPage).
        Offset(input.Offset)

    if err := q.Find(&products).Error; err != nil {
        return nil, 0, err
    }

    return toProductEntities(products), total, nil
}

// Response contoh:
// {
//   "success": true,
//   "data": {
//     "data": [...],
//     "total": 150,
//     "page": 2,
//     "per_page": 10,
//     "total_pages": 15,
//     "has_next": true,
//     "has_prev": true
//   }
// }
```


### 🏋️ Latihan 3.5

1. Implementasikan `UserRepository` interface dengan method: `Create`, `FindByID`, `FindByEmail`, `Update`, `Delete`. Implementasikan dengan GORM. Tulis unit test menggunakan mock repository.
2. Buat `ProductRepository` dengan method `Search(ctx, query string, page, perPage int) ([]*Product, int64, error)` yang support full-text search pada field name dan description. Gunakan GORM LIKE query.
3. Implementasikan **repository caching layer**: buat `CachedUserRepository` yang wrap `UserRepository`, cache `FindByID` di Redis selama 5 menit. Cache invalidate saat `Update` atau `Delete` dipanggil.


## 📦 Modul 3.6 — JWT Authentication

```bash
go get github.com/golang-jwt/jwt/v5
go get golang.org/x/crypto
```

```go
// auth/jwt.go
package auth

import (
    "errors"
    "time"

    "github.com/golang-jwt/jwt/v5"
)

type Claims struct {
    UserID uint   `json:"user_id"`
    Email  string `json:"email"`
    Role   string `json:"role"`
    jwt.RegisteredClaims
}

type JWTService struct {
    secretKey     []byte
    accessExpiry  time.Duration
    refreshExpiry time.Duration
}

func NewJWTService(secretKey string) *JWTService {
    return &JWTService{
        secretKey:     []byte(secretKey),
        accessExpiry:  15 * time.Minute,
        refreshExpiry: 7 * 24 * time.Hour,
    }
}

func (j *JWTService) GenerateToken(userID uint, email, role string) (string, error) {
    claims := Claims{
        UserID: userID,
        Email:  email,
        Role:   role,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(j.accessExpiry)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
            Issuer:    "blog-api",
        },
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(j.secretKey)
}

func (j *JWTService) ValidateToken(tokenStr string) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenStr, &Claims{}, func(token *jwt.Token) (interface{}, error) {
        if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
        }
        return j.secretKey, nil
    })

    if err != nil {
        return nil, err
    }

    claims, ok := token.Claims.(*Claims)
    if !ok || !token.Valid {
        return nil, errors.New("invalid token")
    }

    return claims, nil
}
```

```go
// Password hashing
// auth/password.go
package auth

import "golang.org/x/crypto/bcrypt"

func HashPassword(password string) (string, error) {
    bytes, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    return string(bytes), err
}

func CheckPassword(password, hash string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
    return err == nil
}
```

```go
// middleware/auth.go
package middleware

import (
    "net/http"
    "strings"
    "github.com/gin-gonic/gin"
    "github.com/kamu/blog-api/auth"
)

func JWTAuthMiddleware(jwtService *auth.JWTService) gin.HandlerFunc {
    return func(c *gin.Context) {
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "error": "authorization header is required",
            })
            return
        }

        parts := strings.SplitN(authHeader, " ", 2)
        if len(parts) != 2 || parts[0] != "Bearer" {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "error": "authorization format must be Bearer {token}",
            })
            return
        }

        claims, err := jwtService.ValidateToken(parts[1])
        if err != nil {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{
                "error": "invalid or expired token",
            })
            return
        }

        // Set ke context
        c.Set("user_id", claims.UserID)
        c.Set("user_email", claims.Email)
        c.Set("user_role", claims.Role)
        c.Next()
    }
}

// Helper untuk get user ID dari context
func GetUserID(c *gin.Context) uint {
    userID, _ := c.Get("user_id")
    return userID.(uint)
}

func GetUserRole(c *gin.Context) string {
    role, _ := c.Get("user_role")
    return role.(string)
}
```

---


### Refresh Token Implementation

```go
// internal/domain/entity/refresh_token.go
type RefreshToken struct {
    ID        uint      `gorm:"primarykey"`
    UserID    uint      `gorm:"not null;index"`
    Token     string    `gorm:"size:255;not null;uniqueIndex"`
    ExpiresAt time.Time `gorm:"not null"`
    CreatedAt time.Time
    RevokedAt *time.Time
    UserAgent string    `gorm:"size:255"`
    IPAddress string    `gorm:"size:45"`
}

func (r *RefreshToken) IsExpired() bool { return time.Now().After(r.ExpiresAt) }
func (r *RefreshToken) IsRevoked() bool { return r.RevokedAt != nil }
func (r *RefreshToken) IsValid() bool   { return !r.IsExpired() && !r.IsRevoked() }

// internal/usecase/auth/refresh_token.go
type RefreshTokenUseCase struct {
    refreshRepo RefreshTokenRepository
    userRepo    UserRepository
    jwtSvc      JWTService
}

type RefreshTokenInput struct {
    RefreshToken string
    UserAgent    string
    IPAddress    string
}

type RefreshTokenOutput struct {
    AccessToken  string
    RefreshToken string // NEW refresh token (rotation!)
    ExpiresAt    time.Time
}

func (uc *RefreshTokenUseCase) Execute(ctx context.Context, input RefreshTokenInput) (*RefreshTokenOutput, error) {
    // 1. Cari refresh token di database
    rt, err := uc.refreshRepo.FindByToken(ctx, input.RefreshToken)
    if err != nil || rt == nil || !rt.IsValid() {
        return nil, apperror.ErrInvalidRefreshToken
    }

    // 2. Load user
    user, err := uc.userRepo.FindByID(ctx, rt.UserID)
    if err != nil || user == nil || !user.IsActive {
        return nil, apperror.ErrUserNotFound
    }

    // 3. REVOKE token lama (refresh token rotation)
    now := time.Now()
    rt.RevokedAt = &now
    uc.refreshRepo.Update(ctx, rt)

    // 4. Generate token baru
    accessToken, err := uc.jwtSvc.GenerateAccessToken(user.ID, user.Email, user.Role)
    if err != nil {
        return nil, err
    }

    newRefreshToken, expiresAt, err := uc.jwtSvc.GenerateRefreshToken()
    if err != nil {
        return nil, err
    }

    // 5. Simpan refresh token baru
    newRT := &RefreshToken{
        UserID:    user.ID,
        Token:     newRefreshToken,
        ExpiresAt: expiresAt,
        UserAgent: input.UserAgent,
        IPAddress: input.IPAddress,
    }
    uc.refreshRepo.Save(ctx, newRT)

    return &RefreshTokenOutput{
        AccessToken:  accessToken,
        RefreshToken: newRefreshToken,
        ExpiresAt:    expiresAt,
    }, nil
}
```


### 🏋️ Latihan 3.6

1. Implementasikan JWT authentication lengkap: (a) `POST /auth/register` buat user baru, (b) `POST /auth/login` return access_token (15 menit) dan refresh_token (7 hari), (c) `GET /auth/me` return user info (protected).
2. Implementasikan **refresh token rotation**: `POST /auth/refresh` menerima refresh_token, return access_token baru dan refresh_token baru. Refresh_token lama langsung invalid setelah dipakai (rotation).
3. Implementasikan **token blacklisting**: saat logout (`POST /auth/logout`), simpan token ke Redis blacklist dengan TTL = sisa waktu token. Middleware cek blacklist sebelum accept request.


## 📦 Modul 3.7 — Consistent Error Response

```go
// response/response.go
package response

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

// Standard API Response
type Response struct {
    Success bool        `json:"success"`
    Message string      `json:"message,omitempty"`
    Data    interface{} `json:"data,omitempty"`
    Error   *ErrorInfo  `json:"error,omitempty"`
    Meta    *Meta       `json:"meta,omitempty"`
}

type ErrorInfo struct {
    Code    string      `json:"code"`
    Message string      `json:"message"`
    Details interface{} `json:"details,omitempty"`
}

type Meta struct {
    Page       int   `json:"page"`
    PerPage    int   `json:"per_page"`
    Total      int64 `json:"total"`
    TotalPages int   `json:"total_pages"`
}

// Success responses
func OK(c *gin.Context, data interface{}) {
    c.JSON(http.StatusOK, Response{
        Success: true,
        Data:    data,
    })
}

func OKWithMeta(c *gin.Context, data interface{}, meta *Meta) {
    c.JSON(http.StatusOK, Response{
        Success: true,
        Data:    data,
        Meta:    meta,
    })
}

func Created(c *gin.Context, data interface{}) {
    c.JSON(http.StatusCreated, Response{
        Success: true,
        Data:    data,
    })
}

func NoContent(c *gin.Context) {
    c.Status(http.StatusNoContent)
}

// Error responses
func BadRequest(c *gin.Context, message string, details ...interface{}) {
    var d interface{}
    if len(details) > 0 {
        d = details[0]
    }
    c.JSON(http.StatusBadRequest, Response{
        Success: false,
        Error: &ErrorInfo{
            Code:    "BAD_REQUEST",
            Message: message,
            Details: d,
        },
    })
}

func Unauthorized(c *gin.Context, message string) {
    c.JSON(http.StatusUnauthorized, Response{
        Success: false,
        Error: &ErrorInfo{
            Code:    "UNAUTHORIZED",
            Message: message,
        },
    })
}

func Forbidden(c *gin.Context) {
    c.JSON(http.StatusForbidden, Response{
        Success: false,
        Error: &ErrorInfo{
            Code:    "FORBIDDEN",
            Message: "you don't have permission to perform this action",
        },
    })
}

func NotFound(c *gin.Context, resource string) {
    c.JSON(http.StatusNotFound, Response{
        Success: false,
        Error: &ErrorInfo{
            Code:    "NOT_FOUND",
            Message: resource + " not found",
        },
    })
}

func Conflict(c *gin.Context, message string) {
    c.JSON(http.StatusConflict, Response{
        Success: false,
        Error: &ErrorInfo{
            Code:    "CONFLICT",
            Message: message,
        },
    })
}

func InternalError(c *gin.Context) {
    c.JSON(http.StatusInternalServerError, Response{
        Success: false,
        Error: &ErrorInfo{
            Code:    "INTERNAL_SERVER_ERROR",
            Message: "an unexpected error occurred",
        },
    })
}
```

---


### 🏋️ Latihan 3.7

1. Buat **standard error response** untuk semua endpoint: `{"success":false,"error":{"code":"PRODUCT_NOT_FOUND","message":"Product with id 123 not found"}}`. Implementasikan dengan custom error types dan error handler middleware.
2. Buat **mapping tabel**: setiap domain error (`ErrProductNotFound`, `ErrUnauthorized`, dll) map ke HTTP status code yang tepat dan error code yang konsisten.
3. Buat **error documentation**: file `docs/error-codes.md` yang mendaftar semua error codes, HTTP status, dan contoh response-nya. Gunakan ini sebagai kontrak dengan frontend team.


## 📦 Modul 3.8 — Validation

```bash
go get github.com/go-playground/validator/v10
```

```go
// validation/validator.go
package validation

import (
    "fmt"
    "strings"

    "github.com/go-playground/validator/v10"
)

var validate *validator.Validate

func init() {
    validate = validator.New()

    // Custom validator
    validate.RegisterValidation("slug", validateSlug)
    validate.RegisterValidation("strong_password", validateStrongPassword)
}

func validateSlug(fl validator.FieldLevel) bool {
    slug := fl.Field().String()
    // hanya lowercase, angka, dan tanda hubung
    for _, c := range slug {
        if !((c >= 'a' && c <= 'z') || (c >= '0' && c <= '9') || c == '-') {
            return false
        }
    }
    return len(slug) > 0
}

func validateStrongPassword(fl validator.FieldLevel) bool {
    pwd := fl.Field().String()
    var hasUpper, hasLower, hasDigit bool
    for _, c := range pwd {
        switch {
        case c >= 'A' && c <= 'Z':
            hasUpper = true
        case c >= 'a' && c <= 'z':
            hasLower = true
        case c >= '0' && c <= '9':
            hasDigit = true
        }
    }
    return hasUpper && hasLower && hasDigit && len(pwd) >= 8
}

// Format error messages
func FormatValidationErrors(err error) map[string]string {
    errors := make(map[string]string)

    if validationErrors, ok := err.(validator.ValidationErrors); ok {
        for _, e := range validationErrors {
            field := strings.ToLower(e.Field())
            errors[field] = getErrorMessage(e)
        }
    }

    return errors
}

func getErrorMessage(e validator.FieldError) string {
    switch e.Tag() {
    case "required":
        return fmt.Sprintf("%s is required", e.Field())
    case "email":
        return "invalid email format"
    case "min":
        return fmt.Sprintf("%s must be at least %s characters", e.Field(), e.Param())
    case "max":
        return fmt.Sprintf("%s must be at most %s characters", e.Field(), e.Param())
    case "slug":
        return "only lowercase letters, numbers, and hyphens allowed"
    case "strong_password":
        return "password must contain uppercase, lowercase, and number"
    default:
        return fmt.Sprintf("invalid value for %s", e.Field())
    }
}
```

```go
// Request structs dengan validasi
type CreatePostRequest struct {
    Title   string   `json:"title" binding:"required,min=5,max=255"`
    Slug    string   `json:"slug" binding:"required,slug"`
    Content string   `json:"content" binding:"required,min=10"`
    Excerpt string   `json:"excerpt" binding:"omitempty,max=500"`
    Status  string   `json:"status" binding:"omitempty,oneof=draft published"`
    Tags    []uint   `json:"tags" binding:"omitempty"`
}

// Handler yang menggunakan validasi
func (h *PostHandler) Create(c *gin.Context) {
    var req CreatePostRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        response.BadRequest(c, "validation failed", validation.FormatValidationErrors(err))
        return
    }

    // Proses...
}
```

---


### 🏋️ Latihan 3.8

1. Buat **custom validator** menggunakan `go-playground/validator`: (a) `indophone` untuk nomor HP Indonesia (08xx atau +62xx), (b) `strongpassword` yang cek min 8 chars, ada uppercase, lowercase, dan angka. Register di Gin validator.
2. Implementasikan **struct-level validation**: validasi bahwa `end_date` selalu setelah `start_date` dalam struct `DateRange`. Test dengan request yang valid dan berbagai kasus invalid.
3. Buat **validation error formatter**: fungsi yang mengubah `validator.ValidationErrors` menjadi map `field → error_message` yang user-friendly dalam Bahasa Indonesia.


## 📦 Modul 3.9 — File Upload

File upload adalah fitur umum di REST API — untuk avatar, dokumen, atau bulk import.

### Setup

```bash
# Tidak butuh dependency tambahan, sudah ada di Gin
```

### Single File Upload

```go
// handler/upload_handler.go
package handler

import (
    "fmt"
    "io"
    "net/http"
    "os"
    "path/filepath"
    "strings"
    "time"

    "github.com/gin-gonic/gin"
)

const (
    maxFileSize    = 5 * 1024 * 1024 // 5 MB
    uploadDir      = "./uploads"
)

// Tipe file yang diizinkan berdasarkan MIME type
var allowedImageTypes = map[string]string{
    "image/jpeg": ".jpg",
    "image/png":  ".png",
    "image/gif":  ".gif",
    "image/webp": ".webp",
}

// UploadAvatar menangani upload avatar user
// POST /users/avatar
// Content-Type: multipart/form-data
func UploadAvatar(c *gin.Context) {
    // Batasi ukuran request (PENTING untuk mencegah DoS!)
    c.Request.Body = http.MaxBytesReader(c.Writer, c.Request.Body, maxFileSize)

    // Ambil file dari form
    fileHeader, err := c.FormFile("avatar")
    if err != nil {
        if strings.Contains(err.Error(), "http: request body too large") {
            c.JSON(http.StatusRequestEntityTooLarge, gin.H{
                "error": fmt.Sprintf("file too large. max size: %d MB", maxFileSize/1024/1024),
            })
            return
        }
        c.JSON(http.StatusBadRequest, gin.H{"error": "file 'avatar' is required"})
        return
    }

    // Validasi ukuran file
    if fileHeader.Size > maxFileSize {
        c.JSON(http.StatusRequestEntityTooLarge, gin.H{
            "error": fmt.Sprintf("file too large. max size: %d MB", maxFileSize/1024/1024),
        })
        return
    }

    // Buka file untuk membaca MIME type
    file, err := fileHeader.Open()
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "failed to open file"})
        return
    }
    defer file.Close()

    // Baca 512 byte pertama untuk deteksi MIME type
    // (extension bisa dipalsukan, MIME type lebih reliable)
    buffer := make([]byte, 512)
    n, err := file.Read(buffer)
    if err != nil && err != io.EOF {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "failed to read file"})
        return
    }

    mimeType := http.DetectContentType(buffer[:n])
    ext, allowed := allowedImageTypes[mimeType]
    if !allowed {
        c.JSON(http.StatusBadRequest, gin.H{
            "error": fmt.Sprintf("file type '%s' not allowed. allowed: jpeg, png, gif, webp", mimeType),
        })
        return
    }

    // Reset file pointer ke awal setelah membaca untuk deteksi MIME
    if seeker, ok := file.(io.Seeker); ok {
        seeker.Seek(0, io.SeekStart)
    }

    // Generate nama file unik untuk mencegah overwrite dan path traversal
    userID := c.GetUint("user_id") // dari JWT middleware
    filename := fmt.Sprintf("avatar_%d_%d%s", userID, time.Now().UnixNano(), ext)
    savePath := filepath.Join(uploadDir, filename)

    // Buat direktori jika belum ada
    if err := os.MkdirAll(uploadDir, 0755); err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "failed to create upload directory"})
        return
    }

    // Simpan file
    if err := c.SaveUploadedFile(fileHeader, savePath); err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "failed to save file"})
        return
    }

    // Return URL yang bisa diakses
    fileURL := fmt.Sprintf("/uploads/%s", filename)
    c.JSON(http.StatusOK, gin.H{
        "message":  "avatar uploaded successfully",
        "file_url": fileURL,
        "filename": filename,
        "size":     fileHeader.Size,
        "mime":     mimeType,
    })
}
```

### Multiple File Upload

```go
// POST /posts/:id/attachments
func UploadAttachments(c *gin.Context) {
    // Parse multipart form dengan limit 50MB total
    if err := c.Request.ParseMultipartForm(50 * 1024 * 1024); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "failed to parse form"})
        return
    }

    form, _ := c.MultipartForm()
    files := form.File["files"] // nama field di form

    if len(files) == 0 {
        c.JSON(http.StatusBadRequest, gin.H{"error": "no files provided"})
        return
    }
    if len(files) > 10 {
        c.JSON(http.StatusBadRequest, gin.H{"error": "max 10 files per upload"})
        return
    }

    var uploadedFiles []map[string]interface{}
    var errors []string

    for _, fileHeader := range files {
        if fileHeader.Size > maxFileSize {
            errors = append(errors, fmt.Sprintf("%s: too large", fileHeader.Filename))
            continue
        }

        // Sanitize filename — cegah path traversal attack!
        safeFilename := filepath.Base(fileHeader.Filename)
        safeFilename = strings.ReplaceAll(safeFilename, "..", "")
        
        uniqueName := fmt.Sprintf("%d_%s", time.Now().UnixNano(), safeFilename)
        savePath := filepath.Join(uploadDir, uniqueName)

        if err := c.SaveUploadedFile(fileHeader, savePath); err != nil {
            errors = append(errors, fmt.Sprintf("%s: save failed", fileHeader.Filename))
            continue
        }

        uploadedFiles = append(uploadedFiles, map[string]interface{}{
            "original_name": fileHeader.Filename,
            "saved_name":    uniqueName,
            "size":          fileHeader.Size,
            "url":           "/uploads/" + uniqueName,
        })
    }

    c.JSON(http.StatusOK, gin.H{
        "uploaded": uploadedFiles,
        "errors":   errors,
        "count":    len(uploadedFiles),
    })
}
```

### Serve Static Files

```go
func SetupRouter() *gin.Engine {
    r := gin.Default()

    // Serve uploaded files
    r.Static("/uploads", "./uploads")

    // Atau dengan handler custom untuk validasi akses
    r.GET("/uploads/:filename", func(c *gin.Context) {
        filename := filepath.Base(c.Param("filename")) // sanitize
        filePath := filepath.Join(uploadDir, filename)

        // Cek file exist
        if _, err := os.Stat(filePath); os.IsNotExist(err) {
            c.JSON(http.StatusNotFound, gin.H{"error": "file not found"})
            return
        }

        c.File(filePath)
    })

    return r
}
```

### Contoh Request (curl)

```bash
# Single file upload
curl -X POST http://localhost:8080/users/avatar \
  -H "Authorization: Bearer <token>" \
  -F "avatar=@/path/to/photo.jpg"

# Multiple files
curl -X POST http://localhost:8080/posts/1/attachments \
  -H "Authorization: Bearer <token>" \
  -F "files=@doc1.pdf" \
  -F "files=@doc2.pdf"
```

### 🏋️ Latihan 3.9

1. Tambahkan endpoint `POST /users/document` yang menerima upload PDF dan Word document (MIME: `application/pdf`, `application/vnd.openxmlformats-officedocument.wordprocessingml.document`). Validasi bahwa ukuran maksimal 10MB.
2. Buat fungsi `generateThumbnail(inputPath, outputPath string, maxWidth, maxHeight int) error` (gunakan package `golang.org/x/image` atau `github.com/disintegration/imaging`) yang membuat thumbnail dari gambar yang diupload.
3. Implementasikan `StorageService` interface dengan dua implementasi: `LocalStorage` (simpan ke disk lokal) dan `MockStorage` (simpan di memori untuk testing). Gunakan interface ini di upload handler.

---

## 📦 Modul 3.10 — API Versioning

Versioning API sangat penting untuk backward compatibility saat kamu merilis fitur baru.

### Strategi Versioning di Go/Gin

Ada 3 pendekatan utama. Masing-masing punya trade-off:

```go
// Strategi 1: URL Path Versioning (PALING UMUM & DIREKOMENDASIKAN)
// GET /api/v1/users
// GET /api/v2/users
// ✅ Eksplisit dan mudah debug
// ✅ Bisa cache di CDN per version
// ❌ URL "kotor"

// Strategi 2: Header Versioning
// GET /api/users
// Header: API-Version: 1
// atau: Accept: application/vnd.myapp.v1+json
// ✅ URL bersih
// ❌ Sulit ditest di browser

// Strategi 3: Query Parameter (TIDAK direkomendasikan untuk production)
// GET /api/users?version=1
// ❌ Mudah terlupakan, tidak clean
```

### Implementasi URL Path Versioning

```go
// internal/delivery/http/router.go
package http

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

// SetupRoutes menyiapkan semua route dengan versioning
func SetupRoutes(
    r *gin.Engine,
    v1UserHandler *v1.UserHandler,
    v2UserHandler *v2.UserHandler,
) {
    // Health check (tidak berversi — selalu tersedia)
    r.GET("/health", healthCheck)

    // API v1 — stable, backward compatible
    apiV1 := r.Group("/api/v1")
    {
        users := apiV1.Group("/users")
        {
            users.GET("", v1UserHandler.List)         // daftar semua user
            users.GET("/:id", v1UserHandler.GetByID)  // detail user
            users.POST("", v1UserHandler.Create)
            users.PUT("/:id", v1UserHandler.Update)
            users.DELETE("/:id", v1UserHandler.Delete)
        }
    }

    // API v2 — versi baru dengan breaking changes
    // Misal: v2 punya response format berbeda, field baru, endpoint baru
    apiV2 := r.Group("/api/v2")
    {
        users := apiV2.Group("/users")
        {
            users.GET("", v2UserHandler.List)         // response format berbeda
            users.GET("/:id", v2UserHandler.GetByID)
            users.POST("", v2UserHandler.Create)
            users.PUT("/:id", v2UserHandler.Update)
            users.DELETE("/:id", v2UserHandler.Delete)
            // Endpoint baru di v2
            users.GET("/:id/followers", v2UserHandler.GetFollowers)
            users.POST("/:id/follow", v2UserHandler.Follow)
        }
    }
}
```

### Handler yang Berbeda per Versi

```go
// internal/delivery/http/v1/user_handler.go
package v1

// V1 Response — format lama
type UserResponseV1 struct {
    ID    uint   `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

func (h *UserHandler) GetByID(c *gin.Context) {
    // ... ambil user ...

    // V1: Response flat sederhana
    c.JSON(http.StatusOK, UserResponseV1{
        ID:    user.ID,
        Name:  user.Name,
        Email: user.Email,
    })
}
```

```go
// internal/delivery/http/v2/user_handler.go
package v2

// V2 Response — format baru dengan nested object dan metadata
type UserResponseV2 struct {
    ID       uint   `json:"id"`
    Profile  Profile `json:"profile"`   // nested — breaking change dari v1!
    Contact  Contact `json:"contact"`
    Metadata Meta   `json:"_metadata"`
}

type Profile struct {
    Name      string `json:"name"`
    AvatarURL string `json:"avatar_url"`
    Bio       string `json:"bio"`
}

type Contact struct {
    Email string `json:"email"`
    Phone string `json:"phone"`
}

type Meta struct {
    Version   string `json:"version"`
    CreatedAt string `json:"created_at"`
}

func (h *UserHandler) GetByID(c *gin.Context) {
    // V2: Response lebih kaya struktur
    c.JSON(http.StatusOK, UserResponseV2{
        ID: user.ID,
        Profile: Profile{
            Name:      user.Name,
            AvatarURL: user.AvatarURL,
        },
        Contact: Contact{
            Email: user.Email,
        },
        Metadata: Meta{
            Version:   "2",
            CreatedAt: user.CreatedAt.Format(time.RFC3339),
        },
    })
}
```

### Shared Use Cases (BEST PRACTICE)

```
// Use case SAMA — hanya delivery layer yang berbeda per versi!
// Ini adalah keuntungan Clean Architecture:

UseCase (1 file, dipakai oleh v1 DAN v2)
    ↑              ↑
HTTP v1 Handler   HTTP v2 Handler
(format lama)    (format baru)
```

```go
// Use case tidak perlu diubah saat versioning
type GetUserUseCase interface {
    Execute(ctx context.Context, id uint) (*entity.User, error)
}

// V1 Handler pakai use case yang sama
type UserHandlerV1 struct {
    getUser GetUserUseCase  // interface yang sama!
}

// V2 Handler pakai use case yang sama
type UserHandlerV2 struct {
    getUser GetUserUseCase  // interface yang sama!
}
```

### Deprecation Strategy

```go
// Middleware untuk memberi tahu client bahwa versi sudah deprecated
func V1DeprecationMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // Tambahkan header peringatan
        c.Header("Deprecation", "true")
        c.Header("Sunset", "2026-01-01")  // tanggal penghapusan
        c.Header("Link", `</api/v2/users>; rel="successor-version"`)
        c.Next()
    }
}

// Pasang ke route v1
apiV1 := r.Group("/api/v1")
apiV1.Use(V1DeprecationMiddleware())
```

### 🏋️ Latihan 3.10

1. Refactor Blog API dari Fase 3 untuk mendukung **v1** (format response flat seperti yang sudah ada) dan **v2** (tambahkan `_metadata` dengan `request_id`, `version`, `timestamp` di setiap response). Pastikan kedua versi menggunakan Use Case yang sama.
2. Buat middleware `VersionLogger` yang mencatat versi API mana yang paling banyak dipakai (simpan di in-memory counter). Tambahkan endpoint `/api/admin/version-stats` yang menampilkan statistik ini.
3. Implementasikan **header versioning** sebagai alternatif: `Accept: application/vnd.blogapi.v2+json`. Buat middleware yang membaca header ini dan redirect ke handler yang tepat.

---

## 📦 Modul 3.11 — Configuration dengan Viper

```bash
go get github.com/spf13/viper
```

```yaml
# config/config.yaml
app:
  name: "Blog API"
  env: "development"
  port: 8080

database:
  host: "localhost"
  port: 5432
  name: "blogdb"
  user: "bloguser"
  password: "blogpassword"
  sslmode: "disable"
  max_open_conns: 25
  max_idle_conns: 5

jwt:
  secret: "your-super-secret-key-change-in-production"
  access_expiry: "15m"
  refresh_expiry: "168h"

log:
  level: "debug"
  format: "json"
```

```go
// config/config.go
package config

import (
    "fmt"
    "strings"

    "github.com/spf13/viper"
)

type Config struct {
    App      AppConfig
    Database DatabaseConfig
    JWT      JWTConfig
    Log      LogConfig
}

type AppConfig struct {
    Name string
    Env  string
    Port int
}

type DatabaseConfig struct {
    Host         string
    Port         int
    Name         string
    User         string
    Password     string
    SSLMode      string
    MaxOpenConns int `mapstructure:"max_open_conns"`
    MaxIdleConns int `mapstructure:"max_idle_conns"`
}

func (d DatabaseConfig) DSN() string {
    return fmt.Sprintf(
        "host=%s port=%d user=%s password=%s dbname=%s sslmode=%s",
        d.Host, d.Port, d.User, d.Password, d.Name, d.SSLMode,
    )
}

type JWTConfig struct {
    Secret        string
    AccessExpiry  string `mapstructure:"access_expiry"`
    RefreshExpiry string `mapstructure:"refresh_expiry"`
}

type LogConfig struct {
    Level  string
    Format string
}

func Load() (*Config, error) {
    viper.SetConfigName("config")
    viper.SetConfigType("yaml")
    viper.AddConfigPath("./config")
    viper.AddConfigPath(".")

    // Override dengan environment variables
    // DATABASE_HOST → database.host
    viper.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))
    viper.AutomaticEnv()

    // Default values
    viper.SetDefault("app.port", 8080)
    viper.SetDefault("app.env", "development")

    if err := viper.ReadInConfig(); err != nil {
        return nil, fmt.Errorf("failed to read config: %w", err)
    }

    var cfg Config
    if err := viper.Unmarshal(&cfg); err != nil {
        return nil, fmt.Errorf("failed to unmarshal config: %w", err)
    }

    return &cfg, nil
}
```

---


### 🏋️ Latihan 3.11

1. Buat file `config/config.yaml` dengan Viper untuk: app (name, env, port), database (host, port, user, password, name), jwt (secret, expiry). Tambahkan `config.production.yaml` yang override beberapa nilai.
2. Buat `ConfigWatcher` yang reload config saat file `config.yaml` berubah (gunakan `viper.WatchConfig()`). Test dengan mengubah log level secara live tanpa restart service.
3. Implementasikan **config validation**: struct tag `validate` untuk memastikan required fields ada dan format benar saat startup. Service tidak boleh start dengan config yang invalid.


## 📦 Modul 3.12 — API Documentation dengan Swagger

Swagger (OpenAPI 3.0) menghasilkan dokumentasi interaktif otomatis dari komentar di kode Go.

### Setup Swaggo

```bash
# Install swag CLI
go install github.com/swaggo/swag/cmd/swag@latest

# Install dependencies
go get github.com/swaggo/gin-swagger
go get github.com/swaggo/files
go get github.com/swaggo/swag

# Verifikasi
swag --version
```

### Anotasi di main.go

```go
// main.go

// @title           Blog Platform API
// @version         1.0
// @description     REST API untuk platform blogging dengan JWT authentication.
// @termsOfService  https://example.com/terms

// @contact.name    API Support
// @contact.url     https://www.example.com/support
// @contact.email   api@example.com

// @license.name    MIT
// @license.url     https://opensource.org/licenses/MIT

// @host            localhost:8080
// @BasePath        /api/v1

// @securityDefinitions.apikey BearerAuth
// @in header
// @name Authorization
// @description Type "Bearer" followed by a space and JWT token.

package main

import (
    "github.com/gin-gonic/gin"
    swaggerFiles "github.com/swaggo/files"
    ginSwagger "github.com/swaggo/gin-swagger"

    _ "github.com/kamu/blog-api/docs" // Generated docs — JANGAN hapus!
)

func main() {
    r := gin.Default()

    // Swagger UI endpoint
    r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))

    // ... routes lainnya ...

    r.Run(":8080")
}
```

### Anotasi di Handler

```go
// internal/delivery/http/handler/post_handler.go
package handler

// GetPosts godoc
// @Summary      List semua post yang dipublish
// @Description  Mengambil daftar post dengan pagination dan filter
// @Tags         posts
// @Accept       json
// @Produce      json
// @Param        page      query  int     false  "Halaman (default: 1)"      default(1)
// @Param        per_page  query  int     false  "Item per halaman (max: 100)" default(10)
// @Param        status    query  string  false  "Filter status"              Enums(published, draft, all)
// @Param        search    query  string  false  "Cari berdasarkan judul"
// @Success      200  {object}  response.PostListResponse
// @Failure      400  {object}  response.ErrorResponse
// @Failure      500  {object}  response.ErrorResponse
// @Router       /posts [get]
func (h *PostHandler) GetPosts(c *gin.Context) {
    // implementasi...
}

// CreatePost godoc
// @Summary      Buat post baru
// @Description  Membuat post baru. User harus login.
// @Tags         posts
// @Accept       json
// @Produce      json
// @Security     BearerAuth
// @Param        request  body      request.CreatePostRequest  true  "Data post baru"
// @Success      201      {object}  response.PostResponse
// @Failure      400      {object}  response.ErrorResponse      "Validation error"
// @Failure      401      {object}  response.ErrorResponse      "Unauthorized"
// @Failure      409      {object}  response.ErrorResponse      "Slug sudah dipakai"
// @Router       /posts [post]
func (h *PostHandler) Create(c *gin.Context) {
    // implementasi...
}

// GetPost godoc
// @Summary      Get detail post
// @Description  Mengambil detail post berdasarkan slug (otomatis increment view count)
// @Tags         posts
// @Accept       json
// @Produce      json
// @Param        slug  path  string  true  "Slug post" example("getting-started-with-go")
// @Success      200   {object}  response.PostDetailResponse
// @Failure      404   {object}  response.ErrorResponse
// @Router       /posts/{slug} [get]
func (h *PostHandler) GetBySlug(c *gin.Context) {
    // implementasi...
}

// DeletePost godoc
// @Summary      Hapus post
// @Description  Menghapus post. Hanya bisa dilakukan oleh owner atau admin.
// @Tags         posts
// @Produce      json
// @Security     BearerAuth
// @Param        id   path  int  true  "ID post"
// @Success      204  "No Content"
// @Failure      401  {object}  response.ErrorResponse
// @Failure      403  {object}  response.ErrorResponse  "Bukan owner post ini"
// @Failure      404  {object}  response.ErrorResponse
// @Router       /posts/{id} [delete]
func (h *PostHandler) Delete(c *gin.Context) {
    // implementasi...
}
```

### Anotasi untuk Request/Response Structs

```go
// internal/delivery/http/request/post_request.go
package request

// CreatePostRequest adalah body request untuk membuat post baru
type CreatePostRequest struct {
    // Judul post. Minimal 5 karakter.
    // example: Getting Started with Go
    Title string `json:"title" binding:"required,min=5,max=255" example:"Getting Started with Go"`

    // Slug unik untuk URL. Hanya huruf kecil, angka, dan tanda hubung.
    // example: getting-started-with-go
    Slug string `json:"slug" binding:"required,slug" example:"getting-started-with-go"`

    // Isi konten post dalam format Markdown
    Content string `json:"content" binding:"required,min=10" example:"Go adalah bahasa pemrograman..."`

    // Ringkasan singkat post (opsional, maks 500 karakter)
    // example: Pelajari dasar-dasar Go
    Excerpt string `json:"excerpt" binding:"omitempty,max=500" example:"Pelajari dasar-dasar Go"`

    // Status awal post
    // Enum: draft, published
    // example: draft
    Status string `json:"status" binding:"omitempty,oneof=draft published" example:"draft"`

    // ID tag yang ingin ditambahkan ke post
    // example: [1, 2, 3]
    TagIDs []uint `json:"tag_ids" binding:"omitempty"`
}
```

```go
// internal/delivery/http/response/post_response.go
package response

import "time"

// PostResponse adalah response untuk satu post
type PostResponse struct {
    // ID unik post
    // example: 1
    ID uint `json:"id"`

    // Judul post
    // example: Getting Started with Go
    Title string `json:"title"`

    // Slug URL-friendly
    // example: getting-started-with-go
    Slug string `json:"slug"`

    // Status post
    // Enum: draft, published
    // example: published
    Status string `json:"status"`

    // Jumlah view
    // example: 1250
    ViewCount int `json:"view_count"`

    // Penulis post
    Author AuthorInfo `json:"author"`

    // Tanggal dibuat
    // example: 2025-01-15T10:30:00Z
    CreatedAt time.Time `json:"created_at"`
}

// AuthorInfo adalah info ringkas tentang penulis
type AuthorInfo struct {
    // example: 1
    ID uint `json:"id"`
    // example: Alice Wonderland
    Name string `json:"name"`
}

// PostListResponse adalah response untuk list post
type PostListResponse struct {
    // example: true
    Success bool           `json:"success"`
    Data    []PostResponse `json:"data"`
    Meta    PaginationMeta `json:"meta"`
}

// PaginationMeta berisi info pagination
type PaginationMeta struct {
    // example: 1
    Page int `json:"page"`
    // example: 10
    PerPage int `json:"per_page"`
    // example: 48
    Total int64 `json:"total"`
    // example: 5
    TotalPages int `json:"total_pages"`
}

// ErrorResponse adalah format error response yang konsisten
type ErrorResponse struct {
    // example: false
    Success bool      `json:"success"`
    Error   ErrorInfo `json:"error"`
}

// ErrorInfo berisi detail error
type ErrorInfo struct {
    // Kode error yang machine-readable
    // example: VALIDATION_ERROR
    Code string `json:"code"`

    // Pesan error yang human-readable
    // example: validation failed
    Message string `json:"message"`

    // Detail error per field (opsional)
    Details interface{} `json:"details,omitempty"`
}
```

### Generate Docs & Akses UI

```bash
# Generate docs dari anotasi (jalankan dari root project)
swag init -g cmd/api/main.go --output docs

# File yang dihasilkan:
# docs/
# ├── docs.go        ← embed ke binary
# ├── swagger.json   ← OpenAPI spec (JSON)
# └── swagger.yaml   ← OpenAPI spec (YAML)

# Jalankan server
go run ./cmd/api/main.go

# Akses Swagger UI di browser:
# http://localhost:8080/swagger/index.html
```

### Re-generate saat ada perubahan

```makefile
# Makefile
.PHONY: docs
docs:
	swag init -g cmd/api/main.go --output docs --parseDependency --parseInternal
	@echo "✅ Swagger docs generated"
```

### 🏋️ Latihan 3.12

1. Tambahkan anotasi Swagger **lengkap** ke semua handler Blog API (GetPosts, GetPost, CreatePost, UpdatePost, DeletePost, Login, Register). Pastikan request body, query params, response sukses, dan response error semuanya terdokumentasi.
2. Buat custom Swagger config yang menambahkan contoh response di UI. Gunakan `// example:` annotation di setiap field struct response.
3. Tambahkan endpoint `GET /api/v1/spec` yang mengembalikan OpenAPI spec dalam format JSON, sehingga client bisa auto-generate SDK.

## 📦 Modul 3.13 — Testing REST API

### Mengapa Testing Penting untuk REST API?

```
Tanpa test:
  - "Coba dulu di Postman" → manual, tidak reproducible
  - Refactor handler → takut break sesuatu
  - Bug terdeteksi di production → mahal

Dengan test:
  - Setiap endpoint terdokumentasi dengan behavior yang expected
  - Refactor dengan percaya diri
  - Bug terdeteksi saat development
```

### httptest — Testing Handler tanpa Server Nyata

```go
// internal/delivery/http/handler/product_handler_test.go
package handler_test

import (
    "bytes"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"

    "github.com/gin-gonic/gin"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
    "github.com/stretchr/testify/require"

    "github.com/kamu/blog-api/internal/delivery/http/handler"
    "github.com/kamu/blog-api/internal/mocks"
    productuc "github.com/kamu/blog-api/internal/usecase/product"
)

func init() {
    gin.SetMode(gin.TestMode) // matikan verbose output
}

// setupProductRouter helper yang buat Gin router dengan mock dependencies
func setupProductRouter(t *testing.T) (*gin.Engine, *mocks.CreateProductUseCase, *mocks.GetProductUseCase) {
    t.Helper()
    mockCreate := new(mocks.CreateProductUseCase)
    mockGet := new(mocks.GetProductUseCase)

    h := handler.NewProductHandler(mockCreate, mockGet, nil, nil)
    r := gin.New()
    r.Use(gin.Recovery())

    api := r.Group("/api/v1")
    api.POST("/products", h.Create)
    api.GET("/products/:id", h.GetByID)
    api.GET("/products", h.List)

    return r, mockCreate, mockGet
}

// doRequest helper untuk membuat HTTP request
func doRequest(t *testing.T, router *gin.Engine, method, path string, body interface{}) *httptest.ResponseRecorder {
    t.Helper()

    var bodyBytes []byte
    if body != nil {
        var err error
        bodyBytes, err = json.Marshal(body)
        require.NoError(t, err)
    }

    req := httptest.NewRequest(method, path, bytes.NewBuffer(bodyBytes))
    if body != nil {
        req.Header.Set("Content-Type", "application/json")
    }

    w := httptest.NewRecorder()
    router.ServeHTTP(w, req)
    return w
}

func TestProductHandler_Create(t *testing.T) {
    tests := []struct {
        name           string
        body           interface{}
        setupMock      func(*mocks.CreateProductUseCase)
        wantStatusCode int
        wantSuccess    bool
        wantErrorCode  string
    }{
        {
            name: "success - valid product",
            body: map[string]interface{}{
                "name":  "Laptop Gaming",
                "price": 15000000,
                "stock": 10,
                "sku":   "ELEC-00001",
            },
            setupMock: func(m *mocks.CreateProductUseCase) {
                m.On("Execute", mock.Anything, mock.MatchedBy(func(input productuc.CreateProductInput) bool {
                    return input.Name == "Laptop Gaming" && input.Price == 15000000
                })).Return(&productuc.CreateProductOutput{
                    Product: &entity.Product{ID: 1, Name: "Laptop Gaming", Price: 15000000},
                }, nil)
            },
            wantStatusCode: http.StatusCreated,
            wantSuccess:    true,
        },
        {
            name: "error - empty name",
            body: map[string]interface{}{
                "name":  "",
                "price": 15000000,
                "sku":   "ELEC-00001",
            },
            setupMock:      func(m *mocks.CreateProductUseCase) {},
            wantStatusCode: http.StatusBadRequest,
            wantSuccess:    false,
            wantErrorCode:  "VALIDATION_ERROR",
        },
        {
            name: "error - negative price",
            body: map[string]interface{}{
                "name":  "Laptop",
                "price": -1000,
                "sku":   "ELEC-00001",
            },
            setupMock:      func(m *mocks.CreateProductUseCase) {},
            wantStatusCode: http.StatusBadRequest,
            wantSuccess:    false,
        },
        {
            name: "error - duplicate SKU",
            body: map[string]interface{}{
                "name":  "Laptop",
                "price": 15000000,
                "sku":   "ELEC-00001",
            },
            setupMock: func(m *mocks.CreateProductUseCase) {
                m.On("Execute", mock.Anything, mock.Anything).
                    Return(nil, apperror.ErrSKUExists)
            },
            wantStatusCode: http.StatusConflict,
            wantSuccess:    false,
            wantErrorCode:  "SKU_EXISTS",
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            router, mockCreate, _ := setupProductRouter(t)
            tt.setupMock(mockCreate)

            w := doRequest(t, router, http.MethodPost, "/api/v1/products", tt.body)

            assert.Equal(t, tt.wantStatusCode, w.Code)

            var resp map[string]interface{}
            require.NoError(t, json.Unmarshal(w.Body.Bytes(), &resp))
            assert.Equal(t, tt.wantSuccess, resp["success"])

            if tt.wantErrorCode != "" {
                errObj, ok := resp["error"].(map[string]interface{})
                require.True(t, ok, "response should have error object")
                assert.Equal(t, tt.wantErrorCode, errObj["code"])
            }

            mockCreate.AssertExpectations(t)
        })
    }
}

func TestProductHandler_GetByID(t *testing.T) {
    t.Run("success", func(t *testing.T) {
        router, _, mockGet := setupProductRouter(t)
        mockGet.On("Execute", mock.Anything, productuc.GetProductInput{ID: 1}).
            Return(&productuc.GetProductOutput{
                Product: &entity.Product{ID: 1, Name: "Laptop Gaming"},
            }, nil)

        w := doRequest(t, router, http.MethodGet, "/api/v1/products/1", nil)

        assert.Equal(t, http.StatusOK, w.Code)
        var resp map[string]interface{}
        json.Unmarshal(w.Body.Bytes(), &resp)
        assert.True(t, resp["success"].(bool))
        data := resp["data"].(map[string]interface{})
        assert.Equal(t, float64(1), data["id"])
    })

    t.Run("not found", func(t *testing.T) {
        router, _, mockGet := setupProductRouter(t)
        mockGet.On("Execute", mock.Anything, mock.Anything).
            Return(nil, apperror.ErrProductNotFound)

        w := doRequest(t, router, http.MethodGet, "/api/v1/products/999", nil)
        assert.Equal(t, http.StatusNotFound, w.Code)
    })

    t.Run("invalid id - not a number", func(t *testing.T) {
        router, _, _ := setupProductRouter(t)
        w := doRequest(t, router, http.MethodGet, "/api/v1/products/abc", nil)
        assert.Equal(t, http.StatusBadRequest, w.Code)
    })
}
```

### Integration Test dengan Database Nyata

```go
// test/integration/product_handler_integration_test.go
//go:build integration

package integration_test

import (
    "encoding/json"
    "fmt"
    "net/http"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

// TestProductAPI_Integration menjalankan full test dengan server dan DB nyata
func TestProductAPI_Integration(t *testing.T) {
    // Setup: start test server dengan real DB (testcontainers)
    db := SetupTestDatabase(t)
    server := SetupTestServer(t, db)
    defer server.Close()

    baseURL := server.URL

    // Helper untuk auth
    token := getAuthToken(t, baseURL, "admin@test.com", "AdminPass123")

    t.Run("CRUD flow", func(t *testing.T) {
        // 1. CREATE
        createResp, err := http.Post(
            baseURL+"/api/v1/products",
            "application/json",
            strings.NewReader(`{"name":"Test Product","price":50000,"stock":100,"sku":"TEST-00001"}`),
        )
        require.NoError(t, err)
        assert.Equal(t, http.StatusCreated, createResp.StatusCode)

        var createBody map[string]interface{}
        json.NewDecoder(createResp.Body).Decode(&createBody)
        productID := int(createBody["data"].(map[string]interface{})["id"].(float64))

        // 2. GET
        getResp, _ := http.Get(fmt.Sprintf("%s/api/v1/products/%d", baseURL, productID))
        assert.Equal(t, http.StatusOK, getResp.StatusCode)

        // 3. LIST
        listResp, _ := http.Get(baseURL + "/api/v1/products?page=1&per_page=10")
        assert.Equal(t, http.StatusOK, listResp.StatusCode)

        // 4. DELETE
        req, _ := http.NewRequest(http.MethodDelete,
            fmt.Sprintf("%s/api/v1/products/%d", baseURL, productID), nil)
        req.Header.Set("Authorization", "Bearer "+token)
        deleteResp, _ := http.DefaultClient.Do(req)
        assert.Equal(t, http.StatusNoContent, deleteResp.StatusCode)

        // Verify deleted
        getAfterDelete, _ := http.Get(fmt.Sprintf("%s/api/v1/products/%d", baseURL, productID))
        assert.Equal(t, http.StatusNotFound, getAfterDelete.StatusCode)
    })
}
```

### Test dengan Authentication Header

```go
// Helper untuk test authenticated endpoints
func withAuth(t *testing.T, token string, req *http.Request) *http.Request {
    t.Helper()
    req.Header.Set("Authorization", "Bearer "+token)
    return req
}

func TestProductHandler_CreateRequiresAuth(t *testing.T) {
    router, _, _ := setupProductRouter(t)

    // Tanpa auth header → 401
    w := doRequest(t, router, http.MethodPost, "/api/v1/products", map[string]interface{}{
        "name": "Laptop", "price": 1000, "sku": "TST-00001",
    })
    assert.Equal(t, http.StatusUnauthorized, w.Code)
}
```

### 🏋️ Latihan 3.13

1. Tulis **complete HTTP handler tests** untuk Blog API: (a) `POST /posts` — success 201, validation error 400, unauthorized 401, (b) `GET /posts/:id` — success, not found 404, (c) `PUT /posts/:id` — success, not owner 403. Gunakan table-driven tests.
2. Buat **integration test** untuk `GET /posts?page=1&per_page=10&category=tech`: test pagination (total, per_page, current_page), filter by category, dan sort by created_at DESC. Gunakan testcontainers dengan real PostgreSQL.
3. Buat **API test helper package** `test/apitest` dengan helper functions: `NewRequest(method, path, body)`, `WithAuth(token)`, `AssertSuccess(t, w, statusCode)`, `AssertError(t, w, errorCode)`, `DecodeResponse(t, w, target)`. Gunakan di semua handler tests.

---


## 🎯 Review & Checkpoint Fase 3

- [ ] Bisa membuat REST API lengkap dengan Gin
- [ ] Memahami middleware chain dan cara kerjanya
- [ ] Bisa integrasi database PostgreSQL dengan GORM
- [ ] Implementasi JWT authentication
- [ ] Consistent error response format
- [ ] Input validation dengan binding tags
- [ ] Configuration management dengan Viper
- [ ] Unit test dengan httptest dan mock

---

## 🎯 Project Akhir Fase 3

Kerjakan project **REST API Blog Platform** berdasarkan PRD di file:

**`FASE-3-PRD-Blog-API.md`**

---

*Setelah selesai Fase 3, lanjut ke `FASE-4-Clean-Architecture.md`*

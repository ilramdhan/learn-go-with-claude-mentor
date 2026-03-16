# 📘 FASE 5: gRPC + Protocol Buffers

> **Prasyarat:** Selesaikan Fase 4 — User Auth Service  
> **Durasi:** 3–4 minggu  
> **Project Akhir:** Product Catalog gRPC Service  
> **Tujuan:** Memahami dan mengimplementasikan komunikasi antar service menggunakan gRPC + Protobuf, yang digunakan secara luas di perusahaan tech seperti Google, Netflix, dan Gojek

---

## 🗂️ Daftar Modul

| # | Modul | Topik |
|---|-------|-------|
| 5.1 | Mengapa gRPC? | REST vs gRPC, kapan pakai apa |
| 5.2 | Protocol Buffers | Syntax, types, field numbers |
| 5.3 | Setup Toolchain | protoc, buf, go plugins |
| 5.4 | Unary RPC | Request-Response sederhana |
| 5.5 | Server Streaming | Satu request, banyak response |
| 5.6 | Client Streaming | Banyak request, satu response |
| 5.7 | Bidirectional Streaming | Full-duplex communication |
| 5.8 | Error Handling gRPC | Status codes, error details |
| 5.9 | Metadata & Interceptors | Seperti middleware di HTTP |
| 5.10 | Authentication di gRPC | Token via metadata |
| 5.11 | Reflection & gRPC UI | Dev tools |
| 5.12 | gRPC + Clean Architecture | Integrasi dengan Fase 4 |
| 5.13 | gRPC Gateway | Expose gRPC sebagai REST API |

---

## 📦 Modul 5.1 — Mengapa gRPC?

### REST vs gRPC

| Aspek | REST | gRPC |
|-------|------|------|
| **Protocol** | HTTP/1.1 | HTTP/2 |
| **Format Data** | JSON (text, besar) | Protobuf (binary, 3-10x lebih kecil) |
| **Type Safety** | ❌ Tidak ada | ✅ Strict typing dari .proto |
| **Code Generation** | Manual/Swagger | ✅ Otomatis dari .proto |
| **Streaming** | Terbatas (SSE, WS terpisah) | ✅ 4 tipe native |
| **Performance** | Sedang | ✅ 7-10x lebih cepat |
| **Browser Support** | ✅ Native | ❌ Butuh grpc-web |
| **Human-readable** | ✅ Easy debug dengan curl | ❌ Binary, butuh tool |

### Kapan Pakai gRPC?
```
✅ Pakai gRPC untuk:
   - Komunikasi internal antar microservice
   - Service yang butuh high performance / low latency
   - Streaming data (real-time, large dataset)
   - Ketika strict typing sangat penting

✅ Pakai REST untuk:
   - API publik yang dikonsumsi browser
   - API yang perlu mudah di-debug
   - Mobile app (lebih familiar)
   - Simple CRUD dengan tim kecil
```

### Cara Kerja gRPC

```
Client                          Server
  │                               │
  │  1. Call ProductService.      │
  │     GetProduct(id: 123)       │
  │ ─────────────────────────────▶│
  │  (Protobuf binary)            │ 2. Decode, process
  │                               │
  │  3. Return Product{...}       │
  │ ◀─────────────────────────────│
  │  (Protobuf binary)            │
```

---



### Bagaimana gRPC Bekerja di Dalam (Deep Dive)

#### HTTP/2 sebagai Transport Layer

gRPC menggunakan HTTP/2 yang memberi beberapa keunggulan besar dibanding HTTP/1.1:

```
HTTP/1.1:
  Request 1 ──────────────────────▶ Response 1
                                              │
  Request 2 ──────────────────────▶ Response 2
  (harus tunggu response sebelumnya selesai — Head-of-Line Blocking)

HTTP/2 (yang dipakai gRPC):
  Request 1 ──────────▶ Response 1
  Request 2 ──────────▶ Response 2
  Request 3 ──────────▶ Response 3
  (semua berjalan BERSAMAAN dalam satu koneksi TCP — Multiplexing)
```

Keuntungan HTTP/2 untuk gRPC:
- **Multiplexing**: banyak RPC call bersamaan di satu koneksi TCP
- **Header compression (HPACK)**: metadata/header dikompress, hemat bandwidth
- **Binary framing**: data dikirim dalam frame biner, lebih efisien dari text
- **Server push**: server bisa kirim data tanpa diminta (dasar streaming)
- **Stream prioritization**: prioritas request bisa diatur

#### Lifecycle Sebuah gRPC Call

```
CLIENT                                    SERVER
  │                                          │
  │  1. Buat HTTP/2 stream                  │
  │  2. Kirim HEADERS frame                 │
  │     (method, path, content-type,        │
  │      grpc-timeout, metadata)            │
  │ ───────────────────────────────────────▶│
  │                                          │  3. Parse headers
  │                                          │  4. Route ke handler
  │  5. Kirim DATA frame                    │
  │     (length-prefixed protobuf message)  │
  │ ───────────────────────────────────────▶│
  │                                          │  6. Deserialize protobuf
  │                                          │  7. Jalankan handler logic
  │                                          │  8. Serialize response
  │  9. Terima HEADERS frame (response)     │
  │     (grpc-status, grpc-message)         │
  │ ◀───────────────────────────────────────│
  │  10. Terima DATA frame                  │
  │      (length-prefixed protobuf)         │
  │ ◀───────────────────────────────────────│
  │  11. Terima HEADERS frame (trailers)    │
  │      (grpc-status: 0 = OK)             │
  │ ◀───────────────────────────────────────│
```

#### Format Wire Protocol: Length-Prefixed Message

gRPC membungkus setiap protobuf message dengan 5-byte prefix:

```
┌──────────────────────────────────────────────────────────┐
│  Byte 0: Compression flag (0 = no compression, 1 = gzip) │
│  Byte 1-4: Message length (big-endian uint32)             │
│  Byte 5+: Protobuf-encoded message body                   │
└──────────────────────────────────────────────────────────┘

Contoh:
0x00                    → tidak dikompress
0x00 0x00 0x00 0x12     → panjang pesan: 18 bytes
0x0a 0x07 0x4c 0x61 ... → protobuf message (18 bytes)
```

#### Status Code gRPC

gRPC menggunakan status codes sendiri yang berbeda dari HTTP status codes:

```go
// google.golang.org/grpc/codes
codes.OK                  = 0   // Sukses
codes.Cancelled           = 1   // Request dibatalkan oleh client
codes.Unknown             = 2   // Error tidak diketahui
codes.InvalidArgument     = 3   // Input tidak valid (≈ HTTP 400)
codes.DeadlineExceeded    = 4   // Timeout (≈ HTTP 408/504)
codes.NotFound            = 5   // Resource tidak ada (≈ HTTP 404)
codes.AlreadyExists       = 6   // Resource sudah ada (≈ HTTP 409)
codes.PermissionDenied    = 7   // Akses ditolak (≈ HTTP 403)
codes.ResourceExhausted   = 8   // Quota habis (≈ HTTP 429)
codes.FailedPrecondition  = 9   // State tidak valid
codes.Aborted             = 10  // Operasi dibatalkan (concurrency)
codes.OutOfRange          = 11  // Nilai di luar range
codes.Unimplemented       = 12  // Method belum ada (≈ HTTP 501)
codes.Internal            = 13  // Error internal (≈ HTTP 500)
codes.Unavailable         = 14  // Service tidak tersedia (≈ HTTP 503)
codes.DataLoss            = 15  // Data hilang/korup
codes.Unauthenticated     = 16  // Belum autentikasi (≈ HTTP 401)
```



### 🏋️ Latihan 5.1 — Mengapa gRPC?

1. Buat tabel perbandingan REST vs gRPC untuk **tiga skenario nyata**: (a) API publik untuk mobile app, (b) komunikasi antar microservice internal, (c) real-time data streaming untuk dashboard. Tentukan mana yang lebih cocok untuk setiap skenario dan alasannya.
2. Ukur perbedaan ukuran payload: serialisasi struct `Product{ID: 1, Name: "Laptop", Price: 25000000, Stock: 10}` dalam format JSON vs Protobuf binary. Hitung persentase penghematannya.


## 📦 Modul 5.2 — Protocol Buffers

Protocol Buffers (protobuf) adalah **Interface Definition Language (IDL)** yang mendefinisikan struktur data dan service.

### Sintaks Dasar

```protobuf
// product.proto
syntax = "proto3";  // versi protobuf yang dipakai

// Package untuk Go — menentukan go package path
package product.v1;

option go_package = "github.com/kamu/product-service/gen/proto/product/v1;productv1";

// Import types dari protobuf standard library
import "google/protobuf/timestamp.proto";
import "google/protobuf/empty.proto";

// ===== MESSAGES (seperti struct di Go) =====

message Product {
  uint64 id          = 1;  // field number (1-15 = 1 byte, 16-2047 = 2 byte)
  string name        = 2;
  string description = 3;
  double price       = 4;
  uint32 stock       = 5;
  string category    = 6;
  bool   is_active   = 7;
  google.protobuf.Timestamp created_at = 8;
  google.protobuf.Timestamp updated_at = 9;
}

// Nested message
message Category {
  uint64 id   = 1;
  string name = 2;
  string slug = 3;
}

// Enum
enum ProductStatus {
  PRODUCT_STATUS_UNSPECIFIED = 0;  // Selalu mulai dari 0
  PRODUCT_STATUS_ACTIVE      = 1;
  PRODUCT_STATUS_INACTIVE    = 2;
  PRODUCT_STATUS_OUT_OF_STOCK = 3;
}

// Oneof — hanya satu field yang bisa di-set
message SearchFilter {
  oneof filter {
    string keyword    = 1;
    uint64 category_id = 2;
    double max_price  = 3;
  }
}

// Repeated — slice di Go
message ProductList {
  repeated Product products = 1;
  uint32 total              = 2;
  uint32 page               = 3;
  uint32 per_page           = 4;
}

// Map
message ProductMap {
  map<string, Product> products = 1;
}

// ===== SERVICE DEFINITION =====

service ProductService {
  // Unary RPC: satu request, satu response
  rpc GetProduct(GetProductRequest) returns (GetProductResponse);
  rpc CreateProduct(CreateProductRequest) returns (CreateProductResponse);
  rpc UpdateProduct(UpdateProductRequest) returns (UpdateProductResponse);
  rpc DeleteProduct(DeleteProductRequest) returns (google.protobuf.Empty);

  // Server Streaming: satu request, banyak response
  rpc ListProducts(ListProductsRequest) returns (stream Product);

  // Client Streaming: banyak request, satu response
  rpc BatchCreateProducts(stream CreateProductRequest) returns (BatchCreateResponse);

  // Bidirectional Streaming: banyak request, banyak response
  rpc StreamProductUpdates(stream ProductUpdateRequest) returns (stream Product);
}

// ===== REQUEST / RESPONSE MESSAGES =====

message GetProductRequest {
  uint64 id = 1;
}

message GetProductResponse {
  Product product = 1;
}

message CreateProductRequest {
  string name        = 1;
  string description = 2;
  double price       = 3;
  uint32 stock       = 4;
  string category    = 5;
}

message CreateProductResponse {
  Product product = 1;
}

message UpdateProductRequest {
  uint64 id          = 1;
  string name        = 2;  // optional fields — kosong berarti tidak diupdate
  string description = 3;
  double price       = 4;
  uint32 stock       = 5;
}

message UpdateProductResponse {
  Product product = 1;
}

message DeleteProductRequest {
  uint64 id = 1;
}

message ListProductsRequest {
  uint32 page      = 1;
  uint32 per_page  = 2;
  string category  = 3;
  string search    = 4;
  double min_price = 5;
  double max_price = 6;
}

message BatchCreateResponse {
  uint32 created_count = 1;
  uint32 failed_count  = 2;
  repeated string errors = 3;
}

message ProductUpdateRequest {
  uint64 id    = 1;
  double price = 2;
  uint32 stock = 3;
}
```

### Field Numbers — Aturan Penting

```protobuf
// ✅ AMAN: Tambah field baru dengan number baru
message User {
  uint64 id    = 1;
  string name  = 2;
  string email = 3;
  // Tambah field baru di sini:
  string phone = 4;  // OK — backward compatible
}

// ❌ JANGAN: Ubah field number yang sudah ada
message User {
  uint64 id    = 2;  // BAHAYA! Ubah 1 → 2 akan break semua client
  string name  = 1;  // BAHAYA!
}

// ❌ JANGAN: Hapus field, tapi boleh "reserved"
message User {
  uint64 id   = 1;
  // string name = 2;  // JANGAN dihapus begitu saja!
  reserved 2;         // Tandai sebagai reserved
  reserved "name";    // Juga tandai nama fieldnya
  string email = 3;
}
```

### Scalar Types Mapping

| Protobuf | Go | Default |
|----------|-----|---------|
| `double` | `float64` | 0 |
| `float` | `float32` | 0 |
| `int32` | `int32` | 0 |
| `int64` | `int64` | 0 |
| `uint32` | `uint32` | 0 |
| `uint64` | `uint64` | 0 |
| `bool` | `bool` | false |
| `string` | `string` | "" |
| `bytes` | `[]byte` | nil |

---



### Best Practices Protobuf Design

#### Naming Conventions

```protobuf
// ✅ BENAR: Semua naming conventions

// Package: lowercase, dot-separated
package mycompany.product.v1;

// Message: PascalCase
message ProductDetail { ... }

// Field: snake_case
string product_name = 1;
int64 created_at_unix = 2;

// Enum: SCREAMING_SNAKE_CASE
// Nilai pertama SELALU "UNSPECIFIED" = 0
enum OrderStatus {
  ORDER_STATUS_UNSPECIFIED = 0;  // wajib!
  ORDER_STATUS_PENDING     = 1;
  ORDER_STATUS_CONFIRMED   = 2;
}

// Service: PascalCase + "Service"
service ProductService { ... }

// RPC: PascalCase (verb + noun)
rpc GetProduct(GetProductRequest) returns (GetProductResponse);
rpc CreateProduct(CreateProductRequest) returns (CreateProductResponse);
rpc ListProducts(ListProductsRequest) returns (ListProductsResponse);
```

#### Standard Request/Response Pattern (Google API Design Guide)

```protobuf
// ✅ IKUTI Google API Design Guide:
// Setiap RPC punya dedicated request dan response message.
// JANGAN pakai google.protobuf.Empty sebagai request kecuali benar-benar tidak ada parameter.

// ✅ BENAR
rpc GetProduct(GetProductRequest) returns (GetProductResponse);

message GetProductRequest {
  uint64 id = 1;
}

message GetProductResponse {
  Product product = 1;
}

// ❌ SALAH: Pakai type primitif langsung
rpc GetProduct(google.protobuf.UInt64Value) returns (Product);

// ✅ BENAR: Delete response tetap ada untuk extensibility
rpc DeleteProduct(DeleteProductRequest) returns (DeleteProductResponse);

message DeleteProductRequest {
  uint64 id = 1;
}

// Response delete bisa kosong sekarang, tapi bisa ditambah field nanti
message DeleteProductResponse {
  // Sekarang kosong, tapi bisa tambah field nanti tanpa breaking change
}

// Atau pakai Empty jika memang tidak akan pernah ada return value
rpc DeleteProduct(DeleteProductRequest) returns (google.protobuf.Empty);
```

#### Field Numbering Strategy

```protobuf
message Product {
  // Field 1-15: untuk field yang PALING SERING dipakai
  // (encoded dalam 1 byte — lebih efisien)
  uint64 id          = 1;
  string name        = 2;
  double price       = 3;
  uint32 stock       = 4;
  string sku         = 5;
  ProductStatus status = 6;
  // ... field inti lainnya di 7-15

  // Field 16-2047: untuk field yang kurang sering dipakai
  // (encoded dalam 2 bytes)
  string description = 16;
  repeated string images = 17;
  google.protobuf.Timestamp created_at = 18;
  google.protobuf.Timestamp updated_at = 19;

  // Field 2048+: untuk field yang sangat jarang atau metadata
  map<string, string> metadata = 2048;
}
```

#### Versioning Strategy

```protobuf
// Gunakan package versioning untuk breaking changes
package product.v1;  // versi stabil
package product.v2;  // versi baru dengan breaking changes

// Di Go, import keduanya bisa bersamaan:
import (
    productv1 "github.com/kamu/gen/product/v1"
    productv2 "github.com/kamu/gen/product/v2"
)
```

#### Menggunakan Well-Known Types

```protobuf
import "google/protobuf/timestamp.proto";
import "google/protobuf/duration.proto";
import "google/protobuf/wrappers.proto";
import "google/protobuf/empty.proto";
import "google/protobuf/any.proto";
import "google/protobuf/struct.proto";

message Product {
  // Timestamp untuk tanggal/waktu (LEBIH BAIK dari int64 unix)
  google.protobuf.Timestamp created_at = 1;

  // Duration untuk interval waktu
  google.protobuf.Duration cache_ttl = 2;

  // Wrapper types untuk nullable primitives
  // (berbeda dari "" atau 0 — benar-benar "tidak ada nilai")
  google.protobuf.StringValue optional_note = 3;
  google.protobuf.DoubleValue discount_price = 4;

  // Struct untuk free-form JSON-like data
  google.protobuf.Struct extra_attributes = 5;
}
```

#### Anti-Patterns yang Harus Dihindari

```protobuf
// ❌ ANTI-PATTERN 1: God message — satu message untuk semua operasi
message ProductRequest {
  uint64 id = 1;
  string name = 2;         // diisi saat create
  double price = 3;        // diisi saat update
  string action = 4;       // "create", "update", "delete"???
}

// ✅ BENAR: Dedicated request per operasi
message CreateProductRequest { ... }
message UpdateProductRequest { ... }
message DeleteProductRequest { ... }

// ❌ ANTI-PATTERN 2: Mengubah semantik field tanpa mengubah number
message Product {
  string name = 1;          // v1: nama produk
  // Versi baru: developer ubah "name" jadi "full_product_name"
  // Field number 1 TETAP sama — serialization masih kompatibel
  // tapi SEMANTICS berubah → membingungkan!
}

// ✅ BENAR: Tambah field baru, reserved yang lama
message Product {
  reserved 1;
  reserved "name";
  string full_product_name = 10;  // field baru dengan number baru
}

// ❌ ANTI-PATTERN 3: Menyimpan state atau logic di proto
// Proto hanya untuk data definition, BUKAN logic!

// ❌ ANTI-PATTERN 4: Nested message yang terlalu dalam
message A {
  message B {
    message C {
      message D { ... }  // susah di-maintain
    }
  }
}
// ✅ BENAR: Flat structure, gunakan naming untuk context
message ProductCategory { ... }
message ProductCategoryItem { ... }
```



### 🏋️ Latihan 5.2 — Protocol Buffers Best Practices

1. Buat file `.proto` untuk **Notification Service** yang punya: `Notification`, `NotificationType` enum (EMAIL, SMS, PUSH), `SendNotificationRequest`, `SendNotificationResponse`. Ikuti semua naming conventions dan best practices.
2. Buat file `.proto` versi 2 dari `Product` yang **backward compatible**: tambahkan field `tags` (repeated string) dan `metadata` (map<string, string>), tanpa mengubah field number yang sudah ada.
3. Identifikasi breaking changes: (a) mengubah tipe `price` dari `double` ke `string`, (b) menghapus field `description`, (c) mengubah nama field — jelaskan mengapa masing-masing breaking.


## 📦 Modul 5.3 — Setup Toolchain

### Install Dependencies

```bash
# 1. Install protoc compiler
# macOS:
brew install protobuf

# Ubuntu/Debian:
sudo apt install -y protobuf-compiler

# Verifikasi
protoc --version  # libprotoc 25.x

# 2. Install Go plugins
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

# Tambahkan ke PATH (tambahkan ke ~/.bashrc atau ~/.zshrc)
export PATH="$PATH:$(go env GOPATH)/bin"

# 3. Install buf (modern protobuf tool — lebih mudah dari protoc langsung)
# macOS:
brew install bufbuild/buf/buf

# Linux:
curl -sSL https://github.com/bufbuild/buf/releases/download/v1.28.1/buf-Linux-x86_64 \
  -o /usr/local/bin/buf && chmod +x /usr/local/bin/buf

# Verifikasi
buf --version

# 4. Install grpcurl (seperti curl tapi untuk gRPC)
brew install grpcurl  # macOS
# atau
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
```

### Struktur Project gRPC

```
product-service/
├── api/
│   └── proto/
│       └── product/
│           └── v1/
│               └── product.proto         ← File proto kita
│
├── gen/                                  ← Generated code (jangan edit manual!)
│   └── proto/
│       └── product/
│           └── v1/
│               ├── product.pb.go         ← Generated messages
│               └── product_grpc.pb.go    ← Generated service stubs
│
├── internal/
│   ├── domain/         ← Clean Architecture tetap berlaku!
│   ├── repository/
│   ├── usecase/
│   └── delivery/
│       └── grpc/
│           └── handler/
│               └── product_handler.go
│
├── buf.yaml            ← Buf configuration
├── buf.gen.yaml        ← Code generation config
└── go.mod
```

### Konfigurasi buf

```yaml
# buf.yaml
version: v2
modules:
  - path: api/proto
deps:
  - buf.build/googleapis/googleapis  # untuk google.protobuf.* types
lint:
  use:
    - DEFAULT
breaking:
  use:
    - FILE
```

```yaml
# buf.gen.yaml
version: v2
plugins:
  - remote: buf.build/protocolbuffers/go
    out: gen
    opt:
      - paths=source_relative
  - remote: buf.build/grpc/go
    out: gen
    opt:
      - paths=source_relative
      - require_unimplemented_servers=false
```

### Generate Code

```bash
# Update dependencies
buf dep update

# Generate Go code dari proto
buf generate

# Atau dengan protoc langsung (tanpa buf):
protoc \
  --go_out=gen \
  --go_opt=paths=source_relative \
  --go-grpc_out=gen \
  --go-grpc_opt=paths=source_relative \
  api/proto/product/v1/product.proto
```

### Go Dependencies

```bash
go get google.golang.org/grpc
go get google.golang.org/protobuf
go get google.golang.org/grpc/codes
go get google.golang.org/grpc/status
go get google.golang.org/grpc/metadata
```

---


### 🏋️ Latihan 5.3 — Setup Toolchain

1. Setup project gRPC baru dengan `buf` dari awal: inisialisasi `buf.yaml`, `buf.gen.yaml`, buat satu file proto sederhana, generate code, dan pastikan hasilnya bisa di-compile dengan `go build`.
2. Tambahkan `buf lint` ke Makefile. Jalankan dan perbaiki semua lint warnings. Tambahkan `buf breaking --against .git#branch=main` untuk mendeteksi breaking changes saat PR.


## 📦 Modul 5.4 — Unary RPC

### Server Implementation

```go
// internal/delivery/grpc/handler/product_handler.go
package handler

import (
    "context"
    "errors"

    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
    "google.golang.org/protobuf/types/known/timestamppb"

    productv1 "github.com/kamu/product-service/gen/proto/product/v1"
    "github.com/kamu/product-service/internal/domain/apperror"
    "github.com/kamu/product-service/internal/domain/entity"
    productUsecase "github.com/kamu/product-service/internal/usecase/product"
)

// ProductServer mengimplementasikan productv1.ProductServiceServer
type ProductServer struct {
    // Embed UnimplementedProductServiceServer untuk forward compatibility
    // Jika ada method baru di proto, server tidak langsung error
    productv1.UnimplementedProductServiceServer

    getProductUC    productUsecase.GetProductUseCase
    createProductUC productUsecase.CreateProductUseCase
    updateProductUC productUsecase.UpdateProductUseCase
    deleteProductUC productUsecase.DeleteProductUseCase
    listProductsUC  productUsecase.ListProductsUseCase
}

func NewProductServer(
    getProductUC productUsecase.GetProductUseCase,
    createProductUC productUsecase.CreateProductUseCase,
    updateProductUC productUsecase.UpdateProductUseCase,
    deleteProductUC productUsecase.DeleteProductUseCase,
    listProductsUC productUsecase.ListProductsUseCase,
) *ProductServer {
    return &ProductServer{
        getProductUC:    getProductUC,
        createProductUC: createProductUC,
        updateProductUC: updateProductUC,
        deleteProductUC: deleteProductUC,
        listProductsUC:  listProductsUC,
    }
}

// GetProduct mengimplementasikan unary RPC
func (s *ProductServer) GetProduct(
    ctx context.Context,
    req *productv1.GetProductRequest,
) (*productv1.GetProductResponse, error) {
    // Validasi input
    if req.Id == 0 {
        return nil, status.Error(codes.InvalidArgument, "product id is required")
    }

    // Panggil use case
    output, err := s.getProductUC.Execute(ctx, productUsecase.GetProductInput{
        ID: req.Id,
    })
    if err != nil {
        return nil, toGRPCError(err)
    }

    // Convert entity ke proto message
    return &productv1.GetProductResponse{
        Product: toProtoProduct(output.Product),
    }, nil
}

func (s *ProductServer) CreateProduct(
    ctx context.Context,
    req *productv1.CreateProductRequest,
) (*productv1.CreateProductResponse, error) {
    // Validasi
    if req.Name == "" {
        return nil, status.Error(codes.InvalidArgument, "name is required")
    }
    if req.Price <= 0 {
        return nil, status.Error(codes.InvalidArgument, "price must be positive")
    }

    output, err := s.createProductUC.Execute(ctx, productUsecase.CreateProductInput{
        Name:        req.Name,
        Description: req.Description,
        Price:       req.Price,
        Stock:       req.Stock,
        Category:    req.Category,
    })
    if err != nil {
        return nil, toGRPCError(err)
    }

    return &productv1.CreateProductResponse{
        Product: toProtoProduct(output.Product),
    }, nil
}

func (s *ProductServer) DeleteProduct(
    ctx context.Context,
    req *productv1.DeleteProductRequest,
) (*emptypb.Empty, error) {
    if req.Id == 0 {
        return nil, status.Error(codes.InvalidArgument, "product id is required")
    }

    if err := s.deleteProductUC.Execute(ctx, productUsecase.DeleteProductInput{ID: req.Id}); err != nil {
        return nil, toGRPCError(err)
    }

    return &emptypb.Empty{}, nil
}

// ===== Conversion Helpers =====

// toProtoProduct mengkonversi domain entity ke proto message
func toProtoProduct(p *entity.Product) *productv1.Product {
    return &productv1.Product{
        Id:          p.ID,
        Name:        p.Name,
        Description: p.Description,
        Price:       p.Price,
        Stock:       uint32(p.Stock),
        Category:    p.Category,
        IsActive:    p.IsActive,
        CreatedAt:   timestamppb.New(p.CreatedAt),
        UpdatedAt:   timestamppb.New(p.UpdatedAt),
    }
}

// toGRPCError mengkonversi domain error ke gRPC status error
func toGRPCError(err error) error {
    var appErr *apperror.AppError
    if errors.As(err, &appErr) {
        switch appErr.HTTPStatus {
        case 400:
            return status.Error(codes.InvalidArgument, appErr.Message)
        case 401:
            return status.Error(codes.Unauthenticated, appErr.Message)
        case 403:
            return status.Error(codes.PermissionDenied, appErr.Message)
        case 404:
            return status.Error(codes.NotFound, appErr.Message)
        case 409:
            return status.Error(codes.AlreadyExists, appErr.Message)
        default:
            return status.Error(codes.Internal, appErr.Message)
        }
    }
    return status.Error(codes.Internal, "internal error")
}
```

### Server Main

```go
// cmd/grpc/main.go
package main

import (
    "fmt"
    "log"
    "net"
    "os"
    "os/signal"
    "syscall"

    "google.golang.org/grpc"
    "google.golang.org/grpc/reflection"

    productv1 "github.com/kamu/product-service/gen/proto/product/v1"
    grpcHandler "github.com/kamu/product-service/internal/delivery/grpc/handler"
    "github.com/kamu/product-service/internal/delivery/grpc/interceptor"
)

func main() {
    cfg, _ := config.Load()

    // Setup dependencies (sama seperti Fase 4)
    // ... db, repos, usecases ...

    // Buat gRPC server dengan interceptors
    srv := grpc.NewServer(
        grpc.ChainUnaryInterceptor(
            interceptor.RecoveryInterceptor(),
            interceptor.LoggingInterceptor(),
            interceptor.AuthInterceptor(jwtService),
        ),
        grpc.ChainStreamInterceptor(
            interceptor.StreamRecoveryInterceptor(),
            interceptor.StreamLoggingInterceptor(),
        ),
    )

    // Daftarkan service
    productServer := grpcHandler.NewProductServer(
        getProductUC, createProductUC, updateProductUC,
        deleteProductUC, listProductsUC,
    )
    productv1.RegisterProductServiceServer(srv, productServer)

    // Aktifkan reflection untuk grpcurl dan GUI tools
    reflection.Register(srv)

    // Start listener
    lis, err := net.Listen("tcp", fmt.Sprintf(":%d", cfg.App.GRPCPort))
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }

    // Start server di goroutine
    go func() {
        log.Printf("gRPC server listening on port %d", cfg.App.GRPCPort)
        if err := srv.Serve(lis); err != nil {
            log.Fatalf("failed to serve: %v", err)
        }
    }()

    // Graceful shutdown
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    log.Println("Shutting down gRPC server...")
    srv.GracefulStop()
    log.Println("gRPC server stopped")
}
```

### Client

```go
// Cara membuat gRPC client

package main

import (
    "context"
    "log"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
    "google.golang.org/grpc/metadata"

    productv1 "github.com/kamu/product-service/gen/proto/product/v1"
)

func main() {
    // Buat koneksi ke server
    conn, err := grpc.Dial(
        "localhost:50051",
        grpc.WithTransportCredentials(insecure.NewCredentials()), // tanpa TLS (dev)
        grpc.WithBlock(),
        grpc.WithTimeout(5*time.Second),
    )
    if err != nil {
        log.Fatalf("failed to connect: %v", err)
    }
    defer conn.Close()

    // Buat client stub
    client := productv1.NewProductServiceClient(conn)

    // Tambahkan metadata (seperti header di HTTP)
    ctx := metadata.NewOutgoingContext(context.Background(), metadata.Pairs(
        "authorization", "Bearer eyJ...",
        "x-request-id",  "req-123",
    ))

    // Panggil RPC
    resp, err := client.GetProduct(ctx, &productv1.GetProductRequest{Id: 1})
    if err != nil {
        log.Printf("Error: %v", err)
        return
    }
    log.Printf("Product: %+v", resp.Product)

    // Create product
    createResp, err := client.CreateProduct(ctx, &productv1.CreateProductRequest{
        Name:        "Laptop Gaming",
        Description: "High performance laptop",
        Price:       15000000,
        Stock:       50,
        Category:    "electronics",
    })
    if err != nil {
        log.Printf("Error: %v", err)
        return
    }
    log.Printf("Created: %+v", createResp.Product)
}
```

---


### 🏋️ Latihan 5.4 — Unary RPC

1. Implementasikan `GetProductBySKU` unary RPC. Validasi: SKU tidak boleh kosong, format harus `^[A-Z]+-\d{5}$`. Return `InvalidArgument` jika tidak valid, `NotFound` jika SKU tidak ada.
2. Buat client code yang memanggil Create → Get → Update → Delete secara sequential dengan proper error handling. Setiap error harus ditangani berbeda sesuai gRPC status code-nya.


## 📦 Modul 5.5 — Server Streaming RPC

Server streaming berguna untuk: mengirim data besar secara bertahap, real-time updates, atau progress notifications.

```go
// Server implementation — ListProducts dengan streaming
func (s *ProductServer) ListProducts(
    req *productv1.ListProductsRequest,
    stream productv1.ProductService_ListProductsServer,
) error {
    // Ambil context dari stream untuk cancellation
    ctx := stream.Context()

    // Ambil semua produk berdasarkan filter
    products, err := s.listProductsUC.Execute(ctx, productUsecase.ListProductsInput{
        Category: req.Category,
        Search:   req.Search,
        MinPrice: req.MinPrice,
        MaxPrice: req.MaxPrice,
    })
    if err != nil {
        return toGRPCError(err)
    }

    // Stream setiap produk satu per satu
    for _, product := range products {
        // Cek apakah client sudah disconnect
        select {
        case <-ctx.Done():
            return status.Error(codes.Cancelled, "client cancelled the request")
        default:
        }

        // Kirim produk ke client
        if err := stream.Send(toProtoProduct(product)); err != nil {
            return status.Errorf(codes.Internal, "failed to send product: %v", err)
        }
    }

    return nil // stream selesai
}
```

```go
// Client yang mengkonsumsi stream
func consumeProductStream(client productv1.ProductServiceClient) {
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    // Mulai streaming
    stream, err := client.ListProducts(ctx, &productv1.ListProductsRequest{
        Category: "electronics",
    })
    if err != nil {
        log.Printf("Error creating stream: %v", err)
        return
    }

    // Terima produk satu per satu
    var count int
    for {
        product, err := stream.Recv()
        if err == io.EOF {
            // Server selesai mengirim
            break
        }
        if err != nil {
            log.Printf("Error receiving: %v", err)
            break
        }
        count++
        log.Printf("Received product #%d: %s (%.2f)", count, product.Name, product.Price)
    }

    log.Printf("Total received: %d products", count)
}
```

---


### 🏋️ Latihan 5.5 — Server Streaming

1. Buat RPC `ExportProducts(ExportRequest) returns (stream ProductExportRow)` yang mengexport produk sebagai CSV rows via stream. Implementasikan server dan client-nya.
2. Tambahkan **rate limiting** di sisi server: kirim maksimal 100 rows per detik. Gunakan `time.Ticker` untuk mengontrol kecepatan pengiriman.


## 📦 Modul 5.6 — Client Streaming RPC

Client streaming berguna untuk: upload batch data, aggregasi data dari client.

```go
// Server — terima banyak request, balas satu response
func (s *ProductServer) BatchCreateProducts(
    stream productv1.ProductService_BatchCreateProductsServer,
) error {
    ctx := stream.Context()
    var createdCount, failedCount int
    var errs []string

    // Terima produk satu per satu dari client
    for {
        req, err := stream.Recv()
        if err == io.EOF {
            // Client selesai mengirim — kirim response
            return stream.SendAndClose(&productv1.BatchCreateResponse{
                CreatedCount: uint32(createdCount),
                FailedCount:  uint32(failedCount),
                Errors:       errs,
            })
        }
        if err != nil {
            return status.Errorf(codes.Internal, "failed to receive: %v", err)
        }

        // Proses setiap request
        _, err = s.createProductUC.Execute(ctx, productUsecase.CreateProductInput{
            Name:     req.Name,
            Price:    req.Price,
            Stock:    req.Stock,
            Category: req.Category,
        })
        if err != nil {
            failedCount++
            errs = append(errs, fmt.Sprintf("failed to create %s: %v", req.Name, err))
        } else {
            createdCount++
        }
    }
}
```

```go
// Client — kirim banyak request
func batchCreateProducts(client productv1.ProductServiceClient) {
    ctx := context.Background()
    stream, err := client.BatchCreateProducts(ctx)
    if err != nil {
        log.Fatal(err)
    }

    // Data produk yang mau dibuat
    products := []struct{ Name, Category string; Price float64; Stock uint32 }{
        {"Laptop", "electronics", 10000000, 20},
        {"Mouse", "electronics", 150000, 100},
        {"Keyboard", "electronics", 300000, 50},
        {"Monitor", "electronics", 2500000, 15},
    }

    // Kirim satu per satu
    for _, p := range products {
        if err := stream.Send(&productv1.CreateProductRequest{
            Name:     p.Name,
            Category: p.Category,
            Price:    p.Price,
            Stock:    p.Stock,
        }); err != nil {
            log.Printf("Failed to send %s: %v", p.Name, err)
        }
        time.Sleep(100 * time.Millisecond) // simulasi delay
    }

    // Tutup stream dan terima response
    resp, err := stream.CloseAndRecv()
    if err != nil {
        log.Fatal(err)
    }

    log.Printf("Batch create: %d created, %d failed", resp.CreatedCount, resp.FailedCount)
    for _, e := range resp.Errors {
        log.Printf("Error: %s", e)
    }
}
```

---


### 🏋️ Latihan 5.6 — Client Streaming

1. Implementasikan client untuk `BatchCreateProducts` yang membaca data dari **CSV file** baris per baris dan mengirimnya sebagai stream. Handle error per baris (skip yang gagal, lanjutkan yang lain, catat errors).
2. Tambahkan progress indicator di client: setiap 100 item yang dikirim, print "Sent X items..."


## 📦 Modul 5.7 — Bidirectional Streaming

Bidi streaming berguna untuk: real-time chat, live price feed, collaborative editing.

```go
// Server — real-time product update feed
func (s *ProductServer) StreamProductUpdates(
    stream productv1.ProductService_StreamProductUpdatesServer,
) error {
    ctx := stream.Context()

    for {
        // Terima update request dari client
        req, err := stream.Recv()
        if err == io.EOF {
            return nil
        }
        if err != nil {
            return status.Errorf(codes.Internal, "receive error: %v", err)
        }

        // Update produk
        output, err := s.updateProductUC.Execute(ctx, productUsecase.UpdateProductInput{
            ID:    req.Id,
            Price: req.Price,
            Stock: req.Stock,
        })
        if err != nil {
            // Kirim error tapi jangan stop stream
            if sendErr := stream.Send(&productv1.Product{
                Id: req.Id,
                // Bisa kirim dengan field error khusus
            }); sendErr != nil {
                return sendErr
            }
            continue
        }

        // Kirim updated product balik ke client
        if err := stream.Send(toProtoProduct(output.Product)); err != nil {
            return status.Errorf(codes.Internal, "send error: %v", err)
        }
    }
}
```

---


### 🏋️ Latihan 5.7 — Bidirectional Streaming

1. Buat bidirectional streaming RPC `Chat(stream ChatMessage) returns (stream ChatMessage)` — echo server. Implementasikan server dan client dengan goroutine untuk send dan receive bersamaan.
2. Modifikasi `StreamStockUpdates` sehingga server menolak update jika stok menjadi negatif dan mengirim error response (bukan disconnect stream).


## 📦 Modul 5.8 — Error Handling gRPC

### gRPC Status Codes

```go
// Mapping gRPC codes ke situasi yang tepat
var grpcCodeGuide = map[codes.Code]string{
    codes.OK:                 "Sukses",
    codes.Cancelled:          "Request dibatalkan oleh client",
    codes.Unknown:            "Error tidak diketahui",
    codes.InvalidArgument:    "Input tidak valid (seperti 400)",
    codes.DeadlineExceeded:   "Timeout (seperti 408)",
    codes.NotFound:           "Resource tidak ditemukan (seperti 404)",
    codes.AlreadyExists:      "Resource sudah ada (seperti 409)",
    codes.PermissionDenied:   "Akses ditolak (seperti 403)",
    codes.ResourceExhausted:  "Rate limit atau quota habis (seperti 429)",
    codes.FailedPrecondition: "State tidak memenuhi syarat",
    codes.Aborted:            "Operasi dibatalkan (concurrency conflict)",
    codes.OutOfRange:         "Nilai di luar range yang valid",
    codes.Unimplemented:      "Method belum diimplementasikan (seperti 501)",
    codes.Internal:           "Error internal server (seperti 500)",
    codes.Unavailable:        "Server tidak tersedia (seperti 503)",
    codes.DataLoss:           "Data hilang atau korup",
    codes.Unauthenticated:    "Tidak terautentikasi (seperti 401)",
}
```

### Error dengan Detail (Rich Errors)

```go
// Kirim error dengan detail tambahan
import (
    "google.golang.org/grpc/status"
    "google.golang.org/genproto/googleapis/rpc/errdetails"
)

func sendDetailedError() error {
    st := status.New(codes.InvalidArgument, "validation failed")
    
    // Tambah BadRequest detail
    br := &errdetails.BadRequest{}
    br.FieldViolations = append(br.FieldViolations,
        &errdetails.BadRequest_FieldViolation{
            Field:       "price",
            Description: "price must be positive",
        },
        &errdetails.BadRequest_FieldViolation{
            Field:       "name",
            Description: "name cannot be empty",
        },
    )
    
    st, _ = st.WithDetails(br)
    return st.Err()
}

// Client yang membaca error detail
func handleGRPCError(err error) {
    st, ok := status.FromError(err)
    if !ok {
        log.Printf("Non-gRPC error: %v", err)
        return
    }

    log.Printf("gRPC error: code=%s, message=%s", st.Code(), st.Message())

    for _, detail := range st.Details() {
        switch v := detail.(type) {
        case *errdetails.BadRequest:
            for _, violation := range v.FieldViolations {
                log.Printf("  Field '%s': %s", violation.Field, violation.Description)
            }
        case *errdetails.RetryInfo:
            log.Printf("  Retry after: %v", v.RetryDelay.AsDuration())
        }
    }
}
```

---


### 🏋️ Latihan 5.8 — Error Handling

1. Implementasikan rich error: saat `CreateProduct` gagal validasi, kembalikan `InvalidArgument` dengan `errdetails.BadRequest` yang berisi daftar field violations (nama field + pesan error spesifik).
2. Buat client yang mem-parse `errdetails.BadRequest` dan menampilkan field violations dalam format user-friendly seperti: `\nValidation errors:\n  - name: must be at least 3 characters\n  - price: must be positive`


## 📦 Modul 5.9 — Interceptors (Middleware gRPC)

Interceptors adalah middleware di gRPC. Ada dua jenis:
- **Unary Interceptor** — untuk unary RPC
- **Stream Interceptor** — untuk streaming RPC

```go
// internal/delivery/grpc/interceptor/logging.go
package interceptor

import (
    "context"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/status"
)

// LoggingInterceptor mencatat setiap RPC call
func LoggingInterceptor() grpc.UnaryServerInterceptor {
    return func(
        ctx context.Context,
        req interface{},
        info *grpc.UnaryServerInfo,
        handler grpc.UnaryHandler,
    ) (interface{}, error) {
        start := time.Now()

        // Jalankan handler
        resp, err := handler(ctx, req)

        // Log setelah handler selesai
        duration := time.Since(start)
        code := codes.OK
        if err != nil {
            code = status.Code(err)
        }

        log.Printf(
            "gRPC | method=%s | code=%s | duration=%v | error=%v",
            info.FullMethod, code, duration, err,
        )

        return resp, err
    }
}

// RecoveryInterceptor mencegah panic dari crash server
func RecoveryInterceptor() grpc.UnaryServerInterceptor {
    return func(
        ctx context.Context,
        req interface{},
        info *grpc.UnaryServerInfo,
        handler grpc.UnaryHandler,
    ) (resp interface{}, err error) {
        defer func() {
            if r := recover(); r != nil {
                log.Printf("PANIC in %s: %v", info.FullMethod, r)
                err = status.Errorf(codes.Internal, "internal server error")
            }
        }()
        return handler(ctx, req)
    }
}

// AuthInterceptor memvalidasi JWT dari metadata
func AuthInterceptor(jwtService jwt.JWTService) grpc.UnaryServerInterceptor {
    // Methods yang tidak perlu auth
    publicMethods := map[string]bool{
        "/product.v1.ProductService/GetProduct":    true,
        "/product.v1.ProductService/ListProducts":  true,
    }

    return func(
        ctx context.Context,
        req interface{},
        info *grpc.UnaryServerInfo,
        handler grpc.UnaryHandler,
    ) (interface{}, error) {
        // Skip auth untuk public methods
        if publicMethods[info.FullMethod] {
            return handler(ctx, req)
        }

        // Ambil token dari metadata
        md, ok := metadata.FromIncomingContext(ctx)
        if !ok {
            return nil, status.Error(codes.Unauthenticated, "missing metadata")
        }

        authHeader := md.Get("authorization")
        if len(authHeader) == 0 {
            return nil, status.Error(codes.Unauthenticated, "missing authorization")
        }

        // Format: "Bearer <token>"
        tokenStr := strings.TrimPrefix(authHeader[0], "Bearer ")
        claims, err := jwtService.ValidateToken(tokenStr)
        if err != nil {
            return nil, status.Error(codes.Unauthenticated, "invalid token")
        }

        // Set user info ke context
        ctx = context.WithValue(ctx, "user_id", claims.UserID)
        ctx = context.WithValue(ctx, "user_role", claims.Role)

        return handler(ctx, req)
    }
}

// RateLimitInterceptor membatasi jumlah request per client
func RateLimitInterceptor(limiter *rate.Limiter) grpc.UnaryServerInterceptor {
    return func(
        ctx context.Context,
        req interface{},
        info *grpc.UnaryServerInfo,
        handler grpc.UnaryHandler,
    ) (interface{}, error) {
        if !limiter.Allow() {
            return nil, status.Error(codes.ResourceExhausted, "rate limit exceeded")
        }
        return handler(ctx, req)
    }
}
```

---



### 🏋️ Latihan 5.9 — Interceptors

1. Buat `RetryInterceptor` di sisi **client** yang otomatis retry request yang gagal dengan kode `Unavailable` atau `ResourceExhausted`. Maksimal 3 kali retry dengan exponential backoff (1s, 2s, 4s delay).
2. Buat `TimeoutInterceptor` yang otomatis menambahkan context timeout ke semua request jika client tidak menyediakan deadline. Default timeout: 30 detik.


## 📦 Modul 5.10 — Authentication & Authorization di gRPC

Di gRPC, auth dilakukan melalui **metadata** (setara header di HTTP). Ada dua pendekatan utama.

### Approach 1: JWT via Metadata (Paling Umum)

```go
// pkg/jwt/jwt.go — sama seperti Fase 4, reusable
package jwt

type Claims struct {
    UserID uint   `json:"user_id"`
    Email  string `json:"email"`
    Role   string `json:"role"`
    jwt.RegisteredClaims
}

type JWTService interface {
    GenerateAccessToken(userID uint, email, role string) (string, error)
    ValidateToken(tokenStr string) (*Claims, error)
}
```

```go
// Client: kirim token di metadata
package main

import (
    "context"
    "google.golang.org/grpc/metadata"
)

// Cara 1: Per-call credentials (token berbeda per call)
func callWithToken(ctx context.Context, token string) context.Context {
    return metadata.AppendToOutgoingContext(ctx,
        "authorization", "Bearer "+token,
    )
}

// Cara 2: Per-connection credentials (token sama untuk semua call di koneksi ini)
type tokenCredential struct {
    token string
}

func (t *tokenCredential) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {
    return map[string]string{
        "authorization": "Bearer " + t.token,
    }, nil
}

func (t *tokenCredential) RequireTransportSecurity() bool {
    return false // true untuk production (pakai TLS)
}

// Gunakan saat membuat koneksi
conn, err := grpc.Dial("localhost:50051",
    grpc.WithTransportCredentials(insecure.NewCredentials()),
    grpc.WithPerRPCCredentials(&tokenCredential{token: "eyJ..."}),
)
```

```go
// Server: interceptor auth yang lengkap
// internal/delivery/grpc/interceptor/auth.go
package interceptor

import (
    "context"
    "strings"

    "google.golang.org/grpc"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/metadata"
    "google.golang.org/grpc/status"

    "github.com/kamu/product-service/pkg/jwt"
)

type contextKey string

const (
    ContextKeyUserID contextKey = "user_id"
    ContextKeyRole   contextKey = "user_role"
    ContextKeyEmail  contextKey = "user_email"
)

// publicMethods adalah methods yang tidak butuh autentikasi
var publicMethods = map[string]bool{
    "/product.v1.ProductService/GetProduct":           true,
    "/product.v1.ProductService/ListProducts":         true,
    "/product.v1.ProductService/StreamProductsByCategory": true,
    "/product.v1.CategoryService/ListCategories":      true,
}

// AuthInterceptor adalah unary server interceptor untuk autentikasi
func AuthInterceptor(jwtSvc jwt.JWTService) grpc.UnaryServerInterceptor {
    return func(
        ctx context.Context,
        req interface{},
        info *grpc.UnaryServerInfo,
        handler grpc.UnaryHandler,
    ) (interface{}, error) {
        // Skip auth untuk public methods
        if publicMethods[info.FullMethod] {
            return handler(ctx, req)
        }

        // Ekstrak dan validasi token
        userCtx, err := authenticateRequest(ctx, jwtSvc)
        if err != nil {
            return nil, err
        }

        return handler(userCtx, req)
    }
}

// StreamAuthInterceptor untuk streaming RPC
func StreamAuthInterceptor(jwtSvc jwt.JWTService) grpc.StreamServerInterceptor {
    return func(
        srv interface{},
        ss grpc.ServerStream,
        info *grpc.StreamServerInfo,
        handler grpc.StreamHandler,
    ) error {
        if publicMethods[info.FullMethod] {
            return handler(srv, ss)
        }

        userCtx, err := authenticateRequest(ss.Context(), jwtSvc)
        if err != nil {
            return err
        }

        // Wrap stream dengan context baru yang berisi user info
        wrapped := &wrappedStream{ss, userCtx}
        return handler(srv, wrapped)
    }
}

// wrappedStream mengganti context di server stream
type wrappedStream struct {
    grpc.ServerStream
    ctx context.Context
}

func (w *wrappedStream) Context() context.Context {
    return w.ctx
}

func authenticateRequest(ctx context.Context, jwtSvc jwt.JWTService) (context.Context, error) {
    md, ok := metadata.FromIncomingContext(ctx)
    if !ok {
        return nil, status.Error(codes.Unauthenticated, "missing metadata")
    }

    // Ambil authorization header
    authValues := md.Get("authorization")
    if len(authValues) == 0 {
        return nil, status.Error(codes.Unauthenticated, "authorization header is required")
    }

    authHeader := authValues[0]
    if !strings.HasPrefix(authHeader, "Bearer ") {
        return nil, status.Error(codes.Unauthenticated, "authorization format must be: Bearer {token}")
    }

    tokenStr := strings.TrimPrefix(authHeader, "Bearer ")

    // Validasi JWT
    claims, err := jwtSvc.ValidateToken(tokenStr)
    if err != nil {
        return nil, status.Errorf(codes.Unauthenticated, "invalid token: %v", err)
    }

    // Tambahkan user info ke context
    ctx = context.WithValue(ctx, ContextKeyUserID, claims.UserID)
    ctx = context.WithValue(ctx, ContextKeyRole, claims.Role)
    ctx = context.WithValue(ctx, ContextKeyEmail, claims.Email)

    return ctx, nil
}

// Helper functions untuk mengambil user info dari context di handler
func GetUserID(ctx context.Context) (uint, bool) {
    id, ok := ctx.Value(ContextKeyUserID).(uint)
    return id, ok
}

func GetUserRole(ctx context.Context) string {
    role, _ := ctx.Value(ContextKeyRole).(string)
    return role
}

// RequireRole adalah helper untuk cek role di dalam handler
func RequireRole(ctx context.Context, requiredRole string) error {
    role := GetUserRole(ctx)
    if role != requiredRole {
        return status.Errorf(codes.PermissionDenied,
            "required role: %s, got: %s", requiredRole, role)
    }
    return nil
}
```

```go
// Penggunaan di handler
func (s *ProductServer) CreateProduct(
    ctx context.Context,
    req *productv1.CreateProductRequest,
) (*productv1.CreateProductResponse, error) {
    // Cek role admin
    if err := interceptor.RequireRole(ctx, "admin"); err != nil {
        return nil, err
    }

    userID, ok := interceptor.GetUserID(ctx)
    if !ok {
        return nil, status.Error(codes.Internal, "user id not found in context")
    }

    // Lanjutkan proses...
    _ = userID
    return nil, nil
}
```

### Approach 2: mTLS (Mutual TLS) untuk Service-to-Service

Untuk komunikasi antar microservice internal, mTLS lebih aman dari JWT karena tidak perlu manage token expiry.

```go
// Server dengan TLS
func serverWithTLS() {
    // Load server certificate dan key
    cert, err := tls.LoadX509KeyPair("server.crt", "server.key")
    if err != nil {
        log.Fatal(err)
    }

    // Load CA certificate untuk verifikasi client
    caCert, err := os.ReadFile("ca.crt")
    if err != nil {
        log.Fatal(err)
    }
    caCertPool := x509.NewCertPool()
    caCertPool.AppendCertsFromPEM(caCert)

    creds := credentials.NewTLS(&tls.Config{
        Certificates: []tls.Certificate{cert},
        ClientCAs:    caCertPool,
        ClientAuth:   tls.RequireAndVerifyClientCert, // wajib punya cert
    })

    srv := grpc.NewServer(grpc.Creds(creds))
    // ...
}

// Client dengan mTLS
func clientWithMTLS() *grpc.ClientConn {
    cert, _ := tls.LoadX509KeyPair("client.crt", "client.key")
    caCert, _ := os.ReadFile("ca.crt")
    caCertPool := x509.NewCertPool()
    caCertPool.AppendCertsFromPEM(caCert)

    creds := credentials.NewTLS(&tls.Config{
        Certificates: []tls.Certificate{cert},
        RootCAs:      caCertPool,
        ServerName:   "product-service", // harus match dengan CN di server cert
    })

    conn, _ := grpc.Dial("localhost:50051", grpc.WithTransportCredentials(creds))
    return conn
}
```

### 🏋️ Latihan 5.10

1. Implementasikan `AuthInterceptor` lengkap yang meng-extract JWT dari metadata dan menyimpan user info ke context. Buat unit test interceptor dengan mock JWT service — test: valid token, expired token, missing token, malformed token.
2. Buat `RoleMiddleware(requiredRole string) grpc.UnaryServerInterceptor` yang mengecek role user dari context. Test bahwa admin bisa akses, tapi user biasa dapat `PermissionDenied`.
3. Buat `ServiceTokenInterceptor` untuk service-to-service auth: setiap service punya static API key yang dikirim via metadata `x-service-key`. Server memvalidasi key dari config.

## 📦 Modul 5.11 — Reflection, Tools & Debugging gRPC

### gRPC Server Reflection

Reflection memungkinkan tools seperti grpcurl dan Postman untuk **auto-discover** service tanpa file .proto.

```go
// Aktifkan reflection di server
import "google.golang.org/grpc/reflection"

func main() {
    srv := grpc.NewServer(/* interceptors */)

    productv1.RegisterProductServiceServer(srv, productServer)
    categoryv1.RegisterCategoryServiceServer(srv, categoryServer)

    // Aktifkan reflection — HANYA untuk development/staging!
    // Di production, nonaktifkan untuk keamanan
    if cfg.App.Env != "production" {
        reflection.Register(srv)
    }

    // ...
}
```

### grpcurl — CLI Tool untuk gRPC

```bash
# ===== Discovery =====

# List semua service yang tersedia
grpcurl -plaintext localhost:50051 list
# Output:
# grpc.reflection.v1alpha.ServerReflection
# product.v1.CategoryService
# product.v1.ProductService

# List semua method di service
grpcurl -plaintext localhost:50051 list product.v1.ProductService
# Output:
# product.v1.ProductService.BatchCreateProducts
# product.v1.ProductService.CheckStock
# product.v1.ProductService.CreateProduct
# ...

# Describe message atau service
grpcurl -plaintext localhost:50051 describe product.v1.Product
grpcurl -plaintext localhost:50051 describe product.v1.CreateProductRequest

# ===== Unary Calls =====

# Call sederhana
grpcurl -plaintext \
  -d '{"id": 1}' \
  localhost:50051 product.v1.ProductService/GetProduct

# Call dengan metadata (auth)
grpcurl -plaintext \
  -H 'authorization: Bearer eyJhbGciOiJIUzI1NiJ9...' \
  -H 'x-request-id: req-test-001' \
  -d '{
    "name": "Laptop Gaming",
    "description": "RTX 4070 Ti, 32GB RAM",
    "price": 25000000,
    "stock": 10,
    "category": "electronics",
    "sku": "ELEC-LPT-001"
  }' \
  localhost:50051 product.v1.ProductService/CreateProduct

# ===== Streaming Calls =====

# Server streaming — hasil diprint satu per satu saat diterima
grpcurl -plaintext \
  -d '{"category": "electronics", "per_page": 100}' \
  localhost:50051 product.v1.ProductService/StreamProductsByCategory

# Client streaming — kirim multiple JSON objects, pisahkan dengan newline
echo '{"name":"P1","price":100,"stock":5}
{"name":"P2","price":200,"stock":10}
{"name":"P3","price":300,"stock":15}' | \
grpcurl -plaintext \
  -d @ \
  localhost:50051 product.v1.ProductService/BatchCreateProducts

# ===== Format Output =====
# Default: JSON pretty print
# Tambahkan -format json untuk compact
grpcurl -plaintext -format json -d '{"id":1}' \
  localhost:50051 product.v1.ProductService/GetProduct

# ===== Verbose Mode untuk Debugging =====
grpcurl -plaintext -v \
  -d '{"id": 1}' \
  localhost:50051 product.v1.ProductService/GetProduct
# Output mencakup: request metadata, response headers, body, trailers, timing
```

### Evans — Interactive gRPC Client

Evans adalah REPL (Read-Eval-Print Loop) untuk gRPC, sangat berguna saat development.

```bash
# Install
go install github.com/ktr0731/evans@latest

# Mode REPL
evans --host localhost --port 50051 repl

# Di dalam REPL:
# > show package           → list packages
# > package product.v1     → pilih package
# > show service           → list services
# > service ProductService → pilih service
# > show rpc               → list RPCs
# > call GetProduct        → panggil RPC (interactive!)
#   id (TYPE_UINT64) => 1  → input field
# > header authorization "Bearer eyJ..."  → set header
# > exit

# Mode CLI (non-interactive)
evans --host localhost --port 50051 cli call \
  --service product.v1.ProductService \
  --rpc GetProduct \
  --file request.json  # baca request dari file
```

### Postman untuk gRPC

Postman (versi terbaru) mendukung gRPC natively:
1. Buat Request baru → pilih **gRPC**
2. Masukkan URL: `grpc://localhost:50051`
3. Klik **Import .proto file** atau gunakan **Server Reflection**
4. Pilih service dan method
5. Isi request body dalam JSON
6. Klik **Invoke**

### BloomRPC / Kreya

Alternatif GUI lain:
- **BloomRPC**: https://github.com/bloomrpc/bloomrpc (open source, simple)
- **Kreya**: https://kreya.app (paid, fitur kolaborasi)

### Logging dan Tracing untuk gRPC

```go
// interceptor/logging.go — logging yang informatif
package interceptor

import (
    "context"
    "time"

    "go.uber.org/zap"
    "google.golang.org/grpc"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/peer"
    "google.golang.org/grpc/status"
)

func LoggingInterceptor(logger *zap.Logger) grpc.UnaryServerInterceptor {
    return func(
        ctx context.Context,
        req interface{},
        info *grpc.UnaryServerInfo,
        handler grpc.UnaryHandler,
    ) (interface{}, error) {
        start := time.Now()

        // Ekstrak peer (client address)
        peerInfo, _ := peer.FromContext(ctx)
        clientAddr := ""
        if peerInfo != nil {
            clientAddr = peerInfo.Addr.String()
        }

        // Jalankan handler
        resp, err := handler(ctx, req)

        // Log hasilnya
        duration := time.Since(start)
        code := codes.OK
        if err != nil {
            code = status.Code(err)
        }

        logFields := []zap.Field{
            zap.String("method", info.FullMethod),
            zap.String("code", code.String()),
            zap.Duration("duration", duration),
            zap.String("peer", clientAddr),
        }

        if err != nil {
            logFields = append(logFields, zap.Error(err))
            if code == codes.Internal || code == codes.Unknown {
                logger.Error("gRPC call failed", logFields...)
            } else {
                logger.Warn("gRPC call returned error", logFields...)
            }
        } else {
            logger.Info("gRPC call completed", logFields...)
        }

        return resp, err
    }
}
```

### Debugging Common Issues

```bash
# Issue 1: "connection refused"
# → Server belum berjalan atau port salah
grpcurl -plaintext localhost:50051 list 2>&1
# Solusi: cek dengan netstat -tlnp | grep 50051

# Issue 2: "transport: error while dialing: dial tcp: missing address"
# → Lupa isi host:port
grpcurl -plaintext localhost:50051 list  # BENAR
grpcurl -plaintext :50051 list           # SALAH

# Issue 3: "failed to dial: context deadline exceeded"
# → Reflection tidak aktif atau firewall
# Solusi: pastikan reflection.Register(srv) dipanggil

# Issue 4: "code = Unauthenticated"
# → Token tidak dikirim atau format salah
grpcurl -plaintext \
  -H 'authorization: Bearer TOKEN_HERE' \  # spasi setelah Bearer!
  -d '{}' localhost:50051 service/Method

# Issue 5: Streaming tidak menerima data
# → Cek apakah server menutup stream dengan benar (return nil dari handler)
```

### 🏋️ Latihan 5.11

1. Gunakan **grpcurl** untuk test semua endpoint Product Catalog service. Buat file `scripts/test-grpc.sh` yang berisi semua perintah grpcurl yang diperlukan untuk test lengkap (create, get, update, delete, stream, batch).
2. Setup **Evans** dan lakukan interactive call ke semua RPC. Screenshot atau record hasilnya.
3. Aktifkan logging interceptor di Product Catalog service. Jalankan beberapa request dan verifikasi bahwa: (a) setiap request di-log, (b) duration tercatat, (c) error code untuk request yang gagal berbeda dengan yang sukses.

---

## 📦 Modul 5.12 — gRPC + Clean Architecture

Integrasi gRPC ke Clean Architecture sangat natural — gRPC handler hanyalah **delivery layer** yang berbeda, di samping HTTP handler.

```
DELIVERY LAYER:
├── http/
│   ├── handler/user_handler.go     ← HTTP delivery
│   └── router.go
└── grpc/
    ├── handler/product_handler.go  ← gRPC delivery
    └── server.go

USE CASE LAYER (sama!):
└── usecase/
    └── product/
        ├── get_product.go          ← Use case SAMA untuk HTTP dan gRPC
        └── create_product.go
```

Keuntungannya: **Use Case yang sama** bisa dipakai oleh HTTP handler DAN gRPC handler. Kamu hanya perlu menulis business logic sekali.

```go
// Use case ini bisa dipakai oleh HTTP handler DAN gRPC handler
type getProductUseCase struct {
    productRepo repository.ProductRepository
}

func (uc *getProductUseCase) Execute(ctx context.Context, input GetProductInput) (*GetProductOutput, error) {
    product, err := uc.productRepo.FindByID(ctx, input.ID)
    // ... business logic SAMA untuk semua delivery ...
}

// ===== HTTP Handler =====
func (h *ProductHTTPHandler) GetProduct(c *gin.Context) {
    id, _ := strconv.ParseUint(c.Param("id"), 10, 64)
    output, err := h.getProductUC.Execute(c.Request.Context(), GetProductInput{ID: id})
    // format ke JSON response
}

// ===== gRPC Handler =====
func (s *ProductGRPCServer) GetProduct(ctx context.Context, req *productv1.GetProductRequest) (*productv1.GetProductResponse, error) {
    output, err := s.getProductUC.Execute(ctx, GetProductInput{ID: req.Id})
    // format ke proto response
}
```

---


### 🏋️ Latihan 5.12 — gRPC + Clean Architecture

1. Refactor Product Service sehingga **use case yang sama** bisa dipanggil dari HTTP handler (Gin) DAN gRPC handler. Buat integration test yang memverifikasi keduanya menghasilkan data yang konsisten.
2. Buat `ProductServiceClient` interface di Order Service yang mengabstraksi komunikasi ke Product Service. Implementasikan dengan gRPC client, dan buat mock untuk testing Order Service.


## 📦 Modul 5.13 — gRPC Gateway

gRPC Gateway memungkinkan kamu expose gRPC service sebagai REST API — satu codebase untuk dua protokol!

```bash
go get github.com/grpc-ecosystem/grpc-gateway/v2
go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@latest
```

```protobuf
// product.proto — tambahkan HTTP annotations
syntax = "proto3";

import "google/api/annotations.proto";

service ProductService {
  rpc GetProduct(GetProductRequest) returns (GetProductResponse) {
    option (google.api.http) = {
      get: "/api/v1/products/{id}"  // Expose sebagai GET /api/v1/products/{id}
    };
  }

  rpc CreateProduct(CreateProductRequest) returns (CreateProductResponse) {
    option (google.api.http) = {
      post: "/api/v1/products"
      body: "*"
    };
  }

  rpc DeleteProduct(DeleteProductRequest) returns (google.protobuf.Empty) {
    option (google.api.http) = {
      delete: "/api/v1/products/{id}"
    };
  }
}
```

```go
// cmd/gateway/main.go
package main

import (
    "context"
    "net/http"

    "github.com/grpc-ecosystem/grpc-gateway/v2/runtime"
    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"

    productv1 "github.com/kamu/product-service/gen/proto/product/v1"
)

func main() {
    ctx := context.Background()
    mux := runtime.NewServeMux()

    opts := []grpc.DialOption{grpc.WithTransportCredentials(insecure.NewCredentials())}

    // Daftarkan gateway ke gRPC server
    err := productv1.RegisterProductServiceHandlerFromEndpoint(
        ctx, mux, "localhost:50051", opts,
    )
    if err != nil {
        panic(err)
    }

    // Start HTTP server
    http.ListenAndServe(":8080", mux)
}
```

---

## 🛠️ Testing gRPC dengan grpcurl

```bash
# List semua services
grpcurl -plaintext localhost:50051 list

# List methods dalam service
grpcurl -plaintext localhost:50051 list product.v1.ProductService

# Describe message
grpcurl -plaintext localhost:50051 describe product.v1.Product

# Call unary RPC
grpcurl -plaintext \
  -d '{"id": 1}' \
  localhost:50051 product.v1.ProductService/GetProduct

# Call dengan metadata (auth token)
grpcurl -plaintext \
  -H 'authorization: Bearer eyJ...' \
  -d '{"name":"Laptop","price":15000000,"stock":10}' \
  localhost:50051 product.v1.ProductService/CreateProduct

# Call streaming (server stream)
grpcurl -plaintext \
  -d '{"category":"electronics","per_page":5}' \
  localhost:50051 product.v1.ProductService/ListProducts
```

---


### 🏋️ Latihan 5.13 — gRPC Gateway

1. Tambahkan HTTP annotations ke semua unary RPC di Product Service. Generate gateway, jalankan di port 8080. Verifikasi bahwa semua endpoint REST berfungsi dengan curl.
2. Konfigurasi custom error handling di gateway: pastikan gRPC error codes diterjemahkan ke HTTP status yang tepat (NotFound → 404, InvalidArgument → 400, dll.) dalam format response yang konsisten.


## 🎯 Review & Checkpoint Fase 5

### Konseptual
- [ ] Jelaskan perbedaan 4 tipe RPC di gRPC
- [ ] Mengapa protobuf lebih efisien dari JSON?
- [ ] Apa itu field number dan mengapa penting untuk backward compatibility?
- [ ] Apa perbedaan interceptor unary vs stream?
- [ ] Mengapa Use Case tidak perlu diubah saat menambah gRPC delivery?

### Praktis
- [ ] Bisa menulis .proto file dengan message dan service yang kompleks
- [ ] Bisa generate Go code dari proto
- [ ] Bisa implementasi unary, server-streaming, client-streaming RPC
- [ ] Bisa menulis interceptor (logging, auth)
- [ ] Bisa test dengan grpcurl

---

## 🎯 Project Akhir Fase 5

Kerjakan project **Product Catalog gRPC Service** berdasarkan PRD di file:

**`FASE-5-PRD-Product-Catalog.md`**

---

*Setelah selesai Fase 5, lanjut ke `FASE-6-DDD.md`*

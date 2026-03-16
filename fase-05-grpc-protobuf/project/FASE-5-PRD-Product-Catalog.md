# 📋 PRD — Product Catalog gRPC Service

> **Fase:** 5 — gRPC + Protocol Buffers  
> **Tipe Project:** gRPC Microservice dengan Clean Architecture  
> **Estimasi Pengerjaan:** 7–10 hari  
> **Konsep yang Diuji:** Protobuf, All 4 RPC types, Interceptors, Error handling, gRPC Gateway

---

## 🎯 Tujuan Project

Membangun **Product Catalog Service** menggunakan gRPC + Protobuf dengan Clean Architecture. Service ini akan menjadi salah satu service di ekosistem microservices yang dibangun di Fase 7.

---

## 📌 Latar Belakang

Platform e-commerce membutuhkan service yang mengelola katalog produk dengan performa tinggi. Service ini akan dikonsumsi oleh:
- Order Service (gRPC — cek stok sebelum checkout)
- Frontend (REST via gRPC Gateway)
- Admin Panel (gRPC)

---

## 📡 gRPC Service Definition

```protobuf
syntax = "proto3";
package product.v1;
option go_package = "github.com/kamu/product-service/gen/proto/product/v1;productv1";

import "google/protobuf/timestamp.proto";
import "google/protobuf/empty.proto";
import "google/api/annotations.proto";

// ===== ENUMS =====
enum ProductStatus {
  PRODUCT_STATUS_UNSPECIFIED = 0;
  PRODUCT_STATUS_ACTIVE      = 1;
  PRODUCT_STATUS_INACTIVE    = 2;
  PRODUCT_STATUS_OUT_OF_STOCK = 3;
}

// ===== MESSAGES =====
message Product {
  uint64 id          = 1;
  string name        = 2;
  string description = 3;
  double price       = 4;
  uint32 stock       = 5;
  string category    = 6;
  string sku         = 7;
  ProductStatus status = 8;
  repeated string images = 9;
  google.protobuf.Timestamp created_at = 10;
  google.protobuf.Timestamp updated_at = 11;
}

message Category {
  uint64 id       = 1;
  string name     = 2;
  string slug     = 3;
  uint32 product_count = 4;
}

// ===== REQUESTS / RESPONSES =====
message GetProductRequest   { uint64 id = 1; }
message GetProductResponse  { Product product = 1; }
message GetProductBySkuRequest { string sku = 1; }

message CreateProductRequest {
  string name        = 1;
  string description = 2;
  double price       = 3;
  uint32 stock       = 4;
  string category    = 5;
  string sku         = 6;
  repeated string images = 7;
}
message CreateProductResponse { Product product = 1; }

message UpdateProductRequest {
  uint64 id          = 1;
  string name        = 2;
  string description = 3;
  double price       = 4;
  string category    = 5;
  repeated string images = 6;
}
message UpdateProductResponse { Product product = 1; }

message UpdateStockRequest {
  uint64 id       = 1;
  int32  quantity = 2;  // bisa negatif untuk kurangi stok
  string reason   = 3;  // "sale", "restock", "adjustment"
}
message UpdateStockResponse {
  Product product     = 1;
  uint32  prev_stock  = 2;
  uint32  new_stock   = 3;
}

message DeleteProductRequest { uint64 id = 1; }

message ListProductsRequest {
  uint32 page      = 1;
  uint32 per_page  = 2;
  string category  = 3;
  string search    = 4;
  double min_price = 5;
  double max_price = 6;
  ProductStatus status = 7;
  string sort_by   = 8;
  string sort_dir  = 9;
}

message BatchGetProductsRequest  { repeated uint64 ids = 1; }
message BatchGetProductsResponse { repeated Product products = 1; }

message BatchCreateRequest  { repeated CreateProductRequest products = 1; }
message BatchCreateResponse {
  uint32 created = 1;
  uint32 failed  = 2;
  repeated string errors = 3;
}

message CheckStockRequest  {
  uint64 product_id = 1;
  uint32 quantity   = 2;
}
message CheckStockResponse {
  bool   available    = 1;
  uint32 current_stock = 2;
}

message StockUpdateEvent {
  uint64 product_id = 1;
  uint32 old_stock  = 2;
  uint32 new_stock  = 3;
  string reason     = 4;
  google.protobuf.Timestamp updated_at = 5;
}

// ===== SERVICE =====
service ProductService {
  // --- Unary RPCs ---
  rpc GetProduct(GetProductRequest) returns (GetProductResponse) {
    option (google.api.http) = { get: "/api/v1/products/{id}" };
  }
  rpc GetProductBySku(GetProductBySkuRequest) returns (GetProductResponse) {
    option (google.api.http) = { get: "/api/v1/products/sku/{sku}" };
  }
  rpc CreateProduct(CreateProductRequest) returns (CreateProductResponse) {
    option (google.api.http) = { post: "/api/v1/products"; body: "*" };
  }
  rpc UpdateProduct(UpdateProductRequest) returns (UpdateProductResponse) {
    option (google.api.http) = { put: "/api/v1/products/{id}"; body: "*" };
  }
  rpc DeleteProduct(DeleteProductRequest) returns (google.protobuf.Empty) {
    option (google.api.http) = { delete: "/api/v1/products/{id}" };
  }
  rpc UpdateStock(UpdateStockRequest) returns (UpdateStockResponse);
  rpc CheckStock(CheckStockRequest) returns (CheckStockResponse);
  rpc BatchGetProducts(BatchGetProductsRequest) returns (BatchGetProductsResponse);

  // --- Server Streaming ---
  // Stream semua produk dalam kategori (untuk export/sync)
  rpc StreamProductsByCategory(ListProductsRequest) returns (stream Product);

  // --- Client Streaming ---
  // Upload banyak produk sekaligus (bulk import)
  rpc BatchCreateProducts(stream CreateProductRequest) returns (BatchCreateResponse);

  // --- Bidirectional Streaming ---
  // Real-time stock update (untuk warehouse system)
  rpc StreamStockUpdates(stream UpdateStockRequest) returns (stream StockUpdateEvent);
}

service CategoryService {
  rpc ListCategories(google.protobuf.Empty) returns (stream Category);
  rpc GetCategory(GetCategoryRequest) returns (GetCategoryResponse);
  rpc CreateCategory(CreateCategoryRequest) returns (CreateCategoryResponse);
}
```

---

## ✅ Business Rules

### Product Rules
1. SKU harus unik di seluruh sistem (format: `CAT-XXXXX`, contoh: `ELEC-00001`)
2. Harga tidak boleh negatif
3. Stok tidak boleh negatif (operasi yang menyebabkan negatif harus ditolak)
4. Produk dengan stok 0 otomatis berstatus `OUT_OF_STOCK`
5. Produk yang di-delete tidak bisa diakses (soft delete)
6. Maksimal 10 gambar per produk

### Stock Rules
1. Pengurangan stok (quantity negatif) harus menyertakan `reason`
2. Reason yang valid: `sale`, `return`, `adjustment`, `restock`, `damage`
3. Tidak bisa kurangi stok melebihi stok yang tersedia
4. Setiap perubahan stok dicatat di `stock_history` table

---

## 📁 Struktur Project

```
product-service/
├── api/
│   └── proto/
│       └── product/
│           └── v1/
│               └── product.proto
├── gen/                         ← Generated (jangan edit!)
│   └── proto/
│       └── product/v1/
├── internal/
│   ├── domain/
│   │   ├── entity/
│   │   │   ├── product.go
│   │   │   └── category.go
│   │   ├── valueobject/
│   │   │   ├── sku.go           ← SKU value object dengan validasi
│   │   │   └── price.go         ← Price value object
│   │   └── apperror/
│   ├── repository/
│   │   ├── product_repository.go
│   │   └── category_repository.go
│   ├── usecase/
│   │   └── product/
│   │       ├── get_product.go
│   │       ├── create_product.go
│   │       ├── update_product.go
│   │       ├── delete_product.go
│   │       ├── list_products.go
│   │       ├── update_stock.go
│   │       └── check_stock.go
│   ├── delivery/
│   │   └── grpc/
│   │       ├── handler/
│   │       │   ├── product_handler.go
│   │       │   └── category_handler.go
│   │       ├── interceptor/
│   │       │   ├── logging.go
│   │       │   ├── recovery.go
│   │       │   └── auth.go
│   │       └── server.go
│   └── infrastructure/
│       └── persistence/postgres/
│           ├── product_repository.go
│           └── category_repository.go
├── cmd/
│   ├── grpc/main.go
│   └── gateway/main.go          ← Optional: REST gateway
├── buf.yaml
├── buf.gen.yaml
├── docker-compose.yml
└── go.mod
```

---

## 🗂️ Database Schema

```sql
CREATE TABLE categories (
    id         BIGSERIAL PRIMARY KEY,
    name       VARCHAR(100) NOT NULL UNIQUE,
    slug       VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE products (
    id          BIGSERIAL PRIMARY KEY,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at  TIMESTAMPTZ,
    name        VARCHAR(255) NOT NULL,
    description TEXT,
    price       DECIMAL(15,2) NOT NULL CHECK (price >= 0),
    stock       INT NOT NULL DEFAULT 0 CHECK (stock >= 0),
    sku         VARCHAR(50) NOT NULL,
    status      VARCHAR(30) NOT NULL DEFAULT 'PRODUCT_STATUS_ACTIVE',
    images      TEXT[],
    category_id BIGINT REFERENCES categories(id),
    CONSTRAINT products_sku_unique UNIQUE (sku) WHERE deleted_at IS NULL
);

CREATE INDEX idx_products_category ON products(category_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_products_sku ON products(sku) WHERE deleted_at IS NULL;
CREATE INDEX idx_products_status ON products(status) WHERE deleted_at IS NULL;

CREATE TABLE stock_history (
    id         BIGSERIAL PRIMARY KEY,
    product_id BIGINT NOT NULL REFERENCES products(id),
    old_stock  INT NOT NULL,
    new_stock  INT NOT NULL,
    change     INT NOT NULL,  -- positive=tambah, negative=kurang
    reason     VARCHAR(50) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 🧪 Test dengan grpcurl

```bash
# Start server
go run ./cmd/grpc/main.go

# List services
grpcurl -plaintext localhost:50051 list

# Create product
grpcurl -plaintext \
  -H 'authorization: Bearer <admin_token>' \
  -d '{
    "name": "Laptop Gaming ASUS",
    "description": "RTX 4070, 32GB RAM",
    "price": 25000000,
    "stock": 20,
    "category": "electronics",
    "sku": "ELEC-00001"
  }' \
  localhost:50051 product.v1.ProductService/CreateProduct

# Get product
grpcurl -plaintext \
  -d '{"id": 1}' \
  localhost:50051 product.v1.ProductService/GetProduct

# Check stock
grpcurl -plaintext \
  -d '{"product_id": 1, "quantity": 5}' \
  localhost:50051 product.v1.ProductService/CheckStock

# Stream products
grpcurl -plaintext \
  -d '{"category": "electronics", "per_page": 100}' \
  localhost:50051 product.v1.ProductService/StreamProductsByCategory

# Error cases
grpcurl -plaintext \
  -d '{"id": 999}' \
  localhost:50051 product.v1.ProductService/GetProduct
# Expected: NOT_FOUND

grpcurl -plaintext \
  -d '{"id": 1, "quantity": -1000, "reason": "sale"}' \
  localhost:50051 product.v1.ProductService/UpdateStock
# Expected: FAILED_PRECONDITION (stok tidak cukup)
```

---

## 🏆 Kriteria Penilaian

### Minimum
- [ ] .proto file lengkap dengan semua messages dan service
- [ ] Semua unary RPC (GetProduct, CreateProduct, UpdateProduct, DeleteProduct)
- [ ] CheckStock dan UpdateStock
- [ ] LoggingInterceptor dan RecoveryInterceptor
- [ ] Error handling dengan gRPC status codes
- [ ] Clean Architecture tetap diterapkan

### Good
- [ ] StreamProductsByCategory (server streaming)
- [ ] BatchCreateProducts (client streaming)
- [ ] AuthInterceptor
- [ ] Unit tests untuk use case
- [ ] grpcurl test berhasil untuk semua endpoint

### Excellent
- [ ] StreamStockUpdates (bidirectional streaming)
- [ ] gRPC Gateway (expose sebagai REST)
- [ ] BatchGetProducts dengan optimasi query
- [ ] Rich error details (errdetails)
- [ ] Integration tests

---

*Service ini akan menjadi bagian dari sistem microservices di Fase 7. Pastikan gRPC berjalan dengan baik!*

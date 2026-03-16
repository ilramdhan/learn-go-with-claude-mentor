# 📘 FASE 6: Domain-Driven Design (DDD)

> **Prasyarat:** Selesaikan Fase 4 + Fase 5  
> **Durasi:** 3–4 minggu  
> **Project Akhir:** Order Service dengan Full DDD  
> **Tujuan:** Memahami dan mengimplementasikan DDD pattern yang digunakan di perusahaan tech besar untuk mengelola kompleksitas domain bisnis

---

## 🗂️ Daftar Modul

| # | Modul | Topik |
|---|-------|-------|
| 6.1 | Apa itu DDD? | Filosofi, tactical vs strategic |
| 6.2 | Ubiquitous Language | Shared vocabulary bisnis & teknis |
| 6.3 | Entity & Value Object | Perbedaan, identitas, immutability |
| 6.4 | Aggregate & Aggregate Root | Batasan konsistensi, invariants |
| 6.5 | Repository Pattern (DDD) | Persistensi aggregate |
| 6.6 | Domain Events | Kejadian penting di domain |
| 6.7 | Domain Services | Logic yang tidak milik entity manapun |
| 6.8 | Application Services | Orchestration, tidak ada business logic |
| 6.9 | Bounded Context | Pemisahan domain yang jelas |
| 6.10 | CQRS Pattern | Command Query Responsibility Segregation |
| 6.11 | Event Sourcing Intro | State sebagai urutan events |
| 6.12 | Anti-Corruption Layer | Isolasi dari sistem lain |

---

## 📦 Modul 6.1 — Apa itu DDD?

### Filosofi DDD

DDD (Domain-Driven Design) adalah pendekatan untuk membangun software yang **kompleksitas-nya dikelola dengan cara memodelkan domain bisnis secara akurat ke dalam kode**.

> "The heart of software is its ability to solve domain-related problems for its users."
> — Eric Evans, Domain-Driven Design (2003)

### Masalah yang DDD Selesaikan

```
Tanpa DDD:
├── Business logic tersebar di mana-mana
├── Model database = model bisnis (anemic domain model)
├── Perubahan requirement → perubahan di banyak tempat
├── Developer tidak paham domain bisnis
└── Bug karena asumsi bisnis yang salah

Dengan DDD:
├── Business logic terpusat di domain layer
├── Model bisnis ekspresif dan self-documenting
├── Perubahan requirement → terlokalisir di domain yang tepat
├── Developer berbicara bahasa yang sama dengan bisnis
└── Bug berkurang karena invariants dijaga oleh domain
```

### Tactical vs Strategic DDD

```
STRATEGIC DDD (Arsitektur tinggi)
├── Bounded Context  — batas domain yang jelas
├── Context Map      — hubungan antar bounded context
├── Ubiquitous Language — bahasa bersama per context
└── Core/Supporting/Generic Domain

TACTICAL DDD (Pattern implementasi)
├── Entity           — objek dengan identitas
├── Value Object     — objek immutable tanpa identitas
├── Aggregate        — cluster entity yang konsisten
├── Repository       — persistensi aggregate
├── Domain Event     — kejadian penting di domain
├── Domain Service   — logic yang tidak milik entity
└── Factory          — cara membuat objek yang kompleks
```

### Anemic Domain Model vs Rich Domain Model

```go
// ❌ ANEMIC DOMAIN MODEL — model hanya data, logic di service
type Order struct {
    ID     uint
    Items  []OrderItem
    Status string
    Total  float64
}

// Logic ada di "service" yang terpisah
type OrderService struct{}
func (s *OrderService) AddItem(order *Order, item Item) { ... }
func (s *OrderService) Cancel(order *Order) { ... }
func (s *OrderService) Calculate(order *Order) { ... }
// Masalah: Order bisa dalam state tidak valid kapan saja

// ✅ RICH DOMAIN MODEL — model punya logic sendiri
type Order struct {
    id     OrderID       // private — tidak bisa diset sembarangan
    items  []OrderItem   // private — hanya bisa diubah via method
    status OrderStatus
    total  Money
}

// Method mengekspresikan business rules
func (o *Order) AddItem(item Item, quantity int) error {
    if o.status != StatusDraft {
        return ErrOrderNotDraft  // invariant dijaga!
    }
    // ...
}

func (o *Order) Cancel(reason string) error {
    if o.status == StatusDelivered {
        return ErrCannotCancelDelivered  // business rule!
    }
    // ...
}
```

---



### 🏋️ Latihan 6.1

1. Ambil kode Order Service yang ada dan identifikasi: mana yang seharusnya jadi Entity (punya identitas), mana yang Value Object (tidak punya identitas). Buat daftar justifikasi untuk setiap keputusan.
2. Buat diagram **domain model** untuk sistem perpustakaan: buku, anggota, peminjaman, denda. Identifikasi aggregate boundaries dan tentukan aggregate root-nya.
3. Tulis **Ubiquitous Language glossary** untuk domain e-commerce Order: minimal 15 istilah dengan definisi bisnis yang presisi (bukan definisi teknis).


## 📦 Modul 6.2 — Ubiquitous Language

Ubiquitous Language adalah **kosakata bersama** yang dipakai oleh developer DAN domain expert (bisnis) untuk berbicara tentang domain yang sama.

### Contoh: Domain E-Commerce Order

```
Tanpa Ubiquitous Language:
  Dev: "Kita update status order di database"
  Bisnis: "Maksudnya apa? Status apa?"

Dengan Ubiquitous Language:
  Dev: "Kita confirm order setelah payment berhasil"
  Bisnis: "Ya benar, dan setelah confirm, stok sudah dikurangi"
  Dev: "Lalu order bisa di-dispatch ke warehouse"
  Bisnis: "Tepat. Tapi kalau cancel setelah dispatch, stok harus dikembalikan"
```

### Order Lifecycle yang Terdefinisi

```
Order Status Flow:
                        ┌──────────────┐
                        │   DRAFT      │ ← Bisa tambah/hapus item
                        └──────┬───────┘
                               │ customer.submitOrder()
                        ┌──────▼───────┐
                        │   PENDING    │ ← Menunggu pembayaran
                        └──────┬───────┘
                    ┌──────────┤
          timeout   │          │ payment.confirmed()
                    │   ┌──────▼───────┐
                    │   │  CONFIRMED   │ ← Stok sudah dikurangi
                    │   └──────┬───────┘
                    │          │ warehouse.dispatch()
                    │   ┌──────▼───────┐
                    │   │  DISPATCHED  │ ← Dalam pengiriman
                    │   └──────┬───────┘
                    │          │ courier.delivered()
                    │   ┌──────▼───────┐
                    │   │  DELIVERED   │ ← Selesai
                    │   └──────────────┘
                    │
             ┌──────▼───────┐
             │  CANCELLED   │ ← Stok dikembalikan jika dari CONFIRMED
             └──────────────┘
```

---



### 🏋️ Latihan 6.2

1. Wawancarai domain expert (bayangkan skenario): kamu developer, ada product manager. Tulis daftar 10 pertanyaan yang perlu kamu tanyakan untuk memahami domain "Order Cancellation" dengan baik.
2. Identifikasi **ambiguitas bahasa** dalam requirements berikut: "User bisa cancel order kapan saja sebelum dikirim". Apa yang perlu dikklarifikasi? Tulis semua asumsi yang mungkin salah.
3. Buat **state diagram** yang lengkap untuk Order lifecycle dengan semua transisi, kondisi pre/post, dan siapa yang bisa memicu setiap transisi.


## 📦 Modul 6.3 — Entity & Value Object (DDD Style)

### Entity

Entity memiliki **identitas** yang persisten, terlepas dari nilainya.

```go
// internal/domain/order/entity/order_id.go
package entity

import "fmt"

// OrderID adalah typed ID untuk mencegah passing ID yang salah
type OrderID uint64

func NewOrderID(id uint64) (OrderID, error) {
    if id == 0 {
        return 0, fmt.Errorf("order id cannot be zero")
    }
    return OrderID(id), nil
}

func (id OrderID) IsZero() bool { return id == 0 }
func (id OrderID) Value() uint64 { return uint64(id) }
func (id OrderID) String() string { return fmt.Sprintf("ORDER-%d", id) }
```

### Value Object (Lengkap dan Immutable)

```go
// internal/domain/order/valueobject/money.go
package valueobject

import (
    "fmt"
    "math"
)

// Money adalah value object yang merepresentasikan nilai uang
// Immutable setelah dibuat
type Money struct {
    amount   int64  // dalam satuan terkecil (sen/rupiah), hindari floating point!
    currency string // "IDR", "USD", dll
}

// NewMoney membuat Money baru dengan validasi
func NewMoney(amount float64, currency string) (Money, error) {
    if currency == "" {
        return Money{}, fmt.Errorf("currency cannot be empty")
    }
    if amount < 0 {
        return Money{}, fmt.Errorf("amount cannot be negative: %.2f", amount)
    }
    // Konversi ke int64 (dalam sen) untuk menghindari floating point error
    amountInCents := int64(math.Round(amount * 100))
    return Money{amount: amountInCents, currency: currency}, nil
}

// MustNewMoney untuk konstanta yang dijamin valid
func MustNewMoney(amount float64, currency string) Money {
    m, err := NewMoney(amount, currency)
    if err != nil {
        panic(err)
    }
    return m
}

// Accessor methods — tidak ada setter!
func (m Money) Amount() float64    { return float64(m.amount) / 100 }
func (m Money) AmountInCents() int64 { return m.amount }
func (m Money) Currency() string   { return m.currency }

// Operations — selalu return Money baru (immutable!)
func (m Money) Add(other Money) (Money, error) {
    if m.currency != other.currency {
        return Money{}, fmt.Errorf("cannot add %s and %s", m.currency, other.currency)
    }
    return Money{amount: m.amount + other.amount, currency: m.currency}, nil
}

func (m Money) Subtract(other Money) (Money, error) {
    if m.currency != other.currency {
        return Money{}, fmt.Errorf("cannot subtract %s from %s", other.currency, m.currency)
    }
    if m.amount < other.amount {
        return Money{}, fmt.Errorf("insufficient amount: %.2f < %.2f", m.Amount(), other.Amount())
    }
    return Money{amount: m.amount - other.amount, currency: m.currency}, nil
}

func (m Money) Multiply(factor int) Money {
    return Money{amount: m.amount * int64(factor), currency: m.currency}
}

func (m Money) IsZero() bool                  { return m.amount == 0 }
func (m Money) Equals(other Money) bool        { return m.amount == other.amount && m.currency == other.currency }
func (m Money) GreaterThan(other Money) bool   { return m.amount > other.amount }
func (m Money) LessThan(other Money) bool      { return m.amount < other.amount }

func (m Money) String() string {
    return fmt.Sprintf("%.2f %s", m.Amount(), m.currency)
}
```

```go
// internal/domain/order/valueobject/address.go
package valueobject

import (
    "fmt"
    "strings"
)

// Address adalah value object untuk alamat pengiriman
type Address struct {
    street     string
    city       string
    province   string
    postalCode string
    country    string
}

func NewAddress(street, city, province, postalCode, country string) (Address, error) {
    street = strings.TrimSpace(street)
    city = strings.TrimSpace(city)

    if street == "" {
        return Address{}, fmt.Errorf("street cannot be empty")
    }
    if city == "" {
        return Address{}, fmt.Errorf("city cannot be empty")
    }
    if len(postalCode) < 5 {
        return Address{}, fmt.Errorf("invalid postal code: %s", postalCode)
    }
    if country == "" {
        return Address{}, fmt.Errorf("country cannot be empty")
    }

    return Address{
        street:     street,
        city:       city,
        province:   province,
        postalCode: postalCode,
        country:    country,
    }, nil
}

func (a Address) Street() string     { return a.street }
func (a Address) City() string       { return a.city }
func (a Address) Province() string   { return a.province }
func (a Address) PostalCode() string { return a.postalCode }
func (a Address) Country() string    { return a.country }

func (a Address) FullAddress() string {
    return fmt.Sprintf("%s, %s, %s %s, %s",
        a.street, a.city, a.province, a.postalCode, a.country)
}

func (a Address) Equals(other Address) bool {
    return a.street == other.street &&
        a.city == other.city &&
        a.postalCode == other.postalCode &&
        a.country == other.country
}
```

---



### 🏋️ Latihan 6.3

1. Implementasikan Value Object `PhoneNumber` yang: (a) mendukung format Indonesia (+62xxx, 08xxx), (b) normalisasi ke format +62, (c) immutable, (d) punya method `IsValid()`, `Formatted()`, `Local()`. Tulis unit test lengkap.
2. Buat Entity `Customer` dengan business methods: `UpgradeTier(newTier Tier)`, `UpdateProfile(name, phone string)`, `Deactivate(reason string)`. Setiap method harus validate invariants dan raise domain events.
3. Jelaskan mengapa `OrderID(1) == OrderID(1)` adalah TRUE (entity equality by ID), tapi `Money{100, "IDR"} == Money{100, "IDR"}` juga TRUE (value object equality by value) — tulis penjelasannya dalam komentar kode.


## 📦 Modul 6.4 — Aggregate & Aggregate Root

**Aggregate** adalah cluster entity dan value object yang diperlakukan sebagai satu unit untuk tujuan perubahan data. **Aggregate Root** adalah entity utama yang menjadi pintu masuk semua operasi.

### Aturan Aggregate
1. **Referensikan aggregate lain hanya via ID**, bukan objek
2. **Satu transaction, satu aggregate** (idealnya)
3. **Konsistensi dalam boundary aggregate** dijaga oleh aggregate root
4. **Akses dari luar hanya melalui aggregate root**

```go
// internal/domain/order/aggregate/order.go
package aggregate

import (
    "fmt"
    "time"

    "github.com/kamu/order-service/internal/domain/order/entity"
    "github.com/kamu/order-service/internal/domain/order/event"
    "github.com/kamu/order-service/internal/domain/order/valueobject"
    "github.com/kamu/order-service/internal/domain/shared/apperror"
)

// OrderStatus adalah status order yang valid
type OrderStatus string

const (
    StatusDraft      OrderStatus = "DRAFT"
    StatusPending    OrderStatus = "PENDING"
    StatusConfirmed  OrderStatus = "CONFIRMED"
    StatusDispatched OrderStatus = "DISPATCHED"
    StatusDelivered  OrderStatus = "DELIVERED"
    StatusCancelled  OrderStatus = "CANCELLED"
)

// OrderItem adalah entity di dalam aggregate Order
// Tidak bisa diakses langsung dari luar, hanya via Order methods
type OrderItem struct {
    id         entity.OrderItemID
    productID  uint64           // referensi ke Product aggregate via ID
    productName string
    quantity   int
    unitPrice  valueobject.Money
}

func (i OrderItem) ID() entity.OrderItemID   { return i.id }
func (i OrderItem) ProductID() uint64        { return i.productID }
func (i OrderItem) ProductName() string      { return i.productName }
func (i OrderItem) Quantity() int            { return i.quantity }
func (i OrderItem) UnitPrice() valueobject.Money { return i.unitPrice }
func (i OrderItem) Subtotal() valueobject.Money  { return i.unitPrice.Multiply(i.quantity) }

// Order adalah Aggregate Root
// Semua perubahan ke state Order harus melalui method di struct ini
type Order struct {
    // Identity
    id entity.OrderID

    // Customer (referensi ke Customer aggregate via ID)
    customerID uint64

    // Items
    items []OrderItem

    // State
    status          OrderStatus
    shippingAddress valueobject.Address
    totalAmount     valueobject.Money
    notes           string

    // Timestamps
    createdAt   time.Time
    updatedAt   time.Time
    confirmedAt *time.Time
    cancelledAt *time.Time

    // Domain Events — dikumpulkan, di-publish setelah save
    domainEvents []event.DomainEvent
}

// ===== Factory Method =====

// NewOrder adalah satu-satunya cara membuat Order baru
func NewOrder(customerID uint64, shippingAddress valueobject.Address) (*Order, error) {
    if customerID == 0 {
        return nil, apperror.ErrInvalidCustomerID
    }

    order := &Order{
        customerID:      customerID,
        items:           make([]OrderItem, 0),
        status:          StatusDraft,
        shippingAddress: shippingAddress,
        totalAmount:     valueobject.MustNewMoney(0, "IDR"),
        createdAt:       time.Now(),
        updatedAt:       time.Now(),
    }

    // Raise domain event
    order.addEvent(event.NewOrderCreated(order))

    return order, nil
}

// ReconstitutOrder dipakai saat memuat Order dari database
// Tidak raise domain events
func ReconstitutOrder(
    id entity.OrderID,
    customerID uint64,
    items []OrderItem,
    status OrderStatus,
    shippingAddress valueobject.Address,
    totalAmount valueobject.Money,
    notes string,
    createdAt, updatedAt time.Time,
    confirmedAt, cancelledAt *time.Time,
) *Order {
    return &Order{
        id:              id,
        customerID:      customerID,
        items:           items,
        status:          status,
        shippingAddress: shippingAddress,
        totalAmount:     totalAmount,
        notes:           notes,
        createdAt:       createdAt,
        updatedAt:       updatedAt,
        confirmedAt:     confirmedAt,
        cancelledAt:     cancelledAt,
    }
}

// ===== Read methods =====

func (o *Order) ID() entity.OrderID              { return o.id }
func (o *Order) CustomerID() uint64              { return o.customerID }
func (o *Order) Items() []OrderItem              { return o.items }  // return copy
func (o *Order) Status() OrderStatus             { return o.status }
func (o *Order) ShippingAddress() valueobject.Address { return o.shippingAddress }
func (o *Order) TotalAmount() valueobject.Money  { return o.totalAmount }
func (o *Order) Notes() string                   { return o.notes }
func (o *Order) CreatedAt() time.Time            { return o.createdAt }
func (o *Order) UpdatedAt() time.Time            { return o.updatedAt }
func (o *Order) ConfirmedAt() *time.Time         { return o.confirmedAt }
func (o *Order) CancelledAt() *time.Time         { return o.cancelledAt }
func (o *Order) DomainEvents() []event.DomainEvent { return o.domainEvents }

// ===== Business methods (mengubah state + validasi invariants) =====

// AddItem menambahkan produk ke order
// Invariant: hanya bisa tambah item jika status DRAFT
func (o *Order) AddItem(productID uint64, productName string, quantity int, unitPrice valueobject.Money) error {
    // Invariant check
    if o.status != StatusDraft {
        return apperror.NewDomainError("INVALID_ORDER_STATUS",
            fmt.Sprintf("cannot add item to order with status %s, must be DRAFT", o.status),
        )
    }

    if quantity <= 0 {
        return apperror.NewDomainError("INVALID_QUANTITY", "quantity must be positive")
    }
    if productID == 0 {
        return apperror.NewDomainError("INVALID_PRODUCT", "product id cannot be zero")
    }

    // Cek apakah produk sudah ada di order — update quantity
    for i, item := range o.items {
        if item.productID == productID {
            o.items[i].quantity += quantity
            o.recalculateTotal()
            o.updatedAt = time.Now()
            return nil
        }
    }

    // Tambah item baru
    newItem := OrderItem{
        id:          entity.NewOrderItemID(),
        productID:   productID,
        productName: productName,
        quantity:    quantity,
        unitPrice:   unitPrice,
    }
    o.items = append(o.items, newItem)
    o.recalculateTotal()
    o.updatedAt = time.Now()

    return nil
}

// RemoveItem menghapus item dari order
func (o *Order) RemoveItem(productID uint64) error {
    if o.status != StatusDraft {
        return apperror.NewDomainError("INVALID_ORDER_STATUS",
            "cannot remove item from non-draft order",
        )
    }

    for i, item := range o.items {
        if item.productID == productID {
            o.items = append(o.items[:i], o.items[i+1:]...)
            o.recalculateTotal()
            o.updatedAt = time.Now()
            return nil
        }
    }

    return apperror.ErrOrderItemNotFound
}

// Submit mengubah status order ke PENDING (menunggu pembayaran)
// Invariant: order harus punya minimal 1 item
func (o *Order) Submit() error {
    if o.status != StatusDraft {
        return apperror.NewDomainError("INVALID_ORDER_STATUS",
            "can only submit draft orders",
        )
    }
    if len(o.items) == 0 {
        return apperror.NewDomainError("EMPTY_ORDER", "cannot submit empty order")
    }

    o.status = StatusPending
    o.updatedAt = time.Now()

    // Raise domain event
    o.addEvent(event.NewOrderSubmitted(o))

    return nil
}

// Confirm mengubah status ke CONFIRMED setelah pembayaran berhasil
func (o *Order) Confirm(paymentID string) error {
    if o.status != StatusPending {
        return apperror.NewDomainError("INVALID_ORDER_STATUS",
            "can only confirm pending orders",
        )
    }

    now := time.Now()
    o.status = StatusConfirmed
    o.confirmedAt = &now
    o.updatedAt = now

    // Raise domain event — event ini akan trigger stock reduction!
    o.addEvent(event.NewOrderConfirmed(o, paymentID))

    return nil
}

// Dispatch mengubah status ke DISPATCHED
func (o *Order) Dispatch(trackingNumber string) error {
    if o.status != StatusConfirmed {
        return apperror.NewDomainError("INVALID_ORDER_STATUS",
            "can only dispatch confirmed orders",
        )
    }

    o.status = StatusDispatched
    o.updatedAt = time.Now()
    o.addEvent(event.NewOrderDispatched(o, trackingNumber))

    return nil
}

// Deliver mengubah status ke DELIVERED
func (o *Order) Deliver() error {
    if o.status != StatusDispatched {
        return apperror.NewDomainError("INVALID_ORDER_STATUS",
            "can only deliver dispatched orders",
        )
    }

    o.status = StatusDelivered
    o.updatedAt = time.Now()
    o.addEvent(event.NewOrderDelivered(o))

    return nil
}

// Cancel membatalkan order
// Business rule: tidak bisa cancel jika sudah DELIVERED
func (o *Order) Cancel(reason string) error {
    if o.status == StatusDelivered {
        return apperror.NewDomainError("CANNOT_CANCEL_DELIVERED",
            "cannot cancel a delivered order",
        )
    }
    if o.status == StatusCancelled {
        return apperror.NewDomainError("ALREADY_CANCELLED",
            "order is already cancelled",
        )
    }
    if reason == "" {
        return apperror.NewDomainError("CANCEL_REASON_REQUIRED",
            "cancellation reason is required",
        )
    }

    wasConfirmed := o.status == StatusConfirmed || o.status == StatusDispatched
    now := time.Now()
    o.status = StatusCancelled
    o.cancelledAt = &now
    o.updatedAt = now

    // Jika order sudah CONFIRMED, stok harus dikembalikan
    o.addEvent(event.NewOrderCancelled(o, reason, wasConfirmed))

    return nil
}

// UpdateShippingAddress mengubah alamat pengiriman
// Business rule: hanya bisa diubah saat DRAFT atau PENDING
func (o *Order) UpdateShippingAddress(newAddress valueobject.Address) error {
    if o.status != StatusDraft && o.status != StatusPending {
        return apperror.NewDomainError("INVALID_ORDER_STATUS",
            fmt.Sprintf("cannot update shipping address for %s order", o.status),
        )
    }

    o.shippingAddress = newAddress
    o.updatedAt = time.Now()
    return nil
}

// ===== Private helpers =====

func (o *Order) recalculateTotal() {
    total := valueobject.MustNewMoney(0, "IDR")
    for _, item := range o.items {
        subtotal := item.unitPrice.Multiply(item.quantity)
        // Ignore error karena currency sama
        total, _ = total.Add(subtotal)
    }
    o.totalAmount = total
}

func (o *Order) addEvent(e event.DomainEvent) {
    o.domainEvents = append(o.domainEvents, e)
}

// ClearDomainEvents menghapus events setelah di-publish
func (o *Order) ClearDomainEvents() {
    o.domainEvents = nil
}
```

---



### 🏋️ Latihan 6.4

1. Tambahkan method `ChangeItemQuantity(productID uint64, newQty int) error` ke Order aggregate. Pastikan: (a) hanya bisa dilakukan saat DRAFT, (b) qty tidak bisa 0 atau negatif, (c) total amount di-recalculate, (d) jika qty 0, item otomatis dihapus.
2. Tulis unit test lengkap untuk Order aggregate — target coverage 100%. Test setiap method dengan: success case, setiap possible error case, dan verifikasi bahwa domain events di-raise.
3. Buat test yang membuktikan bahwa concurrent calls ke `order.AddItem()` dari 2 goroutine berbeda dengan data berbeda menghasilkan total yang benar (hint: aggregate seharusnya tidak thread-safe — itu tanggung jawab repository/transaction layer).



## 📦 Modul 6.5 — Repository Pattern dalam DDD

Repository di DDD berbeda konsepnya dengan repository di Fase 3. Di DDD, repository adalah **abstraksi untuk collection of aggregates** — bukan sekadar CRUD wrapper.

### Prinsip Repository di DDD

```
Repository di DDD:
├── Hanya ada untuk Aggregate Root (bukan untuk setiap entity!)
├── Interface ada di DOMAIN layer (bukan infrastructure)
├── Bekerja dengan Aggregate, bukan ORM model
├── Menyembunyikan semua detail persistensi dari domain
└── Menyediakan koleksi-like API (seperti in-memory collection)
```

### Interface Repository yang Benar

```go
// internal/domain/order/repository/order_repository.go
// PERHATIAN: ini ada di domain layer, bukan infrastructure!
package repository

import (
    "context"
    "github.com/kamu/order-service/internal/domain/order/aggregate"
    "github.com/kamu/order-service/internal/domain/order/entity"
)

// OrderCommandRepository untuk operasi write — bekerja dengan aggregate
type OrderCommandRepository interface {
    // Save menyimpan atau mengupdate Order aggregate (insert or update)
    // Harus atomic: jika ada domain events di outbox, disimpan dalam transaksi yang sama
    Save(ctx context.Context, order *aggregate.Order) error

    // FindByID memuat Order aggregate lengkap dari storage
    // Mengembalikan nil, nil jika tidak ditemukan (bukan error)
    FindByID(ctx context.Context, id entity.OrderID) (*aggregate.Order, error)

    // FindByIDForUpdate memuat dengan row-level lock (untuk concurrent scenarios)
    FindByIDForUpdate(ctx context.Context, id entity.OrderID) (*aggregate.Order, error)

    // Delete menghapus order (jarang dipakai di DDD — prefer status change)
    Delete(ctx context.Context, id entity.OrderID) error
}

// OrderQueryRepository untuk operasi read — return view model/DTO, BUKAN aggregate
// (ini adalah sisi Q dari CQRS)
type OrderQueryRepository interface {
    // GetOrderList mengembalikan daftar order dengan pagination dan filter
    GetOrderList(ctx context.Context, opts OrderListQuery) (*OrderListResult, error)

    // GetOrderDetail mengembalikan detail order yang sudah di-join dengan data lain
    GetOrderDetail(ctx context.Context, id uint64) (*OrderDetailView, error)

    // GetOrdersByCustomer mengembalikan order milik customer
    GetOrdersByCustomer(ctx context.Context, customerID uint64, opts PaginationOpts) (*OrderListResult, error)

    // GetOrderStats mengembalikan statistik aggregate (revenue, count, dll)
    GetOrderStats(ctx context.Context, opts StatsQuery) (*OrderStats, error)
}

// Query options dan view models
type OrderListQuery struct {
    CustomerID *uint64
    Status     *string
    DateFrom   *time.Time
    DateTo     *time.Time
    Page       int
    PerPage    int
    OrderBy    string
    OrderDir   string
}

type OrderListResult struct {
    Orders     []*OrderListItem
    Total      int64
    Page       int
    PerPage    int
    TotalPages int
}

type OrderListItem struct {
    ID           uint64
    Status       string
    TotalAmount  float64
    Currency     string
    ItemCount    int
    CustomerID   uint64
    CustomerName string    // join dari customer service
    CreatedAt    time.Time
    ConfirmedAt  *time.Time
}

type OrderDetailView struct {
    ID              uint64
    Status          string
    Items           []*OrderItemView
    ShippingAddress string
    TotalAmount     float64
    Currency        string
    Notes           string
    CreatedAt       time.Time
    ConfirmedAt     *time.Time
    CancelledAt     *time.Time
    CancelReason    string
    TrackingNumber  string
}

type OrderItemView struct {
    ProductID   uint64
    ProductName string
    Quantity    int
    UnitPrice   float64
    Subtotal    float64
}
```

### Implementasi PostgreSQL Repository

```go
// internal/infrastructure/persistence/postgres/order_command_repo.go
package postgres

import (
    "context"
    "errors"
    "fmt"

    "gorm.io/gorm"
    "gorm.io/gorm/clause"

    "github.com/kamu/order-service/internal/domain/order/aggregate"
    "github.com/kamu/order-service/internal/domain/order/entity"
    domainRepo "github.com/kamu/order-service/internal/domain/order/repository"
    "github.com/kamu/order-service/internal/infrastructure/persistence/postgres/mapper"
    "github.com/kamu/order-service/internal/infrastructure/persistence/postgres/model"
)

type orderCommandRepository struct {
    db *gorm.DB
}

func NewOrderCommandRepository(db *gorm.DB) domainRepo.OrderCommandRepository {
    return &orderCommandRepository{db: db}
}

func (r *orderCommandRepository) Save(ctx context.Context, order *aggregate.Order) error {
    // Konversi domain aggregate → GORM model
    orderModel := mapper.ToOrderModel(order)

    // Gunakan transaction untuk menyimpan order + domain events (outbox)
    return r.db.WithContext(ctx).Transaction(func(tx *gorm.DB) error {
        // Upsert order
        result := tx.Clauses(clause.OnConflict{
            Columns:   []clause.Column{{Name: "id"}},
            DoUpdates: clause.AssignmentColumns([]string{
                "status", "notes", "shipping_street", "shipping_city",
                "shipping_province", "shipping_postal", "shipping_country",
                "total_amount", "confirmed_at", "cancelled_at",
                "cancel_reason", "tracking_number", "updated_at",
            }),
        }).Create(orderModel)

        if result.Error != nil {
            return fmt.Errorf("save order: %w", result.Error)
        }

        // Simpan items (hapus semua lama, insert baru)
        if order.ID().Value() > 0 {
            if err := tx.Where("order_id = ?", order.ID().Value()).Delete(&model.OrderItemModel{}).Error; err != nil {
                return fmt.Errorf("delete old items: %w", err)
            }
        }

        for _, item := range order.Items() {
            itemModel := mapper.ToOrderItemModel(order.ID().Value(), item)
            if err := tx.Create(itemModel).Error; err != nil {
                return fmt.Errorf("save item: %w", err)
            }
        }

        // Simpan domain events ke outbox table
        for _, event := range order.DomainEvents() {
            outboxEntry := mapper.ToOutboxModel(event)
            if err := tx.Create(outboxEntry).Error; err != nil {
                return fmt.Errorf("save outbox event: %w", err)
            }
        }

        return nil
    })
}

func (r *orderCommandRepository) FindByID(ctx context.Context, id entity.OrderID) (*aggregate.Order, error) {
    var orderModel model.OrderModel
    result := r.db.WithContext(ctx).
        Preload("Items").
        First(&orderModel, id.Value())

    if result.Error != nil {
        if errors.Is(result.Error, gorm.ErrRecordNotFound) {
            return nil, nil // tidak ditemukan, bukan error
        }
        return nil, fmt.Errorf("find order by id: %w", result.Error)
    }

    // Konversi GORM model → domain aggregate
    return mapper.ToOrderAggregate(&orderModel)
}

func (r *orderCommandRepository) FindByIDForUpdate(ctx context.Context, id entity.OrderID) (*aggregate.Order, error) {
    var orderModel model.OrderModel
    result := r.db.WithContext(ctx).
        Clauses(clause.Locking{Strength: "UPDATE"}).
        Preload("Items").
        First(&orderModel, id.Value())

    if result.Error != nil {
        if errors.Is(result.Error, gorm.ErrRecordNotFound) {
            return nil, nil
        }
        return nil, fmt.Errorf("find order for update: %w", result.Error)
    }

    return mapper.ToOrderAggregate(&orderModel)
}
```

### Mapper: Konversi Aggregate ↔ Model

```go
// internal/infrastructure/persistence/postgres/mapper/order_mapper.go
package mapper

import (
    "github.com/kamu/order-service/internal/domain/order/aggregate"
    "github.com/kamu/order-service/internal/domain/order/entity"
    "github.com/kamu/order-service/internal/domain/order/valueobject"
    "github.com/kamu/order-service/internal/infrastructure/persistence/postgres/model"
)

// ToOrderModel konversi aggregate → GORM model untuk disimpan ke DB
func ToOrderModel(order *aggregate.Order) *model.OrderModel {
    addr := order.ShippingAddress()
    return &model.OrderModel{
        ID:               order.ID().Value(),
        CustomerID:       order.CustomerID(),
        Status:           string(order.Status()),
        Notes:            order.Notes(),
        ShippingStreet:   addr.Street(),
        ShippingCity:     addr.City(),
        ShippingProvince: addr.Province(),
        ShippingPostal:   addr.PostalCode(),
        ShippingCountry:  addr.Country(),
        TotalAmount:      order.TotalAmount().Amount(),
        Currency:         order.TotalAmount().Currency(),
        ConfirmedAt:      order.ConfirmedAt(),
        CancelledAt:      order.CancelledAt(),
        CreatedAt:        order.CreatedAt(),
        UpdatedAt:        order.UpdatedAt(),
    }
}

// ToOrderAggregate konversi GORM model → domain aggregate untuk diproses
func ToOrderAggregate(m *model.OrderModel) (*aggregate.Order, error) {
    // Rekonstruksi value objects
    addr, err := valueobject.NewAddress(
        m.ShippingStreet, m.ShippingCity,
        m.ShippingProvince, m.ShippingPostal, m.ShippingCountry,
    )
    if err != nil {
        return nil, err
    }

    totalAmount, err := valueobject.NewMoney(m.TotalAmount, m.Currency)
    if err != nil {
        return nil, err
    }

    // Rekonstruksi items
    items := make([]aggregate.OrderItem, len(m.Items))
    for i, item := range m.Items {
        unitPrice, _ := valueobject.NewMoney(item.UnitPrice, item.Currency)
        items[i] = aggregate.NewOrderItemFromModel(
            entity.OrderItemID(item.ID),
            item.ProductID,
            item.ProductName,
            item.Quantity,
            unitPrice,
        )
    }

    // Rekonstruksi aggregate — pakai factory reconstitution (bukan NewOrder!)
    return aggregate.ReconstitutOrder(
        entity.OrderID(m.ID),
        m.CustomerID,
        items,
        aggregate.OrderStatus(m.Status),
        addr,
        totalAmount,
        m.Notes,
        m.CreatedAt,
        m.UpdatedAt,
        m.ConfirmedAt,
        m.CancelledAt,
    ), nil
}
```

### 🏋️ Latihan 6.5

1. Implementasikan `OrderCommandRepository` lengkap dengan PostgreSQL (GORM). Pastikan `Save` bekerja untuk: (a) insert order baru, (b) update order yang sudah ada. Tulis integration test untuk keduanya.
2. Buat `InMemoryOrderRepository` yang mengimplementasikan interface yang sama untuk dipakai di unit test. Pastikan thread-safe dengan `sync.RWMutex`.
3. Implementasikan `OrderQueryRepository` dengan metode `GetOrderList` yang support semua filter di `OrderListQuery`. Gunakan query builder GORM secara dinamis.


## 📦 Modul 6.6 — Domain Events

Domain Events merepresentasikan **sesuatu yang terjadi di domain** yang relevan untuk sistem atau service lain.

```go
// internal/domain/order/event/events.go
package event

import (
    "time"
    "github.com/kamu/order-service/internal/domain/order/aggregate"
)

// DomainEvent adalah interface yang harus diimplementasikan semua event
type DomainEvent interface {
    EventName() string
    OccurredAt() time.Time
    AggregateID() string
}

// ===== Base event =====
type baseEvent struct {
    eventName   string
    occurredAt  time.Time
    aggregateID string
}

func (e baseEvent) EventName() string    { return e.eventName }
func (e baseEvent) OccurredAt() time.Time { return e.occurredAt }
func (e baseEvent) AggregateID() string  { return e.aggregateID }

// ===== Order Events =====

type OrderCreated struct {
    baseEvent
    CustomerID      uint64
    ShippingAddress string
}

func NewOrderCreated(order *aggregate.Order) OrderCreated {
    return OrderCreated{
        baseEvent: baseEvent{
            eventName:   "order.created",
            occurredAt:  time.Now(),
            aggregateID: order.ID().String(),
        },
        CustomerID:      order.CustomerID(),
        ShippingAddress: order.ShippingAddress().FullAddress(),
    }
}

type OrderSubmitted struct {
    baseEvent
    CustomerID  uint64
    TotalAmount float64
    ItemCount   int
}

func NewOrderSubmitted(order *aggregate.Order) OrderSubmitted {
    return OrderSubmitted{
        baseEvent: baseEvent{
            eventName:   "order.submitted",
            occurredAt:  time.Now(),
            aggregateID: order.ID().String(),
        },
        CustomerID:  order.CustomerID(),
        TotalAmount: order.TotalAmount().Amount(),
        ItemCount:   len(order.Items()),
    }
}

type OrderConfirmed struct {
    baseEvent
    CustomerID  uint64
    TotalAmount float64
    PaymentID   string
    Items       []OrderItemSnapshot
}

type OrderItemSnapshot struct {
    ProductID   uint64
    ProductName string
    Quantity    int
    UnitPrice   float64
}

func NewOrderConfirmed(order *aggregate.Order, paymentID string) OrderConfirmed {
    items := make([]OrderItemSnapshot, len(order.Items()))
    for i, item := range order.Items() {
        items[i] = OrderItemSnapshot{
            ProductID:   item.ProductID(),
            ProductName: item.ProductName(),
            Quantity:    item.Quantity(),
            UnitPrice:   item.UnitPrice().Amount(),
        }
    }
    return OrderConfirmed{
        baseEvent: baseEvent{
            eventName:   "order.confirmed",
            occurredAt:  time.Now(),
            aggregateID: order.ID().String(),
        },
        CustomerID:  order.CustomerID(),
        TotalAmount: order.TotalAmount().Amount(),
        PaymentID:   paymentID,
        Items:       items,
    }
}

type OrderCancelled struct {
    baseEvent
    CustomerID      uint64
    Reason          string
    StockToRestore  bool  // true jika perlu kembalikan stok
    Items           []OrderItemSnapshot
}

func NewOrderCancelled(order *aggregate.Order, reason string, stockToRestore bool) OrderCancelled {
    items := make([]OrderItemSnapshot, len(order.Items()))
    for i, item := range order.Items() {
        items[i] = OrderItemSnapshot{
            ProductID: item.ProductID(),
            Quantity:  item.Quantity(),
        }
    }
    return OrderCancelled{
        baseEvent: baseEvent{
            eventName:   "order.cancelled",
            occurredAt:  time.Now(),
            aggregateID: order.ID().String(),
        },
        CustomerID:     order.CustomerID(),
        Reason:         reason,
        StockToRestore: stockToRestore,
        Items:          items,
    }
}

type OrderDispatched struct {
    baseEvent
    TrackingNumber string
}

func NewOrderDispatched(order *aggregate.Order, trackingNumber string) OrderDispatched {
    return OrderDispatched{
        baseEvent: baseEvent{
            eventName:   "order.dispatched",
            occurredAt:  time.Now(),
            aggregateID: order.ID().String(),
        },
        TrackingNumber: trackingNumber,
    }
}

type OrderDelivered struct {
    baseEvent
}

func NewOrderDelivered(order *aggregate.Order) OrderDelivered {
    return OrderDelivered{
        baseEvent: baseEvent{
            eventName:   "order.delivered",
            occurredAt:  time.Now(),
            aggregateID: order.ID().String(),
        },
    }
}
```

### Event Publisher

```go
// internal/domain/order/event/publisher.go
package event

import "context"

// EventPublisher adalah interface untuk publish domain events
// Implementasinya bisa ke in-memory, Kafka, RabbitMQ, dll
type EventPublisher interface {
    Publish(ctx context.Context, events ...DomainEvent) error
}

// EventHandler adalah interface untuk menangani domain events
type EventHandler interface {
    Handle(ctx context.Context, event DomainEvent) error
    EventName() string
}
```

---



### 🏋️ Latihan 6.6

1. Buat `InMemoryEventPublisher` yang menyimpan semua events yang dipublish ke slice. Gunakan di unit test untuk memverifikasi bahwa event yang tepat di-raise saat operasi domain dilakukan.
2. Implementasikan **Outbox Pattern**: saat `OrderCommandRepository.Save()` dipanggil, simpan domain events ke tabel `order_outbox` dalam transaksi yang sama. Buat `OutboxProcessor` yang secara periodic membaca events yang belum dikirim dan mempublishnya.
3. Desain event schema evolution: `OrderConfirmed` v1 punya field `payment_id: string`. Di v2 perlu tambah `payment_method: string` dan `payment_amount: float64`. Tulis migration strategy yang backward compatible.


## 📦 Modul 6.7 — Domain Services

Domain Service berisi **business logic yang tidak secara natural milik Entity atau Value Object** manapun.

```go
// internal/domain/order/service/pricing_service.go
package service

import (
    "context"
    "github.com/kamu/order-service/internal/domain/order/valueobject"
)

// PricingService menghitung harga order dengan mempertimbangkan
// diskon, voucher, dan ongkir — logika ini tidak milik Order maupun Customer
type PricingService interface {
    CalculateDiscount(
        ctx context.Context,
        customerID uint64,
        items []OrderItemPricing,
        voucherCode string,
    ) (discount valueobject.Money, err error)

    CalculateShippingCost(
        ctx context.Context,
        destination valueobject.Address,
        totalWeight float64,
        courier string,
    ) (cost valueobject.Money, err error)
}

type OrderItemPricing struct {
    ProductID uint64
    Quantity  int
    UnitPrice valueobject.Money
}

// StockValidationService memvalidasi ketersediaan stok
// Dia perlu komunikasi ke Product service — bukan tanggung jawab Order entity
type StockValidationService interface {
    ValidateStock(
        ctx context.Context,
        items []StockCheckItem,
    ) error
}

type StockCheckItem struct {
    ProductID uint64
    Quantity  int
}
```

---



### 🏋️ Latihan 6.7

1. Implementasikan `PricingDomainService` yang menghitung total order dengan mempertimbangkan: (a) base price dari items, (b) volume discount (>10 items: 5% off), (c) voucher code (fixed amount atau percentage), (d) shipping cost. Tulis unit test dengan berbagai kombinasi.
2. Identifikasi mengapa logic berikut TIDAK seharusnya ada di entity Order, dan jelaskan bahwa itu adalah tanggung jawab Domain Service: "Saat total order melebihi Rp 1.000.000, otomatis upgrade shipping ke express".


## 📦 Modul 6.8 — Application Services

Application Service **mengorkestrasikan** use case. Tidak mengandung business logic, tapi mengkoordinasikan domain objects.

```go
// internal/application/usecase/create_order.go
package usecase

import (
    "context"
    "fmt"

    "github.com/kamu/order-service/internal/domain/order/aggregate"
    "github.com/kamu/order-service/internal/domain/order/event"
    "github.com/kamu/order-service/internal/domain/order/service"
    "github.com/kamu/order-service/internal/domain/order/valueobject"
    "github.com/kamu/order-service/internal/domain/repository"
)

type CreateOrderInput struct {
    CustomerID      uint64
    Items           []CreateOrderItemInput
    ShippingAddress valueobject.Address
    VoucherCode     string
    CourierService  string
}

type CreateOrderItemInput struct {
    ProductID   uint64
    ProductName string
    Quantity    int
    UnitPrice   float64
}

type CreateOrderOutput struct {
    Order *aggregate.Order
}

type CreateOrderUseCase interface {
    Execute(ctx context.Context, input CreateOrderInput) (*CreateOrderOutput, error)
}

type createOrderUseCase struct {
    orderRepo       repository.OrderRepository
    pricingService  service.PricingService
    stockService    service.StockValidationService
    eventPublisher  event.EventPublisher
}

func NewCreateOrderUseCase(
    orderRepo repository.OrderRepository,
    pricingService service.PricingService,
    stockService service.StockValidationService,
    eventPublisher event.EventPublisher,
) CreateOrderUseCase {
    return &createOrderUseCase{
        orderRepo:      orderRepo,
        pricingService: pricingService,
        stockService:   stockService,
        eventPublisher: eventPublisher,
    }
}

func (uc *createOrderUseCase) Execute(ctx context.Context, input CreateOrderInput) (*CreateOrderOutput, error) {
    // Step 1: Validasi ketersediaan stok (domain service)
    stockItems := make([]service.StockCheckItem, len(input.Items))
    for i, item := range input.Items {
        stockItems[i] = service.StockCheckItem{
            ProductID: item.ProductID,
            Quantity:  item.Quantity,
        }
    }
    if err := uc.stockService.ValidateStock(ctx, stockItems); err != nil {
        return nil, fmt.Errorf("create order: validate stock: %w", err)
    }

    // Step 2: Buat aggregate Order (factory method)
    order, err := aggregate.NewOrder(input.CustomerID, input.ShippingAddress)
    if err != nil {
        return nil, fmt.Errorf("create order: new order: %w", err)
    }

    // Step 3: Tambah items ke order (business method di aggregate)
    for _, item := range input.Items {
        unitPrice, err := valueobject.NewMoney(item.UnitPrice, "IDR")
        if err != nil {
            return nil, fmt.Errorf("create order: invalid price: %w", err)
        }
        if err := order.AddItem(item.ProductID, item.ProductName, item.Quantity, unitPrice); err != nil {
            return nil, fmt.Errorf("create order: add item: %w", err)
        }
    }

    // Step 4: Submit order (transisi state di aggregate)
    if err := order.Submit(); err != nil {
        return nil, fmt.Errorf("create order: submit: %w", err)
    }

    // Step 5: Simpan order ke repository
    if err := uc.orderRepo.Save(ctx, order); err != nil {
        return nil, fmt.Errorf("create order: save: %w", err)
    }

    // Step 6: Publish domain events
    if err := uc.eventPublisher.Publish(ctx, order.DomainEvents()...); err != nil {
        // Log error tapi jangan fail — event bisa di-retry
        // Atau gunakan outbox pattern untuk guaranteed delivery
    }
    order.ClearDomainEvents()

    return &CreateOrderOutput{Order: order}, nil
}
```

---



### 🏋️ Latihan 6.8

1. Implementasikan `CancelOrderUseCase` lengkap: load order → validate state → cancel dengan reason → save → publish events. Tulis unit test dengan mock semua dependencies. Pastikan: (a) order tidak ditemukan → NotFound error, (b) order sudah delivered → domain error, (c) sukses → OrderCancelled event di-publish.
2. Refactor `CreateOrderUseCase` untuk menggunakan **transaction script pattern** di application service: tambahkan rollback jika event publishing gagal (atau gunakan outbox untuk eventual consistency).


## 📦 Modul 6.9 — Bounded Context

```
E-Commerce System — Bounded Contexts:
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  ORDER CONTEXT  │    │ PRODUCT CONTEXT │    │  USER CONTEXT   │
│                 │    │                 │    │                 │
│  Order          │    │  Product        │    │  Customer       │
│  OrderItem      │    │  Category       │    │  Address        │
│  Payment        │    │  Inventory      │    │  Authentication │
│                 │    │                 │    │                 │
│  "Order"=       │    │  "Product"=     │    │  "User"=        │
│  confirmed,     │    │  catalog item   │    │  shopper with   │
│  paid order     │    │  with stock     │    │  profile        │
└────────┬────────┘    └────────┬────────┘    └────────┬────────┘
         │                      │                      │
         └──────────────────────┴──────────────────────┘
                    Context Map (via events / API)
```

### Catatan Penting
Kata yang sama bisa **berarti berbeda** di bounded context yang berbeda:
- **"Product"** di Order Context = snapshot harga saat order dibuat (immutable)
- **"Product"** di Catalog Context = item katalog dengan harga yang bisa berubah
- **"User"** di Auth Context = akun dengan credentials
- **"Customer"** di Order Context = entitas yang bisa beli dengan shipping address

---



### 🏋️ Latihan 6.9

1. Gambar **Context Map** yang lengkap untuk sistem e-commerce dengan minimal 4 bounded context: Order, Product, User/Auth, Payment. Identifikasi: (a) upstream/downstream relationships, (b) integration patterns (Conformist, ACL, Partnership, Shared Kernel).
2. Identifikasi **shared kernel** antara Order Context dan Inventory Context. Buat package `shared` yang berisi model yang dipakai bersama, dengan aturan: perubahan harus disetujui kedua tim.


## 📦 Modul 6.10 — CQRS Pattern

**Command Query Responsibility Segregation** — pisahkan operasi tulis (Command) dari baca (Query).

```go
// Tanpa CQRS (satu model untuk semua):
type OrderRepository interface {
    Save(ctx context.Context, order *aggregate.Order) error
    FindByID(ctx context.Context, id entity.OrderID) (*aggregate.Order, error)
    FindAllByCustomer(ctx context.Context, customerID uint64) ([]*aggregate.Order, error)
    GetOrderSummaryReport(ctx context.Context) (*Report, error) // Tidak natural di domain model
}

// Dengan CQRS — Commands (Write side) menggunakan domain aggregate
type OrderCommandRepository interface {
    Save(ctx context.Context, order *aggregate.Order) error
    FindByID(ctx context.Context, id entity.OrderID) (*aggregate.Order, error)
}

// Queries (Read side) menggunakan Read Models yang dioptimasi untuk tampilan
type OrderQueryRepository interface {
    GetOrderList(ctx context.Context, opts OrderListQuery) ([]*OrderListItem, error)
    GetOrderDetail(ctx context.Context, id uint64) (*OrderDetailView, error)
    GetOrderSummary(ctx context.Context, customerID uint64) (*OrderSummaryView, error)
    GetSalesReport(ctx context.Context, from, to time.Time) (*SalesReport, error)
}

// Read models dioptimasi untuk query — bisa denormalized
type OrderListItem struct {
    ID            uint64
    Status        string
    TotalAmount   float64
    ItemCount     int
    CreatedAt     time.Time
    CustomerName  string  // join dari customer table — tidak ada di aggregate
}

type OrderDetailView struct {
    ID              uint64
    Status          string
    Items           []OrderItemView
    ShippingAddress string
    TotalAmount     float64
    CreatedAt       time.Time
    ConfirmedAt     *time.Time
    Customer        CustomerView
    Payment         *PaymentView
}
```

---



### 🏋️ Latihan 6.10

1. Implementasikan CQRS di Order Service: buat `OrderSummaryProjection` yang mendengarkan domain events dan memperbarui read model di tabel `order_summaries` (denormalized). Verifikasi bahwa query via `OrderQueryRepository` return data yang up-to-date setelah events diproses.
2. Buat `OrderDashboardQuery` yang mengembalikan: total revenue hari ini, jumlah order per status, top 5 produk terlaris. Implementasikan sebagai query langsung ke read model (bukan via aggregate).



## 📦 Modul 6.11 — Event Sourcing (Pengenalan)

Event Sourcing adalah pola dimana **state aplikasi tidak disimpan sebagai nilai saat ini, melainkan sebagai urutan events** yang menghasilkan state tersebut.

### Konsep Dasar

```
Traditional (State Sourcing):        Event Sourcing:
┌─────────────────────┐              ┌─────────────────────────────────────┐
│ Order Table          │              │ Order Events Table                   │
│ id: 1               │              │ 1. OrderCreated   (t=10:00)          │
│ status: CONFIRMED   │    vs        │ 2. ItemAdded      (t=10:01)          │
│ total: 250000       │              │ 3. ItemAdded      (t=10:02)          │
│ items: [...]        │              │ 4. OrderSubmitted (t=10:05)          │
└─────────────────────┘              │ 5. OrderConfirmed (t=10:30)          │
                                     └─────────────────────────────────────┘
                                     State saat ini = replay semua events
```

### Keuntungan Event Sourcing

```
✅ Audit log gratis — semua perubahan tercatat
✅ Bisa "time travel" — lihat state di waktu tertentu
✅ Debugging lebih mudah — "kenapa order ini cancelled?"
✅ Event-driven architecture natural — events sudah tersimpan
✅ Read models bisa di-rebuild kapan saja dari event history

❌ Querying lebih kompleks (butuh read model/projection)
❌ Event schema evolusi yang hati-hati
❌ Eventual consistency di read models
❌ Lebih complex untuk implement
```

### Implementasi Sederhana

```go
// internal/domain/order/event/store.go
package event

import (
    "context"
    "encoding/json"
    "fmt"
    "time"
)

// StoredEvent adalah event yang disimpan ke database
type StoredEvent struct {
    ID          uint64          `gorm:"primarykey"`
    AggregateID string          `gorm:"index;not null"`
    EventType   string          `gorm:"not null"`
    Version     int             `gorm:"not null"` // optimistic concurrency
    Payload     json.RawMessage `gorm:"type:jsonb;not null"`
    OccurredAt  time.Time       `gorm:"not null"`
}

// EventStore adalah interface untuk menyimpan dan memuat events
type EventStore interface {
    // AppendEvents menyimpan events baru untuk aggregate
    // version adalah expected current version (optimistic locking)
    AppendEvents(ctx context.Context, aggregateID string, events []DomainEvent, expectedVersion int) error

    // LoadEvents memuat semua events untuk aggregate
    LoadEvents(ctx context.Context, aggregateID string) ([]*StoredEvent, error)

    // LoadEventsFrom memuat events dari version tertentu
    LoadEventsFrom(ctx context.Context, aggregateID string, fromVersion int) ([]*StoredEvent, error)
}
```

```go
// Aggregate yang di-reconstruct dari events
type OrderES struct {
    id      entity.OrderID
    status  OrderStatus
    items   []OrderItem
    version int // track event version untuk optimistic locking
    events  []DomainEvent // uncommitted events
}

// Apply menerapkan event ke state (dipakai saat replay)
func (o *OrderES) Apply(event DomainEvent) {
    switch e := event.(type) {
    case OrderCreated:
        o.id = entity.OrderID(e.OrderID)
        o.status = StatusDraft
        o.version++

    case ItemAdded:
        o.items = append(o.items, OrderItem{
            ProductID: e.ProductID,
            Quantity:  e.Quantity,
        })
        o.version++

    case OrderSubmitted:
        o.status = StatusPending
        o.version++

    case OrderConfirmed:
        o.status = StatusConfirmed
        o.version++

    case OrderCancelled:
        o.status = StatusCancelled
        o.version++
    }
}

// ReconstitutFromEvents membangun state dari list events
func ReconstitutOrderFromEvents(events []DomainEvent) *OrderES {
    order := &OrderES{}
    for _, event := range events {
        order.Apply(event)
    }
    order.events = nil // clear uncommitted events
    return order
}

// Raise mencatat event baru DAN langsung menerapkannya ke state
func (o *OrderES) Raise(event DomainEvent) {
    o.Apply(event)      // update state sekarang
    o.events = append(o.events, event) // simpan untuk di-persist nanti
}
```

```go
// Penggunaan: repository event sourcing
type ESOrderRepository struct {
    store event.EventStore
}

func (r *ESOrderRepository) FindByID(ctx context.Context, id entity.OrderID) (*OrderES, error) {
    // Load semua events dari store
    storedEvents, err := r.store.LoadEvents(ctx, id.String())
    if err != nil {
        return nil, err
    }
    if len(storedEvents) == 0 {
        return nil, nil
    }

    // Deserialize events
    var domainEvents []event.DomainEvent
    for _, se := range storedEvents {
        e, err := deserializeEvent(se.EventType, se.Payload)
        if err != nil {
            return nil, err
        }
        domainEvents = append(domainEvents, e)
    }

    // Reconstruct aggregate dari events
    return ReconstitutOrderFromEvents(domainEvents), nil
}

func (r *ESOrderRepository) Save(ctx context.Context, order *OrderES) error {
    uncommitted := order.UncommittedEvents()
    if len(uncommitted) == 0 {
        return nil
    }

    // Append events baru dengan optimistic locking
    return r.store.AppendEvents(ctx,
        order.id.String(),
        uncommitted,
        order.version - len(uncommitted), // expected version sebelum events baru
    )
}
```

### Kapan Pakai Event Sourcing?

```
✅ COCOK untuk:
- Domain dengan audit trail yang ketat (keuangan, healthcare, hukum)
- Sistem yang butuh "undo" atau history
- Event-driven microservices dengan banyak subscribers
- Domain yang kompleks dan sering berubah state

❌ TIDAK COCOK untuk:
- CRUD sederhana tanpa audit requirement
- Tim kecil yang baru mulai dengan DDD
- Deadline ketat — ES menambah kompleksitas signifikan
- Ketika traditional state sourcing sudah cukup
```

### 🏋️ Latihan 6.11

1. Buat `InMemoryEventStore` yang mengimplementasikan `EventStore` interface. Simpan events di `map[string][]StoredEvent`. Implement optimistic locking: jika `expectedVersion` tidak match dengan jumlah events yang ada, return `ErrConcurrencyConflict`.
2. Buat `OrderProjection` yang mendengarkan events dari store dan membangun read model `map[uint64]*OrderSummary`. `OrderSummary` berisi: ID, Status, TotalItems, TotalAmount. Test bahwa setelah replay semua events, read model konsisten dengan state aggregate.
3. Implementasikan fungsi `ReplayOrderAt(store EventStore, orderID string, at time.Time) (*OrderES, error)` yang membangun state order pada waktu tertentu (time travel).



## 📦 Modul 6.12 — Anti-Corruption Layer (ACL)

Anti-Corruption Layer (ACL) adalah pola untuk **melindungi domain kamu dari "polusi" model domain sistem lain**.

### Masalah Tanpa ACL

```go
// ❌ TANPA ACL: Order domain terkontaminasi dengan model dari Product Service

// Model dari Product Service gRPC (generated code)
type ProductResponse struct {
    ProductId   uint64  `json:"product_id"`     // naming tidak konsisten
    ProductName string  `json:"product_name"`   // dengan domain kita
    ListPrice   float64 `json:"list_price"`
    AvailQty    int     `json:"avail_qty"`
}

// Order domain langsung pakai model luar!
type OrderItem struct {
    Product ProductResponse // POLUSI! Domain order tahu tentang model product external
    Qty     int
}
```

### ACL yang Benar

```go
// ✅ DENGAN ACL: Domain terlindungi

// internal/domain/order/aggregate/order_item.go
// Domain mendefinisikan model internalnya sendiri
type OrderItem struct {
    productID   uint64       // hanya ID, bukan object penuh
    productName string       // snapshot nama saat order dibuat
    unitPrice   valueobject.Money
    quantity    int
}

// ======================================================
// Anti-Corruption Layer — ada di infrastructure layer
// internal/infrastructure/acl/product_acl.go
// ======================================================

// ProductInfo adalah representasi internal dari data product
// Yang penting untuk Order domain — bukan semua field Product
type ProductInfo struct {
    ID       uint64
    Name     string
    Price    valueobject.Money
    InStock  bool
    Stock    int
}

// ProductCatalogACL adalah ACL yang menerjemahkan model external ke domain internal
type ProductCatalogACL struct {
    grpcClient productv1.ProductServiceClient // external system
}

func NewProductCatalogACL(client productv1.ProductServiceClient) *ProductCatalogACL {
    return &ProductCatalogACL{grpcClient: client}
}

// GetProductInfo mengambil info product dan menerjemahkan ke model domain internal
func (acl *ProductCatalogACL) GetProductInfo(ctx context.Context, productID uint64) (*ProductInfo, error) {
    // Panggil external service
    resp, err := acl.grpcClient.GetProduct(ctx, &productv1.GetProductRequest{Id: productID})
    if err != nil {
        // Terjemahkan gRPC error ke domain error
        st, _ := status.FromError(err)
        switch st.Code() {
        case codes.NotFound:
            return nil, apperror.ErrProductNotFound
        default:
            return nil, fmt.Errorf("get product info: %w", err)
        }
    }

    // Terjemahkan proto message → domain model
    price, err := valueobject.NewMoney(resp.Product.Price, "IDR")
    if err != nil {
        return nil, fmt.Errorf("invalid price: %w", err)
    }

    return &ProductInfo{
        ID:      resp.Product.Id,
        Name:    resp.Product.Name,
        Price:   price,
        InStock: resp.Product.Stock > 0,
        Stock:   int(resp.Product.Stock),
    }, nil
}

// CheckAndReserveStock mengecek stok dan memintanya direservasi
func (acl *ProductCatalogACL) CheckAndReserveStock(
    ctx context.Context,
    items []StockCheckItem,
) error {
    for _, item := range items {
        resp, err := acl.grpcClient.CheckStock(ctx, &productv1.CheckStockRequest{
            ProductId: item.ProductID,
            Quantity:  uint32(item.Quantity),
        })
        if err != nil {
            return fmt.Errorf("check stock for product %d: %w", item.ProductID, err)
        }
        if !resp.Available {
            return apperror.NewInsufficientStockError(item.ProductID, item.Quantity, int(resp.CurrentStock))
        }
    }
    return nil
}
```

```go
// Domain Service menggunakan ACL melalui interface
// (domain tidak tahu bahwa ini komunikasi ke gRPC service!)

// internal/domain/order/service/stock_validation_service.go
type StockValidationService interface {
    ValidateStock(ctx context.Context, items []StockCheckItem) error
}

// internal/domain/order/service/product_info_service.go
type ProductInfoService interface {
    GetProductInfo(ctx context.Context, productID uint64) (*ProductInfo, error)
}

// Di use case — inject interface, bukan concrete ACL
type createOrderUseCase struct {
    orderRepo      repository.OrderCommandRepository
    stockValidator service.StockValidationService  // ← interface!
    productInfo    service.ProductInfoService       // ← interface!
    eventPublisher event.EventPublisher
}
```

```go
// main.go — wire ACL sebagai implementasi domain service interface
productGRPCClient := productv1.NewProductServiceClient(conn)
productACL := acl.NewProductCatalogACL(productGRPCClient)

// ProductCatalogACL mengimplementasikan KEDUA interface domain service!
createOrderUC := usecase.NewCreateOrderUseCase(
    orderRepo,
    productACL,   // implements StockValidationService
    productACL,   // implements ProductInfoService
    eventPublisher,
)
```

### Keuntungan ACL

```
Tanpa ACL:              Dengan ACL:
Domain ←────────────── External System    Domain ←── ACL ←── External System
(terkontaminasi)       (berubah-ubah)     (terlindungi)      (berubah-ubah)

Jika External System berubah:            Jika External System berubah:
→ Domain harus diubah                    → Hanya ACL yang diubah
→ Use case harus diubah                  → Domain dan Use case TIDAK berubah
→ Test harus diubah                      → Test hanya perlu update mock
```

### 🏋️ Latihan 6.12

1. Buat `UserContextACL` yang menerjemahkan response dari User Auth Service (JWT claims) ke `CustomerInfo{ID, Name, Email, Tier}` yang dibutuhkan oleh Order domain. Tambahkan logika: jika tier customer adalah "premium", beri diskon 5%.
2. Buat integration test untuk ACL menggunakan **mock gRPC server**: jalankan gRPC server test in-process menggunakan `bufconn`, test bahwa ACL menerjemahkan response dengan benar ke domain model.
3. Implementasikan **Facade pattern** di atas beberapa ACL: buat `ExternalServicesACL` yang mengkompilasikan `ProductCatalogACL`, `UserContextACL`, dan `PaymentGatewayACL` menjadi satu struct yang mudah di-inject ke use cases.


## 🎯 Review & Checkpoint Fase 6

### Konseptual
- [ ] Apa perbedaan Entity dan Value Object di DDD?
- [ ] Apa itu Aggregate Root dan mengapa penting?
- [ ] Kapan menggunakan Domain Service vs Application Service?
- [ ] Apa itu Domain Event dan bagaimana cara publikasinya?
- [ ] Apa itu Bounded Context dan mengapa diperlukan?
- [ ] Apa keuntungan CQRS?

### Praktis
- [ ] Bisa membuat Value Object yang immutable dan self-validating
- [ ] Bisa membuat Aggregate yang menjaga invariants
- [ ] Bisa membuat Domain Events yang bermakna
- [ ] Bisa menulis unit test untuk aggregate (tanpa database!)
- [ ] Bisa menjelaskan Bounded Context dari sistem yang ada

---

## 🎯 Project Akhir Fase 6

Kerjakan project **Order Service dengan Full DDD** berdasarkan PRD di file:

**`FASE-6-PRD-Order-Service.md`**

---

*Setelah selesai Fase 6, lanjut ke `FASE-7-Microservices.md` untuk menyatukan semua service!*

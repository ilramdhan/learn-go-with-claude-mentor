# 📘 Fase 5: gRPC + Protocol Buffers

> **Level:** 🟠 Intermediate  
> **Durasi Estimasi:** 3–4 minggu  
> **Prasyarat:** ✅ Selesaikan Fase 4 — User Auth Service

---

## 🎯 Tujuan Fase Ini

- ✅ Memahami kapan dan mengapa menggunakan gRPC vs REST
- ✅ Menulis file .proto untuk mendefinisikan service dan message
- ✅ Implementasikan semua 4 tipe RPC (unary, server stream, client stream, bidi)
- ✅ Menulis interceptors (middleware gRPC)
- ✅ Integrasi gRPC ke Clean Architecture tanpa mengubah Use Case

---

## 🗂️ Isi Folder

```
fase-05-grpc-protobuf/
├── README.md
├── materi/
│   └── FASE-5-gRPC-Protobuf.md      ← 13 modul materi
└── project/
    └── FASE-5-PRD-Product-Catalog.md
```

---

## 📚 Daftar Modul

| # | Modul | Konsep Utama |
|---|-------|-------------|
| 5.1 | Mengapa gRPC? | REST vs gRPC perbandingan mendalam |
| 5.2 | Protocol Buffers | Syntax, field numbers, scalar types |
| 5.3 | Setup Toolchain | protoc, buf CLI, go plugins |
| 5.4 | Unary RPC | Request-Response sederhana |
| 5.5 | Server Streaming | Satu request, banyak response |
| 5.6 | Client Streaming | Banyak request, satu response |
| 5.7 | Bidirectional Streaming | Full-duplex communication |
| 5.8 | Error Handling gRPC | Status codes, rich error details |
| 5.9 | Interceptors | Logging, recovery, auth middleware |
| 5.10 | Authentication | JWT via metadata |
| 5.11 | Reflection & gRPC UI | grpcurl, Postman, Evans |
| 5.12 | gRPC + Clean Architecture | Integrasi tanpa ubah use case |
| 5.13 | gRPC Gateway | Expose gRPC sebagai REST API |

---

## 🛠️ Tools yang Perlu Di-install

```bash
# protoc compiler
brew install protobuf  # macOS
# atau download dari https://github.com/protocolbuffers/protobuf/releases

# buf (modern protobuf tool)
brew install bufbuild/buf/buf

# Go plugins
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

# grpcurl (untuk test)
brew install grpcurl
```

---

## 🎯 Project Akhir: Product Catalog gRPC Service

File: `project/FASE-5-PRD-Product-Catalog.md`

gRPC service untuk katalog produk dengan semua 4 tipe RPC.
Akan menjadi bagian dari microservices di Fase 7.

---

## ✅ Checklist Kelulusan

- [ ] Bisa tulis .proto file dan generate Go code
- [ ] Implementasi unary, server-streaming, client-streaming RPC
- [ ] Interceptor logging dan auth berjalan
- [ ] Test berhasil dengan grpcurl
- [ ] **Menyelesaikan project Product Catalog gRPC Service**

---

## ➡️ Selanjutnya

Setelah selesai Fase 5 → [Fase 6: Domain-Driven Design](../fase-06-ddd/)

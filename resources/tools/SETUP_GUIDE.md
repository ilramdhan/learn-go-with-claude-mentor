# 🛠️ Tools & Setup Guide

Panduan instalasi semua tools yang dibutuhkan selama belajar Go.

---

## 🔧 Tools Wajib (Install Sekarang)

### 1. Go 1.22+

```bash
# Download dari https://go.dev/dl/
# Pilih installer sesuai OS

# Verifikasi
go version  # go version go1.22.x ...
go env GOPATH
```

### 2. VS Code + Extensions

```bash
# Download VS Code: https://code.visualstudio.com/

# Extensions yang harus diinstall:
# - Go (by Google) — IntelliSense, debugging
# - REST Client — test API langsung dari editor
# - Docker — manage containers
# - GitLens — visualisasi git history
# - Error Lens — inline error messages
# - Thunder Client — Postman alternatif
```

Setelah install extension Go:
```
Ctrl+Shift+P → "Go: Install/Update Tools" → Select All → Install
```

### 3. Git

```bash
# Ubuntu/Debian
sudo apt install git

# macOS (via Homebrew)
brew install git

# Konfigurasi
git config --global user.name "Nama Kamu"
git config --global user.email "email@kamu.com"
```

### 4. Docker Desktop

```bash
# Download: https://www.docker.com/products/docker-desktop/

# Verifikasi
docker --version
docker compose version
```

---

## 🔧 Tools Fase 3+ (PostgreSQL)

```bash
# PostgreSQL via Docker (cara termudah)
docker run -d \
  --name postgres-dev \
  --restart unless-stopped \
  -e POSTGRES_USER=devuser \
  -e POSTGRES_PASSWORD=devpassword \
  -e POSTGRES_DB=devdb \
  -p 5432:5432 \
  postgres:16-alpine

# Cek berjalan
docker ps
docker logs postgres-dev

# Connect via psql
docker exec -it postgres-dev psql -U devuser -d devdb
```

**GUI Client (pilih satu):**
- [DBeaver](https://dbeaver.io/) — free, multi-database
- [TablePlus](https://tableplus.com/) — premium, clean UI
- [pgAdmin](https://www.pgadmin.org/) — free, powerful

---

## 🔧 Tools Fase 5 (gRPC)

```bash
# 1. protoc compiler
# macOS
brew install protobuf

# Ubuntu
sudo apt install -y protobuf-compiler

# Verifikasi
protoc --version  # libprotoc 25.x

# 2. buf (modern protobuf tool)
# macOS
brew install bufbuild/buf/buf

# Linux
curl -sSL https://github.com/bufbuild/buf/releases/download/v1.28.1/buf-Linux-x86_64 \
  -o /usr/local/bin/buf && chmod +x /usr/local/bin/buf

# 3. Go plugins (jalankan dari terminal)
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

# Tambahkan GOPATH/bin ke PATH
echo 'export PATH="$PATH:$(go env GOPATH)/bin"' >> ~/.bashrc
source ~/.bashrc

# 4. grpcurl — seperti curl untuk gRPC
# macOS
brew install grpcurl
# atau
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest

# Verifikasi
grpcurl --version
```

---

## 🔧 Tools Fase 7+ (Microservices)

```bash
# Redis
docker run -d \
  --name redis-dev \
  -p 6379:6379 \
  redis:7-alpine

# RabbitMQ (dengan management UI)
docker run -d \
  --name rabbitmq-dev \
  -e RABBITMQ_DEFAULT_USER=user \
  -e RABBITMQ_DEFAULT_PASS=password \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3-management-alpine
# Management UI: http://localhost:15672

# Kafka (via Docker Compose)
# Lihat docker-compose.yml di fase-08-message-broker
```

---

## 🔧 Tools Fase 9+ (Observability)

```bash
# Prometheus + Grafana + Jaeger via Docker Compose
# Lihat docker-compose.yml di fase-09-observability

# Jaeger (distributed tracing)
docker run -d \
  --name jaeger \
  -p 16686:16686 \
  -p 14268:14268 \
  jaegertracing/all-in-one:latest
# UI: http://localhost:16686
```

---

## 🔧 Useful Go CLI Tools

```bash
# golangci-lint — linter dengan banyak rules
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

# air — hot reload saat development
go install github.com/cosmtrek/air@latest

# godoc — generate dan serve dokumentasi
go install golang.org/x/tools/cmd/godoc@latest

# dlv — debugger Go
go install github.com/go-delve/delve/cmd/dlv@latest

# govulncheck — cek vulnerabilities di dependencies
go install golang.org/x/vuln/cmd/govulncheck@latest

# benchstat — compare benchmark results
go install golang.org/x/perf/cmd/benchstat@latest

# mockery — generate mocks dari interface
go install github.com/vektra/mockery/v2@latest
```

---

## 📁 Recommended Workspace Structure

```
~/
└── projects/
    └── go-learning/
        ├── fase-01-cli-todo/          ← Project Fase 1
        ├── fase-02-file-processor/    ← Project Fase 2
        ├── fase-03-blog-api/          ← Project Fase 3
        ├── fase-04-user-auth/         ← Project Fase 4
        ├── fase-05-product-grpc/      ← Project Fase 5
        ├── fase-06-order-service/     ← Project Fase 6
        └── fase-07-ecommerce/         ← Project Fase 7
```

---

## ⚙️ VS Code Settings untuk Go

Tambahkan ke `.vscode/settings.json` di setiap project:

```json
{
  "go.useLanguageServer": true,
  "go.lintTool": "golangci-lint",
  "go.lintOnSave": "package",
  "go.formatTool": "gofmt",
  "go.formatFlags": ["-s"],
  "editor.formatOnSave": true,
  "[go]": {
    "editor.codeActionsOnSave": {
      "source.organizeImports": true
    }
  },
  "go.testFlags": ["-v", "-race"],
  "go.coverOnSave": false,
  "go.coverageDecorator": {
    "type": "gutter"
  }
}
```

---

## 🐛 Troubleshooting Umum

### "go: command not found"
```bash
# Tambahkan ke ~/.bashrc atau ~/.zshrc:
export PATH=$PATH:/usr/local/go/bin
source ~/.bashrc
```

### "cannot find module"
```bash
# Pastikan kamu di direktori dengan go.mod
go mod tidy
go mod download
```

### "unused import"
```go
// Go sangat strict: import yang tidak dipakai = error
// Gunakan _ untuk sementara:
import _ "fmt"  // gunakan saat development
```

### Port sudah dipakai
```bash
# Cek proses yang menggunakan port
lsof -i :8080
kill -9 <PID>
```

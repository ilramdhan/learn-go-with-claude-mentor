# 📚 Referensi & Sumber Belajar

Koleksi sumber belajar terbaik untuk melengkapi setiap fase kurikulum ini.

---

## 📖 Dokumentasi Resmi (Wajib Bookmarked)

| Sumber | URL | Fase yang Relevan |
|--------|-----|-------------------|
| Go Documentation | https://go.dev/doc/ | Semua |
| Effective Go | https://go.dev/doc/effective_go | Fase 1-2 |
| Go Specification | https://go.dev/ref/spec | Semua |
| Go Standard Library | https://pkg.go.dev/std | Semua |
| Go Blog | https://go.dev/blog | Semua |
| Go Playground | https://go.dev/play | Fase 1-2 |

---

## 📘 Buku Terbaik

| Buku | Penulis | Level | Fase |
|------|---------|-------|------|
| The Go Programming Language | Donovan & Kernighan | Beginner-Mid | 1-2 |
| Go in Action | Kennedy, Ketelsen, Martin | Intermediate | 2-4 |
| Learning Go | Jon Bodner | Intermediate | 1-4 |
| Cloud Native Go | Matthew Titmus | Advanced | 7-11 |
| Domain-Driven Design | Eric Evans | Advanced | 6 |
| Building Microservices | Sam Newman | Advanced | 7-8 |
| Designing Data-Intensive Applications | Martin Kleppmann | Advanced | 7-9 |

---

## 🎮 Belajar Interaktif

| Platform | Konten | Level |
|----------|--------|-------|
| [Tour of Go](https://go.dev/tour) | Tutorial interaktif resmi | Beginner |
| [Go by Example](https://gobyexample.com) | Contoh kode annotated | Beginner-Mid |
| [Exercism Go Track](https://exercism.org/tracks/go) | Latihan dengan mentor | Semua |
| [LeetCode (Go)](https://leetcode.com) | Algoritma & data structures | Semua |
| [Gophercises](https://gophercises.com) | Project exercises | Intermediate |

---

## 🎥 Video & Kursus

| Platform | Konten | Level |
|----------|--------|-------|
| [freeCodeCamp Go Course](https://www.youtube.com/watch?v=un6ZyFkqFKo) | 25 jam, Go lengkap | Beginner |
| [Ardan Labs Go Training](https://www.ardanlabs.com/training/) | Ultimate Go | Intermediate-Advanced |
| [TechWorld with Nana](https://youtube.com/@TechWorldwithNana) | Docker, K8s, DevOps | Fase 7-11 |
| [Hussein Nasser](https://youtube.com/@hnasr) | Backend engineering | Fase 4-8 |
| [ByteByteGo](https://youtube.com/@ByteByteGo) | System Design | Fase 7 |

---

## 🔧 Tools yang Wajib Dipasang

### Fase 1-3 (Dasar)
```bash
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
go install honnef.co/go/tools/cmd/staticcheck@latest
go install github.com/go-delve/delve/cmd/dlv@latest  # debugger
```

### Fase 4-6 (Arsitektur)
```bash
go install github.com/vektra/mockery/v2@latest          # mock generator
go install github.com/golang-migrate/migrate/v4/cmd/migrate@latest
```

### Fase 5 (gRPC)
```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
go install github.com/bufbuild/buf/cmd/buf@latest
brew install grpcurl   # gRPC CLI client
```

### Fase 7-11 (Production)
```bash
# Docker & Kubernetes
brew install --cask docker
brew install kubectl minikube helm
brew install stern     # multi-pod log streaming
brew install k9s       # Kubernetes TUI dashboard

# Load testing
brew install k6

# Profiling
go install github.com/google/pprof@latest

# Terraform
brew install terraform

# ArgoCD CLI
brew install argocd
```

---

## 🏛️ Project & Repository Referensi

| Repository | Konten | Pelajari untuk |
|------------|--------|----------------|
| [go-micro](https://github.com/micro/micro) | Microservices framework | Fase 7 |
| [prometheus](https://github.com/prometheus/prometheus) | Codebase Go production | Fase 9 |
| [kubernetes](https://github.com/kubernetes/kubernetes) | Go di scale besar | Fase 7, 11 |
| [docker](https://github.com/moby/moby) | Go networking & container | Fase 7 |
| [cockroachdb](https://github.com/cockroachdb/cockroach) | Distributed systems Go | Fase 7-8 |
| [etcd](https://github.com/etcd-io/etcd) | Distributed key-value | Fase 7 |

---

## 📰 Blog & Newsletter

| Sumber | Topik |
|--------|-------|
| [The Go Blog](https://go.dev/blog) | Official Go announcements |
| [Dave Cheney's Blog](https://dave.cheney.net) | Go internals & best practices |
| [Ardan Labs Blog](https://www.ardanlabs.com/blog/) | Go deep dives |
| [Go Weekly](https://golangweekly.com) | Newsletter mingguan |
| [High Scalability](https://highscalability.com) | System design case studies |

---

## 🌐 Komunitas

| Komunitas | Platform | Link |
|-----------|----------|------|
| Gophers Slack | Slack | https://gophers.slack.com |
| r/golang | Reddit | https://reddit.com/r/golang |
| Go Forum | Forum | https://forum.golangbridge.org |
| GoBridge | Inclusivity | https://gobridge.org |
| Golang Indonesia | Telegram | https://t.me/go_id |

---

## 📑 Paper & Design Docs

| Paper | Topik | Relevan |
|-------|-------|---------|
| [MapReduce](https://research.google/pubs/pub62/) | Distributed computing | Fase 7 |
| [The Chubby Lock Service](https://research.google/pubs/pub27897/) | Distributed coordination | Fase 7 |
| [Kafka Paper](https://notes.stephenholiday.com/Kafka.pdf) | Event streaming design | Fase 8 |
| [DORA State of DevOps](https://dora.dev/research/) | DevOps metrics | Fase 11 |
| [Google SRE Book](https://sre.google/sre-book/table-of-contents/) | SRE practices | Fase 9, 11 |

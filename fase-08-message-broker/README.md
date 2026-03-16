# 📘 Fase 8: Message Broker & Event-Driven Architecture

> **Level:** 🔴 Advanced
> **Durasi Estimasi:** 3–4 minggu
> **Prasyarat:** ✅ Selesaikan Fase 7 — Microservices Architecture

---

## 🎯 Tujuan Fase Ini

- ✅ Memahami kapan pakai async communication vs sync
- ✅ Menguasai Apache Kafka (producer, consumer, consumer group)
- ✅ Menguasai RabbitMQ (exchange types, work queue, pub/sub)
- ✅ Implementasi Outbox Pattern untuk guaranteed event delivery
- ✅ Menangani poison messages dengan Dead Letter Queue
- ✅ Merancang event schema yang backward compatible

---

## 🗂️ Isi Folder

```
fase-08-message-broker/
├── README.md
├── materi/
│   └── FASE-8-Message-Broker.md    ← 13 modul (83KB)
└── project/
    └── FASE-8-PRD-Notification-Analytics.md
```

---

## 📚 Daftar Modul

| # | Modul | Konsep Utama |
|---|-------|-------------|
| 8.1 | Mengapa Message Broker? | Queue vs Pub/Sub vs Streaming |
| 8.2 | Kafka Fundamentals | Topic, partition, offset, consumer group |
| 8.3 | Setup Kafka | Docker Compose, kafka-go, Kafka UI |
| 8.4 | Kafka Producer | Keys, headers, partitioning, batch |
| 8.5 | Kafka Consumer | Consumer group, offset, graceful shutdown |
| 8.6 | RabbitMQ Fundamentals | Exchange types, queue properties |
| 8.7 | RabbitMQ dengan Go | amqp091-go, connection, channel |
| 8.8 | RabbitMQ Patterns | Work Queue, Fanout, Topic, RPC |
| 8.9 | Event-Driven Patterns | Event sourcing, CQRS, choreography |
| 8.10 | Outbox Pattern | Guaranteed delivery, transactional outbox |
| 8.11 | DLQ & Error Handling | Dead Letter Queue, retry, poison messages |
| 8.12 | Event Schema Evolution | Versioning, backward compatibility |
| 8.13 | Integrasi Microservices | Update Fase 7 dengan async events |

---

## 🛠️ Tools yang Dibutuhkan

```bash
# Kafka
docker compose -f docker-compose.kafka.yml up -d
# Kafka UI: http://localhost:8090

# RabbitMQ
docker compose -f docker-compose.rabbitmq.yml up -d
# Management UI: http://localhost:15672

# Go libraries
go get github.com/segmentio/kafka-go
go get github.com/rabbitmq/amqp091-go
```

---

## 🎯 Project Akhir: Notification & Analytics System

File PRD: `project/FASE-8-PRD-Notification-Analytics.md`

Extend sistem dari Fase 7 dengan:
- Kafka untuk async events
- Notification Service (email/SMS simulation)
- Analytics Service (revenue stats, product popularity)
- Outbox Pattern + DLQ

---

## ✅ Checklist Kelulusan

- [ ] Kafka berjalan dan bisa produce/consume messages
- [ ] Consumer group mendistribusikan messages ke multiple instances
- [ ] Outbox Pattern berjalan (events guaranteed delivery)
- [ ] DLQ menangkap poison messages setelah retry habis
- [ ] Event schema menggunakan versioning
- [ ] Notification Service log events yang diterima
- [ ] Analytics dashboard menampilkan stats yang benar
- [ ] **Menyelesaikan project Notification & Analytics System**

---

## ➡️ Selanjutnya

Setelah selesai Fase 8 → [Fase 9: Observability & Production](../fase-09-observability/)

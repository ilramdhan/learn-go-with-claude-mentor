# 📘 FASE 8: Message Broker & Event-Driven Architecture

> **Prasyarat:** Selesaikan Fase 7 — Microservices Architecture
> **Durasi:** 3–4 minggu
> **Project Akhir:** Notification & Analytics System
> **Tujuan:** Memahami dan mengimplementasikan komunikasi asynchronous antar service menggunakan Kafka dan RabbitMQ, menguasai event-driven patterns yang dipakai di perusahaan tech besar

---

## 🗂️ Daftar Modul

| # | Modul | Topik |
|---|-------|-------|
| 8.1 | Mengapa Message Broker? | Queue vs Pub-Sub, kapan pakai async |
| 8.2 | Apache Kafka Fundamentals | Topic, partition, offset, consumer group |
| 8.3 | Setup Kafka | Docker Compose, kafka-go library |
| 8.4 | Kafka Producer | Producing messages, keys, headers, partitioning |
| 8.5 | Kafka Consumer | Consumer groups, offset commit, error handling |
| 8.6 | RabbitMQ Fundamentals | Exchange types, queues, bindings, routing |
| 8.7 | RabbitMQ dengan Go | amqp091-go, publish, consume, prefetch |
| 8.8 | RabbitMQ Patterns | Direct, Fanout, Topic, Work Queue, RPC |
| 8.9 | Event-Driven Architecture | Event sourcing, CQRS with events, choreography |
| 8.10 | Outbox Pattern | Guaranteed delivery, transactional outbox |
| 8.11 | Dead Letter Queue & Error Handling | DLQ, retry, poison messages |
| 8.12 | Event Schema Design & Evolution | Avro, Protobuf, schema registry, versioning |
| 8.13 | Integrasi dengan Microservices | Menghubungkan Fase 7 services |

---

## 📦 Modul 8.1 — Mengapa Message Broker?

### Masalah Komunikasi Synchronous

Di Fase 7 kita sudah pelajari bahwa microservices berkomunikasi lewat gRPC (sync). Ini bagus untuk operasi yang butuh response langsung, tapi punya masalah serius:

```
MASALAH SYNC COMMUNICATION:

Order Service ──gRPC──> Email Service
                            │
                    Email Service DOWN!
                            │
                    Order Service error ❌

Order selesai tapi user tidak dapat email karena Email Service down.
Haruskah Order fail hanya gara-gara Email Service down? TIDAK!

Solusi: Kirim event, lanjutkan tanpa nunggu response.
```

### Apa itu Message Broker?

Message Broker adalah **perantara** yang menyimpan dan meneruskan pesan antara producer dan consumer.

```
Tanpa Broker:
  Order Service ──langsung──> Notification Service
  (tight coupling, synchronous, consumer harus online)

Dengan Message Broker:
  Order Service ──> [Broker] ──> Notification Service
  Order Service ──> [Broker] ──> Analytics Service
  Order Service ──> [Broker] ──> Inventory Service
  (loose coupling, asynchronous, consumer bisa offline sementara)
```

### Queue vs Pub/Sub vs Event Streaming

```
MESSAGE QUEUE (Point-to-Point):
  Producer ──> [Queue] ──> Consumer A
  
  - Satu pesan dikonsumsi TEPAT SATU consumer
  - Cocok untuk: background jobs, task distribution
  - Contoh: "Proses order #123" — hanya satu worker yang proses
  - Tools: RabbitMQ (queue mode), Redis Queue

PUB/SUB (Publish-Subscribe):
  Publisher ──> [Topic/Exchange] ──> Consumer A (subscriber)
                                 ──> Consumer B (subscriber)
                                 ──> Consumer C (subscriber)
  
  - Satu pesan dikonsumsi SEMUA subscriber
  - Cocok untuk: notifikasi, event broadcasting
  - Contoh: "Order confirmed" → email + SMS + analytics semuanya terima
  - Tools: RabbitMQ (fanout exchange), Kafka, Redis Pub/Sub

EVENT STREAMING (Persistent Log):
  Producer ──> [Topic: ordered log] ──> Consumer Group A (offset 5)
                                    ──> Consumer Group B (offset 3)
                                    ──> Consumer Group C (offset 7)
  
  - Events disimpan sebagai ordered log
  - Consumer bisa replay dari offset manapun
  - Cocok untuk: event sourcing, audit log, data pipeline
  - Tools: Apache Kafka, Amazon Kinesis
```

### Kapan Pakai Message Broker?

```
✅ PAKAI message broker untuk:
  - Side effects (kirim email, SMS, update analytics)
  - Operasi yang bisa di-delay (tidak perlu instant)
  - Satu event perlu banyak consumer
  - Decoupling service yang sering berubah
  - Background processing (image resize, report generation)
  - Rate limiting downstream (queue sebagai buffer)

❌ JANGAN pakai message broker untuk:
  - Operasi yang butuh response langsung (check stock saat checkout)
  - Data yang harus konsisten saat request selesai
  - Simple query/read operations
  - Tim yang belum familiar → over-engineering
```

### Perbandingan Kafka vs RabbitMQ

```
APACHE KAFKA:
  ✅ Throughput sangat tinggi (jutaan msg/detik)
  ✅ Persistent storage (bisa replay events)
  ✅ Consumer bisa baca dari offset manapun
  ✅ Ordered per partition
  ✅ Event sourcing, data pipeline, audit log
  ❌ Kompleks untuk setup dan operasi
  ❌ Tidak ada routing yang fleksibel
  ❌ Message tidak bisa di-acknowledge per-message (commit offset)
  Cocok untuk: high throughput, event streaming, data pipeline

RABBITMQ:
  ✅ Fleksibel routing (exchange types)
  ✅ Per-message acknowledgement
  ✅ Mudah setup dan operate
  ✅ Plugin ekosistem kaya
  ✅ RPC pattern built-in
  ❌ Throughput lebih rendah dari Kafka
  ❌ Message dihapus setelah dikonsumsi (tidak persistent by default)
  Cocok untuk: task queue, complex routing, RPC, work distribution
```

### 🏋️ Latihan 8.1

1. Gambar **communication diagram** untuk sistem e-commerce: tentukan setiap komunikasi mana yang harus sync (gRPC/HTTP) dan mana yang harus async (message broker). Buat tabel dengan kolom: From, To, Event/Command, Sync/Async, Alasan.
2. Analisis skenario ini dan tentukan solusi terbaik: "Saat order confirmed, sistem harus (a) kurangi stok — butuh konfirmasi bahwa berhasil, (b) kirim email ke user, (c) update dashboard analytics, (d) notifikasi warehouse team". Mana yang sync, mana yang async?
3. Buat **trade-off analysis** antara Kafka vs RabbitMQ untuk kasus: e-commerce dengan 1000 order/hari yang butuh kirim notifikasi email + SMS + update analytics. Rekomendasikan mana yang lebih cocok.

---

## 📦 Modul 8.2 — Apache Kafka Fundamentals

### Arsitektur Kafka

```
KAFKA CLUSTER:
  ┌────────────────────────────────────────────────┐
  │                  Kafka Cluster                  │
  │                                                │
  │  Broker 1          Broker 2          Broker 3  │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────┐  │
  │  │Topic:orders  │  │Topic:orders  │  │ ...  │  │
  │  │ Partition 0  │  │ Partition 1  │  │      │  │
  │  │ [0][1][2][3] │  │ [0][1][2]   │  │      │  │
  │  └──────────────┘  └──────────────┘  └──────┘  │
  └────────────────────────────────────────────────┘
        ▲                    ▲
     Producer            Consumer Group
  (Order Service)       (Notif + Analytics)
```

### Konsep Inti Kafka

```
TOPIC:
  Named channel/stream untuk menyimpan events.
  Seperti "tabel" di database tapi append-only.
  Contoh: "orders", "payments", "user-events"

PARTITION:
  Sebuah topic dibagi menjadi N partisi untuk parallelism.
  Messages dalam satu partisi: ORDERED (urutan dijaga).
  Messages antar partisi: TIDAK ordered.
  
  Gunakan:
  - partition key yang sama → partition yang sama → ordered
  - Contoh: pakai order_id sebagai key → semua events satu order ordered

OFFSET:
  Posisi unik setiap message dalam partisi (0, 1, 2, ...).
  Consumer menyimpan offset terakhir yang sudah diproses.
  Bisa replay dari offset manapun!

CONSUMER GROUP:
  Sekumpulan consumer yang bekerja sama membaca satu topic.
  Setiap partition hanya dibaca SATU consumer dalam satu group.
  
  Contoh: topic dengan 3 partisi, consumer group dengan 3 consumer:
  Consumer 1 → Partition 0
  Consumer 2 → Partition 1
  Consumer 3 → Partition 2
  
  Kalau 2 consumer group berbeda membaca topic yang sama:
  Mereka masing-masing mendapat SEMUA messages (seperti pub/sub!)

RETENTION:
  Kafka menyimpan messages untuk jangka waktu tertentu (default: 7 hari).
  Berbeda dengan RabbitMQ yang hapus message setelah dikonsumsi.
  Berguna untuk: replay events, debugging, new consumer catch up.

LEADER & REPLICA:
  Setiap partition punya 1 leader dan N-1 replika.
  Semua read/write ke leader.
  Replika untuk fault tolerance.
```

### Partitioning Strategy

```
STRATEGY 1: Round Robin (default, tanpa key)
  Message 1 → Partition 0
  Message 2 → Partition 1
  Message 3 → Partition 2
  Pros: Load distribution merata
  Cons: Messages dari entitas sama bisa di partisi berbeda → tidak ordered

STRATEGY 2: Key-based (pilihan yang lebih baik)
  hash(order_id) % num_partitions = partition
  Order #123, event A → Partition 1
  Order #123, event B → Partition 1 (sama!)
  Order #456, event A → Partition 0
  Pros: Events satu order selalu ordered
  Cons: Hot partition jika key tidak merata

STRATEGY 3: Custom Partitioner
  Bisa implementasi logic custom (misalnya: VIP customers → dedicated partition)
```

### Kafka Guarantees

```
AT-MOST-ONCE: Message mungkin hilang, tidak pernah duplikat
  - Commit offset SEBELUM process message
  - Jika crash saat process → message hilang
  - Cocok untuk: metrics, logging (OK jika ada yang hilang)

AT-LEAST-ONCE: Message tidak hilang, mungkin duplikat
  - Commit offset SETELAH process message berhasil
  - Jika crash setelah process tapi sebelum commit → message diproses 2x
  - PALING UMUM dipakai
  - Butuh idempotent consumer untuk handle duplikat

EXACTLY-ONCE: Tidak hilang, tidak duplikat
  - Pakai Kafka transactions
  - Sangat kompleks, ada performance penalty
  - Cocok untuk: financial systems, critical data
```

### 🏋️ Latihan 8.2

1. Desain **topic structure** untuk e-commerce system: tentukan nama topics, jumlah partisi, retention period, dan partition key untuk setiap topic. Minimal 5 topics: orders, payments, notifications, inventory, user-events.
2. Jelaskan mengapa partition key `order_id` lebih baik dari random key untuk topic `order-events`. Buat skenario di mana random key menyebabkan masalah.
3. Tentukan delivery guarantee yang tepat untuk masing-masing use case: (a) update analytics view count, (b) kirim email konfirmasi order, (c) debit saldo payment, (d) update search index.

---

## 📦 Modul 8.3 — Setup Kafka

### Docker Compose untuk Kafka

```yaml
# docker-compose.kafka.yml
version: '3.8'

services:
  # Zookeeper (diperlukan oleh Kafka versi lama, masih banyak dipakai)
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    networks:
      - kafka-net
    healthcheck:
      test: ["CMD", "nc", "-z", "localhost", "2181"]
      interval: 10s
      timeout: 5s
      retries: 5

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka
    depends_on:
      zookeeper:
        condition: service_healthy
    ports:
      - "9092:9092"       # untuk host machine
      - "29092:29092"     # untuk container-to-container
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      # Dua listeners: satu untuk external (host), satu untuk internal (container)
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_LOG_RETENTION_HOURS: 168    # 7 hari
      KAFKA_LOG_SEGMENT_BYTES: 1073741824
    networks:
      - kafka-net
    healthcheck:
      test: ["CMD", "kafka-broker-api-versions", "--bootstrap-server", "localhost:29092"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 30s

  # Kafka UI — visualisasi topics, messages, consumer groups
  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    container_name: kafka-ui
    depends_on:
      - kafka
    ports:
      - "8090:8080"    # akses di http://localhost:8090
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:29092
    networks:
      - kafka-net

  # Alternative: KRaft mode (tanpa Zookeeper, Kafka 3.3+)
  # kafka-kraft:
  #   image: confluentinc/cp-kafka:7.5.0
  #   environment:
  #     KAFKA_NODE_ID: 1
  #     KAFKA_PROCESS_ROLES: broker,controller
  #     KAFKA_LISTENERS: PLAINTEXT://kafka:29092,CONTROLLER://kafka:29093
  #     KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:29093
  #     CLUSTER_ID: MkU3OEVBNTcwNTJENDM2Qk

networks:
  kafka-net:
    driver: bridge
```

### Install kafka-go Library

```bash
go get github.com/segmentio/kafka-go
```

### Membuat dan Mengelola Topics

```go
// pkg/kafka/admin.go
package kafka

import (
    "context"
    "fmt"
    "net"
    "strconv"

    "github.com/segmentio/kafka-go"
)

// AdminClient mengelola topics dan konfigurasi Kafka
type AdminClient struct {
    brokers []string
}

func NewAdminClient(brokers []string) *AdminClient {
    return &AdminClient{brokers: brokers}
}

// CreateTopic membuat topic baru
func (a *AdminClient) CreateTopic(ctx context.Context, topic string, partitions, replicationFactor int) error {
    conn, err := kafka.DialContext(ctx, "tcp", a.brokers[0])
    if err != nil {
        return fmt.Errorf("dial kafka: %w", err)
    }
    defer conn.Close()

    controller, err := conn.Controller()
    if err != nil {
        return fmt.Errorf("get controller: %w", err)
    }

    controllerConn, err := kafka.DialContext(ctx, "tcp",
        net.JoinHostPort(controller.Host, strconv.Itoa(controller.Port)))
    if err != nil {
        return fmt.Errorf("dial controller: %w", err)
    }
    defer controllerConn.Close()

    err = controllerConn.CreateTopics(kafka.TopicConfig{
        Topic:             topic,
        NumPartitions:     partitions,
        ReplicationFactor: replicationFactor,
    })
    if err != nil {
        return fmt.Errorf("create topic %s: %w", topic, err)
    }

    fmt.Printf("Topic '%s' created with %d partitions\n", topic, partitions)
    return nil
}

// EnsureTopics memastikan semua topic yang dibutuhkan ada
func (a *AdminClient) EnsureTopics(ctx context.Context, configs []TopicConfig) error {
    for _, cfg := range configs {
        if err := a.CreateTopic(ctx, cfg.Name, cfg.Partitions, cfg.ReplicationFactor); err != nil {
            // Topic mungkin sudah ada — ignore error itu
            fmt.Printf("Note: topic '%s': %v\n", cfg.Name, err)
        }
    }
    return nil
}

type TopicConfig struct {
    Name              string
    Partitions        int
    ReplicationFactor int
}

// Definisikan semua topics yang dibutuhkan aplikasi
var AppTopics = []TopicConfig{
    {Name: "orders",        Partitions: 3, ReplicationFactor: 1},
    {Name: "payments",      Partitions: 3, ReplicationFactor: 1},
    {Name: "notifications", Partitions: 2, ReplicationFactor: 1},
    {Name: "inventory",     Partitions: 3, ReplicationFactor: 1},
    {Name: "user-events",   Partitions: 2, ReplicationFactor: 1},
}
```

### Test Koneksi

```bash
# Start kafka
docker compose -f docker-compose.kafka.yml up -d

# Verifikasi berjalan
docker compose -f docker-compose.kafka.yml ps

# Buat topic manual via CLI
docker exec kafka kafka-topics \
  --create \
  --topic orders \
  --partitions 3 \
  --replication-factor 1 \
  --bootstrap-server localhost:29092

# List topics
docker exec kafka kafka-topics \
  --list \
  --bootstrap-server localhost:29092

# Produce test message
docker exec -it kafka kafka-console-producer \
  --topic orders \
  --bootstrap-server localhost:29092
# Ketik beberapa message lalu Ctrl+C

# Consume test message
docker exec kafka kafka-console-consumer \
  --topic orders \
  --from-beginning \
  --bootstrap-server localhost:29092

# Akses Kafka UI di browser: http://localhost:8090
```

### 🏋️ Latihan 8.3

1. Setup Kafka dengan Docker Compose. Buka Kafka UI di `localhost:8090`. Buat topic `test-events` dengan 3 partisi menggunakan CLI. Verifikasi topic muncul di Kafka UI.
2. Buat `AdminClient` dan gunakan `EnsureTopics` untuk membuat semua `AppTopics` saat aplikasi startup. Jalankan dua kali — pastikan tidak error jika topic sudah ada.
3. Buat function `GetTopicInfo(ctx, topicName) (*TopicInfo, error)` yang mengembalikan: jumlah partisi, replication factor, dan jumlah messages di setiap partisi. Tampilkan hasilnya dalam format table.

---

## 📦 Modul 8.4 — Kafka Producer

### Producer Dasar

```go
// pkg/kafka/producer.go
package kafka

import (
    "context"
    "encoding/json"
    "fmt"
    "time"

    "github.com/segmentio/kafka-go"
)

// Producer menulis messages ke Kafka topic
type Producer struct {
    writer *kafka.Writer
    topic  string
}

// NewProducer membuat producer baru
func NewProducer(brokers []string, topic string) *Producer {
    writer := &kafka.Writer{
        Addr:  kafka.TCP(brokers...),
        Topic: topic,

        // Balancer: tentukan partisi untuk setiap message
        // LeastBytes: kirim ke partisi dengan bytes paling sedikit (default)
        // Hash: pakai message key untuk konsisten routing
        Balancer: &kafka.Hash{},  // partition by key

        // Batching untuk performa
        BatchSize:    100,                    // kirim per 100 message
        BatchTimeout: 10 * time.Millisecond, // atau setiap 10ms
        
        // Reliability settings
        RequiredAcks: kafka.RequireAll,  // tunggu semua replika konfirmasi
        
        // Compression (hemat bandwidth)
        Compression: kafka.Snappy,
        
        // Retry untuk transient errors
        MaxAttempts: 3,

        // Error logger
        ErrorLogger: kafka.LoggerFunc(func(msg string, a ...interface{}) {
            fmt.Printf("[kafka-producer] ERROR: "+msg+"\n", a...)
        }),
    }

    return &Producer{writer: writer, topic: topic}
}

// Message adalah struktur pesan yang akan dikirim
type Message struct {
    Key     string            // partition key
    Payload interface{}       // data yang akan di-serialize ke JSON
    Headers map[string]string // metadata (event type, correlation ID, dll)
}

// Publish mengirim satu message ke Kafka
func (p *Producer) Publish(ctx context.Context, msg Message) error {
    // Serialize payload ke JSON
    body, err := json.Marshal(msg.Payload)
    if err != nil {
        return fmt.Errorf("marshal payload: %w", err)
    }

    // Build Kafka headers
    var headers []kafka.Header
    headers = append(headers, kafka.Header{
        Key:   "content-type",
        Value: []byte("application/json"),
    })
    headers = append(headers, kafka.Header{
        Key:   "produced-at",
        Value: []byte(time.Now().UTC().Format(time.RFC3339)),
    })
    for k, v := range msg.Headers {
        headers = append(headers, kafka.Header{
            Key:   k,
            Value: []byte(v),
        })
    }

    kafkaMsg := kafka.Message{
        Key:     []byte(msg.Key),
        Value:   body,
        Headers: headers,
        Time:    time.Now(),
    }

    if err := p.writer.WriteMessages(ctx, kafkaMsg); err != nil {
        return fmt.Errorf("write message to topic %s: %w", p.topic, err)
    }

    return nil
}

// PublishBatch mengirim banyak message sekaligus (lebih efisien)
func (p *Producer) PublishBatch(ctx context.Context, msgs []Message) error {
    kafkaMsgs := make([]kafka.Message, len(msgs))
    for i, msg := range msgs {
        body, err := json.Marshal(msg.Payload)
        if err != nil {
            return fmt.Errorf("marshal message[%d]: %w", i, err)
        }
        kafkaMsgs[i] = kafka.Message{
            Key:   []byte(msg.Key),
            Value: body,
            Time:  time.Now(),
        }
    }

    if err := p.writer.WriteMessages(ctx, kafkaMsgs...); err != nil {
        return fmt.Errorf("batch write to %s: %w", p.topic, err)
    }
    return nil
}

// Close menutup producer — panggil saat shutdown
func (p *Producer) Close() error {
    return p.writer.Close()
}
```

### Event Types dan Domain Events

```go
// events/order_events.go
package events

import "time"

// EventType adalah konstanta untuk semua event names
const (
    EventOrderCreated   = "order.created"
    EventOrderSubmitted = "order.submitted"
    EventOrderConfirmed = "order.confirmed"
    EventOrderCancelled = "order.cancelled"
    EventOrderDelivered = "order.delivered"
)

// BaseEvent adalah struct dasar yang diwarisi semua events
type BaseEvent struct {
    EventID     string    `json:"event_id"`     // UUID unik per event
    EventType   string    `json:"event_type"`   // "order.confirmed"
    AggregateID string    `json:"aggregate_id"` // ID entity yang berkaitan
    OccurredAt  time.Time `json:"occurred_at"`
    Version     int       `json:"version"`      // untuk schema evolution
}

// OrderCreatedEvent dipublish saat order baru dibuat
type OrderCreatedEvent struct {
    BaseEvent
    CustomerID      uint64  `json:"customer_id"`
    CustomerEmail   string  `json:"customer_email"`
    TotalAmount     float64 `json:"total_amount"`
    Currency        string  `json:"currency"`
    ItemCount       int     `json:"item_count"`
}

// OrderConfirmedEvent dipublish saat pembayaran berhasil
type OrderConfirmedEvent struct {
    BaseEvent
    CustomerID    uint64            `json:"customer_id"`
    CustomerEmail string            `json:"customer_email"`
    PaymentID     string            `json:"payment_id"`
    TotalAmount   float64           `json:"total_amount"`
    Items         []OrderItemEvent  `json:"items"`
}

type OrderItemEvent struct {
    ProductID   uint64  `json:"product_id"`
    ProductName string  `json:"product_name"`
    Quantity    int     `json:"quantity"`
    UnitPrice   float64 `json:"unit_price"`
}

// OrderCancelledEvent dipublish saat order dibatalkan
type OrderCancelledEvent struct {
    BaseEvent
    CustomerID    uint64            `json:"customer_id"`
    Reason        string            `json:"reason"`
    StockRestored bool              `json:"stock_restored"`
    Items         []OrderItemEvent  `json:"items"` // untuk restore stok
}
```

```go
// Penggunaan producer di Order Service
type OrderEventPublisher struct {
    producer *kafka.Producer
}

func NewOrderEventPublisher(brokers []string) *OrderEventPublisher {
    return &OrderEventPublisher{
        producer: kafka.NewProducer(brokers, "orders"),
    }
}

func (p *OrderEventPublisher) PublishOrderConfirmed(ctx context.Context, order *Order, paymentID string) error {
    event := events.OrderConfirmedEvent{
        BaseEvent: events.BaseEvent{
            EventID:     uuid.New().String(),
            EventType:   events.EventOrderConfirmed,
            AggregateID: fmt.Sprintf("%d", order.ID),
            OccurredAt:  time.Now().UTC(),
            Version:     1,
        },
        CustomerID:    order.CustomerID,
        CustomerEmail: order.CustomerEmail,
        PaymentID:     paymentID,
        TotalAmount:   order.TotalAmount,
        Items:         toOrderItemEvents(order.Items),
    }

    return p.producer.Publish(ctx, kafka.Message{
        // Pakai order ID sebagai key → events satu order di partisi yang sama
        Key: fmt.Sprintf("order-%d", order.ID),
        Payload: event,
        Headers: map[string]string{
            "event-type":     events.EventOrderConfirmed,
            "correlation-id": ctx.Value("request_id").(string),
            "source-service": "order-service",
        },
    })
}
```

### 🏋️ Latihan 8.4

1. Buat `OrderEventPublisher` dengan method untuk publish: `OrderCreated`, `OrderConfirmed`, `OrderCancelled`. Gunakan order ID sebagai partition key. Test dengan kafka-console-consumer untuk verifikasi messages masuk.
2. Implementasikan **producer dengan transaction**: gunakan `kafka.NewWriter` dengan mode transaksional sehingga batch publish bersifat atomic (semua berhasil atau semua gagal). Test dengan kill process di tengah batch.
3. Tambahkan **metrics** ke producer: hitung total messages produced, failed messages, dan average latency per publish. Expose sebagai `ProducerStats` struct yang bisa di-query.

---

## 📦 Modul 8.5 — Kafka Consumer

### Consumer dengan Consumer Group

```go
// pkg/kafka/consumer.go
package kafka

import (
    "context"
    "encoding/json"
    "fmt"
    "time"

    "github.com/segmentio/kafka-go"
)

// MessageHandler adalah fungsi yang dipanggil untuk setiap message
type MessageHandler func(ctx context.Context, msg ConsumedMessage) error

// ConsumedMessage adalah pesan yang sudah di-parse dari Kafka
type ConsumedMessage struct {
    Key       string
    Value     []byte
    Headers   map[string]string
    Topic     string
    Partition int
    Offset    int64
    Time      time.Time
}

// Consumer membaca messages dari Kafka topic
type Consumer struct {
    reader  *kafka.Reader
    topic   string
    groupID string
}

// NewConsumer membuat consumer baru yang bergabung dengan consumer group
func NewConsumer(brokers []string, topic, groupID string) *Consumer {
    reader := kafka.NewReader(kafka.ReaderConfig{
        Brokers:     brokers,
        Topic:       topic,
        GroupID:     groupID, // consumer group ID

        // Offset management
        // StartOffset: kafka.LastOffset,   // hanya pesan baru (default)
        // StartOffset: kafka.FirstOffset,  // mulai dari awal

        // Batching
        MinBytes: 10e3,               // 10KB minimum sebelum fetch
        MaxBytes: 10e6,               // 10MB maximum per fetch
        MaxWait:  100 * time.Millisecond,

        // Error handling
        Logger: kafka.LoggerFunc(func(msg string, a ...interface{}) {
            fmt.Printf("[kafka-consumer] INFO: "+msg+"\n", a...)
        }),
        ErrorLogger: kafka.LoggerFunc(func(msg string, a ...interface{}) {
            fmt.Printf("[kafka-consumer] ERROR: "+msg+"\n", a...)
        }),

        // Commit interval — seberapa sering offset di-commit ke Kafka
        CommitInterval: time.Second,  // commit setiap detik
    })

    return &Consumer{
        reader:  reader,
        topic:   topic,
        groupID: groupID,
    }
}

// Start mulai membaca messages dan mengirim ke handler
// Blocking — jalankan di goroutine
func (c *Consumer) Start(ctx context.Context, handler MessageHandler) error {
    fmt.Printf("[consumer] Starting consumer group '%s' for topic '%s'\n", c.groupID, c.topic)

    for {
        // Fetch satu message (blocking sampai ada message atau context cancelled)
        m, err := c.reader.FetchMessage(ctx)
        if err != nil {
            if ctx.Err() != nil {
                // Context cancelled — shutdown graceful
                fmt.Printf("[consumer] Context cancelled, stopping consumer\n")
                return nil
            }
            return fmt.Errorf("fetch message: %w", err)
        }

        // Parse headers
        headers := make(map[string]string)
        for _, h := range m.Headers {
            headers[h.Key] = string(h.Value)
        }

        msg := ConsumedMessage{
            Key:       string(m.Key),
            Value:     m.Value,
            Headers:   headers,
            Topic:     m.Topic,
            Partition: m.Partition,
            Offset:    m.Offset,
            Time:      m.Time,
        }

        // Proses message
        if err := handler(ctx, msg); err != nil {
            // PENTING: Handler error — jangan commit offset!
            // Message akan di-retry saat consumer restart
            fmt.Printf("[consumer] Handler error for offset %d: %v\n", m.Offset, err)

            // Option: tergantung pada error handling strategy:
            // 1. Return error → consumer restart, retry message (at-least-once)
            // 2. Log error, commit offset → message di-skip (at-most-once)
            // 3. Send to DLQ → message di-skip tapi disimpan untuk inspection
            continue
        }

        // Commit offset setelah berhasil — AT-LEAST-ONCE guarantee
        if err := c.reader.CommitMessages(ctx, m); err != nil {
            fmt.Printf("[consumer] Commit offset error: %v\n", err)
        }
    }
}

// Close menutup consumer
func (c *Consumer) Close() error {
    return c.reader.Close()
}
```

### Consumer dengan Manual Offset Management

```go
// Untuk fine-grained control atas offset commits
func consumeWithManualCommit(ctx context.Context, brokers []string) {
    reader := kafka.NewReader(kafka.ReaderConfig{
        Brokers: brokers,
        Topic:   "orders",
        GroupID: "notification-service",
        // Matikan auto-commit
        CommitInterval: 0, // manual commit saja
    })
    defer reader.Close()

    for {
        msg, err := reader.FetchMessage(ctx)
        if err != nil {
            if ctx.Err() != nil {
                return
            }
            fmt.Printf("fetch error: %v\n", err)
            continue
        }

        // Proses message
        if err := processMessage(ctx, msg); err != nil {
            // Jangan commit — message akan di-retry
            fmt.Printf("process error at offset %d: %v\n", msg.Offset, err)
            // Bisa: kirim ke DLQ atau stop consumer
            continue
        }

        // Commit hanya setelah berhasil diproses
        if err := reader.CommitMessages(ctx, msg); err != nil {
            fmt.Printf("commit error: %v\n", err)
        }
    }
}
```

### Event Router — Dispatch ke Handler yang Tepat

```go
// pkg/kafka/router.go
package kafka

import (
    "context"
    "encoding/json"
    "fmt"
)

// EventRouter mendistribusikan messages ke handler berdasarkan event type
type EventRouter struct {
    handlers map[string]TypedHandler
}

type TypedHandler func(ctx context.Context, payload []byte) error

func NewEventRouter() *EventRouter {
    return &EventRouter{handlers: make(map[string]TypedHandler)}
}

// On mendaftarkan handler untuk event type tertentu
func (r *EventRouter) On(eventType string, handler TypedHandler) {
    r.handlers[eventType] = handler
}

// Route adalah MessageHandler yang bisa dipakai oleh Consumer
func (r *EventRouter) Route(ctx context.Context, msg ConsumedMessage) error {
    eventType := msg.Headers["event-type"]
    if eventType == "" {
        // Fallback: coba parse dari payload
        var base struct {
            EventType string `json:"event_type"`
        }
        if err := json.Unmarshal(msg.Value, &base); err == nil {
            eventType = base.EventType
        }
    }

    if eventType == "" {
        return fmt.Errorf("missing event-type in message (offset: %d)", msg.Offset)
    }

    handler, ok := r.handlers[eventType]
    if !ok {
        // Unknown event type — skip (bukan error)
        fmt.Printf("[router] Unknown event type: %s, skipping\n", eventType)
        return nil
    }

    return handler(ctx, msg.Value)
}

// Penggunaan di Notification Service
func setupNotificationConsumer(brokers []string) {
    router := kafka.NewEventRouter()

    // Daftarkan handler per event type
    router.On(events.EventOrderConfirmed, handleOrderConfirmed)
    router.On(events.EventOrderCancelled, handleOrderCancelled)
    router.On(events.EventOrderDelivered, handleOrderDelivered)

    consumer := kafka.NewConsumer(brokers, "orders", "notification-service-v1")
    
    ctx := context.Background()
    if err := consumer.Start(ctx, router.Route); err != nil {
        fmt.Printf("Consumer error: %v\n", err)
    }
}

func handleOrderConfirmed(ctx context.Context, payload []byte) error {
    var event events.OrderConfirmedEvent
    if err := json.Unmarshal(payload, &event); err != nil {
        return fmt.Errorf("unmarshal OrderConfirmedEvent: %w", err)
    }

    // Kirim email konfirmasi
    fmt.Printf("Sending confirmation email to %s for order %s\n",
        event.CustomerEmail, event.AggregateID)
    // emailService.SendOrderConfirmation(ctx, event)
    return nil
}
```

### 🏋️ Latihan 8.5

1. Buat `NotificationConsumer` yang consume topic `orders`. Gunakan `EventRouter` untuk dispatch ke handler yang berbeda per event type. Test dengan produce beberapa events dari test code dan verifikasi semua terhandle.
2. Test **consumer group behavior**: jalankan 2 instance consumer dengan group ID yang sama. Produce 6 messages ke topic dengan 3 partisi. Verifikasi bahwa setiap message hanya diproses oleh SATU consumer.
3. Implementasikan **graceful shutdown**: consumer harus selesai memproses message yang sedang berjalan sebelum stop. Test dengan send SIGTERM saat ada message yang diproses (tambahkan artificial delay di handler).

---

### Consumer Group Rebalance

Rebalance terjadi saat: consumer baru join, consumer crash/leave, atau topic partisi berubah.

```go
// pkg/kafka/consumer_with_rebalance.go
// kafka-go handles rebalance otomatis, tapi penting untuk dipahami

// Apa yang terjadi saat rebalance:
// 1. STOP - semua consumer PAUSE (tidak bisa consume!)
// 2. Coordinator memilih kembali partition assignment
//    Consumer 1 (baru): Partition 0, 1
//    Consumer 2:        Partition 2
// 3. RESUME - consumer melanjutkan dari offset yang di-commit terakhir
//
// MASALAH: jika ada uncommitted offset saat rebalance
//   Consumer 1 sedang proses message di offset 50, belum commit
//   Rebalance terjadi → partition 0 pindah ke Consumer 2
//   Consumer 2 start dari offset 49 (terakhir yang committed)
//   → MESSAGE DIPROSES 2X (duplicate)!
//
// SOLUSI: commit offset sesering mungkin dan buat consumer IDEMPOTENT

// Memonitor rebalance via stats
reader := kafka.NewReader(kafka.ReaderConfig{
    Brokers: brokers,
    Topic:   "orders",
    GroupID: "notification-service-v1",
    // CommitInterval: 0 → manual commit setelah setiap message
    CommitInterval: time.Second,
})

// Stats yang berguna untuk monitoring
go func() {
    ticker := time.NewTicker(10 * time.Second)
    for range ticker.C {
        stats := reader.Stats()
        fmt.Printf("Consumer Stats: lag=%d fetches=%d messages=%d bytes=%d\n",
            stats.Lag,
            stats.Fetches,
            stats.Messages,
            stats.Bytes,
        )
        // Expose sebagai Prometheus metrics!
        kafkaConsumerLag.WithLabelValues(stats.Topic, stats.Partition, stats.GroupID).
            Set(float64(stats.Lag))
    }
}()
```

### Kafka Consumer Lag Monitoring

Consumer lag = selisih antara offset TERBARU di partition vs offset yang sudah di-COMMIT consumer.
Lag tinggi = consumer ketinggalan = notifikasi terlambat!

```bash
# Cek consumer lag via CLI
kafka-consumer-groups \
  --bootstrap-server localhost:29092 \
  --describe \
  --group notification-service-v1

# Output:
# GROUP                    TOPIC   PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
# notification-service-v1  orders  0          1250            1250            0
# notification-service-v1  orders  1          980             985             5   ← lag!
# notification-service-v1  orders  2          1100            1100            0
```

```yaml
# Prometheus alert untuk consumer lag
groups:
  - name: kafka.alerts
    rules:
      - alert: KafkaConsumerHighLag
        expr: kafka_consumer_lag > 1000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Kafka consumer {{ $labels.group_id }} lag is high"
          description: "Consumer lag is {{ $value }} for topic {{ $labels.topic }}, partition {{ $labels.partition }}"

      - alert: KafkaConsumerVeryHighLag
        expr: kafka_consumer_lag > 10000
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Kafka consumer {{ $labels.group_id }} lag is critical"
```


## 📦 Modul 8.6 — RabbitMQ Fundamentals

### Arsitektur RabbitMQ

```
PRODUCER ──> EXCHANGE ──> QUEUE ──> CONSUMER
              (routing)   (buffer)
```

### Exchange Types

```
DIRECT EXCHANGE:
  Routing berdasarkan routing key yang EXACT match.
  
  Exchange "orders" (direct):
    routing_key="order.confirmed" ──> Queue "confirmed-orders"
    routing_key="order.cancelled" ──> Queue "cancelled-orders"
  
  Cocok untuk: routing ke service yang spesifik

FANOUT EXCHANGE:
  Kirim ke SEMUA queue yang terikat, abaikan routing key.
  
  Exchange "events" (fanout):
    ──> Queue "notification-queue"
    ──> Queue "analytics-queue"
    ──> Queue "audit-queue"
  
  Cocok untuk: broadcast event ke banyak consumer

TOPIC EXCHANGE:
  Routing berdasarkan routing key pattern (wildcard).
  * = satu kata
  # = nol atau lebih kata
  
  Exchange "logs" (topic):
    routing_key="order.#"    ──> Queue "all-order-events"
    routing_key="*.error"    ──> Queue "error-events"
    routing_key="payment.*"  ──> Queue "payment-events"
    routing_key="#"          ──> Queue "all-events" (catch-all)
  
  Cocok untuk: flexible routing dengan pattern

HEADERS EXCHANGE:
  Routing berdasarkan message headers, bukan routing key.
  Jarang dipakai — topic exchange lebih fleksibel.
```

### Queue Properties

```
DURABLE vs TRANSIENT:
  durable=true  → queue survive broker restart (disimpan ke disk)
  durable=false → queue hilang jika broker restart
  SELALU pakai durable=true di production!

EXCLUSIVE:
  exclusive=true → queue hanya bisa diakses oleh connection yang membuatnya
  Cocok untuk: temporary reply queues

AUTO-DELETE:
  auto_delete=true → queue dihapus saat tidak ada consumer lagi
  Cocok untuk: temporary work queues

ARGUMENTS:
  x-dead-letter-exchange  → kemana message yang gagal dikirim
  x-message-ttl           → berapa lama message disimpan (ms)
  x-max-length            → batas jumlah message di queue
  x-queue-mode: lazy      → simpan ke disk, hemat memory
```

### Message Properties & Persistence

```go
// Message properties yang penting untuk production

ch.PublishWithContext(ctx,
    "order-events",  // exchange
    "order.confirmed", // routing key
    false,           // mandatory
    false,           // immediate
    amqp.Publishing{
        // Persistence: message survive broker restart
        DeliveryMode: amqp.Persistent,  // 1=transient, 2=persistent

        // Content type untuk consumer tahu cara parse
        ContentType: "application/json",

        // Message ID untuk idempotency
        MessageId: uuid.New().String(),

        // Correlation ID untuk trace request end-to-end
        CorrelationId: correlationID,

        // Reply-to untuk RPC pattern
        ReplyTo: replyQueue,

        // TTL: message expired setelah 1 jam jika tidak diproses
        Expiration: "3600000", // milliseconds

        // Priority: 0-9 (butuh x-max-priority queue argument)
        Priority: 5,

        // Timestamp
        Timestamp: time.Now(),

        // App ID untuk identify publisher
        AppId: "order-service",

        // Custom headers
        Headers: amqp.Table{
            "event-type":   "order.confirmed",
            "event-version": "1",
            "source-service": "order-service",
        },

        Body: body,
    },
)
```

### Queue Arguments Penting

```go
// Queue dengan semua arguments production-ready
queue, err := ch.QueueDeclare(
    "order-processing",
    true,   // durable: survive broker restart
    false,  // auto-delete: jangan hapus saat consumer disconnect
    false,  // exclusive: bisa diakses banyak connection
    false,  // no-wait
    amqp.Table{
        // Dead Letter Exchange: kemana message yang gagal
        "x-dead-letter-exchange":    "order-dlx",
        "x-dead-letter-routing-key": "order-processing.dlq",

        // Max messages di queue (prevent memory exhaustion)
        "x-max-length": int32(10000),

        // Max total bytes
        "x-max-length-bytes": int64(100 * 1024 * 1024), // 100MB

        // Message TTL: hapus message setelah 24 jam jika tidak diproses
        "x-message-ttl": int32(86400000), // 24 jam dalam ms

        // Queue TTL: hapus queue jika tidak ada consumer
        // "x-expires": int32(3600000),

        // Lazy mode: simpan messages di disk (hemat memory, cocok untuk banyak messages)
        "x-queue-mode": "lazy",

        // Single Active Consumer: pastikan hanya 1 consumer aktif
        // (untuk ordered processing)
        // "x-single-active-consumer": true,
    },
)
```

### Management API — Monitoring via HTTP

```bash
# RabbitMQ punya REST API untuk monitoring
# Gunakan untuk health checks dan monitoring

# List semua queues dengan stats
curl -u admin:pass http://localhost:15672/api/queues

# Stats queue tertentu (termasuk consumer count, message count, dll)
curl -u admin:pass http://localhost:15672/api/queues/%2F/order-processing

# Publish message via management API (untuk testing)
curl -u admin:pass \
  -X POST http://localhost:15672/api/exchanges/%2F/order-events/publish \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {"delivery_mode": 2, "content_type": "application/json"},
    "routing_key": "order.confirmed",
    "payload": "{\"order_id\": 123}",
    "payload_encoding": "string"
  }'

# Overview cluster
curl -u admin:pass http://localhost:15672/api/overview

# Connection list
curl -u admin:pass http://localhost:15672/api/connections
```


### 🏋️ Latihan 8.6

1. Gambar **topology diagram** untuk e-commerce system menggunakan RabbitMQ: definisikan exchanges, queues, dan bindings. Gunakan setidaknya 2 exchange types yang berbeda.
2. Bandingkan kapan pakai Kafka vs RabbitMQ untuk 5 use case berbeda dalam e-commerce. Buat tabel keputusan dengan kolom: use case, pilihan, alasan utama.
3. Desain **Dead Letter Queue topology** untuk order processing: jika handler gagal 3x, message harus masuk DLQ untuk manual inspection. Gambar flow-nya.

---

## 📦 Modul 8.7 — RabbitMQ dengan Go

### Setup RabbitMQ

```yaml
# docker-compose.rabbitmq.yml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3.12-management-alpine
    container_name: rabbitmq
    restart: unless-stopped
    ports:
      - "5672:5672"    # AMQP port
      - "15672:15672"  # Management UI
    environment:
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_USER:-admin}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASS:-adminpassword}
      RABBITMQ_DEFAULT_VHOST: /
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 30s

volumes:
  rabbitmq-data:
```

```bash
# Install library
go get github.com/rabbitmq/amqp091-go

# Akses Management UI: http://localhost:15672
# Default: admin/adminpassword
```

### Connection dan Channel

```go
// pkg/rabbitmq/connection.go
package rabbitmq

import (
    "fmt"
    "sync"
    "time"

    amqp "github.com/rabbitmq/amqp091-go"
)

// Connection mengelola koneksi RabbitMQ dengan auto-reconnect
type Connection struct {
    mu       sync.RWMutex
    conn     *amqp.Connection
    url      string
    maxRetry int
}

func NewConnection(url string) (*Connection, error) {
    c := &Connection{url: url, maxRetry: 5}
    if err := c.connect(); err != nil {
        return nil, err
    }
    return c, nil
}

func (c *Connection) connect() error {
    var err error
    for i := 0; i < c.maxRetry; i++ {
        c.conn, err = amqp.Dial(c.url)
        if err == nil {
            fmt.Println("[rabbitmq] Connected successfully")
            return nil
        }
        delay := time.Duration(i+1) * 2 * time.Second
        fmt.Printf("[rabbitmq] Connection failed, retry %d/%d in %v: %v\n",
            i+1, c.maxRetry, delay, err)
        time.Sleep(delay)
    }
    return fmt.Errorf("failed to connect after %d attempts: %w", c.maxRetry, err)
}

// Channel mendapatkan channel baru dari koneksi
// PENTING: setiap goroutine harus punya channel sendiri!
func (c *Connection) Channel() (*amqp.Channel, error) {
    c.mu.RLock()
    defer c.mu.RUnlock()

    if c.conn == nil || c.conn.IsClosed() {
        return nil, fmt.Errorf("connection is closed")
    }
    return c.conn.Channel()
}

// Close menutup koneksi
func (c *Connection) Close() error {
    c.mu.Lock()
    defer c.mu.Unlock()
    if c.conn != nil && !c.conn.IsClosed() {
        return c.conn.Close()
    }
    return nil
}

// URL untuk RabbitMQ: amqp://user:pass@host:port/vhost
// Contoh: amqp://admin:adminpassword@localhost:5672/
```

### Publisher

```go
// pkg/rabbitmq/publisher.go
package rabbitmq

import (
    "context"
    "encoding/json"
    "fmt"
    "time"

    amqp "github.com/rabbitmq/amqp091-go"
)

// Publisher mempublish messages ke RabbitMQ exchange
type Publisher struct {
    conn     *Connection
    exchange string
    kind     string // direct, fanout, topic, headers
}

func NewPublisher(conn *Connection, exchange, kind string) (*Publisher, error) {
    p := &Publisher{conn: conn, exchange: exchange, kind: kind}
    
    // Declare exchange saat publisher dibuat
    ch, err := conn.Channel()
    if err != nil {
        return nil, err
    }
    defer ch.Close()

    if err := ch.ExchangeDeclare(
        exchange,
        kind,
        true,   // durable
        false,  // auto-deleted
        false,  // internal
        false,  // no-wait
        nil,    // arguments
    ); err != nil {
        return nil, fmt.Errorf("declare exchange %s: %w", exchange, err)
    }

    return p, nil
}

// Publish mengirim message ke exchange
func (p *Publisher) Publish(ctx context.Context, routingKey string, payload interface{}) error {
    // Setiap publish pakai channel baru (channel tidak thread-safe)
    ch, err := p.conn.Channel()
    if err != nil {
        return fmt.Errorf("get channel: %w", err)
    }
    defer ch.Close()

    body, err := json.Marshal(payload)
    if err != nil {
        return fmt.Errorf("marshal payload: %w", err)
    }

    return ch.PublishWithContext(ctx,
        p.exchange,  // exchange
        routingKey,  // routing key
        false,       // mandatory (return error jika tidak ada queue)
        false,       // immediate
        amqp.Publishing{
            ContentType:  "application/json",
            DeliveryMode: amqp.Persistent, // survive broker restart
            Timestamp:    time.Now(),
            Body:         body,
            Headers: amqp.Table{
                "event-type": routingKey,
                "source":     "ecommerce-system",
            },
        },
    )
}
```

### Consumer

```go
// pkg/rabbitmq/consumer.go
package rabbitmq

import (
    "context"
    "encoding/json"
    "fmt"

    amqp "github.com/rabbitmq/amqp091-go"
)

// Consumer membaca messages dari RabbitMQ queue
type Consumer struct {
    conn     *Connection
    queue    string
    prefetch int // jumlah message yang di-fetch sekaligus
}

func NewConsumer(conn *Connection, queueName string, prefetch int) (*Consumer, error) {
    c := &Consumer{conn: conn, queue: queueName, prefetch: prefetch}
    
    // Declare queue saat consumer dibuat
    ch, err := conn.Channel()
    if err != nil {
        return nil, err
    }
    defer ch.Close()

    if _, err := ch.QueueDeclare(
        queueName,
        true,   // durable — survive restart
        false,  // auto-deleted
        false,  // exclusive
        false,  // no-wait
        amqp.Table{
            // Dead Letter Exchange untuk pesan yang gagal
            "x-dead-letter-exchange": queueName + ".dlx",
            // Max retry sebelum ke DLQ
            "x-delivery-count": 3,
        },
    ); err != nil {
        return nil, fmt.Errorf("declare queue %s: %w", queueName, err)
    }

    return c, nil
}

// Start mulai membaca messages
func (c *Consumer) Start(ctx context.Context, handler func(context.Context, []byte) error) error {
    ch, err := c.conn.Channel()
    if err != nil {
        return err
    }
    defer ch.Close()

    // Prefetch: ambil N message sekaligus, tidak ambil baru sebelum ack yang lama
    // Ini penting untuk load balancing antar multiple consumers
    if err := ch.Qos(c.prefetch, 0, false); err != nil {
        return fmt.Errorf("set QoS: %w", err)
    }

    msgs, err := ch.Consume(
        c.queue,
        "",     // consumer tag (auto-generate)
        false,  // auto-ack=false → manual acknowledgment
        false,  // exclusive
        false,  // no-local
        false,  // no-wait
        nil,
    )
    if err != nil {
        return fmt.Errorf("start consuming: %w", err)
    }

    fmt.Printf("[consumer] Started consuming from queue '%s'\n", c.queue)

    for {
        select {
        case <-ctx.Done():
            fmt.Println("[consumer] Context cancelled, stopping")
            return nil

        case msg, ok := <-msgs:
            if !ok {
                return fmt.Errorf("channel closed")
            }

            // Process message
            if err := handler(ctx, msg.Body); err != nil {
                fmt.Printf("[consumer] Handler error: %v, nacking message\n", err)
                // Nack dengan requeue=true untuk retry
                // Nack dengan requeue=false → ke Dead Letter Queue
                msg.Nack(false, true) // false=single, true=requeue
                continue
            }

            // Acknowledge sukses
            msg.Ack(false)
        }
    }
}
```

### 🏋️ Latihan 8.7

1. Setup RabbitMQ dengan Docker Compose. Buka Management UI di `localhost:15672`. Buat exchange `order-events` (topic) dan queue `notification-queue`. Bind dengan routing key `order.#`. Publish test message dari Management UI.
2. Implementasikan `OrderEventPublisher` menggunakan RabbitMQ yang publish ke topic exchange dengan routing key `order.confirmed`, `order.cancelled`. Buat consumer yang subscribe ke `order.#` dan log setiap event.
3. Implementasikan **prefetch** yang tepat: buat 3 consumer instance dari queue yang sama. Kirim 9 messages dengan artificial processing time 1 detik. Verifikasi bahwa ketiga consumer bekerja paralel dan total waktu ~3 detik (bukan 9 detik).

---

## 📦 Modul 8.8 — RabbitMQ Patterns

### Work Queue Pattern

```go
// Work Queue: distribusi tasks ke multiple workers
// Setiap task dikerjakan oleh SATU worker

// Producer: submit jobs
func submitJobs(publisher *Publisher, jobs []Job) {
    for _, job := range jobs {
        publisher.Publish(ctx, "", job) // routing key kosong untuk default exchange
    }
}

// Worker (jalankan beberapa instance):
func startWorker(id int, consumer *Consumer) {
    consumer.Start(ctx, func(ctx context.Context, body []byte) error {
        var job Job
        json.Unmarshal(body, &job)
        
        fmt.Printf("Worker %d processing job: %s\n", id, job.ID)
        time.Sleep(time.Duration(job.Duration) * time.Second) // simulasi kerja
        return nil
    })
}

// Jalankan 3 worker:
// go startWorker(1, consumer1)
// go startWorker(2, consumer2)
// go startWorker(3, consumer3)
```

### Fanout Pattern (Event Broadcasting)

```go
// Setup fanout exchange untuk broadcast events
func setupFanout(conn *Connection) {
    ch, _ := conn.Channel()
    defer ch.Close()

    // Declare fanout exchange
    ch.ExchangeDeclare("events", "fanout", true, false, false, false, nil)

    // Setiap service punya queue sendiri yang bind ke exchange
    // Notification Service:
    notifQ, _ := ch.QueueDeclare("notification.events", true, false, false, false, nil)
    ch.QueueBind(notifQ.Name, "", "events", false, nil) // routing key kosong untuk fanout

    // Analytics Service:
    analyticsQ, _ := ch.QueueDeclare("analytics.events", true, false, false, false, nil)
    ch.QueueBind(analyticsQ.Name, "", "events", false, nil)

    // Audit Service:
    auditQ, _ := ch.QueueDeclare("audit.events", true, false, false, false, nil)
    ch.QueueBind(auditQ.Name, "", "events", false, nil)
}

// Publisher kirim ke fanout exchange — semua queue menerima
func publishEvent(publisher *Publisher, event interface{}) {
    publisher.Publish(ctx, "", event) // routing key diabaikan untuk fanout
}
```

### Topic Pattern (Flexible Routing)

```go
// Setup topic exchange
func setupTopicRouting(conn *Connection) {
    ch, _ := conn.Channel()
    defer ch.Close()

    ch.ExchangeDeclare("topic-events", "topic", true, false, false, false, nil)

    // Order notification queue: hanya terima events yang berhubungan dengan order
    orderNotifQ, _ := ch.QueueDeclare("order-notifications", true, false, false, false, nil)
    ch.QueueBind(orderNotifQ.Name, "order.#", "topic-events", false, nil)
    // Akan menerima: order.created, order.confirmed, order.cancelled, dll

    // Error monitoring queue: terima semua error events
    errorQ, _ := ch.QueueDeclare("error-monitor", true, false, false, false, nil)
    ch.QueueBind(errorQ.Name, "*.error", "topic-events", false, nil)
    // Akan menerima: order.error, payment.error, user.error, dll

    // Admin queue: terima SEMUA events
    adminQ, _ := ch.QueueDeclare("admin-all-events", true, false, false, false, nil)
    ch.QueueBind(adminQ.Name, "#", "topic-events", false, nil)
}

// Routing key yang sesuai pattern
func publishOrderConfirmed(pub *Publisher, event OrderConfirmedEvent) {
    pub.Publish(ctx, "order.confirmed", event)
    // Diterima oleh: order-notifications, admin-all-events
}

func publishPaymentError(pub *Publisher, event PaymentErrorEvent) {
    pub.Publish(ctx, "payment.error", event)
    // Diterima oleh: error-monitor, admin-all-events
}
```

### Request-Reply (RPC over RabbitMQ)

```go
// RPC Pattern: request-reply dengan temporary queue
// Cocok untuk: query ke service lain yang butuh response

// Client (RPC caller)
type RPCClient struct {
    conn      *Connection
    exchange  string
    replyQueue string
    pending   sync.Map // map[correlationID]chan []byte
}

func (c *RPCClient) Call(ctx context.Context, routingKey string, request interface{}) ([]byte, error) {
    correlationID := uuid.New().String()
    responseCh := make(chan []byte, 1)
    c.pending.Store(correlationID, responseCh)
    defer c.pending.Delete(correlationID)

    ch, _ := c.conn.Channel()
    defer ch.Close()

    body, _ := json.Marshal(request)
    ch.PublishWithContext(ctx, c.exchange, routingKey, false, false,
        amqp.Publishing{
            ContentType:   "application/json",
            CorrelationId: correlationID,
            ReplyTo:       c.replyQueue,  // "please reply to this queue"
            Body:          body,
        },
    )

    select {
    case response := <-responseCh:
        return response, nil
    case <-ctx.Done():
        return nil, ctx.Err()
    case <-time.After(10 * time.Second):
        return nil, fmt.Errorf("RPC timeout")
    }
}
```

### 🏋️ Latihan 8.8

1. Implementasikan **Work Queue** untuk email sending: producer kirim `SendEmailJob` ke queue, jalankan 3 worker yang memproses secara paralel. Kirim 10 jobs dan verifikasi total waktu lebih singkat dari sequential.
2. Setup **Topic Exchange** dengan 3 routing patterns: `order.#` → notification queue, `*.error` → alert queue, `#` → audit queue. Test dengan publish berbagai events dan verifikasi routing yang benar.
3. Implementasikan **RPC pattern** untuk stock check: Order Service call Product Service via RabbitMQ RPC (bukan gRPC). Bandingkan latency dan complexity dengan gRPC approach.

---

## 📦 Modul 8.9 — Event-Driven Architecture Patterns

### Event-Driven Architecture (EDA)

```
Traditional (Request-Driven):
  Client asks: "Give me order status"
  Service replies: "Order is CONFIRMED"

Event-Driven:
  Something happens: "Order was CONFIRMED" (past tense!)
  All interested parties REACT to this fact

Perbedaan kunci:
  - Events adalah FAKTA yang sudah terjadi (immutable, past tense)
  - Commands adalah PERMINTAAN untuk melakukan sesuatu (future tense)
  - Queries adalah PERTANYAAN tentang state saat ini (present tense)

CQRS + Events:
  Command → "ConfirmOrder" → Domain Logic → State Change → Publish Event
  Event → Consumer → Update Read Model (projection)
  Query → Read from projected read model (denormalized, fast)
```

### Event Sourcing dengan Kafka

```go
// Event Sourcing: state sebagai urutan events
// Setiap perubahan disimpan sebagai event, BUKAN overwrite state

// Alih-alih UPDATE orders SET status='CONFIRMED' WHERE id=1
// Kita PUBLISH: OrderConfirmed{order_id: 1, confirmed_at: ...}
// Dan store event tersebut SECARA PERMANEN di Kafka

// Untuk mendapatkan state order saat ini:
// REPLAY semua events untuk order_id=1
//   OrderCreated → status=DRAFT
//   ItemAdded → items=[...]
//   OrderSubmitted → status=PENDING
//   OrderConfirmed → status=CONFIRMED
//   State saat ini = CONFIRMED

// Event store menggunakan Kafka dengan key = aggregate_id
type EventStore struct {
    producer *Producer
    consumer *Consumer
    topic    string
}

func (es *EventStore) AppendEvent(ctx context.Context, aggregateID string, event interface{}) error {
    return es.producer.Publish(ctx, kafka.Message{
        Key:     aggregateID, // semua events satu aggregate di partisi yang sama!
        Payload: event,
        Headers: map[string]string{
            "aggregate-id": aggregateID,
            "event-type":   getEventType(event),
        },
    })
}

// Rebuild state dari events
func (es *EventStore) ReplayEvents(ctx context.Context, aggregateID string) ([]interface{}, error) {
    // Baca dari offset 0 untuk aggregate ini
    // Kafka partition key memastikan semua events aggregate ada di partisi yang sama
    // ... implementation ...
    return nil, nil
}
```

### CQRS dengan Events

```go
// Command Side: write model
type OrderCommandHandler struct {
    orderRepo  repository.OrderCommandRepository
    eventPub   EventPublisher
}

func (h *OrderCommandHandler) Handle(ctx context.Context, cmd ConfirmOrderCommand) error {
    // Load aggregate
    order, err := h.orderRepo.FindByIDForUpdate(ctx, cmd.OrderID)
    if err != nil {
        return err
    }

    // Apply business logic
    if err := order.Confirm(cmd.PaymentID); err != nil {
        return err
    }

    // Save state
    if err := h.orderRepo.Save(ctx, order); err != nil {
        return err
    }

    // Publish events untuk update read models
    return h.eventPub.Publish(ctx, order.DomainEvents()...)
}

// Query Side: read model (projection)
type OrderProjection struct {
    db *gorm.DB
}

// Handles OrderConfirmed event → update read model
func (p *OrderProjection) HandleOrderConfirmed(ctx context.Context, payload []byte) error {
    var event OrderConfirmedEvent
    if err := json.Unmarshal(payload, &event); err != nil {
        return err
    }

    // Update denormalized read model (optimized for queries)
    return p.db.Model(&OrderSummary{}).
        Where("order_id = ?", event.AggregateID).
        Updates(map[string]interface{}{
            "status":       "CONFIRMED",
            "payment_id":   event.PaymentID,
            "confirmed_at": event.OccurredAt,
        }).Error
}

// Query: fast read dari projected read model
func (p *OrderProjection) GetOrderSummary(ctx context.Context, orderID uint64) (*OrderSummary, error) {
    var summary OrderSummary
    return &summary, p.db.First(&summary, "order_id = ?", orderID).Error
}
```

### Choreography vs Orchestration (Revisited)

```
CHOREOGRAPHY (EDA native):
  Services berkomunikasi via events, tidak ada koordinator.
  
  OrderService → OrderConfirmed event → [Kafka]
                     ▼
              InventoryService (listen) → update stock
              NotificationService (listen) → send email
              AnalyticsService (listen) → update dashboard
  
  ✅ Pros: Truly decoupled, easy to add new consumers
  ❌ Cons: Hard to trace full flow, saga compensation complex

ORCHESTRATION:
  Satu service koordinasi flow secara eksplisit.
  
  OrderService:
    1. Call InventoryService.ReserveStock()
    2. Call PaymentService.Charge()
    3. Call NotificationService.SendEmail()
  
  ✅ Pros: Clear flow, easy debugging
  ❌ Cons: Coupling ke banyak service, single point of failure
  
HYBRID (recommended untuk complex flows):
  Orchestration untuk critical path (checkout)
  Choreography untuk side effects (notifications, analytics)
```

### 🏋️ Latihan 8.9

1. Implementasikan **Event Store** sederhana menggunakan Kafka: method `AppendEvent(aggregateID, event)` dan `GetEvents(aggregateID) []Event`. Test dengan sequence events untuk satu order.
2. Buat **OrderProjection** yang consume events dari Kafka dan update read model di PostgreSQL. Test: publish beberapa events, lalu query read model dan verifikasi state konsisten.
3. Buat **Event Replay** function: function yang membaca semua events dari awal untuk satu aggregate dan rebuild state dari scratch. Ini berguna untuk debugging dan recovery.

---

## 📦 Modul 8.10 — Outbox Pattern

### Masalah: Dual Write Problem

```
MASALAH tanpa Outbox:

func confirmOrder(ctx context.Context, order *Order) error {
    // Step 1: Save ke database
    db.Save(order)  // berhasil

    // Step 2: Publish event
    kafka.Publish(event)  // GAGAL! (network error, kafka down)

    // Akibat: order tersimpan sebagai CONFIRMED
    // tapi event tidak sampai ke consumer
    // → Notification Service tidak kirim email
    // → Inventory Service tidak update stok
    // → DATA INCONSISTENT!
}
```

### Solusi: Transactional Outbox Pattern

```
IDEA:
  Simpan event di tabel outbox dalam TRANSAKSI YANG SAMA dengan business logic.
  Background process yang terpisah akan publish event dari outbox ke Kafka.

  Database Transaction:
    BEGIN
    UPDATE orders SET status='CONFIRMED'
    INSERT INTO order_outbox (event_type, payload, published=false)
    COMMIT
  
  Background publisher (Outbox Worker):
    SELECT * FROM order_outbox WHERE published=false
    FOR EACH event:
      kafka.Publish(event)
      UPDATE order_outbox SET published=true
  
  Dijamin: jika order tersimpan, event PASTI dipublish (eventual)
```

### Implementasi Outbox

```go
// internal/infrastructure/persistence/postgres/model/outbox_model.go
package model

import "time"

// OutboxMessage adalah event yang menunggu dipublish
type OutboxMessage struct {
    ID          uint64    `gorm:"primarykey;autoIncrement"`
    CreatedAt   time.Time `gorm:"not null;default:NOW()"`
    EventType   string    `gorm:"size:100;not null;index"`
    AggregateID string    `gorm:"size:100;not null;index"`
    Payload     string    `gorm:"type:jsonb;not null"`
    Published   bool      `gorm:"not null;default:false;index"`
    PublishedAt *time.Time
    RetryCount  int       `gorm:"not null;default:0"`
}
```

```go
// internal/infrastructure/outbox/outbox_repository.go
package outbox

import (
    "context"
    "encoding/json"
    "fmt"
    "time"

    "gorm.io/gorm"
)

type OutboxRepository struct {
    db *gorm.DB
}

func NewOutboxRepository(db *gorm.DB) *OutboxRepository {
    return &OutboxRepository{db: db}
}

// Save menyimpan event ke outbox DALAM TRANSAKSI yang sama
func (r *OutboxRepository) Save(ctx context.Context, tx *gorm.DB, event DomainEvent) error {
    payload, err := json.Marshal(event)
    if err != nil {
        return fmt.Errorf("marshal event: %w", err)
    }

    msg := &OutboxMessage{
        EventType:   event.EventType(),
        AggregateID: event.AggregateID(),
        Payload:     string(payload),
        Published:   false,
    }

    // Gunakan tx (transaction) yang sama dengan business logic!
    return tx.WithContext(ctx).Create(msg).Error
}

// GetUnpublished mengambil events yang belum dipublish
func (r *OutboxRepository) GetUnpublished(ctx context.Context, limit int) ([]*OutboxMessage, error) {
    var messages []*OutboxMessage
    err := r.db.WithContext(ctx).
        Where("published = false AND retry_count < 5").
        Order("created_at ASC").
        Limit(limit).
        Find(&messages).Error
    return messages, err
}

// MarkPublished menandai event sudah dipublish
func (r *OutboxRepository) MarkPublished(ctx context.Context, id uint64) error {
    now := time.Now()
    return r.db.WithContext(ctx).Model(&OutboxMessage{}).
        Where("id = ?", id).
        Updates(map[string]interface{}{
            "published":    true,
            "published_at": now,
        }).Error
}

// IncrementRetry menambah retry counter (untuk events yang gagal dipublish)
func (r *OutboxRepository) IncrementRetry(ctx context.Context, id uint64) error {
    return r.db.WithContext(ctx).Model(&OutboxMessage{}).
        Where("id = ?", id).
        UpdateColumn("retry_count", gorm.Expr("retry_count + 1")).Error
}
```

```go
// internal/infrastructure/outbox/publisher_worker.go
package outbox

import (
    "context"
    "encoding/json"
    "fmt"
    "time"
)

// PublisherWorker adalah background worker yang publish events dari outbox ke Kafka
type PublisherWorker struct {
    outboxRepo *OutboxRepository
    kafkaPub   KafkaPublisher
    interval   time.Duration
    batchSize  int
}

type KafkaPublisher interface {
    Publish(ctx context.Context, topic, key string, payload []byte) error
}

func NewPublisherWorker(outboxRepo *OutboxRepository, kafkaPub KafkaPublisher) *PublisherWorker {
    return &PublisherWorker{
        outboxRepo: outboxRepo,
        kafkaPub:   kafkaPub,
        interval:   5 * time.Second,
        batchSize:  100,
    }
}

// Start menjalankan worker secara periodic
func (w *PublisherWorker) Start(ctx context.Context) {
    ticker := time.NewTicker(w.interval)
    defer ticker.Stop()

    fmt.Println("[outbox-worker] Started")

    for {
        select {
        case <-ctx.Done():
            fmt.Println("[outbox-worker] Stopped")
            return
        case <-ticker.C:
            if err := w.processOutbox(ctx); err != nil {
                fmt.Printf("[outbox-worker] Error: %v\n", err)
            }
        }
    }
}

func (w *PublisherWorker) processOutbox(ctx context.Context) error {
    messages, err := w.outboxRepo.GetUnpublished(ctx, w.batchSize)
    if err != nil {
        return fmt.Errorf("get unpublished: %w", err)
    }

    if len(messages) == 0 {
        return nil // nothing to do
    }

    fmt.Printf("[outbox-worker] Publishing %d events\n", len(messages))

    for _, msg := range messages {
        topic := eventTypeToTopic(msg.EventType)

        if err := w.kafkaPub.Publish(ctx, topic, msg.AggregateID, []byte(msg.Payload)); err != nil {
            fmt.Printf("[outbox-worker] Failed to publish event %d: %v\n", msg.ID, err)
            w.outboxRepo.IncrementRetry(ctx, msg.ID)
            continue
        }

        if err := w.outboxRepo.MarkPublished(ctx, msg.ID); err != nil {
            fmt.Printf("[outbox-worker] Failed to mark published %d: %v\n", msg.ID, err)
        }
    }

    return nil
}

// eventTypeToTopic memetakan event type ke Kafka topic
func eventTypeToTopic(eventType string) string {
    topicMap := map[string]string{
        "order.created":   "orders",
        "order.confirmed": "orders",
        "order.cancelled": "orders",
        "payment.success": "payments",
        "payment.failed":  "payments",
    }
    if topic, ok := topicMap[eventType]; ok {
        return topic
    }
    return "events" // default topic
}
```

```go
// Penggunaan di Use Case — tambahkan outbox save dalam transaksi
func (uc *confirmOrderUseCase) Execute(ctx context.Context, input ConfirmOrderInput) error {
    return uc.db.Transaction(func(tx *gorm.DB) error {
        // Load order
        order, err := uc.orderRepo.FindByIDForUpdate(ctx, input.OrderID)
        if err != nil {
            return err
        }

        // Business logic
        if err := order.Confirm(input.PaymentID); err != nil {
            return err
        }

        // Save order state
        if err := tx.Save(toOrderModel(order)).Error; err != nil {
            return err
        }

        // KUNCI: simpan events ke outbox dalam TRANSAKSI YANG SAMA
        for _, event := range order.DomainEvents() {
            if err := uc.outboxRepo.Save(ctx, tx, event); err != nil {
                return err
            }
        }

        // Transaksi commit → order tersimpan + events di outbox
        // Worker akan publish ke Kafka secara eventual
        return nil
    })
}
```

### 🏋️ Latihan 8.10

1. Implementasikan Outbox Pattern lengkap: `OutboxRepository`, `PublisherWorker`, dan integrasi di `ConfirmOrderUseCase`. Test: konfirmasi order, stop Kafka, verifikasi order tersimpan. Start Kafka kembali, verifikasi event akhirnya dipublish.
2. Tambahkan **idempotency** di OutboxWorker: sebelum publish, cek apakah message dengan ID yang sama sudah pernah dipublish sebelumnya (pakai event ID di Kafka header). Ini penting untuk saat worker restart.
3. Buat **cleanup job** yang hapus outbox messages yang sudah published lebih dari 7 hari (untuk mencegah tabel tumbuh tak terbatas). Jadwalkan berjalan setiap jam.

---

## 📦 Modul 8.11 — Dead Letter Queue & Error Handling

### Masalah: Poison Messages

```
POISON MESSAGE: message yang selalu gagal diproses
  - Format JSON tidak valid
  - Data tidak konsisten (product_id tidak exist)
  - Bug di consumer code

Tanpa DLQ:
  Consumer retry → retry → retry → queue penuh → SEMUA message stuck!

Dengan DLQ:
  Consumer retry 3x → gagal → pindah ke Dead Letter Queue
  Queue utama tetap jalan → consumer terus proses message lain
  DLQ bisa di-inspect manual dan message bisa di-retry atau dibuang
```

### Dead Letter Queue di Kafka

```go
// Kafka tidak punya DLQ built-in, tapi bisa implementasi manual

// pkg/kafka/dlq_consumer.go
package kafka

// DLQConfig mengkonfigurasi DLQ behavior
type DLQConfig struct {
    MaxRetries  int
    DLQTopic    string // topic untuk dead letters
    RetryTopics []string // topics untuk retry dengan delay
}

// DLQConsumer adalah consumer dengan DLQ support
type DLQConsumer struct {
    consumer   *Consumer
    dlqProd    *Producer
    retryProd  *Producer
    cfg        DLQConfig
    retryStore map[string]int // messageKey → retry count (in-memory, simplification)
    mu         sync.Mutex
}

func (c *DLQConsumer) Start(ctx context.Context, handler MessageHandler) error {
    return c.consumer.Start(ctx, func(ctx context.Context, msg ConsumedMessage) error {
        err := handler(ctx, msg)
        if err == nil {
            return nil // sukses
        }

        // Handler gagal — cek retry count
        retryKey := fmt.Sprintf("%s-%d", msg.Key, msg.Partition)
        c.mu.Lock()
        c.retryStore[retryKey]++
        count := c.retryStore[retryKey]
        c.mu.Unlock()

        if count <= c.cfg.MaxRetries {
            fmt.Printf("[dlq] Message failed (attempt %d/%d), will retry\n", count, c.cfg.MaxRetries)
            return err // return error → offset tidak di-commit → Kafka re-deliver
        }

        // Habis retry — kirim ke DLQ
        fmt.Printf("[dlq] Message failed %d times, sending to DLQ topic: %s\n",
            count, c.cfg.DLQTopic)

        dlqPayload := DLQMessage{
            OriginalTopic: msg.Topic,
            OriginalKey:   msg.Key,
            Payload:       msg.Value,
            Error:         err.Error(),
            RetryCount:    count,
            FailedAt:      time.Now(),
        }

        dlqErr := c.dlqProd.Publish(ctx, kafka.Message{
            Key:     msg.Key,
            Payload: dlqPayload,
        })

        if dlqErr != nil {
            fmt.Printf("[dlq] Failed to send to DLQ: %v\n", dlqErr)
            return dlqErr // tetap retry jika DLQ publish gagal
        }

        // Reset retry count dan commit offset — message dipindahkan ke DLQ
        c.mu.Lock()
        delete(c.retryStore, retryKey)
        c.mu.Unlock()

        return nil // commit offset, message sudah "handled" (via DLQ)
    })
}

type DLQMessage struct {
    OriginalTopic string    `json:"original_topic"`
    OriginalKey   string    `json:"original_key"`
    Payload       []byte    `json:"payload"`
    Error         string    `json:"error"`
    RetryCount    int       `json:"retry_count"`
    FailedAt      time.Time `json:"failed_at"`
}
```

### Dead Letter Queue di RabbitMQ

```go
// RabbitMQ punya DLQ built-in via x-dead-letter-exchange

func setupQueueWithDLQ(ch *amqp.Channel) error {
    // 1. Declare DLX (Dead Letter Exchange)
    if err := ch.ExchangeDeclare("orders.dlx", "direct", true, false, false, false, nil); err != nil {
        return err
    }

    // 2. Declare DLQ (Dead Letter Queue)
    if _, err := ch.QueueDeclare("orders.dlq", true, false, false, false, nil); err != nil {
        return err
    }

    // 3. Bind DLQ ke DLX
    if err := ch.QueueBind("orders.dlq", "orders.dlq", "orders.dlx", false, nil); err != nil {
        return err
    }

    // 4. Declare main queue dengan DLX configuration
    if _, err := ch.QueueDeclare(
        "orders",
        true,
        false,
        false,
        false,
        amqp.Table{
            // Jika message di-nack dengan requeue=false → ke DLX
            "x-dead-letter-exchange":    "orders.dlx",
            "x-dead-letter-routing-key": "orders.dlq",
            // Opsional: max retry sebelum ke DLQ
            "x-message-ttl": 30000, // 30 detik TTL
        },
    ); err != nil {
        return err
    }

    return nil
}

// Consumer yang pakai DLQ
func consumeWithDLQ(msgs <-chan amqp.Delivery, maxRetry int) {
    retryCount := make(map[string]int)

    for msg := range msgs {
        msgID := msg.MessageId
        if msgID == "" {
            msgID = string(msg.Body[:min(20, len(msg.Body))])
        }

        if err := processMessage(msg); err != nil {
            retryCount[msgID]++

            if retryCount[msgID] >= maxRetry {
                fmt.Printf("Message %s failed %d times, sending to DLQ\n", msgID, retryCount[msgID])
                // Nack dengan requeue=false → pesan ke DLX → DLQ
                msg.Nack(false, false)
                delete(retryCount, msgID)
            } else {
                fmt.Printf("Message %s failed (attempt %d), requeueing\n", msgID, retryCount[msgID])
                // Nack dengan requeue=true → kembali ke queue
                msg.Nack(false, true)
            }
            continue
        }

        // Sukses
        msg.Ack(false)
        delete(retryCount, msgID)
    }
}
```

### DLQ Inspector dan Replay

```go
// Tool untuk inspect DLQ messages dan retry yang berhasil diperbaiki

type DLQInspector struct {
    consumer *Consumer
    producer *Producer
}

// GetDLQMessages mengambil semua messages di DLQ tanpa hapus
func (d *DLQInspector) GetDLQMessages(ctx context.Context) ([]DLQMessage, error) {
    // ... baca dari DLQ topic tanpa commit offset ...
    return nil, nil
}

// ReplayMessage mengirim message dari DLQ kembali ke original topic
func (d *DLQInspector) ReplayMessage(ctx context.Context, dlqMsg DLQMessage) error {
    fmt.Printf("Replaying message from DLQ to topic: %s\n", dlqMsg.OriginalTopic)

    return d.producer.Publish(ctx, kafka.Message{
        Key:     dlqMsg.OriginalKey,
        Payload: json.RawMessage(dlqMsg.Payload),
        Headers: map[string]string{
            "replayed-from-dlq": "true",
            "original-error":    dlqMsg.Error,
        },
    })
}
```

### 🏋️ Latihan 8.11

1. Setup DLQ untuk Kafka consumer: message yang gagal 3x dipindahkan ke `orders.dlq` topic. Buat test yang sengaja throw error di handler, verifikasi message masuk DLQ setelah 3 retry.
2. Setup DLQ untuk RabbitMQ menggunakan `x-dead-letter-exchange`. Test: publish message dengan format salah, verifikasi masuk ke DLQ setelah 3 nack. Baca DLQ message dari Management UI.
3. Buat `DLQDashboard` endpoint: `GET /api/v1/admin/dlq` yang menampilkan count messages di setiap DLQ, dan `POST /api/v1/admin/dlq/replay` untuk replay semua messages yang ada.

---

## 📦 Modul 8.12 — Event Schema Design & Evolution

### Prinsip Schema Design

```
ATURAN EMAS:
  Events adalah kontrak antara producer dan consumer.
  Perubahan breaking bisa crash consumer yang tidak update!

SAFE CHANGES (backward compatible — consumer lama tetap berjalan):
  ✅ Tambah field baru dengan nilai default
  ✅ Rename field (tapi keep yang lama sebagai alias)
  ✅ Expand enum (tambah nilai baru)
  ✅ Ubah tipe ke yang lebih luas (int32 → int64)

BREAKING CHANGES (jangan lakukan!):
  ❌ Hapus field yang existing
  ❌ Rename field (tanpa alias)
  ❌ Ubah semantik field (price dari cent ke rupiah)
  ❌ Ubah tipe ke yang tidak kompatibel (string → int)
  ❌ Ubah nama event

STRATEGI untuk breaking changes:
  1. Schema versioning (EventV1, EventV2)
  2. Dual-publish: publish ke kedua schema selama transition
  3. Consumer-driven contract testing
```

### Versioning Events

```go
// events/versioned.go

// OrderConfirmedV1 — versi awal
type OrderConfirmedV1 struct {
    EventID     string    `json:"event_id"`
    EventType   string    `json:"event_type"`
    Version     int       `json:"version"` // 1
    AggregateID string    `json:"aggregate_id"`
    OccurredAt  time.Time `json:"occurred_at"`
    CustomerID  uint64    `json:"customer_id"`
    PaymentID   string    `json:"payment_id"`
    TotalAmount float64   `json:"total_amount"`
}

// OrderConfirmedV2 — tambah field items (breaking: consumer V1 ignore field ini, OK)
type OrderConfirmedV2 struct {
    EventID     string              `json:"event_id"`
    EventType   string              `json:"event_type"`
    Version     int                 `json:"version"` // 2
    AggregateID string              `json:"aggregate_id"`
    OccurredAt  time.Time           `json:"occurred_at"`
    CustomerID  uint64              `json:"customer_id"`
    CustomerEmail string            `json:"customer_email"` // NEW - V2
    PaymentID   string              `json:"payment_id"`
    TotalAmount float64             `json:"total_amount"`
    Currency    string              `json:"currency"`        // NEW - V2
    Items       []OrderItemV2       `json:"items"`           // NEW - V2
}

type OrderItemV2 struct {
    ProductID   uint64  `json:"product_id"`
    ProductName string  `json:"product_name"`
    Quantity    int     `json:"quantity"`
    UnitPrice   float64 `json:"unit_price"`
    Subtotal    float64 `json:"subtotal"`
}

// Consumer yang bisa handle kedua versi
func handleOrderConfirmed(ctx context.Context, payload []byte) error {
    // 1. Parse versi dulu
    var versionCheck struct {
        Version int `json:"version"`
    }
    json.Unmarshal(payload, &versionCheck)

    switch versionCheck.Version {
    case 1:
        var event OrderConfirmedV1
        if err := json.Unmarshal(payload, &event); err != nil {
            return err
        }
        return handleV1(ctx, event)

    case 2:
        var event OrderConfirmedV2
        if err := json.Unmarshal(payload, &event); err != nil {
            return err
        }
        return handleV2(ctx, event)

    default:
        // Unknown version — log dan skip (bukan error)
        fmt.Printf("Unknown event version: %d, skipping\n", versionCheck.Version)
        return nil
    }
}
```

### Envelope Pattern

```go
// Gunakan envelope untuk metadata yang konsisten

type EventEnvelope struct {
    // Metadata
    EventID       string    `json:"event_id"`        // UUID unik per event
    EventType     string    `json:"event_type"`      // "order.confirmed"
    Version       int       `json:"version"`         // schema version
    AggregateID   string    `json:"aggregate_id"`
    AggregateName string    `json:"aggregate_name"`  // "Order"
    OccurredAt    time.Time `json:"occurred_at"`
    Source        string    `json:"source"`          // service yang publish
    CorrelationID string    `json:"correlation_id"`  // untuk tracing

    // Payload (bisa berbeda per event type)
    Data json.RawMessage `json:"data"`
}

// Helper untuk wrap event dalam envelope
func WrapEvent(event interface{}, eventType, aggregateID, source, correlationID string) (*EventEnvelope, error) {
    data, err := json.Marshal(event)
    if err != nil {
        return nil, err
    }

    return &EventEnvelope{
        EventID:       uuid.New().String(),
        EventType:     eventType,
        Version:       1,
        AggregateID:   aggregateID,
        AggregateName: extractAggregateName(eventType), // "order" dari "order.confirmed"
        OccurredAt:    time.Now().UTC(),
        Source:        source,
        CorrelationID: correlationID,
        Data:          data,
    }, nil
}

// Unwrap dan parse payload
func (e *EventEnvelope) UnmarshalData(target interface{}) error {
    return json.Unmarshal(e.Data, target)
}
```

### 🏋️ Latihan 8.12

1. Buat `EventEnvelope` dan gunakan untuk semua events di order service. Verifikasi bahwa setiap event punya: event_id unik, version, occurred_at, source, correlation_id.
2. Simulasikan **schema evolution**: publish `OrderConfirmedV1`. Kemudian upgrade ke `OrderConfirmedV2` (tambah field `items`). Buat consumer yang bisa handle kedua versi. Verifikasi consumer tidak crash saat menerima V1 setelah upgrade.
3. Buat **Consumer Contract Test**: tulis test yang memastikan bahwa format event yang dipublish Order Service sesuai dengan yang diharapkan Notification Service. Ini seperti API contract testing tapi untuk events.

---

## 📦 Modul 8.13 — Integrasi dengan Microservices (Fase 7)

### Menghubungkan Semua Service

Di Fase 7 kita sudah punya 5 service yang berkomunikasi via gRPC dan HTTP. Sekarang kita tambahkan Kafka untuk komunikasi asynchronous.

```
SEBELUM (Fase 7):
  Order Service ──gRPC──> Product Service (check stock)
  Order Service ──gRPC──> Product Service (update stock)
  Order Service ──HTTP──> Notification Service (kirim email) ← SYNC!

SESUDAH (Fase 8):
  Order Service ──gRPC──> Product Service (check stock — TETAP SYNC)
  Order Service ──gRPC──> Product Service (update stock — TETAP SYNC)
  Order Service ──Kafka──> [orders topic] ──> Notification Service ← ASYNC!
                                          ──> Analytics Service    ← ASYNC!
                                          ──> Inventory Service    ← ASYNC!
```

### Architecture Update

```yaml
# docker-compose.yml — tambahkan Kafka
services:
  # ... existing services ...

  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
    networks:
      - backend

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
    networks:
      - backend

  notification-service:
    build:
      context: ./services/notification-service
    environment:
      KAFKA_BROKERS: kafka:29092
      KAFKA_GROUP_ID: notification-service-v1
      KAFKA_TOPICS: orders,payments
    depends_on:
      kafka:
        condition: service_healthy
    networks:
      - backend
```

### Order Service: Ganti Sync dengan Async

```go
// services/order-service/internal/usecase/confirm_order.go

// SEBELUM — sync call ke notification service
func (uc *confirmOrderUC) Execute(ctx context.Context, input Input) error {
    // ... confirm order logic ...

    // SYNC: order service harus tunggu notification service
    uc.notificationClient.SendOrderConfirmation(ctx, order) // BLOCKING!
    return nil
}

// SESUDAH — publish event, notification service handle async
func (uc *confirmOrderUC) Execute(ctx context.Context, input Input) error {
    return uc.db.Transaction(func(tx *gorm.DB) error {
        // Load dan confirm order
        order, _ := uc.orderRepo.FindByIDForUpdate(ctx, input.OrderID)
        order.Confirm(input.PaymentID)

        // Save order
        tx.Save(toModel(order))

        // Simpan events ke outbox (dalam transaksi yang sama!)
        for _, event := range order.DomainEvents() {
            uc.outboxRepo.Save(ctx, tx, event)
        }

        return nil
        // Transaksi commit → order confirmed + events di outbox
        // OutboxWorker akan publish ke Kafka → Notification Service akan handle
    })
    // Order Service TIDAK tunggu email terkirim — ASYNC!
}
```

### Notification Service: Full Event Consumer

```go
// services/notification-service/cmd/main.go
package main

import (
    "context"
    "fmt"
    "os"
    "os/signal"
    "syscall"
)

func main() {
    cfg := loadConfig()

    // Setup services
    emailService := email.NewSMTPService(cfg.SMTP)
    smsService := sms.NewTwilioService(cfg.Twilio)

    // Setup event router
    router := kafka.NewEventRouter()

    router.On(events.EventOrderConfirmed, func(ctx context.Context, payload []byte) error {
        var event events.OrderConfirmedEvent
        if err := json.Unmarshal(payload, &event); err != nil {
            return err
        }

        // Kirim email (async dari Order Service, tapi sync dari sini)
        if err := emailService.SendOrderConfirmation(ctx, email.OrderConfirmationData{
            To:          event.CustomerEmail,
            OrderID:     event.AggregateID,
            TotalAmount: event.TotalAmount,
            Items:       event.Items,
        }); err != nil {
            return fmt.Errorf("send email: %w", err)
        }

        // Kirim SMS
        smsService.SendSMS(ctx, event.CustomerPhone,
            fmt.Sprintf("Order #%s confirmed! Total: Rp%.0f", event.AggregateID, event.TotalAmount))

        return nil
    })

    router.On(events.EventOrderCancelled, func(ctx context.Context, payload []byte) error {
        var event events.OrderCancelledEvent
        if err := json.Unmarshal(payload, &event); err != nil {
            return err
        }

        return emailService.SendOrderCancellation(ctx, email.OrderCancellationData{
            To:      event.CustomerEmail,
            OrderID: event.AggregateID,
            Reason:  event.Reason,
        })
    })

    // Start consumer
    consumer := kafka.NewConsumer(
        strings.Split(cfg.Kafka.Brokers, ","),
        "orders",
        cfg.Kafka.GroupID,
    )

    ctx, cancel := context.WithCancel(context.Background())

    // Graceful shutdown
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)

    go func() {
        fmt.Printf("[notification-service] Starting event consumer\n")
        if err := consumer.Start(ctx, router.Route); err != nil {
            fmt.Printf("[notification-service] Consumer error: %v\n", err)
        }
    }()

    <-quit
    fmt.Println("[notification-service] Shutting down gracefully...")
    cancel()
    consumer.Close()
}
```

### Analytics Service: Event Projection

```go
// services/analytics-service/internal/consumer/order_analytics.go
package consumer

// Analytics Service mendengarkan events dan update dashboard

type OrderAnalyticsHandler struct {
    db *gorm.DB
}

func (h *OrderAnalyticsHandler) HandleOrderConfirmed(ctx context.Context, payload []byte) error {
    var event events.OrderConfirmedEvent
    json.Unmarshal(payload, &event)

    // Update revenue stats
    h.db.Exec(`
        INSERT INTO daily_revenue (date, total_orders, total_revenue)
        VALUES (?, 1, ?)
        ON CONFLICT (date) DO UPDATE
        SET total_orders = daily_revenue.total_orders + 1,
            total_revenue = daily_revenue.total_revenue + ?
    `, time.Now().Format("2006-01-02"), event.TotalAmount, event.TotalAmount)

    // Update product popularity
    for _, item := range event.Items {
        h.db.Exec(`
            INSERT INTO product_sales (product_id, total_sold, total_revenue)
            VALUES (?, ?, ?)
            ON CONFLICT (product_id) DO UPDATE
            SET total_sold = product_sales.total_sold + ?,
                total_revenue = product_sales.total_revenue + ?
        `, item.ProductID, item.Quantity, float64(item.Quantity)*item.UnitPrice,
            item.Quantity, float64(item.Quantity)*item.UnitPrice)
    }

    return nil
}
```

### End-to-End Flow dengan Kafka

```bash
# 1. Start semua services termasuk Kafka
docker compose up -d

# 2. Konfirmasi order (trigger OrderConfirmed event)
curl -X POST localhost:8080/internal/orders/1/payment \
  -d '{"payment_result":"success"}'

# 3. Lihat event di Kafka
docker exec kafka kafka-console-consumer \
  --topic orders \
  --from-beginning \
  --bootstrap-server localhost:29092

# Expected output:
# {"event_id":"abc123","event_type":"order.confirmed","version":1,...}

# 4. Verifikasi Notification Service menerima event
docker compose logs -f notification-service
# Expected: "Sending confirmation email to alice@test.com for order 1"

# 5. Verifikasi Analytics Service update
curl localhost:8084/analytics/daily-revenue
# Expected: revenue bertambah sesuai order amount
```

### 🏋️ Latihan 8.13

1. Update Order Service dari Fase 7: ganti sync call ke Notification Service dengan publish event ke Kafka topic `orders`. Implementasikan OutboxPattern. Verifikasi dengan test: konfirmasi order → event masuk Kafka → Notification Service log pesan.
2. Buat **Analytics Service** sederhana yang consume events `order.confirmed` dan `order.cancelled` dan update tabel `daily_stats`. Expose endpoint `GET /analytics/summary` yang menampilkan: total orders hari ini, total revenue, cancellation rate.
3. Buat **end-to-end test**: script yang verifikasi full async flow: order dibuat → confirmed → event sampai ke Notification Service (cek logs) → Analytics diupdate (cek endpoint). Test harus selesai dalam 30 detik.

---

## 🎯 Review & Checkpoint Fase 8

### Konseptual
- [ ] Jelaskan perbedaan Queue, Pub/Sub, dan Event Streaming
- [ ] Apa itu consumer group di Kafka dan bagaimana partisi dibagi?
- [ ] Apa perbedaan AT-LEAST-ONCE vs EXACTLY-ONCE delivery?
- [ ] Apa itu Outbox Pattern dan masalah apa yang diselesaikannya?
- [ ] Kapan message masuk ke Dead Letter Queue?
- [ ] Apa risiko breaking change di event schema dan bagaimana menghindarinya?
- [ ] Apa perbedaan Kafka dan RabbitMQ — kapan pakai masing-masing?

### Praktis
- [ ] Bisa setup Kafka dan RabbitMQ dengan Docker Compose
- [ ] Bisa membuat Kafka Producer dan Consumer dengan consumer group
- [ ] Bisa membuat RabbitMQ Publisher dengan topic exchange dan consumer dengan prefetch
- [ ] Bisa mengimplementasikan Outbox Pattern untuk guaranteed delivery
- [ ] Bisa setup Dead Letter Queue untuk menangani poison messages
- [ ] Bisa merancang event schema yang backward compatible
- [ ] **Menyelesaikan project Notification & Analytics System**

---

## 🎯 Project Akhir Fase 8

Kerjakan project berdasarkan PRD di: **`FASE-8-PRD-Notification-Analytics.md`**

---

*Setelah selesai Fase 8, lanjut ke `FASE-9-Observability.md`*

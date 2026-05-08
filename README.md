# KafkaConsumerProducer

A Spring Boot application that demonstrates real-time message streaming using Apache Kafka. Messages are published to a Kafka topic through a REST endpoint and consumed asynchronously by a dedicated listener service.
   
Built with Spring Boot and Spring Kafka.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│               Spring Boot Application                           │
│                (com.ankitamahajan)                              │
│                                                                 │
│  [HTTP Client]                                                  │
│       │                                                         │
│       ▼                                                         │
│  [REST Controller]     ┌─────────────────────┐                 │
│       │ publish        │    Kafka Broker      │                 │
│       ▼                │   localhost:9092     │                 │
│  [Producer Service] ──►│                     │                 │
│  KafkaTemplate.send()  │  Topic              │                 │
│                        │  ├── Partition 0    │──► [Consumer Service]
│                        │  └── Partition 1    │    @KafkaListener
│                        │                     │         │
│                        │  Zookeeper          │         ▼
│                        │  localhost:2181     │   [Message Model]
│                        └─────────────────────┘   JSON Deserializer
│                                                         │
│  [KafkaConfig] ── configures ──────────────────►        ▼
│  @EnableKafka, Beans                             [Logger / Output]
└─────────────────────────────────────────────────────────────────┘
```

### Component overview

| Component | Package | Responsibility |
|---|---|---|
| REST Controller | `com.ankitamahajan.controller` | Exposes HTTP endpoints to trigger message production |
| Producer Service | `com.ankitamahajan.service` | Sends messages to Kafka topic via `KafkaTemplate` |
| Kafka Config | `com.ankitamahajan.config` | Configures `ProducerFactory`, `ConsumerFactory`, and listener container |
| Consumer Service | `com.ankitamahajan.service` | Listens to Kafka topic using `@KafkaListener` and processes messages |
| Message Model | `com.ankitamahajan.model` | POJO representing the Kafka message payload (JSON) |

---

## Tech stack

- **Java 17+**
- **Spring Boot 3.x**
- **Spring Kafka**
- **Apache Kafka**
- **Apache Zookeeper**
- **Maven**

---

## Prerequisites

Make sure the following are installed and running before starting the application.

**Install Kafka** (macOS with Homebrew):

```bash
brew install kafka
```

**Start Zookeeper:**

```bash
zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties
```

**Start Kafka broker:**

```bash
kafka-server-start /usr/local/etc/kafka/server.properties
```

**Create the Kafka topic:**

```bash
kafka-topics --create \
  --topic my-topic \
  --bootstrap-server localhost:9092 \
  --partitions 2 \
  --replication-factor 1
```

---

## Getting started

**Clone the repository:**

```bash
git clone https://github.com/theankitamahajan/KafkaConsumerProducer.git
cd KafkaConsumerProducer
```

**Build the project:**

```bash
mvn clean install
```

**Run the application:**

```bash
mvn spring-boot:run
```

The application starts on `http://localhost:8080`.

---

## Configuration

`src/main/resources/application.properties`:

```properties
# Kafka broker
spring.kafka.bootstrap-servers=localhost:9092

# Producer
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer

# Consumer
spring.kafka.consumer.group-id=ankita-consumer-group
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.trusted.packages=*
```

---

## Usage

**Publish a message:**

```bash
curl -X POST http://localhost:8080/api/publish \
  -H "Content-Type: application/json" \
  -d '{"id": 1, "message": "Hello from Ankita!"}'
```

**Expected console output:**

```
Received message: KafkaMessage{id=1, message='Hello from Ankita!'}
```

---

## Project structure

```
KafkaConsumerProducer/
├── config/
│   └── KafkaConfig.java
├── src/main/java/com/ankitamahajan/
│   ├── controller/
│   │   └── MessageController.java
│   ├── service/
│   │   ├── ProducerService.java
│   │   └── ConsumerService.java
│   └── model/
│       └── KafkaMessage.java
├── src/main/resources/
│   └── application.properties
└── pom.xml
```

---

## How it works

1. A client sends an HTTP POST request to the REST controller.
2. The controller calls `ProducerService`, which uses `KafkaTemplate` to publish a JSON message to the Kafka topic.
3. The Kafka broker stores the message across its partitions.
4. `ConsumerService` — annotated with `@KafkaListener` — picks up the message and processes it.
5. `KafkaConfig` wires up the serializer, deserializer, consumer group, and listener container factory.

---

## Running with Docker (optional)

```yaml
# docker-compose.yml
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  kafka:
    image: confluentinc/cp-kafka:7.4.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

```bash
docker-compose up -d
```

---

## Author

**Ankita Mahajan**
[github.com/theankitamahajan](https://github.com/theankitamahajan)

---

## License

[MIT](LICENSE)

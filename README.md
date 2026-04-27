# KafkaConsumerProducer

A Spring Boot application demonstrating Apache Kafka Producer and Consumer integration using Spring Kafka. Messages are published to a Kafka topic via a REST endpoint and consumed asynchronously by a listener service.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                  Spring Boot Application                        │
│                   (com.emreakin)                                │
│                                                                 │
│  [HTTP Client]                                                  │
│       │                                                         │
│       ▼                                                         │
│  [REST Controller]     ┌─────────────────────┐                 │
│       │ publish        │    Kafka Broker      │                 │
│       ▼                │   localhost:9092     │                 │
│  [Producer Service] ──►│                     │                 │
│  KafkaTemplate.send()  │  Topic              │                 │
│                        │  ├── Partition 0    │                 │
│                        │  └── Partition 1    │                 │
│                        │                     │──► [Consumer Service]
│                        │  Zookeeper          │    @KafkaListener
│                        │  localhost:2181     │         │
│                        └─────────────────────┘         ▼
│                                                  [Message Model]
│  [KafkaConfig] ─────────── configures ──────►   JSON Deserializer
│  @EnableKafka, Beans                                    │
│                                                         ▼
│                                                  [Logger / Output]
└─────────────────────────────────────────────────────────────────┘
```

### Component overview

| Component | Class | Responsibility |
|---|---|---|
| REST Controller | `*Controller.java` | Exposes HTTP endpoints to trigger message production |
| Producer Service | `*ProducerService.java` | Sends messages to Kafka topic via `KafkaTemplate` |
| Kafka Config | `KafkaConfig.java` | Configures `ProducerFactory`, `ConsumerFactory`, and listener container |
| Consumer Service | `*ConsumerService.java` | Listens to Kafka topic using `@KafkaListener` and processes messages |
| Message Model | `*.java` (model/dto) | POJO representing the Kafka message payload (JSON) |

---

## Tech stack

- **Java 17+**
- **Spring Boot 3.x**
- **Spring Kafka** (`spring-kafka`)
- **Apache Kafka** (local broker)
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

**Create the Kafka topic** (in a new terminal):

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

The application starts on `http://localhost:8080` by default.

---

## Configuration

Edit `src/main/resources/application.properties` (or `application.yml`) to match your environment:

```properties
# Kafka broker
spring.kafka.bootstrap-servers=localhost:9092

# Producer settings
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer

# Consumer settings
spring.kafka.consumer.group-id=my-consumer-group
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.trusted.packages=*
```

---

## Usage

**Send a message via REST:**

```bash
curl -X POST http://localhost:8080/api/publish \
  -H "Content-Type: application/json" \
  -d '{"id": 1, "message": "Hello Kafka!"}'
```

The consumer will receive the message and log it to the console:

```
Received message: KafkaMessage{id=1, message='Hello Kafka!'}
```

---

## Project structure

```
KafkaConsumerProducer/
├── config/
│   └── KafkaConfig.java          # Producer/consumer factory beans
├── src/main/java/com/emreakin/
│   ├── controller/               # REST endpoints
│   ├── service/
│   │   ├── ProducerService.java  # KafkaTemplate.send()
│   │   └── ConsumerService.java  # @KafkaListener
│   └── model/                    # Message POJO / DTO
├── src/main/resources/
│   └── application.properties
└── pom.xml
```

---

## How it works

1. A client sends an HTTP POST request to the REST controller.
2. The controller delegates to `ProducerService`, which uses `KafkaTemplate` to publish a JSON message to the configured Kafka topic.
3. The Kafka broker stores the message in one of the topic's partitions.
4. `ConsumerService` — annotated with `@KafkaListener` — receives the message from the broker and processes it (currently logs it to console).
5. `KafkaConfig` wires up the serializer, deserializer, consumer group, and listener container factory.

---

## Running with Docker (optional)

If you prefer not to install Kafka locally, use Docker Compose:

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

## Contributing

Pull requests are welcome. For major changes, please open an issue first.

---

## License

[MIT](LICENSE)

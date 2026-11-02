# Inventory Kafka application

This Kotlin/Spring Boot application exposes a GraphQL API for creating inventory items. The mutation publishes an Avro event to Kafka; a Kafka consumer stores received items in memory, where the `inventoryItems` query can read them.

## Run Kafka and Schema Registry locally

Docker Compose runs the pinned Kafka and Schema Registry images; no custom Kafka Dockerfile is needed:

```shell
docker compose up -d
```

Kafka is available at `localhost:9092`, and Schema Registry at `http://localhost:8081`. The application defaults to these endpoints, so start it with:

```shell
./gradlew bootRun
```

Kafka UI is available at `http://localhost:8082`. Open the `local` cluster and browse the `inventory-items` topic to inspect messages.

Create an item at `http://localhost:8080/graphql`:

```graphql
mutation {
  createInventoryItem(name: "Apples", quantity: 12) {
    id
    name
    quantity
  }
}
```

You can explore and run GraphQL queries visually in GraphiQL at `http://localhost:8080/graphiql`.

Query items after the consumer receives the Kafka event:

```graphql
query {
  inventoryItems {
    id
    name
    quantity
    createdAt
  }
}
```

The same create and list operations are available over REST. Creating an item returns `202 Accepted`; listing shows items after the Kafka consumer has processed their events:

```shell
curl -X POST http://localhost:8080/api/inventory-items \
  -H "Content-Type: application/json" \
  -d '{"name":"Oranges","quantity":8}'

curl http://localhost:8080/api/inventory-items
```

Run the repeatable Karate REST-to-Kafka end-to-end test while the app and Kafka are running:

```shell
./gradlew test --tests com.coles.poc.RestKafkaE2eTest
```

The test posts a uniquely named item, then polls the GET endpoint until that exact item is returned by the Kafka consumer (or fails after 20 seconds). Override the app URL with `-Dkarate.baseUrl=http://localhost:8080`.

## Demonstrate Kafka publish retries and circuit breaker

Publishing waits for Kafka's send acknowledgement. Each failed send is retried up to five total attempts with delays of 1, 3, 7, and 15 seconds. Five failed attempts within 60 seconds open the circuit. The open-state duration and half-open behavior use Resilience4j defaults.

The circuit breaker is configured and active for all Spring profiles. For a repeatable, simulated failure demo, enable the optional failure-switch property when starting the app (no profile change is needed):

```shell
./gradlew bootRun --args='--app.kafka.publish-demo.failure-switch-enabled=true'
```

Enable the failure switch, submit a publish request, then inspect the breaker status:

```shell
curl -X POST http://localhost:8080/api/demo/kafka-publishing/failure-mode \
  -H "Content-Type: application/json" \
  -d '{"enabled":true}'

curl -X POST http://localhost:8080/api/inventory-items \
  -H "Content-Type: application/json" \
  -d '{"name":"Circuit breaker demo","quantity":1}'

curl http://localhost:8080/api/demo/kafka-publishing
```

The status reports the failure switch and breaker state. Disable the switch with `{"enabled":false}` after the circuit opens; the breaker remains open until its configured recovery interval elapses. The failure switch and demo endpoints are disabled by default and only become available when explicitly enabled with `app.kafka.publish-demo.failure-switch-enabled=true`.

To demonstrate a real broker outage, start Kafka and the app with the failure-switch property enabled, then stop Kafka with `docker compose stop kafka`. Submit a publish request and inspect the same status endpoint; restart Kafka with `docker compose start kafka` to allow recovery. Producer request/delivery timeouts are kept short so failed sends can be counted within the one-minute window; override them with `KAFKA_PRODUCER_REQUEST_TIMEOUT_MS`, `KAFKA_PRODUCER_DELIVERY_TIMEOUT_MS`, and `KAFKA_PRODUCER_MAX_BLOCK_MS` if needed.

Run integration tests with `./gradlew test`. The integration test starts Kafka and Schema Registry using Testcontainers.

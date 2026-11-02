# Implementation plan

- [x] Review the existing GraphQL, Kafka, and inventory application flow.
- [x] Add `POST /api/inventory-items` to publish inventory items through the existing create use case.
- [x] Add `GET /api/inventory-items` to list items recorded by the Kafka consumer.
- [x] Document REST usage in `README.md`.
- [x] Compile and package the application with `compileKotlin bootJar`.
- [x] Verify a GraphQL-created message is consumed and visible through REST.
- [x] Verify a REST-created message is consumed and visible through GraphQL.
- [x] Confirm REST POST produces a Kafka record and REST GET returns the item after consumption.
- [x] Add and document a Karate end-to-end test to repeatably verify REST-to-Kafka-to-REST behavior.
- [x] Run the targeted Karate end-to-end test successfully against the running application and Kafka.
- [ ] Integration tests - intentionally skipped as requested.

The Karate REST end-to-end test passed. The app already running in the IDE was left untouched.

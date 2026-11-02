# Repository structure and file access register

This register records repository files inspected or created during this work. Consult it before reading or creating repository files, then update it after each file access or creation.

## Files read before this register was created

| File | Purpose |
|---|---|
| `build.gradle.kts` | Gradle plugins, dependencies, and test configuration |
| `src/main/kotlin/com/coles/poc/inventory/adapter/kafka/InventoryKafkaAdapter.kt` | Kafka producer and consumer adapter |
| `src/main/kotlin/com/coles/poc/inventory/config/InventoryConfiguration.kt` | Producer and application bean configuration |
| `README.md` | Local run instructions, APIs, and current demo/test documentation |
| `src/main/resources/application.yaml` | Spring Kafka configuration |
| `src/test/kotlin/com/coles/poc/RestKafkaE2eTest.kt` | Karate end-to-end test runner |
| `src/test/resources/com/coles/poc/rest-kafka-e2e.feature` | REST-to-Kafka end-to-end scenario |
| `src/main/kotlin/com/coles/poc/inventory/application/InventoryService.kt` | Inventory creation and publish contract |
| `src/main/kotlin/com/coles/poc/inventory/adapter/rest/InventoryRestController.kt` | REST create/list API |
| `src/main/kotlin/com/coles/poc/inventory/adapter/graphql/InventoryGraphQlController.kt` | GraphQL create/list API |
| `src/test/kotlin/com/coles/poc/TestPocApplication.kt` | Test application entry point |
| `compose.yaml` | Local Kafka, Schema Registry, and Kafka UI services |

## Files created

| File | Purpose |
|---|---|
| `repository structure.md` | This file-access and repository structure register |
| `.github/skills/read-files/SKILL.md` | Repository-local Copilot skill requiring consultation of this register |
| `src/main/kotlin/com/coles/poc/inventory/adapter/kafka/KafkaPublishingDemoController.kt` | Opt-in failure toggle and breaker status endpoint |
| `src/test/kotlin/com/coles/poc/inventory/adapter/kafka/InventoryKafkaPublishResilienceTest.kt` | Focused retry-delay and circuit-breaker tests |

## Files modified

| File | Change |
|---|---|
| `src/main/kotlin/com/coles/poc/inventory/adapter/kafka/InventoryKafkaAdapter.kt` | Wait for Kafka send results and execute each send through retry and circuit-breaker policies |
| `src/main/kotlin/com/coles/poc/inventory/config/InventoryConfiguration.kt` | Configure profile-independent publish resilience and opt-in failure-mode bean |
| `src/main/resources/application.yaml` | Configure bounded Kafka producer timeouts |
| `README.md` | Document simulated and real-outage demonstrations |

## Files inspected during review

| File | Purpose |
|---|---|
| `build.gradle.kts` | Existing build and test dependencies/configuration |
| `src/main/kotlin/com/coles/poc/inventory/adapter/kafka/InventoryKafkaAdapter.kt` | Blocking Kafka send, retry/circuit-breaker interaction, and shared failure window |
| `src/main/kotlin/com/coles/poc/inventory/adapter/kafka/KafkaPublishingDemoController.kt` | Concurrent demo failure toggle and status endpoint |
| `src/main/kotlin/com/coles/poc/inventory/config/InventoryConfiguration.kt` | Retry/circuit-breaker policies and producer timeout configuration |
| `src/main/resources/application.yaml` | Kafka producer timeout defaults and environment overrides |
| `src/test/kotlin/com/coles/poc/inventory/adapter/kafka/InventoryKafkaPublishResilienceTest.kt` | Resilience retry and circuit-breaker regression coverage |
| `src/main/kotlin/com/coles/poc/inventory/application/InventoryService.kt` | Publish call contract and item creation flow |
| `src/main/kotlin/com/coles/poc/inventory/adapter/rest/InventoryRestController.kt` | Synchronous REST create endpoint behavior |
| `src/main/kotlin/com/coles/poc/inventory/config/InventoryConfiguration.kt` | Profile-independent resilience beans and existing failure-mode bean wiring |
| `src/main/kotlin/com/coles/poc/inventory/adapter/kafka/KafkaPublishingDemoController.kt` | Demo-only controller and failure-mode profile gates |
| `src/main/kotlin/com/coles/poc/inventory/adapter/kafka/InventoryKafkaAdapter.kt` | Failure-mode dependency and breaker execution flow |
| `src/main/kotlin/com/coles/poc/inventory/adapter/kafka/InventoryKafkaPublishResilienceTest.kt` | Existing retry and breaker tests |
| `README.md` | Profile-specific demo instructions |
| `.github/skills/read-files/SKILL.md` | File-register workflow instructions |

## Activity log

- Created this register and the `read-files` skill; recorded the pre-existing files already inspected in this session.
- Read `repository structure.md` to consult the register before further repository file reads.
- Created the demo controller and focused resilience test.
- Modified the Kafka adapter, configuration, application settings, and README for publish resilience and demo instructions.
- Updated this register to record the new and modified files.
- Ran the focused resilience test successfully. The complete suite's REST-to-Kafka end-to-end test failed while the app was unavailable on port 8080; Kafka and Schema Registry ports were reachable.
- Documented that the demo switch is enabled with a property rather than a Spring profile.
- Ran whitespace/status checks over the modified files.
- The demo smoke test exposed that five failures did not necessarily trip Resilience4j's percentage-based threshold when a recent success was in the same time window. Updated the publisher to open after five actual failed attempts in its rolling 60-second failure window, and strengthened the test to cover a preceding success.
- Confirmed the demo failure switch now produces five recorded failures and opens the circuit.
- Reran the full test suite with the demo-profile app and local Kafka services available; it completed successfully.
- Stopped the temporary app and ran final whitespace and worktree checks.
- Read the file register, resilience configuration, demo controller, publisher adapter, resilience tests, README, and read-files skill to remove the demo-profile requirement while retaining safe opt-in demo behavior.
- Updated the demo failure switch from a profile gate to an explicit opt-in property, preserving the circuit breaker on every profile and keeping disruptive demo endpoints disabled by default.
- Verified the app exposes the opt-in demo endpoints without activating any special Spring profile.
- Ran the focused resilience test and complete test suite successfully after the refactor.
- Stopped the temporary app and confirmed the final changes have no whitespace errors.
- Read this register before performing a manual end-to-end Kafka and circuit-breaker smoke test.
- Checked Docker Compose service state and ports; the local Kafka and Schema Registry containers are already running and healthy.
- Started Kafka and Schema Registry, then started the app without a Spring profile and with the demo status switch explicitly enabled.
- Manually POSTed an inventory item and confirmed the consumer exposed the same ID and fields through GET (`38b4ec73-b57c-4a25-8483-f1ffce9c7d35`).
- Stopped Kafka and manually POSTed another item; the request failed as expected and the breaker endpoint reported `OPEN` with five failed calls.
- Restarted Kafka and confirmed it returned to `healthy`; stopped the temporary app after the smoke test.
- Inspected the current working-tree diff and the Kafka publishing path, resilience configuration, demo controller, related tests, and existing build configuration for correctness and concurrency defects.
- Reviewed the current diff and the Kafka publish/retry path, rolling failure window synchronization, atomic demo failure toggle, configuration, tests, and REST publish flow; no concrete actionable defects were identified.

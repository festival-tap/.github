# festival-tap

---

**festival-tap** is a cloud-native, event-sourced self-service beer tap platform built using **Hexagonal Architecture**, **Java**, **Angular**, and **AWS Serverless infrastructure**.

Built as a polyrepo of single-responsibility modules, the project features executable behavioral specifications with black-box acceptance suites (`user-stories`) as its core source of value and quality gate, pure domain logic at the center (`core`), isolated adapters (`persistence`, `ingress`, `adapters`), a shared REST contract (`api-contract`), a web interface (`ui`), and an assembly root (`cloud-deploy`) for infrastructure orchestration.

---

## Table of Contents

<details open>
<summary><strong>Overview & Strategy</strong></summary>

* [Architectural Principles](#architectural-principles)
* [Economic & Ergonomic Impact](#economic--ergonomic-impact)
</details>

<details open>
<summary><strong>Repositories</strong></summary>

* [`festival-tap/user-stories`](#festival-tapuser-stories-executable-behavioral-specs--acceptance-suites) — Executable Behavioral Specs & Acceptance Suites
* [`festival-tap/api-contract`](#festival-tapapi-contract-design-first-api-contract--openapi-30--swagger) — Design-First API Contract (OpenAPI 3.0)
* [`festival-tap/ui`](#festival-tapui-external-client--angular-web-application) — Angular 21 Web Application
* [`festival-tap/core`](#festival-tapcore-hexagon-core--domain--ports) — Pure Java 24 Hexagon Core
* [`festival-tap/persistence`](#festival-tappersistence-outbound-persistence-adapters--jdk--aws) — Swappable JDK & AWS Persistence Adapters
* [`festival-tap/ingress`](#festival-tapingress-inbound-ingress-adapters--jdk--aws) — Swappable Pure JDK & AWS Serverless Ingress Adapters
* [`festival-tap/adapters`](#festival-tapadapters-miscellaneous-support-adapters--jdk--aws) — Swappable JDK & AWS Support Adapters
* [`festival-tap/cloud-deploy`](#festival-tapcloud-deploy-composition-root-e2e-runner--cloud-infrastructure) — Composition Root, E2E Runner & IaC
</details>

<details open>
<summary><strong>Development, Testing & Governance</strong></summary>

* [API Contracts & Consumer-Driven Pact Testing](#api-contracts--consumer-driven-pact-testing)
* [User Stories & End-to-End Acceptance Suites](#user-stories--end-to-end-acceptance-suites)
* [Outside-In Development Workflow](#outside-in-development-workflow)
* [Release Governance & Version Hygiene](#release-governance--version-hygiene)
* [Non-Redundant Testing Pyramid](#non-redundant-testing-pyramid)
</details>

<details open>
<summary><strong>Architecture & Cloud Infrastructure</strong></summary>

* [AWS Serverless Technology Stack](#aws-serverless-technology-stack)
* [Mapping to Classical Architecture & Request Flow](#mapping-to-classical-architecture--request-flow)
* [Serverless Scalability & Partitioning](#serverless-scalability--partitioning)
* [Observability & Production Readiness](#observability--production-readiness)
</details>

<details open>
<summary><strong>Deployment, Security & Usage</strong></summary>

* [Zero-Cost Delivery & Dry-Run Deployment Strategy](#zero-cost-delivery--dry-run-deployment-strategy)
* [Dry-Run to Cloud Promotion Flow](#dry-run-to-cloud-promotion-flow)
* [Access Control & Private Cloud Sandbox](#access-control--private-cloud-sandbox)
* [Using the Application](#using-the-application)
</details>

---

## Architectural Principles

* **Externalized Behavioral Verification:** Functional user stories (`us-01`–`us-08`) and black-box acceptance suites live in their own repository ([`festival-tap/user-stories`](https://github.com)), decoupled from both domain code and deployment infrastructure. This allows behavioral specifications to be authored independently, executed across multiple boundaries (from naked `core` ports to full HTTP/cloud endpoints), and imported by `cloud-deploy` as an external, tamper-proof quality gate.
* **Zero-Framework Backend:** The entire Java 24 backend operates without application frameworks (no Spring Boot, Micronaut, or Quarkus). It relies strictly on native Java 24 capabilities (Records, Sealed Interfaces, Pattern Matching, Virtual Threads, `jdk.httpserver`), pure constructor-based dependency injection, and official AWS SDKs—eliminating reflection overhead, framework lock-in, and AWS Lambda cold-start latency.
* **Cross-Project Infrastructure Reuse:** Generic, domain-agnostic infrastructure—such as event-sourcing and state-store persistence engines—is extracted into standalone libraries under the [`sugarcoat-labs`](https://github.com) organization (`persistence-engines-core`, `persistence-engines-dynamodb`). Keeping these engines decoupled from business domains turns complex storage and concurrency mechanics into battle-tested, TCK-verified building blocks ready for immediate reuse across multiple projects.
* **Autonomous Polyrepo Topology (Isolation & Parallelization):** Each architectural boundary lives in its own repository with an independent build, test, and release lifecycle. Designed to mirror a distributed multi-team engineering workflow, this separation enforces strict contract-first integration (OpenAPI + Pact), prevents accidental coupling, and allows foundation layers (`core`, `api-contract`) and adapter layers (`ui`, `persistence`, `ingress`, `adapters`) to evolve and verify in parallel.
* **AI-Native Repository Sizing & Context Purity:** Small, single-responsibility repositories directly maximize the accuracy and autonomy of AI coding agents (Copilot, Claude Code, Cursor). By eliminating cross-layer noise, the entire module fits cleanly inside an agent's active context window; strict compile-time dependency boundaries prevent architectural shortcuts; explicit contracts (`openapi.yaml`, Java interfaces) act as deterministic specifications; and sub-second local test loops enable rapid autonomous self-correction.

---

## Economic & Ergonomic Impact

Every architectural decision in `festival-tap` directly reduces cloud operating costs while improving developer and user ergonomics compared to traditional framework-heavy monoliths:

* **Eliminating the Serverless "Framework Tax" (Economic):** Cloud functions bill by **memory allocated × execution duration (GB-seconds)**. Traditional framework containers require 512 MB–1 GB of RAM and 3–6 seconds to scan annotations on cold starts—burning through free-tier quotas or forcing expensive, always-on *Provisioned Concurrency* instances. Pure Java 24 boots in **~150 ms** with minimal memory, ensuring **$0/month** cloud operation with zero pre-warmed servers.
* **Sub-Second Feedback Loops (Developer Ergonomics):** Swappable **JDK** factories (`Ingress.jdk()`, `Persistence.jdk()`, `Adapters.jdk()`) allow full end-to-end user stories—alongside isolated `Ingress.jdk()` Pact contract verifications—to run in milliseconds in pure Java, eliminating multi-minute Docker/Testcontainers startups during daily coding.
* **Instant Tap Responsiveness (User Ergonomics):** At a live self-service bar, a 5-second cold-start delay makes a dispenser feel broken. Sub-second Lambda initialization ensures immediate tap actuation without keeping idle infrastructure running overnight.
* **Zero-Friction AI Collaboration (Engineering Velocity):** Small, dependency-isolated repositories eliminate cross-layer noise, allowing AI coding agents to grasp 100% of a repository's context and generate accurate, leak-free code on the first pass.

---

## Repositories

### [`festival-tap/user-stories`](https://github.com) *(Executable Behavioral Specs & Acceptance Suites)*
A dedicated repository housing the human-readable functional specifications (`us-01` through `us-08`) alongside black-box cURL API scripts and frontend end-to-end verification suites.
* **Decoupled Behavioral Truth:** Isolates business acceptance criteria from cloud infrastructure and deployment wiring, allowing product requirements and acceptance tests to evolve independently.
* **Multi-Boundary Verification:** Designed to validate system behavior across different hexagonal boundaries—from direct core port verification to black-box HTTP/cURL and UI checks against the assembled application.
* **Versioned Quality Gate:** Tagged and imported as an external test suite by `cloud-deploy` during CI pipelines to gate staging and production promotions.

### [`festival-tap/api-contract`](https://github.com) *(Design-First API Contract — OpenAPI 3.0 / Swagger)*
The single source of truth defining the REST API contract between the frontend client (`ui`) and the ingress adapters (`ingress`), keeping HTTP and JSON schemas completely isolated from `core`.
* **OpenAPI 3.0 / Swagger Specifications:** Houses the versioned REST endpoint definitions, request/response schemas, and error status contracts.
* **Contract-First Artifact Generation:** Publishes generated DTOs/interfaces (via GitHub Packages / releases) so both `ingress` and `ui` build against the exact same API specification without drift.

### [`festival-tap/ui`](https://github.com) *(External Client — Angular Web Application)*
The standalone Angular 21 frontend providing the public self-service bar interface and the protected admin dashboard.
* **Public Bar & Admin Dashboards:** Delivers real-time tap pouring interfaces (with auto-close limits) alongside administrative tap configuration and spending reports.
* **Pact Consumer Contract Tests:** Consumes the OpenAPI spec from `api-contract` and executes **Pact consumer tests** to generate and publish executable HTTP contract files (`pact.json`) verified by `ingress`.
* **Static Asset Packaging:** Compiles optimized web bundles for static cloud hosting.

### [`festival-tap/core`](https://github.com) *(Hexagon Core — Domain & Ports)*
The pure-Java 24 heart of the Beer Tap Dispenser application holding all business rules, domain models, and port interfaces with zero external runtime dependencies.
* **Pure Domain Production Code:** `src/main` contains domain models (taps, pours, pricing), immutable events, inbound use-case interfaces, outbound port interfaces (including clock and ID ports), and domain business services with zero framework, persistence-engine, or cloud SDK dependencies.
* **Deterministic Testing:** `src/test` uses [`sugarcoat-labs/persistence-engines-core`](https://github.com) (`Jdk.inMemory()`) and lightweight test doubles in test scope to verify pricing rules, pour timeouts, and end-to-end business workflows with zero circular dependencies on downstream adapter repositories.
* **Boundary Enforcement & Publishing:** Enforces zero external imports in `src/main` via isolated Maven/JPMS module boundaries and publishes versioned JAR artifacts for downstream adapters.

### [`festival-tap/persistence`](https://github.com) *(Outbound Persistence Adapters — JDK & AWS)*
A single repository housing the thin outbound adapters that bridge `core` domain persistence ports (`TapEventPublishingPort`, `TapEventFetchingPort`, `TapRegistryPort`) to [`sugarcoat-labs/persistence-engines-core`](https://github.com) and [`sugarcoat-labs/persistence-engines-dynamodb`](https://github.com).
* **Unified Factory Selection:** Exposes a clean factory API allowing callers to select either the **JDK in-memory** realization (for tests and local runs) or the **AWS DynamoDB** realization (for cloud deployment) on demand.
* **Port Bridging & Exception Translation:** Maps `core` domain operations to `EventStreamEngine` and `StateStoreEngine`, translating storage-level concurrency and uniqueness errors into domain exceptions (`SequenceConflictException`, `DuplicateTapException`).
* **Domain Serialization Codecs:** Implements zero-framework `DynamoDbCodec<T>` serializers for domain records and events without polluting `core` with serialization annotations.

### [`festival-tap/ingress`](https://github.com) *(Inbound Ingress Adapters — JDK & AWS)*
A unified repository housing the driving adapters that translate external requests and events into invocations on `core` inbound use-case interfaces without framework annotations.
* **Unified Factory Selection:** Exposes a clean factory API to select either the **pure-JDK** HTTP server (built on Java's native `jdk.httpserver` and Virtual Threads for zero-dependency local execution) or the **AWS Serverless** handlers (for cloud deployment) on demand.
* **Synchronous REST & API Gateway Handlers:** Maps HTTP requests (tap registration, pour open/close commands, admin spending reports) to core use cases and translates domain responses into HTTP status codes.
* **Asynchronous Event & Workflow Consumers:** Implements lightweight AWS Lambda handlers for SQS queues, Step Functions task workers, and S3 event triggers.
* **Pact Provider Verification Tests:** Pulls the Pact contracts generated by `ui` and runs **Pact provider verification tests** directly against the pure-JDK HTTP server in CI to guarantee API compatibility.

### [`festival-tap/adapters`](https://github.com) *(Miscellaneous Support Adapters — JDK & AWS)*
A dedicated repository housing secondary outbound adapters for cross-cutting domain ports such as system clocks, deterministic test clocks, UUID/ID generators, and external cloud utilities (e.g., S3 receipt storage or event notifications).
* **Unified Factory Selection:** Exposes a clean factory API to supply **pure-JDK** realizations (fixed/system clocks, UUIDs) for unit and integration testing or **AWS** realizations for production on demand.
* **Deterministic Time & Identity:** Decouples business logic from `Instant.now()` and `UUID.randomUUID()` so time-sensitive rules (like pour duration and auto-close limits) remain 100% reproducible.

### [`festival-tap/cloud-deploy`](https://github.com) *(Composition Root, E2E Runner & Cloud Infrastructure)*
The assembly and deployment repository that wires all hexagonal layers together, defines the cloud architecture schema, runs imported acceptance suites, and automates releases.
* **Composition Root:** Wires `ingress`, `core`, `persistence`, and `adapters` (configured via their **JDK** factories for local E2E runs or **AWS** factories for cloud deployment) using pure Java constructor injection.
* **Cloud Infrastructure & Schemas:** Defines the serverless topology, DynamoDB schemas, API Gateway throttling, and Infrastructure as Code (IaC) templates.
* **Dual-Target Acceptance & Dry-Run CI/CD:** Imports [`festival-tap/user-stories`](https://github.com) to execute full-stack acceptance tests against the **pure-JDK** assembly first, followed by a local AWS emulator dry-run smoke test before deploying to the **AWS Always Free Tier**.

---

## API Contracts & Consumer-Driven Pact Testing

To prevent schema drift between the Angular frontend (`ui`) and the Java 24 backend (`ingress`) without coupling their release cycles or leaking HTTP details into `core`, integration boundaries are governed by a two-tier contract strategy:

* **Static Design-First Schema ([`festival-tap/api-contract`](https://github.com)):** Acts as the single source of truth for REST endpoints, request/response JSON payloads, and HTTP error codes via OpenAPI 3.0 specifications—generating typed client and server DTOs before implementation begins.
* **Consumer Contract Generation ([`festival-tap/ui`](https://github.com)):** During CI builds, Angular consumer tests execute against mock providers to record actual HTTP expectations into executable Pact contract files (`pact.json`), published to **PactFlow (Free Plan)**.
* **Provider Contract Verification ([`festival-tap/ingress`](https://github.com)):** Pulls the latest consumer Pact files and replays every interaction directly against the **pure-JDK HTTP server** (`Ingress.jdk()` on `jdk.httpserver` backed by in-memory fakes), guaranteeing 100% behavioral compatibility in milliseconds without Docker or cloud deployments.

---

## User Stories & End-to-End Acceptance Suites

Functional specifications (`us-01` through `us-08`) and executable black-box acceptance suites (cURL API scripts and frontend end-to-end checks) live in their own dedicated repository, **[`festival-tap/user-stories`](https://github.com)**, keeping behavioral requirements decoupled from both core domain code and cloud infrastructure:

* **Single Behavioral Truth:** Each user story (`us-01-create-dispenser` through `us-08-event-sourcing`) pairs human-readable requirements with automated verification scripts that exercise real HTTP endpoints, authentication headers, and event-sourced state transitions.
* **Multi-Boundary & Dual-Target Execution:** Imported by [`festival-tap/cloud-deploy`](https://github.com) during CI, the acceptance suite runs first against the **pure-JDK assembly** (`Ingress.jdk()` + `Persistence.jdk()` + `Adapters.jdk()`) for sub-second functional verification, and subsequently as a smoke check against the **AWS dry-run / cloud sandbox** before release promotion.
* **Independent QA & AI Context Isolation:** Separating user stories into a dedicated repository prevents deployment scripts from polluting functional specs and allows QA/product workflows (or AI agents) to author acceptance tests without touching IaC templates.

---

## Outside-In Development Workflow

Feature delivery follows a strict **Contract-First, Outside-In** progression driven directly by user stories and client entry points:

1. **Capture Client Intent (`api-contract` & `user-stories`):** Translate user stories into versioned OpenAPI 3.0 specifications (`openapi.yaml`), publishing generated DTOs and API contracts before writing implementation code.
2. **Design & Verify Domain Core (`core`):** Define inbound use cases, outbound ports, and immutable domain events required by the contract. Implement pure business rules, verify them with `Jdk.inMemory()`, and publish the versioned `core` JAR.
3. **Implement Adapters & Client in Parallel (`ingress`, `persistence`, `adapters`, `ui`):**
   * **`ingress`:** Bridges external REST/AWS triggers (`api-contract`) to `core` use cases via `Jdk` and `Aws` factories.
   * **`persistence` & `adapters`:** Implement `core` outbound ports using `sugarcoat-labs` engines, clocks, and access guards (`Jdk` and `Aws` factories).
   * **`ui`:** Builds Angular features against `api-contract` and publishes **Pact consumer** contracts verified by `ingress`.
4. **Assemble, Validate & Ship (`cloud-deploy`):** Wire the full application via pure constructor injection, run all `user-stories` acceptance tests against the **pure-JDK** assembly, execute the local AWS emulator dry-run smoke test, and deploy to AWS.

> For the sequential, critical-path build checklist across all eight repositories, see the **[Execution Roadmap](../ROADMAP.md)**.

---

## Release Governance & Version Hygiene

Operating an eight-repository polyrepo requires disciplined release mechanics so downstream consumers never drift out of sync, while preserving every working milestone permanently:

* **Semantic Versioning (SemVer) Across Repositories:** Every release of `core`, `api-contract`, `user-stories`, and the adapter modules follows strict `MAJOR.MINOR.PATCH` tagging (`v1.0.0`). Breaking schema or port changes increment the `MAJOR` version, while backward-compatible additions increment `MINOR` or `PATCH`.
* **Immutable Package & Tag Archiving:** Published libraries (`core`, `api-contract`, `persistence`, `ingress`, `adapters`) reside as immutable versioned artifacts in **GitHub Packages**, while `user-stories`, `ui`, and `cloud-deploy` are pinned via immutable Git release tags—keeping every milestone permanently preserved, auditable, and reproducible on demand.
* **Automated Dependency Propagation (Dependabot):** Downstream repositories (`persistence`, `ingress`, `adapters`, `ui`, `cloud-deploy`) run **GitHub Dependabot** to automatically open Pull Requests whenever upstream artifacts update, triggering isolated CI, Pact verification, and acceptance suites before merge.

---

## Non-Redundant Testing Pyramid

Because each architectural boundary is isolated in its own repository with swappable **JDK** and **AWS** factories, every layer is verified once at its fastest possible boundary without redundant intermediate assemblies:

* **Domain Unit & In-Memory Integration (`core`):** Verifies pure pricing rules, pour timeouts, and full event-sourcing flows in milliseconds using `Jdk.inMemory()` and lightweight test doubles—eliminating any need for a separate "core + outbound adapters" test assembly.
* **Sliced Adapter & Contract Verification (`persistence`, `ingress`, `adapters`, `ui`):** Each adapter repository tests its own translation logic in isolation—including **Pact consumer/provider** HTTP verification against `Ingress.jdk()` (backed by lightweight domain stubs) and **DynamoDB Local** codec tests for AWS persistence.
* **Pure-JDK End-to-End Acceptance (`cloud-deploy`):** Executes all functional user-story acceptance scripts (`user-stories`) against the full stack assembled with **pure-JDK factories** (`Ingress.jdk()` + `core` + `Persistence.jdk()` + `Adapters.jdk()`), delivering sub-second E2E validation with zero Docker or cloud overhead.
* **Cloud Wiring Smoke Test (`cloud-deploy`):** Runs a targeted dry-run smoke test with the **AWS factories** against a local cloud emulator solely to verify IAM permissions, Infrastructure as Code templates, and cloud serialization before live rollout.

---

## AWS Serverless Technology Stack

The cloud deployment uses a team of **AWS Always Free Tier** services—each plugged into an outer adapter so `core` stays 100% pure Java 24 with zero framework dependencies:

| Category | AWS Service | Analogs in Classical Architecture | Repository | Primary Architectural Role |
| :--- | :--- | :--- | :--- | :--- |
| **Ingress & Routing** | **Amazon API Gateway** | **Nginx / Spring Cloud Gateway** | `ingress` & `cloud-deploy` | Managed HTTP entry point, request routing, and rate limiting. |
| **Hardware Ingress *(Next Stage)*** | **AWS IoT Core** | **Mosquitto / EMQX MQTT Brokers** | `ingress` & `cloud-deploy` | Managed MQTT broker for physical tap hardware telemetry and actuation. |
| **Compute** | **AWS Lambda** | **Tomcat / Embedded Jetty / Netty** | `ingress` & `cloud-deploy` | On-demand, stateless Java 24 execution workers. |
| **Domain Persistence** | **Amazon DynamoDB** | **PostgreSQL + Hibernate / JPA** | `persistence` | Transactional database for append-only event streams and tap state. |
| **Static Files & Hosting** | **Amazon S3** | **Nginx Static Server / Local Disk** | `ui`, `adapters` & `ingress` | Static web hosting, immutable receipt archiving, and S3 event triggers. |
| **Async & Non-Blocking** | **Amazon SQS** | **RxJava / Project Reactor / Kafka / RabbitMQ** | `ingress` & `cloud-deploy` | Durable message queue that triggers background Lambda workers for non-blocking async jobs. |
| **State & Workflows** | **AWS Step Functions** | **Quartz Scheduler / Camunda** | `ingress` & `cloud-deploy` | Durable state machines for time-bound, multi-step lifecycles. |
| **Observability** | **Amazon CloudWatch** | **ELK Stack / Prometheus / Grafana** | `adapters` & `cloud-deploy` | Structured log aggregation, custom metrics, dashboards, and threshold alarms. |
| **Observability** | **AWS X-Ray** | **Jaeger / Zipkin / OpenTelemetry Agents** | `ingress` & `cloud-deploy` | End-to-end distributed request tracing and service maps. |
| **Observability** | **Amazon SNS** | **Alertmanager / Custom SMTP Mailers** | `cloud-deploy` | Outbound push notification bus for operational alerts. |
| **Security & Secrets** | **AWS SSM Parameter Store** | **HashiCorp Vault / Spring Cloud Config** | `adapters` & `cloud-deploy` | Encrypted runtime configuration and secret vault. |
| **Governance & IAM** | **IAM OIDC & AWS Budgets** | **Static Service Accounts / Manual Audits** | `cloud-deploy` | Keyless CI/CD federation and automated cost guardrails. |

### Mapping to Classical Architecture & Request Flow

- **Amazon S3 (Analog: Nginx Static Servers & Local Disk Storage):** Serves the compiled Angular 21 bundle directly to browsers (alongside GitHub Pages), stores finalized pour receipts, and emits object-creation event triggers to Lambda without maintaining a web server or shared filesystem.
- **Amazon API Gateway (Analog: Nginx & Spring Cloud Gateway):** Intercepts incoming HTTP requests from the Angular app, validates the `X-Festival-Pass` header, and enforces a 5-requests-per-second throttle at the network edge.
- **AWS IoT Core *(Next Stage — Physical Hardware)* (Analog: Mosquitto & Self-Hosted MQTT Brokers):** Provides a managed, mutually-authenticated MQTT message broker for the next stage when real physical tap controllers replace browser simulation—routing hardware flow-meter events directly into existing Lambda handlers without altering `core`.
- **AWS SSM Parameter Store (Analog: HashiCorp Vault & Spring Cloud Config):** Supplies encrypted runtime secrets (such as the private invite pass and admin credentials) to outer adapters at startup without running a dedicated configuration server.
- **AWS Lambda (Analog: Tomcat, Jetty & Spring Boot Containers):** Wakes up in ~150 ms to run pure-Java 24 domain logic (`core`), recording pours in **Amazon DynamoDB** (analogous to **PostgreSQL + Hibernate/JPA** with atomic, schema-less event storage) before shutting down to zero cost.
- **AWS Step Functions (Analog: Quartz Scheduler & Camunda):** Tracks time-bound lifecycles—such as enforcing a safety auto-shutoff timer if a tap remains open too long—without keeping JVM scheduler threads or stateful workflow engines alive.
- **Amazon SQS (Analog: RxJava, Project Reactor, Kafka & RabbitMQ):** Keeps user requests non-blocking without pulling reactive stream libraries (`Observable`, `Flux`, `Mono`) or heavy broker clients into Java code. SQS holds post-pour messages on disk and automatically instantiates background **AWS Lambda** workers to execute jobs across `core` and outer adapters (S3 receipt archiving, billing, alerts), surviving crashes via automatic retries and Dead-Letter Queues (DLQs).
- **AWS X-Ray (Analog: Jaeger, Zipkin & Bytecode Agents):** Propagates correlation IDs across API Gateway, Lambda, DynamoDB, SQS, and Step Functions to map end-to-end latency per pour without attaching heavy reflection agents to the JVM.
- **Amazon CloudWatch & Amazon SNS (Analogs: ELK Stack, Prometheus, Grafana & Alertmanager):** CloudWatch ingests single-line JSON logs and `TelemetryPort` metrics into live browser dashboards and alarms over standard HTTPS (443), triggering **SNS** push alerts (email, SMS, or Slack) whenever a Dead-Letter Queue or error threshold fires.
- **IAM OIDC & AWS Budgets (Analogs: Static Credentials & Manual Billing Checks):** OpenID Connect federation grants GitHub Actions short-lived deployment access without stored AWS keys, while a $1.00 billing alarm enforces zero-cost operation.

---

## Serverless Scalability & Partitioning

By adopting a fully serverless stack, infrastructure capacity planning, server sizing, and manual auto-scaling rules are eliminated—shifting architectural focus purely to clean data partitioning:

* **Zero-Config Compute & Messaging Elasticity:** **AWS Lambda**, **API Gateway**, **SQS**, and **S3** scale automatically from zero to thousands of concurrent festival pours without load balancers, container clusters, or idle servers.
* **Natural High-Cardinality Partitioning:** Each beer dispenser emits its immutable event stream under its own independent partition identity (`tapId`). Because every tap writes to its own stream, **Amazon DynamoDB** distributes load evenly across physical storage partitions—preventing single-key bottlenecks ("hot partitions") and enabling seamless horizontal growth as more taps are added.
* **Stateless Execution:** Because all operational state lives either in DynamoDB event streams or inside managed **Step Functions** workflows, compute workers remain completely stateless and horizontally interchangeable.

---

## Observability & Production Readiness

The system provides full operational visibility across synchronous HTTP boundaries, DynamoDB event streams, SQS queues, and Step Functions workflows. Live throughput, latency, and health metrics are natively available in the browser alongside automated push alerts, with zero separate monitoring servers to maintain.

The architecture preserves **zero framework overhead** and keeps `core` 100% free of monitoring SDKs by embedding observability directly at the hexagonal boundaries:

* **Domain-Level Telemetry Port (`core` & `adapters`):** Business-critical milestones (e.g., tap actuation, safety auto-shutoff triggers, sequence conflicts) are modeled through a lightweight outbound port (`TelemetryPort`), keeping domain logic pure. Local runs (`Adapters.jdk()`) stream structured logs to standard output (`stdout`), while cloud deployments (`Adapters.aws()`) emit single-line JSON logs and custom **Amazon CloudWatch** metrics without heavy reflection-based agents.
* **End-to-End Distributed Tracing (`ingress` & `cloud-deploy`):** Every inbound request carries a correlation ID (`X-Amzn-Trace-Id` / W3C `traceparent`) across **API Gateway**, **AWS Lambda**, **SQS** message attributes, and **Step Functions** executions via **AWS X-Ray** (with W3C `traceparent` context propagation), making a single beer pour traceable from tap open to asynchronous receipt generation.
* **Structured Logging & Business Context:** All ingress and background logs contain high-cardinality domain identifiers (`tapId`, `pourId`, `eventSequence`), enabling instant root-cause queries in **CloudWatch Logs Insights**.
* **Live Dashboards, Push Alerts & Zero Open Ports (`cloud-deploy`):** Because the cloud infrastructure is fully managed and serverless, **no custom network ports or self-hosted monitoring servers exist**. Operational views are accessible over standard **HTTPS (443)** via IAM-authenticated AWS Console links exported by `cloud-deploy`:
  * **Live Operational Dashboard:** The exported `CloudWatchDashboardUrl` displays real-time pour rates, safety shutoffs, and Lambda latency in the browser within 10–60 seconds of traffic.
  * **Visual Service Map:** **AWS X-Ray -> Service Map** provides an interactive graph of live request flows across API Gateway, Lambda, DynamoDB, SQS, and Step Functions.
  * **Automated Push Alerts:** The `AlertsTopicArn` **Amazon SNS** output connects directly to email, SMS, or Slack (via **AWS Chatbot**), delivering instant notifications whenever an **SQS Dead-Letter Queue (DLQ)** or **CloudWatch Alarm** detects a stuck pour or failed billing worker.

---

## Zero-Cost Delivery & Dry-Run Deployment Strategy

The entire build, test, and cloud release pipeline is designed as a **$0/month sandbox and testing environment**, combining three free-tier service layers into a single automated workflow:

1. **GitHub Actions + GitHub Packages + PactFlow (Free Plan) — $0/mo:** Compiles Java 24 and Angular 21, enforces waterproof compile-time boundaries via isolated Maven/JPMS repositories, runs unit and Pact contract tests, and publishes versioned JAR artifacts and Pact contracts.
2. **Floci / LocalStack Hobby / DynamoDB Local — $0/mo:** Runs open-source and free local Docker cloud emulators inside CI to execute a complete **dry-run deployment**—provisioning DynamoDB tables, SQS queues, Step Functions, and Lambdas locally to validate end-to-end user stories before touching AWS.
3. **AWS Always Free Tier + GitHub Pages — $0/mo:** Hosts the verified serverless backend indefinitely within free quotas (1M Lambda requests, 25 GB DynamoDB storage, 1M SQS requests, and 4,000 Step Function transitions per month) alongside free static frontend hosting on **AWS S3** or **GitHub Pages**.

---

### Dry-Run to Cloud Promotion Flow

1. **Component Packaging:** Foundation (`core`, `api-contract`) and adapter (`persistence`, `ingress`, `adapters`, `ui`) repositories verify contracts independently and publish versioned artifacts.
2. **Dry-Run Verification (`cloud-deploy`):** Before touching AWS, CI spins up the local AWS emulator stack, deploys the Infrastructure as Code templates into the sandbox, and runs full acceptance tests against the assembled AWS-wired application.
3. **Zero-Cost Cloud Rollout:** Once the dry-run succeeds, GitHub Actions authenticates via OpenID Connect (OIDC) and promotes the stack to the **AWS Always Free Tier** (protected by a $1.00 AWS Budgets billing alarm).

---

## Access Control & Private Cloud Sandbox

To prevent unauthorized visitors or bots from exhausting the **$0/month AWS Free Tier** quotas while keeping `core` 100% cloud-agnostic, access control is modeled as a standard secondary outbound port (`AccessGuardPort`) realized inside **`festival-tap/adapters`**:

* **Local & Test Mode (`Adapters.jdk()`):** Uses a lightweight in-memory/permissive realization so local execution and automated test suites run without external secret stores or cloud credentials.
* **Private Cloud Mode (`Adapters.aws()`):** Validates an invite pass (`X-Festival-Pass`) and admin credentials against secret parameters stored securely in **AWS Systems Manager (SSM) Parameter Store** (free tier, cached in Lambda memory for zero latency).
* **Edge Quota Protection (`cloud-deploy`):** Pairs the adapter check with strict API Gateway rate-limiting throttling (e.g., max 5 requests/second) so unauthorized traffic is dropped before impacting free-tier compute or database limits.

---

## Using the Application

Once assembled and started (either locally via the pure-JDK factories or deployed to AWS via [`festival-tap/cloud-deploy`](https://github.com)), the platform exposes two interactive experiences:

* **Public Self-Service Bar (`/public`):** Visitors enter using their invite pass (`X-Festival-Pass`), view all active beer taps with live prices per liter, and open or close taps in real time. Dispensed volume and cost update live, with an automatic safety shut-off at **0.15 L**.
* **Admin Dashboard (`/login`):** Operators sign in with admin credentials to register new taps, adjust flow rates and pricing, and inspect real-time spending and volume reports computed by replaying the immutable event stream.

For step-by-step startup commands, environment configuration, and end-to-end demo walkthroughs, see the **[`festival-tap/cloud-deploy` README](https://github.com#readme)**.


> For step-by-step startup commands, environment configuration, and end-to-end demo walkthroughs, see the **[`festival-tap/cloud-deploy` README](https://github.com)**.

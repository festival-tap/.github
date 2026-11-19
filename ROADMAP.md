# festival-tap — Critical Path Development & Delivery Roadmap

> *Part of the **[festival-tap Architecture Overview](./profile/README.md)**.*

This roadmap outlines the sequential, test-driven execution plan across all eight repositories for the `festival-tap` project, prioritizing a zero-speculative backend-first approach.

---

## Phase 0: Organization & Governance Setup
- [x] **[0.1] Organization & Architecture Blueprint:** Establish the `festival-tap` GitHub organization and document the architecture in `.github/profile/README.md`.
- [ ] **[0.2] Package Registry & Automation:** Configure GitHub Packages and Dependabot update policies.
- [ ] **[0.3] Reusable Infrastructure Check:** Confirm publication of base persistence engines (`persistence-engines-core` and `persistence-engines-dynamodb`) under `sugarcoat-labs`.

---

## Phase 1: ATDD Red-State Harness & Domain Contracts
- [ ] **[1.1] [`festival-tap/user-stories`](https://github.com) — Executable Behavioral Specs:** Extract functional specs (`us-01` to `us-08`) and black-box cURL scripts; release `v1.0.0`.
- [ ] **[1.2] [`festival-tap/cloud-deploy`](https://github.com) — ATDD Red-State Test Harness:** Import user stories and configure CI to run the initial RED cURL acceptance suite.
- [ ] **[1.3] [`festival-tap/api-contract`](https://github.com) — Design-First REST Specification:** Extract OpenAPI 3.0 spec, generate Java 24 / TypeScript models via Maven, and publish `v1.0.0`.
- [ ] **[1.4] [`festival-tap/core`](https://github.com) — Domain Models & Port Interfaces (`v1.0.0`):** Define pure Java 24 domain records, immutable events, inbound use cases, and outbound port interfaces (`TapEventPublishingPort`, `TapRegistryPort`, `ClockPort`, `AccessGuardPort`), and publish the contract JAR to GitHub Packages.

---

## Phase 2: Outbound Adapters & Core Business Logic
- [ ] **[2.1] [`festival-tap/persistence`](https://github.com) — Outbound Persistence Adapters (`v1.0.0`):** Bridge `core` persistence ports to `sugarcoat-labs/persistence-engines-core` (`Jdk`) and `persistence-engines-dynamodb` (`Aws`) + `DynamoDbCodec<T>` serializers; publish `v1.0.0`.
- [ ] **[2.2] [`festival-tap/core`](https://github.com) — Core Business Logic & Event-Sourcing Verification (`v1.1.0`):** Implement domain services and verify all pricing, pour timeout, and event-replay rules using `Persistence.jdk()` (`v1.0.0`); publish `v1.1.0`.
- [ ] **[2.3] [`festival-tap/adapters`](https://github.com) — Outbound Support Adapters (`v1.0.0`):** Build Clock, IdGenerator, and `AccessGuardPort` adapters (`Adapters.jdk()` vs `Adapters.aws()`); publish `v1.0.0`.
- [ ] **[2.4] [`festival-tap/ingress`](https://github.com) — Inbound Ingress Adapters (`v1.0.0`):** Implement HTTP/JSON routers, pure-JDK virtual thread server (`jdk.httpserver`), and AWS Serverless handlers (`Ingress.jdk()` vs `Ingress.aws()`); publish `v1.0.0`.

---

## Phase 3: Backend Proof Gate
- [ ] **[3.1] [`festival-tap/cloud-deploy`](https://github.com) — Pure-JDK Backend Assembly:** Wire all layers using pure-JDK factories and run `user-stories` cURL acceptance suites until all tests turn **GREEN**.

---

## Phase 4: Frontend Client & Contract Lock-In
- [ ] **[4.1] [`festival-tap/ui`](https://github.com) — Angular 21 Web Application:** Build standalone Angular 21 interface, 0.15 L safety shut-off, admin dashboard, and run Pact Consumer tests.
- [ ] **[4.2] [`festival-tap/ingress`](https://github.com) — Pact Provider Verification:** Execute Pact Provider verification tests against `Ingress.jdk()` to lock in API compatibility.

---

## Phase 5: Cloud Dry-Run & Live AWS Free-Tier Rollout
- [ ] **[5.1] [`festival-tap/cloud-deploy`](https://github.com) — AWS Wiring, Dry-Run & Cloud Promotion:** Wire production `Aws` factories, deploy Free Tier IaC templates, execute local dry-runs with Floci / LocalStack, configure OIDC / billing alarms, and promote to live AWS.

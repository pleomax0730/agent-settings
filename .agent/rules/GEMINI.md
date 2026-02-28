---
trigger: always_on
description: Best practices when building project
---

# Engineering Rules

## 0. Goals
- Localize changes: modifying one feature should not ripple across the system.
- Core logic is covered by fast, I/O-free unit tests.
- External dependencies (DB / HTTP / filesystem / queues / AI) are swappable without polluting core logic.
- Clear boundaries for multi-person collaboration: fewer conflicts, lower merge cost.

## 1. Architecture Principles
### 1.1 Vertical Slice Architecture (VSA)
Organize code by **feature / use case**, not by technical layer.

- MUST: A slice contains at minimum: boundary (API/route), core logic (service), data contracts (schemas), and tests.
- MUST: A slice only exposes its public entry point (e.g. router). Everything else is internal.
- SHOULD: Avoid cross-slice imports of internal modules. Use shared utilities or explicit ports when sharing is needed.
- SHOULD: Let internal layering within a slice grow organically. Don't force multi-layer splits upfront.
- SHOULD: Within a project, use consistent naming for slice internals. Exact names are not prescribed.

### 1.2 Dependency Inversion (Ports & Adapters) + Constructor DI
Core concept: **core logic depends on interfaces (ports); infrastructure provides implementations (adapters); the composition root wires them together.**

- MUST: External I/O (DB, HTTP clients, clock, UUID, AI clients, filesystem…) must be injected via constructor.
- MUST: Define ports using language-idiomatic interfaces (e.g. `typing.Protocol` in Python, `interface` in Go/TS). Services depend only on ports.
- MUST NOT: Create ports for pure logic that has no I/O, no non-determinism, and only one implementation. Import these directly.

### 1.3 SRP & YAGNI
- MUST: One module/class has one business reason to change.
- MUST: Do not create abstractions for hypothetical future needs (e.g. "we might switch DB someday").
- DO: Introduce abstractions when they make **current** tests faster/more reliable, or when you have 2+ concrete implementations now.

### 1.4 Thin Composition Root
The application entry point (e.g. `main.py`, `index.ts`, `cmd/server/main.go`) is only responsible for wiring:
- Create app / server instance
- Configure middleware
- Register routes / routers
- Assemble dependencies (DI providers)

- MUST NOT: Place endpoint implementations or business logic in the entry point.

## 2. Glossary
- **Slice**: The smallest self-contained unit of a feature (API + schemas + service + tests).
- **Port**: An interface / protocol describing a capability that core logic requires.
- **Adapter**: A concrete implementation of a port (e.g. DB repository, HTTP client, AI client).
- **Infra**: Adapters that wrap external dependencies and I/O (DB / HTTP / filesystem / queues / third-party SDKs). Code here imports third-party packages.
- **Shared**: Cross-slice **pure** utilities, types, and constants that depend only on stdlib. No third-party imports. Extraction has a threshold (see Section 7).

## 3. Reference Directory Layout
The structure below is conceptual. Exact directory names may vary by language or framework; what matters is **dependency direction and boundary enforcement**.

### 3.1 Single-Service Repository
```text
src/
  app/                # composition root: wiring, server bootstrap
  features/
    <feature_name>/
      api.*           # routing / controllers
      schemas.*       # boundary DTOs
      service.*       # use case orchestration
      ports.*         # interfaces (only if the slice has external I/O)
      engine/         # pure logic subfolder (only when service grows large)
      tests/
  infra/
    # Group adapters by technology or purpose.
    # If a category has only one file, put it directly in infra/ (e.g. infra/email.py).
    db/
    http/
    ai/               # e.g. OpenAI, Anthropic, Local LLM
    vision/           # e.g. OpenCV, YOLO, OCR
    payment/          # e.g. Stripe, PayPal
  shared/
    errors.*
    logging.*
    settings.*

tests/                # higher-level integration / e2e tests
scripts/
```
Not every slice needs `ports.*` or `engine/`. A simple slice may only have `api.*`, `service.*`, `schemas.*`, and `tests/`.

### 3.2 Monorepo (Multiple Services)
```text
apps/
  api/
    src/...
  worker/
    src/...
packages/
  shared/             # cross-app shared (stricter extraction threshold)
```

## 4. Slice Boundaries & Dependency Rules
### 4.1 Dependency Direction (inside → outside)
These are conceptual layers, not folder names. They map onto the directory structure like this: pure logic lives inside a slice's `engine/` or `service.*`; API lives in `api.*`; adapters live in `infra/`.

- pure logic → depends only on stdlib and type definitions
- service → depends on pure logic + ports + plain data structures
- API layer → depends on schemas + service
- adapters → depends on external packages and concrete I/O

### 4.2 Forbidden Dependencies
- MUST NOT: domain/application layer directly imports web frameworks, ORMs, HTTP clients, file/image parsers, or third-party SDKs.
- MUST: All of the above must be wrapped in adapters and injected via ports.

### 4.3 Shared vs Infra Boundary
The deciding factor is **whether the code imports third-party packages**:
- Imports third-party packages (e.g. ORM, HTTP lib, image lib, AI SDK) → `infra/`
- Pure stdlib only (types, constants, error classes, pure functions) → `shared/`

### 4.4 Cross-Slice Rules
- MUST: A slice cannot import another slice's internals.
- SHOULD: If slice A needs a capability owned by slice B (e.g. calling B's business logic), define a port in A rather than importing B's service directly.
- SHOULD: Only extract into `shared/` when the code is a stable, pure utility (see Section 7 for threshold).

## 5. DI Guidelines
### 5.1 When to Create a Port
Create a port ONLY when the dependency falls into one of these categories:
- I/O: DB, HTTP, queues, filesystem
- Non-determinism: clock, UUID, random
- Heavy or expensive: AI/LLM clients, CV engines

If the dependency is **none of the above** (e.g. a pure helper class, a data transformer, a validation function), do NOT create a port. Import it directly.

### 5.2 Where to Place Ports vs Adapters
- Port: inside the slice (`features/<feature>/ports.*`)
- Adapter: inside infra (`infra/*`)
- Wiring: inside the composition root (`app/*`)

### 5.3 Exceptions (Avoid Over-Engineering)
- MAY: Pure helpers (stateless functions) within a slice can be imported directly.
- MAY: Small internal-only components that are trivially replaceable can skip port extraction initially. Extract once you need I/O-free unit testing or a second implementation appears.
- A simple slice with no external I/O does not need `ports.*` at all.

## 6. Testing Strategy
### 6.1 TDD Workflow (Recommended)
- RED: Write a failing test that describes the expected behavior.
- GREEN: Make the minimal change to pass it.
- REFACTOR: Remove duplication, improve naming, extract pure logic.

### 6.2 Unit Tests (Must Be I/O-Free)
- MUST: No DB connections, no HTTP calls, no real filesystem reads.
- MUST: For services that depend on ports, inject fake/mock implementations.
- SHOULD: Cover service and engine (pure logic) decision and transformation logic.

### 6.3 Integration Tests (Minimal but Necessary)
- SHOULD: At least 1 integration test per slice to verify wiring, routing, and contracts.
- MUST: CI must be able to reliably start dependencies (e.g. test DB / container) or explicitly use mocks.

### 6.4 Contract / E2E (As Needed)
- MAY: Use contract tests for external service boundaries.
- MAY: Write a small number of E2E tests for critical user journeys.

## 7. Shared Extraction Threshold
- DO: Allow reasonable duplication between slices (duplication is usually cheaper than wrong abstractions).
- ONLY extract into shared when ALL of the following are true:
  - 2+ slices (or 2+ apps) genuinely need it
  - The contract is stable
  - Extraction actually reduces coupling (not just "looks cleaner")

## 8. Collaboration Rules
- MUST: Keep PRs small and reviewable. Include Why / What / How to test.
- SHOULD: Scope changes to a single slice. Avoid PRs that touch 3+ slices simultaneously.
- SHOULD: When adding or changing architectural rules or dependency directions, add tests or a brief ADR (Architecture Decision Record).

## 9. Git Workflow
### 9.1 Branch Naming
- `feat/<scope>-<desc>`
- `fix/<scope>-<desc>`
- `chore/<desc>` / `refactor/<desc>`

### 9.2 Conventional Commits
Format: `<type>(<scope>): <subject>`
- type: feat | fix | chore | refactor | test | docs
- scope: feature name (e.g. `chat`, `billing`, `analyze`)

Examples:
- `feat(chat): add streaming response support`
- `refactor(core): inject clock port for deterministic tests`

## 10. Legacy & Incremental Migration
Early-stage projects won't be fully compliant from day one. Legacy is acceptable as long as there are migration rules.

- MUST: New features must not extend legacy patterns (e.g. don't add new endpoints to the entry point).
- SHOULD: Apply the Boy Scout Rule—leave it better than you found it.

### 10.1 When to Trigger Migration
- A second endpoint or use case appears in the same entry file.
- Logic starts requiring mocks (AI/DB/HTTP) to be testable.
- PRs begin frequently conflicting on the same file.

### 10.2 Migration Approach (Strangler Fig)
1. Move routes into a feature slice (no behavior change).
2. Extract core logic into service and engine modules (add unit tests).
3. Extract I/O into ports/adapters (add integration tests).

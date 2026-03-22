---
trigger: always_on
description: General-purpose engineering and architecture guidance for product codebases
---

# Engineering Guidelines

## 0. Purpose

This document is a **general-purpose architecture guide**, not a rigid template.

Its goal is to help teams build systems that are:

- easy to understand
- easy to change safely
- easy to test
- resilient to external dependency churn
- practical for multi-person collaboration

Use these rules as **decision support**, not as folder-name dogma.

## 1. Rule Levels

Not every rule in this document has the same weight.

### 1.1 Universal Principles

These are close to language- and framework-independent:

- Separate core decision-making from side effects.
- Keep dependency direction clear.
- Prefer local reasoning over global coupling.
- Optimize for maintainability and safe change, not visual neatness alone.
- Introduce abstractions only when they solve a current problem.

### 1.2 Strong Defaults

These are good defaults for many service and product codebases:

- Prefer organizing code by feature, use case, or capability.
- Isolate external systems behind explicit boundaries when that improves testing and changeability.
- Keep app startup and wiring code thin.
- Prefer fast unit tests for important logic and fewer, targeted integration tests for real wiring.

### 1.3 Change Scope and Simplicity

Prefer the smallest change that directly solves the current problem.

- Do not add abstractions, configurability, validation, or fallbacks for hypothetical needs.
- Keep fixes and features narrowly scoped; avoid unrelated refactors or cleanup.
- Prefer simple local code over one-off helpers or premature reuse.
- Trust internal invariants and framework guarantees; validate at real system boundaries.

### 1.4 Project Conventions

Folder names, exact slice shapes, and DI style are conventions. They should serve the principles above, not replace them.

Examples:

- `features/`, `infra/`, `shared/`
- `skills/`, `workflows/`, `agents/`, `jobs/`, `pipelines/`
- constructor injection, factory wiring, function-parameter injection

## 2. Core Model

Think in terms of **responsibility** first.

- **Capability / use-case code**: Represents what the system does.
- **Core logic**: Pure rules, transformations, decisions, validation, orchestration-friendly helpers.
- **Application orchestration**: Coordinates use cases, invokes ports, sequences work.
- **Infrastructure**: Talks to external systems or wraps non-deterministic / heavyweight dependencies.
- **Shared**: Stable, pure utilities reused across multiple areas.

Classification should be based on a module's **primary responsibility**, not on whether it eventually calls another layer.

## 3. Placement Rules

### 3.1 Infra

`infra` is for code whose main job is integration with the outside world.

Common examples:

- database repositories
- HTTP clients
- filesystem access
- queues and messaging
- cache adapters
- cloud SDK wrappers
- AI / LLM / OCR / CV providers
- clock / UUID / randomness providers when isolated behind explicit boundaries

Signals that code likely belongs in `infra`:

- performs I/O
- depends on non-deterministic runtime behavior
- wraps a third-party SDK or heavyweight library
- exists mainly to translate between your system and an external system

Important:

- Third-party imports are a strong signal for `infra`, but not the sole definition.
- Stdlib-only code can still be `infra` if it performs I/O or runtime integration.

### 3.2 Shared

`shared` is for stable, low-level, pure reuse.

Typical contents:

- pure helpers
- stable cross-cutting types
- constants
- error types
- small utility functions

`shared` should stay boring. If something is complex, unstable, or strongly owned by one capability, keep it local instead.

### 3.3 Capability Layers

Projects may define first-class capability modules when they reflect real concepts in the problem space.

Examples:

- `features/`
- `skills/`
- `workflows/`
- `agents/`
- `jobs/`
- `pipelines/`

These are valid top-level modules when they represent **what the system does**, even if they depend on infra underneath.

Do not force these modules into `infra` just because they eventually call adapters.

## 4. Dependency Direction

Prefer this conceptual direction:

- pure logic depends on language/runtime basics and plain data
- application/use-case orchestration depends on pure logic and explicit boundaries
- API/UI/boundary code depends on application/use-case orchestration
- infra depends on external packages and concrete runtime integrations

The exact folders do not matter as much as this rule:

- business or application logic should not directly depend on framework-heavy or integration-heavy details unless there is a deliberate, low-cost reason

## 5. Preferred Organization

### 5.1 Default for Product and Service Codebases

Prefer organizing code by feature, use case, or capability rather than by technical layer alone.

Why:

- better change locality
- clearer ownership
- fewer cross-team merge conflicts
- easier reasoning about user-facing behavior

### 5.2 Example Shapes

These are examples, not laws:

```text
src/
  app/                 # startup, wiring, registration
  features/
    <slice>/
      api.*
      schemas.*
      service.*
      ports.*
      engine/
      tests/
  infra/
    db/
    http/
    ai/
  shared/
  tests/
```

```text
src/
  agents/
  skills/
  workflows/
  infra/
  shared/
  app/
```

What matters is whether the structure preserves:

- clear ownership
- low coupling
- understandable boundaries
- safe dependency direction

## 6. Ports and Adapters

Use ports/adapters when they help isolate external concerns.

Create an explicit boundary when a dependency is:

- I/O-related
- non-deterministic
- expensive or heavyweight
- unstable enough that isolating it improves tests or change safety

Examples:

- DB access
- HTTP clients
- message brokers
- file storage
- AI/LLM providers
- time, UUIDs, randomness

Do not create ports for:

- pure logic
- stateless transformations
- simple validation helpers
- code with only one obvious implementation and no testing or ownership pressure

### 6.1 DI Guidance

The goal is not “constructor injection everywhere.”
The goal is that external dependencies are supplied at the system edge and replaceable in tests.

Valid approaches include:

- constructor injection
- function parameters
- thin factories/providers
- composition-root assembly

Choose the lightest option that keeps boundaries clear.

## 7. Composition Root

Keep startup code thin.

Typical responsibilities:

- create app/server/process entrypoint
- configure middleware
- register routes / handlers / jobs
- assemble dependencies
- choose concrete adapters

Avoid putting business logic, endpoint logic, or multi-step workflows directly in the entrypoint.

## 8. Testing Strategy

### 8.1 Universal Testing Guidance

- Test important behavior, not just structure.
- Prefer fast tests for logic-heavy code.
- Test real wiring where integration bugs are likely.
- Keep most tests deterministic.

### 8.2 Good Defaults

- unit tests for pure logic and application decisions
- fake or mock boundaries for external systems
- integration tests for boundary wiring and adapter contracts
- a small number of end-to-end or contract tests for critical paths

### 8.3 Avoid

- requiring real DB/HTTP/filesystem/AI in ordinary unit tests
- over-indexing on integration tests for logic that could be tested in-process

## 9. Shared Extraction Threshold

Do not extract into `shared` just because two files look similar.

Extract only when most of the following are true:

- multiple areas genuinely need it
- the concept is stable
- extraction reduces coupling
- the abstraction makes code easier, not merely “cleaner”

Local duplication is often cheaper than a premature global abstraction.

## 10. Collaboration and Scope

Prefer changes that are:

- small
- reviewable
- locally understandable
- scoped to one capability or slice where practical

When a change alters dependency direction, ownership boundaries, or top-level structure, document the reason briefly in the PR or an ADR.

## 11. Context-Specific Exceptions

Different codebases have different optimal structure.

### 11.1 Simpler Is Better For

- scripts
- one-off internal tools
- prototypes
- experiments

In these cases, avoid architecture ceremony unless the code is growing.

### 11.2 Different Priorities May Apply For

- research / ML experimentation code
- frontend-heavy UI systems
- data pipelines
- embedded / systems programming

The principles still apply, but the preferred module shape may differ.

## 12. Decision Checklist

Before adding a module or abstraction, ask:

1. What is this module's primary responsibility?
2. Is it expressing what the system does, or how it integrates with something external?
3. Does this boundary improve testing, replaceability, or ownership clarity today?
4. Am I introducing a reusable abstraction too early?
5. Will this make future changes more local or more scattered?

If the answers are unclear, prefer the simpler design and keep the code closer to where it is used.

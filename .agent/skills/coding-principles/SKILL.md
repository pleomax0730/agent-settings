---
name: coding-principles
description: Apply project guardrails for architecture, slice boundaries, dependency injection, testing strategy, composition roots, shared-vs-infra decisions, collaboration scope, and incremental migration. Use when work affects structure, dependency direction, module boundaries, or broader implementation design.
---

# Architecture Guardrails

Use this skill whenever work affects architecture, module boundaries, dependency design, testing strategy, or repo structure.

Load the detailed reference before making structural decisions:
- [`references/ARCHITECTURE_GUIDELINES.md`](references/ARCHITECTURE_GUIDELINES.md)

## Core Workflow

1. Read the relevant part of [`references/ARCHITECTURE_GUIDELINES.md`](references/ARCHITECTURE_GUIDELINES.md) before proposing or making structural changes.
2. Check that the design keeps responsibilities clear, prefers feature or capability locality where helpful, and keeps business logic out of framework-heavy layers.
3. Use ports/adapters only for external I/O, non-determinism, or heavy dependencies. Do not abstract pure logic unnecessarily.
4. Keep composition roots thin and keep external libraries out of domain and service logic.
5. Prefer fast, I/O-free unit tests for core logic and add integration tests only where wiring or contracts need proof.
6. Avoid extracting to shared code too early. Prefer local duplication over unstable abstractions.
7. Keep changes scoped and reviewable. If the change crosses slices or alters dependency direction, call that out explicitly.
8. Prefer the simplest change that solves the requested problem. Avoid speculative abstraction and unrelated cleanup.

## Decision Rules

- Prefer feature- or capability-oriented organization over purely horizontal layering when it improves locality and ownership.
- Keep each capability or slice self-contained enough to preserve clear ownership, contracts, core logic, and tests where applicable.
- Do not import another slice's internals directly.
- Put external-I/O and integration-heavy code in infra, not shared.
- Import pure helpers directly unless there is a current need for an interface.
- New work should move the codebase toward the target architecture, not deepen legacy patterns.

## During Reviews

- Flag cross-slice leakage, misplaced framework code, unnecessary abstractions, and missing tests for core logic.
- Check whether DI is used for real external dependencies instead of being added everywhere by default.
- Check whether the plan and implementation keep the change within one slice where practical.

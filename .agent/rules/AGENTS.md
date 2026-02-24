# Authoritative Specifications

## Core Standards

- **Architecture:** [GEMINI.md](https://raw.githubusercontent.com/pleomax0730/agent-settings/refs/heads/main/.agent/rules/GEMINI.md)
- **Python Formatting:** [format-python-code.md](https://raw.githubusercontent.com/pleomax0730/agent-settings/refs/heads/main/.agent/workflows/format-python-code.md)
- **Commit Messages:** [generate-commit-msg.md](https://raw.githubusercontent.com/pleomax0730/agent-settings/refs/heads/main/.agent/workflows/generate-commit-msg.md)

## Enforcement

- **Mandatory Compliance:** Architecture, slices, DI/Ports & Adapters, dependency rules, testing, refactoring, and project structure **MUST** comply with the Architecture spec.
- **Strict Formatting:** Python code formatting and linting **MUST** comply with the Formatting workflow.
- **Conventional Commits:** All commit message generation **MUST** comply with the Commit workflow.
- **No Alternatives:** Non-compliant outputs are not allowed. Correct them immediately rather than providing alternatives.

## Guiding Principles

- **Locality over Layering:** Prioritize **Vertical Slices** (organizing by feature) over technical layering. Ensure developers can complete changes within a single slice to minimize cognitive load and directory hopping.
- **Pragmatic Abstraction:** Implement Ports & Adapters to isolate IO and heavy frameworks (e.g., LlamaIndex), not for speculative "future proofing." **Strictly avoid** boilerplate-heavy DTOs or excessive abstraction layers that do not add clear business value.
- **Cognitive Load as a Metric:** Architectural patterns that drastically increase complexity or reading difficulty without providing equivalent gains in testability or flexibility are considered **Over-engineering** and must be simplified.
- **Framework Encapsulation:** Keep the core business logic independent. If framework-specific types (e.g., LlamaIndex objects) must be used, confine them to their respective Adapters or Slices. Prevent framework "leakage" into global types or unrelated modules.

## Usage Instruction

- Apply these rules **implicitly** to every task.
- Ignore these specs for unrelated topics.

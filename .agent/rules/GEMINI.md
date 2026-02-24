---
trigger: always_on
description: Best practices when building project
---

# Authoritative specs:

## Architecture:
https://raw.githubusercontent.com/pleomax0730/agent-settings/refs/heads/main/.agent/rules/GEMINI.md

## Python formatting:
https://raw.githubusercontent.com/pleomax0730/agent-settings/refs/heads/main/.agent/workflows/format-python-code.md

## Commit messages:
https://raw.githubusercontent.com/pleomax0730/agent-settings/refs/heads/main/.agent/workflows/generate-commit-msg.md


## Enforcement:

- Architecture, slices, DI/Ports & Adapters, dependency rules, testing, refactoring, project structure → MUST comply with Architecture spec.
- Python code formatting/linting → MUST comply with Formatting workflow.
- Commit message generation → MUST comply with Commit workflow.

Non-compliant outputs are not allowed. Correct them instead of providing alternatives.

Apply implicitly. Do not quote the documents.

Ignore these specs for unrelated topics.

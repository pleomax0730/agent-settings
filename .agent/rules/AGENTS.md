# Project Context

- **General Understanding:** Please read the `README.md` file in the root directory to gain a general understanding of the project's purpose, domain, and workflow before beginning tasks.

## Core Standards

- **Planning / Execution / Review Skill Cycle:** Use the local `.agent` skills as a coordinated workflow when the task benefits from deliberate planning, structured execution, and explicit verification.
  - **Plan First:** Use `plan-execution` to explore, clarify, and create or update `PLAN.md` as the single source of truth for scope, assumptions, phased tasks, and acceptance checks.
  - **Execute Second:** Use `agent-coder` to implement against `PLAN.md`, execute the next open phase, and update progress in the same `PLAN.md` file. Do not create separate `task.md` or `implementation_plan.md` files.
  - **Review Third:** Use `code-review` to verify the implementation against `PLAN.md`, run the relevant checks, and decide whether the work is approved, should go back to `agent-coder` for fixes, or should return to `plan-execution` for plan updates.
  - **Cycle Rule:** Treat these three skills as a loop: `plan-execution -> agent-coder -> code-review ->` back to planning or execution if needed, with `PLAN.md` remaining the shared artifact throughout.
- **Coding Principles Skill:** Use the local `coding-principles` skill for architecture, module slicing, Dependency Injection (DI/Ports & Adapters), testing strategy, and project structural decisions during planning, implementation, and review. The detailed reference is mirrored locally from the upstream rules file.
- **Python Formatting Skill:** Use the local `format-python-code` skill whenever writing, modifying, or refactoring Python code. You **MUST** run this formatting and linting workflow before delivering code or preparing a commit.
- **Commit Message Skill:** Use the local `generate-commit-msg` skill when code modifications are finalized and ready to be committed. Commit messages **MUST** follow this workflow and the Conventional Commits standard.

## Usage Instruction

- Ignore these specs for unrelated topics.

# Project Context

- **General Understanding:** Read the relevant `README.md` before starting work. If the workspace root does not contain one, read the `README.md` of the target project instead.

## Core Standards

- **User Problem First:** Always identify and understand the user's actual problem, goal, and constraints before deciding whether any skill is relevant.
- **Keep Changes Simple:** Keep implementation simple and narrowly scoped. Prefer the smallest change that directly solves the requested problem. Avoid speculative abstractions, unrelated refactors, one-off helpers, and unnecessary validation or fallback logic.
- **Independent Workflow Skills:** Treat `plan-execution`, `agent-coder`, and `code-review` as independent skills, not as a required sequence. Do not invoke them by default; use them only when the user explicitly requests them or clearly asks for that style of help.
- **Workflow Skill Mapping:** Use `plan-execution` for structured planning and for creating or updating `PLAN.md` as the single source of truth. Use `agent-coder` for implementation against `PLAN.md` and to update progress there. Use `code-review` for review against `PLAN.md`, including relevant checks and an approval or follow-up outcome.
- **Coding Principles for Design Decisions:** Use the local `coding-principles` skill when work affects architecture, module boundaries, dependency direction, testing strategy, or other design-level implementation decisions.
- **Python Formatting Skill:** Use the local `format-python-code` skill whenever writing, modifying, or refactoring Python code. You **MUST** run this formatting and linting workflow before delivering code or preparing a commit.
- **Commit Message Skill:** Use the local `generate-commit-msg` skill when code modifications are finalized and ready to be committed. Commit messages **MUST** follow this workflow and the Conventional Commits standard.

## Usage Instruction

- Ignore these specs for unrelated topics.

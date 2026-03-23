---
name: agent-coder
description: Execute one decision-complete execution slice from PLAN.md, perform the minimum necessary validation, and update progress before handing off for review or the next slice.
---

# Agent Coder Workflow

Use this skill when a decision-complete `PLAN.md` already exists and the user wants implementation rather than more planning.

This skill owns the execution stage of a three-skill cycle:
- `plan-execution` prepares and updates `PLAN.md`,
- `agent-coder` implements the next open execution slice from `PLAN.md`,
- `code-review` verifies the result and either approves it or sends it back for fixes or replanning.

## Core Contract

Treat `PLAN.md` as the execution contract.

Implement the next open execution slice without silently expanding scope.

The default goal is to complete one clear execution slice, gather evidence that it works, update `PLAN.md`, and then either stop, continue to another coder-owned execution slice, or hand off to review.

## Workflow

### 1. Re-anchor on the plan and repo context

- Read `PLAN.md` first.
- Confirm the next open execution slice is decision-complete enough to execute.
- Read the relevant repo-local guidance such as `AGENTS.md`, `README.md`, and any linked docs.
- Explore the touched code paths, existing conventions, and tests before editing.

Do not ask the user to re-send repo files that are already available locally.

### 2. Use `PLAN.md` as the source of truth

- Do not create alternate planning files when `PLAN.md` is present.
- Treat `PLAN.md` as the single source of truth for scope, execution-slice boundaries, assumptions, and progress tracking.
- If `PLAN.md` is missing or materially ambiguous, stop execution and hand the task back to `plan-execution` instead of inventing missing plan details.

### 3. Execute the next bounded execution slice

- Implement only the next open coder-owned execution slice from `PLAN.md`.
- Keep changes narrow and aligned with the current slice.
- When the plan references verification or evidence expectations that belong to execution, satisfy them as part of the slice.
- If execution exposes a true spec gap, stop and route the task back to planning rather than making new product decisions in code.

If the next open item is primarily:
- a formal review checkpoint, hand off to `code-review`,
- a planning or unresolved decision block, hand off to `plan-execution`,
- a coder-owned test or validation task tied to the implementation, treat it as part of the current execution slice when appropriate.

### 4. Run lightweight execution checks

Before declaring the execution slice complete:
- confirm the intended files changed,
- run the required formatter or lint workflow for touched languages,
- run the most relevant targeted tests or smoke checks when practical,
- confirm the implemented behavior matches the current slice goals,
- capture concise evidence that the slice is working.

Examples of evidence:
- targeted test results,
- smoke command outputs,
- API responses,
- manual verification notes for UI behavior,
- config or migration outcomes when relevant.

Do not treat this as the formal review stage. Use `code-review` for deeper verification and final review output.

### 5. Update `PLAN.md`

When the slice is complete:
- mark the completed slice items in `PLAN.md`,
- update any execution notes that the next slice depends on,
- keep the plan aligned with what actually landed,
- avoid rewriting unrelated plan sections.

### 6. Decide the handoff

- By default, stop after one completed execution slice unless there is a clear reason to continue.
- Continue only if the next open item is still a coder-owned execution slice, remains decision-complete, and does not belong to formal review.
- If the implementation is complete enough for formal verification, hand off to `code-review`.
- If review or execution reveals plan gaps or contradictions, return to `plan-execution`.

Report execution completion separately from review approval.

## Guardrails

- Do not silently expand scope beyond what `PLAN.md` says.
- Do not perform deep review work that belongs in `code-review`.
- Do not keep executing if the next open item is ambiguous enough that new decisions would be required.
- Do not treat formal review checkpoints as coder work.
- Do not skip required formatting or targeted validation for touched code.
- Do not mark a phase complete if the expected evidence has not been gathered.

## Response Style

Be concise and execution-focused.

During implementation:
- state which execution slice is being executed,
- note the key files or systems being touched,
- report targeted validation performed,
- update the plan before moving on or handing off.

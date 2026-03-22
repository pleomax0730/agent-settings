---
name: code-review
description: Review and verify code changes against PLAN.md after execution. Use when implementation is complete enough for formal verification, diff review, and acceptance checks before closing the loop or sending work back for fixes.
---

# Code Review

Follow this skill when the user explicitly invokes `$code-review` or when work produced from `PLAN.md` needs formal review and verification.

## Core Contract

Treat this skill as the quality gate in a three-skill cycle:
- `plan-execution` writes and updates `PLAN.md`,
- `agent-coder` implements against `PLAN.md`,
- `code-review` verifies whether the changes satisfy `PLAN.md` and are safe to accept.

This skill does not own feature planning and should not invent new scope. It may request fixes, identify plan gaps, or recommend returning to planning when the implementation reveals missing decisions.

## Workflow

### 1. Re-anchor on the plan

- Read `PLAN.md` first.
- Identify which phases or acceptance criteria were meant to be completed.
- Check whether assumptions, interfaces, and test expectations in `PLAN.md` match what was actually implemented.

### 2. Inspect the implementation

- Review the relevant diff, touched files, and surrounding code paths.
- Focus on correctness, regressions, missing edge cases, and mismatches against the plan.
- Prefer concrete findings with file references over general commentary.

### 3. Verify with commands

- Run the most relevant formatters, linters, tests, or smoke checks for the changed area.
- Prefer targeted verification first, then broader checks when risk justifies it.
- If using Codex review output, persist it to `/tmp` reliably:
  - `codex review --uncommitted -c model="gpt-5.4" -c model_reasoning_effort="high" 2>&1 | tee /tmp/review_gpt54_high.txt`
  - validate the file is non-empty before parsing:
    - `wc -c /tmp/review_gpt54_high.txt`
  - only read or summarize the `/tmp` file after the command is fully completed.

### 4. Decide the next hop

- If the code is correct and aligned with `PLAN.md`, approve it and say the cycle can continue or close.
- If the code is fixable without changing the plan, send it back to `agent-coder` with the concrete issues.
- If the implementation exposed missing decisions, contradictions, or scope changes, send it back to `plan-execution` and explain what must be clarified in `PLAN.md`.

## Review Output

Findings come first.

Order feedback by severity:
- correctness bugs,
- regressions or integration risk,
- missing tests or verification gaps,
- plan mismatch or unresolved assumptions.

If there are no findings, say so explicitly and mention any residual risk or checks you did not run.

## Guardrails

- Do not treat successful test runs as sufficient if the implementation still misses the plan.
- Do not rewrite `PLAN.md` unless the user explicitly asks you to update the plan; recommend a return to `plan-execution` instead.
- Do not bury the key findings under a long summary.
- Do not approve work that still depends on unstated assumptions.

---
name: code-review
description: Review and verify completed code changes against the agreed scope, expected behavior, and acceptance criteria. Use when implementation is ready for formal verification, diff review, and targeted checks before acceptance or follow-up fixes.
---

# Code Review

Follow this skill when the user explicitly invokes `$code-review` or when implementation is complete enough to review for correctness, regressions, and acceptance readiness.

## Core Contract

This skill is the quality gate for completed code changes.

- Verify that the implementation matches the agreed scope.
- Prefer concrete findings over general commentary.
- Focus on user-visible correctness, regression risk, and verification evidence.
- Do not invent new feature scope during review.

If a `PLAN.md` exists, use it as the primary source of expected behavior and acceptance criteria. If no plan exists, derive expectations from the user request, task context, and the touched code.

## Workflow

### 1. Re-anchor on the expected outcome

- Read `PLAN.md` first when present.
- Identify the intended behavior, constraints, and acceptance criteria.
- Check whether assumptions, interfaces, and test expectations match what was actually implemented.

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

### 4. Decide the outcome

- If the code is correct and aligned with the expected outcome, approve it and note any residual risk.
- If the code is fixable without changing scope, report the concrete issues and expected follow-up.
- If the implementation exposes missing decisions, contradictions, or unclear scope, call that out explicitly and explain what must be clarified before approval.

## Review Output

Findings come first.

Order feedback by severity:
- correctness bugs,
- regressions or integration risk,
- missing tests or verification gaps,
- plan mismatch or unresolved assumptions.

If there are no findings, say so explicitly and mention any residual risk or checks you did not run.

## Guardrails

- Do not treat successful test runs as sufficient if the implementation still misses the expected behavior.
- Do not rewrite `PLAN.md` unless the user explicitly asks for it.
- Do not bury the key findings under a long summary.
- Do not approve work that still depends on unstated assumptions.

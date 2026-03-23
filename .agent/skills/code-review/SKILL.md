---
name: code-review
description: Review completed code changes against the intended scope, likely regression risk, and available verification evidence. Use when implementation is ready for formal review and acceptance assessment.
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
- Use `codex review --uncommitted`, `codex review --base <branch>`, or `codex review --commit <sha>` as a diff-focused review pass when useful.
- Treat `codex review` as one review signal, not the whole review. It reviews git changes and surfaces likely implementation issues; it does not by itself confirm scope alignment, product correctness, or acceptance completion.

### 4. Decide the outcome

- If the code is correct and aligned with the expected outcome, approve it and note any residual risk.
- If the code is fixable without changing scope, report the concrete issues and expected follow-up.
- If the implementation exposes missing decisions, contradictions, or unclear scope, call that out explicitly and explain what must be clarified before approval.

### 5. Feed back systemic issues

When the same class of issue appears repeatedly, call out whether the gap would be better fixed by:
- repo-local docs,
- lint or static checks,
- test coverage,
- templates or workflow guardrails.

Use this only when it materially helps prevent the same review issue from recurring.

## Review Output

Findings come first.

Order feedback by severity:
- correctness bugs,
- regressions or integration risk,
- missing tests or verification gaps,
- plan mismatch or unresolved assumptions,
- systemic guardrail gaps when relevant.

If there are no findings, say so explicitly and mention any residual risk or checks you did not run.

## Guardrails

- Do not treat successful test runs as sufficient if the implementation still misses the expected behavior.
- Do not rewrite `PLAN.md` unless the user explicitly asks for it.
- Do not bury the key findings under a long summary.
- Do not approve work that still depends on unstated assumptions.
- Do not describe `codex review` as if it performs full product or plan validation.

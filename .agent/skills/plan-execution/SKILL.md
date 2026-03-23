---
name: plan-execution
description: Plan-first workflow for tasks that need deliberate discovery, decomposition, and a decision-complete PLAN.md before coding starts. Use when the user wants structured planning, explicit assumptions, and implementation-ready phases rather than immediate code changes.
---

# Plan Execution

Follow this skill when the user explicitly invokes `$plan-execution` or clearly asks for a plan-first workflow.

## Core Contract

Treat the interaction as a planning engagement first.

Do not implement, edit files, or run mutating commands while this skill is active unless the user explicitly tells you to stop planning and execute.

Aim for a decision-complete plan that another engineer or agent could implement without making meaningful product or implementation decisions.

This skill owns the planning stage of a three-skill cycle:
- `plan-execution` discovers and writes `PLAN.md`,
- `agent-coder` executes the next open execution slice from `PLAN.md`,
- `code-review` verifies the resulting changes and either approves completion or sends the work back for fixes or replanning.

## Workflow

### 1. Ground in the environment

Explore before asking questions.

Use non-mutating inspection to resolve discoverable facts:
- read likely entrypoints, configs, schemas, manifests, and repo-local docs,
- search the codebase for relevant symbols and flows,
- inspect existing tests and conventions,
- run non-mutating checks when they improve confidence.

Do not ask questions that can be answered from the repo or runtime.

### 2. Maintain an explicit plan

If `update_plan` is available and allowed in the current runtime, use it to maintain a short checklist with:
- completed discovery steps,
- the current in-progress planning step,
- remaining planning steps.

If `update_plan` is unavailable or disallowed, mirror the same checklist briefly in normal assistant text.

Keep the plan concise and current. Do not create speculative implementation subtasks before the intent is clear.

### 3. Clarify in a loop

After each exploration pass, identify only the remaining ambiguities that materially affect the plan.

Ask focused follow-up questions that:
- change scope,
- lock an important assumption,
- choose between meaningful tradeoffs,
- define interfaces, acceptance criteria, or rollout constraints.

Prefer one to three short questions at a time.

Repeat this loop:
1. explore,
2. summarize what is now known,
3. ask the next high-impact question,
4. update the plan checklist.

Continue until the user explicitly accepts the plan direction and no decision that matters is left open.

### 4. Decompose the work into execution blocks

Break larger goals into smaller building blocks that can be handed to execution cleanly.

The final plan should separate work by the blocks that matter for delivery, such as:
- design or decision updates,
- execution slices,
- review checkpoints,
- test or validation tasks,
- rollout or evidence capture when relevant.

For each execution phase, make clear:
- what is in scope,
- what must already be true before execution starts,
- what output or repo-visible artifact is expected,
- how completion will be verified.

Use labels and wording that make ownership obvious:
- execution slices are meant for `agent-coder`,
- formal review checkpoints are meant for `code-review`,
- planning or unresolved decision blocks stay with `plan-execution`.

### 5. Plan instead of execute

While this skill is active, interpret requests to "implement", "do it", or "go ahead" as requests to refine the implementation plan unless the user clearly says to exit planning and begin execution.

If the user changes scope mid-stream, update the plan and continue the clarification loop.

### 6. Finalize with a complete plan

When the spec is decision complete, create or update a `PLAN.md` file with the final plan.

The `PLAN.md` content must include:
- a clear title,
- a short summary,
- progress checkboxes for phased execution,
- key implementation changes grouped by behavior, subsystem, or execution block,
- public API or interface changes when relevant,
- test cases and acceptance scenarios,
- explicit assumptions and defaults chosen,
- verification notes for each major phase.

When possible, label each checkbox or phase so the intended owner is obvious, for example:
- `[coder] Implement request validation`
- `[coder] Add targeted regression test`
- `[review] Verify diff and acceptance criteria`
- `[plan] Resolve rollout decision`

Keep the final plan concise but implementation-ready.

After writing `PLAN.md`, respond by briefly confirming that the proposed plan was created or updated there. Do not wrap the final plan in a `<proposed_plan>` block.
If execution should start next, explicitly hand off to `agent-coder`.

## Guardrails

Do not pretend this skill changes hidden system modes.

Do not claim that `update_plan`, `request_user_input`, or any other tool exists unless it is actually present in the current runtime.

Do not skip exploration just to ask obvious repo questions.

Do not produce or write a final plan while material ambiguity remains.

Do not execute mutating work until the user explicitly tells you to stop planning and switch to execution.

Do not leave phase boundaries vague enough that the coder must re-decide scope during implementation.

## Response Style

Be collaborative and direct.

During planning:
- give short progress updates,
- summarize discovered facts before asking questions,
- surface tradeoffs clearly,
- make reasonable default recommendations,
- record assumptions explicitly,
- emphasize the phase breakdown and acceptance path.

When the user accepts the plan, write the final answer into `PLAN.md`, then return a brief confirmation and stop there.

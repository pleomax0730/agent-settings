---
description: Plan tasks as implementation_plan.md and use Codex to execute them while maintaining session context.
---

# Agent Coder Workflow

This workflow delegates code writing to Codex while keeping the AI Assistant in the role of a planner and reviewer. This leverages Codex's editing capabilities while maintaining the same session context for efficiency and token caching.

## Steps

1. **Information Gathering**
   - Review the requested feature or bug fix.
   - **Actively Ask the User**: Ask the user to provide any relevant coding principle files (e.g., `AGENTS.md`), architectural diagrams, or specific constraints before proceeding. Do not assume any principles or guidelines without confirmation.
   - Explore the codebase to understand the existing context.

2. **Generate Implementation Plan & Task List**
   - Create a `task.md` to track the overall progress.
   - Create an `implementation_plan.md` containing clear, phase-by-phase steps with markdown checkboxes (`- [ ]`).
   - Describe exactly which files to edit, what functions to create, and the logic to implement for each phase.

3. **Execute Phase 1 with Codex**
   - Trigger the initial execution run in the background using `run_command` (set a short `WaitMsBeforeAsync`). Ensure that the context gathered in Step 1 is explicitly provided to Codex:
     `codex exec "Read the implementation_plan.md file. Fully execute Phase 1, making any necessary file edits. Update the plan to mark Phase 1 as [x] once complete. Ensure code complies with the following provided guidelines/files: [INSERT GATHERED FILES/RULES HERE]." --full-auto`
   - Use `command_status` to monitor the command's background execution. 
   - **Crucial:** From the initial output snapshot, note down the `session id:` (which is a UUID like `019c...`) so you can resume it later.

4. **Review and Verify**
   - Once the background command completes, verify Codex's changes.
   - Run necessary linters, tests (e.g., `uv run pytest`), or use `git status` / `git diff` to ensure correctness.
   - Ensure the `implementation_plan.md` was correctly updated.
   - If there are errors or failing tests, feed the error logs back into the next Codex prompt to fix them before proceeding to the next phase.

5. **Resume Codex for Subsequent Phases (Loop)**
   - To continue with the next phase while maintaining the identical session, run the resume command using the Session ID you captured:
     `codex exec resume <SESSION_ID> "Great. Proceed to execute the next open Phase in implementation_plan.md. Mark it as [x] once complete. Remember to adhere to the previously provided strict constraints." --full-auto`
   - Use `command_status` to wait for completion.
   - Repeat the "Review and Verify" step.
   - Continue this loop until all items in `implementation_plan.md` are finished.

6. **Finish**
   - Report the completion of the task to the user.

---
description: Plan tasks as implementation_plan.md and use Codex to execute them while maintaining session context.
---
# Agent Coder Workflow

This workflow delegates code writing to Codex while keeping the AI Assistant in the role of a planner and reviewer. This leverages Codex's editing capabilities while maintaining the same session context for efficiency and token caching.

## Steps

1. **Understand Task and Codebase**
   - Review the requested feature or bug fix.
   - Explore the codebase to understand the context.

2. **Generate Implementation Plan**
   - Create an `implementation_plan.md` file in the workspace containing clear, phase-by-phase steps with markdown checkboxes (`- [ ]`).
   - Describe exactly which files to edit, what functions to create, and the logic to implement for each phase.

// turbo
3. **Execute Phase 1 with Codex**
   - Trigger the initial execution run:
     `codex exec "Read the implementation_plan.md file. Fully execute Phase 1, making any necessary file edits. Update the plan to mark Phase 1 as [x] once complete." --full-auto`
   - Use `command_status` to monitor the command to completion.

4. **Review and Verify**
   - Verify Codex's changes (run `git status`, test commands, or view diffs).
   - Ensure the `implementation_plan.md` was correctly updated.

// turbo
5. **Resume Codex for Subsequent Phases (Loop)**
   - To continue with the next phase while maintaining the identical session, run:
     `codex exec resume --last "Great. Proceed to execute the next open Phase in implementation_plan.md. Mark it as [x] once complete." --full-auto`
   - Wait for it to finish and review the changes.
   - Repeat this step until all items in `implementation_plan.md` are finished.

6. **Finish**
   - Report the completion of the task to the user.

---
agent: agent
description: "Bootstrap a platform hardening session — load plan, check progress, route to correct model"
---

# Hardening Sprint

Start a platform hardening work session. Load the review, execution plan, and guide you to the next task.

## Steps

1. **Load the plan and execution state.** Read these files:
   - `planning/tasks/platform-review-2026-04-17.md` — the full architecture review (Parts 1–6)
   - `planning/tasks/platform-hardening-execution.md` — the execution plan with checkboxes

2. **Identify progress.** Count checked (`- [x]`) vs unchecked (`- [ ]`) tasks. Report:
   - Current phase (the earliest phase with unchecked tasks)
   - Next unchecked task
   - Any blocked tasks (dependencies not yet complete)
   - Updated progress table

3. **Check model routing.** Read Part 6 of the review ("Execution Guidance — Model Routing"). For the next task:

   ⚠️ **If the task is marked `model: 4.7`** (currently tasks 1.4 and 3.6), warn the user:

   > **Model routing notice:** Task {id} is flagged for Opus 4.7 in the execution guidance.
   > This task involves {reason}. Consider switching to Opus 4.7 for this task, or proceed
   > with 4.6 using the sub-task breakdown in the execution plan.

   If the task is `model: 4.6`, proceed normally.

4. **Load relevant context.** Based on the next task's repo(s), read:
   - The relevant source files referenced in the review finding
   - `copilot/.github/shared/architecture-cheatsheet.instructions.md` if the task touches Core/SSI/UI integration
   - Repo memory (`/memories/repo/`) for lessons learned

5. **Present the work plan.** Show:
   - Task ID, title, repo, effort, model recommendation
   - The specific finding from the review (with file references)
   - Acceptance criteria from the execution plan
   - Dependencies (met or unmet)
   - Suggested implementation approach

6. **Confirm and execute.** Ask the user to confirm, then use the `work-on-task` routine pattern:
   implement → test → document → validate → summarize.

7. **After completion.** Check off the task in the execution plan, update the progress table,
   and note the change in `planning/change_log.md`.

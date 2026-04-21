---
agent: agent
description: "[Planner] Verify the Executor's implementation against the original manifest"
---

# Verify Handoff

After an Executor finishes a Task Manifest, run this routine (as the Planner) to confirm intent was preserved and capture lessons.

> **Start a fresh chat for this routine.** Verify only needs the manifest file and `git diff` — not the Executor's full implementation transcript. A fresh session can save tens of thousands of input tokens on the Planner model.

## Steps

1. **Gather inputs.**
   - The manifest path (from `planning/handoffs/`).
   - The Executor's execution report (pasted, or from the last chat turn).
   - `git diff` across the touched repos.

2. **Check fidelity.** For each:
   - Every numbered step — was it executed? Any skipped?
   - Every `<files>` entry — was it changed as specified? Any files outside the list modified?
   - Every `<do_not>` — was it respected?
   - Every `<verification>` command — was it run, and did it pass?

3. **Inspect the diff for drift.** Look specifically for:
   - Unrelated refactors or stylistic edits.
   - New dependencies, new env vars, new files not authorized.
   - Changes that weaken tests (e.g., `assert True`, deleted cases, overly broad mocks).
   - Silent API or schema changes.

4. **Grade the handoff.**
   - **Pass** — manifest executed faithfully, verification green, no drift.
   - **Pass with follow-ups** — primary work done, minor deltas to track.
   - **Rework** — drift or failed verification requires another Executor pass with an updated manifest.

5. **Record the lesson.** If you found a recurring Executor failure mode (e.g., "adds unrequested error handling", "refactors adjacent code"), append a bullet to the "Planner/Executor lessons" section of `/memories/repo/workflow-patterns.md` via `/record-lesson`. Over time, feed these back into `handoff-manifest.instructions.md` as sharper `<do_not>` defaults.

6. **Decide next step.**
   - Pass → proceed to `/sprint-wrapup` or `/merge`.
   - Pass with follow-ups → add them as separate tasks in `planning/`.
   - Rework → update the manifest (bump version), hand back to Executor.

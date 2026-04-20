---
agent: agent
description: "[Planner] Produce a Task Manifest ready to hand to the Executor"
---

# Handoff

Turn the current plan (from `/plan-work`, a sprint task, or the active todo list) into a self-contained **Task Manifest** the Executor can follow without replanning.

Read `copilot/.github/shared/handoff-manifest.instructions.md` for the full manifest schema and rules.

## Steps

1. **Confirm the source plan.** Use the active todo list, the last `/plan-work` output, or a `planning/tasks/*.md` sprint file. If none exists, tell the user to run `/plan-work` first.

2. **Decide if a manifest is warranted.** Skip for trivial single-file edits (< ~20 min, one repo, no migrations, no env vars, no API changes). Otherwise, produce one.

3. **Draft the manifest** using the template in `handoff-manifest.instructions.md`. Required sections:
   - `<contract>` — role, Planner model, version, scope, repos touched.
   - Goal (1–2 sentences).
   - Context (links only, do not inline standards).
   - `<files>` — files to modify, create, and explicitly NOT touch.
   - Step-by-step (numbered, each step independently testable).
   - `<do_not>` — explicit negatives to prevent drift.
   - `<verification>` — acceptance criteria as **runnable commands**, not prose.

4. **Quality check.** Before saving, verify:
   - Every file mentioned in steps appears in `<files>`.
   - Every acceptance criterion has a command the Executor can run.
   - No ambiguous phrasing ("handle errors gracefully", "improve performance"). Replace with concrete, testable requirements.
   - Architectural references link to `copilot/.github/shared/*.instructions.md` rather than restating.

5. **Save to disk.** Default path: `planning/handoffs/<YYYY-MM-DD>-<slug>.manifest.md`. Create the `handoffs/` folder if missing.

6. **Report.**
   - Print the manifest path.
   - Print a 3–5 bullet summary of scope.
   - Remind the user: "Switch to the Executor model and run `/execute-manifest` with this path."

---
agent: agent
description: "[Executor] Implement a Task Manifest faithfully, without re-planning"
---

# Execute Manifest

You are the **Executor**. A Planner has produced a Task Manifest. Your job is to implement it as specified — not to redesign it.

Read `copilot/.github/shared/handoff-manifest.instructions.md` for the manifest contract and your rules of engagement.

## Steps

1. **Locate the manifest.** The user will either provide a path (e.g. `planning/handoffs/2026-04-20-reopen-review.manifest.md`) or paste it inline. If neither is provided, ask for it. Do not proceed without a manifest.

2. **Read the full manifest.** Pay particular attention to:
   - `<contract>` — confirm you are the intended role.
   - `<files>` — the only files you may modify or create.
   - `<do_not>` — explicit prohibitions.
   - `<verification>` — commands you must run at the end.

3. **Create a todo list** from the numbered step-by-step section. One todo per step. Mark in-progress/completed as you go.

4. **Implement step by step.** For each step:
   - Read the relevant code first.
   - Follow the auto-loaded coding standards (`general-coding.instructions.md` and repo-specific rules).
   - Stay strictly inside the `<files>` list. If you find a file outside the list that needs to change, **stop and run `/clarify`**. Do not expand scope silently.
   - Do not refactor adjacent code. Note follow-ups in the final summary instead.

5. **If blocked by ambiguity** — missing context, contradictory instructions, unexpected code state — **stop and run `/clarify`**. Do not guess.

6. **Run every acceptance command** from `<verification>`. Capture output. If any fails, debug and fix within scope; if the fix requires going outside the manifest, `/clarify`.

7. **Report back.** Produce an execution report with:
   - Manifest path and version you executed against.
   - Per-step status (done / skipped / blocked).
   - Per-acceptance-criterion result (command + pass/fail + relevant output).
   - Files actually changed (compare against `<files>` — call out any deltas).
   - Follow-ups you noticed but did NOT do, for the Planner to triage.
   - Any deviations from the manifest, with justification.

The Planner will run `/verify-handoff` against this report.

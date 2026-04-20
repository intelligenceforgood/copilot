---
applyTo: "**/planning/handoffs/**,**/planning/tasks/**"
---

# Task Manifest — Planner→Executor Handoff Contract

This document defines the **Task Manifest** format used when a Planner hands work to an Executor. A manifest is the single source of truth for _what_ gets built; the Executor implements it faithfully and does not re-plan.

## When a manifest is required

- Any task the Planner expects to take more than ~20 minutes of Executor time.
- Any task touching more than one repo.
- Any task involving migrations, new env vars, or public API changes.

Small, single-file, single-repo edits can skip the manifest and run `/work-on-task` directly.

## File location

- `planning/handoffs/<YYYY-MM-DD>-<slug>.manifest.md` (one file per handoff), OR
- Embedded as a fenced block in an existing `planning/tasks/*.md` sprint plan.

## Required structure

Use Markdown for readability **with XML tags around contract-critical sections**. Executors are instructed to treat XML-tagged content as authoritative; surrounding Markdown is context.

````markdown
# <Task title>

<contract>
**Role:** Executor
**Planner model:** <e.g., Claude Opus 4.7>
**Manifest version:** 1
**Estimated scope:** <S | M | L>
**Repos touched:** core, ui
</contract>

## Goal

<One or two sentences. High-level "done looks like…".>

## Context

<Background the Executor needs: linked PRD, relevant architecture notes, upstream decisions.
Link to `copilot/.github/shared/architecture-cheatsheet.instructions.md` sections rather than re-stating.>

<files>
### Files to modify

- `core/src/i4g/api/review.py` — add `POST /reviews/{id}/reopen` endpoint; reuse `ReviewStore.reopen()`.
- `core/tests/unit/api/test_review.py` — add coverage for the new endpoint (success + 404).
- `ui/apps/web/...` — wire a "Reopen" button in the review detail page.

### Files to create

- `core/src/i4g/api/schemas/reopen.py` — Pydantic request/response models.

### Files NOT to touch

- `core/src/i4g/worker/**` — out of scope for this task.
  </files>

## Step-by-step

1. …
2. …
3. …

<do_not>

- Do not change the database schema.
- Do not introduce new dependencies.
- Do not refactor adjacent code, even if it looks wrong — log it in `planning/change_log.md` as a follow-up instead.
- Do not skip tests.
  </do_not>

<verification>
### Acceptance criteria

- [ ] `conda run -n i4g pytest tests/unit/api/test_review.py -x` passes.
- [ ] `conda run -n i4g pre-commit run --files <changed files>` clean on pass 2.
- [ ] Manual: `curl -X POST .../reviews/<id>/reopen` returns 200 with expected JSON.
- [ ] UI: Reopen button appears only for closed reviews; click triggers success toast.

### Commands to run

```bash
conda run -n i4g pytest tests/unit/api/test_review.py -x
cd ui && pnpm test -- review-detail
```
````

</verification>

## If blocked

Run `/clarify` — produce a short structured question and stop. Do not guess.

```

## Rules for the Executor

1. **Do not re-plan.** The manifest is the plan. If it's wrong, stop and `/clarify`.
2. **Treat `<contract>`, `<files>`, `<do_not>`, and `<verification>` as authoritative.** If other prose conflicts with them, the XML tags win.
3. **Stay inside the "Files to modify / create" list.** If a file outside that list needs a change, stop and `/clarify`.
4. **Every acceptance criterion must be executed and reported** in the final summary (pass/fail, with the command output).
5. **Prefer minimal change.** No refactors, no stylistic edits outside touched lines.

## Rules for the Planner

1. **Write verification as commands**, not prose. Executors follow commands more reliably than narratives.
2. **Use `<do_not>` liberally** to prevent drift — models are more obedient about explicit negatives than implicit scope.
3. **Link, don't repeat** — reference `architecture-cheatsheet.instructions.md` and `general-coding.instructions.md` by path; do not inline their content.
4. **Version the manifest** if you revise it mid-execution. Bump `Manifest version` and tell the Executor to restart from the affected step.
```

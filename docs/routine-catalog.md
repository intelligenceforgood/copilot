# Routine Catalog

Complete reference for all available Copilot routines. Each routine is a prompt template that guides Copilot through a multi-step workflow.

## How to Invoke

1. Open Copilot Chat
2. Click the prompt picker (attachment icon) or type `/`
3. Select the routine from the list
4. Copilot executes the steps — you review and approve each action

## Role Labels

Every routine is tagged for the role that should run it:

- **[Planner]** — run on a deep-thinking model (e.g. Claude Opus 4.7). Use for architecture, design, breakdown, verification.
- **[Executor]** — run on a cheap, faithful model (e.g. Claude Sonnet 4.6, GPT‑5.4). Use for implementing a pre-written Task Manifest.
- **[Either]** — works on any model; useful for mechanical session chores.

> Role labels are about _function_, not specific model names. If you swap models (e.g. Mythos as Planner, Sonnet 4.6 as Executor), the labels still apply.

See [split-model workflow](#split-model-workflow-plannerexecutor) below for why this split exists and when to use it.

---

## Split-Model Workflow (Planner/Executor)

To stay within Copilot quota while keeping quality high, daily work is split between two roles:

1. **Planner** does deep/wide thinking: product design, architecture review, technical design, task breakdown, post-execution verification.
2. **Planner** produces a **Task Manifest** (see `copilot/.github/shared/handoff-manifest.instructions.md`) — a structured Markdown+XML document listing files, steps, `<do_not>` rules, and runnable verification commands.
3. **Executor** picks up the manifest and implements it faithfully, without re-planning. Scope is strictly limited to the manifest.
4. If blocked, **Executor** runs `/clarify` to bounce the ambiguity back to the Planner instead of guessing.
5. **Planner** runs `/verify-handoff` at the end to confirm fidelity and record lessons.

Typical session:

```
[Planner]  /rehydrate-session → /plan-work → /handoff
           ↓ (switch model)
[Executor] /execute-manifest   (→ /clarify if blocked)
           ↓ (switch model)
[Planner]  /verify-handoff → /sprint-wrapup → /merge
```

Small tasks (single file, no migrations, no API change) can skip the manifest: `/plan-work` → `/work-on-task` directly.

---

## Session Management

### rehydrate-session **[Either]**

**When:** Start of every working session.
**What it does:**

- Checks git status across all repos
- Reads recent change log entries
- Loads repo memory (lessons learned, workflow patterns)
- Identifies the active task/sprint
- Recommends what to work on next

### sprint-wrapup **[Planner]**

**When:** End of a sprint or significant work session.
**What it does:**

- Summarizes completed work from git history
- Updates task checkboxes in planning files
- Appends change log entry
- Lists manual steps for the user (migrations, deploys)
- Assesses merge readiness and risks
- Records lessons learned

### record-lesson **[Either]**

**When:** You discover a pattern, pitfall, or useful technique.
**What it does:**

- Categorizes the lesson (coding, architecture, workflow, config)
- Saves to repo memory for future sessions
- Checks if enough similar lessons warrant promotion to a permanent standard

---

## Planning & Handoff

### plan-work **[Planner]**

**When:** Starting a new feature or task.
**What it does:**

- Clarifies scope and identifies affected repos
- Checks architecture patterns
- Breaks work into ordered, testable steps
- Flags risks (API changes, migrations, new env vars)
- Creates a todo list to track progress
- Ends by recommending `/handoff` for non-trivial work

### handoff **[Planner]**

**When:** A plan is ready and you want an Executor to implement it.
**What it does:**

- Converts the current plan/todo list into a **Task Manifest** file
- Enforces the schema in `copilot/.github/shared/handoff-manifest.instructions.md` (contract, files, step-by-step, `<do_not>`, runnable verification)
- Saves to `planning/handoffs/<YYYY-MM-DD>-<slug>.manifest.md`
- Reports the path and tells you to switch to the Executor model

### execute-manifest **[Executor]**

**When:** You have a Task Manifest ready to implement.
**What it does:**

- Reads the manifest, creates a todo list from its steps
- Implements each step strictly inside the `<files>` scope
- Runs every acceptance command in `<verification>`
- Produces a structured execution report (per-step status, per-criterion result, files changed, follow-ups)
- Bounces to `/clarify` if ambiguous — does not guess

### clarify **[Executor]**

**When:** You're blocked by ambiguity, contradiction, or unexpected state mid-execution.
**What it does:**

- Halts work immediately; does not commit speculative changes
- Produces a structured `<clarify>` block (manifest, step, category, block, options, recommendation)
- Tells you to switch to the Planner for a manifest update

### verify-handoff **[Planner]**

**When:** An Executor has finished implementing a manifest.
**What it does:**

- Diffs intent (manifest) vs. result (git diff + execution report)
- Checks every step, every `<files>` entry, every `<do_not>`, every verification command
- Grades: Pass / Pass with follow-ups / Rework
- Captures recurring Executor failure modes as lessons

---

## Implementation

### work-on-task **[Either]**

**When:** Implementing a specific task without a manifest (small tasks, or when Planner and Executor are the same model).
**What it does:**

- Reads existing code before modifying
- Implements following auto-loaded coding standards
- Writes/updates unit tests
- Updates documentation if behavior changed
- Runs pre-commit on changed files
- Summarizes what was done

> For Executor-model runs on non-trivial work, prefer `/execute-manifest` — stricter scope discipline.

### fix-bug **[Either]**

**When:** A bug needs investigation and fixing.
**What it does:**

- Reproduces the issue (reads errors, traces code paths)
- Root-cause analysis (reads code, checks recent changes)
- Applies minimal correct fix
- Writes regression test that would have caught the bug
- Runs full test suite to verify no collateral damage
- Records the lesson for future prevention

> Deep/unclear bugs benefit from a Planner pass first (`/plan-work` to scope the investigation) before handing a tight manifest to an Executor.

---

## Quality & Deployment

### pre-merge-review **[Either]**

**When:** You want to review changes without committing.
**What it does:**

- Identifies all repos with changes
- Runs the full code audit (type hints, docstrings, imports, dead code, secrets)
- Executes quality gates per repo (pre-commit double-pass, UI build, terraform fmt)
- Runs unit tests
- Verifies docs/config are updated
- Produces a summary with issues and fixes
- Does NOT commit or push — use `/merge` for the full workflow

### merge **[Either]**

**When:** You're ready to review, commit, and push all changed repos.
**What it does:**

- Runs the full pre-merge review (same as `/pre-merge-review`)
- Checks for stray files, secrets, and debug artifacts
- Commits all changed repos with conventional commit messages
- Pushes to `origin/main`
- Produces a combined summary (review findings + commit/push status)

This is the standard end-of-task routine. No need to run `/pre-merge-review` separately.

### deploy-to-dev **[Either]**

**When:** Pushing changes to the i4g-dev environment.
**What it does:**

- Verifies pre-merge review was completed
- Runs local smoke test
- Identifies which Docker images need rebuilding
- Checks for database migrations
- Guides through build and deploy
- Reminds about dev/prod parity

### manual-verification **[Either]**

**When:** After a deployment, to verify everything works.
**What it does:**

- Checks service health
- Tests key API endpoints
- Verifies UI functionality (if deployed)
- Checks worker/job execution (if deployed)
- Compares results with expectations
- Signs off or recommends escalation

### check-log **[Either]**

**When:** A Cloud Run job or service fails and you have the log filter from GCP Console.
**What it does:**

- Takes a pasted GCP logging filter query
- Fetches logs via `gcloud logging read`
- Identifies the root error from stack traces
- Reads the relevant source code
- Proposes (and optionally implements) a fix

---

## Platform Hardening

### hardening-sprint **[Planner]**

**When:** Starting a platform hardening work session (Phase 0–3 of the April 2026 architecture review).
**What it does:**

- Loads the architecture review and execution plan
- Identifies the current phase and next unchecked task
- **Checks model routing** — warns if the next task is Planner-only (tasks 1.4, 3.6)
- Loads relevant source files and architecture context for the task
- Presents the work plan with acceptance criteria and dependencies
- Hands off to `/handoff` → `/execute-manifest` for the implementation itself
- Checks off the task and updates the progress tracker

**Files referenced:**

- `planning/tasks/platform-review-2026-04-17.md` (review + execution guidance)
- `planning/tasks/platform-hardening-execution.md` (execution plan with checkboxes)

---

## Model Guidance (April 2026)

Role labels, not specific models. Today's recommended mapping:

| Role                   | Recommended model | Multiplier | Use for                                                                       |
| ---------------------- | ----------------- | ---------- | ----------------------------------------------------------------------------- |
| Planner                | Claude Opus 4.7   | 7.5x       | Architecture, design, manifests, verification, hardest bugs                   |
| Executor (default)     | Claude Sonnet 4.6 | 1x         | Faithful manifest execution; shares prompt conventions with Opus              |
| Executor (alternative) | GPT‑5.4           | 1x         | Second-opinion runs; tasks where Sonnet stalls                                |
| Executor (mechanical)  | Mini-class        | 0.33x      | Renames, import sweeps, boilerplate expansion from a fully-specified manifest |

**Rule of thumb:** anything under 3x multiplier is acceptable as an Executor. Reserve the Planner for work where a mistake cascades (architecture, data model, public API).

---

## Cost Economics (read before adopting the split-model workflow)

The split-model workflow is not automatically cheaper. It wins in specific regimes and loses in others. Calibrate accordingly.

### What actually drives cost

1. **Iteration count, not routine count.** A premium request = one user turn. A sprint's implementation is 15–40 turns of read → edit → run tests → fix. That iteration count dominates — not the number of routines you invoke.
2. **Session length.** Every turn re-sends the full conversation history as input tokens. Turn 50 costs roughly 50× the input of turn 1. Under token-based billing this is the single largest lever.
3. **Auto-loaded instructions.** Every `.github/copilot-instructions.md` and auto-loaded `*.instructions.md` is re-sent on every turn. Keep them lean.
4. **Opus reads are 7.5× everything.** When Opus reads a large file, you pay 7.5× input cost. Prefer having the Executor gather context and summarize up to the Planner, not the other way around.

### When split-model wins

- Multi-file, multi-repo work with 15+ implementation turns.
- Batched manifests covering several sprints at once (Planner cost is mostly fixed per handoff).
- Work where a scope slip or silent refactor would be expensive to untangle later.

### When split-model LOSES (just use Planner-inline)

- Tasks under ~8 Executor turns, < 3 files, single repo, no migrations/env/API changes.
- Exploratory or throwaway work where the plan itself will change mid-flight.
- Situations where you expect 3+ `/clarify` round-trips — each clarify is a Planner turn at 7.5×.

### Cost-control rituals

- **Start a fresh chat at handoff boundaries.** Executor doesn't need Planner's thinking-out-loud history. `/verify-handoff` only needs the manifest + `git diff`.
- **Batch sprints into one manifest** when they share files and verification.
- **Trim `copilot-instructions.md` files** — audit for load-bearing content only; every token is re-sent on every turn across every workspace folder.
- **Prefer parallel tool calls** to reduce round-trips (already in the general instructions).
- **Under token billing, consider BYOK for the Executor role** (your own Anthropic/OpenAI key) — this is a contingency lever if Pro+ runs tight.

### Rough per-sprint cost, mobile-prototype scale (~6k lines output, ~25 implementation turns)

| Path                                             | Estimated premium requests |
| ------------------------------------------------ | -------------------------- |
| Pure Opus 4.7 end-to-end                         | ~150–300                   |
| Split, typical manifest                          | ~60–95                     |
| Split, tight manifest + batched + fresh sessions | ~40–50                     |

A 5-sprint project under pure Opus can exhaust a 1,500/month quota; under a disciplined split flow it lands around 200–450.

---

## Adding New Routines

See [customization-guide.md](customization-guide.md) for how to create new routines.

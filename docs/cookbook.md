# Cookbook — Real-World Workflow Examples

Concrete examples of how to interact with Copilot in the i4g workspace. Read these to understand what a typical session looks like and what prompts to use at each step.

---

## The Sprint Lifecycle

This is the most common pattern — taking a feature from idea to merged and deployed.

### Phase 1: Product Definition

> **You:** As a chief product officer, we have received feedback about how feature XYZ works. Here is a list of brief user comments: [paste comments]. Please review these requests and identify a coherent theme. Research similar products in the industry to see how others are addressing this area. If you can come up with any innovation, that would be even better. Please write a PRD and put it in the `planning/proposals/` folder for review.

Copilot will:

- Synthesize the user feedback into themes
- Research industry patterns
- Produce a structured PRD at `planning/proposals/<feature-name>.md`

**Tip:** Give Copilot the raw user comments, not your interpretation. It draws better conclusions from raw signals.

---

### Phase 2: Implementation Planning

> **You:** As a chief architect and lead developer, please review the PRD `planning/proposals/<feature-name>.md` and draft an implementation plan. The plan should be phased, with each phase completing in 5–10 weeks. Each phase should have a task list with checkboxes so we can mark tasks complete as we go.

Alternatively, invoke the **`plan-work`** routine from the prompt picker to run a structured planning workflow.

The result is a plan file at `planning/tasks/<feature-name>.md` with:

- Phased breakdown
- `- [ ]` checkboxes per task
- Risk flags for breaking changes, migrations, or new env vars

**Why checkboxes matter:** Even across multiple sessions or weeks, the plan stays current. At the start of each session, `rehydrate-session` reads the plan and tells Copilot which tasks remain.

---

### Phase 3: Implementation

For each task in the plan, use the appropriate routine:

| Situation                                     | Routine                |
| --------------------------------------------- | ---------------------- |
| Implementing a well-defined task              | `work-on-task`         |
| Diagnosing an unexpected error                | `fix-bug`              |
| Starting a large change across multiple repos | `plan-work` (sub-plan) |

**example:**

> **You:** [select `work-on-task` from prompt picker]

Or just describe the task and mention the routine:

> **You:** Work on task 2.3 from `planning/tasks/threat-feeds.md` — implement the eCrimeX polling job.

Copilot will:

- Read the relevant existing code before touching anything
- Implement following the auto-loaded coding standards for the file type
- Write unit tests
- Update docs if behavior changed
- Run pre-commit hooks on changed files

**Session length tip:**

> **You:** We have written a lot of code in this session. Please recommend whether we should start a new chat session to continue. If yes, please draft a prompt for the next session that captures the current context.

Copilot will assess the session length and produce a ready-to-paste "Continue from here" prompt.

---

### Phase 4: Pre-Merge Review

Before merging any branch, run the `pre-merge-review` routine:

> **You:** [select `pre-merge-review` from prompt picker]

Or:

> **You:** I'm ready to merge the work we have done for this sprint. Please do a pre-merge review.

Copilot will:

- Audit all changed files against coding standards
- Run pre-commit hooks (double-pass)
- Run unit tests
- Verify docs and config are updated
- Produce a summary of issues found, fixes applied, and remaining items

**The review is not complete until the pre-commit double-pass exits clean** — formatting tools run on pass 1, and pass 2 must show no remaining failures.

---

### Phase 5: Deploy to Dev

> **You:** [select `deploy-to-dev` from prompt picker]

Or:

> **You:** Please give me a checklist for deploying the new changes to the dev environment, including everything I need to do — images, services, Terraform, and Alembic migrations.

Copilot will:

- Identify which services changed and which images to rebuild
- Produce the exact build commands
- List any Alembic migration steps
- List any Terraform changes
- Post-deploy verification steps

---

### Phase 6: Verify in Dev

> **You:** [select `manual-verification` from prompt picker]

Or:

> **You:** The new code is deployed to dev. Please walk me through manual verification.

For **browser console errors:**

> **You:** The new code is causing errors in the local env. Here are the errors from the browser console: [paste errors].

For **Cloud Run log errors** that span multiple entries (hard to paste):

> **You:** The new code is causing errors in the dev environment. The error spans multiple GCP log entries and I can't paste them here. Please use `gcloud logging` to inspect the logs yourself. The Cloud Run service name is `core-svc`.

Copilot will use the `gcloud logging` CLI to query and analyze the logs directly.

---

### Phase 7: Sprint Wrapup

> **You:** [select `sprint-wrapup` from prompt picker]

Or:

> **You:** We're done with this sprint. Please summarize what was done, update the task checkboxes, write a change log entry, and tell me what risks exist and what I need to manually verify. Is this a good merge point?

Copilot produces:

- Updated `- [x]` checkboxes in the plan file
- An entry in `planning/change_log.md`
- A list of manual steps (migrations, deploys, config changes)
- A risk assessment
- A merge readiness statement

---

## The Feedback Loop (Discover → Record → Accumulate → Promote)

This is how the system gets smarter over time. Use it whenever you learn something from a session.

### Step 1: Discover

During any session, you find something worth remembering:

- A bug pattern that recurs
- A workflow shortcut
- A Copilot mistake to avoid
- An architecture nuance that caused confusion

### Step 2: Record

> **You:** [select `record-lesson` from prompt picker]

Or:

> **You:** Please save this lesson to repo memory: When fixing a test that catches an exception, always import the specific exception class rather than catching `Exception`. Ruff flags the redundant broad catch.

Copilot categorizes the lesson and saves it to `/memories/repo/lessons-learned.md`.

### Step 3: Accumulate

Over several sessions, related lessons pile up. Copilot reads repo memory at `rehydrate-session` and applies past lessons automatically.

### Step 4: Promote

When Copilot notices 3+ lessons on the same topic:

> **You:** We've recorded several lessons about Pydantic alias handling. Please promote these into a permanent standard.

Copilot will:

- Consolidate the lessons into a concise rule set
- Add it to `copilot/.github/standards/python.instructions.md` (or create a new standards file)
- Update the repo memory entry to note the promotion

### Example Interaction

A session discovers that `except (SpecificError, Exception):` is redundant and ruff flags it:

> "Save this to repo memory: The pattern `except (SpecificError, Exception):` is redundant — `Exception` subsumes the specific exception. Always catch the specific exception only. ruff flags this under rule E722-adjacent checks."

Future sessions will have this in memory and Copilot will flag the pattern proactively in code review.

---

## Other Common Interactions

### Debugging Remote Errors

> **You:** The new code is causing errors in the dev env. The error spans multiple GCP log entries so I'm unable to copy and paste here. You should use gcloud logging to inspect the log yourself. The Cloud Run Job name is `report-job`.

Copilot will run:

```bash
gcloud logging read 'resource.type="cloud_run_job" AND resource.labels.job_name="report-job"' \
  --project=i4g-dev --limit=50 --format=json
```

...then analyze the logs and identify the root cause.

---

### Doc Refactoring

> **You:** The section on storage in `docs/book/architecture/data-pipeline.md` is out of date. Please review the current implementation in `src/i4g/services/factories.py` and update the doc to match reality.

Use **`work-on-task`** if the doc update is part of a tracked task, otherwise just describe it directly.

---

### Bug Investigation

Use the **`fix-bug`** routine:

> **You:** [select `fix-bug`]

Or describe the bug directly:

> **You:** Users report that saved searches disappear after a page reload. The search was just saved successfully — the API returned 200. Please investigate.

Copilot will:

- Reproduce the issue by reading the relevant code paths
- Identify the root cause
- Apply a minimal fix
- Write a regression test
- Run the test suite

---

### Feature Review Before Scoping

> **You:** As a product manager, please review the current implementation of the campaign governance feature (see `src/i4g/api/governance.py`) and compare it against the PRD at `planning/prd_production.md` section 4.2. Identify any gaps and draft a list of follow-up tasks.

---

## Tips for Effective Sessions

1. **Use routines, not raw prompts.** Routines encode the team's best practices. A raw "review my code" prompt will miss steps that `pre-merge-review` catches systematically.

2. **Be explicit about context.** Paste error messages, file paths, and relevant snippets. Copilot works best with concrete inputs.

3. **Start fresh sessions for new phases.** After merging a sprint, start a new chat. Old context from a completed sprint adds noise.

4. **Trust the standards.** When Copilot insists on a type hint, specific exception, or docstring — it's enforcing team conventions, not being pedantic.

5. **Record surprises.** Any time something takes longer than expected because of an architecture quirk or tooling gotcha — record it immediately with `record-lesson`.

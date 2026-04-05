# Routine Catalog

Complete reference for all available Copilot routines. Each routine is a prompt template that guides Copilot through a multi-step workflow.

## How to Invoke

1. Open Copilot Chat
2. Click the prompt picker (attachment icon) or type `#`
3. Select the routine from the list
4. Copilot executes the steps — you review and approve each action

---

## Session Management

### rehydrate-session

**When:** Start of every working session.
**What it does:**

- Checks git status across all repos
- Reads recent change log entries
- Loads repo memory (lessons learned, workflow patterns)
- Identifies the active task/sprint
- Recommends what to work on next

### sprint-wrapup

**When:** End of a sprint or significant work session.
**What it does:**

- Summarizes completed work from git history
- Updates task checkboxes in planning files
- Appends change log entry
- Lists manual steps for the user (migrations, deploys)
- Assesses merge readiness and risks
- Records lessons learned

### record-lesson

**When:** You discover a pattern, pitfall, or useful technique.
**What it does:**

- Categorizes the lesson (coding, architecture, workflow, config)
- Saves to repo memory for future sessions
- Checks if enough similar lessons warrant promotion to a permanent standard

---

## Implementation

### plan-work

**When:** Starting a new feature or task.
**What it does:**

- Clarifies scope and identifies affected repos
- Checks architecture patterns
- Breaks work into ordered, testable steps
- Flags risks (API changes, migrations, new env vars)
- Creates a todo list to track progress

### work-on-task

**When:** Implementing a specific task.
**What it does:**

- Reads existing code before modifying
- Implements following auto-loaded coding standards
- Writes/updates unit tests
- Updates documentation if behavior changed
- Runs pre-commit on changed files
- Summarizes what was done

### fix-bug

**When:** A bug needs investigation and fixing.
**What it does:**

- Reproduces the issue (reads errors, traces code paths)
- Root-cause analysis (reads code, checks recent changes)
- Applies minimal correct fix
- Writes regression test that would have caught the bug
- Runs full test suite to verify no collateral damage
- Records the lesson for future prevention

---

## Quality & Deployment

### pre-merge-review

**When:** Before merging any branch.
**What it does:**

- Identifies all repos with changes
- Runs the full code audit (type hints, docstrings, imports, dead code, secrets)
- Executes quality gates per repo (pre-commit double-pass, UI build, terraform fmt)
- Runs unit tests
- Verifies docs/config are updated
- Produces a summary with issues and fixes

### deploy-to-dev

**When:** Pushing changes to the i4g-dev environment.
**What it does:**

- Verifies pre-merge review was completed
- Runs local smoke test
- Identifies which Docker images need rebuilding
- Checks for database migrations
- Guides through build and deploy
- Reminds about dev/prod parity

### manual-verification

**When:** After a deployment, to verify everything works.
**What it does:**

- Checks service health
- Tests key API endpoints
- Verifies UI functionality (if deployed)
- Checks worker/job execution (if deployed)
- Compares results with expectations
- Signs off or recommends escalation

### check-log

**When:** A Cloud Run job or service fails and you have the log filter from GCP Console.
**What it does:**

- Takes a pasted GCP logging filter query
- Fetches logs via `gcloud logging read`
- Identifies the root error from stack traces
- Reads the relevant source code
- Proposes (and optionally implements) a fix

---

## Adding New Routines

See [customization-guide.md](customization-guide.md) for how to create new routines.

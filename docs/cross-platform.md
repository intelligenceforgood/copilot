# Cross-Platform Guide — Using This System with Claude Code

This guide is for developers who use Claude Code (the CLI tool) instead of, or in addition to, GitHub Copilot in VS Code. It explains how the system maps between the two environments and how to adopt the workflows manually.

---

## Concept Mapping

The i4g workflow system is built around VS Code Copilot constructs. Here is how each piece translates to Claude Code concepts:

| VS Code Copilot                                      | Claude Code equivalent                                    | Notes                                                                         |
| ---------------------------------------------------- | --------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `.github/prompts/*.prompt.md`                        | Slash commands (e.g., `/review`, `/deploy`)               | Claude Code slash commands are defined in `CLAUDE.md` or as tools — see below |
| `.github/standards/*.instructions.md` with `applyTo` | `CLAUDE.md` context sections or Skills                    | Claude Code reads `CLAUDE.md` at startup; add language-specific rules there   |
| `.github/copilot-instructions.md`                    | `CLAUDE.md` (repo or workspace level)                     | Claude Code reads `CLAUDE.md` from the project root                           |
| `.github/shared/` reference docs                     | Referenced in `CLAUDE.md` with explicit read instructions | Claude Code can read any file in the project                                  |
| `/memories/repo/`                                    | `CLAUDE.md` `# Notes` section or a committed `notes.md`   | Claude Code memory is managed via `CLAUDE.md` or custom files                 |

---

## Setting Up `CLAUDE.md`

Claude Code reads a file called `CLAUDE.md` from the project root at startup. This is the equivalent of `copilot-instructions.md`.

### Step 1: Create a `CLAUDE.md` at each repo root

Use the structure from each repo's `copilot-instructions.md` as a starting point. Adapt it to Claude Code's expected format:

````markdown
# i4g/core — Claude Code Context

## Environment

- Conda env: `i4g`
- All commands: `conda run -n i4g ...`

## Build & Test

```bash
pip install -e .
uvicorn i4g.api.app:app --reload
pytest tests/unit
```
````

## Architecture

Read `core/.github/shared/architecture-cheatsheet.instructions.md` for the full reference.

## Shared Standards

Read `copilot/.github/shared/general-coding.instructions.md` for coding conventions.

````

Claude Code will read the `CLAUDE.md` at session start and follow any `Read <file>` instructions by loading those files into context.

### Step 2: Copy shared standards into `CLAUDE.md` sections

Since Claude Code does not support `applyTo` auto-loading, embed the standards directly in `CLAUDE.md` or add explicit sections:

```markdown
## Python Standards (applies to all .py files)
<!-- Paste contents of copilot/.github/standards/python.instructions.md here -->
````

Or instruct Claude Code to read them:

```markdown
## Standards

When editing Python files, read `copilot/.github/standards/python.instructions.md`.
When editing TypeScript files, read `copilot/.github/standards/typescript-react.instructions.md`.
When editing Terraform files, read `copilot/.github/standards/terraform.instructions.md`.
```

---

## Translating Routines to Slash Commands

VS Code Copilot routines are `.prompt.md` files invoked from the chat picker. In Claude Code, the equivalent is a **slash command** defined in `CLAUDE.md`.

### Step 1: Add a `## Commands` section to `CLAUDE.md`

```markdown
## Commands

### /rehydrate

Check `git status -sb`, read `planning/change_log.md` last 20 lines, read `/memories/repo/lessons-learned.md` and `/memories/repo/workflow-patterns.md`, read the active task plan in `planning/tasks/`. Report: what changed recently, what the active task is, what to work on next.

### /review

Execute the pre-merge review checklist from `copilot/.github/shared/pre-merge-checklist.instructions.md` against all files changed in the current branch. Run: `pre-commit run --all-files` (pass 1), `git add -u`, then `pre-commit run --all-files` (pass 2 — must be clean). Run `pytest tests/unit`. Report all issues found, fixes applied, and remaining items.

### /deploy

Follow the deploy checklist from `copilot/.github/prompts/deploy-to-dev.prompt.md`. Identify which images changed, produce exact build commands, list migrations, list Terraform changes.

### /lesson <text>

Save the following lesson to repo notes: categorize it as coding/architecture/workflow/config, append it to `NOTES.md`, and assess whether enough similar lessons have accumulated to promote to a permanent standard.
```

### Step 2: Invoke with `/command` in Claude Code chat

In Claude Code, type `/rehydrate` or `/review` to invoke the corresponding workflow.

---

## Translating the Memory System

VS Code Copilot's memory system (`/memories/repo/`) is internal to VS Code. In Claude Code, persist lessons differently:

### Option A: Committed Notes File

Add a `NOTES.md` to each repo (committed to git). Claude Code reads it at session start if `CLAUDE.md` instructs it to.

In `CLAUDE.md`:

```markdown
## Session Start

At the start of every session, read `NOTES.md` for past lessons and apply them to this session.
```

### Option B: `CLAUDE.md` Notes Section

Append lessons directly to `CLAUDE.md` under a `## Notes` section:

```markdown
## Notes (Accumulated Lessons)

- 2026-03-15: `except (SpecificError, Exception)` is redundant — ruff flags it. Always catch specific exceptions only.
- 2026-03-10: Pydantic alias_generator handles snake_case↔camelCase automatically. Never write manual translation functions.
```

When a lesson deserves promotion to a permanent standard, move it from `## Notes` into the appropriate `## Standards` section.

---

## The Sprint Lifecycle in Claude Code

The same sprint lifecycle from [cookbook.md](cookbook.md) works in Claude Code. Use the slash commands instead of the prompt picker:

| VS Code step                             | Claude Code step                                                                                |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Open prompt picker → `rehydrate-session` | Type `/rehydrate`                                                                               |
| Open prompt picker → `plan-work`         | Type `/plan` or describe the task                                                               |
| Open prompt picker → `work-on-task`      | Describe what to implement (no equivalent slash command needed — Claude Code is conversational) |
| Open prompt picker → `pre-merge-review`  | Type `/review`                                                                                  |
| Open prompt picker → `deploy-to-dev`     | Type `/deploy`                                                                                  |
| Open prompt picker → `record-lesson`     | Type `/lesson <text>`                                                                           |

---

## Inspired by gstack

The [gstack project](https://github.com/garrytan/gstack) demonstrates a mature Claude Code skill framework with commands like `/review`, `/ship`, `/investigate`, and `/qa`. The i4g system uses a similar structure. If you adopt Claude Code, studying gstack's `CLAUDE.md` patterns is worthwhile — the concepts transfer directly.

Key ideas from gstack worth borrowing:

- **One `CLAUDE.md` per repo** with environment, build, test, and standards sections
- **Slash commands for every workflow phase** — don't rely on freeform conversation for structured tasks
- **An explicit `## Notes` section** updated after each session with lessons learned

---

## Should the `copilot/` Repo Be Renamed?

The `copilot/` repo name reflects its origin as a VS Code Copilot intelligence repo. If the team adopts Claude Code broadly, a more neutral name (e.g., `ai-workflows/` or `dev-intelligence/`) would be more accurate.

**Recommendation for this phase:** Keep the name `copilot/`. The VS Code/Copilot path is well-established and actively used. Add Claude Code support as a layer (via `CLAUDE.md` files in product repos) without restructuring the repo. If/when Claude Code usage grows to match Copilot usage, revisit the naming.

---

## Quick Checklist for Claude Code Adoption

- [ ] Create `CLAUDE.md` at each repo root using the repo's `copilot-instructions.md` as a template
- [ ] Add language standards to `CLAUDE.md` (copy or reference)
- [ ] Add a `## Commands` section with `/rehydrate`, `/review`, `/deploy`, `/lesson`
- [ ] Add a `## Notes` section (empty) for accumulated lessons
- [ ] Optionally: create `NOTES.md` as a separate committed file for a longer lesson history
- [ ] Test each slash command with a simple invocation to confirm Claude Code reads it correctly

# Copilot Workflow System — Entry Point

This is **the single doc to read first**. It explains what the system is, how it is structured, and where each piece lives. From here you can navigate to any other guide.

---

## What This System Is

The `copilot/` repo provides shared AI-assistant intelligence for the entire i4g workspace. It ensures that when you open any repo in VS Code and ask Copilot a question or invoke a workflow, Copilot already knows:

- How the platform is architected
- Which coding standards apply to the file you are editing
- What the workflows are for planning, implementing, reviewing, and deploying
- What lessons the team has learned from past sessions

The system is designed around three principles:

1. **Zero friction** — Copilot knows what it needs without you configuring anything per session.
2. **Cumulative learning** — Lessons discovered in one session are recorded and applied in future sessions.
3. **Consistency** — Every developer (and every AI session) follows the same standards and workflows.

---

## Architecture at a Glance

```
┌─────────────────────────────────────────────────────────────────┐
│  Layer 3: Routines (.prompt.md)                                 │
│  Invoked on demand from the Copilot chat prompt picker          │
│  Example: pre-merge-review, deploy-to-dev, plan-work            │
├─────────────────────────────────────────────────────────────────┤
│  Layer 2: Auto-loaded Standards (.instructions.md + applyTo)    │
│  Loaded automatically when you edit a matching file type        │
│  Example: python.instructions.md loads when editing any .py     │
├─────────────────────────────────────────────────────────────────┤
│  Layer 1: Workspace Instructions (copilot-instructions.md)      │
│  Always loaded — repo-specific context (env, build, arch)       │
│  Each repo has its own minimal file; shared content lives here  │
├─────────────────────────────────────────────────────────────────┤
│  Foundation: Memory System (/memories/repo/, /memories/)        │
│  Persistent lessons and patterns that survive across sessions   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Where Everything Lives

### This repo (`copilot/`)

| Path                              | What it contains                                                            |
| --------------------------------- | --------------------------------------------------------------------------- |
| `.github/copilot-instructions.md` | Workspace-level instructions loaded in every Copilot conversation           |
| `.github/prompts/`                | 10 routine prompt files — invokable from the chat prompt picker             |
| `.github/standards/`              | 4 auto-loaded standards files (Python, TypeScript/React, Terraform, config) |
| `.github/shared/`                 | 3 shared reference docs (architecture, general coding, pre-merge checklist) |
| `.github/repo-templates/`         | Template for creating a new repo's `copilot-instructions.md`                |
| `docs/`                           | Human-readable guides (this file and its siblings)                          |

### Per-product repos (`core/`, `ui/`, `ssi/`, `infra/`, etc.)

Each product repo has only `.github/copilot-instructions.md` with **repo-specific** context:

- Conda environment name
- Build/test commands
- Repo-specific architecture notes
- Any non-standard pre-commit procedure

Everything else (shared standards, routines, platform architecture) lives in `copilot/`.

---

## Available Routines

All 10 routines are described in [routine-catalog.md](routine-catalog.md). Quick reference:

| Routine               | When                                  |
| --------------------- | ------------------------------------- |
| `rehydrate-session`   | Start of every session                |
| `plan-work`           | Before starting a feature/task        |
| `work-on-task`        | Implementing code                     |
| `fix-bug`             | Diagnosing a bug                      |
| `check-log`           | Diagnosing a failed Cloud Run job/svc |
| `pre-merge-review`    | Before merging                        |
| `deploy-to-dev`       | Deploying to dev environment          |
| `manual-verification` | After deploying                       |
| `sprint-wrapup`       | End of sprint                         |
| `record-lesson`       | Saving a new lesson                   |

**How to invoke:** Open Copilot Chat → click the **`+` (attachment) icon** in the chat input → select **"Prompt..."** → pick the routine name.

---

## Available Standards

Standards auto-load when you open a matching file. You do not need to invoke them.

| File                               | Triggers on                     | Contents                                                |
| ---------------------------------- | ------------------------------- | ------------------------------------------------------- |
| `python.instructions.md`           | `**/*.py`                       | Type hints, docstrings, Black, Pydantic, error handling |
| `typescript-react.instructions.md` | `**/*.ts`, `**/*.tsx`           | Naming, hooks, Prettier, CSS Modules, accessibility     |
| `terraform.instructions.md`        | `**/*.tf`                       | Naming, sensitive values, file layout, dev-first        |
| `settings-config.instructions.md`  | `**/settings*.toml`, `**/.env*` | `get_settings()`, env var contract, smoke discipline    |

---

## The Learning Loop

The system gets smarter over time through a feedback loop:

```
Work on a task
     ↓
Discover something (pattern, pitfall, preference)
     ↓
Record it → use the `record-lesson` routine
     ↓
Lesson saved to repo memory (persists across sessions)
     ↓
3+ similar lessons accumulate
     ↓
Promote to .instructions.md (permanent, auto-loaded)
     ↓
System is now smarter for everyone — repeat
```

---

## Guides in This Folder

| Guide                                            | Purpose                                                             |
| ------------------------------------------------ | ------------------------------------------------------------------- |
| [README.md](README.md)                           | This file — start here                                              |
| [onboarding.md](onboarding.md)                   | New developer guide — workspace setup, daily workflow, key commands |
| [routine-catalog.md](routine-catalog.md)         | Full description of all 10 routines                                 |
| [cookbook.md](cookbook.md)                       | Example interactions — sprint lifecycle, debugging, doc work        |
| [customization-guide.md](customization-guide.md) | How to extend the system — add routines, standards, shared docs     |
| [cross-platform.md](cross-platform.md)           | Using these workflows with Claude Code instead of VS Code Copilot   |

---

## Quick Start (3 steps)

1. **Verify setup.** Open VS Code with `i4g.code-workspace`. Confirm `copilot/` appears in the Explorer pane. Confirm GitHub Copilot extension is active.

2. **Run the rehydration routine.** At the start of every session, open Copilot Chat and run `rehydrate-session`. Copilot will check git status, load recent context, and tell you what to work on.

3. **Use routines for each workflow step.** Don't type raw instructions — use the prompt picker to select the right routine. The routines encode the team's best practices and ensure nothing is missed.

---

## Common Questions

**Q: Do I need to tell Copilot about the project structure each session?**
No. The `rehydrate-session` routine does this. It reads git status, recent change logs, and repo memory — so Copilot starts each session with full context.

**Q: Where do I report a Copilot mistake or learn from an error?**
Use the `record-lesson` routine. It saves the insight to repo memory so future sessions (yours and others') avoid the same issue.

**Q: The routine mentions a file path that doesn't exist. What do I do?**
Check [routine-catalog.md](routine-catalog.md) for the current accurate file paths. If you find a stale reference, fix it and use the `record-lesson` routine to note it.

**Q: I'm not using VS Code. Can I still use this system?**
Yes — read [cross-platform.md](cross-platform.md) for a guide on adopting these workflows in Claude Code.

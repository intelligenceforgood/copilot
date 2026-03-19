# Developer Onboarding — Working with Copilot

Welcome to the I4G Platform. This guide explains how to use GitHub Copilot effectively in this workspace. Copilot has been configured with project-specific knowledge, coding standards, and developer routines so you can be productive from day one.

## Workspace Setup

The i4g workspace is multi-root — it has several repos open simultaneously:

| Repo            | Language         | Purpose                              |
| --------------- | ---------------- | ------------------------------------ |
| `core/`         | Python           | API, workers, business logic         |
| `ui/`           | TypeScript/React | Analyst console (Next.js)            |
| `ssi/`          | Python           | Scam-site investigation agent        |
| `infra/`        | Terraform        | Cloud infrastructure                 |
| `docs/`         | Markdown         | Project documentation                |
| `planning/`     | Markdown         | Roadmap, PRDs, task tracking         |
| `mobile/`       | Swift/Kotlin     | Mobile app + design tokens           |
| **`copilot/`** | Markdown         | **This repo — Copilot intelligence** |

### First-Time Setup

1. Open the `i4g.code-workspace` file in VS Code
2. Ensure GitHub Copilot extension is installed and signed in
3. Verify `copilot/` appears as a folder in the Explorer sidebar
4. That's it — all routines and standards are loaded automatically

## Daily Workflow

Your day follows a routine. Copilot has prompts for each phase:

```
Start Session → Plan Work → Implement → Review → Deploy → Verify
     ↓              ↓          ↓          ↓         ↓        ↓
 rehydrate     plan-work   work-on-task  pre-merge  deploy  manual-
 -session                  or fix-bug    -review    -to-dev  verification
```

### How to Use Routines

1. Open Copilot Chat (Cmd+Shift+I or the chat icon)
2. In the chat input, click the **prompt picker** (attachment icon or type `#`)
3. Select a prompt from the list — they appear with descriptions
4. The prompt gives Copilot step-by-step instructions for that workflow

### Available Routines

| Routine                 | When to use                                      |
| ----------------------- | ------------------------------------------------ |
| **rehydrate-session**   | Start of every working session                   |
| **plan-work**           | Breaking a feature/task into implementable steps |
| **work-on-task**        | Implementing a specific task (code + test + doc) |
| **fix-bug**             | Diagnosing and fixing a bug with regression test |
| **pre-merge-review**    | Before merging — full quality checklist          |
| **deploy-to-dev**       | Pre-deployment checklist for i4g-dev             |
| **manual-verification** | Post-deployment verification                     |
| **sprint-wrapup**       | End of sprint — update plans, assess readiness   |
| **record-lesson**       | Save a lesson for future sessions                |

## What Copilot Knows Automatically

### Coding Standards (auto-loaded by file type)

When you open a `.py` file, Python standards load. When you open a `.ts` file, TypeScript standards load. You don't need to do anything — the standards are injected automatically.

These standards cover:

- Naming conventions
- Formatting rules
- Error handling patterns
- Framework-specific patterns (Pydantic, React hooks, etc.)

### Project Architecture

Copilot knows:

- How the API routes requests (UI → Next.js proxy → FastAPI)
- The store/factory pattern for data access
- How background workers and jobs are structured
- Environment profiles (local vs dev vs prod)
- The settings system and env var conventions

### Lessons from Past Sessions

The team records lessons learned in repo memory. These persist across sessions and help avoid repeating mistakes. When you discover something new, use the `record-lesson` routine to save it.

## Key Conventions

### Python Repos (core/, ssi/)

```bash
# Activate environment
conda activate i4g        # for core
conda activate i4g-ssi    # for ssi

# Run tests
conda run -n i4g pytest tests/unit

# Start API server
conda run -n i4g uvicorn i4g.api.app:app --reload

# Pre-commit hooks (double-pass)
conda run -n i4g pre-commit run --all-files
git add -u
conda run -n i4g pre-commit run --all-files
```

### UI Repo

```bash
cd ui/
pnpm install
pnpm dev          # Start dev server
pnpm format       # ALWAYS run after editing
pnpm lint
pnpm test
pnpm build        # Catches type errors
```

### Settings Access

Never hard-code values. Always use the settings system:

```python
from i4g.settings import get_settings
settings = get_settings()
# Access nested config: settings.llm.provider, settings.storage.db_url, etc.
```

Override via environment variables: `I4G_LLM__PROVIDER=mock` (double underscore for nesting).

## Getting Help

- **Copilot Chat:** Ask anything about the codebase — it has full context
- **Architecture:** Read `copilot/.github/shared/architecture-cheatsheet.instructions.md`
- **Coding standards:** Read `copilot/.github/shared/general-coding.instructions.md`
- **This guide:** You're reading it
- **Routine catalog:** See [routine-catalog.md](routine-catalog.md) for detailed descriptions

## Tips for Working with Copilot

1. **Use routines, not raw prompts.** The routines encode best practices and ensure nothing is missed.
2. **Record lessons.** When you learn something the hard way, save it. Future sessions (yours and others') benefit.
3. **Let Copilot read before writing.** Copilot works best when it reads existing code before modifying it. The routines enforce this.
4. **Trust the standards.** When Copilot suggests type hints, docstrings, or specific error handling — it's following team conventions.
5. **Ask for the routine.** If you're not sure which routine to use, just describe what you're trying to do. Copilot will suggest the right one.

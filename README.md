# i4g Workflow

Copilot intelligence, developer routines, and coding standards for the I4G Platform.

## What This Repo Does

This repo is the **single source of truth** for how developers (and Copilot) work in the i4g workspace. It contains:

- **Routines** — Prompt templates for daily developer workflows (planning, coding, reviewing, deploying)
- **Standards** — Coding conventions that auto-load by file type
- **Shared instructions** — Platform-wide context that Copilot uses in every conversation
- **Onboarding** — Everything a new developer needs to work effectively with Copilot

Add this repo to your `i4g.code-workspace` and all prompts + instructions become available automatically.

## Quick Start

1. **Add to workspace.** Add `workflow/` as a folder in `i4g.code-workspace`
2. **Use routines.** In Copilot chat, click the prompt picker and select any routine
3. **Follow standards.** Standards auto-load when you edit matching file types — no action needed

## Repo Structure

```
workflow/
├── README.md                    # This file
├── .github/
│   ├── copilot-instructions.md  # Shared platform context (loaded in every conversation)
│   ├── prompts/                 # Routines — invokable from Copilot chat
│   │   ├── plan-work.prompt.md
│   │   ├── work-on-task.prompt.md
│   │   ├── fix-bug.prompt.md
│   │   ├── pre-merge-review.prompt.md
│   │   ├── deploy-to-dev.prompt.md
│   │   ├── manual-verification.prompt.md
│   │   ├── rehydrate-session.prompt.md
│   │   ├── sprint-wrapup.prompt.md
│   │   └── record-lesson.prompt.md
│   ├── standards/               # Auto-loading coding standards
│   │   ├── python.instructions.md
│   │   ├── typescript-react.instructions.md
│   │   ├── terraform.instructions.md
│   │   └── settings-config.instructions.md
│   ├── shared/                  # Shared reference docs for Copilot
│   │   ├── general-coding.instructions.md
│   │   ├── architecture-cheatsheet.instructions.md
│   │   └── pre-merge-checklist.instructions.md
│   └── repo-templates/          # Templates for new repos joining the workspace
│       └── copilot-instructions.template.md
└── docs/
    ├── onboarding.md            # New developer guide
    ├── customization-guide.md   # How to extend this system
    └── routine-catalog.md       # Full catalog of available routines
```

## How It Works

### For New Developers

1. Read [docs/onboarding.md](docs/onboarding.md) — covers workspace setup, daily workflow, and how Copilot helps
2. Use the routines — they guide you through each workflow step by step
3. Ask Copilot — it has all the project context loaded automatically

### For Experienced Developers

1. Use routines to stay consistent and avoid missing steps
2. Record lessons with the `record-lesson` prompt — they persist across sessions
3. Extend the system (see [docs/customization-guide.md](docs/customization-guide.md))

### What Stays in Product Repos

Each product repo (`core/`, `infra/`, `mobile`, `ssi/`, `ui/`) keeps a minimal `.github/copilot-instructions.md` with only repo-specific context:

- Conda environment name and activation
- Build/test commands specific to that repo
- Architecture notes unique to that codebase
- Repo-specific pre-commit procedure (if different from standard)

All shared standards, routines, and platform context live here.

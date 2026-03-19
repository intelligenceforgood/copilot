# Copilot Instructions — i4g Workflow Repo

This repo contains the shared Copilot intelligence for the I4G Platform workspace. It is part of the `i4g` multi-root workspace.

## Purpose

This repo provides:
- **Routines** (`.github/prompts/`) — step-by-step workflow templates for daily development
- **Standards** (`.github/standards/`) — coding conventions auto-loaded by file type
- **Shared instructions** (`.github/shared/`) — platform-wide reference documents
- **Onboarding** (`docs/`) — guides for new developers

## How This Integrates

In a multi-root VS Code workspace, Copilot loads `.github/copilot-instructions.md` from every workspace root. Prompt templates and scoped `.instructions.md` files from any root are available workspace-wide.

Each product repo (`core/`, `ui/`, `ssi/`, `infra/`) has its own minimal `copilot-instructions.md` with repo-specific context (conda env, build commands, architecture). All shared intelligence lives here.

## When Editing This Repo

- **Prompts** should focus on decision-making routines, not single commands
- **Standards** should be concise bullet points — auto-loaded means every token counts
- **Shared instructions** are reference docs that other instructions cross-reference
- **Keep it DRY** — if the same guidance appears in 2+ product repos, move it here
- After adding or changing a routine, update `docs/routine-catalog.md`

# Copilot Instructions for i4g/<REPO_NAME>

<!-- TEMPLATE: Copy this file to <repo>/.github/copilot-instructions.md -->
<!-- Replace all <PLACEHOLDER> values with repo-specific details -->
<!-- Delete these comments after filling in -->

**Unified Workspace Context:** This repository is part of the `i4g` multi-root workspace. Shared coding standards, routines, and platform context live in the `workflow/` repo. These instructions contain only repo-specific context.

## Environment

- **Conda env:** `<CONDA_ENV_NAME>`
- **Language:** <Python | TypeScript | Terraform | etc.>
- **All commands prefix:** `conda run -n <CONDA_ENV_NAME> ...`

## Build & Test

```bash
# Install editable
conda run -n <CONDA_ENV_NAME> pip install -e ".[dev,test]"

# Run tests
conda run -n <CONDA_ENV_NAME> pytest tests/unit

# Start dev server (if applicable)
conda run -n <CONDA_ENV_NAME> uvicorn <module>.api.app:app --reload --port <PORT>
```

## Architecture (repo-specific)

- Entry point: `src/<module>/`
- API: `src/<module>/api/app.py` (if applicable)
- Config: `<module>.settings.get_settings()` with `<PREFIX>_*` env vars

## Pre-Commit

```bash
cd <repo-root>
conda run -n <CONDA_ENV_NAME> pre-commit run --all-files   # Pass 1
git add -u
conda run -n <CONDA_ENV_NAME> pre-commit run --all-files   # Pass 2 — must be clean
```

## Repo-Specific Notes

<!-- Add anything unique to this repo that doesn't belong in shared standards -->

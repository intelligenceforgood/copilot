---
agent: agent
description: "Full review + merge — runs pre-merge-review then commits and pushes all changed repos to main"
---

# Merge

Run the full pre-merge review, then commit and push all changed repos to `main`.
This is the single combined routine — no need to run `/pre-merge-review` separately.

## Phase 1 — Pre-Merge Review

Execute every step of the pre-merge review routine (`copilot/.github/prompts/pre-merge-review.prompt.md`):
code audit, quality gates, tests, docs/config check. Fix all issues found before proceeding.

Do NOT proceed to Phase 2 if any quality gate fails or tests have failures.

## Phase 2 — Clean Working Tree

After the review passes, verify the working tree is merge-ready:

1. **No stray files.** `git status` in each changed repo shows only intentional changes.
   - No merge conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
   - No temp files (`*.tmp`, `*.bak`, `*.swp`, `.DS_Store`, `__pycache__/`)
   - No debug scripts outside `tests/` (`debug_*.py`, `scratch_*.py`)
   - If any found, alert the user and do NOT proceed

2. **No secrets.** Scan staged content for:
   - API keys / tokens (`sk-`, `ghp_`, `AIza`, `AKIA`)
   - Private keys (`BEGIN RSA PRIVATE KEY`, etc.)
   - `.env` files that should be gitignored
   - Hard-coded local paths (`/Users/<name>/`)
   - If any found, alert the user and do NOT proceed

## Phase 3 — Commit and Push

Follow `copilot/.github/standards/merge-commit-discipline.instructions.md` for all commit hygiene rules.

For each changed repo:

```bash
cd <repo-root>
git add -A
git commit -m "<conventional commit message>"
git push origin main
```

- Use conventional commit format: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`
- Generate descriptive commit messages from the changes — do NOT ask the user to confirm
- If push fails (e.g., diverged), alert the user — do NOT force push

## Phase 4 — Summary

Report per-repo: issues found + fixed, test results, files committed, commit hash, push status.

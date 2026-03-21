---
agent: agent
description: "Pre-commit merge gate — clean working tree, no temp files, no secrets, then commit and push"
---

# Merge

Verify the working tree is merge-ready, then commit and push all changed repos.

## Steps

1. **Identify changed repos.** Run `git status -sb` in each workspace root. Only process repos with uncommitted or staged changes.

2. **Clean working tree check.** For each changed repo, confirm:
   - `git status` shows only intentional changes (no untracked mystery files)
   - No merge conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) in any changed file
   - No `TODO(merge)` or `FIXME(merge)` comments left behind

3. **Temp file / debug artifact check.** Scan all changed/new files for:
   - Temporary files: `*.tmp`, `*.bak`, `*.swp`, `*.orig`, `__pycache__/`, `.DS_Store`
   - Debug scripts: standalone `test_*.py` files outside `tests/`, `debug_*.py`, `scratch_*.py`
   - Jupyter checkpoints: `.ipynb_checkpoints/`
   - Build artifacts: `dist/`, `build/`, `*.egg-info/`
   - If any found, alert the user and do NOT proceed until resolved

4. **Secrets / credentials check.** Scan all staged content (`git diff --cached`) for:
   - API keys, tokens, passwords (patterns: `sk-`, `ghp_`, `gho_`, `AIza`, `AKIA`, `password\s*=`, `secret\s*=`, `token\s*=`)
   - `.env` files, `.env.local` (should be in `.gitignore`)
   - Private keys (`BEGIN RSA PRIVATE KEY`, `BEGIN EC PRIVATE KEY`, `BEGIN OPENSSH PRIVATE KEY`)
   - Hard-coded GCP service account JSON key files
   - Local paths specific to one developer's machine (e.g., `/Users/<name>/`)
   - If any found, alert the user and do NOT proceed until resolved

5. **Verify quality gates passed.** Confirm (by checking recent terminal output or re-running):
   - Pre-commit hooks pass (for Python repos)
   - `terraform fmt -check -recursive` passes (for infra)
   - Tests pass with zero failures
   - If not verified, suggest running `/pre-merge-review` first

6. **Stage and commit.** For each changed repo:
   ```bash
   cd <repo-root>
   git add -A
   git status                    # show what will be committed — ask user to confirm
   git commit -m "<message>"     # use a descriptive conventional commit message
   ```
   - Use conventional commit format: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`
   - Ask user to confirm the commit message before committing

7. **Push.** For each committed repo:
   ```bash
   git push origin main
   ```
   - If push fails (e.g., diverged), alert the user — do NOT force push

8. **Summary.** Report per-repo: files committed, commit hash, push status.

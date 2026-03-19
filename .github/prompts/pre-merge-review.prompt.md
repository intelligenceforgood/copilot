---
mode: agent
description: "Full quality checklist before merging — code audit, tests, hooks"
---

# Pre-Merge Review

Execute the complete pre-merge review for all repos with changes.

## Steps

1. **Identify changed repos.** Run `git status -sb` in each workspace root to find which repos have uncommitted or staged changes.

2. **Read the full checklist.** Load and follow the shared pre-merge checklist at `workflow/.github/shared/pre-merge-checklist.instructions.md`.

3. **Code audit.** Review all changed files against coding standards:
   - Type hints on every function (parameters + return)
   - Google-style docstrings on public/private methods
   - No unused imports, dead code, bare excepts, hard-coded secrets
   - Correct use of stores/factories/settings
   - No circular imports

4. **Run quality gates** for each changed repo:

   **Python repos (core/, ssi/):**
   ```
   cd <repo-root>
   conda run -n <env> pre-commit run --all-files   # Pass 1 — auto-fixes
   git add -u                                      # Stage fixes
   conda run -n <env> pre-commit run --all-files   # Pass 2 — must be clean
   ```
   Conda envs: `i4g` for core, `i4g-ssi` for ssi.

   **UI repo:**
   ```
   cd ui/ && make check && make build
   ```

   **Infra repo:**
   ```
   cd infra/ && terraform fmt -check -recursive
   ```

5. **Run tests.** `conda run -n i4g pytest tests/unit` in core (and ssi if changed).

6. **Check docs/config.** If behavior or env vars changed, verify:
   - `docs/config/settings_manifest.yaml` is updated
   - `planning/change_log.md` reflects the changes

7. **Summary.** Report: issues found, fixes applied, test results, hook output, remaining items.

8. **Record lessons.** If the review revealed new patterns, add to `/memories/repo/lessons-learned.md`.

---
agent: agent
description: "Implement a specific task — code, test, document"
---

# Work on Task

Execute a single implementation task with proper testing and documentation.

## Steps

1. **Understand the task.** Read the relevant code to understand what exists. Don't modify code you haven't read.

2. **Implement.** Write the code following the standards that auto-load for the file type. Key principles:
   - Settings access via `get_settings()`, not hard-coded values
   - Stores via factories in `src/i4g/services/factories.py`
   - Type hints on every function, Google-style docstrings on public methods
   - Specific exception handling (no bare `except:`)

3. **Test.** Write or update tests for the changed logic:
   - Unit tests under `tests/unit/`
   - Run: `conda run -n i4g pytest tests/unit -x` (stop on first failure)
   - If adding env vars, add coverage under `tests/unit/settings/`

4. **Document.** If behavior changed:
   - Update `docs/` if user-facing
   - Update config manifests if env vars changed
   - Note in `planning/change_log.md`

5. **Validate locally.** Run pre-commit hooks to catch formatting issues early:

   ```
   conda run -n i4g pre-commit run --files <changed-files>
   ```

6. **Summarize.** Report what was done, tests that pass, and any follow-up items.

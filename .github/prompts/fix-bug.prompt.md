---
agent: agent
description: "Diagnose and fix a bug — reproduce, root-cause, fix, regression test"
---

# Fix Bug

Systematic bug investigation and fix workflow.

## Steps

1. **Reproduce.** Understand the symptoms. If possible, reproduce the bug:
   - Read error messages, logs, or stack traces
   - Identify the failing code path
   - Check if there's a test that should have caught this

2. **Root-cause analysis.** Trace the issue:
   - Read the relevant source files (don't guess — read the code)
   - Check recent changes: `git log --oneline -10 -- <suspect-files>`
   - Look for common patterns: missing null checks, wrong env var, race condition, schema mismatch

3. **Fix.** Apply the minimal correct fix:
   - Fix the root cause, not the symptom
   - Follow coding standards (type hints, specific exceptions, etc.)
   - Don't refactor surrounding code unless directly related to the bug

4. **Regression test.** Write a test that:
   - Would have caught this bug before the fix
   - Verifies the fix works
   - Run: `conda run -n i4g pytest tests/unit -x`

5. **Verify no collateral damage.** Run the full test suite to ensure the fix doesn't break anything else.

6. **Record lesson.** If the bug reveals a pattern or pitfall, save it:
   - Add to `/memories/repo/lessons-learned.md`
   - If it's the 3rd+ occurrence of a pattern, consider promoting to a `.instructions.md`

7. **Summarize.** Report: root cause, fix applied, test added, and any related risks.

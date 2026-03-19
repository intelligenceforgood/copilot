---
mode: agent
description: "End of sprint — update plans, log changes, assess merge readiness"
---

# Sprint Wrap-Up

Close out a sprint or work session with proper documentation and handoff.

## Steps

1. **Identify completed work.** Check `git log` across repos since the sprint started. Summarize what was implemented, fixed, or changed.

2. **Update task checkboxes.** Find the active implementation plan in `planning/tasks/` and mark completed tasks as `- [x]`.

3. **Update change log.** Append a dated summary to `planning/change_log.md`:
   - What changed (features, fixes, refactors)
   - Which repos were affected
   - Any env var or config changes

4. **Manual steps for the user.** List any steps that require manual execution:
   - Alembic migrations: `i4g db migrate dev` / `i4g db migrate prod`
   - Docker image builds and which images
   - Cloud Run deploys
   - Include exact commands.

5. **Risk assessment.** Identify risks of breaking existing functionality:
   - API contract changes
   - Database schema changes
   - New env vars that need setting in production
   - Quick validation tests the user can run

6. **Merge readiness.** State whether this is a good merge point. List blockers or caveats.

7. **Record lessons.** If anything was learned (patterns, pitfalls, workflow improvements), add to `/memories/repo/lessons-learned.md`.

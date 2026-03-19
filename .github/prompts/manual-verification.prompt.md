---
agent: agent
description: "Post-deployment verification — check endpoints, test key flows"
---

# Manual Verification

Verify a deployment is working correctly after pushing to an environment.

## Steps

1. **Determine environment.** Ask the user which environment to verify (dev or prod). Default to dev.

2. **Health checks.** Verify core services are running:
   - Core API: hit the health/readiness endpoint
   - Check Cloud Run service logs for startup errors or crash loops

3. **API smoke tests.** Test key API endpoints:
   - `GET /reviews/search` — search should return results
   - `GET /reviews/{id}` — single review lookup
   - `GET /tasks/{task_id}` — task status (if background jobs are relevant)

4. **UI verification.** If UI was deployed:
   - Load the analyst console
   - Verify search works end-to-end (UI → API proxy → FastAPI → ReviewStore)
   - Check that recent data is visible

5. **Worker/job verification.** If worker jobs were deployed:
   - Trigger a test job (or check recent job execution logs)
   - Verify job completion and output artifacts

6. **Compare with expectations.** Based on what was changed, verify:
   - New features are accessible
   - Fixed bugs are resolved
   - No regressions in adjacent functionality

7. **Report.** Summarize: what was verified, pass/fail for each check, any issues found.

8. **Sign off or escalate.** If everything passes, confirm the deployment is good. If issues are found, recommend next steps (rollback, hotfix, or investigation).

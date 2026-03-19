---
mode: agent
description: "Pre-deployment checklist — smoke test, build, deploy to i4g-dev"
---

# Deploy to Dev

Pre-flight checklist before deploying to the `i4g-dev` environment.

## Steps

1. **Pre-merge review first.** Ensure the pre-merge review routine has been completed. If not, run it first.

2. **Local smoke test.** Verify the code works locally before any cloud operation:
   ```
   conda run -n i4g I4G_PROJECT_ROOT=$PWD I4G_ENV=dev I4G_LLM__PROVIDER=mock i4g jobs ingest --help
   ```

3. **Identify images to build.** Based on changed files, determine which Docker images need rebuilding:
   - Core API changes → `core-svc`
   - Worker/job changes → `dossier-job`, `ingest-job`, `intake-job`, `report-job`
   - UI changes → `cd ui/ && scripts/build_image.sh i4g-console dev`
   - SSI changes → check ssi docker configs

4. **Database migrations.** If Alembic migrations were added:
   ```
   i4g db migrate dev
   ```
   ⚠️ Run on dev first. Never migrate prod without a successful dev migration.

5. **Build and push images.** From the appropriate repo root:
   ```
   scripts/build_image.sh <image-name> dev
   ```

6. **Post-deploy verification.** After Cloud Run picks up the new image:
   - Check Cloud Run logs for startup errors
   - Hit the health endpoint
   - Run the manual verification routine (see `manual-verification` prompt)

7. **Update change log.** Record the deployment in `planning/change_log.md` with date and what was deployed.

8. **Dev/prod parity.** If this deployment adds new Cloud Run jobs or env vars, check that `infra/environments/app/prod/terraform.tfvars` is updated to match (see `/memories/repo/infra-parity.md`).

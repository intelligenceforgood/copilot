---
agent: agent
description: "Fetch and diagnose Cloud Run job/service logs from GCP"
---

# Check Log

Fetch logs from GCP using a user-provided log filter, then diagnose the issue.

## Input

The user will paste a GCP logging filter query. Example:

```
resource.type = "cloud_run_job" resource.labels.job_name = "ingest-bootstrap" labels."run.googleapis.com/execution_name" = "ingest-bootstrap-cn462" resource.labels.location = "us-central1" timestamp >= "2026-04-05T01:49:30.683763Z" labels."run.googleapis.com/task_index" = "0" severity>=DEFAULT
```

## Steps

1. **Detect the GCP project** from the filter (job name prefix conventions):
   - `ingest-bootstrap`, `generate-reports`, `process-intakes`, `core-svc` → `i4g-dev`
   - ML-related resources → `i4g-ml`
   - If ambiguous, ask.

2. **Fetch logs** using `gcloud logging read`:
   ```bash
   gcloud logging read '<PASTE_FILTER>' --project=<PROJECT> --limit=200 --format="value(textPayload)" | head -500
   ```
   - If `textPayload` is empty, retry with `--format=json` and extract `jsonPayload` or `httpRequest`.
   - For severity filtering, start with the filter as-is. If output is too noisy, tighten to `severity>=WARNING`.

3. **Diagnose** the output:
   - Identify the root error (usually the first exception or the last "Caused by").
   - Trace the call stack to the source file and line.
   - Read the relevant source code to understand the failure.

4. **Report** findings:
   - State the root cause clearly (one sentence).
   - Show the relevant log snippet.
   - Propose a fix with file paths and specific code changes.
   - If the fix is straightforward, implement it.

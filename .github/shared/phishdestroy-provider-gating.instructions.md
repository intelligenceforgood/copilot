---
applyTo: "**/osint/**,**/ingestion/phishdestroy/**,**/worker/jobs/merklemap_*.py,**/worker/jobs/phishdestroy_*.py,**/review.py,**/discoveries/**,**/actors/**"
---

# PhishDestroy Provider Gating (quota-gated skip contract)

The PhishDestroy integration depends on several paid third-party APIs whose budgets are
**not guaranteed at implementation time**. This file defines the single pattern every
ingestion job, OSINT module, API route, and UI surface must follow so that:

1. Code builds and tests pass regardless of key availability.
2. Disabled providers **never** surface as silent failures — they surface as explicit
   "skipped because quota-gated" signals at every layer (logs, audit log, review UI).
3. Flipping a provider from disabled → enabled is a single config change plus a Secret
   Manager entry — no code changes, no redeploy of unrelated services.

## 1. Gated providers (as of Sprint 0)

| Provider | Status | Keyed off | Funded? | Consumes |
| --- | --- | --- | --- | --- |
| `merklemap` | funded (dev key available) | `providers.merklemap.enabled` + `api_key` | yes | Sprint 1 merklemap tail |
| `whoxy` | **budget TBD** | `providers.whoxy.enabled` + `api_key` | pending | Sprint 3 `whoxy_reverse.py` |
| `ghunt` | ops-readiness TBD | `providers.ghunt.enabled` + cookie blob | pending | Sprint 3 `ghunt.py` |
| `virustotal` | funded | `providers.virustotal.enabled` + `api_key` | yes | existing SSI |
| `urlscan` | funded | `providers.urlscan.enabled` + `api_key` | yes | existing SSI |
| `securitytrails` | budget TBD | `enrichment.securitytrails_api_key` | pending | existing enrichment |

When a provider's status is not "funded", the PRD acceptance metric(s) that depend on it
(§10) are considered **deferred**, not failing. See §6.

## 2. Settings shape

Provider config lives under a `[providers.<name>]` section in the repo's settings TOML.
Both `core/` and `ssi/` expose identical schema; env-var aliases are `I4G_PROVIDERS__*`
and `SSI_PROVIDERS__*` respectively.

```toml
[providers.merklemap]
enabled = false              # flip to true after key is set
api_key = ""                 # injected via Secret Manager in dev/prod; paste locally in settings.local.toml
base_url = "https://api.merklemap.com"
rate_limit_per_sec = 0       # 0 = no client-side limit; provider limit governs

[providers.whoxy]
enabled = false
api_key = ""
monthly_query_cap = 0        # 0 = no cap; set to budget guardrail once funded

[providers.ghunt]
enabled = false
cookie_blob_path = ""        # path to JSON cookie dump; injected via Secret Manager in cloud
```

Adding a new gated provider requires an entry here **and** a row in §1.

## 3. Runtime contract: `ProviderGate`

Every gated module must gate at the public entry point, not inside helpers. The shape:

```python
from i4g.providers import ProviderGate, SkippedResult  # naming is illustrative

GATE = ProviderGate("whoxy")

async def scan(target: str) -> ScanResult | SkippedResult:
    if not GATE.enabled:
        return GATE.skip(reason="quota_gated", detail="whoxy disabled in settings")
    ...
```

Rules:

- `ProviderGate(name).enabled` returns `True` only if BOTH `providers.<name>.enabled` is
  true AND the required credential (api_key / cookie blob) is non-empty.
- `SkippedResult` is a structured value, never an exception. It carries
  `reason ∈ {"quota_gated", "auth_expired", "rate_limited", "disabled"}` plus a free-form
  `detail` string.
- Orchestrators and ingestion jobs must treat `SkippedResult` as a first-class outcome:
  count it in metrics (`skipped_total{provider=..., reason=...}`) and write one
  `audit_log.log_action(action="provider_skipped", ...)` per skip.
- Never return a default/empty payload when gated. Callers must be able to distinguish
  "provider ran and found nothing" from "provider did not run".

## 4. Surfacing at boundaries

- **Review UI / Discoveries UI:** show a "provider skipped" badge on any scan result that
  includes `skipped[]`. Click reveals provider name + reason + last-enabled date.
- **API responses:** the outer `ScanResult` / `ReviewRecord` schema has a `skipped` array
  of `{provider, reason, detail}`. Clients already ignore unknown fields; adding the
  array is non-breaking.
- **Dossier rendering:** insert a "Coverage gaps" section listing skipped providers with
  reason. Dossier signing includes the skipped list so gaps are auditable after the fact.
- **Metrics (Sprint 4 SLOs):** `provider_skipped_total{provider,reason}` becomes a
  dashboard panel. Persistent non-zero `quota_gated` is informational; persistent
  `auth_expired` is alertable.

## 5. Terraform / deploy gating

Infra changes for a gated provider must ship **as disabled**:

- `enable_whoxy` etc. default to `false` in tfvars.
- The Cloud Run job / Scheduler is created (so enabling is a var-flip) but suspended or
  set to `min_instances = 0` when disabled.
- Secret Manager entries are created with an empty payload; funding the key is a pure
  secret-version bump, no Terraform apply.

This is also why the Sprint 1 merklemap tail ships **enabled** (we have a dev key) while
the Sprint 3 whoxy/ghunt modules ship **disabled** with the code paths in place.

## 6. PRD acceptance-metric deferral

When a gated provider is unfunded, affected PRD §10 acceptance metrics move to a
"deferred" column in the sprint verification report. They are NOT marked failing and do
NOT block merge. The report must list:

- Which metric is deferred.
- Which provider gates it.
- What budget or ops-readiness decision unblocks it.

This keeps the merge/verify loop unblocked on things the engineering team does not
control (budget, counsel, upstream availability).

## 7. Rotation hooks

Rotation is a three-step sequence, identical for every provider:

1. Revoke or re-issue the key at the provider's console.
2. Paste new value into `<repo>/config/settings.local.toml` under
   `[providers.<name>] api_key = "..."` **or** push a new Secret Manager version (dev
   and prod use Secret Manager; local dev uses the TOML override).
3. Restart the consuming service (`uvicorn` reload locally; Cloud Run rolls on next
   revision).

Runbook bodies for each provider live in `copilot/docs/runbooks/` (Sprint 4 deliverable).

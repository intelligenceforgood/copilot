---
applyTo: "**/ingestion/phishdestroy/**,**/worker/jobs/phishdestroy_*.py,**/worker/jobs/merklemap_*.py,**/osint/blocklist_aggregator.py,**/osint/merklemap_client.py,**/osint/ctlog_lookup.py,**/alembic/versions/phishdestroy_*.py"
---

# PhishDestroy Provenance Contract (frozen Sprint 0)

Every row written by any PhishDestroy ingestion path — `destroylist`, `ScamIntelLogs`
archive, `DestroyScammers` actor data, blocklist aggregator, merklemap tail — **MUST**
carry the provenance block defined here. No exceptions.

This file is authoritative. If the PRD or a task manifest conflicts with it, this file
wins; update the PRD, not the data.

## 1. `source_provenance` JSON shape

Every `source_provenance` JSON column stores an object with exactly these fields:

```json
{
  "source": "phishdestroy.destroylist",
  "team": "TrustWalletPanel",
  "commit_sha": "83d0307420fcc865fcb8a34b8c454acbc6d56f1f",
  "record_id": "domains.txt#L412",
  "ingested_at": "2026-04-23T14:17:03Z",
  "ingest_job": "i4g-jobs-ingest-phishdestroy-archive",
  "ingest_job_run_id": "cloudrun-exec-abc123"
}
```

### Field rules

| Field | Required | Type | Rule |
| --- | --- | --- | --- |
| `source` | yes | string | Dotted identifier, one of the values in §3. Never free text. |
| `team` | conditional | string | Required for archive / actor data; omit for feeds not scoped to a team (e.g. merklemap, blocklist aggregator). |
| `commit_sha` | yes | string | Full 40-char hex SHA of the upstream repo HEAD at ingest time. Pinned SHAs in §4. For live feeds (merklemap), use the `commit_sha` of the deployed **filter config**, not the feed. |
| `record_id` | yes | string | Stable pointer to the exact source record. See §2. |
| `ingested_at` | yes | string | RFC 3339 / ISO 8601 UTC with `Z` suffix. |
| `ingest_job` | yes | string | Cloud Run job resource name, or CLI command name for local runs. |
| `ingest_job_run_id` | no | string | Cloud Run execution ID when available. |

No extra keys. If you need more context, promote it to a first-class column.

## 2. `record_id` rules

`record_id` must be **deterministic and reproducible** from the upstream source. If two
runs against the same `commit_sha` produce different `record_id` values for the same
logical row, the rule is wrong and must be fixed before merge.

| Source | `record_id` format |
| --- | --- |
| `phishdestroy.destroylist` | `sha256(normalized_indicator)` (hex) when sourced from `DestroyScammers/data/data.json`; `<path>#L<line>` when sourced from the standalone repo (not currently checked out). |
| `phishdestroy.archive.iocs` | `<team>/iocs.json#<jsonpointer>` (RFC 6901) |
| `phishdestroy.archive.chat` | `<team>/chat/<filename>#<message_index>` |
| `phishdestroy.archive.damage` | `<team>/successful_thefts/<filename>#<jsonpointer>` |
| `phishdestroy.actors` | `data.json#/<actor_key>` |
| `phishdestroy.registrants` | `registrants.json#/<pivot_type>/<pivot_value>` |
| `blocklist.<source>` | `sha256(normalized_indicator \| source)` (hex) |
| `merklemap.tail` | `sha256(domain \| first_seen_unix)` (hex) |
| `ctlog.crtsh` | `crtsh:<entry_id>` |

## 3. Allowed `source` values

Controlled vocabulary. Reject any value outside this list at write time.

```
phishdestroy.destroylist
phishdestroy.archive.iocs
phishdestroy.archive.chat
phishdestroy.archive.damage
phishdestroy.archive.infrastructure
phishdestroy.archive.brands
phishdestroy.actors
phishdestroy.registrants
blocklist.metamask
blocklist.scamsniffer
blocklist.openphish
blocklist.seal
blocklist.enkrypt
blocklist.phishdestroy
blocklist.polkadot
blocklist.cryptofirewall
merklemap.tail
ctlog.crtsh
```

Adding a new `source` requires editing this file and the provenance vocabulary test
in `core/tests/unit/ingestion/test_provenance.py`.

## 4. Pinned upstream commit SHAs (Sprints 1–2 baseline)

These SHAs are the baseline for Sprint 1 / Sprint 2 ingestion. When re-ingesting from a
newer upstream, bump the pin here, update `change_log.md`, and re-run the ingest job —
idempotency (§5) will update rows in place.

| Upstream repo | Local checkout | Pinned SHA | Pinned date |
| --- | --- | --- | --- |
| `github.com/phishdestroy/ScamIntelLogs` | `phishdestroy/ScamIntelLogs` | `83d0307420fcc865fcb8a34b8c454acbc6d56f1f` | 2026-03-01 |
| `github.com/phishdestroy/DestroyScammers` | `phishdestroy/DestroyScammers` | `c40cbbf527dd9e5e232090346e1a8ceab32d1683` | 2025-11-30 |
| `github.com/phishdestroy/destroylist` | `phishdestroy/DestroyScammers/data/data.json` (`registrants.json`) | see DestroyScammers SHA | — |
| `github.com/phishdestroy/merklemap-cli` | `phishdestroy/merklemap-cli` | `550cb04aa633c000724c339ada085c59444d5b78` | 2024-10-06 |

> The standalone `destroylist` repo is not checked out locally; the same domain list is
> reachable through `DestroyScammers/data/data.json` and is covered by that SHA. If the
> Sprint 1 `destroylist` ingestion job pulls from the standalone repo instead, pin its
> SHA here in the same update.

## 5. Idempotency key

All ingestion jobs MUST be idempotent under this key:

```
(source, team, commit_sha, record_id)
```

- `team` contributes to the key only when the `source` entry in §3 includes a team scope.
- The composite key is the natural UNIQUE index for staging tables; production tables
  index it as a non-unique composite because the same logical row can be emitted by
  multiple ingest runs (newer `commit_sha`) and upsert on match.
- Re-running a job with the **same** `commit_sha` must be a no-op (unchanged rows).
- Re-running with a **newer** `commit_sha` updates in place and appends the new SHA to a
  `provenance_history` JSON array on the row (schema detail handled in Sprint 1
  migration).

## 6. Sensitive-column marker

Columns containing PII per PRD §11 Q3 (real names, chat transcripts, leak passwords,
raw contact addresses) MUST be flagged with `info={"sensitive": True}` at the SQLAlchemy
column definition. Example:

```python
real_name = Column(String, info={"sensitive": True})
```

Downstream rules that key off this marker (automatically enforced by tests):

- Dossier renderer redacts unless reader has `role=senior_analyst`.
- Every read of a `sensitive` column emits an `audit_log.log_action` with actor identity
  + `reason_code` (required — API rejects requests without one).
- BigQuery export policy: sensitive columns land in a separate, access-restricted view.

A unit test (`tests/unit/schema/test_sensitive_markers.py`) asserts the expected set of
sensitive columns; changing the set requires updating the test in the same commit.

## 7. What this contract does NOT cover

- Storage layout of evidence blobs (chat exports, screenshots) — see Sprint 2.4 and
  `core/docs/design/storage.md`.
- Access-control mechanics (RBAC, reason-code enforcement) — see Sprint 3.5 and the
  API instructions.
- Rate-limit / quota budgeting for paid providers — see
  `phishdestroy-provider-gating.instructions.md`.

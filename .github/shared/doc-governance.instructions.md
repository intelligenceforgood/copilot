---
applyTo: "**"
---

# Documentation Governance

Every engineer on the I4G platform is a documentation owner. These rules apply to all repos in the i4g workspace.

## Definition of Done for Documentation

A PR is **not ready to merge** until documentation is current. The reviewer is responsible for enforcing this — not CI.

**For any PR that adds, changes, or removes component behavior:**

- [ ] **Tier 2 TDD updated** — the affected subsystem's design document reflects the change. If the subsystem has no TDD, create a stub using [`tdd-template.md`](tdd-template.md).
- [ ] **Config manifest updated** — if any `I4G_*` or `SSI_*` env vars were added or removed, update `core/docs/config/settings_manifest.yaml` and the relevant `settings.default.toml`.
- [ ] **Integration contracts updated** — if the change affects a cross-service boundary (UI↔Core, Core↔SSI, SSI→Core, Scheduler→Job), update [`planning/architecture/integration_contracts.md`](../../../planning/architecture/integration_contracts.md).
- [ ] **End-user doc updated** — if the change affects a workflow visible in the analyst console or API reference, update the relevant page in `docs/book/`.
- [ ] **Change log entry added** — add a bullet point to `planning/change_log.md` under today's date section.

**For a PR that only touches infrastructure (Terraform):**

- [ ] If a new Cloud Run service or job is added: update `planning/architecture/system_narrative.md` Section 2 (Component Inventory).
- [ ] If a new Cloud Scheduler job is added: update `infra/docs/scheduler_inventory.md`.
- [ ] Add a change log entry.

## Ownership Model

Documentation ownership follows code ownership. The engineer who writes the code owns the doc update for that changeset. The tech lead for the affected component is the documentation reviewer.

| Tier | Reviewer |
|---|---|
| 0 — System narrative | Chief Architect reviews any substantive change |
| 1 — System architecture | CTO or Chief Architect |
| 2 — Component TDD | Repo tech lead |
| 3 — Operational guides | Repo tech lead or SRE lead |
| 4 — End-user docs | CPO (editorial), tech lead (accuracy) |
| 5 — Copilot intelligence | CTO |

## Staleness Detection

Every Tier 1 and Tier 2 document must include a `Last Verified` date in its front matter or header:

```markdown
**Last Verified:** [Month YYYY]
```

**A document is stale if its `Last Verified` date is more than 90 days ago.**

Use the staleness banner for sections you cannot confidently verify without reading the code:

```markdown
> ⚠️ **Verification needed** — This section was last checked [date].
> If you find it out of date, update it and remove this banner.
```

Do not remove the banner without actually verifying the content against the running code.

## Quarterly Documentation Review

Once per quarter, each tech lead performs a documentation review for their area:

1. Open `planning/architecture/doc_audit_matrix.md`.
2. For each Tier 1–2 document with a `Last Verified` date older than 90 days: verify it against the current code and update both the document and the audit matrix.
3. For any `GAP` entries in your area: either create the document or log a specific blocker.
4. Update `Last Verified` dates in the audit matrix.
5. Add a change log entry summarizing what was reviewed.

**Target:** Zero Tier 1–2 documents with `Last Verified` older than 90 days by the end of each quarterly review.

## New Component Checklist

When adding a new component (service, job, major subsystem):

1. Create `<component>_tdd.md` using [`tdd-template.md`](tdd-template.md).
2. Add the component to `planning/architecture/system_narrative.md` Section 2 (Component Inventory) and Section 3 (Integration Map).
3. Add cross-service integration contracts to `planning/architecture/integration_contracts.md`.
4. Add a row to `planning/architecture/doc_audit_matrix.md`.
5. Update the master TDD for the owning repo's `tdd.md` (add a row to the Subsystem Index).
6. If it's a Cloud Run service/job: update `infra/docs/service_catalog.md`.
7. If it has a scheduler: update `infra/docs/scheduler_inventory.md`.
8. If it's user-facing: add a page to `docs/book/`.

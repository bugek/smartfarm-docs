# MVP Scope (Phase 1)

The Phase 1 vertical slice is **audit-readiness for one crop cycle under one compliance scheme**.

## In scope

### Org and identity
- organization, membership, and role assignment (farmer, expert, compliance lead, admin)
- single-organization context per session; users may belong to multiple orgs

### Farm structure
- farm site with location
- plots within a site (geometry not required; name + size + crop is enough)

### Crop cycle
- a crop cycle belongs to a plot
- has crop, planned start, planned end, status (`planned`, `active`, `closed`)
- is the parent for all GAP records and evidence in Phase 1
- has a derived readiness state separate from lifecycle status: `ready`, `partial`, `not_ready`

### GAP records
- structured record entry tied to a crop cycle
- record types reflect the chosen compliance scheme's checklist (e.g. land prep, input application, harvest, post-harvest handling)
- each record has: type, occurred-at timestamp, actor, free-text notes, optional structured fields per type
- records are append-only; corrections are new records that reference the prior one
- records can be submitted before all required evidence is attached; missing required evidence keeps the item out of `ready`
- there is no long-lived draft state in Phase 1

### Evidence
- image, video, and document evidence may be attached to a crop cycle or a specific GAP record
- evidence stores: file metadata, capture timestamp, uploader, optional geo-tag
- evidence is never deleted from history; it can be marked superseded
- only record-level evidence counts toward satisfying a checklist item's required proof; cycle-level evidence is contextual only

### Expert review workflow
- experts can list crop cycles assigned to them
- experts can comment on records and on evidence
- experts can mark the current record as `reviewed`, `needs_more_evidence`, or `blocking`
- the UI must also show the implicit `unreviewed` state when a current record has no review yet
- expert reviews are append-only history tied to a specific record version
- `needs_more_evidence` keeps the crop cycle in `partial`; `blocking` forces `not_ready`
- a crop cycle has a derived **readiness state** computed from record presence, required evidence presence, and the latest review on each current record

### Readiness reporting
- per crop cycle: a readiness summary listing required record types, current records, missing items, required evidence, actual evidence, and outstanding expert flags
- report output includes the compliance scheme revision used at export time plus the frozen readiness state for that snapshot
- report output includes open blockers, needs-more-evidence items, and a chronological log that preserves superseded history
- exportable as a single document (PDF or printable HTML in Phase 1)

## Narrowing assumptions for Phase 1

- One current record per checklist item is enough to demonstrate the workflow. Repeated seasonal events may generate multiple records later, but engineering only needs the current-version chain for Phase 1.
- Review resolution is manual. The system does not auto-clear an old `needs_more_evidence` or `blocking` decision without a new expert review on the current version.
- Readiness is conservative: missing required records, missing required record-level evidence, or any `blocking` review state means the crop cycle is not ready.
- `partial` means the crop cycle has enough structure to review, but at least one required item is still waiting on expert sign-off or additional evidence.

## Explicitly not in Phase 1 delivery

- automated recommendations
- in-app messaging beyond record/evidence comments
- commerce, payments, marketplace
- expert ratings or reputation scores
- offline-first sync (assume connectivity; capture local cache only as an optimization)
- multi-scheme comparison (one scheme picked per organization in Phase 1)

## Acceptance signal for Phase 1

A demo organization can:

1. set up org -> site -> plot -> crop cycle
2. a farmer logs at least one current record per required GAP record type and can attach record-level evidence from a phone browser
3. the system distinguishes missing evidence, unreviewed records, and blocking findings without mutating history
4. an expert reviews the current records and can mark them `reviewed`, `needs_more_evidence`, or `blocking`
5. a compliance lead exports a readiness report that shows the frozen readiness state, open issues, and complete traceability: who recorded what, when, and the evidence behind it

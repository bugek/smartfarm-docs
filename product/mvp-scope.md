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
- has crop, planned start, planned end, status (planned, active, closed)
- is the parent for all GAP records and evidence in Phase 1

### GAP records
- structured record entry tied to a crop cycle
- record types reflect the chosen compliance scheme's checklist (e.g. land prep, input application, harvest, post-harvest handling)
- each record has: type, occurred-at timestamp, actor, free-text notes, optional structured fields per type
- records are append-only; corrections are new records that reference the prior one

### Evidence
- image, video, and document evidence may be attached to a crop cycle or a specific GAP record
- evidence stores: file metadata, capture timestamp, uploader, optional geo-tag
- evidence is never deleted from history; it can be marked superseded

### Expert review workflow
- experts can list crop cycles assigned to them
- experts can comment on records and on evidence
- experts can mark a record as "reviewed", "needs more evidence", or "blocking"
- a crop cycle has a derived **readiness state** computed from record review states

### Readiness reporting
- per crop cycle: a readiness summary listing required record types, present records, missing items, and outstanding expert flags
- exportable as a single document (PDF or printable HTML in Phase 1)

## Explicitly not in Phase 1 delivery

- automated recommendations
- in-app messaging beyond record/evidence comments
- commerce, payments, marketplace
- expert ratings or reputation scores
- offline-first sync (assume connectivity; capture local cache only as an optimization)
- multi-scheme comparison (one scheme picked per organization in Phase 1)

## Acceptance signal for Phase 1

A demo organization can:

1. set up org → site → plot → crop cycle
2. a farmer logs at least one record per required GAP record type with evidence
3. an expert reviews and either approves all records or flags missing evidence
4. a compliance lead exports the readiness report
5. the report shows complete traceability: who recorded what, when, and the evidence behind it

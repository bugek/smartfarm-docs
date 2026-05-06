# Phase 1 User Flows

Three flows must work end-to-end. Everything else is optional polish.

## 1. Farmer records work in the field

Trigger: farmer completes a GAP-relevant activity (e.g. fertilizer application, pesticide spray, harvest).

Steps:
1. Farmer opens the active crop cycle.
2. Selects a record type from the GAP checklist for that scheme.
3. Fills the structured fields for that type (e.g. product used, rate, area).
4. Adds a free-text note.
5. Captures or attaches evidence (photo, short video, or document).
6. Submits. Record is saved with `occurred_at`, `created_by`, and a server-side `created_at`.

Notes:
- Evidence capture must work from a phone browser. Native app is not in Phase 1.
- The farmer sees a confirmation that the record is now visible to the assigned expert.
- Records are immutable after submit. Edits are made by adding a follow-up record that references the original.

## 2. Expert reviews readiness

Trigger: expert opens their queue of assigned crop cycles.

Steps:
1. Expert sees a list of crop cycles with a readiness indicator (complete / incomplete / flagged).
2. Opens a crop cycle.
3. Sees the GAP checklist for that scheme with each item showing: required, recorded, last record date, evidence count.
4. Opens an individual record.
5. Reviews the structured fields and evidence.
6. Sets a review state: `reviewed`, `needs_more_evidence`, or `blocking`.
7. Optionally adds a comment that the farmer will see.

Notes:
- The expert never edits farmer data in Phase 1.
- A `blocking` state freezes readiness reporting on that crop cycle until it is resolved by a new record from the farmer.

## 3. Compliance lead produces the readiness report

Trigger: compliance lead needs an audit-ready snapshot of a crop cycle.

Steps:
1. Compliance lead opens the crop cycle.
2. Triggers "Export readiness report".
3. System composes a report containing:
   - org, site, plot, crop cycle metadata
   - the compliance scheme and its checklist
   - for each checklist item: present records, missing records, evidence list
   - all expert review states and comments
   - chronological activity log
4. Output is a single downloadable document (PDF or printable HTML in Phase 1).

Notes:
- The report is a snapshot. Re-exporting after new activity produces a new snapshot.
- Snapshots should be referenceable by URL inside the org for review purposes.

## Cross-cutting expectations

- All three flows share the same data model. The expert and compliance lead views are read-and-annotate over what the farmer captured.
- No flow depends on automated recommendations or AI in Phase 1.
- All flows must produce records that include enough context (org, site, plot, cycle, actor, time, evidence refs) to be usable as audit input.

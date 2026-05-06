# Phase 1 User Flows

Three flows must work end-to-end. Everything else is optional polish.

## Phase 1 assumptions

- A farmer may submit a record before every required evidence item is attached. The record exists immediately, but readiness stays degraded until the required evidence is present.
- There is no long-lived draft workflow in Phase 1. A record is either newly submitted or superseded by a later correction record.
- Review is always against the latest current record for a checklist item. Prior records and prior reviews remain in history.

## 1. Farmer records work in the field

Trigger: farmer completes a GAP-relevant activity (e.g. fertilizer application, pesticide spray, harvest).

Steps:
1. Farmer opens the active crop cycle.
2. Selects a record type from the GAP checklist for that scheme.
3. Fills the structured fields for that type (e.g. product used, rate, area).
4. Adds a free-text note.
5. Captures or attaches evidence (photo, short video, or document) at the record level when the proof is specific to that activity.
6. Optionally adds cycle-level evidence only for general context that is not proof of the specific record.
7. Submits. Record is saved with `occurred_at`, `created_by`, and a server-side `created_at`.

Resulting system behavior:
- The newly submitted record becomes the current record for that activity unless it explicitly supersedes an older one.
- If the scheme requires evidence and the record lacks the required evidence kinds, the record is still saved but the checklist item remains not ready.
- The farmer immediately sees whether the checklist item is missing evidence, waiting for expert review, or blocked by expert feedback.

Notes:
- Evidence capture must work from a phone browser. Native app is not in Phase 1.
- The farmer sees a confirmation that the record is now visible to the assigned expert.
- Records are immutable after submit. Corrections are made by adding a follow-up record that references the original via supersession.
- Phase 1 does not support expert-authored farmer records. If the expert finds a problem, the farmer must submit a replacement or supplemental record.

## 2. Expert reviews readiness

Trigger: expert opens their queue of assigned crop cycles.

Steps:
1. Expert sees a list of crop cycles with a readiness indicator: `ready`, `partial`, or `not_ready`.
2. Opens a crop cycle.
3. Sees the GAP checklist for that scheme with each item showing: required, current record present or missing, latest record date, evidence completeness, and latest review badge.
4. Opens an individual current record.
5. Reviews the structured fields and evidence.
6. Sets a review state: `reviewed`, `needs_more_evidence`, or `blocking`.
7. Optionally adds a comment that the farmer will see.

Review-state semantics:
- `unreviewed`: implicit default when a current record exists and no expert review has been recorded yet.
- `reviewed`: current record is acceptable for the checklist item as captured.
- `needs_more_evidence`: the record may be substantively correct, but the proof is incomplete or unclear. This keeps the cycle in `partial`.
- `blocking`: the current record is not acceptable for audit readiness. This forces the cycle to `not_ready`.

Resolution rules:
- Experts do not edit or overwrite prior reviews. Each review action is append-only history tied to a record version.
- `needs_more_evidence` and `blocking` are resolved only when the farmer submits a new current record or attaches the missing evidence to the current record, then the expert reviews again.
- If a farmer supersedes a record, the prior review stays attached to the older record and no longer controls the checklist item's current state.

Notes:
- The expert never edits farmer data in Phase 1.
- The expert queue should sort `not_ready` first, then `partial`, then `ready`.
- A `blocking` state prevents the crop cycle from reaching `ready` until a newer acceptable record is reviewed.

## 3. Compliance lead produces the readiness report

Trigger: compliance lead needs an audit-ready snapshot of a crop cycle.

Steps:
1. Compliance lead opens the crop cycle.
2. Triggers "Export readiness report".
3. System composes a report containing:
   - org, site, plot, crop cycle metadata
   - the active compliance scheme and ruleset revision
   - the derived readiness state at export time: `ready`, `partial`, or `not_ready`
   - for each required checklist item: current record present or missing, latest record date, required evidence kinds, present evidence list, latest expert review state, latest expert comment
   - a list of open blockers and needs-more-evidence items
   - chronological activity log
   - appendix of superseded records and superseded evidence references
4. Output is a single downloadable document (PDF or printable HTML in Phase 1).

Notes:
- The report is a snapshot. Re-exporting after new activity produces a new snapshot.
- Snapshots should be referenceable by URL inside the org for review purposes.
- The report must never silently flatten history. The current record is primary, but superseded records remain discoverable in the snapshot.

## Cross-cutting expectations

- All three flows share the same data model. The expert and compliance lead views are read-and-annotate over what the farmer captured.
- No flow depends on automated recommendations or AI in Phase 1.
- All flows must produce records that include enough context (org, site, plot, cycle, actor, time, evidence refs) to be usable as audit input.
- The readiness indicator shown in the farmer, expert, and compliance lead views must be computed from the same rules and use the same three states: `ready`, `partial`, `not_ready`.

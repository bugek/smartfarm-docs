# GAP Workflow UX/UI Acceptance Criteria

This document defines the Phase 1 SmartFarm Web UX contract for GAP workflows. It is written for implementation and review of the web UI, not as a visual design system.

## Product intent

The UI must make it easy for a farmer to record real field work, attach usable proof, and understand what is still missing. It must make it easy for an expert to review the same records without changing farmer data. It must make readiness reporting traceable and consistent with the existing GAP data contract.

Assumptions:

- Phase 1 is mobile-first for farmers and responsive for experts and compliance leads.
- Connectivity is assumed, but the UI should be tolerant of slow upload and retry states.
- The UI may use Thai or another configured organization language later, but Phase 1 copy must be plain, action-oriented, and consistent across roles.
- This issue does not change the API or data model contract.

## UX principles

- Start from the user's next job, not from data tables. Farmers see "what do I need to record now"; experts see "what needs review"; compliance leads see "can this cycle support an audit snapshot".
- Keep GAP status visible at every level: organization dashboard, crop cycle, checklist item, record detail, evidence upload, and report export.
- Use the same readiness language everywhere: `ready`, `partial`, `not_ready`; use `unreviewed`, `reviewed`, `needs_more_evidence`, and `blocking` only for record review state.
- Make required proof obvious before submit and after submit. A farmer can submit an incomplete record, but the UI must clearly show that readiness is still degraded.
- Preserve trust in the trail. Avoid UI affordances that imply edits overwrite submitted records or evidence.
- Optimize field entry for phones: large tap targets, short forms, camera-first evidence capture, progressive sections, and no dense desktop-only tables for farmer-critical tasks.
- Optimize expert review for scan speed: queue priority, checklist badges, evidence preview, latest review state, and direct access to the current record.

## Dashboard acceptance criteria

### Farmer dashboard

- Shows assigned active crop cycles with crop, plot, planned date range, assigned expert, and derived readiness state.
- Shows the top missing or blocked GAP items for each active crop cycle without requiring the farmer to open a report.
- Provides a primary action to add a GAP record for each active crop cycle.
- Separates actions the farmer can take from expert-only states; for example, a `blocking` item shows the expert comment and asks for a new or supplemental record, not "edit expert review".
- Does not expose admin or compliance configuration actions to farmer-only users.

### Expert dashboard

- Shows assigned crop cycles sorted by `not_ready`, then `partial`, then `ready`.
- Shows counts for missing records, missing required evidence, unreviewed records, `needs_more_evidence`, and `blocking` items.
- Allows the expert to open the next record needing review in one action.
- Shows whether a `blocking` state is still attached to the current record or belongs only to superseded history.
- Does not provide controls that create or overwrite farmer records.

### Compliance lead dashboard

- Shows all crop cycles in the organization with readiness state, scheme revision, assigned expert, and last activity date.
- Provides a readiness report export action only from a crop cycle context where the scheme and crop cycle are known.
- Warns before export when readiness is `partial` or `not_ready`, but still allows an export because reports are snapshots.
- Shows previous readiness snapshots with generated time, generator, frozen readiness state, and linkable snapshot reference.

## GAP checklist acceptance criteria

- Checklist items come from the active compliance scheme data, not hard-coded UI labels.
- Each item shows required status, current record presence, latest record date, required evidence kinds, present record-level evidence count, latest review badge, and readiness contribution.
- Missing required records are visually distinct from records that exist but are missing evidence.
- `unreviewed` is visible when a current record exists and no review has been recorded yet.
- Superseded records remain discoverable from the item detail but do not control the current readiness badge.
- Checklist filters include all, missing record, missing evidence, unreviewed, needs more evidence, blocking, and ready.
- The same checklist item status appears consistently for farmer, expert, and compliance lead views, with role-specific actions only where authorized.

## Evidence upload acceptance criteria

- Farmer evidence capture is record-first: when launched from a record flow, uploaded files attach to that record by default.
- Cycle-level evidence is available only as a secondary option for general context, with copy that says it does not satisfy required record proof.
- Upload supports image, video, and document files only.
- The UI shows file name or thumbnail, evidence kind, upload progress, upload result, and retry action on failure.
- Required evidence kinds for the selected record type are visible before and after upload.
- The user can remove a file from the pre-submit selection, but after submission the UI must use supersede or replace language rather than delete/overwrite language.
- Expert and compliance views show thumbnails or document previews, uploader, captured time where available, uploaded time, and geo flag when available.
- Large upload, permission-denied camera access, unsupported file type, and network failure each have specific user-facing error states.

## Expert review acceptance criteria

- Expert review is always against the current record version for a checklist item.
- The review screen shows structured fields, farmer note, record-level evidence, relevant cycle context, prior reviews for this record, and superseded history access.
- Expert can set exactly one review state per review action: `reviewed`, `needs_more_evidence`, or `blocking`.
- Expert comments are optional for `reviewed` and required for `needs_more_evidence` and `blocking`.
- Submitting a review appends a new `RecordReview`; it never edits an existing review.
- The UI explains the readiness effect before submit: `needs_more_evidence` keeps the cycle `partial`; `blocking` forces `not_ready`.
- The farmer view surfaces the latest expert comment and the next farmer action after a `needs_more_evidence` or `blocking` review.
- If the farmer supersedes a record, the expert queue shows the new current record as `unreviewed` even if the older record had a review.

## Chemical use record flow acceptance criteria

This flow covers the hazardous substance use slice and must follow the existing scheme field and evidence contract.

- The entry point is a checklist action for the hazardous substance use record type, not a generic free-form note.
- The flow asks for `occurred_at`, plot or crop cycle context, operator, product reference, reason for use, application method, dose, treated area, total product amount, and units.
- Product reference is selected from an existing product record when available; if missing, the UI routes the user to create or request the product record rather than storing only free text.
- The UI shows the selected product label evidence status and warns when the product record lacks required label proof.
- Dose, area, and amount fields use numeric inputs with explicit unit selectors; units must not be embedded only in free text.
- The user can attach field evidence when required by the scheme, and the UI distinguishes product label evidence from field application evidence.
- Validation blocks impossible or contract-breaking values before submit, such as zero or negative dose, missing product reference, or crop cycle mismatch.
- Risk warnings such as expired product, stock shortage, or pre-harvest interval conflict are shown as review risks or blocking findings according to the data contract; the UI must not silently mark the cycle ready when these conditions apply.
- Submit creates an append-only GAP record and, where supported by the backend contract, a related stock deduction event; the UI must not imply mutable inventory editing.

## Information hierarchy

- Page title: name the operational context first, such as crop cycle or checklist item.
- Status band: show readiness and review state near the title, not buried inside tabs.
- Primary action: one main action per screen, matching the role and context.
- Required fields and required evidence: show before optional notes and attachments.
- History: visible but secondary; current record and current readiness drive the first view.
- Metadata: actor, timestamps, scheme revision, plot, and evidence refs must be available for audit traceability, but can sit below the primary task area.

## Empty, loading, and error states

- Empty dashboard: explain what setup is missing and show the next valid action for the role.
- Empty checklist: say no scheme is assigned or no checklist items are available; do not show a blank table.
- Empty evidence: show required evidence kinds and camera/upload action when the user can act.
- Empty expert queue: distinguish "nothing assigned" from "all assigned cycles are ready".
- Loading states: preserve page structure with skeleton rows or disabled action areas; do not flash an incorrect `ready` state while data loads.
- Upload in progress: keep the user on the record and show progress, cancellation before submit, and retry after failure.
- Validation errors: appear next to the field, use plain language, and preserve entered data.
- Authorization errors: tell the user they do not have access in this organization context and remove invalid actions.
- Server errors: keep the user on the same screen, preserve unsent form data where possible, and provide retry.

## Screen language

- Use verbs that match the workflow: record, attach evidence, submit, review, request more evidence, mark blocking, export snapshot.
- Avoid vague status words such as done, good, approved, failed, or complete when a GAP-specific state exists.
- Do not say a record was edited after submit; say a correction or replacement record was submitted.
- Do not say evidence was deleted after submit; say it was superseded or replaced, while history is retained.
- Farmer-facing copy should name the next action: "Add label photo", "Submit replacement record", "Attach field photo".
- Expert-facing copy should name the review effect: "This will keep readiness partial" or "This will force not ready".

## Data contract constraints to preserve

- All UI status is derived from the existing records, evidence, and review state contract; the UI must not invent a separate readiness source of truth.
- Records and evidence are append-only after submit. Corrections use supersession.
- Evidence must remain attached to a crop cycle or a specific record; record-level evidence is the only evidence that satisfies required proof for a checklist item.
- The UI must preserve org, site, plot, crop cycle, actor, `occurred_at`, server `created_at`, evidence metadata, and scheme revision visibility where relevant.
- Compliance scheme content, record types, required fields, and required evidence kinds are loaded from data.
- Expert review states remain `reviewed`, `needs_more_evidence`, and `blocking`; `unreviewed` is an implicit display state, not a stored review action.
- Readiness report export uses a frozen snapshot with current records, missing items, evidence list, latest expert review state, open blockers, and chronological history.

## Review checklist for SmartFarm Web implementation

- Farmer can complete the full mobile record flow, attach record-level evidence, submit, and understand missing readiness work.
- Expert can triage assigned crop cycles, review current records, inspect evidence, and submit append-only review decisions.
- Compliance lead can see organization readiness, export a snapshot, and understand why the state is `ready`, `partial`, or `not_ready`.
- Dashboard, checklist, record detail, evidence upload, expert review, and report export all use the same readiness and review semantics.
- Empty, loading, validation, upload failure, authorization failure, and server failure states are implemented for the critical flows.
- No UI path overwrites farmer records, deletes evidence history, hard-codes scheme checklist items, or creates a second readiness truth.

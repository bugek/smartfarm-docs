# GAP Ruleset Clarifications

Versioned, append-only clarifications to the SmartFarm GAP ruleset and commodity overlays. Each entry is a patch-level (clarification only — no semantic widening, no relaxation) tightening of an existing control. Engineering applies these when bumping the corresponding JSON artifacts.

Authoritative artifacts (located in the engineering repo, current path `compliance/`):

- base ruleset: `usda-hgap-v1.json`
- leafy-greens overlay: `usda-hgap-overlay-leafy-greens-v1.json`

Versioning rules:

- patch bump (`x.y.Z`) when wording is tightened, an `advisory_note` is added, or a vacuous-pass is closed without changing the set of conformant evidence already accepted.
- minor bump when a new control is added or evidence semantics widen.
- major bump on any incompatibility with prior accepted evidence.

A clarification is **not** a relaxation. If applying a clarification would invalidate evidence that was previously accepted, escalate to a minor or major bump and open a separate issue.

---

## Patch: ruleset 1.0.0 → 1.0.1, leafy-greens overlay 0.1.0 → 0.1.1

- source: OME-80 (this issue)
- input: OME-24a dry-run report `compliance/dry-runs/synthetic-leafy-greens-v1-report.md`
- status: proposed
- target artifacts:
  - `compliance/usda-hgap-v1.json` → bump `ruleset_revision` to `1.0.1` only if base wording in LG.2.1's predecessor (`HGAP.FO.2.2`) needs the cross-reference note below; otherwise leave base at `1.0.0`.
  - `compliance/usda-hgap-overlay-leafy-greens-v1.json` → bump `overlay_version` to `0.1.1`.

### 1. `HGAP.LG.6.2` — harvest container traceability, timestamp granularity

**Problem.** `evidence_spec` does not pin whether timestamps are required per-bin (one record per physical container) or per-event (one record per harvest event covering many containers). Dry-run accepted per-event; a stricter auditor could insist on per-bin and call it a gap.

**Clarification (chosen approach: split the control).** Replace `HGAP.LG.6.2` with two sibling controls so per-bin and per-event traceability are explicit and independently auditable. This is the most durable for AI labeling later.

```yaml
controls:
  - id: HGAP.LG.6.2a
    supersedes: HGAP.LG.6.2  # for the per-event slice
    title: Harvest container traceability — per-event log
    evidence_required: true
    expert_review_required: false
    evidence_spec:
      - kind: document
        guidance: >
          One log entry per harvest event covering all containers in that event.
          Required fields: event_id, harvest_date, crew_id, plot_id, container_count,
          first_container_id, last_container_id, event_started_at, event_ended_at.
      - kind: note
        guidance: Free-text notes about the event (weather, substitutions).
    advisory_note: >
      Per-event is sufficient when containers in one event are homogeneous in
      origin (same plot, same crew, same harvest window). If containers within
      a single event come from multiple plots or are commingled with other
      events, HGAP.LG.6.2b applies.

  - id: HGAP.LG.6.2b
    supersedes: HGAP.LG.6.2  # for the per-bin slice
    title: Harvest container traceability — per-bin log
    evidence_required: true
    expert_review_required: true
    applies_when:
      any_of:
        - containers_span_multiple_plots: true
        - containers_commingled_across_events: true
        - buyer_or_scheme_requires_per_bin: true
    evidence_spec:
      - kind: document
        guidance: >
          One log entry per physical container. Required fields:
          container_id, plot_id, harvest_date, packed_at (timestamp,
          minute granularity), crew_id, weight_or_count.
      - kind: photo
        guidance: Photo of container label or QR tag, captured at pack time.
    advisory_note: >
      `packed_at` minute-granularity is the floor; second granularity is
      preferred where the device clock supports it. Time zone must be
      explicit (ISO 8601 with offset).
```

**Migration.** Existing per-event evidence captured under the old `HGAP.LG.6.2` maps to `HGAP.LG.6.2a` with no relabeling. Records that meet the `applies_when` triggers must be re-labeled to `HGAP.LG.6.2b` and the per-bin evidence collected on the next harvest.

### 2. `HGAP.LG.7.1` — in-field worker handwash attestation, vacuous-pass

**Problem.** Current control passes if the attestation log file exists, even with zero entries. A farm that never logs handwash events would still be marked compliant.

**Clarification.** Require at least one log entry per harvest day on which workers are present in the field, and mark the control as expert-reviewed so a human catches edge cases (e.g. solo-operator harvests where the log convention may differ).

```yaml
controls:
  - id: HGAP.LG.7.1
    title: In-field worker handwash attestation
    evidence_required: true
    expert_review_required: true   # was false
    evidence_spec:
      - kind: document
        guidance: >
          Handwash attestation log. MUST contain at least one entry for each
          calendar day on which any worker (including the operator) is present
          in the field for harvest or in-field handling activity. Each entry
          MUST carry: date, time, worker_id_or_initials, station_id_or_location,
          attested_by.
        constraints:
          min_entries_per_harvest_day: 1
      - kind: photo
        guidance: Optional photo of handwash station condition at start of day.
    advisory_note: >
      Solo-operator farms still require at least one self-attestation entry
      per harvest day. "No workers in field" days are exempt — the absence
      must be recorded in the activity log under HGAP.LG.6.2a so the auditor
      can reconcile.
```

**Engine note for engineering.** The `min_entries_per_harvest_day: 1` constraint is the first structured constraint we are introducing into `evidence_spec`. Engineering can either (a) treat it as advisory text rendered to the expert reviewer and rely on review to catch zero-entry days, or (b) implement a derived check during readiness computation. (a) is acceptable for 1.0.1; (b) is preferred and tracked separately.

### 3. `HGAP.LG.2.1` — water testing frequency, advisory third test

**Problem.** Overlay text is silent on the LGMA-style third confirmatory test that some growers run after a positive or borderline result. A reviewer reading the spec literally could flag a missing third test as a gap when the third test is in fact out of scope of the LGMA practice being referenced.

**Clarification.** Add an `advisory_note` (no change to `evidence_required` or pass criteria) that names when the third test is in scope and when it is not. This is a documentation tightening; no evidence already accepted is invalidated.

```yaml
controls:
  - id: HGAP.LG.2.1
    title: Pre-harvest water testing frequency (leafy greens)
    supersedes: HGAP.FO.2.2          # base control still exists; overlay narrows it for leafy greens
    evidence_required: true
    expert_review_required: true
    evidence_spec:
      - kind: measurement
        guidance: >
          Two pre-harvest water tests per crop cycle, taken from the active
          irrigation source within the windows defined by the LGMA practice
          referenced. Each test record carries: sample_date, source_id,
          analyte (generic E. coli MPN/100 mL or scheme-specified analyte),
          result, lab_id, method.
    advisory_note: >
      A third confirmatory test is in scope ONLY when (i) either of the two
      required tests returns above the LGMA action threshold, or (ii) the
      grower's contract or buyer specification calls for it. Outside those
      cases the third test is optional and its absence MUST NOT be flagged
      as a gap. When a third test is run, store it under the same control
      id with `sample_role: confirmatory` so the reviewer can see why it
      exists.
    references:
      - LGMA Commodity Specific Food Safety Guidelines, water section (current edition)
      - HGAP.FO.2.2 (base control this overlay narrows)
```

---

## Application checklist for the engineer who picks this up

1. Read this section end to end.
2. Open `compliance/usda-hgap-overlay-leafy-greens-v1.json`. Apply the YAML above by translating to the overlay's JSON shape, keeping field names identical. Bump `overlay_version` to `0.1.1` and append a changelog entry pointing to OME-80.
3. If `HGAP.FO.2.2` in `usda-hgap-v1.json` does not already cross-reference the leafy-greens overlay, add a one-line `references` entry pointing to `HGAP.LG.2.1` and bump `ruleset_revision` to `1.0.1`. Otherwise leave the base ruleset untouched.
4. Re-run the OME-24a-style synthetic dry-run against the new overlay. Expected: per-event evidence still passes (now under `HGAP.LG.6.2a`), zero-entry handwash logs now fail (`HGAP.LG.7.1`), and the missing third water test no longer surfaces as a finding when the action threshold is not crossed.
5. Comment back on OME-80 with the file diff and the dry-run delta.

## Open questions for the next clarification batch

- whether `min_entries_per_harvest_day` should be promoted to a first-class field in `evidence_spec` across all schemes, or kept inside `constraints:` until we have more examples.
- whether `applies_when` predicates should reference farm metadata (plot count, commingling policy) directly, or be evaluated by the expert reviewer.

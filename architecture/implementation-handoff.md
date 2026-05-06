# Implementation Handoff (Phase 1)

This is the contract engineering must build to. The product and GAP docs are the why; this is the what-and-where.

## Domain entities (Phase 1)

- `Organization`
- `User`
- `Membership` (user × organization × role)
- `FarmSite` (organization)
- `Plot` (farm site)
- `ComplianceScheme` (loaded as data; see `gap/phase-1-gap.md`)
- `OrganizationSchemeBinding` (organization × compliance scheme; one active at a time in Phase 1)
- `CropCycle` (plot, crop, planned dates, status, assigned expert)
- `GapRecord` (crop cycle, record type, payload JSON matching scheme schema, occurred_at, actor, supersedes_record_id)
- `Evidence` (see `gap/evidence-strategy.md` for fields)
- `RecordReview` (gap record × expert × state × comment × created_at)
- `ReadinessSnapshot` (crop cycle × generated_at × frozen content × generator_user_id)

## Roles and authorization

See `product/roles.md`. Authorization is checked per organization context. A request always carries org context plus the acting user's role in that org.

## API surfaces engineering must expose in Phase 1

Grouped by capability, not endpoint shape (Tech Lead chooses REST/RPC details):

- org and membership management
- farm site and plot CRUD
- compliance scheme assignment for the org
- crop cycle CRUD and expert assignment
- gap record create, list (by cycle), get, supersede
- evidence upload, list (by cycle and by record), supersede
- record review create and list (by cycle and by record)
- readiness state read for a crop cycle
- readiness snapshot generate and fetch

## Non-negotiables

- All writes record `actor_user_id`, `org_id`, server `created_at`.
- Records and evidence are append-only; supersession instead of mutation.
- Authorization is per-org; cross-org reads are not allowed.
- Compliance scheme content is data, not code.
- The readiness state is derivable from records + evidence + reviews; never trust a stored boolean as truth.
- Evidence storage preserves originals and content hash.

## What engineering should NOT build in Phase 1

- recommendation engines
- AI/ML inference of any kind
- public marketplace or expert directory
- billing
- offline sync engine (caching is fine; conflict resolution is not in scope)

## Source of truth for changes to this contract

This file. Changes to the Phase 1 contract are made by editing this file in `smartfarm-docs` and referencing the change in the relevant engineering issue. Engineering issues should link back to specific files in this repo rather than restating product intent.

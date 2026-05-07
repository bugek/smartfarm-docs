# smartfarm-docs

Documentation repository for the SmartFarm platform. This is the **canonical source of truth** for product intent, GAP scope, and the Phase 1 implementation contract.

## Phase 1 canonical files

Engineering should treat the following as the contract of record. Link to these from issues instead of restating intent.

### Product
- [`product/vision.md`](product/vision.md) — what we are building and why, GAP-first
- [`product/mvp-scope.md`](product/mvp-scope.md) — Phase 1 in/out of scope and acceptance signal
- [`product/roles.md`](product/roles.md) — farmer, expert, compliance lead, admin
- [`product/user-flows.md`](product/user-flows.md) — record entry, expert review, readiness reporting
- [`product/gap-workflow-ux-acceptance.md`](product/gap-workflow-ux-acceptance.md) — UX/UI acceptance criteria for GAP workflows

### GAP
- [`gap/phase-1-gap.md`](gap/phase-1-gap.md) — what GAP-first means for the platform
- [`gap/evidence-strategy.md`](gap/evidence-strategy.md) — image, video, document evidence rules
- [`gap/ruleset-clarifications.md`](gap/ruleset-clarifications.md) — versioned clarifications applied when bumping ruleset / overlay JSON
- [`gap/thai-gap-clause-8-records-traceability.md`](gap/thai-gap-clause-8-records-traceability.md) — Thai GAP clause 8 contract for seasonal records, lot traceability, buyer logs, and follow-up actions

- [`gap/thai-gap-clause-3-hazardous-substances.md`](gap/thai-gap-clause-3-hazardous-substances.md) - Thai GAP clause 3 contract for hazardous substance records, evidence, and controls
- [`gap/fertilizer-application-phase-1-2.md`](gap/fertilizer-application-phase-1-2.md) - Phase 1.2 fertilizer application record requirement and product/API handoff

### Architecture
- [`architecture/repo-map.md`](architecture/repo-map.md) — repos and ownership
- [`architecture/implementation-handoff.md`](architecture/implementation-handoff.md) — entities, capabilities, non-negotiables for engineering

### Operations
- [`operations/agent-ownership.md`](operations/agent-ownership.md) — who owns what

## Folder guide

- `product/` product framing, scope, roles, flows
- `gap/` GAP rules and evidence strategy
- `architecture/` system boundaries, data model, engineering contract
- `operations/` rollout and team operating model

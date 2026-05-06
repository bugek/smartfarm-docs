# Phase 1 GAP Focus

The platform starts with GAP before advisory automation. This document defines what "GAP-first" means for engineering in Phase 1.

## Scheme scope

Phase 1 supports **one configurable compliance scheme per organization**. The scheme defines:

- the set of required GAP record types
- for each record type, the required structured fields
- for each record type, the required evidence kinds (e.g. image of label, document of certificate)
- the rule for when a crop cycle is considered "ready"

The exact scheme content (e.g. ThaiGAP, GLOBALG.A.P. subset) is the GAP Compliance Expert's deliverable. The platform must not hard-code it; it must load it as data.

## Engineering guarantees

A Phase 1 GAP record must:

- be tied to an organization, site, plot, and crop cycle
- carry a typed payload that matches the scheme's record type schema
- be append-only with a clear chain to any superseding record
- carry actor, `occurred_at`, and `created_at`
- support zero-or-more evidence attachments

A Phase 1 evidence item must:

- be one of: image, video, document
- carry uploader, `captured_at` (best available), `uploaded_at`, optional geo
- be attached to a record or a crop cycle (never orphan)
- be retained for the life of the crop cycle; supersession is allowed, deletion from history is not

## Readiness state

A crop cycle's readiness is derived, not stored as the source of truth. The derivation:

1. For each required record type in the scheme, check that at least one current (non-superseded) record exists.
2. For each present record, check that required evidence kinds are attached.
3. Apply expert review states. Any `blocking` state forces overall state to `not_ready`.
4. Otherwise: `ready` if all required items are present and reviewed, `partial` if some items lack expert review but no blocking flags, `not_ready` if items are missing.

## What this enables later

The same record + evidence shape, plus expert review states, is the training and inference substrate for later AI-assisted review. Phase 1 must keep payloads structured enough to be queryable and labelable later. No AI work in Phase 1.

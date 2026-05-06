# Evidence Strategy

Evidence is the spine of GAP credibility. If a record is not backed by evidence the scheme requires, it is incomplete.

## Evidence kinds (Phase 1)

- **image**: photo from phone, primarily to show field conditions, applied product label, or completed work.
- **video**: short clip, used where a still image is not enough (e.g. spray demonstration, equipment calibration).
- **document**: PDF or image of an external document (certificate, invoice, lab result).

No other kinds in Phase 1. Audio, sensor streams, and IoT telemetry are deferred.

## Required metadata per evidence item

- `id`
- `kind` (image | video | document)
- `crop_cycle_id` (always)
- `record_id` (when attached to a specific record)
- `uploader_user_id`
- `captured_at` (best-effort: EXIF for images, file timestamp for documents, capture moment for videos; fall back to `uploaded_at`)
- `uploaded_at`
- `geo` (lat/lon, optional, only if the capturing device provides it)
- `mime_type`, `byte_size`, content hash
- `superseded_by_evidence_id` (nullable)

## Storage and lifecycle

- Originals are immutable. We never overwrite stored bytes.
- Supersession: a newer evidence item can mark an older one as superseded with a reason. Both are retained.
- Deletion: hard delete is not a Phase 1 user action. Org-level redaction is an admin-only edge case and must leave a tombstone.

## Capture expectations

- Capture works from a mobile browser.
- Image capture should attempt to preserve EXIF (for `captured_at` and geo).
- Large videos must be size-capped in Phase 1 (suggest 100 MB per item, to be confirmed by Tech Lead based on storage budget).

## Linking rules

- Evidence may be attached at the crop cycle level (general context) or at the record level (specific proof for that activity).
- The readiness check counts only evidence attached at the record level toward "evidence required for record type X".
- Cycle-level evidence appears in the readiness report appendix.

## What the expert sees

- Thumbnails and a preview pane for each evidence item attached to the record under review.
- Capture time and uploader.
- Geo flag if the location is far from the registered plot location (advisory only in Phase 1, no auto-block).

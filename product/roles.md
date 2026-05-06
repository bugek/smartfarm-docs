# Roles

Phase 1 ships with four roles. Roles are scoped per organization. A user account may hold different roles in different organizations.

## Farmer
- Primary capture role.
- Creates and edits GAP records on crop cycles they are assigned to.
- Uploads evidence.
- Can read expert comments on their own records.
- Cannot change another farmer's records.

## Expert (advisor)
- Review role.
- Sees crop cycles where they are assigned as expert.
- Comments on records and evidence.
- Sets per-record review state: `reviewed`, `needs_more_evidence`, `blocking`.
- Cannot author farmer records on the farmer's behalf in Phase 1.

## Compliance Lead
- Org-level oversight role.
- Sees all crop cycles in the organization.
- Configures which compliance scheme the organization is preparing for.
- Triggers and exports readiness reports.
- Can reassign experts to crop cycles.

## Admin
- Org administration only.
- Manages members, role assignments, farm sites, plots.
- Does not perform GAP review and does not author farmer records.
- May also hold another role concurrently if needed.

## Permissions summary

| Capability                           | Farmer | Expert | Compliance Lead | Admin |
|--------------------------------------|:------:|:------:|:---------------:|:-----:|
| Create/edit own GAP records          | yes    | no     | no              | no    |
| Upload evidence                      | yes    | yes\*  | yes\*           | no    |
| Comment on records/evidence          | yes    | yes    | yes             | no    |
| Set record review state              | no     | yes    | yes             | no    |
| Configure compliance scheme          | no     | no     | yes             | no    |
| Export readiness report              | no     | no     | yes             | no    |
| Manage members, sites, plots         | no     | no     | no              | yes   |

\* Experts and compliance leads may upload evidence only as part of a review action (e.g. annotated screenshot, supporting document); they do not author farmer activity records.

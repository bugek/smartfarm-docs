# Repo Map

## Repositories

- `smartfarm-api` backend system of record (Node/TypeScript service, primary workspace)
- `smartfarm-web` user-facing application (web frontend; mobile browsers are a first-class target in Phase 1)
- `smartfarm-docs` documentation and domain decisions (this repo; canonical product and GAP contract)
- `smartfarm-infra` local and deployment infrastructure

## Ownership

- **Product GAP Lead**: docs and product scope (`smartfarm-docs`)
- **GAP Compliance Expert**: GAP scheme content and evidence expectations
- **Tech Lead**: backend architecture and `smartfarm-api`
- **Platform Engineer**: `smartfarm-web` and integration

## How issues should reference this repo

Engineering issues for Phase 1 should link to specific files in `smartfarm-docs` instead of restating product intent. The canonical files are listed in the README index.

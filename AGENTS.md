# AGENTS.md

Instructions for contributors and coding agents working in `smartfarm-docs`.

## Workflow Mode

This repository uses a **branch + pull request** workflow.

- Do not push content changes directly to `main`.
- Create a branch for each issue or document slice.
- Push the branch and open a PR.

## Branch Naming

- `docs/ome-<number>-<slug>`
- `feat/ome-<number>-<slug>` when the change creates a new durable contract artifact

Examples:

- `docs/ome-86-gap-first-contract`
- `docs/ome-80-ruleset-clarifications`

## Pull Request Rules

- Title format: `[OME-86] Publish SmartFarm docs baseline`
- Name the files that become the contract of record.
- Keep docs concise, implementation-oriented, and phase-aware.

## Documentation Priorities

1. product vision
2. GAP scope
3. workflow definitions
4. domain vocabulary
5. engineering handoff contracts


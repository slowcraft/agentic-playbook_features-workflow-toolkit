# Sequence + dependencies — <feature>

Phase-ordered stories. Updated when a story moves between phases.

## Reading guide

- **Phase** groups stories by when they unblock
- **Can parallelise with** lists sibling stories workable simultaneously
- **Blocked by** lists stories that must finish first
- **Owner** distinguishes this team from external

## Done

| # | Story file | What landed |
|---|---|---|
| (none yet) | | |

## Phase 1 — <name>

| # | Story | Owner | Can parallelise with | Blocked by |
|---|---|---|---|---|
| 1.1 | `<slug>.md` | this team | <other story> | nothing |

## Phase 2 — <name>

| # | Story | Owner | Can parallelise with | Blocked by |
|---|---|---|---|---|

## Backlog / Future

| # | Story | Notes |
|---|---|---|

## Convention — story lifecycle

When a story lands and the work is verified complete:

1. Rename with the next `NNNN_` prefix (`0001_`, `0002_`, ...)
2. Move to `stories/done/`
3. Update this doc — add to Done; remove from open phase tables
4. Reference the story in commits per your project's commit format (e.g., `[#<ticket-id>]` or `[story:<slug>]`)

The CI check `sequence_doc_invariants.py` (if installed) validates that every open story file is referenced here and every entry resolves to a file.

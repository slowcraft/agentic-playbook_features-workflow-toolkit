# Feature docs scaffold

Drop this directory into your project at `.claude/docs/<feature>/` (typically gitignored — these are private design notes per developer). It's the scaffold the `agent-workflow-toolkit` plugin's `initializing-feature-docs` skill populates automatically.

If you scaffolded this directory by hand, copy the contents wholesale, then:

1. Customise `decisions/decisions_and_punch_list.md` for your project (first A-rows: branch model, package layout, test approach, commit convention)
2. If your feature has an eval target, sketch initial scenarios in `specs/test_scenarios_catalog.md` with stable IDs (you'll need to write this file fresh — the scaffold doesn't ship a catalog template because catalog families are domain-specific)
3. Populate `stories/SEQUENCE_AND_DEPENDENCIES.md` with your first 1–3 stories
4. Add the first ADR under `decisions/adrs/` for any decision with credible alternatives worth recording

## What lives here

| Path | Purpose |
|---|---|
| `decisions/decisions_and_punch_list.md` | Append-only A-table + open-questions table |
| `decisions/adrs/` | Long-form ADRs for decisions with credible alternatives |
| `specs/` | Canonical artifacts (output contracts, scenario catalogs, implementation plans) |
| `stories/SEQUENCE_AND_DEPENDENCIES.md` | Phase-ordered open work + dependency graph |
| `stories/` | One file per open story; moves to `done/` with numeric prefix when landed |
| `stories/done/` | Numerically-prefixed landed stories (linear history of what shipped) |
| `stories/future/` | Post-MVP backlog |
| `experiments/` | Append-only "I tried X, observed Y, picked Z" log |
| `research/` | Deep-research outputs (`deep-research-toolkit` skill writes here) |

# Decisions log — <feature>

Architecture decisions for the `<feature>` feature. Each row's `A-NN` identifier is stable forever; rows are append-only. Long-form ADRs live under `adrs/` for decisions with credible alternatives worth recording.

## A-table

| ID | Decision | Rationale | Status | Cross-refs |
|---|---|---|---|---|
| A1 | <your first decision here, e.g., branch model> | <why this and not the alternatives> | settled | ADR-0001 (if applicable) |
| A2 | <next decision> | <rationale> | settled | — |
| ... | | | | |

## Open questions

| # | Question | Why it matters | Owner | Status |
|---|---|---|---|---|
| Q1 | <unresolved question> | <what's blocked by this> | <team / person> | open |
| ... | | | | |

## Supersession

When a decision changes, add the new A-row, mark the superseded row's Status as `superseded by A<N>`, and leave both for the historical record.

## Conventions

- Append-only — never renumber A-rows. The CI check `stable_id_no_renumber.py` (if installed) enforces this.
- Settled decisions belong in the A-table. Use ADRs (under `adrs/`) for the subset that had credible alternatives.
- Open questions belong in the open-questions table with owners + status. When resolved, they become A-rows.
- Every commit that's informed by a decision should cite the A-row in its body.

# ADRs — <feature>

Long-form rationale for significant decisions. Use an ADR (not just an A-row in `../decisions_and_punch_list.md`) when:

- The decision had at least one credible alternative actively considered and rejected
- The decision has consequences worth spelling out in detail
- A future reader will benefit from the reasoning, not just the conclusion

## Index

| # | Title | Status | Date |
|---|---|---|---|
| 0001 | <first ADR title> | <Accepted / Proposed> | YYYY-MM-DD |

## Writing a new ADR

1. Copy `_template.md` to `NNNN-<slug>.md` (NNNN = next four-digit number)
2. Fill in: context, decision, alternatives considered, consequences
3. Add the row to the Index above
4. Cross-reference from the relevant A-row(s) in `../decisions_and_punch_list.md`

Accepted ADRs are append-only. To change a decision, write a new ADR and mark the old one `Superseded by ADR-NNNN`.

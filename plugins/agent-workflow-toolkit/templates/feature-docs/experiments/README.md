# Experiments — <feature>

Append-only record of non-trivial technical decisions where the rationale is "I tried X, observed Y, picked Z". Useful when six months from now someone asks "why is this threshold set to 0.85?"

Each experiment is a single file at `<YYYY-MM-DD>-<slug>.md` following `_template.md`.

## When to write an experiment

- When you set a threshold value (and another value would have plausibly worked)
- When you choose a library / model / approach over a credible alternative
- When you make a heuristic decision based on empirical observation
- When a test or benchmark informed a code change worth remembering

## When NOT to write an experiment

- For uncontested choices ("we used Python because the project is Python")
- For implementation details documented in code comments
- For decisions captured fully in an A-row or ADR

## Lifecycle

- Append-only — never edit a landed experiment except to add cross-references or fix typos
- If conclusions change, write a follow-up experiment that cites the original
- A-rows + ADRs citing an experiment should link to the file by name

## Index

| Date | Slug | A-row(s) | ADR(s) |
|---|---|---|---|
| (none yet) | | | |

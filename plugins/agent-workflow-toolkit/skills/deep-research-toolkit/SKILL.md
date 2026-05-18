---
name: deep-research-toolkit
description: Graduated autonomous research for evaluating tools, technologies, ideas, and approaches. Use when researching any topic that requires consulting multiple sources, comparing options, or building a structured knowledge base. Triggers on "research X", "evaluate X", "compare X vs Y", "what's the best approach for X", "investigate X".
---

# deep-research-toolkit

Graduated, source-cited research pass that scales depth to the question.

## When to use

- Evaluating tools, frameworks, libraries, platforms
- Comparing approaches (tool A vs B vs C)
- Investigating feasibility of an idea
- Building context on an unfamiliar topic
- Exploring prior art or competition
- Any question that needs more than a single web search

## When NOT to use

- Writing code (use implementation skills)
- Debugging (use `systematic-debugging`)
- Simple factual lookups (just answer directly)
- Medical / scientific literature review (use a domain-specific skill instead)

## Constants

```
DEPTH = auto           # auto | quick | medium | deep
MAX_SOURCES = 50       # cap per depth level: quick=10, medium=25, deep=50
OUTPUT_DIR = docs/research/   # findings go here (or .claude/docs/.../research/ for private)
STATE_FILE = .research-state.json
```

Override via natural language: "research X at deep depth" or "quick research on Y."

## Depth levels

| Level | Time | When | What happens |
|---|---|---|---|
| **Quick** | 5–10 min | Clear question, likely documented | Web search + skim top results. Stop if confident. |
| **Medium** | 15–30 min | Multiple options to compare, need structured analysis | Multiple sources, build comparison, check GitHub repos, read docs. |
| **Deep** | 30–60 min | Novel topic, high-stakes decision, need exhaustive coverage | Parallel subagent research, cross-reference, build knowledge base, verify claims. |

**Auto mode** (default): start at Quick. If confidence is LOW after Quick, escalate to Medium. If still LOW after Medium, escalate to Deep. Stop as soon as confident.

## Workflow

### Phase 1 — Frame the question

Before searching anything:

- **Restate** the research question in one sentence
- **Identify** what type of answer is needed: comparison, recommendation, feasibility, survey, or deep-dive
- **Define** success criteria: what would a GOOD answer contain?
- **Select** depth (auto / quick / medium / deep)

Present the framing to the user. Wait for confirmation unless explicitly told to auto-proceed.

### Phase 2 — Quick check (all depths)

Search external sources. **Never answer from training data — it's stale and prone to fabrication.** Use whichever of these tools are available in the current session, in roughly this priority order:

| Tier | Tool | When |
|---|---|---|
| **Always available** | `WebSearch` (built-in) | Broad coverage, start here |
| **Always available** | `WebFetch` (built-in) | Pulling specific URLs surfaced by search |
| Plugin (optional) | `firecrawl:firecrawl-search`, `firecrawl:firecrawl-scrape` | Higher-quality scraping when content is JS-rendered |
| Plugin (optional) | `mcp__context7__resolve-library-id` + `query-docs` | Authoritative library / framework docs |
| Tool (optional) | `gh api` | GitHub repo metadata, stars, recent commits, issues |
| Project | Existing `docs/research/` or `.claude/docs/<feature>/research/` | Avoid redoing past work |

If none of the web-capable tools are available, **stop and tell the user** — do not invent answers from memory.

**Assess confidence:**

- **HIGH**: authoritative answer from 2+ independent sources that agree → output and stop
- **MEDIUM**: partial answers, some gaps or contradictions → if quick depth, output with caveats; otherwise escalate
- **LOW**: no clear answer, or conflicting information → escalate

### Phase 3 — Structured research (medium + deep)

Build a structured comparison or analysis using one of three templates.

**Tool / technology evaluation template:**

```markdown
## [Topic] Research

### Candidates

| Tool | What it is | Stars/Adoption | Last updated | License |
|---|---|---|---|---|

### Evaluation dimensions

| Dimension | Tool A | Tool B | Tool C |
|---|---|---|---|
| Setup complexity | | | |
| Documentation quality | | | |
| Community size | | | |
| Maintenance activity | | | |
| Cost | | | |
| Fit for our use case | | | |

### Key findings

### Recommendation

### Sources
```

**Approach / architecture evaluation template:**

```markdown
## [Topic] Research

### Options considered

- Option A — …
- Option B — …
- Option C — …

### Trade-offs

| Dimension | Option A | Option B | Option C |
|---|---|---|---|

### Decision factors

- What matters most for our context?
- What are the irreversible choices?
- What can we change later?

### Recommendation
```

**Idea / feasibility investigation template:**

```markdown
## [Topic] Research

### Prior art

What already exists? Who's done this before?

### Technical feasibility

- Can this be built with available tools?
- What are the hard parts?

### Open questions

What we still don't know.

### Next steps

What to investigate further.
```

### Phase 4 — Deep investigation (deep only)

Spawn parallel subagents (the `Agent` tool) for independent research tracks:

```
Agent 1: Research track A — specific question
Agent 2: Research track B — specific question
Agent 3: Research track C — specific question
```

Each subagent:

- Gets a focused research question
- Returns structured findings
- Runs independently (no shared context)

When using parallel agents, set `model: "opus"` and `run_in_background: true` per project convention. After all subagents return, synthesize:

- **Aggregate** findings from all tracks
- **Cross-reference** — do sources agree? Flag contradictions.
- **Rank** options by fit for our specific context
- **Identify** gaps — what do we still not know?

### Phase 5 — Output

Write findings to `OUTPUT_DIR/<topic-slug>.md` with:

- Research question
- Methodology (what was searched, how many sources)
- Findings (structured per the templates above)
- Recommendation with confidence level
- Sources with URLs
- Date researched
- Open questions / future research

## State persistence

For medium / deep research that spans multiple interactions, write `.research-state.json` next to the findings file:

```json
{
  "topic": "...",
  "depth": "medium",
  "phase": 3,
  "started": "2026-05-15T22:00:00Z",
  "sources_consulted": 15,
  "findings_file": "docs/research/topic-slug.md",
  "status": "in_progress",
  "next_action": "evaluate remaining 3 candidates"
}
```

On resumption: read state file + findings file to restore context. Continue from saved phase.

## Anti-patterns

| Anti-pattern | Why it fails | Correct behavior |
|---|---|---|
| **Answering from memory** | Training data is stale; you may hallucinate details | Always search, even for topics you "know" |
| **Single-source trust** | One blog post is not truth | Cross-reference 2+ independent sources |
| **Scope creep** | Researching everything instead of answering the question | Stay focused on the success criteria from Phase 1 |
| **Analysis paralysis** | Collecting more data instead of synthesizing | Set a source cap per depth level. Stop and synthesize. |
| **Recency bias** | Newest is not best | Consider maturity + stability, not just publish date |
| **Ignoring negatives** | Only finding evidence FOR an option | Actively search for criticisms and failure cases |

## Iron rules

1. **NEVER fabricate sources.** If you can't find a URL, say "could not verify."
2. **NEVER present opinion as fact.** Label uncertainty explicitly.
3. **ALWAYS cite sources** with URLs in the output.
4. **STOP when confident.** Don't over-research simple questions.

## Output placement convention

- Public / committed research: `docs/research/<topic-slug>.md`
- Private / per-developer research: `.claude/docs/<feature>/research/<topic-slug>.md`

Pick based on whether the artifact is meant to be shared with the team via the repo, or kept as personal working notes.

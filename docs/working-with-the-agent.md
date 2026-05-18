# Working with the agent — day-to-day operator guide

This is the practical guide for using the toolkit once it's set up. Read it after `setup_a_new_agent_to_work_on_this.md` and `configuration_reference.md`.

The goal: tell you what to say to Claude, when, and how to keep the workflow's discipline intact without it feeling like overhead.

## Mental model — what Claude does well vs. poorly

**Claude is good at**:
- Synthesizing across many docs at once (decisions + stories + specs + tests)
- Writing structured documents (catalogs, ADRs, plans) when given clear conventions
- Finding cross-references and inconsistencies between artifacts
- Brainstorming options with explicit trade-offs at each fork
- Iterating on a single artifact based on your steering

**Claude is poor at**:
- Remembering details across sessions (no persistent memory between sessions unless captured in files)
- Knowing what state your project is in without being told
- Making decisions without your input on irreversible / opinionated choices
- Catching its own drift from your project's conventions (without explicit anchors)

The workflow compensates for the weaknesses by **writing everything down**, **stably numbering everything**, and **cross-referencing everything**.

## Starting a session

When you open a new Claude Code session on a project, your first message sets the tone. Three useful framings:

### "Tell me where things are"

For a session where you don't yet know what's in flight:

```
What's the current status of <feature>? Read the sequence doc and recent ADRs and tell me:
1. What's done
2. What's in flight
3. What's blocked and on whom
4. What's the recommended next step
```

Claude reads the docs, summarizes, and surfaces blockers. This grounds the session in current reality.

### "I want to work on X"

For a session with a specific goal:

```
I want to work on <story slug or ticket id>. Read the relevant story file + any specs/ADRs it cites and tell me what's the smallest first step.
```

Claude reads the context, proposes a first step, and asks for steering. Don't skip the "read first" — answers from cold are worse than answers from context.

### "Brainstorm with me"

For exploratory work — designing a new feature, evaluating an approach, debugging a confusing pattern:

```
I want to brainstorm <topic>. Don't write anything yet — first frame the question, list what we know, list what we don't, and propose 2-3 options to evaluate.
```

This delays artifact creation until the shape is clear. Most brainstorms produce a doc at the end; delaying that doc until you've explored prevents premature commitment.

If you have a dedicated brainstorming skill installed (e.g., `superpowers:brainstorming`), use it here — it's a natural pairing with this toolkit's downstream skills, which assume you've already decided *what* to scope.

## The seven primary actions in this workflow

These are the recurring operations. For each: what it is, when to do it, and how to invoke it.

### 1. Write a story

When you have a unit of work scoped at "what + why" level.

```
You: write a story for <topic>. Source: <link to slack thread, incident, or ticket>.

Claude: [invokes writing-stories skill]
        Produces a story under stories/<slug>.md with the template.
        Asks you to confirm scope and dependencies.
```

The skill enforces: active-verb title, why-not-how summary, 3–5 actionable Description of Work bullets, and a Dependencies section covering anything the story is blocked on or coupled to.

**Custom story shapes.** If your team has its own story format, drop a markdown file at `<stories_dir>/_template.md` and the skill will use it verbatim instead of the built-in default. You can also point `stories.template_file` in `.claude-toolkit.yaml` at any path, or let the skill infer the format from the most recent file in `<stories_dir>/done/`. On first invocation in a project with none of those signals, the skill asks once how you want stories structured — see the skill's "Template resolution" section for the full ladder.

After landing the story, ask Claude to update `SEQUENCE_AND_DEPENDENCIES.md` to include the new story in the appropriate phase.

### 2. Log an architecture decision

When you make a choice that future-you (or a teammate) might second-guess.

```
You: we just decided that <decision>. Log it.

Claude: [appends to decisions_and_punch_list.md as the next A<N> row]
        Asks: "Does this have credible alternatives that were rejected? If yes, write a long-form ADR."
```

For decisions with at least one rejected alternative, write the long-form ADR under `decisions/adrs/`. For uncontested decisions, the A-row is enough. See `decisions/adrs/_template.md` in the toolkit scaffold for the ADR shape.

### 3. Add a scenario to the test catalog

When a new edge case, attack vector, or boundary surfaces during discussion.

```
You: what about <edge case>? — that's not in the catalog.

Claude: [adds a row to specs/test_scenarios_catalog.md with the next stable ID]
        Includes: slug, description, expected verdict, synthesis method.
```

Stable IDs (P-NN, M-NN, …) never get renumbered — only appended. The catalog grows monotonically.

### 4. Run an experiment, then log it

When you pick a threshold, choose a library/model over alternatives, or make a non-trivial technical choice.

```
You: we set <threshold> to <value> because <reasoning>. Log the experiment.

Claude: [creates experiments/<date>-<slug>.md]
        Includes: hypothesis, setup, results, conclusion.
```

Append-only. Never edit a landed experiment — write a follow-up entry instead. Decisions that change should cite the experiment file that justified the change.

### 5. Dispatch parallel work to subagents

When you have 2+ independent stories that can be worked simultaneously and your harness has the `Agent` tool.

```
You: dispatch agents for stories A, B, and C.

Claude: [invokes dispatching-client-story-agent skill]
        Reads .claude-toolkit.yaml for defaults.
        Produces 3 Agent tool calls, sent in a single message so they run in parallel.
        Each agent runs in an isolated worktree on its own story branch.
```

After all agents complete, audit each branch (tests, scope adherence, cross-branch conflicts) before merging into the integration branch.

**If your harness doesn't have the Agent tool** the dispatching skill is still useful — it produces a prompt you can paste into a separate Claude Code session manually. Or do the work serially in the main session.

### 6. Do a deep research pass

When evaluating tools, comparing approaches, or investigating an unfamiliar topic.

```
You: research <topic>. Depth: <quick / medium / deep>.

Claude: [invokes deep-research-toolkit skill]
        Phase 1: frames the question and asks for confirmation.
        Phase 2: quick check (all depths).
        Phase 3: structured comparison (medium+).
        Phase 4: parallel subagent research (deep only).
        Phase 5: writes findings to research/<topic-slug>.md with sources.
```

Default to auto-mode — Claude escalates depth only when needed. Cap sources per depth to avoid analysis paralysis.

## How to brainstorm effectively

Brainstorming is out of scope for this toolkit — the skills assume you've already decided *what* to scope. Use a dedicated brainstorming skill (e.g., `superpowers:brainstorming` from the [Superpowers plugin](https://github.com/anthropics/claude-code)) or an unstructured back-and-forth with Claude *before* invoking `writing-stories`. When a brainstorm settles into a decision, capture the decision in the ADR or decisions-log; keep the brainstorm itself as exploratory history, separate from the spec it produced.

## How to use deep research effectively

The `deep-research-toolkit` skill scales depth to the question. To use it well:

- **Frame the question precisely** before invoking. "Research React state management" is too broad; "Compare Zustand vs Jotai vs Redux Toolkit for a SPA with <X> requirements" is workable.
- **Default to auto-depth**. Don't ask for `deep` unless you actually need 30+ sources. Most questions resolve at `quick` or `medium`.
- **Read the framing in Phase 1** and either correct it or confirm. Bad framings produce bad research.
- **Source-cite everything**. The skill's iron rule is "never fabricate sources" — if Claude can't cite, it must say so.

For research that depends on private context (your team's prior decisions, your company's licensing posture), tell Claude up front. Public research is necessary but rarely sufficient.

## Anti-patterns to avoid

These all cost time when you do them. Watch for them.

### Anti-pattern: starting work without reading the sequence doc

If you don't know what's in flight, you'll either duplicate work or break someone else's. First action in any new session: read `SEQUENCE_AND_DEPENDENCIES.md` and recent ADRs.

### Anti-pattern: skipping the "fix base commit" preamble when dispatching

The dispatching skill's prompt template has a `CRITICAL FIRST STEP — fix your base commit` block. Agents default to a stale commit and won't see the integration branch. If you skip the preamble, the agent fails silently.

### Anti-pattern: editing landed artifacts to "fix" them

ADRs, experiment logs, and `done/` stories are append-only. Edit them only to add cross-references or correct typos. If a decision changes, write a new ADR that supersedes the old one. If an experiment's conclusion changes, write a follow-up experiment.

### Anti-pattern: not invoking the skills

Skills only fire when Claude detects the trigger phrases. If you describe a decision conversationally but don't say "log this" or "we decided that," Claude might not invoke the maintaining-decisions skill. **Be explicit**: say "log this decision" / "add a catalog row" / "write a story for this" when you want the artifacts created.

### Anti-pattern: dispatching dependent stories in parallel

If story B depends on story A (e.g., B builds on data structures A introduces), they're not independent. Dispatch A first, then B. The dispatching skill's "Anti-patterns" section calls this out — re-read it before dispatching.

### Anti-pattern: ignoring the output contract

If your project has a canonical output contract (a verdict envelope, an API response schema, a serialized result type), every story that touches the output should reference it. Drift between stories produces incompatible field shapes.

## Working without subagents

If your harness doesn't expose the `Agent` tool, everything still works — just serially in the main session. The `dispatching-client-story-agent` skill becomes a **prompt template generator** instead of a dispatcher: it produces the prompt you'd give a subagent, which you can either:

- Paste into a second Claude Code session manually, or
- Use as your own structured framing for working on the story serially

The hooks and CI checks (if installed) work regardless of whether the Agent tool is available — they fire on shell + git events, not on subagent dispatch.

## When the workflow itself needs to change

If you find a pattern in the workflow that's hurting more than helping, propose a change:

```
You: the <X> pattern is causing <problem>. What would change if we did <Y> instead?

Claude: [discusses trade-offs, proposes alternatives]
```

Then capture the outcome as an ADR — meta-decisions about the workflow are still decisions and deserve the same discipline as feature decisions.

## What you can ask Claude to do at any time

A non-exhaustive list of session-time operations that are always available:

- "What's the current state of <feature>?"
- "What's blocked and on whom?"
- "Read these three docs and tell me if they contradict each other"
- "Audit the catalog for missing scenarios in the <X> family"
- "Dispatch story <slug> as a subagent" (if Agent tool available)
- "Verify the integration branch tests pass on every story branch"
- "Brainstorm <topic> — list options + trade-offs before writing anything"
- "Research <topic> — auto depth, write findings to research/"
- "Write a story for <topic>" (invokes writing-stories skill)
- "Log this decision" (invokes maintaining-decisions skill if installed; otherwise appends to A-table directly)
- "Add a catalog row for <edge case>"
- "Write an ADR for <decision>" — for decisions with rejected alternatives
- "Write an experiment log entry for <choice>" — for thresholds + library/model selections
- "Move <story> to done/"
- "Update the sequence doc to reflect <change>"

If the workflow has a moment where you don't know what to do, ask Claude to recommend a next move:

```
You: what should I do next?

Claude: Reads the sequence doc + open questions, recommends the highest-leverage next step.
```

## Maintenance ritual

Once a week (or once per work session if you're working intensively):

1. **Scan open questions in decisions_and_punch_list.md** — are any now answerable that weren't before?
2. **Scan the sequence doc** — any stories that should move between phases? Any newly-blocking dependencies?
3. **Scan recent commits** — do any reference stories that are still in the open table? If yes, the stories should probably move to `done/`.
4. **Scan unresolved decisions** — any blocked on external input that's now available? Any newly arising?
5. **Scan recent experiments and ADRs** — any cross-references to add that weren't obvious at write time?

This is a 10–15 minute ritual that prevents months of drift. Ask Claude to do it with you.

## Where this all comes from

This workflow emerged organically from one mid-sized engineering effort and was generalized afterward. The patterns travel; the specifics (criteria taxonomy, vendor allowlist, output shape, etc.) are project-specific.

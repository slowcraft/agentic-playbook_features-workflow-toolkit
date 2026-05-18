---
name: dispatching-client-story-agent
description: Use when dispatching a background subagent to implement a story under a feature's integration branch. Triggers on "dispatch agent for story X", "spin up a subagent for this story", "send this story to a parallel worker", "create the Agent-tool prompt for story X", or any phrasing that means "delegate this scoped implementation work to a background agent on its own branch". Produces a self-contained Agent-tool prompt with model selection, worktree isolation, base-commit-fix preamble, scope guards, exclusion rules, and project conventions all bundled.
---

# Dispatching a client-story agent

This skill bundles the boilerplate every story-dispatch needs. Use it to produce one or more parallel Agent-tool invocations with consistent setup.

This is the **generic** version. A project that uses this workflow may keep a project-specific instance (e.g., `dispatching-<feature>-agent`) that hardcodes that project's paths and commands. When porting to a new project, copy this generic skill and parameterize from your project's `.claude-toolkit.yaml`.

## When to use

- You're about to call the `Agent` tool to dispatch background work on a `story/<ticket-id>_<slug>` branch under a feature's integration umbrella
- The work is implementation of a scoped story (typically from your team's ticket system: ADO, JIRA, GitHub Issues, Linear, etc.)
- You want to run the agent in parallel with others and integrate later

## Required inputs

Before invoking, decide:

| Input | Example |
|---|---|
| `ticket_id` | `1234` (ADO), `JIRA-1234`, `GH-456`, etc. |
| `slug` | `short_descriptive_slug` (snake_case) |
| `module_path` | `src/<project-specific-path>/<submodule>/` |
| `forbidden_paths` | List of paths another agent owns or that should not be touched. **Always include `.claude/`** — agents shouldn't edit your private design docs. |
| `deliverable_summary` | 3–5 sentence description of what the agent must produce (modules, classes, functions, tests) |
| `architecture_context` | Verdict shape, finding taxonomy, schema bits relevant to this story. Bundle inline because `.claude/` is gitignored and invisible to the agent. |

Read most of these from `.claude-toolkit.yaml` if it exists in the project root.

## Boilerplate produced by this skill

Every dispatch must include the following blocks. Adapt the *italicised* parts; keep the rest verbatim.

### Agent tool call shape

```
Agent({
  description: "<short story name>",
  subagent_type: "general-purpose",
  isolation: "worktree",
  model: "opus",
  run_in_background: true,
  prompt: <PROMPT_BELOW>
})
```

Defaults: `model: "opus"`, `isolation: "worktree"`, `run_in_background: true`. Read these from `.claude-toolkit.yaml` if the project overrides them.

### Prompt template

```
You are implementing Story <TICKET_ID> (<story title>) for <project name>. You are running on Opus. Your job is to land <deliverable_summary in 1 sentence>.

## CRITICAL FIRST STEP — fix your base commit

Your worktree was created on the default base commit, NOT on the integration branch. Your very first commands:

```bash
git fetch origin
git checkout -b story/<TICKET_ID>_<slug> origin/<integration_branch>
git branch --show-current   # confirm
git log -1 --oneline
```

After this, <module_path> should exist with the expected structure. If it doesn't, stop and report — the base is still wrong.

## Repo + branch

- Repo path: <repo_path> (isolated worktree)
- Integration branch: <integration_branch>
- YOUR branch: story/<TICKET_ID>_<slug>
- DO NOT push to origin. The parent reviews and pushes.
- DO NOT open a PR.

## Story description (verbatim from ticket)

<story description from existing_<source>_stories.md>

## Definition of Done

<DOD bullets from the ticket>

## Architecture context

<the architecture_context input — verdict shape, finding taxonomy, schema, etc.>

## Your scope

<deliverable_summary expanded — module layout, function signatures, what tests to write>

## What NOT to touch

- `.claude/` (gitignored, not in your worktree — must not be created either)
<forbidden_paths input — other agents' files, etc.>

## Coding conventions

<read from project config — language, commit format, lint rules, type-check requirements, etc.>

## Environment

<read from project config — env-prep block>

Run tests:
```
<read from project config — test_command>
```

## Build incrementally

Make multiple small commits. Each commit leaves the tree in a working state. Sample commit subjects:
<3–5 example subjects appropriate to the story>

## Deliverable

1. All work committed on `story/<TICKET_ID>_<slug>` in your worktree
2. Tests pass per the documented command
3. Report-back under 200 words: summary, commit subjects in order, test results, any open questions or surprises

Begin.
```

## Workflow

1. Gather the inputs listed above for the story you're dispatching (read defaults from `.claude-toolkit.yaml`)
2. Fill them into the prompt template
3. Call `Agent` with the documented defaults
4. Repeat for each independent story in the batch — send all the `Agent` calls in a single message so they run truly in parallel
5. Wait for completion notifications (do NOT poll the agent output files)
6. When all in batch complete, audit each branch (tests, scope, cross-branch conflicts) before merging into the integration branch

## Anti-patterns

- Skipping the "fix base commit" preamble — agents default to a stale commit and don't see the module
- Forgetting `model: "opus"` — defaults to a weaker model
- Forgetting the `forbidden_paths` block — agents wander into shared files and create unnecessary merge conflicts
- Dispatching agents whose stories depend on each other in the same batch — sequence, don't parallelize
- Telling agents to push or open PRs — parent handles integration

## Notes

- Agents typically inherit `effortLevel` from the operator's settings
- Agents do NOT see `.claude/` (it's gitignored), so bundle relevant architecture context directly into the prompt
- Agents land on the wrong base commit by default — the "fix base commit" preamble is required, not optional
- After dispatch, never read the agent's output file via shell (would overflow parent context); wait for the notification

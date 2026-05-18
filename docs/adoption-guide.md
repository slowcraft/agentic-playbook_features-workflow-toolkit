# Setting up a new Claude Code agent to work on this workflow

Cold-start guide for a developer adopting the toolkit on their own client project. Assumes Claude Code is installed locally.

## What you're setting up

By the end of this guide:
- Claude Code knows about your project's conventions (commits, branches, test runner, env prep)
- The three portable skills are installed and Claude can auto-trigger them
- Your project has a `.claude/docs/<feature>/` tree with the workflow layers (decisions, stories, specs, experiments)
- Your project has a `CLAUDE.md` that points new sessions at the right context
- (Optional) Hooks fire at the right moments to keep the docs in sync

This is a one-time setup per project + per feature. After it, you work normally.

## Prerequisites

- Claude Code installed and authenticated
- A target project (greenfield or existing) with git initialized
- ~30 minutes for first-time setup; ~5 minutes per subsequent feature

## Step 1 — Decide on your project's conventions

Before configuring anything, answer these questions. The answers go into your project config and are referenced by the skills.

| Question | Why it matters | Example answers |
|---|---|---|
| What's the package manager / language? | Skills tailor their env-prep and test commands | `uv` + Python, `pnpm` + TypeScript, `cargo` + Rust |
| What's the test command? | Skills use it to verify story branches | `pytest`, `pnpm test`, `cargo test` |
| What's the commit format? | Skills generate compliant commit messages | Conventional Commits, gitmoji, semantic-release |
| What integration branch model? | Skills dispatch story branches off the right base | Single integration branch / two-tier (foundation + dev) / trunk-based |
| What ticketing system? | Skills tag commits with ticket IDs | ADO, JIRA, Linear, GitHub Issues, none |
| What paths must agents never touch? | Skills include forbidden-paths in dispatched prompts | UI source, other teams' modules, fixtures (often read-only), `.claude/` |
| Which docs are private vs committed? | Drives where artifacts live | `.claude/` (gitignored, private) or `docs/<feature>/` (committed) |

Capture the answers in a `.claude-toolkit.yaml` at the repo root (template below) — every skill reads from it.

## Step 2 — Install the toolkit

Install via the Claude Code marketplace — see the [install snippet in the root README](../README.md#install). After install, verify with:

```
/plugin info agent-workflow-toolkit@agent-workflow-toolkit
```

You should see `writing-stories`, `deep-research-toolkit`, and `dispatching-client-story-agent` among the available skills.

## Step 3 — Scaffold the feature docs tree in your project

In your project root, create:

```
.claude/docs/<feature>/
├── decisions/
│   ├── decisions_and_punch_list.md          # A-table + open-questions + supersession links
│   └── adrs/                                # long-form ADRs for significant decisions
│       ├── README.md
│       └── _template.md
├── specs/
│   ├── test_scenarios_catalog.md            # rows × criteria × verdicts (if your feature has an eval target)
│   └── <feature>_implementation_plan.md     # phase split for the biggest implementation story
├── stories/
│   ├── SEQUENCE_AND_DEPENDENCIES.md         # phase-ordered open work + parallelism + Done table
│   ├── existing_<source>_stories.md         # local mirror of external ticket system (optional)
│   ├── done/                                # numeric-prefixed landed stories
│   └── future/                              # post-MVP backlog
├── experiments/
│   ├── _template.md                         # hypothesis / setup / results / conclusion
│   └── README.md
└── research/                                # deep-research-toolkit outputs (private)
```

Adapt the scaffold template at `client-project-toolkit/templates/feature-docs/` (or the equivalent reference tree shipped with this manual) as your starting point.

## Step 4 — Configure your project

Create `.claude-toolkit.yaml` at your repo root. See `configuration_reference.md` for the full schema. Minimal example:

```yaml
project:
  name: <your-project>
  feature: <feature-name>
  language: python
  test_command: pytest
  env_prep: |
    cd src/api
    uv sync

commits:
  format: conventional-commits
  co_author_trailer: false

branches:
  model: single-tier
  integration_branch: feature/<feature>-dev
  story_prefix: story/

paths:
  decisions_dir: .claude/docs/<feature>/decisions/
  stories_dir: .claude/docs/<feature>/stories/
  specs_dir: .claude/docs/<feature>/specs/
  experiments_dir: .claude/docs/<feature>/experiments/

forbidden_paths:
  - .claude/
  - <other paths>
```

The file is committed (no secrets). Anyone cloning the repo gets the same conventions automatically.

## Step 5 — Add a CLAUDE.md at the project root

`CLAUDE.md` is auto-loaded by Claude Code at session start as part of the system prompt. Use it to tell new sessions where the workflow docs live and what conventions apply.

Minimal example:

```markdown
# <Project name>

## Workflow

This project uses the client-project-toolkit workflow. Key docs:
- Decisions: `.claude/docs/<feature>/decisions/decisions_and_punch_list.md`
- Stories + sequence: `.claude/docs/<feature>/stories/SEQUENCE_AND_DEPENDENCIES.md`
- Test scenario catalog: `.claude/docs/<feature>/specs/test_scenarios_catalog.md`
- Operator manual: `.claude/docs/meta/README.md`

## Conventions

- Commits: <your project's commit format — see `.claude-toolkit.yaml`>
- Branches: see `.claude-toolkit.yaml`
- Tests: <test command>
- Setup: <env-prep>

## What to do when starting a new session

1. Read the sequence doc to find what's in flight
2. Read recent ADRs to know what's settled
3. Ask the user which story they want to work on
```

CLAUDE.md is committed — every teammate gets the same starting context.

## Step 6 — (Optional) Settings for Claude Code

`.claude/settings.json` configures permissions, hooks, and model preferences per project. See `configuration_reference.md` for the full schema. Common useful pieces:

- **Allowlist common safe commands** so you don't get permission prompts on every `ls` or `git status`
- **Deny dangerous commands** like `rm -rf` and `git push --force`
- **Stop hook** to play a sound or print a session summary when Claude finishes a turn

Refer to your own user-global `~/.claude/settings.json` as a starting point, or check Claude Code's official docs for the permission schema.

## Step 7 — Seed the workflow

Now do these once to bootstrap the workflow into a working state:

1. Author the first 1–3 stories in `stories/` describing what you want to ship
2. Add them to `stories/SEQUENCE_AND_DEPENDENCIES.md` in the appropriate phase tables
3. Author the first 3–5 architecture decisions in `decisions/decisions_and_punch_list.md` (A1, A2, A3, …) — at minimum: branch model, package layout, test approach, commit conventions
4. If your feature has an eval target, sketch the first 10–20 scenarios in `specs/test_scenarios_catalog.md` with stable IDs
5. Push the integration branch to the remote so CI + teammates see it

After this, normal work begins. See `working_with_the_agent.md` for the day-to-day rhythm.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Claude doesn't see the skills | Skills not in `~/.claude/skills/` or wrong filename | Verify with `ls ~/.claude/skills/`; rename to `<skill>/SKILL.md` if needed |
| Claude doesn't pick up CLAUDE.md | File not at repo root | Move to repo root; check that the Claude Code session is launched from the repo root |
| Skills trigger but reference wrong paths | `.claude-toolkit.yaml` missing or paths wrong | Verify the YAML at repo root; ensure `paths.*` keys point to existing directories |
| `dispatching-client-story-agent` errors saying the Agent tool isn't available | Your harness doesn't load the `Agent` tool | Fall back to doing the work in the main session — see `working_with_the_agent.md` §"Working without subagents" |
| Hooks not firing | Hooks not enabled in `.claude/settings.json` | Add the hooks block per `configuration_reference.md` |

## What you've set up

You now have:
- A project that any new Claude session can pick up without ramp-up
- A docs tree that you and your teammates can keep in sync as a side effect of normal work
- Skills that fire at the right moments to keep the discipline
- A CLAUDE.md and `.claude-toolkit.yaml` that propagate your conventions automatically to anyone who clones the repo

Next: read `working_with_the_agent.md` for the day-to-day rhythm.

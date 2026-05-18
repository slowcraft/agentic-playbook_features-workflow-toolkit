# agent-workflow-toolkit (plugin)

The plugin payload inside the marketplace. This README is what `/plugin info agent-workflow-toolkit@agent-workflow-toolkit` shows.

## What this plugin gives you

**Skills** (auto-trigger on intent phrases or explicit invocation):

| Skill | What it does |
|---|---|
| `writing-stories` | Turn investigations / requests into formally-scoped backlog items with description, dependencies, and a numeric-prefix lifecycle. |
| `deep-research-toolkit` | Graduated source-cited research for tool evaluation, comparing approaches, or building context. Scales depth (quick / medium / deep) to the question. |
| `dispatching-client-story-agent` | Produce a self-contained Agent-tool prompt to dispatch a background subagent on an isolated worktree with the right base-commit-fix preamble, scope guards, and conventions. |

More skills land in T.2 — see `CHANGELOG.md` at the marketplace root for the plan.

**Templates** (under `templates/`):

- `claude-toolkit.yaml.template` — project config the skills read from
- `feature-docs/` — the scaffold tree (decisions/, specs/, stories/, experiments/, research/) an adopter copies into their project's `.claude/docs/<feature>/`

## What this plugin does NOT do

- It does NOT modify your `~/.claude/settings.json` automatically
- It does NOT install hooks, CI checks, or any kind of background automation — invariants live inside skill content only

## How to use after install

1. Run `/plugin info agent-workflow-toolkit@agent-workflow-toolkit` to verify install
2. Optionally invoke the (future) `initializing-feature-docs` skill on a new feature to scaffold the `templates/feature-docs/` tree into your project
3. Create a `.claude-toolkit.yaml` at your project root using `templates/claude-toolkit.yaml.template` as the starting point
4. Read the operator manual under `docs/` in the marketplace repo (`adoption-guide.md`, `working-with-the-agent.md`, `configuration-reference.md`)

## Customization

Each skill reads conventions from `.claude-toolkit.yaml` at your project root. The most-touched knobs:

- **`writing-stories`** — drop a markdown file at `<stories_dir>/_template.md` to override the built-in story template, or set `stories.template_file` in the yaml to point anywhere. With nothing configured, the skill infers format from your most recent file in `<stories_dir>/done/`, or asks you once on first invocation.
- **`deep-research-toolkit`** — override depth, source cap, or output directory inline via natural language (*"research X at deep depth"*, *"save research to docs/notes/"*). Defaults: `auto` depth, 50-source cap, `docs/research/` output.
- **`dispatching-client-story-agent`** — reads `dispatch.default_model`, `dispatch.default_isolation`, `dispatch.default_background` from the yaml. Honors `forbidden_paths` and `high_risk_shared_files` to keep parallel agents from colliding.

See `templates/claude-toolkit.yaml.template` for the full schema; each skill's `SKILL.md` documents its full configuration surface.

## License

MIT — see `LICENSE` at the marketplace repo root.

## Issues / contributions

File issues + PRs on the marketplace repo: `slowcraft/agentic-playbook_features-workflow-toolkit`.

# agent-workflow-toolkit

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-plugin-blueviolet.svg)](https://code.claude.com)
[![Status: pre-v1](https://img.shields.io/badge/status-pre--v1-orange.svg)](CHANGELOG.md)

> A structured-artifacts workflow for client-project engineering with Claude Code.

Ships skills, templates, and conventions that turn ad-hoc engineering work into a self-maintaining documentation layer: stable-IDed architecture decisions, scenario catalogs, story sequences, experiment logs, and worktree-isolated parallel-agent dispatch.

## Install

In your `~/.claude/settings.json`, add the marketplace:

```json
{
  "extraKnownMarketplaces": {
    "agent-workflow-toolkit": {
      "source": {
        "source": "github",
        "repo": "slowcraft/agentic-playbook_features-workflow-toolkit"
      }
    }
  }
}
```

Then in a Claude Code session:

```
/plugin install agent-workflow-toolkit@agent-workflow-toolkit
```

Verify with `/plugin info agent-workflow-toolkit@agent-workflow-toolkit` — you should see the three launch skills below.

## What you get on day one

Three skills land at install:

| Skill | Use when |
|---|---|
| `writing-stories` | Turning a request / investigation into a scoped backlog item with description, dependencies, and a numeric-prefix lifecycle |
| `deep-research-toolkit` | Evaluating tools, comparing approaches, or building context — graduated depth, source-cited |
| `dispatching-client-story-agent` | Dispatching a background subagent on an isolated worktree with the right base-commit-fix preamble + conventions |

Plus a `feature-docs/` scaffold (decisions, specs, stories, experiments, research) you can drop into your project's `.claude/docs/<feature>/`.

**Recommended companion (not required):** brainstorming a feature or decision *before* writing a story usually produces better stories. Any brainstorming skill works — for example `superpowers:brainstorming` from the [Superpowers plugin](https://github.com/anthropics/claude-code), or just an unstructured back-and-forth with Claude. The toolkit's skills assume you've already decided *what* to scope; they don't replace the deciding step.

## Why use this

Three concrete pains the workflow prevents:

1. **"Why is this threshold 0.85?"** — Six months from now, you'll want to know. The decisions log + ADR answer it in two minutes. Without them, you re-derive the answer wrong.
2. **"Did we already test this edge case?"** — The scenario catalog with stable IDs answers it in 30 seconds. Without it, you write the test twice.
3. **"What's the current state of the project?"** — `SEQUENCE_AND_DEPENDENCIES.md` answers it. Your teammate picks up where you left off without asking you.

## How it works

The toolkit is opinionated about *one* thing: cross-reference everything by stable identifiers. The decisions log cites the catalog; the catalog cites decisions; stories cite both; eval reports cite catalog rows; experiments cite the decisions they informed. Nothing stands alone.

Skills enforce that discipline at the right moments. Templates scaffold the directory tree. Opt-in hooks and CI checks make invariants self-enforcing.

## Learn more

- [Adoption guide](docs/adoption-guide.md) — cold-start instructions for a new project
- [Working with the agent](docs/working-with-the-agent.md) — day-to-day operator guide
- [Configuration reference](docs/configuration-reference.md) — every setting, env var, and config file

## Roadmap

Phase T.1 (foundation) shipped in v0.1.0: the three skills above + the feature-docs scaffold + manifests. Phase T.2 will add six more skills for everyday discipline maintenance (decisions-log appending, catalog-row appending, sequence-doc updates, ADR writing, experiment-log entries, feature-docs initialization). See [CHANGELOG.md](CHANGELOG.md) for the in-flight plan.

## Compatibility

- Tested with Claude Code on macOS (recent build)
- Marketplace + plugin manifest format per [code.claude.com/docs/en/plugin-marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
- Linux + Windows expected to work; not yet verified

**Standalone:** All three skills are self-contained. They depend only on Claude Code itself, an optional `.claude-toolkit.yaml` in the consuming project, and (for `dispatching-client-story-agent`) the host harness's `Agent` tool. Nothing in the plugin reaches outside the marketplace install.

**Coexistence with other plugins or user-level skills:** Skill names are namespaced by plugin (`agent-workflow-toolkit:deep-research-toolkit`, etc.), so they do not shadow or collide with similarly-themed skills installed elsewhere. No SessionStart hooks, no settings.json mutations, no auto-installed CI ship with the plugin — invariants live inside skill content only.

**Using outside Claude Code:** Codex, Cursor, Aider, and other coding agents can apply the same workflow without the plugin install. See [AGENTS.md](AGENTS.md) at the repo root — it's the agent-readable shim that translates Claude Code tool names, points at the canonical `SKILL.md` files, and notes which primitives (notably subagent dispatch) don't translate.

## License

MIT — see [LICENSE](LICENSE).

## Contributing

Issues and PRs welcome on the marketplace repo. See [CONTRIBUTING.md](CONTRIBUTING.md) (drafted at publish time).

## Acknowledgements

The workflow pattern was developed during one mid-sized engineering effort and generalised afterward. The cross-reference discipline is its defining feature.

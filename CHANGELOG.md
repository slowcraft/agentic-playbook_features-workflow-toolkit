# Changelog

All notable changes to `agent-workflow-toolkit`.

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versions follow [Semantic Versioning](https://semver.org/) once v1 ships; pre-v1 versions may break compatibility within the 0.x series.

## [Unreleased]

## [0.1.1]

### Added

- `AGENTS.md` at the repo root — instruction surface for non-Claude-Code coding agents (Codex, Cursor, Aider, Cline, Continue, Gemini CLI, etc.). Tells each agent to read the `SKILL.md` files directly, translates Claude Code tool names (`WebSearch`, `WebFetch`, `Agent`, MCP-prefixed) to host-equivalent guidance, and explicitly carves Claude Code out (which uses the plugin install path instead). The file is the portability shim; the `SKILL.md` files remain canonical.

### Planned (T.2)

- New skill: `reading-toolkit-config` (library — resolves `.claude-toolkit.yaml`)
- New skill: `maintaining-the-decisions-log` (auto-triggers on "we decided", "from now on", "the rule is")
- New skill: `appending-to-the-scenario-catalog` (auto-triggers on "what if", "edge case")
- New skill: `maintaining-the-sequence-doc` (auto-triggers on story-status-change phrases)
- New skill: `initializing-feature-docs` (scaffolds `.claude/docs/<feature>/` from templates)
- New skill: `writing-an-experiment-log-entry` (auto-triggers on threshold / library-choice phrases)
- New skill: `writing-an-adr` (auto-triggers from `maintaining-the-decisions-log` when alternatives mentioned)

### Deferred post-v1

- MCP servers: `toolkit-ticket-bridge`, `toolkit-eval-history`

## [0.1.0] - Initial release

### Added

- 3 portable skills as the plugin payload:
  - `writing-stories` — turn investigations into formally-scoped backlog items with story lifecycle. Supports a 4-tier template-resolution ladder (config field → `<stories_dir>/_template.md` → most recent file in `done/` → built-in default) and a one-shot first-run prompt when no signals exist. The resolution is bounded to the current project root — the skill never walks the filesystem looking for `stories/` directories elsewhere.
  - `deep-research-toolkit` — graduated source-cited research that scales depth (quick / medium / deep) to the question. Phase 2 mandates external grounding via `WebSearch` / `WebFetch` (always available) plus optional `firecrawl:`, `mcp__context7__*`, `gh api` when present; stops and surfaces an error if no web-capable tools are available rather than answering from training data.
  - `dispatching-client-story-agent` — generate Agent-tool dispatch prompts with base-commit-fix preamble, scope guards, conventions
- Marketplace manifest at `.claude-plugin/marketplace.json`
- Plugin manifest at `plugins/agent-workflow-toolkit/.claude-plugin/plugin.json` (auto-versioned by commit SHA during pre-v1)
- Templates for adopters:
  - `claude-toolkit.yaml.template` — per-project config schema including the optional `stories.template_file` knob
  - `feature-docs/` — scaffold tree (decisions/, specs/, stories/, experiments/, research/) including `stories/_template.md` mirroring the built-in story default
- Operator manual under `docs/`:
  - `adoption-guide.md` — cold-start instructions
  - `working-with-the-agent.md` — day-to-day operator guide
  - `configuration-reference.md` — every setting, env var, config file
- Plugin README's "Customization" section lists the per-skill knobs visible to anyone running `/plugin info`
- Repo security baseline: pre-commit hooks (detect-secrets, gitleaks, JSON/YAML validation, large-files, whitespace, shellcheck), `.secrets.baseline`, `gitleaks.toml`, GitHub Actions CI with SHA-pinned actions, branch protection on `main` (PR + status checks + force-push block + tag protection ruleset)

### Known limitations

- T.2 skills not yet shipped
- Trigger-accuracy of the 3 ported skills is the original implementation's accuracy — formal `skill-creator` audit recommended before broad adoption

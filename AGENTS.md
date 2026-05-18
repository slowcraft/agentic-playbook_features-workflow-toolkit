# AGENTS.md

This file is for coding agents that are **not** Claude Code.

> **If you are Claude Code: stop reading this file.** Install the marketplace plugin instead — see `README.md`. Do not apply this file's instructions; the plugin install path supersedes it.

For every other agent (Codex, Cursor, Aider, Cline, Continue, Gemini CLI, Codeium, or anything else landing in this repository): this file tells you how to apply the toolkit's workflow to yourself without the Claude Code plugin install path.

## What this toolkit provides

Three workflow disciplines for engineering work with a coding agent. The source of truth for each is its `SKILL.md` file:

| Discipline | Source of truth |
|---|---|
| **Writing stories** — scope ad-hoc requests, investigations, or tickets into trackable backlog items with description, dependencies, and a numeric-prefix lifecycle | `plugins/agent-workflow-toolkit/skills/writing-stories/SKILL.md` |
| **Deep research** — graduated source-cited research that scales depth (quick / medium / deep) to the question and never answers from training data | `plugins/agent-workflow-toolkit/skills/deep-research-toolkit/SKILL.md` |
| **Parallel-agent dispatch** — generate ready-to-execute prompts for subagent batches on isolated worktrees (base-commit-fix preamble + scope guards + project conventions bundled inline) | `plugins/agent-workflow-toolkit/skills/dispatching-client-story-agent/SKILL.md` |

Each `SKILL.md` has YAML frontmatter (`name`, `description`) followed by a markdown body. The frontmatter is Claude Code's auto-trigger metadata — ignore it. The body is the actual instruction set you apply.

## How to self-configure

### 1. Treat the `SKILL.md` bodies as your instructions

When the user asks for any of the disciplines listed above, read the relevant `SKILL.md` body and follow it as if it were a personal directive. No transformation required — the bodies are written in plain English directives, not Claude-specific code.

### 2. Translate Claude Code tool names to your host's equivalents

The skills reference tools by Claude Code's identifiers. Translate before applying. If no equivalent exists in your host, surface that to the user rather than silently degrading.

| Reference in `SKILL.md` | Translation |
|---|---|
| `WebSearch`, `WebFetch` | Your host's web search and URL-fetch tools (Bing, Google, DuckDuckGo, native fetch, etc.) |
| `firecrawl:firecrawl-search`, `firecrawl:firecrawl-scrape` | Skip if Firecrawl isn't available in your host; the skill's resolution ladder treats them as optional |
| `mcp__context7__resolve-library-id`, `mcp__context7__query-docs` | Skip if the Context7 MCP server isn't connected to your host |
| `gh api` / `gh` | Standard GitHub CLI — install if missing, or skip GitHub-specific steps |
| **`Agent` tool** (subagent dispatch with worktree isolation, `run_in_background: true`) | **No equivalent in Codex, Cursor, Aider, Cline, or Continue at the time of writing.** Treat `dispatching-client-story-agent` as a prompt-template generator only: produce the prompt and either paste into a second session manually or execute the story sequentially in the main loop. Do not silently pretend you dispatched something you didn't. |

### 3. Honor `.claude-toolkit.yaml` if present at the project root

The project root may contain a `.claude-toolkit.yaml` declaring paths, commit format, branch model, forbidden paths, and other conventions. The schema lives in `plugins/agent-workflow-toolkit/templates/claude-toolkit.yaml.template`.

When the yaml exists, read it **before** invoking any skill — its fields override defaults documented in the `SKILL.md` files. The `paths.stories_dir`, `stories.template_file`, `commits.format`, `branches.*`, and `forbidden_paths` fields in particular shape how the skills behave.

### 4. Honor the `feature-docs/` scaffold if present

If the project's `.claude/docs/<feature>/` (or wherever the yaml's `paths` point) follows the scaffold layout — `decisions/`, `stories/`, `specs/`, `experiments/`, `research/` — read `plugins/agent-workflow-toolkit/templates/feature-docs/README.md` for the conventions, then respect them when writing artifacts into those directories.

In particular: the `writing-stories` skill resolves its template via a 4-tier ladder (`stories.template_file` config field → `<stories_dir>/_template.md` → most recent file in `<stories_dir>/done/` → built-in default). That ladder is project-bounded — never walk outside the project root looking for a `stories/` folder.

## Authority and limits

- When this file disagrees with a `SKILL.md` body, **the `SKILL.md` wins**. It is the canonical instruction; this file is a portability shim.
- When a `SKILL.md` references a Claude-Code-specific tool you don't have, **surface the limitation to the user** instead of silently substituting something close-enough. The dispatching skill is the most common case — most non-Claude-Code hosts cannot dispatch isolated-worktree subagents.
- This file is licensed identically to the rest of the repo (MIT — see `LICENSE`).

# Configuration reference

Every setting, env var, and config file the toolkit uses. Useful when setting up a new project, debugging why something isn't firing, or adapting the workflow to a different language / build tool.

## File locations

| File | Location | Tracked in git? | Purpose |
|---|---|---|---|
| `CLAUDE.md` | Repo root | Yes | Auto-loaded at session start. Tells Claude where the workflow docs live + project conventions |
| `.claude-toolkit.yaml` | Repo root | Yes | Project conventions consumed by all toolkit skills |
| `.claude/settings.json` | Repo root | Optional (typically gitignored) | Per-project Claude Code permissions, hooks, model preferences |
| `.claude/settings.local.json` | Repo root | Always gitignored | Per-developer overrides; never commit |
| `~/.claude/settings.json` | User home | Per-developer | User-global Claude Code config; affects all projects |
| `~/.claude/skills/<name>/SKILL.md` | User home | Per-developer | User-global portable skills; affects all projects |
| `.claude/docs/<feature>/` | Repo root | Typically gitignored (per developer) | Private design docs — decisions, stories, specs, experiments |

## `.claude-toolkit.yaml` schema

The project config file. Read by every toolkit skill. Commit it — no secrets.

```yaml
project:
  name: <repo-name>                    # human-readable project name
  feature: <feature-slug>              # the feature this config governs (one feature per repo for now)
  language: python | typescript | go | rust | ...
  package_manager: uv | poetry | pip | pnpm | npm | yarn | cargo | go-mod | ...
  test_runner: pytest | vitest | jest | go-test | cargo-test | ...
  test_command: "<full command>"       # e.g., "pytest", "pnpm test", or "MY_FLAG=False pytest" if your project needs env flags
  test_marker_for_eval: <marker>       # optional — pytest marker for the slow eval suite
  lint_command: "<full command>"       # optional
  type_check_command: "<full command>" # optional
  env_prep: |
    <multi-line block that activates the env / cds into the right dir>

commits:
  format: conventional-commits | gitmoji | semantic-release | free
  co_author_trailer: false             # default false — no `Co-Authored-By: Claude` etc.
  body_style: short | long
  ticket_patterns:                     # how to reference tickets in commit messages
    - "[#<id>]"                        # ADO style
    - "[story:<slug>]"                 # internal stories without external ticket

branches:
  model: single-tier | two-tier | trunk
  # If two-tier:
  foundation_branch: feature/<feature>-mvp
  integration_branch: feature/<feature>-dev
  # If single-tier:
  # integration_branch: feature/<feature>
  story_prefix: story/

paths:
  decisions_dir: .claude/docs/<feature>/decisions/
  decisions_table_file: decisions_and_punch_list.md
  adrs_dir: .claude/docs/<feature>/decisions/adrs/
  specs_dir: .claude/docs/<feature>/specs/
  scenario_catalog_file: specs/test_scenarios_catalog.md
  stories_dir: .claude/docs/<feature>/stories/
  stories_done_dir: .claude/docs/<feature>/stories/done/
  stories_future_dir: .claude/docs/<feature>/stories/future/
  sequence_file: stories/SEQUENCE_AND_DEPENDENCIES.md
  experiments_dir: .claude/docs/<feature>/experiments/
  research_dir: .claude/docs/<feature>/research/
  fixtures_dir: <project-specific>
  module_root: <project-specific>

forbidden_paths:                       # paths agents must never touch
  - .claude/                           # always include this
  - <other paths — UI, other teams' modules, etc.>

high_risk_shared_files:                # files where multiple parallel agents collide
  - <package manifest>
  - <lockfile>
  - <shared test config>

stable_id_patterns:                    # regex for stable IDs the toolkit enforces
  decision: '^A\d+'
  catalog_row:
    <criterion>: '^<PREFIX>-\d+'
  adr: '^ADR-\d{4}'
  story_done: '^\d{4}_'

dispatch:                              # defaults for subagent dispatch (when Agent tool is available)
  default_model: opus
  default_isolation: worktree
  default_background: true

automation:                            # which automation surfaces are enabled
  skills:
    auto_trigger_on_decision_phrases: true
    auto_trigger_on_catalog_phrases: true
    auto_trigger_on_experiment_phrases: true
  ci:
    enforce_stable_ids: true
    enforce_conventional_commits: true
    enforce_sequence_doc_invariants: true
```

## `.claude/settings.json` schema (Claude Code config)

Per-project Claude Code config. Format documented at the official Claude Code docs; common-useful keys below.

```json
{
  "env": {
    "ENABLE_LSP_TOOL": "1"
  },
  "permissions": {
    "allow": [
      "Bash(ls:*)",
      "Bash(pwd)",
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Write(**)",
      "Edit(**)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(git push --force:*)",
      "Bash(git commit --no-verify:*)",
      "Read(**/.env)",
      "Read(**/jwt.txt)",
      "Edit(**/.claude/settings*.json)",
      "Write(**/.claude/settings*.json)"
    ],
    "ask": [
      "Bash(<long-running server commands>):*"
    ]
  },
  "model": "opus[1m]",
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "printf '\\a' > /dev/tty 2>/dev/null || true"
          }
        ]
      }
    ]
  },
  "enabledPlugins": {
    "<plugin>@<marketplace>": true
  }
}
```

### Permission allow/deny patterns

- **`Bash(<command>:*)`** — allow / deny a shell command and any of its arguments
- **`Bash(<command>)`** — allow / deny exactly that command, no args
- **`Write(<glob>)`** — allow / deny writing to files matching the glob
- **`Edit(<glob>)`** — same for editing
- **`Read(<glob>)`** — same for reading
- **`Skill(<name>)`** — control which skills can be invoked

Deny rules take precedence over allow rules. The skills `update-config` and `fewer-permission-prompts` are typically denied at the user-global level to prevent the agent from quietly editing your permission config.

### Hooks

Hooks run shell commands at specific lifecycle events. Common events:

| Event | Fires when | Typical use |
|---|---|---|
| `SessionStart` | New Claude Code session opens | Remind about open stories; surface recent decisions |
| `Stop` | Claude finishes a turn | Audible bell; write session summary |
| `PreToolUse` | Before any tool call (filterable by `matcher`) | Validate the call (e.g., "is this story file referenced in the sequence doc?") |
| `PostToolUse` | After any tool call (filterable by `matcher`) | React to the result (e.g., "git commit referenced a story — should we move it to done/?") |

Each hook is a shell command. The exit code controls whether the action proceeds. See your reference project's `settings.json` for a real example.

## Environment variables

| Variable | Used by | Purpose |
|---|---|---|
| `PYTHONPATH` | Python projects | Required to be set before running tests in many Python projects (where the test command can't import the source tree by default) |
| `<PROJECT_AUTH_FLAG>` | Project-specific | Many projects expose a flag like `ENABLE_AUTH=False` to skip auth in local dev — capture yours in `.claude-toolkit.yaml`'s `env_prep` or `test_command` |
| `ANTHROPIC_API_KEY` | Claude Code itself | Required for API access; typically set in shell profile |
| `CLAUDE_CODE_CONFIG` | Claude Code | Override config location; rarely needed |
| `TOOLKIT_HOME` | Hooks (if used) | Path to the toolkit repo; lets hooks reference `${TOOLKIT_HOME}/hooks/<script>.py` |

## CLAUDE.md (auto-loaded by Claude Code)

The `CLAUDE.md` at the repo root is loaded at session start and included in Claude's system prompt. Keep it short — every line is a token Claude reads on every turn.

Recommended structure:

```markdown
# <Project name>

## What this is
<1–2 sentence project description>

## Workflow docs
<bulleted list of key files Claude should read on first turn>

## Conventions
<bulleted: commits, branches, tests, anything else essential>

## What's NOT obvious from the docs
<gotchas: env vars that must be set, tests that require flags, paths to avoid>

## When starting a new session
<bulleted: where to look first, what to ask the user>
```

A user-global `~/.claude/CLAUDE.md` is also loaded — that's the right place for personal conventions (e.g., "never add a Co-Authored-By trailer to commits") that apply to ALL projects.

## Memory (auto-loaded by Claude Code)

`~/.claude/projects/<project-hash>/memory/MEMORY.md` (or per-project) is a developer-private auto-memory file. Use it for preferences and corrections that should persist across sessions on the same project. Claude updates it when you give explicit "remember this" guidance.

## Toolkit + project: which config goes where?

Quick decision matrix:

| Setting | Where to put it |
|---|---|
| Your personal commit-message preference | `~/.claude/CLAUDE.md` |
| Your project's commit format (e.g., Conventional Commits) | `.claude-toolkit.yaml` + `CLAUDE.md` at repo root |
| Skills that all your projects benefit from | `~/.claude/skills/` |
| Project-specific skills | `<repo>/.claude/skills/` (gitignored or committed depending on team) |
| Permission rules that apply everywhere | `~/.claude/settings.json` |
| Permission rules specific to this project | `.claude/settings.json` (consider gitignoring) |
| Hooks that all your projects benefit from | `~/.claude/settings.json` |
| Hooks specific to this project | `.claude/settings.json` |
| Private design docs (decisions, stories, etc.) | `.claude/docs/<feature>/` (gitignored) |
| Public design docs (architecture diagrams for the team) | `docs/` or `docs/<feature>/` (committed) |

## Validation: is your config correct?

Quick self-check after setup:

```
You: tell me what conventions you'll follow on this project
```

Claude should respond with: branch model, commit format, test command, paths to never touch, and where to find decisions / stories / specs. If any of that is wrong or missing, fix the corresponding config file.

```
You: show me the open stories in this project
```

Claude should list the files under `stories/` (excluding `done/` and `future/`) by reading the directory directly or by parsing the sequence doc. If it can't find them, your path config is wrong.

```
You: list the available skills
```

Should include the four portable skills + any project-specific skills. If a skill is missing, check `~/.claude/skills/` and the skill's `SKILL.md` frontmatter.

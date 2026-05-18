# Contributing

Thanks for your interest in `agent-workflow-toolkit`.

## Local setup

Install [pre-commit](https://pre-commit.com/) once per machine (`pipx install pre-commit`, `brew install pre-commit`, or `uv tool install pre-commit`) plus the scanners the config invokes:

```bash
pipx install detect-secrets      # or: brew install detect-secrets
brew install gitleaks
brew install shellcheck
```

Then in your clone:

```bash
pre-commit install                    # installs the pre-commit + commit-msg + pre-push hooks
pre-commit run --all-files            # run once to verify everything passes
```

CI runs the same hooks on every PR; failing locally is faster than failing in CI. If `detect-secrets` flags a *legitimate* placeholder, regenerate the baseline (`detect-secrets scan > .secrets.baseline`) in the same PR rather than disabling the hook.

## Issues

Use the issue templates under `.github/ISSUE_TEMPLATE/`:
- Bug reports
- Feature requests
- New skill requests

Include your Claude Code version (`/version`), OS, and plugin commit SHA (`/plugin info agent-workflow-toolkit@agent-workflow-toolkit`). Redact secrets before pasting settings or logs.

## Pull requests

- Branch from `main`, open a PR, no direct commits to `main`
- Conventional commit format (`feat:`, `fix:`, `docs:`, `chore:` …)
- For new or changed skills: run a trigger-accuracy check via `skill-creator` before merge
- Add a `CHANGELOG.md` entry under `[Unreleased]`

## Adding a skill

- Write `SKILL.md` at `plugins/agent-workflow-toolkit/skills/<name>/SKILL.md`
- Add the skill to the table in `README.md` and `plugins/agent-workflow-toolkit/README.md`
- Add a CHANGELOG entry
- Validate auto-trigger accuracy (`skill-creator`)

## Security

Do not open a public issue for security findings. Use GitHub's [private vulnerability reporting](https://github.com/slowcraft/agentic-playbook_features-workflow-toolkit/security/advisories/new) — it's enabled on this repo. Use a non-security issue only as a placeholder reference if you need one.

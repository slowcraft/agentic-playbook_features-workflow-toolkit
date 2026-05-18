---
name: New skill request
about: Propose a new skill for the agent-workflow-toolkit plugin
title: '[skill] '
labels: enhancement, skill
assignees: ''
---

## Skill name (kebab-case)

<e.g., `tracking-cross-team-dependencies`>

## What discipline does this enforce?

<What recurring workflow pattern would this skill capture? Why isn't it already covered by an existing skill?>

## Trigger phrases (auto-trigger or explicit?)

- Auto-trigger / explicit-only: <choose one>
- Example phrases that should trigger the skill:
  - "<phrase 1>"
  - "<phrase 2>"
  - "<phrase 3>"
- Example phrases that should NOT trigger (false-positive guard):
  - "<counter-phrase 1>"
  - "<counter-phrase 2>"

## What the skill does

<Describe the skill's behaviour. What does it read? What does it write? Does it confirm before writing?>

## Dependencies

<Which other skills, config fields, or files does this skill depend on?>
- `.claude-toolkit.yaml` fields read: <list>
- Other skills called: <list, if any>

## Where it fits

<Which phase / category: T.2 planned, new category, replacement for existing skill?>

## Acceptance criteria

- [ ] Trigger accuracy ≥ 85% on the example phrases (validated via `skill-creator`)
- [ ] False-positive rate < 5% on the counter-phrases (validated via `skill-creator`)
- [ ] SKILL.md ships at `plugins/agent-workflow-toolkit/skills/<name>/SKILL.md`
- [ ] CHANGELOG entry under [Unreleased]

## Related

<Existing skills this complements or overlaps with>

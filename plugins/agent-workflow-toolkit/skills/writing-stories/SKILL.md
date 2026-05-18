---
name: writing-stories
description: Use when scoping work into a formal backlog item with description and dependencies. Triggers on "create a story", "write a story", "make a ticket", "scope this work", "file this as a story", "turn this into a backlog item", or whenever an investigation, Slack thread, or feature request needs to become a tracked unit of work. Apply this skill whenever ad-hoc work needs to become a tracked entry — even without the word "story" — because it enforces active-verb titles and why-not-how summaries that prevent shallow tickets.
---

# Writing Stories

## Overview

Turn investigations or feature requests into backlog items scoped at a what-and-why level — never implementation detail. Each story carries: a why-paragraph, scoped what-bullets, and any dependencies — other stories or tasks, teams, systems, services, decisions, deliverables, or anything else that blocks or shapes the work.

## When to Use

- Triggered explicitly by "create a story" / "write a story" / "make a ticket"
- When work has been described but not yet scoped formally
- When an investigation has identified work that needs to land in the backlog
- When a Slack thread or incident needs a corresponding ticket

**Do NOT use** for: speculative ideas not yet ready for the backlog, internal-only design notes (those belong in spec/design docs), or implementation plans (those go in build specs, not stories).

## Template resolution

**`<stories_dir>` is resolved only from explicit signals — never inferred by searching the filesystem.** It is:

- `paths.stories_dir` if set in `.claude-toolkit.yaml` (resolved relative to the project root containing that file), OR
- the literal path `.claude/docs/<feature>/stories/` if that path exists in the current project root

If neither yields a path inside the current project, treat `<stories_dir>` as unresolved and skip to the first-run prompt below. **Never walk upward, sideways, or into sibling directories looking for a `stories/` folder.**

On invocation, resolve the story template in this order. Stop at the first one found.

1. **Explicit config** — `stories.template_file` in `.claude-toolkit.yaml` (if set). Read that file verbatim.
2. **Convention** — `<stories_dir>/_template.md` (only if `<stories_dir>` was resolved per the rules above). Read it verbatim.
3. **Inferred from example** — most recent file in `<stories_dir>/done/`, else most recent file in `<stories_dir>/` (only if `<stories_dir>` is resolved). Match its section headings, field order, and level of detail.
4. **Built-in default** — the template shown below. Used when `<stories_dir>` is resolved but contains no template or existing stories.

Always announce which path resolved (e.g., *"Using template from `.claude/docs/auth/stories/_template.md`"* or *"Inferring format from `stories/done/0023_repair_oauth.md`"*) so the operator knows what's driving the structure.

When step 4 (built-in default) fires, the announcement must also surface how to override:

> Using built-in default story template. To customize: drop a markdown file at `<stories_dir>/_template.md`, or set `stories.template_file` in `.claude-toolkit.yaml`.

This is the discovery moment for operators who configured everything else but never touched the template knob — never let the built-in default fire silently.

### First-run prompt

When `<stories_dir>` cannot be resolved AND no `stories.template_file` is configured — i.e., the toolkit has not been adopted in this project — ask **once**, before writing the first story:

> No story template found for this project. How do you want stories structured?
>
> 1. **Use the built-in default** — produces a story right now using the format shown below. Recommended for new projects.
> 2. **I'll write `_template.md` first** — pause; I'll create a blank `<stories_dir>/_template.md` you can fill in, then re-invoke me.
> 3. **Follow an existing file as a reference** — paste a path. I'll match its shape.
> 4. **Paste the structure inline** — give me the headings and rules you want; I'll use them and offer to save them as `_template.md`.

After the answer:

- Option 1 → write the story; offer to copy the built-in template to `<stories_dir>/_template.md` so future sessions skip step 4.
- Option 2 → write a stub `_template.md` with the built-in default sections + comments explaining each one; tell the operator to edit and re-invoke.
- Option 3 → read the referenced file, treat it as the template for this invocation, offer to copy it to `_template.md`.
- Option 4 → use the inline structure for this invocation, offer to save it as `_template.md`.

Subsequent invocations follow the resolution ladder above and do not re-prompt.

## Built-in default template

Used as the fallback in step 4 of the resolution ladder above. Shipped as `templates/feature-docs/stories/_template.md` in this plugin's scaffold so adopters can drop it directly into their project.

```markdown
# <Title — active verb + specific scope>

<1–2 sentence summary that captures the why: root cause, symptoms,
evidence, or business driver that triggered this work.>

## Description of Work
- <3–5 actionable bullets — what is being done, not how>
- <No file paths, function names, code snippets, or schema fields>

## Dependencies
- <Anything this story depends on — other stories or tasks, teams, systems, external services, prerequisite decisions, third-party deliverables, or other blocking items. Note what's needed from each.>
- <Omit the section if there are none>
```

## Rules

| Field | Rule |
|---|---|
| **Title** | Active verb + specific scope. Avoid vague nouns like "issue" or "thing." |
| **Summary** | The *why*, not the *what*. 1–2 sentences. Anchor on the trigger: incident, business driver, prior decision, identified gap. |
| **Description of Work** | The *what*, not the *how*. 3–5 bullets. Each bullet describes a unit of work in plain language. No file paths, function names, schema fields, or code. |
| **Dependencies** | Anything this story is blocked on or coupled to: other stories or tasks, teams, systems, external services, prerequisite decisions, third-party deliverables, or anything else. Note what's needed from each. Omit the section if none. |

## Anti-Patterns

- **Implementation detail in Description of Work.** File paths, function names, class names, code snippets, schema fields — these belong in build specs, not stories. If a bullet would require knowing the codebase to understand it, rewrite it.
- **Vague bullets** like *"Look into the issue"* or *"Fix the bug"* — be specific about what is being delivered, even when "how" is unknown.
- **Title that just renames a problem** — *"OCR timeout"* describes a symptom; *"Investigate timeout of OCR upload path and recommend remediation"* describes work.

## Quick Reference Checklist

Before submitting a story, verify:

- [ ] Title is active-verb + scope
- [ ] Summary paragraph explains the *why* in 1–2 sentences
- [ ] Description of Work is 3–5 bullets, no implementation detail
- [ ] Dependencies section present, or omitted because there are none

## Story lifecycle (after the work lands)

When a story's implementation lands and tests pass:

1. **Rename** the story file with the next available numeric prefix: `stories/done/<NNNN>_<slug>.md`. The counter increments globally across `done/`.
2. **Move** into `stories/done/`.
3. **Update** the project's sequence/dependency index (e.g. `SEQUENCE_AND_DEPENDENCIES.md`) — add the entry to the Done table; remove from the open phase tables.
4. **Reference the story** in every commit that implemented it using the project's commit format (e.g., Conventional Commits with `feat(<scope>): description [#<ticket-id>]`, or `feat(<scope>): description [story:<slug>]` when no external ticket id exists). The exact format comes from `.claude-toolkit.yaml`.

## Bugs as stories

When a bug surfaces during implementation or testing:

1. **Write** a new story for the bug using the same template (title = active verb describing the fix, summary = the failure mode + how it was found, Description of Work = what produces the observable signal that the bug is gone).
2. **Implement** the fix on a branch named after the bug story.
3. **Close** the bug story through the same lifecycle above (rename with next `NNNN_` prefix, move to `done/`).

This keeps every fix paper-trailed without separate bug-tracker tooling.

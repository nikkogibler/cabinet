# Contributing

Cabinet is intentionally small. New additions should make the library sharper, not broader.

## What Belongs Here

- Reusable skills that solve repeatable operational problems.
- Guidance that can travel across repos, teams, and projects.
- Skills with clear inputs, explicit workflow steps, and verifiable outcomes.
- Supporting references that materially improve a skill's reliability.

## What Does Not Belong Here

- Client-specific or company-specific instructions.
- Skills that depend on private paths, private task systems, or private naming schemes.
- Broad prompt dumps without a clear operational use case.
- Redundant variants of an existing skill unless the new version materially changes the workflow.

## Before Opening A Change

- Keep the scope narrow.
- Preserve the existing tone: compact, practical, and calm.
- Prefer Markdown-first assets over extra tooling.
- Keep dependencies local to the skill folder whenever possible.
- If a skill needs companion references, store them next to the skill.
- Verify examples, paths, and file references before submitting.

## Expected Skill Shape

Each skill should usually include:

- `SKILL.md` as the entry point.
- Clear frontmatter with `name`, `description`, and other stable metadata.
- A concise statement of when to use the skill.
- A workflow that is specific enough to execute, not just describe.
- Verification guidance when the skill can make claims about state.
- Local `references/` files only when the skill genuinely depends on them.

## Review Bar

A change is ready when:

- The skill is portable and not tied to one private project.
- The workflow is concrete and testable.
- The prose is clean and free of filler.
- The file layout is coherent.
- The README and docs still reflect the repo accurately.

## Pull Requests

Use small pull requests with a short explanation of:

- what changed
- why it belongs in Cabinet
- any compatibility or migration notes

If your change adds a new skill, update the roadmap or README when the public capability map changes.

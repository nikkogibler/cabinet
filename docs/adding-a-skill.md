# Adding a Skill

Cabinet should grow slowly. A new skill should earn its place by being portable, concrete, and clearly better than a vague prompt block.

## Standard Folder Shape

Most skills should fit this structure:

```text
skills/
└── your-skill-name/
    ├── SKILL.md
    └── references/
        └── optional-supporting-files.md
```

Use a `references/` directory only when the skill genuinely depends on companion material. If the skill works fine without extra files, keep it to `SKILL.md`.

## Frontmatter Standard

Cabinet uses compact frontmatter so skills are readable both by humans and by tooling:

```yaml
---
name: your-skill-name
description: One sentence explaining what the skill does and when to use it.
version: 1.0.0
user-invocable: true
argument-hint: "[optional input]"
license: MIT
metadata:
  author: Nik Gibler
  copyright: Copyright (c) 2026 Nik Gibler
---
```

If a field does not make sense for a specific skill, omit it deliberately rather than leaving placeholder noise behind.

## Writing Standard

A Cabinet skill should usually answer these questions clearly:

- What is this skill for?
- When should someone use it?
- What outcome should they expect?
- What steps does the skill follow?
- What should be verified before making strong claims?

Useful traits:

- short, direct prose
- reusable instructions
- explicit defaults
- clear escalation boundaries
- honest handling of unknowns

## Admission Rule For Top-Level Skills

A new top-level skill should clear all of these bars before it lands in Cabinet:

- it solves a narrow, repeatable operator problem
- it is concrete enough to execute without private routing or hidden context
- it declares platform assumptions plainly when they matter
- it declares meaningful local side effects and safety boundaries up front
- it improves Cabinet more than it broadens it

If a skill cannot clear those bars, it probably needs more work or it belongs somewhere else.

## Platform And Side-Effect Rules

Platform-specific skills are allowed when the platform is essential to the method rather than incidental to one person's setup.

When a skill mutates files, writes helper code, or runs local automation, it should:

- say so near the top of the skill
- make the side effects explicit before execution
- prefer generated helper files over committed executables unless the repo intentionally changes shape
- give the operator a clear boundary for when not to use the skill

Avoid:

- client-specific paths
- brand-specific rules
- vague motivational filler
- hidden dependencies on private systems
- broad instructions that do not tell the operator what to do next

## Dependency Rules

If a skill depends on companion files:

- keep them inside the skill folder
- reference them with stable relative paths
- make sure the skill still reads coherently without opening every file immediately
- keep the reference set as small as possible

## Review Checklist

Before adding a skill, check the following:

- The skill is useful outside one private project.
- The name is clear and stable.
- The workflow is concrete enough to execute.
- Any examples or commands are generic or clearly scoped.
- All referenced files exist.
- The README still matches the real capability map.
- The roadmap is updated if the launch surface changed.

## Recommendation

Start from `skills/_template/SKILL.md`, then cut rather than add. Cabinet is better when a skill does one job cleanly.

---
name: your-skill-name
description: One sentence explaining what the skill does and when it should be used.
version: 1.0.0
user-invocable: true
argument-hint: "[optional input]"
license: MIT
metadata:
  author: Nik Gibler
  copyright: Copyright (c) 2026 Nik Gibler
---

# Skill Title

State the job of the skill in one or two direct sentences.

## When To Use

Use this skill when the user wants a repeatable operational outcome, not just a vague style of response.

Examples:

- example trigger one
- example trigger two
- example trigger three

## Core Outcome

Describe what should be true when the skill finishes.

## Default Rules

List the rules that keep the skill safe, portable, and predictable.

- prefer existing repo conventions
- verify important claims before acting on them
- keep defaults conservative
- escalate only when necessary

## Workflow

Write the workflow as ordered steps.

1. Identify the target or scope.
2. Gather only the context needed to act.
3. Perform the core workflow.
4. Validate the result.
5. Report the final state clearly.

## Verification

State what must be checked before the skill makes strong claims.

- file exists
- referenced state is current
- output matches the requested artifact or behavior

## Output Standard

Describe what a good final result should contain.

## Notes

If the skill depends on companion files, place them in `references/` next to this file and reference them with stable relative paths.

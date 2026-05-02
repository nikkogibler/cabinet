---
name: repo-audit
description: Systematically assess whether an open-source repository is safe to read, clone, run in a sandbox, or trust with real data. Produces a weighted risk score, verdict ladder, red flags, and concise safe-usage guidance.
version: 1.0.0
user-invocable: true
argument-hint: "[repo URL or local path]"
license: MIT
metadata:
  author: Nik Gibler
  copyright: Copyright (c) 2026 Nik Gibler
---

Audit a repository for operational safety and trust.

This skill is not trying to prove absolute safety. Its job is to determine blast radius, surface hidden risk, and tell the user what they can safely do now.

Read these references when the audit is non-trivial:
- `references/mental-model.md`
- `references/scoring-rubric.md`
- `references/default-workflow.md`
- `references/report-template.md`

## Core Goal

Every audit must answer these four questions:
- Is it safe to read?
- Is it safe to clone?
- Is it safe to run in a sandbox?
- Is it safe to trust with real data?

Every audit must also produce:
- an overall `0-10` risk score
- weighted subscores across multiple dimensions
- a concise plain-language explanation of what is safe, what is sandbox-only, and what is not recommended

## Default Mode

Default to **safe local validation**:
- Always start with static analysis.
- Only run low-risk local validation when it does not require secrets, elevated privileges, package installation, or obvious network activity.
- Treat package installs, browser automation, Docker builds, long-running services, network tracing, or anything credentialed as explicit escalation steps.

## Mandatory Preparation

1. Identify the target:
   - remote repo URL
   - local path
   - current workspace
2. If the target is remote and not already local, clone with `--depth=1` into a temporary directory.
3. Before any execution, inspect manifests, scripts, workflows, and entrypoints.
4. Do not use user secrets, browser profiles, private documents, wallets, SSH keys, or real production credentials.
5. State important assumptions and unknowns.

## Audit Workflow

### Phase 1: Classify the Surface

Quickly identify:
- languages and package managers
- entrypoints and executable surfaces
- CI/workflow files
- Dockerfiles and shell scripts
- bundled binaries, large blobs, generated code, or vendored assets

The goal is to find the likely control points, not to map the whole repo.

### Phase 2: Static Risk Scan

Inspect the highest-risk surfaces first:
- install hooks and package scripts
- workflow automation
- subprocess and shell execution
- dynamic evaluation or obfuscation
- outbound URLs, telemetry, or auto-registration
- local sensitive data access
- privilege requests or persistence mechanisms
- runtime downloads or remote payloads

### Phase 3: Focused Code Reads

Read only the code that decides behavior:
- the owning abstraction
- the actual network/execution path
- the nearest code that mutates the filesystem, launches processes, or sends data off-machine

Prefer narrow, falsifiable reads over broad exploration.

### Phase 4: Safe Validation

Run low-risk validation only when justified:
- tests, lint, or build already supported by the repo
- only if they do not require installing packages, secrets, network access, browser login, or elevated permissions

If validation would require escalation, stop and say so unless the user explicitly asks to proceed.

### Phase 5: Score and Explain

Score the repo using the weighted rubric in `references/scoring-rubric.md`.

Use `0-4` per dimension:
- `0` minimal risk
- `1` low risk
- `2` moderate risk
- `3` high risk
- `4` severe risk

Convert the weighted total into an overall `0-10` risk score.

## Required Output

Every final answer must include, in this order:

1. `Risk score: X/10` with a short band label
2. A verdict ladder for:
   - `Safe to read`
   - `Safe to clone`
   - `Safe to run in sandbox`
   - `Safe with real data`
3. A short plain-language paragraph answering what the user can safely do now
4. A `Plain-English Bottom Line` written for a non-technical reader in 2-4 short sentences
5. A `Simple Yes/No Checklist` covering:
   - `Read it?`
   - `Clone/download it?`
   - `Run it in a sandbox?`
   - `Use it with real data?`
   - `If still uneasy, what should I do?`
6. A weighted scorecard by dimension
7. Top findings with severity and file references
8. Confidence and unknowns

Keep the explanation concise and human. The user should not need security training to understand the recommendation.
The `Plain-English Bottom Line` and `Simple Yes/No Checklist` must avoid jargon, avoid hedging unless necessary, and make the safe next action obvious to a non-technical person.

## Red Flags To Prioritize

Escalate the score quickly for:
- `preinstall`, `postinstall`, bootstrap one-liners, or remote script execution
- obfuscated or encoded payloads
- bundled binaries with no clear reason
- silent telemetry, auto-registration, or hidden outbound traffic
- access to browser profiles, wallets, keychains, clipboard, shell history, or SSH material
- admin/root requests, scheduled tasks, launch agents, persistence, or security-control changes
- README claims that do not match the code

## Positive Signals To Credit

Lower the score when the repo is genuinely simple and transparent:
- small code surface
- dependency-light or dependency-free
- no install hooks
- explicit and minimal network behavior
- clear entrypoints and readable code
- tests or validation that match the claimed behavior

## Verdict Discipline

Be conservative:
- Do not say `safe with real data` just because tests pass.
- Do not treat `safe to clone` as equivalent to `safe to run`.
- Do not bury the main red flags under a long summary.
- If confidence is low, say so directly.

## Never Do These By Default

- install dependencies from the network
- run browser automation against real profiles
- use the user's credentials or env secrets
- start long-running services or Docker containers
- run anything that obviously reaches the network unless the user explicitly wants runtime tracing or higher-risk validation

## Final Standard

The best audit answers this in one sentence:

> "Here is the risk score, here is what worried me, and here is exactly what you can safely do with this repo right now."
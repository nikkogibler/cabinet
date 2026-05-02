# Cabinet

Cabinet is an intentionally small, curated working library of proven operator workflows for AI-assisted software work.

It is built like a field manual, not a broad platform: direct, calm, reusable, and kept deliberately narrow so each skill has to earn its place. The public library is small on purpose. Every included skill solves a repeatable operational problem without leaning on one private repo, one client, or one internal process.

A few concrete examples:

- If you inherit an unfamiliar repository and need to decide whether it is safe to inspect, sandbox, or trust with real data, use `repo-audit`. Outcome: a conservative plain-language trust assessment and a clear next step.
- If you need to pause an agent session and continue later without reconstructing the whole project, use `handoff` and `resume-from-handoff`. Outcome: continuation-grade context transfer instead of restating everything from scratch.

## What Cabinet Covers

### Repository Trust

- `repo-audit` helps you decide whether a repository is safe to read, clone, run in a sandbox, or trust with real data.
- It is optimized for fast, conservative risk assessment with plain-language output.

### Memory and Context Handoffs

- `handoff` writes a continuation-grade markdown handoff for another agent.
- `resume-from-handoff` loads that handoff, verifies the live state, and continues without making the user restate the whole project.
- These live under `skills/handoff-protocol/` as separate skills on disk, but they form one operational workflow.

### End of Day

- `end-of-day` closes a work interval cleanly: review merge readiness, summarize shipped work, sync stale docs, capture next steps, and prepare a session for sign-off or handoff.
- In Cabinet, `end of day` means since the last trustworthy checkpoint, not strictly the clock.

### Screenshot Cleanup

- `screenshot-cleanup` cleans up a screenshot or image dump on macOS by generating a temporary OCR-assisted helper, sorting files into category folders, and renaming them to readable slugs.
- It is intentionally concrete, explicitly macOS-only, and explicit about local side effects.

## Why This Repo Exists

A lot of useful skills are trapped inside one person's environment. Cabinet is an attempt to publish a small public working set of the good ones without sanding off the operational detail that makes them useful.

The quality bar is simple:

- reusable outside one private codebase
- concrete enough to execute
- compact enough to maintain
- honest about verification and uncertainty

## Admission Rule

Cabinet adds a new top-level skill only when it clears all of these bars:

- it solves a narrow, repeatable operator problem
- it states platform assumptions plainly when they matter
- it declares meaningful local side effects and safety boundaries up front
- it stays Markdown-first; generated helpers are fine, committed executables are not the default
- it improves the library more than it broadens it

## Layout

```text
cabinet/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── chat-session-summaries/
├── council-decisions/
├── docs/
│   ├── adding-a-skill.md
│   └── roadmap.md
└── skills/
    ├── repo-audit/
    │   ├── SKILL.md
    │   └── references/
    ├── handoff-protocol/
    │   ├── handoff/
    │   │   └── SKILL.md
    │   └── resume-from-handoff/
    │       └── SKILL.md
    ├── end-of-day/
    │   └── SKILL.md
    ├── screenshot-cleanup/
    │   └── SKILL.md
    └── _template/
        └── SKILL.md
```

## How To Use It

1. Copy the skill folder you want into your own skills directory or prompt system.
2. Keep any local `references/` directory intact when a skill depends on companion files.
3. Adapt outer framing only when necessary; preserve the core workflow unless you have a reason to change it.
4. Prefer using the skills as operational building blocks, not as decorative prompt text.

Cabinet is Markdown-first. There is no package manager, site generator, or release automation in the first cut.

## Current Working Set

This is the current public working set. It is intentionally selective, not a completeness claim.

- `repo-audit`
- `handoff`
- `resume-from-handoff`
- `end-of-day`
- `screenshot-cleanup`
- `_template` as the starter shape for future additions

## Additions And Roadmap

- See `docs/adding-a-skill.md` for the contribution standard.
- See `docs/roadmap.md` for the near-term direction and current non-goals.

## License

Cabinet is released under the MIT License.

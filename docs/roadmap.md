# Roadmap

Cabinet starts small on purpose. The first release is a tight set of operational skills that are already useful together.

## Launch Scope

The current launch set is:

- `repo-audit`
- `handoff`
- `resume-from-handoff`
- `end-of-day`
- `screenshot-cleanup`
- `_template`

These map to four public capabilities:

- Repository Trust
- Memory and Context Handoffs
- End of Day
- Screenshot Cleanup

## Admission Rule

Cabinet should add a new top-level skill only when it is:

- a narrow, repeatable operator workflow
- explicit about platform assumptions when those assumptions matter
- explicit about local side effects and safety boundaries
- still Markdown-first, with generated helpers preferred over committed executables
- strong enough to improve the library without turning it into a grab bag

## Near-Term Additions

Possible next additions should stay close to the same operating model:

- release or changelog workflows that stay repo-agnostic
- safer continuation and recovery workflows
- review-oriented skills that are portable across codebases
- maintenance skills for keeping repo documentation truthful

Any addition should solve a repeatable problem without expanding Cabinet into a grab bag.

## Non-Goals For Now

Cabinet is intentionally not shipping these yet:

- a site generator
- package manager distribution
- release automation
- CI pipelines for the repo itself
- a large taxonomy of overlapping skills
- private integrations or repo-specific routing rules

## Quality Direction

Future work should improve one of these dimensions:

- portability
- clarity
- verification discipline
- public usefulness
- coherence across the library

If a proposed skill mainly adds surface area, it probably does not belong here yet.

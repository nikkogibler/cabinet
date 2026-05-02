---
name: end-of-day
description: Runs a reusable end-of-day or end-of-session closeout workflow for an active repository: review merge readiness, summarize work since the last checkpoint, sync stale docs, capture next steps, and prepare the session for sign-off.
version: 1.0.0
user-invocable: true
argument-hint: "[repo path, checkpoint file, or leave blank for current workspace]"
license: MIT
metadata:
  author: Nik Gibler
  copyright: Copyright (c) 2026 Nik Gibler
---

# End of Day

Close a work interval cleanly.

This skill is for the end of a session, the end of a work block, or the end of a literal day. In this workflow, `end of day` means everything since the last trustworthy checkpoint, not strictly the wall clock.

A checkpoint might be:

- the most recent sprint report
- the most recent daily report
- the last handoff
- the last tagged release
- the last agreed status note
- a user-provided date or commit boundary

If the repo has no established checkpoint artifact, fall back to the earliest relevant commit on the current branch or the repo.

## Core Outcome

A good closeout should leave the repo and the next session in a better state than they were at the start of the workflow.

By the end of this skill, you should have:

- a clear work window
- an explicit merge or non-merge decision
- a concise summary of shipped or ready work
- stale docs and README updated when needed
- next steps captured in the repo's existing system
- a clean sign-off, session title, or handoff when appropriate

## Default Rules

- Prefer the repo's existing conventions over inventing new ones.
- Use the narrowest reliable validation available in the repo.
- Do not merge or document broken primary branches.
- If the repo already has a reporting cadence, continue it.
- If the repo has no reporting convention, create the smallest useful closeout note rather than inventing a large new system.
- Treat unverified claims as unverified.

## Workflow

Execute the steps in order unless a later step is clearly blocked by an earlier decision.

### Step 0 — Determine The Closeout Window

1. Identify the last trustworthy checkpoint.
2. Prefer an existing closeout artifact if the repo already keeps one.
3. If multiple checkpoints exist, pick the newest one that actually reflects shipped or reviewed work.
4. If no checkpoint exists, use the earliest relevant commit as the start boundary.
5. Use that boundary consistently for commit review, summaries, documentation checks, and follow-up tasks.

Helpful commands when the repo has no stronger local convention:

```bash
git log --reverse --format="%ci" | head -1
git tag --sort=-creatordate
```

### Step 1 — Review Merge Readiness

1. Check the current branch.
2. Decide whether the work is ready to merge, ready only in part, or not ready.
3. Ask the developer whether to:
   - merge all ready work
   - cherry-pick specific commits
   - defer merge and continue with a non-merge closeout
4. If the repo has an established integration flow, follow it.
5. If no workflow is documented, default to the primary integration branch and standard git merge behavior.
6. Run the narrowest reliable validation the repo supports before calling the branch ready.
7. If validation fails, stop and fix the issue before treating the branch as ready.

Helpful commands:

```bash
git branch --show-current
git log --oneline <branch> --not <primary-branch> | head -20
```

### Step 2 — Summarize Shipped Or Ready Work

1. Gather commits since the checkpoint boundary.
2. Prefer the repo's existing reporting format and location.
3. If the repo does not keep reports, create a concise markdown closeout note in an existing docs or handoff location.
4. Group changes by feature area or operational impact.
5. Write plain-language summaries that explain:
   - what changed
   - what it replaced or unblocked
   - why it matters to users, maintainers, or the next session
6. If the repo normally commits reports, show the artifact for review before committing it.

Helpful command:

```bash
git log --since="<CHECKPOINT-START>" --pretty=format:"%h %s%n%b---" --reverse
```

### Step 3 — Sync Documentation

1. Review changed code paths against existing documentation.
2. Update only the docs that are now stale.
3. Use the code as the source of truth.
4. If documentation is already accurate, say so and move on.
5. If the repo has manifests, route maps, API summaries, or architecture notes, check the ones touched by the work window.

### Step 4 — Update README If Needed

1. Compare the README against any public-facing changes in the work window.
2. Update only the sections that are now inaccurate.
3. Do not rewrite accurate sections just because you are already in the file.
4. If the README still reflects reality, skip this step explicitly.

### Step 5 — Final Validation

1. Re-run the most relevant validation after docs and README changes.
2. Confirm the primary branch or working branch is in a truthful state.
3. If the repo has no automated validation, state that clearly rather than implying success.

### Step 6 — Capture Next Steps

1. Create or update next-session tasks in the repo's existing task system.
2. If no task system exists, write a concise next-steps list in the closeout artifact or handoff.
3. For each non-trivial shipped or ready change that still needs human verification, create a testing or follow-up task.
4. Make the next action obvious. Avoid vague backlog language.

### Step 7 — Rename The Session Or Write A Handoff

1. If the environment supports session naming, suggest a concise name based on the dominant theme of the work window.
2. If another agent or a future session will continue the work, write a handoff before sign-off.
3. Preserve:
   - what changed
   - what is verified
   - what remains open
   - what constraints the next session should keep

If a dedicated handoff skill exists in the environment, prefer using it instead of improvising.

### Step 8 — Final Status And Sign-Off

1. Verify there are no unreviewed or accidental changes left behind.
2. Commit and push only the intentional closeout changes.
3. Confirm whether the session ended in one of these states:
   - merged and clean
   - documented but not merged
   - blocked and handed off
4. Make the resulting state obvious to the developer before the session closes.

Helpful command:

```bash
git status
```

## Edge Cases

- **No commits since the checkpoint:** Skip merge and summary work. Still capture next steps, validation state, and any needed handoff.
- **Validation fails after merge:** Stop and repair the branch before treating the session as closed.
- **The last checkpoint is several days old:** That is fine. The window is defined by the last checkpoint, not the calendar date.
- **The working branch has uncommitted changes:** Stash or explicitly account for them before merging.
- **There is no established reporting convention:** Create the smallest useful closeout artifact instead of inventing a heavy process.

## Final Standard

The next person should be able to answer three questions immediately:

- What changed since the last checkpoint?
- What state is the repo in now?
- What should happen next?

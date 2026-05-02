# Mental Model

The right question is not "Is this repo perfectly safe?"

The right question is:

> "What level of trust does this repo deserve, and what is the smallest blast radius I can accept?"

Treat repositories as falling into progressively higher-trust buckets:

1. `Safe to read`
   - You can clone or inspect the source as text.
   - You are not executing it.

2. `Safe to clone`
   - Bringing the repo onto disk is low risk.
   - This is still not the same as installing or running it.

3. `Safe to run in sandbox`
   - You can execute it in a venv, temp directory, container, or disposable VM.
   - No secrets. No personal files. No real browser profile. No real credentials.

4. `Safe with real data`
   - You would allow it to touch real documents, accounts, secrets, browser state, or production systems.
   - This is the highest-trust judgment and should be rare.

## Key Rule

Most repos never earn bucket 4.

That is normal. A repo can still be useful even if it is only trustworthy enough for reading or sandboxed execution.

## What Good Audits Do

Good audits reduce uncertainty by answering:
- what code actually runs
- what leaves the machine
- what data it can access
- what privileges it needs
- how much confidence the reviewer can honestly claim

## What Bad Audits Do

Bad audits:
- confuse `tests passed` with `safe to trust`
- ignore hidden network side effects
- treat package installation as harmless by default
- bury critical red flags under long summaries
- overstate certainty when confidence is low
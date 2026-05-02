# Report Template

Use this structure by default.

## 1. Quick Verdict

Start with the score and the decision:

```text
Risk score: X/10 (band)
```

Then the verdict ladder:

```text
Safe to read: Yes / Caution / No
Safe to clone: Yes / Caution / No
Safe to run in sandbox: Yes / Caution / No
Safe with real data: Yes / Caution / No
```

Then one short paragraph in plain language:

```text
This repo looks low/moderate/high risk because .... You can safely do ... now. Keep ... sandboxed. Do not trust it with ... yet.
```

Then add a mandatory non-technical close:

```text
Plain-English Bottom Line:
In simple terms, this looks ... You can ... Do not ... If you still feel uneasy, ...
```

And a simple yes/no checklist:

```text
Read it? Yes / No
Clone/download it? Yes / No
Run it in a sandbox? Yes / No
Use it with real data? Yes / No
If still uneasy, what should I do? ...
```

## 2. Weighted Scorecard

Use a compact table:

| Dimension | Weight | Score | Why it scored this way |
|---|---:|---:|---|
| Install and execution surface | 2.0 | ? | ... |
| Outbound network and telemetry | 2.0 | ? | ... |
| Local data and secrets exposure | 2.0 | ? | ... |
| Privilege and system control | 1.5 | ? | ... |
| Dependency and supply-chain surface | 1.0 | ? | ... |
| Transparency and README alignment | 1.0 | ? | ... |
| Maintainability and validation confidence | 0.5 | ? | ... |

## 3. Findings

List the highest-signal findings first.

Format:

```text
P1: Short finding title
What it means: ...
Where: file path(s)
Why it matters: ...
```

If no significant findings exist, say that directly.

## 4. Confidence and Unknowns

Explicitly state:
- `High confidence` when the repo is small and the control surface is clear
- `Medium confidence` when some behavior is clear but there are moderate blind spots
- `Low confidence` when binaries, generated code, or large opaque surfaces dominate

Also state what you did not verify.

## 5. Safe Next Step

End with a concrete recommendation, for example:
- safe to clone and read, but not worth running
- safe to try in a venv with no secrets
- only run in a disposable container or VM
- do not run unless you explicitly want to investigate suspicious behavior further

## Style Rules

- Keep the top summary short.
- Use human language, not security theater.
- Do not pretend certainty where there is none.
- Do not confuse privacy concerns with malware; explain the difference.
- Always answer the user's real question: "What can I safely do with this repo?"
- The `Plain-English Bottom Line` should read like advice to a cautious non-technical person, not a security report.
- The `Simple Yes/No Checklist` should be explicit and decisive. Prefer `No` over vague wording when trust is not established.
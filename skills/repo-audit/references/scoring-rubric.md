# Scoring Rubric

Use a `0-4` risk score per dimension.

- `0` minimal risk
- `1` low risk
- `2` moderate risk
- `3` high risk
- `4` severe risk

## Weights

Use these weights for the overall `0-10` risk score:

| Dimension | Weight |
|---|---:|
| Install and execution surface | 2.0 |
| Outbound network and telemetry | 2.0 |
| Local data and secrets exposure | 2.0 |
| Privilege and system control | 1.5 |
| Dependency and supply-chain surface | 1.0 |
| Transparency and README alignment | 1.0 |
| Maintainability and validation confidence | 0.5 |

Total weight: `10.0`

## Formula

Overall risk score:

```text
overall_0_to_10 = round(sum(score * weight for each dimension) / 4, 1)
```

Because the total weight is `10`, dividing by `4` maps the weighted `0-4` scale to `0-10`.

## Risk Bands

| Overall Score | Band |
|---|---|
| `0.0 - 1.9` | Very low |
| `2.0 - 3.9` | Low |
| `4.0 - 5.9` | Moderate |
| `6.0 - 7.9` | High |
| `8.0 - 10.0` | Severe |

## Dimension Criteria

### 1. Install and execution surface

| Score | Criteria |
|---|---|
| `0` | No install hooks, no subprocesses, no runtime downloads, local read/write only |
| `1` | Standard build/test scripts, explicit entrypoints, minimal execution surface |
| `2` | Browser automation, local daemons, or moderate execution surface, but explicit and understandable |
| `3` | Install hooks, shell/bootstrap scripts, runtime fetch-and-exec behavior, or opaque execution paths |
| `4` | Arbitrary command execution, hidden bootstrap, persistence, destructive behavior |

### 2. Outbound network and telemetry

| Score | Criteria |
|---|---|
| `0` | No network behavior, or only explicit user-directed connections |
| `1` | Clear, necessary external calls that match the README |
| `2` | Telemetry, analytics, or cloud calls that are disclosed and understandable |
| `3` | Hidden or under-disclosed telemetry, silent registration, multiple undeclared endpoints |
| `4` | Covert exfiltration, suspicious endpoints, remote payload retrieval, C2-like behavior |

### 3. Local data and secrets exposure

| Score | Criteria |
|---|---|
| `0` | Access limited to project-local files or explicit user-selected inputs |
| `1` | Reads only clearly necessary user-provided files or standard config |
| `2` | Uses env vars, credentials, or broader file access in an explicit, feature-motivated way |
| `3` | Touches browser profiles, clipboard, keychains, shell history, SSH material, or broad filesystem state |
| `4` | Harvests credentials, wallets, secrets, or exports sensitive local data |

### 4. Privilege and system control

| Score | Criteria |
|---|---|
| `0` | No system control, no elevated permissions, no privileged resources |
| `1` | User-level sockets or explicit connections to user-specified services |
| `2` | Docker, RCON, browser control, or writes outside the output dir, but explicit and bounded |
| `3` | Admin/root assumptions, service management, scheduled tasks, or broad machine control |
| `4` | Persistence, security-control changes, privileged escalation, system compromise potential |

### 5. Dependency and supply-chain surface

| Score | Criteria |
|---|---|
| `0` | Dependency-free or tiny, understandable dependency set |
| `1` | Small dependency footprint with normal packaging and lockfiles |
| `2` | Moderate dependency surface or some uncertainty around package provenance |
| `3` | Large dependency surface, native packages, missing locks, or significant supply-chain blind spots |
| `4` | Suspicious packages, typosquats, vendored opaque binaries, runtime-downloaded executables |

### 6. Transparency and README alignment

| Score | Criteria |
|---|---|
| `0` | README and code match closely; network, data access, and execution are clearly disclosed |
| `1` | Minor documentation gaps, but behavior is still obvious in code |
| `2` | Some under-disclosed behavior or marketing oversimplification |
| `3` | Major mismatch between README claims and actual code paths |
| `4` | Deceptive or intentionally hidden behavior |

### 7. Maintainability and validation confidence

| Score | Criteria |
|---|---|
| `0` | Small, readable codebase with clear entrypoints and good validation coverage |
| `1` | Mostly understandable with small blind spots |
| `2` | Mixed clarity, partial tests, generated or opaque areas |
| `3` | Difficult to audit responsibly, large blind spots, low confidence |
| `4` | Opaque blobs or obfuscation dominate; confidence is very low |

## Verdict Ladder Guidance

Use the score and the dimension mix together. Do not rely on the overall number alone.

### Safe to read
- `Yes` unless there are clear malicious indicators or illegal payloads.
- `Caution` if the repo is highly opaque or dominated by unreadable blobs.
- `No` only when the content itself is clearly malicious or dangerous to even handle normally.

### Safe to clone
- `Yes` when cloning itself is low risk.
- `Caution` if clone pulls large binaries, submodules, LFS assets, or materially expands the surface.
- `No` if clone-time behavior meaningfully increases risk.

### Safe to run in sandbox
- `Yes` when overall risk is roughly `<= 4.5` and install/data/privilege scores are not severe.
- `Caution` when overall risk is roughly `4.6 - 6.5`, or confidence is limited.
- `No` when overall risk is `> 6.5` or any severe install/data/privilege red flag dominates.

### Safe with real data
- `Yes` only when overall risk is roughly `<= 2.5`, confidence is high, and network/data/privilege scores are all `<= 1`.
- `Caution` when overall risk is roughly `2.6 - 4.0` and behavior is explicit and limited.
- `No` above that, or whenever hidden telemetry, secret access, auto-registration, or trust mismatches are present.

## Severity Tags For Findings

Use these severities in the report:
- `P0` obvious maliciousness, covert exfiltration, credential theft, persistence, or extreme risk
- `P1` major trust/security issue that materially changes whether the repo should be run
- `P2` moderate issue, privacy concern, misleading behavior, or significant unknown
- `P3` minor concern, documentation gap, or low-impact risk signal
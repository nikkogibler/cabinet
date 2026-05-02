# Default Workflow

Repo Audit should proceed in controlled passes.

## Pass 1: Establish the target

Use the most concrete target available:
- local workspace
- local path
- repo URL

If the repo is remote, clone with `--depth=1` into a temp directory.

## Pass 2: High-risk surface mapping

Map only the areas that could decide trust:
- package manifests and lockfiles
- CI/workflows
- Dockerfiles and shell scripts
- entrypoints
- binaries, generated blobs, or vendored assets

The goal is not to read the whole repo. The goal is to find the few places that matter.

## Pass 3: Dangerous-pattern search

Search for the patterns that most often change the trust decision:
- install hooks
- subprocesses and shelling out
- dynamic eval
- outbound URLs and background network calls
- secret or credential access
- clipboard, wallet, browser profile, or keychain access
- privilege and persistence mechanisms
- runtime download or execution of remote content

## Pass 4: Focused verification reads

Once a pattern hits, read the narrow controlling code path:
- where the network call is made
- where the subprocess is launched
- where the secret is read
- where the filesystem is touched

Do not keep exploring broad surrounding surfaces once the deciding path is clear.

## Pass 5: Safe validation

Allowed by default:
- local tests that already run with the present toolchain
- lint or build if no package installation, network, secrets, browser login, or elevated permissions are required

Do not do by default:
- `npm install`, `pip install`, `cargo build`, `docker build`, browser-driven runs, or server startup that reaches the network
- anything that asks for API keys, OAuth, wallets, SSH, or browser profiles

If validation requires these, stop and say so.

## Pass 6: Final recommendation

Produce:
- overall score
- weighted subscores
- verdict ladder
- short safe-usage guidance
- plain-English bottom line for a non-technical reader
- simple yes/no checklist with a clear safest next step

Always say what is still unknown.

## Escalation Rules

Only escalate beyond the default mode when the user explicitly wants:
- runtime tracing
- dependency installation
- network observation
- sandbox execution of the app
- testing with real credentials or real data

When escalating, restate the blast radius before doing it.
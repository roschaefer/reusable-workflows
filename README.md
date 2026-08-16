# reusable-workflows

Shared GitHub Actions reusable workflows for roschaefer's Rust CLI repos
(hledger-document-check, hledger-elster, hledger-journal-check,
markdown-to-cucumber, ...).

## rust-ci.yml

Build/lint/test a Rust crate, optionally producing linux/macos release
binaries and publishing a rolling `latest` GitHub release on push to `main`.
Builds run `just build`, and hledger (when `needs_hledger` is set) is
always pinned to the same version (currently 1.52.1, hardcoded in the
workflow) — both are fixed on purpose so every caller stays in lockstep;
bump them in this repo, not per-caller.

`platforms` controls both the check job's shape and whether the release job
runs. Empty (`'[]'`, the default) means: one non-matrixed check job on
ubuntu-latest, named exactly `check`, no release job — for tools with no
release binary. A non-empty JSON array of platform names (currently only
`linux-x86_64` and `macos-arm64` are supported) matrixes the check job per
platform, each named `check-<platform>` (e.g. `check-linux-x86_64`,
`check-macos-arm64`) and used verbatim as the release asset suffix
(`<binary_name>-<platform>`), and enables the release job.

`platforms` values are descriptive names, not GitHub Actions runner
labels — the only place that distinction matters is `runs-on:`, which
looks the runner label up from a small fixed map
(`{"linux-x86_64":"ubuntu-latest","macos-arm64":"macos-latest"}`). That's
the only lookup in the whole workflow: job names and asset names use the
caller's `platforms` value directly, so `platforms` is the one label
callers, job names, and release assets all agree on. Add a platform by
adding one entry to that map — nothing else needs to change.

The job's explicit `name:` is what makes the bare `check` case possible —
GitHub only auto-appends matrix values to a job's status-check name when
the job doesn't set its own `name:`; setting one (even for a matrixed job)
replaces the auto-generated name entirely instead of adding to it.

```yaml
jobs:
  ci:
    uses: roschaefer/reusable-workflows/.github/workflows/rust-ci.yml@<commit-sha> # main
    with:
      binary_name: hledger-document-check
      platforms: '["linux-x86_64","macos-arm64"]'  # default: '[]' (no release)
      needs_hledger: true   # default: false
      needs_fava: true      # default: false
```

## pr-metadata.yml

Validates PR titles follow Conventional Commits via
`amannn/action-semantic-pull-request`.

```yaml
on:
  pull_request_target:
    types: [opened, edited, reopened, ready_for_review, synchronize]

jobs:
  pr-metadata:
    uses: roschaefer/reusable-workflows/.github/workflows/pr-metadata.yml@<commit-sha> # main
```

## Versioning

Callers pin `uses:` to a full commit SHA of this repo (comment shows the
branch it was resolved from), the same way third-party actions are pinned
inside these workflows. `pr-metadata.yml` in particular runs on
`pull_request_target` with the caller's secrets, so a floating `@main` ref
would let any future push here (or a compromise of this repo) execute with
every caller's privileged token on the next PR event, without a reviewed
change in the caller repo. Bumping this repo therefore means updating the
pinned SHA in every caller, not just pushing here.

Neither workflow declares custom secrets — both only use the automatically
generated `secrets.GITHUB_TOKEN`, which reusable-workflow jobs receive
regardless of `secrets:`/`secrets: inherit` in the caller. So callers don't
need (and shouldn't add) `secrets: inherit`.

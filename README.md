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

`os_list` controls both the check job's shape and whether the release job
runs. Empty (`'[]'`, the default) means: one non-matrixed check job on
ubuntu-latest, named exactly `check`, no release job — for tools with no
release binary. A non-empty JSON array (currently only
`'["ubuntu-latest","macos-latest"]'` is supported, since asset-name and
release-job logic hardcode that pair) matrixes the check job per OS,
each named `check-<os-and-arch>` (e.g. `check-linux-x86_64`,
`check-macos-arm64` — a fixed lookup table in the workflow, not derived
from `binary_name`), and enables the release job.

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
      os_list: '["ubuntu-latest","macos-latest"]'  # default: '[]' (no release)
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

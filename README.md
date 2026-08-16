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

```yaml
jobs:
  ci:
    uses: roschaefer/reusable-workflows/.github/workflows/rust-ci.yml@<commit-sha> # main
    with:
      binary_name: hledger-document-check
      needs_hledger: true   # default: false
      needs_fava: true      # default: false
      # release: true        # default: true — set false for tools with no release binary
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

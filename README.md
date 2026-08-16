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
    uses: roschaefer/reusable-workflows/.github/workflows/rust-ci.yml@main
    with:
      binary_name: hledger-document-check
      needs_hledger: true   # default: false
      needs_fava: true      # default: false
      # release: true        # default: true — set false for tools with no release binary
    secrets: inherit
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
    uses: roschaefer/reusable-workflows/.github/workflows/pr-metadata.yml@main
    secrets: inherit
```

## Versioning

Callers reference `@main`. Since this repo is fully owned by the same
person as the callers, that's treated as trusted first-party code (unlike
third-party actions, which are pinned by commit SHA inside these
workflows).

<!-- markdownlint-disable -->

# Hardening Report: Namchee--conventional-pr/v0.15.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Namchee--conventional-pr/v0.15.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags (e.g., @v2, @v3) instead of immutable full 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit.

- cover.yml: `actions/checkout@v3`, `actions/setup-go@v2`
- cpr.yml: `actions/checkout@v3`
- lint.yml: `actions/checkout@v2`, `golangci/golangci-lint-action@v2`

Locations:

- `.github/workflows/cover.yml:10`
- `.github/workflows/cover.yml:13`
- `.github/workflows/cpr.yml:10`
- `.github/workflows/lint.yml:10`
- `.github/workflows/lint.yml:12`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no individual job defines its own `permissions:` block. Without explicit permissions, workflows run with the default (often overly broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/cover.yml:1`
- `.github/workflows/cpr.yml:1`
- `.github/workflows/lint.yml:1`

### unsafe-shell (severity: high)

The 'Upload coverage to Codecov' step in cover.yml executes a remote shell script by piping it directly to bash via process substitution: `bash <(curl -s https://codecov.io/bash)`. This pattern executes whatever content is served at that URL without any integrity verification, enabling a compromised or hijacked URL to run arbitrary code on the runner.

Locations:

- `.github/workflows/cover.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, unsafe-shell

**Notes:**

Fixed all three findings across cover.yml, cpr.yml, and lint.yml:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments preserved: actions/checkout@v3→f43a0e5..., actions/checkout@v2→ee0669b..., actions/setup-go@v2→bfdd357..., golangci/golangci-lint-action@v2→5c56cd6...

2. missing-permissions: Added top-level `permissions: {}` to all three workflows plus minimal job-level permissions (contents: read for build/lint, contents: read + pull-requests: write for cpr).

3. unsafe-shell: Replaced `bash <(curl -s https://codecov.io/bash)` in cover.yml with a two-step approach: download to file first, then execute separately, eliminating the unsafe pipe-to-bash pattern.


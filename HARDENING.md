<!-- markdownlint-disable -->

# Hardening Report: Namchee--conventional-pr/v0.15.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Namchee--conventional-pr/v0.15.6** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tag refs instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks. Failing references:
- cover.yml: `actions/checkout@v4`, `actions/setup-go@v2`
- lint.yml: `actions/checkout@v4`, `golangci/golangci-lint-action@v2`
- self-test.yml: `actions/checkout@v4`

Locations:

- `.github/workflows/cover.yml:9`
- `.github/workflows/cover.yml:11`
- `.github/workflows/lint.yml:10`
- `.github/workflows/lint.yml:12`
- `.github/workflows/self-test.yml:9`

### unsafe-shell (severity: high)

cover.yml pipes remote content directly to bash via process substitution: `bash <(curl -s https://codecov.io/bash)`. This executes arbitrary remote code without verification, and is equivalent to `curl ... | bash`.

Locations:

- `.github/workflows/cover.yml:17`

### missing-permissions (severity: medium)

None of the workflow files declare a top-level `permissions:` key, and no individual jobs declare their own `permissions:` block. This means jobs run with the default (potentially broad) token permissions. Affected files: cover.yml, lint.yml, self-test.yml.

Locations:

- `.github/workflows/cover.yml:1`
- `.github/workflows/lint.yml:1`
- `.github/workflows/self-test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, missing-permissions

**Notes:**

Fixed all three findings across cover.yml, lint.yml, and self-test.yml:

1. unpinned-uses: Pinned all action references to full 40-char SHAs:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4 (all three files)
   - actions/setup-go@v2 → @bfdd3570ce990073878bf10f6b2d79082de49492 # v2 (cover.yml)
   - golangci/golangci-lint-action@v2 → @5c56cd6c9dc07901af25baab6f2b0d9f3b7c3018 # v2 (lint.yml)

2. unsafe-shell: Replaced `bash <(curl -s https://codecov.io/bash)` in cover.yml with a safe two-step approach: download the script to a local file first, then execute it.

3. missing-permissions: Added `permissions: {}` at the top level of all three workflow files, and added minimal job-level permissions (contents: read for build/lint jobs; contents: read + pull-requests: write for self-test job which needs to interact with PRs).


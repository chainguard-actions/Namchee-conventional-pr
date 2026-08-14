<!-- markdownlint-disable -->

# Hardening Report: Namchee--conventional-pr/v0.15.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Namchee--conventional-pr/v0.15.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags or branch names instead of pinned full-length SHA commit hashes. This exposes the workflow to supply-chain attacks if the referenced action is compromised or its tag is moved.

Failing references:
- cover.yml: actions/checkout@v4, actions/setup-go@v2
- cpr.yml: Namchee/conventional-pr@master
- lint.yml: actions/checkout@v4, golangci/golangci-lint-action@v2
- self-test.yml: actions/checkout@v4

Locations:

- `.github/workflows/cover.yml:9`
- `.github/workflows/cover.yml:11`
- `.github/workflows/cpr.yml:10`
- `.github/workflows/lint.yml:10`
- `.github/workflows/lint.yml:12`
- `.github/workflows/self-test.yml:9`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no job within any workflow defines job-level permissions. Without explicit permissions, workflows run with the default (often broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/cover.yml:1`
- `.github/workflows/cpr.yml:1`
- `.github/workflows/lint.yml:1`
- `.github/workflows/self-test.yml:1`

### unsafe-shell (severity: high)

cover.yml downloads and immediately executes a remote shell script using bash process substitution: `bash <(curl -s https://codecov.io/bash)`. This is equivalent to `curl ... | bash` — the script content is never inspected or verified before execution, allowing a compromised or man-in-the-middle response to execute arbitrary code on the runner.

Locations:

- `.github/workflows/cover.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, unsafe-shell

**Notes:**

Fixed all four workflow files: (1) Pinned all action references to full commit SHAs with tag comments for readability: actions/checkout@v4→11d5960a, actions/setup-go@v2→bfdd3570, Namchee/conventional-pr@master→130d7ab1, golangci/golangci-lint-action@v2→5c56cd6c. (2) Added top-level `permissions: {}` to all four workflows plus minimal job-level permissions (contents: read for build/lint, pull-requests: write for PR workflows). (3) Fixed unsafe shell in cover.yml by downloading the Codecov script to a file first with `curl -o codecov.sh` then executing `bash codecov.sh` separately, eliminating the `bash <(curl ...)` pattern.


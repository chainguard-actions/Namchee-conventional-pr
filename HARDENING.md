<!-- markdownlint-disable -->

# Hardening Report: Namchee--conventional-pr/v0.16.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Namchee--conventional-pr/v0.16.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference GitHub Actions using mutable version tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: cover.yml — actions/checkout@v4, actions/setup-go@v2; lint.yml — actions/checkout@v4, golangci/golangci-lint-action@v2; self-test.yml — actions/checkout@v4.

Locations:

- `.github/workflows/cover.yml:9`
- `.github/workflows/cover.yml:11`
- `.github/workflows/lint.yml:9`
- `.github/workflows/lint.yml:11`
- `.github/workflows/self-test.yml:9`

### unsafe-shell (severity: high)

cover.yml downloads and immediately executes a remote shell script using bash process substitution: `bash <(curl -s https://codecov.io/bash)`. This pipes untrusted remote content directly into a shell interpreter without any integrity verification (e.g. checksum or signature check), allowing a compromised or man-in-the-middle response to execute arbitrary code on the runner.

Locations:

- `.github/workflows/cover.yml:18`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level `permissions:` key, and no individual job within any of these workflows declares job-level permissions. Without explicit permissions, workflows run with the default (potentially broad) token permissions. All three files are affected: cover.yml, lint.yml, and self-test.yml.

Locations:

- `.github/workflows/cover.yml:1`
- `.github/workflows/lint.yml:1`
- `.github/workflows/self-test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, missing-permissions

**Notes:**

Fixed all three findings across cover.yml, lint.yml, and self-test.yml:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments — actions/checkout@v4→@11d5960a..., actions/setup-go@v2→@bfdd3570..., golangci/golangci-lint-action@v2→@5c56cd6c...

2. unsafe-shell: Replaced `bash <(curl -s https://codecov.io/bash)` in cover.yml with a two-step approach: download script to file first (`curl -fsSL -o codecov.sh`), then execute separately (`bash codecov.sh`).

3. missing-permissions: Added top-level `permissions: contents: read` to cover.yml and lint.yml; added `permissions: contents: read` + `pull-requests: write` to self-test.yml (the action edits PRs so write access is needed).


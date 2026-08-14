<!-- markdownlint-disable -->

# Hardening Report: Namchee--conventional-pr/v0.15.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Namchee--conventional-pr/v0.15.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags instead of full 40-character SHA commit digests. This exposes the workflow to supply-chain attacks if the referenced tag is moved or the upstream repository is compromised.

- .github/workflows/cover.yml: `actions/checkout@v3`, `actions/setup-go@v2`
- .github/workflows/cpr.yml: `actions/checkout@v3`
- .github/workflows/lint.yml: `actions/checkout@v2`, `golangci/golangci-lint-action@v2`

All of these should be pinned to their full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/cover.yml:9`
- `.github/workflows/cover.yml:11`
- `.github/workflows/cpr.yml:9`
- `.github/workflows/lint.yml:9`
- `.github/workflows/lint.yml:11`

### unsafe-shell (severity: high)

The `cover.yml` workflow downloads and immediately executes a remote shell script using `bash <(curl -s https://codecov.io/bash)`. This pattern (process substitution piping remote content to a shell) is equivalent to `curl | bash` and executes arbitrary code from a remote server without any integrity verification. If the remote URL is compromised or the content changes, malicious code will run on the runner. The script should be downloaded to a file, its checksum verified, and then executed separately.

Locations:

- `.github/workflows/cover.yml:17`

### permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no job within them defines job-level `permissions:` either. Without explicit permissions, workflows run with the default repository permissions (which may include `write` access to contents, pull requests, etc.), violating the principle of least privilege. Each workflow should declare the minimal permissions required, e.g. `permissions: read-all` or specific scopes like `contents: read`.

Affected files:
- .github/workflows/cover.yml (no permissions key)
- .github/workflows/cpr.yml (no permissions key)
- .github/workflows/lint.yml (no permissions key)

Locations:

- `.github/workflows/cover.yml:1`
- `.github/workflows/cpr.yml:1`
- `.github/workflows/lint.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, permissions

**Notes:**

Fixed all three findings across .github/workflows/cover.yml, .github/workflows/cpr.yml, and .github/workflows/lint.yml:

1. unpinned-uses: Pinned all 5 action references to full 40-char SHAs with tag comments preserved: actions/checkout@v3→a37ce91, actions/setup-go@v2→bfdd357, actions/checkout@v2→0717577, golangci/golangci-lint-action@v2→5c56cd6.

2. unsafe-shell: Replaced `bash <(curl -s https://codecov.io/bash)` in cover.yml with a two-step approach: download to codecov.sh first, then execute with `bash codecov.sh`.

3. permissions: Added top-level `permissions: contents: read` to all three workflow files. cpr.yml additionally gets `pull-requests: read` since it operates on pull requests via pull_request_target.


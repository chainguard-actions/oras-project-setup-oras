<!-- markdownlint-disable -->

# Hardening Report: oras-project--setup-oras/v1.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **oras-project--setup-oras/v1.2.2** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is interpolated directly inside a run: shell command. In update-version.yml, the step 'Tag and push new major and minor versions' contains `VERSION=${{ github.event.release.tag_name }}` which injects the release tag name directly into the shell script before the shell ever sees it. A malicious tag name containing shell metacharacters (e.g. `;`, `$(...)`, `|`) could lead to arbitrary command execution. Fix: move the value into an env: variable and double-quote it in the script, e.g. `env: TAG_NAME: ${{ github.event.release.tag_name }}` then `VERSION="$TAG_NAME"`.

Locations:

- `.github/workflows/update-version.yml:34`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tag refs instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if the upstream tag is moved or compromised. Failing references:
- check-dist.yml: `actions/checkout@v4`, `actions/setup-node@v4`
- license-checker.yml: `actions/checkout@v4`, `apache/skywalking-eyes/header@v0.6.0`, `apache/skywalking-eyes/dependency@v0.6.0`
- test.yml: `actions/checkout@v4` (used in three steps)
- update-version.yml: `actions/checkout@v4`
All should be pinned to full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/check-dist.yml:25`
- `.github/workflows/check-dist.yml:29`
- `.github/workflows/license-checker.yml:27`
- `.github/workflows/license-checker.yml:29`
- `.github/workflows/license-checker.yml:33`
- `.github/workflows/test.yml:35`
- `.github/workflows/test.yml:55`
- `.github/workflows/test.yml:70`
- `.github/workflows/update-version.yml:22`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without explicit permissions, workflows inherit the repository's default token permissions (often broad write access), violating the principle of least privilege. Affected files: check-dist.yml, test.yml, update-version.yml. Each should declare a top-level `permissions:` block with only the minimal scopes required (e.g. `contents: read` for read-only checkouts). Note: license-checker.yml already has a top-level permissions block and is not affected.

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files:

1. script-injection (update-version.yml line 34): Moved `${{ github.event.release.tag_name }}` from the run: shell script into an env: block as TAG_NAME. The script now uses `VERSION="$TAG_NAME"` to safely reference the value.

2. unpinned-uses: Pinned all action references to full 40-char SHAs:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5 (all 5 occurrences across check-dist.yml, license-checker.yml, test.yml, update-version.yml)
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 (check-dist.yml)
   - apache/skywalking-eyes/header@v0.6.0 → @cd7b195c51fd3d6ad52afceb760719ddc6b3ee91 (license-checker.yml)
   - apache/skywalking-eyes/dependency@v0.6.0 → @cd7b195c51fd3d6ad52afceb760719ddc6b3ee91 (license-checker.yml)

3. missing-permissions: Added top-level permissions blocks:
   - check-dist.yml: contents: read
   - test.yml: contents: read
   - update-version.yml: contents: write (required to push tags)
   - license-checker.yml: already had permissions block, not modified

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in `.github/workflows/update-version.yml` by double-quoting all expansions of `${VERSION}`, `${MAJOR}`, and `${MINOR}` throughout the 'Tag and push new major and minor versions' run block. The `${{ github.event.release.tag_name }}` expression was already correctly isolated in the `env:` block as `TAG_NAME`; the fix ensures all derived shell variables are double-quoted in: `echo "${VERSION}"`, `echo "${VERSION}"` (for MINOR), `[ -z "${VERSION}" ]`, `[ -z "${MAJOR}" ]`, `[ -z "${MINOR}" ]`, `git tag -f "${MAJOR}" "${VERSION}"`, `git tag -f "${MINOR}" "${VERSION}"`, `git push origin "${MAJOR}" --force`, and `git push origin "${MINOR}" --force`.


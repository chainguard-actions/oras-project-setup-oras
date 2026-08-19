<!-- markdownlint-disable -->

# Hardening Report: oras-project--setup-oras/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **oras-project--setup-oras/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): `${{ github.event.release.tag_name }}` is directly interpolated into a `run:` shell script on line 35: `VERSION=${{ github.event.release.tag_name }}`. This allows an attacker who can create a release with a crafted tag name to inject arbitrary shell commands. Rule (b): The derived shell variables `${VERSION}`, `${MAJOR}`, and `${MINOR}` — all sourced from the untrusted `github.event.release.tag_name` context — are used unquoted in shell commands such as `git tag -f ${MAJOR} ${VERSION}` and `git push origin ${MAJOR} --force`, enabling shell metacharacter injection.

Locations:

- `.github/workflows/update-version.yml:35`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved or the upstream repository is compromised. Failing references: check-dist.yml: `actions/checkout@v6`, `actions/setup-node@v6`; license-checker.yml: `actions/checkout@v6`, `apache/skywalking-eyes/header@v0.8.0`, `apache/skywalking-eyes/dependency@v0.8.0`; test.yml: `actions/checkout@v6`; update-version.yml: `actions/checkout@v6`.

Locations:

- `.github/workflows/check-dist.yml:22`
- `.github/workflows/check-dist.yml:25`
- `.github/workflows/license-checker.yml:22`
- `.github/workflows/license-checker.yml:24`
- `.github/workflows/license-checker.yml:27`
- `.github/workflows/test.yml:38`
- `.github/workflows/update-version.yml:26`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g., write access to contents). Each workflow should declare the minimal permissions required. Affected files: check-dist.yml, test.yml, update-version.yml.

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files:

1. script-injection (update-version.yml): Moved `${{ github.event.release.tag_name }}` into a step env var `TAG_NAME` and double-quoted all shell variable expansions (`"$VERSION"`, `"$MAJOR"`, `"$MINOR"`) throughout the run script.

2. unpinned-uses: Pinned all mutable tag references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 (all 4 workflow files)
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38 (check-dist.yml)
   - apache/skywalking-eyes/header@v0.8.0 → @61275cc80d0798a405cb070f7d3a8aaf7cf2c2c1 (license-checker.yml)
   - apache/skywalking-eyes/dependency@v0.8.0 → @61275cc80d0798a405cb070f7d3a8aaf7cf2c2c1 (license-checker.yml)

3. missing-permissions: Added top-level permissions blocks:
   - check-dist.yml: `contents: read`
   - test.yml: `contents: read`
   - update-version.yml: `contents: write` (required to push tags)


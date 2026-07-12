<!-- markdownlint-disable -->

# Hardening Report: oras-project--setup-oras/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **oras-project--setup-oras/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In update-releases.yml, `${{ inputs.version }}` is interpolated directly into shell commands: `if [ -n "${{ inputs.version }}" ]` and `node .github/scripts/update-releases.mjs "${{ inputs.version }}"`. An attacker-controlled workflow_dispatch input could inject arbitrary shell commands. In update-version.yml, `${{ github.event.release.tag_name }}` is interpolated directly into a shell assignment: `VERSION=${{ github.event.release.tag_name }}`, and the resulting unquoted variables `${VERSION}`, `${MAJOR}`, `${MINOR}` are used in git commands without quoting (sub-rule b).

Locations:

- `.github/workflows/update-releases.yml:54`
- `.github/workflows/update-releases.yml:55`
- `.github/workflows/update-version.yml:33`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: check-dist.yml: actions/checkout@v7, actions/setup-node@v6; license-checker.yml: actions/checkout@v7, apache/skywalking-eyes/header@v0.8.0, apache/skywalking-eyes/dependency@v0.8.0; test.yml: actions/checkout@v7 (×3); update-releases.yml: actions/checkout@v7, actions/setup-node@v6, peter-evans/create-pull-request@v8; update-version.yml: actions/checkout@v7.

Locations:

- `.github/workflows/check-dist.yml:25`
- `.github/workflows/check-dist.yml:30`
- `.github/workflows/license-checker.yml:26`
- `.github/workflows/license-checker.yml:28`
- `.github/workflows/license-checker.yml:31`
- `.github/workflows/test.yml:38`
- `.github/workflows/test.yml:52`
- `.github/workflows/test.yml:66`
- `.github/workflows/update-releases.yml:38`
- `.github/workflows/update-releases.yml:41`
- `.github/workflows/update-releases.yml:63`
- `.github/workflows/update-version.yml:24`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job, meaning they run with the default (potentially broad) token permissions: check-dist.yml, test.yml, and update-version.yml.

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across 5 workflow files: (1) script-injection in update-releases.yml (moved inputs.version to env block) and update-version.yml (moved github.event.release.tag_name to env block and properly quoted all shell variables); (2) pinned all 12 unpinned action references to full 40-char SHAs with tag comments preserved (actions/checkout@v7, actions/setup-node@v6, peter-evans/create-pull-request@v8, apache/skywalking-eyes/header@v0.8.0, apache/skywalking-eyes/dependency@v0.8.0); (3) added top-level permissions: contents: read to check-dist.yml, test.yml, and update-version.yml (update-version.yml also got job-level contents: write for tag pushing).


<!-- markdownlint-disable -->

# Hardening Report: at-wat--bloom-release-action/v0.0.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **at-wat--bloom-release-action/v0.0.11** was hardened automatically. 3 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: GitHub Actions expressions are directly interpolated inside a run: shell command string in version-tag.yml. The line `git remote set-url origin https://${{ secrets.GITHUB_TOKEN }}@github.com/${{ github.repository }}.git` embeds both ${{ secrets.GITHUB_TOKEN }} and ${{ github.repository }} directly into the shell command. Any ${{ }} expression in a run: block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever quotes it. In particular, github.repository is attacker-influenced (e.g. via a repository name containing shell metacharacters). These should be moved to env: variables and referenced as quoted shell variables (e.g. "$GITHUB_REPOSITORY").

Locations:

- `.github/workflows/version-tag.yml:13`

### unpinned-uses (severity: high)

Both workflow files reference actions/checkout@v3, which is a mutable tag rather than a full 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling supply-chain attacks. Each uses: reference should be pinned to a full SHA, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3.

Locations:

- `.github/workflows/ci.yaml:12`
- `.github/workflows/version-tag.yml:10`

### missing-permissions (severity: medium)

Neither .github/workflows/ci.yaml nor .github/workflows/version-tag.yml declares a top-level permissions: key, and no job within either file declares job-level permissions. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (which may be write-all for older repositories), granting the GITHUB_TOKEN broader access than necessary. A minimal permissions block (e.g. contents: read) should be added at the top level or per job.

Locations:

- `.github/workflows/ci.yaml:1`
- `.github/workflows/version-tag.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across both workflow files:
1. script-injection (version-tag.yml line 13): Moved ${{ secrets.GITHUB_TOKEN }} and ${{ github.repository }} from the run: shell command into the step's env: block as GITHUB_TOKEN and GITHUB_REPOSITORY, then referenced them as quoted shell variables in the git remote set-url command.
2. unpinned-uses (ci.yaml line 12, version-tag.yml line 10): Pinned actions/checkout@v3 to the full commit SHA a37ce9120846195fa4ece8f58b268e6043cb2f26 with a # v3 comment in both files.
3. missing-permissions (ci.yaml line 1, version-tag.yml line 1): Added top-level permissions: contents: read to ci.yaml (sufficient for checkout and testing) and permissions: contents: write to version-tag.yml (required to push tags to the repository).

### Iteration 2

**Fixes applied:** hardcoded-credentials, script-injection

**Notes:**

1. ci.yaml line 35: Replaced hardcoded literal 'dummy' for github_token_bloom with `${{ secrets.GITHUB_TOKEN_BLOOM }}` to use a proper GitHub Actions secret expression instead of a hardcoded value.
2. version-tag.yml lines 19-20: Double-quoted all unquoted shell variable expansions in the 'tag' step's run block. The git tag commands now use `"$(basename "${TAG%.*.*}")"` and `"$(basename "${TAG%.*}")"` to prevent shell metacharacter injection from attacker-controlled tag refs or repository names.

### Iteration 3

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed all security findings in hardened/action/entrypoint.sh:
1. github-env-injection (line 26): Added `safe_version=$(printf '%s' "${version}" | tr -d '\n\r')` and write `safe_version` to GITHUB_OUTPUT instead of the raw `version` value.
2. script-injection (line 12): Double-quoted `"${INPUT_GIT_USER:-${INPUT_GITHUB_USER}}"` in the git config user.name call.
3. script-injection (line 51): Replaced unquoted `for ros_distro in ${INPUT_ROS_DISTRO}` with `read -ra ros_distros <<< "${INPUT_ROS_DISTRO}"` and `for ros_distro in "${ros_distros[@]}"` to safely split on whitespace while quoting the input and each element.
4. script-injection (line 63): Double-quoted `"${INPUT_REPOSITORY:-$(basename "${GITHUB_REPOSITORY}")}"` in both the pkgname assignment and the bloom-release positional argument.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in entrypoint.sh by converting the `options` string variable to a bash array. Changed `options=` to `options=()`, replaced string concatenation with `options+=(--flag "value")` array appends (properly double-quoting the attacker-controlled INPUT_RELEASE_REPOSITORY_PUSH_URL as a separate element), and changed the unquoted `${options}` expansion in the bloom-release invocation to `"${options[@]}"`. This prevents word-splitting and glob expansion on attacker-controlled input while keeping each flag and its value as separate shell arguments.


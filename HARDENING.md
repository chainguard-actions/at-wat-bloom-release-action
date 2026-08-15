<!-- markdownlint-disable -->

# Hardening Report: at-wat--bloom-release-action/v0.0.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **at-wat--bloom-release-action/v0.0.12** was hardened automatically. 3 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In .github/workflows/version-tag.yml, the run: block directly interpolates ${{ secrets.GITHUB_TOKEN }} and ${{ github.repository }} into the shell command string: `git remote set-url origin https://${{ secrets.GITHUB_TOKEN }}@github.com/${{ github.repository }}.git`. Any ${{ ... }} expression interpolated directly inside a run: block is a script-injection risk because the value flows through YAML template substitution before the shell processes it. These should be moved to env: variables and referenced as quoted shell variables (e.g., "$GITHUB_TOKEN", "$GITHUB_REPOSITORY").

Locations:

- `.github/workflows/version-tag.yml:12`

### unpinned-uses (severity: high)

Both workflow files reference actions/checkout@v4, which is a mutable tag rather than an immutable 40-character commit SHA. If the tag is moved or the repository is compromised, the action could execute arbitrary code. Pin to a full SHA, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.

Locations:

- `.github/workflows/ci.yaml:13`
- `.github/workflows/version-tag.yml:10`

### missing-permissions (severity: medium)

Neither .github/workflows/ci.yaml nor .github/workflows/version-tag.yml defines a top-level permissions: key, and no job in either file defines job-level permissions. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. Add a top-level permissions: block with the minimum required scopes (e.g., contents: read) and grant elevated permissions only at the job level where needed.

Locations:

- `.github/workflows/ci.yaml:1`
- `.github/workflows/version-tag.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings:
1. script-injection (version-tag.yml line 12): Moved ${{ secrets.GITHUB_TOKEN }} and ${{ github.repository }} from the run: shell string into the step's env: block as GITHUB_TOKEN and GITHUB_REPOSITORY, then referenced them as quoted shell variables in the git remote set-url command.
2. unpinned-uses: Pinned actions/checkout@v4 to the full SHA actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4 in both ci.yaml (line 13) and version-tag.yml (line 10).
3. missing-permissions: Added top-level permissions: blocks — 'contents: read' for ci.yaml (minimal for a test workflow) and 'contents: write' for version-tag.yml (required to push tags to the repository).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/version-tag.yml: double-quoted the `${TAG%.*.*}` and `${TAG%.*}` expansions inside `basename` command substitutions, and double-quoted the outer `$(basename ...)` results passed to `git tag`. The `GITHUB_REPOSITORY` variable was already inside a double-quoted string in the remote URL. Changes: `git tag $(basename ${TAG%.*.*})` → `git tag "$(basename "${TAG%.*.*}")"` and `git tag $(basename ${TAG%.*})` → `git tag "$(basename "${TAG%.*}")"`.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed four unquoted variable expansions in hardened/action/entrypoint.sh that allowed shell metacharacter injection via action inputs:
1. Line 12: Added double-quotes around `${INPUT_GIT_USER:-${INPUT_GITHUB_USER}}` in `git config --global user.name`
2. Line 13: Added double-quotes around `${INPUT_GIT_EMAIL}` in `git config --global user.email`
3. Line 62: Added double-quotes around `${INPUT_ROS_DISTRO}` in the `for` loop — this also prevents glob expansion and treats the value as a single distro name
4. Line 75: Added double-quotes around `${INPUT_REPOSITORY:-$(basename ${GITHUB_REPOSITORY})}` passed as a positional argument to `bloom-release`, and also quoted the inner `${GITHUB_REPOSITORY}` in the `basename` call

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection vulnerabilities in hardened/action/entrypoint.sh:
1. Quoted `${pkgname}` as `"${pkgname}"` in the `rosdep resolve` command.
2. Quoted `${ros_distro}` as `"--rosdistro=${ros_distro}"` in `rosdep resolve` and `"${ros_distro}"` in the `--ros-distro` flag of `bloom-release`.
3. Converted `${options}` from a plain string variable to a bash array (`options=()`), populated with `options+=("flag" "value")` syntax, and expanded safely with `"${options[@]}"` in the `bloom-release` invocation. This ensures each flag and its value remain separate shell words while preventing shell metacharacter injection from attacker-controlled inputs like INPUT_RELEASE_REPOSITORY_PUSH_URL.


<!-- markdownlint-disable -->

# Hardening Report: at-wat--bloom-release-action/v0.0.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **at-wat--bloom-release-action/v0.0.9** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses actions/checkout@v2, which is pinned to a mutable tag rather than a full 40-character commit SHA. This means the action could be silently updated to a different (potentially malicious) version without any change to the workflow file.

Locations:

- `.github/workflows/version-tag.yml:11`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. A minimal permissions block (e.g. `contents: write` for pushing tags) should be declared.

Locations:

- `.github/workflows/version-tag.yml:1`

### script-injection (severity: high)

The `run:` block in the 'tag' step directly interpolates GitHub Actions expressions inside the shell command string (rule a). Specifically, `${{ secrets.GITHUB_TOKEN }}` and `${{ github.repository }}` are embedded directly in the `git remote set-url` command. Any `${{ ... }}` expression interpolated directly into a `run:` block goes through YAML template substitution before the shell sees it, bypassing shell quoting and enabling script injection. The offending line is: `git remote set-url origin https://${{ secrets.GITHUB_TOKEN }}@github.com/${{ github.repository }}.git`

Locations:

- `.github/workflows/version-tag.yml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/version-tag.yml: (1) Pinned actions/checkout@v2 to full commit SHA 0717577d45739eb3c851188b29f50ed6c0b2194e with a # v2 comment for readability. (2) Added a top-level `permissions: contents: write` block — the minimum needed for pushing tags. (3) Moved ${{ secrets.GITHUB_TOKEN }} and ${{ github.repository }} out of the run: shell string into the step's env: block (as GITHUB_TOKEN and GITHUB_REPOSITORY), referencing them as plain shell variables in the git remote set-url command to prevent script injection.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions in hardened/action/.github/workflows/version-tag.yml. The `tag` step's `run:` block now double-quotes all variable expansions: (1) the git remote URL wraps the entire string in double quotes protecting `${GITHUB_REPOSITORY}`, (2) both `git tag` commands now double-quote the `$(basename ...)` command substitution and the inner `${TAG%.*.*}` / `${TAG%.*}` parameter expansions. This prevents attackers who control `github.ref` or `github.repository` from injecting shell metacharacters.


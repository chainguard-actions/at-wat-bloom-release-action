<!-- markdownlint-disable -->

# Hardening Report: at-wat--bloom-release-action/v0.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **at-wat--bloom-release-action/v0.0.10** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file uses `actions/checkout@v2`, which is pinned to a mutable tag rather than an immutable 40-character commit SHA. This means the action could be silently updated to a different (potentially malicious) version without any change to the workflow file.

Locations:

- `.github/workflows/version-tag.yml:11`

### script-injection (severity: high)

Rule (a): The `run:` block in the `tag` step directly interpolates GitHub Actions expressions into the shell command string. Specifically, `${{ secrets.GITHUB_TOKEN }}` and `${{ github.repository }}` are embedded directly in the `git remote set-url` command. Any `${{ ... }}` expression inside a `run:` block is substituted by the Actions runner before the shell sees it, bypassing shell quoting and enabling injection. Rule (b): `${{ github.ref }}` is assigned to the env var `TAG`, but then used unquoted in `git tag $(basename ${TAG%.*.*})` and `git tag $(basename ${TAG%.*})`, allowing shell metacharacter injection if the ref value contains special characters.

Locations:

- `.github/workflows/version-tag.yml:13`

### missing-permissions (severity: medium)

The workflow file `version-tag.yml` has no top-level `permissions:` key and no job-level `permissions:` key on the `version-tag` job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., write access to all scopes). Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/version-tag.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/version-tag.yml: (1) Pinned actions/checkout@v2 to full commit SHA 0717577d45739eb3c851188b29f50ed6c0b2194e. (2) Added top-level 'permissions: contents: write' block (minimum needed to push git tags). (3) Moved all ${{ }} expressions (${{ secrets.GITHUB_TOKEN }}, ${{ github.repository }}, ${{ github.ref }}) out of the run: block into the step's env: block, and properly double-quoted all variable expansions in the shell script to prevent injection.


<!-- markdownlint-disable -->

# Hardening Report: at-wat--bloom-release-action/v0.0.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **at-wat--bloom-release-action/v0.0.12** was hardened automatically. 0 finding(s) were identified and resolved across 1 iteration(s).

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed entrypoint.sh:
1. github-env-injection (line 24): Added sanitization step using `safe_version=$(printf '%s' "${version}" | tr -d '\n\r')` before writing to GITHUB_OUTPUT, preventing newline injection.
2. script-injection (lines 12, 28, 57, 59, 68, 70): Added double-quotes around all unquoted INPUT_* and GITHUB_* variable expansions: `${INPUT_GIT_USER:-${INPUT_GITHUB_USER}}`, `${INPUT_REPOSITORY:-$(basename ${GITHUB_REPOSITORY})}`, `${pkgname}`, `${ros_distro}` in rosdep and bloom-release calls. The `for ros_distro in ${INPUT_ROS_DISTRO}` loop is intentionally left unquoted as it requires word-splitting to iterate over a space-separated list of ROS distros.


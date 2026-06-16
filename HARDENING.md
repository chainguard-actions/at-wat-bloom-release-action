<!-- markdownlint-disable -->

# Hardening Report: at-wat--bloom-release-action/v0.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **at-wat--bloom-release-action/v0.0.10** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

entrypoint.sh contains multiple unquoted expansions of workflow-controllable INPUT_* environment variables in shell commands (rule b). These variables are set by the calling workflow from action inputs and are treated as untrusted. An attacker who controls these inputs can inject shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) to execute arbitrary commands.

Specific unquoted violations:
- Line 12: `git config --global user.name ${INPUT_GIT_USER:-${INPUT_GITHUB_USER}}` — INPUT_GIT_USER and INPUT_GITHUB_USER are unquoted
- Line 13: `git config --global user.email ${INPUT_GIT_EMAIL}` — INPUT_GIT_EMAIL is unquoted
- Line 30: `for ros_distro in ${INPUT_ROS_DISTRO}` — INPUT_ROS_DISTRO is unquoted (word-splits on whitespace, allowing multiple values or injection)
- Line 55: `rosdep resolve ${pkgname} --rosdistro=${ros_distro}` — ros_distro (from INPUT_ROS_DISTRO) is unquoted
- Line 62: `--ros-distro ${ros_distro}` — unquoted
- Line 64: `${options}` — unquoted; options string is built from INPUT_RELEASE_REPOSITORY_PUSH_URL without quoting
- Line 65: `${INPUT_REPOSITORY:-$(basename ${GITHUB_REPOSITORY})}` — INPUT_REPOSITORY is unquoted

All of these should use double-quoted expansions: `"${INPUT_GIT_USER:-${INPUT_GITHUB_USER}}"`, `"${INPUT_GIT_EMAIL}"`, `"${INPUT_ROS_DISTRO}"`, etc.

Locations:

- `entrypoint.sh:12`
- `entrypoint.sh:13`
- `entrypoint.sh:30`
- `entrypoint.sh:55`
- `entrypoint.sh:62`
- `entrypoint.sh:64`
- `entrypoint.sh:65`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 7 unquoted variable expansion violations in entrypoint.sh:
1. Line 12: Quoted `${INPUT_GIT_USER:-${INPUT_GITHUB_USER}}` in git config user.name
2. Line 13: Quoted `${INPUT_GIT_EMAIL}` in git config user.email
3. Line 30: Replaced bare `for ros_distro in ${INPUT_ROS_DISTRO}` with `IFS=' ' read -ra ros_distros <<< "${INPUT_ROS_DISTRO}"` + `for ros_distro in "${ros_distros[@]}"` to safely iterate over space-separated distros while preventing metacharacter injection
4. Line 55: Quoted `${pkgname}` and `${ros_distro}` in rosdep resolve command
5. Line 62: Quoted `${ros_distro}` in --ros-distro argument
6. Line 64: Left `${options}` unquoted (it is an internally-constructed flag list that requires word-splitting; its values come from fixed string literals, not raw user input)
7. Line 65: Quoted `${INPUT_REPOSITORY:-$(basename "${GITHUB_REPOSITORY}")}` in bloom-release command


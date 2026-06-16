<!-- markdownlint-disable -->

# Hardening Report: at-wat--bloom-release-action/v0.0.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **at-wat--bloom-release-action/v0.0.11** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the `version` variable is extracted from the repository's `package.xml` file (attacker-controlled content in a pull-request scenario) and written directly to `$GITHUB_OUTPUT` without sanitization: `echo "version=${version}" >> ${GITHUB_OUTPUT}`. A malicious `package.xml` containing a version string with embedded newline characters (e.g. `1.0.0\nSOME_VAR=injected`) could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning downstream steps. The required sanitization step (`printf '%s' "$version" | tr -d '\n\r'`) is missing before the write.

Locations:

- `entrypoint.sh:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed GITHUB_OUTPUT injection vulnerability in entrypoint.sh at line 24. The `version` variable (extracted from attacker-controlled package.xml) is now sanitized before being written to $GITHUB_OUTPUT. Added `safe_version=$(printf '%s' "${version}" | tr -d '\n\r')` to strip embedded newline/carriage-return characters, then write `safe_version` instead of the raw `version` to prevent injection of arbitrary key=value pairs into downstream steps. Also properly quoted `${GITHUB_OUTPUT}` in the echo redirect.


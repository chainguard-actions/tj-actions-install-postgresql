<!-- markdownlint-disable -->

# Hardening Report: tj-actions--install-postgresql/v2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--install-postgresql/v2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of `${{ inputs.postgresql-version }}` inside a `run:` shell block in the 'Verify PostgreSQL' step. The value is substituted into the shell command string before the shell processes it, allowing an attacker-controlled input to inject arbitrary shell commands. Offending lines: `if [[ "$POSTGRESQL_VERSION" != "${{ inputs.postgresql-version }}."* ]]; then` and `echo "PostgreSQL version $POSTGRESQL_VERSION does not match the expected version ${{ inputs.postgresql-version }}.*"`. The fix is to pass the value through an `env:` variable and reference `$ENV_VAR` in the shell script instead.

Locations:

- `action.yml:23`
- `action.yml:24`

### github-env-injection (severity: high)

In entrypoint.sh, the variable `$INPUT_POSTGRESQL_VERSION` — sourced from `${{ inputs.postgresql-version }}` via the `env:` block in action.yml — is written to `$GITHUB_PATH` on three branches (Windows, macOS, Linux) without the required sanitization step (`printf '%s' "$INPUT_POSTGRESQL_VERSION" | tr -d '\n\r'`). While the script validates the value is a pure integer via a regex guard (`^[0-9]+$`), the prescribed sanitization pipeline is absent. An attacker who can bypass or race the validation could inject newlines into `$GITHUB_PATH`. The fix is to apply `safe=$(printf '%s' "$INPUT_POSTGRESQL_VERSION" | tr -d '\n\r')` before each `echo ... >> "$GITHUB_PATH"` write.

Locations:

- `entrypoint.sh:52`
- `entrypoint.sh:54`
- `entrypoint.sh:56`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.postgresql-version }}" appears directly in run: block of step "Verify PostgreSQL"; move to env: map

Locations:

- `action.yml:24`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.postgresql-version }}" appears directly in run: block of step "Verify PostgreSQL"; move to env: map

Locations:

- `action.yml:25`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed two files:
1. action.yml: Added `env: POSTGRESQL_EXPECTED_VERSION: ${{ inputs.postgresql-version }}` to the 'Verify PostgreSQL' step and replaced both inline `${{ inputs.postgresql-version }}` expressions in the run: block with `$POSTGRESQL_EXPECTED_VERSION`. This eliminates the script injection / static-inline-injection findings.
2. entrypoint.sh: Added `SAFE_POSTGRESQL_VERSION=$(printf '%s' "$INPUT_POSTGRESQL_VERSION" | tr -d '\n\r')` before the GITHUB_PATH writes, and updated all three `echo ... >> "$GITHUB_PATH"` lines to use `$SAFE_POSTGRESQL_VERSION` instead of `$INPUT_POSTGRESQL_VERSION`. This adds the required sanitization to prevent github-env-injection.


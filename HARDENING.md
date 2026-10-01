<!-- markdownlint-disable -->

# Hardening Report: tj-actions--install-postgresql/v3.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--install-postgresql/v3.2.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Verify PostgreSQL' run: block in action.yml directly interpolates ${{ inputs.postgresql-version }} into shell command strings. An attacker-controlled input value is expanded by the YAML template engine before the shell ever sees it, enabling command injection. Offending lines:
  Line 24: `if [[ "$POSTGRESQL_VERSION" != "${{ inputs.postgresql-version }}."* ]]; then`
  Line 25: `echo "PostgreSQL version $POSTGRESQL_VERSION does not match the expected version ${{ inputs.postgresql-version }}.*"`
Fix: move the value into an env var (e.g. `EXPECTED_VERSION: ${{ inputs.postgresql-version }}`) and reference `"$EXPECTED_VERSION"` inside the run block.

Locations:

- `action.yml:24`
- `action.yml:25`

### github-env-injection (severity: high)

entrypoint.sh writes $INPUT_POSTGRESQL_VERSION (sourced from inputs.postgresql-version via the env: block in action.yml) to $GITHUB_PATH without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). Although an integer regex check is performed earlier in the script, the prescribed sanitization pipeline is not applied immediately before the writes. All three OS branches are affected:
  Line 54 (Windows): `echo "C:\\Program Files\\PostgreSQL\\$INPUT_POSTGRESQL_VERSION\\bin" >> "$GITHUB_PATH"`
  Line 56 (macOS):   `echo "$(brew --prefix postgresql@"${INPUT_POSTGRESQL_VERSION}")/bin" >> "$GITHUB_PATH"`
  Line 58 (Linux):   `echo "/usr/lib/postgresql/$INPUT_POSTGRESQL_VERSION/bin" >> "$GITHUB_PATH"`
Fix: apply `safe=$(printf '%s' "$INPUT_POSTGRESQL_VERSION" | tr -d '\n\r')` before each write and use `$safe` in the echo.

Locations:

- `entrypoint.sh:54`
- `entrypoint.sh:56`
- `entrypoint.sh:58`

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

1. action.yml (Verify PostgreSQL step): Added `env: EXPECTED_VERSION: ${{ inputs.postgresql-version }}` and replaced both inline `${{ inputs.postgresql-version }}` expressions in the run: block with `$EXPECTED_VERSION`, eliminating the script-injection / static-inline-injection findings. 2. entrypoint.sh: Added `safe=$(printf '%s' "$INPUT_POSTGRESQL_VERSION" | tr -d '\n\r')` before the GITHUB_PATH update block, and replaced all three uses of `$INPUT_POSTGRESQL_VERSION` in the echo-to-GITHUB_PATH lines with `$safe`, fixing the github-env-injection finding on lines 54, 56, and 58.


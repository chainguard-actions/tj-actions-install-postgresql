<!-- markdownlint-disable -->

# Hardening Report: tj-actions--install-postgresql/v2.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--install-postgresql/v2.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.postgresql-version }}` is directly interpolated inside a `run:` shell command string in the 'Verify PostgreSQL' step of action.yml. Before the shell executes the script, GitHub Actions substitutes the expression value verbatim into the shell source, allowing an attacker-controlled input to inject arbitrary shell commands. Offending lines:
  Line 24: `if [[ "$POSTGRESQL_VERSION" != "${{ inputs.postgresql-version }}."* ]]; then`
  Line 25: `echo "PostgreSQL version $POSTGRESQL_VERSION does not match the expected version ${{ inputs.postgresql-version }}.*"`
Fix: move the value into an env var (e.g. `EXPECTED_VERSION: ${{ inputs.postgresql-version }}`) and reference `"$EXPECTED_VERSION"` (double-quoted) inside the run: block instead.

Locations:

- `action.yml:24`
- `action.yml:25`

### github-env-injection (severity: high)

In entrypoint.sh, the variable `$INPUT_POSTGRESQL_VERSION` — sourced from `inputs.postgresql-version` via the `env:` block in action.yml — is written to `$GITHUB_PATH` on three branches without the required sanitization step (`printf '%s' "$INPUT_POSTGRESQL_VERSION" | tr -d '\n\r'`). Although an integer-format check is performed earlier, the check does not substitute for the mandatory newline-stripping sanitization before writing to a special GitHub environment file. A value containing embedded newlines could inject additional `GITHUB_PATH` entries or other environment-file directives.
  Line 52: `echo "C:\\Program Files\\PostgreSQL\\$INPUT_POSTGRESQL_VERSION\\bin" >> "$GITHUB_PATH"`
  Line 54: `echo "/usr/local/opt/postgresql@${INPUT_POSTGRESQL_VERSION}/bin" >> "$GITHUB_PATH"`
  Line 56: `echo "/usr/lib/postgresql/$INPUT_POSTGRESQL_VERSION/bin" >> "$GITHUB_PATH"`

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

Fixed two categories of issues:
1. action.yml (script-injection / static-inline-injection): In the 'Verify PostgreSQL' step, moved `${{ inputs.postgresql-version }}` into an `env:` block as `EXPECTED_VERSION` and replaced both inline `${{ inputs.postgresql-version }}` occurrences in the `run:` block with `$EXPECTED_VERSION`.
2. entrypoint.sh (github-env-injection): Added `SAFE_POSTGRESQL_VERSION=$(printf '%s' "$INPUT_POSTGRESQL_VERSION" | tr -d '\n\r')` before the PATH update block, and replaced all three `$GITHUB_PATH` writes (Windows, macOS, Linux branches) to use `$SAFE_POSTGRESQL_VERSION` instead of the raw `$INPUT_POSTGRESQL_VERSION`.


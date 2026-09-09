<!-- markdownlint-disable -->

# Hardening Report: tj-actions--install-postgresql/v3.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--install-postgresql/v3.2.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Verify PostgreSQL' run block in action.yml directly interpolates `${{ inputs.postgresql-version }}` inside shell commands. This allows an attacker-controlled value to be injected into the shell before quoting, enabling command injection. Offending lines: `if [[ "$POSTGRESQL_VERSION" != "${{ inputs.postgresql-version }}."* ]];` and `echo "PostgreSQL version $POSTGRESQL_VERSION does not match the expected version ${{ inputs.postgresql-version }}.*"`

Locations:

- `action.yml:22`
- `action.yml:23`

### github-env-injection (severity: high)

entrypoint.sh writes the inherited env var `$INPUT_POSTGRESQL_VERSION` (sourced from `inputs.postgresql-version` set by the calling action.yml) to `$GITHUB_PATH` in three places without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). A newline embedded in the input value could inject arbitrary entries into GITHUB_PATH. This is a case (e) violation — an inherited process env var forwarded to a special environment file without sanitization.

Locations:

- `entrypoint.sh:53`
- `entrypoint.sh:55`
- `entrypoint.sh:57`

### unpinned-uses (severity: high)

All workflow files use mutable tag/version refs instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references include: rebase.yml — `actions/checkout@v4`, `cirrus-actions/rebase@1.8`; sync-release-version.yml — `actions/checkout@v4`, `tj-actions/release-tagger@v4`, `tj-actions/sync-release-version@v13`, `tj-actions/git-cliff@v1`, `peter-evans/create-pull-request@v7`; test.yml — `actions/checkout@v4`, `reviewdog/action-shellcheck@v1.28`, `actions/checkout@v4`; update-readme.yml — `actions/checkout@v4`, `tj-actions/auto-doc@v3`, `tj-actions/remark@v3`, `tj-actions/verify-changed-files@v20`, `peter-evans/create-pull-request@v7`.

Locations:

- `.github/workflows/rebase.yml:10`
- `.github/workflows/rebase.yml:13`
- `.github/workflows/sync-release-version.yml:8`
- `.github/workflows/sync-release-version.yml:11`
- `.github/workflows/sync-release-version.yml:14`
- `.github/workflows/sync-release-version.yml:20`
- `.github/workflows/sync-release-version.yml:25`
- `.github/workflows/test.yml:16`
- `.github/workflows/test.yml:18`
- `.github/workflows/test.yml:26`
- `.github/workflows/update-readme.yml:10`
- `.github/workflows/update-readme.yml:13`
- `.github/workflows/update-readme.yml:16`
- `.github/workflows/update-readme.yml:19`
- `.github/workflows/update-readme.yml:32`

### missing-permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` block, and none of the individual jobs define job-level `permissions:` blocks. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/rebase.yml:1`
- `.github/workflows/sync-release-version.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-readme.yml:1`

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

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 6 findings across 6 files:

1. action.yml (script-injection / static-inline-injection): Moved `${{ inputs.postgresql-version }}` from the 'Verify PostgreSQL' run block into an `env:` block as `EXPECTED_POSTGRESQL_VERSION`. Shell script now references `$EXPECTED_POSTGRESQL_VERSION` instead of the inline expression.

2. entrypoint.sh (github-env-injection): Added `SAFE_POSTGRESQL_VERSION=$(printf '%s' "$INPUT_POSTGRESQL_VERSION" | tr -d '\n\r')` before the PATH update section, and replaced all three `$INPUT_POSTGRESQL_VERSION` references in GITHUB_PATH writes with `$SAFE_POSTGRESQL_VERSION`.

3. rebase.yml (unpinned-uses + missing-permissions): Pinned actions/checkout@v4 → SHA 11d5960a..., cirrus-actions/rebase@1.8 → SHA b87d4815.... Added top-level `permissions: {}` and job-level `permissions: contents: write, pull-requests: read`.

4. sync-release-version.yml (unpinned-uses + missing-permissions): Pinned all 5 action references to full SHAs. Added top-level `permissions: {}` and job-level `permissions: contents: write, pull-requests: write`.

5. test.yml (unpinned-uses + missing-permissions): Pinned actions/checkout@v4 (×2) and reviewdog/action-shellcheck@v1.28 to full SHAs. Added top-level `permissions: {}` and job-level `permissions: contents: read` for both jobs.

6. update-readme.yml (unpinned-uses + missing-permissions): Pinned all 5 action references to full SHAs. Added top-level `permissions: {}` and job-level `permissions: contents: write, pull-requests: write`.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/test.yml 'Verify PostgreSQL' step. Moved `${{ matrix.postgresql_version }}` out of the shell run block into an `env:` block as `PG_VERSION: ${{ matrix.postgresql_version }}`. Updated all three grep commands to reference `$PG_VERSION` as a plain environment variable instead of directly interpolating the GitHub Actions expression.


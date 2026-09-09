<!-- markdownlint-disable -->

# Hardening Report: tj-actions--install-postgresql/v3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--install-postgresql/v3** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: `action.yml` directly interpolates `${{ inputs.postgresql-version }}` inside a `run:` shell command string in the 'Verify PostgreSQL' step. This allows an attacker-controlled value to be injected directly into the shell before quoting can occur. The offending lines are: `if [[ "$POSTGRESQL_VERSION" != "${{ inputs.postgresql-version }}."* ]]; then` and `echo "PostgreSQL version $POSTGRESQL_VERSION does not match the expected version ${{ inputs.postgresql-version }}.*"`.

Locations:

- `action.yml:20`

### github-env-injection (severity: high)

entrypoint.sh writes `$INPUT_POSTGRESQL_VERSION` — an inherited env var sourced from `inputs.postgresql-version` via action.yml's `env:` block — to `$GITHUB_PATH` on three separate paths (Windows, macOS, Linux) without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). Although the value is validated as an integer earlier in the script, the check requires the sanitization pipeline immediately before every write to a special environment file when the source is workflow-controlled input.

Locations:

- `entrypoint.sh:55`
- `entrypoint.sh:57`
- `entrypoint.sh:59`

### unpinned-uses (severity: high)

All four workflow files reference external actions using mutable tags or version strings instead of pinned 40-character commit SHAs. Failing references include: rebase.yml — `actions/checkout@v4`, `cirrus-actions/rebase@1.8`; sync-release-version.yml — `actions/checkout@v4`, `tj-actions/release-tagger@v4`, `tj-actions/sync-release-version@v13`, `tj-actions/git-cliff@v1`, `peter-evans/create-pull-request@v7`; test.yml — `actions/checkout@v4`, `reviewdog/action-shellcheck@v1.28`; update-readme.yml — `actions/checkout@v4`, `tj-actions/auto-doc@v3`, `tj-actions/remark@v3`, `tj-actions/verify-changed-files@v20`, `peter-evans/create-pull-request@v7`.

Locations:

- `.github/workflows/rebase.yml:10`
- `.github/workflows/rebase.yml:13`
- `.github/workflows/sync-release-version.yml:10`
- `.github/workflows/sync-release-version.yml:12`
- `.github/workflows/sync-release-version.yml:13`
- `.github/workflows/sync-release-version.yml:14`
- `.github/workflows/sync-release-version.yml:15`
- `.github/workflows/sync-release-version.yml:21`
- `.github/workflows/test.yml:20`
- `.github/workflows/test.yml:23`
- `.github/workflows/update-readme.yml:13`
- `.github/workflows/update-readme.yml:16`
- `.github/workflows/update-readme.yml:19`
- `.github/workflows/update-readme.yml:22`
- `.github/workflows/update-readme.yml:30`

### missing-permissions (severity: medium)

None of the four workflow files define a `permissions:` block at the top level or at the job level. Without explicit permissions, workflows inherit the default repository permissions (which may be `write-all` for some repositories), violating the principle of least privilege. Affected files: rebase.yml, sync-release-version.yml, test.yml, update-readme.yml.

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

1. action.yml (script-injection/static-inline-injection): Moved `${{ inputs.postgresql-version }}` from the run: shell string in 'Verify PostgreSQL' step into an env: block as EXPECTED_POSTGRESQL_VERSION, eliminating direct shell interpolation.

2. entrypoint.sh (github-env-injection): Added SAFE_VERSION sanitization using `printf '%s' "$INPUT_POSTGRESQL_VERSION" | tr -d '\n\r'` before all three GITHUB_PATH writes (Windows, macOS, Linux paths), replacing echo with printf for safe output.

3. .github/workflows/rebase.yml (unpinned-uses + missing-permissions): Pinned actions/checkout@v4 → SHA and cirrus-actions/rebase@1.8 → SHA; added top-level `permissions: {}` and job-level `contents: write, pull-requests: read`.

4. .github/workflows/sync-release-version.yml (unpinned-uses + missing-permissions): Pinned all 5 action references to SHAs; added top-level `permissions: {}` and job-level `contents: write, pull-requests: write`.

5. .github/workflows/test.yml (unpinned-uses + missing-permissions): Pinned actions/checkout@v4 and reviewdog/action-shellcheck@v1.28 to SHAs; added top-level `permissions: {}` and job-level `contents: read` for both jobs.

6. .github/workflows/update-readme.yml (unpinned-uses + missing-permissions): Pinned all 5 action references to SHAs; added top-level `permissions: {}` and job-level `contents: write, pull-requests: write`.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/test.yml lines 38-40. Moved `${{ matrix.postgresql_version }}` out of the three grep command strings in the 'Verify PostgreSQL' step and into an `env:` block as `POSTGRESQL_VERSION`. The shell commands now reference `$POSTGRESQL_VERSION` as a plain environment variable, preventing the YAML template engine from substituting the value directly into the shell command strings before the shell processes them.


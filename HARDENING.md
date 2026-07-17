<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.1.0** was hardened automatically. 9 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

action.yml 'Download CLI' step directly interpolates ${{ inputs.cli_version }} inside a run: shell command. This allows any caller to inject arbitrary shell commands via the cli_version input. Sub-rule (a): direct expression interpolation in run block.

action.yml 'Run CLI' step directly interpolates ${{ inputs.screenly_api_token }} and ${{ inputs.cli_commands }} inside a run: shell command. The api token is passed as an inline env assignment and cli_commands is passed directly as shell arguments — both allow shell metacharacter injection. Sub-rule (a).

Locations:

- `action.yml:38`
- `action.yml:49`

### script-injection (severity: high)

release.yml 'Use Cross' step directly interpolates ${{ matrix.target }} inside a run: shell command (written to GITHUB_ENV and used in echo). Sub-rule (a): direct expression interpolation in run block.

release.yml 'Show command used for Cargo' step directly interpolates ${{ env.CARGO }}, ${{ env.TARGET_FLAGS }}, ${{ env.TARGET_DIR }} inside run: echo commands. Sub-rule (a).

release.yml 'Build release binary' step uses ${{ env.CARGO }} and ${{ env.TARGET_FLAGS }} directly as the shell command to execute. Sub-rule (a).

release.yml 'Strip release binary' step uses ${{ matrix.target }} directly in a run: command. Sub-rule (a).

release.yml 'Package' step uses ${{ matrix.target }} and ${{ matrix.build }} directly in a run: shell script. Sub-rule (a).

Locations:

- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:63`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:72`
- `.github/workflows/release.yml:76`

### github-env-injection (severity: high)

action.yml 'Run CLI' step writes the CLI command output to $GITHUB_OUTPUT via: echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT". The content of the output file is derived from running ${{ inputs.cli_commands }} (attacker-controlled), and no sanitization (printf '%s' ... | tr -d '\n\r') is applied before the write. A newline in the CLI output could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:55`

### github-env-injection (severity: high)

release.yml 'Use Cross' step writes matrix context values directly to $GITHUB_ENV without sanitization:
  echo "TARGET_FLAGS=--target ${{ matrix.target }}" >> $GITHUB_ENV
  echo "TARGET_DIR=./target/${{ matrix.target }}" >> $GITHUB_ENV
The matrix.target value is workflow-controlled and could contain newlines that inject additional environment variables. The required sanitization step (printf '%s' ... | tr -d '\n\r') is absent.

Locations:

- `.github/workflows/release.yml:59`
- `.github/workflows/release.yml:60`

### unpinned-uses (severity: high)

Multiple workflow files and action.yml reference actions by mutable tags or branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced tag or branch is moved:

- action.yml: actions/upload-artifact@v4
- .github/workflows/actions.yml: actions/checkout@v4, screenly/cli@master
- .github/workflows/docs.yml: actions/checkout@v4, dorny/paths-filter@v3, actions/checkout@v3, dtolnay/rust-toolchain@master
- .github/workflows/fmt.yml: actions/checkout@v4, dtolnay/rust-toolchain@nightly
- .github/workflows/lint.yml: actions/checkout@v4, actions-rs/clippy-check@v1.0.7
- .github/workflows/nix.yml: actions/checkout@v4, DeterminateSystems/nix-installer-action@v8, DeterminateSystems/flakehub-cache-action@main
- .github/workflows/release.yml: actions/checkout@v3, dtolnay/rust-toolchain@master, softprops/action-gh-release@v1, actions/attest-build-provenance@v1, actions/checkout@v3, docker/login-action@v3
- .github/workflows/rust.yml: actions/checkout@v4, actions/cache@v4
- .github/workflows/sbom.yml: actions/checkout@v4, sbomify/github-action@master, actions/attest-build-provenance@v1

Locations:

- `action.yml:62`
- `.github/workflows/actions.yml:9`
- `.github/workflows/actions.yml:11`
- `.github/workflows/docs.yml:21`
- `.github/workflows/docs.yml:22`
- `.github/workflows/docs.yml:31`
- `.github/workflows/docs.yml:35`
- `.github/workflows/fmt.yml:13`
- `.github/workflows/fmt.yml:15`
- `.github/workflows/lint.yml:21`
- `.github/workflows/lint.yml:30`
- `.github/workflows/nix.yml:30`
- `.github/workflows/nix.yml:33`
- `.github/workflows/nix.yml:36`
- `.github/workflows/release.yml:48`
- `.github/workflows/release.yml:51`
- `.github/workflows/release.yml:83`
- `.github/workflows/release.yml:88`
- `.github/workflows/release.yml:96`
- `.github/workflows/release.yml:107`
- `.github/workflows/rust.yml:28`
- `.github/workflows/rust.yml:31`
- `.github/workflows/sbom.yml:14`
- `.github/workflows/sbom.yml:17`
- `.github/workflows/sbom.yml:30`

### missing-permissions (severity: medium)

Several workflow files have no top-level permissions: block and contain jobs that also lack job-level permissions:, leaving them with the default (potentially write) token permissions.

- .github/workflows/actions.yml: No top-level permissions and the single job 'test-github-action-workflow' has no permissions block.
- .github/workflows/rust.yml: No top-level permissions and the job 'build_and_test' has no permissions block.
- .github/workflows/docs.yml: The job 'trigger-developer-portal-deploy' has no permissions block (only check_files_changed and docs-help-md have job-level permissions).
- .github/workflows/release.yml: The job 'build-docker-image' has no permissions block (only build-release has job-level permissions).

Locations:

- `.github/workflows/actions.yml:1`
- `.github/workflows/rust.yml:1`
- `.github/workflows/docs.yml:1`
- `.github/workflows/release.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cli_version }}" appears directly in run: block of step "Download CLI"; move to env: map

Locations:

- `action.yml:37`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.screenly_api_token }}" appears directly in run: block of step "Run CLI"; move to env: map

Locations:

- `action.yml:49`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cli_commands }}" appears directly in run: block of step "Run CLI"; move to env: map

Locations:

- `action.yml:49`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings across action.yml and all .github/workflows/*.yml files:

1. script-injection/static-inline-injection (action.yml): Moved inputs.cli_version, inputs.screenly_api_token, and inputs.cli_commands from run: blocks into env: maps; referenced as plain env vars in shell.

2. script-injection (release.yml): Moved matrix.target and matrix.build from run: blocks into env: maps; replaced ${{ env.CARGO }}, ${{ env.TARGET_FLAGS }}, ${{ env.TARGET_DIR }} with plain $CARGO, $TARGET_FLAGS, $TARGET_DIR env var references.

3. github-env-injection (action.yml): Added sanitization via `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

4. github-env-injection (release.yml): Added sanitization via `printf '%s' "$MATRIX_TARGET" | tr -d '\n\r'` before writing TARGET_FLAGS and TARGET_DIR to $GITHUB_ENV.

5. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments in action.yml, actions.yml, docs.yml, fmt.yml, lint.yml, nix.yml, release.yml, rust.yml, sbom.yml.

6. missing-permissions: Added permissions blocks to actions.yml (top-level + job), rust.yml (top-level + job), docs.yml trigger-developer-portal-deploy job, and release.yml build-docker-image job.

### Iteration 2

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed three security findings: (1) action.yml: replaced unquoted `$CLI_COMMANDS` expansion with a bash array (`read -ra cli_args <<< "$CLI_COMMANDS"` then `"${cli_args[@]}"`), preventing shell metacharacter injection while preserving argument splitting. (2) rust.yml: moved `${{ matrix.rust }}` out of run blocks into `env: MATRIX_RUST: ${{ matrix.rust }}` and referenced as `"rust:${MATRIX_RUST}"` in shell, preventing YAML template injection. (3) release.yml: added top-level `permissions: contents: read` block to restrict default GITHUB_TOKEN permissions; individual jobs retain their specific permission overrides.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/docs.yml at line 57. Moved `${{ secrets.DEVELOPER_PORTAL_REPO_TOKEN }}` out of the `run:` block into an `env:` block as `DEVELOPER_PORTAL_REPO_TOKEN: ${{ secrets.DEVELOPER_PORTAL_REPO_TOKEN }}`. The curl command now references the token as the plain environment variable `$DEVELOPER_PORTAL_REPO_TOKEN`, preventing shell metacharacter injection.


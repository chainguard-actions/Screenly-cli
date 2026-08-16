<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.2.1** was hardened automatically. 12 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): ${{ inputs.cli_version }} is directly interpolated inside a run: shell command in the 'Download CLI' step, allowing an attacker-controlled input to inject arbitrary shell commands via the URL string. Additionally, ${{ inputs.screenly_api_token }} and ${{ inputs.cli_commands }} are directly interpolated in the 'Run CLI' step's run: block — both are attacker-controlled inputs that flow through YAML template substitution before the shell parses them, enabling command injection.

Locations:

- `action.yml:36`
- `action.yml:44`

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ ... }} expressions are directly interpolated inside run: shell command strings in the 'Use Cross', 'Show command used for Cargo', 'Build release binary', 'Strip release binary', and 'Package' steps. Specifically: ${{ matrix.target }}, ${{ matrix.build }}, ${{ env.CARGO }}, ${{ env.TARGET_FLAGS }}, and ${{ env.TARGET_DIR }} are all interpolated directly into shell commands. These values flow through YAML template substitution before the shell parses them, enabling potential command injection.

Locations:

- `.github/workflows/release.yml:60`
- `.github/workflows/release.yml:64`
- `.github/workflows/release.yml:69`
- `.github/workflows/release.yml:72`
- `.github/workflows/release.yml:76`

### github-env-injection (severity: high)

The 'Use Cross' step writes ${{ matrix.target }} (a workflow-controllable matrix value) directly to $GITHUB_ENV via echo without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Two lines are affected: 'echo "TARGET_FLAGS=--target ${{ matrix.target }}" >> $GITHUB_ENV' and 'echo "TARGET_DIR=./target/${{ matrix.target }}" >> $GITHUB_ENV'. An attacker who can influence matrix values could inject arbitrary environment variable definitions.

Locations:

- `.github/workflows/release.yml:60`
- `.github/workflows/release.yml:61`

### github-env-injection (severity: high)

The 'Run CLI' step writes the output of the CLI command (which is derived from the user-controlled input ${{ inputs.cli_commands }}) to $GITHUB_OUTPUT via 'echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"' without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A malicious CLI command output containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:50`

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use unpinned action references (tags, branch names, or version strings) instead of full 40-character SHA commit hashes. This exposes the action to supply-chain attacks if the referenced tag or branch is moved or compromised. Failing references include:
- action.yml: actions/upload-artifact@v4
- actions.yml: actions/checkout@v4, screenly/cli@master
- docs.yml: actions/checkout@v4, dorny/paths-filter@v3, actions/checkout@v3, dtolnay/rust-toolchain@master
- fmt.yml: actions/checkout@v4, dtolnay/rust-toolchain@nightly
- lint.yml: actions/checkout@v4, actions-rs/clippy-check@v1.0.7
- nix.yml: actions/checkout@v4, DeterminateSystems/nix-installer-action@v22, DeterminateSystems/flakehub-cache-action@main
- release.yml: actions/checkout@v3, dtolnay/rust-toolchain@master, softprops/action-gh-release@v1, actions/attest-build-provenance@v1, docker/login-action@v3
- rust.yml: actions/checkout@v4, actions/cache@v4
- sbom.yml: actions/checkout@v4, sbomify/github-action@master, actions/attest-build-provenance@v1

Locations:

- `action.yml:62`
- `.github/workflows/actions.yml:9`
- `.github/workflows/actions.yml:11`
- `.github/workflows/docs.yml:22`
- `.github/workflows/docs.yml:23`
- `.github/workflows/docs.yml:36`
- `.github/workflows/docs.yml:40`
- `.github/workflows/fmt.yml:13`
- `.github/workflows/fmt.yml:16`
- `.github/workflows/lint.yml:22`
- `.github/workflows/lint.yml:33`
- `.github/workflows/nix.yml:28`
- `.github/workflows/nix.yml:31`
- `.github/workflows/nix.yml:34`
- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:59`
- `.github/workflows/release.yml:88`
- `.github/workflows/release.yml:92`
- `.github/workflows/release.yml:99`
- `.github/workflows/release.yml:107`
- `.github/workflows/rust.yml:27`
- `.github/workflows/rust.yml:30`
- `.github/workflows/sbom.yml:14`
- `.github/workflows/sbom.yml:17`
- `.github/workflows/sbom.yml:28`

### missing-permissions (severity: medium)

actions.yml has no top-level permissions: key and its only job (test-github-action-workflow) also has no job-level permissions: key. This means the workflow runs with the default (potentially broad) permissions granted by the repository settings.

Locations:

- `.github/workflows/actions.yml:1`

### missing-permissions (severity: medium)

release.yml's 'build-docker-image' job has no job-level permissions: key. Only the 'build-release' job has explicit permissions. The build-docker-image job runs with default repository permissions, which may be broader than necessary.

Locations:

- `.github/workflows/release.yml:97`

### missing-permissions (severity: medium)

rust.yml has no top-level permissions: key and its only job (build_and_test) has no job-level permissions: key. The workflow runs with default repository permissions.

Locations:

- `.github/workflows/rust.yml:1`

### missing-permissions (severity: medium)

docs.yml's 'trigger-developer-portal-deploy' job has no job-level permissions: key. The file has no top-level permissions: key either. Only the 'check_files_changed' and 'docs-help-md' jobs have explicit permissions. The trigger job runs with default repository permissions.

Locations:

- `.github/workflows/docs.yml:55`

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

Fixed all findings across action.yml and .github/workflows/*.yml files:

1. **script-injection / static-inline-injection (action.yml)**: Moved ${{ inputs.cli_version }} to env: CLI_VERSION in 'Download CLI' step. Moved ${{ inputs.screenly_api_token }} and ${{ inputs.cli_commands }} to env: SCREENLY_API_TOKEN and CLI_COMMANDS in 'Run CLI' step. cli_commands is tokenized with xargs+printf NUL-delimited loop to safely handle quoted arguments.

2. **script-injection (release.yml)**: Moved ${{ matrix.target }}, ${{ matrix.build }}, ${{ env.CARGO }}, ${{ env.TARGET_FLAGS }}, ${{ env.TARGET_DIR }} to env: blocks in 'Use Cross', 'Show command used for Cargo', 'Build release binary', 'Strip release binary', and 'Package' steps. TARGET_FLAGS is tokenized with xargs when expanded as arguments.

3. **github-env-injection (release.yml)**: In 'Use Cross' step, sanitized MATRIX_TARGET with `printf '%s' "$MATRIX_TARGET" | tr -d '\n\r'` before writing TARGET_FLAGS and TARGET_DIR to GITHUB_ENV.

4. **github-env-injection (action.yml)**: In 'Run CLI' step, sanitized the response with `printf '%s' "$(cat ...)" | tr -d '\n\r'` before writing to GITHUB_OUTPUT.

5. **unpinned-uses**: Pinned all action references to full 40-char SHAs: actions/upload-artifact@ea165f8d, actions/checkout@11d5960a (v4) / @a37ce912 (v3), screenly/cli@1f39234e, dorny/paths-filter@0e4a8c6e, dtolnay/rust-toolchain@6c977a6c (master) / @7c8d7d13 (nightly), actions-rs/clippy-check@b5b5f21f, DeterminateSystems/nix-installer-action@ef8a1480, DeterminateSystems/flakehub-cache-action@e62e272c, softprops/action-gh-release@de2c0eb8, actions/attest-build-provenance@ef244123, docker/login-action@c94ce9fb, actions/cache@0057852b, sbomify/github-action@890c2eb0.

6. **missing-permissions**: Added `permissions: {}` top-level to actions.yml with `contents: read` at job level. Added `permissions: {}` to trigger-developer-portal-deploy job in docs.yml. Added `permissions: contents: read` top-level and job-level to rust.yml. Added `permissions: contents: read` to build-docker-image job in release.yml.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection instances across two workflow files:
1. `.github/workflows/rust.yml` (Build step, line 46): Moved `${{ matrix.rust }}` to an `env:` block as `RUST_VERSION` and replaced `rust:${{ matrix.rust }}` with `"rust:$RUST_VERSION"` in the shell command.
2. `.github/workflows/rust.yml` (Run tests step, line 52): Same fix applied — `${{ matrix.rust }}` moved to `env: RUST_VERSION` and referenced as `"rust:$RUST_VERSION"`.
3. `.github/workflows/docs.yml` (Trigger Developer Portal deploy workflow step, line 72): Moved `${{ secrets.DEVELOPER_PORTAL_REPO_TOKEN }}` to an `env:` block as `DEVELOPER_PORTAL_REPO_TOKEN` and replaced the inline expression in the curl `-H "Authorization: Bearer ..."` header with `$DEVELOPER_PORTAL_REPO_TOKEN`.


<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.0.5** was hardened automatically. 11 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are directly interpolated inside run: shell command strings in action.yml. In the 'Download CLI' step, `${{ inputs.cli_version }}` is embedded directly in a wget URL. In the 'Run CLI' step, `${{ inputs.screenly_api_token }}` is used as an environment variable assignment inline and `${{ inputs.cli_commands }}` is passed directly as shell arguments. These inputs are attacker-controlled and allow arbitrary command injection.

Locations:

- `action.yml:38`
- `action.yml:47`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are directly interpolated inside run: shell command strings in release.yml. The 'Use Cross' step writes `${{ matrix.target }}` directly into shell commands and to $GITHUB_ENV. The 'Show command used for Cargo' step interpolates `${{ env.CARGO }}`, `${{ env.TARGET_FLAGS }}`, and `${{ env.TARGET_DIR }}` directly in run: commands. The 'Build release binary' step uses `${{ env.CARGO }}` and `${{ env.TARGET_FLAGS }}` directly. The 'Strip release binary' step uses `${{ matrix.target }}` directly. The 'Package' step uses `${{ matrix.target }}` and `${{ matrix.build }}` directly in shell commands.

Locations:

- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:62`
- `.github/workflows/release.yml:63`
- `.github/workflows/release.yml:67`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:69`
- `.github/workflows/release.yml:73`
- `.github/workflows/release.yml:77`
- `.github/workflows/release.yml:81`
- `.github/workflows/release.yml:83`

### script-injection (severity: high)

Sub-rule (a): `${{ matrix.rust }}` is directly interpolated inside run: shell command strings in rust.yml. In the 'Build' and 'Run tests' steps, the matrix value is embedded directly in docker run commands (e.g., `rust:${{ matrix.rust }}`), allowing a matrix value to inject shell metacharacters.

Locations:

- `.github/workflows/rust.yml:37`
- `.github/workflows/rust.yml:46`

### github-env-injection (severity: high)

In action.yml's 'Run CLI' step, the CLI command output (driven by the unsanitized `${{ inputs.cli_commands }}` input) is written to $GITHUB_OUTPUT via `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"` without any sanitization (no `printf '%s' ... | tr -d '\n\r'` applied). An attacker-controlled input could inject newlines to poison GITHUB_OUTPUT.

Locations:

- `action.yml:53`

### github-env-injection (severity: high)

In release.yml's 'Use Cross' step, `${{ matrix.target }}` is written directly to $GITHUB_ENV without sanitization: `echo "TARGET_FLAGS=--target ${{ matrix.target }}" >> $GITHUB_ENV` and `echo "TARGET_DIR=./target/${{ matrix.target }}" >> $GITHUB_ENV`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing newline injection to poison the environment.

Locations:

- `.github/workflows/release.yml:60`
- `.github/workflows/release.yml:61`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag or branch is moved or compromised. Unpinned references found:
- action.yml: `actions/upload-artifact@v4`
- actions.yml: `actions/checkout@v4`, `screenly/cli@master`
- docs.yml: `actions/checkout@v4`, `dorny/paths-filter@v3`, `actions/checkout@v3`, `dtolnay/rust-toolchain@master`
- fmt.yml: `actions/checkout@v4`, `dtolnay/rust-toolchain@nightly`
- lint.yml: `actions/checkout@v4`, `actions-rs/clippy-check@v1.0.7`
- nix.yml: `actions/checkout@v4`, `DeterminateSystems/nix-installer-action@v8`, `DeterminateSystems/flakehub-cache-action@main`
- release.yml: `actions/checkout@v3`, `dtolnay/rust-toolchain@master`, `softprops/action-gh-release@v1`, `actions/attest-build-provenance@v1`, `actions/checkout@v3`, `docker/login-action@v3`
- rust.yml: `actions/checkout@v4`, `actions/cache@v4`
- sbom.yml: `actions/checkout@v4`, `sbomify/github-action@master`, `actions/attest-build-provenance@v1`

Locations:

- `action.yml:64`
- `.github/workflows/actions.yml:9`
- `.github/workflows/actions.yml:11`
- `.github/workflows/docs.yml:22`
- `.github/workflows/docs.yml:23`
- `.github/workflows/docs.yml:37`
- `.github/workflows/docs.yml:38`
- `.github/workflows/fmt.yml:13`
- `.github/workflows/fmt.yml:17`
- `.github/workflows/lint.yml:22`
- `.github/workflows/lint.yml:30`
- `.github/workflows/nix.yml:30`
- `.github/workflows/nix.yml:32`
- `.github/workflows/nix.yml:35`
- `.github/workflows/release.yml:50`
- `.github/workflows/release.yml:53`
- `.github/workflows/release.yml:88`
- `.github/workflows/release.yml:92`
- `.github/workflows/release.yml:97`
- `.github/workflows/release.yml:109`
- `.github/workflows/rust.yml:27`
- `.github/workflows/rust.yml:29`
- `.github/workflows/sbom.yml:14`
- `.github/workflows/sbom.yml:16`
- `.github/workflows/sbom.yml:24`

### missing-permissions (severity: medium)

The workflow file actions.yml has no top-level `permissions:` key and its only job (`test-github-action-workflow`) also has no job-level `permissions:` key. This means the job runs with the default (broad) token permissions.

Locations:

- `.github/workflows/actions.yml:1`

### missing-permissions (severity: medium)

The workflow file rust.yml has no top-level `permissions:` key and its only job (`build_and_test`) also has no job-level `permissions:` key. This means the job runs with the default (broad) token permissions.

Locations:

- `.github/workflows/rust.yml:1`

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

Fixed all findings across action.yml and 7 workflow files:

1. script-injection/static-inline-injection (action.yml): Moved ${{ inputs.cli_version }}, ${{ inputs.screenly_api_token }}, ${{ inputs.cli_commands }} into env: blocks; shell scripts reference plain env vars.

2. script-injection (release.yml): Moved ${{ matrix.target }}, ${{ matrix.build }}, ${{ env.CARGO }}, ${{ env.TARGET_FLAGS }}, ${{ env.TARGET_DIR }} into env: blocks for each affected step.

3. script-injection (rust.yml): Moved ${{ matrix.rust }} into env: RUST_VERSION: for Build and Run tests steps; docker image tag uses "rust:${RUST_VERSION}".

4. github-env-injection (action.yml): Added printf/tr sanitization before writing to $GITHUB_OUTPUT.

5. github-env-injection (release.yml): Added printf/tr sanitization of MATRIX_TARGET before writing TARGET_FLAGS and TARGET_DIR to $GITHUB_ENV.

6. unpinned-uses: All 15 unique action references pinned to full 40-char commit SHAs with tag comments preserved.

7. missing-permissions (actions.yml, rust.yml): Added permissions: {} at top level of both workflow files.

### Iteration 2

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed all 5 findings across 3 files:

1. **action.yml (line 51)**: Double-quoted `$CLI_COMMANDS` → `"$CLI_COMMANDS"` to prevent shell metacharacter injection from attacker-controlled input.

2. **docs.yml (line 62)**: Moved `${{ secrets.DEVELOPER_PORTAL_REPO_TOKEN }}` out of the `run:` shell string into a step-level `env:` block as `DEVELOPER_PORTAL_REPO_TOKEN`, then referenced it as `$DEVELOPER_PORTAL_REPO_TOKEN` in the curl command.

3. **docs.yml (line 48)**: Added `permissions: {}` to the `trigger-developer-portal-deploy` job since it only uses an external secret token (not the GitHub token) to call an external API.

4. **release.yml (line 83)**: Double-quoted `$BUILD_CARGO` → `"$BUILD_CARGO"` and used `${BUILD_TARGET_FLAGS:+"$BUILD_TARGET_FLAGS"}` for the optional flags variable (which defaults to empty string) to prevent shell injection while correctly handling the empty-string case.

5. **release.yml (line 107)**: Added `permissions: {}` to the `build-docker-image` job since it only uses Docker Hub credentials from secrets (not the GitHub token).


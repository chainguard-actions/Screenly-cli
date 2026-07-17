<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **Screenly--cli/v1.1.1** was hardened automatically. 20 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ inputs.* }} expressions are directly interpolated inside run: shell commands in action.yml. (1) Line 37: `${{ inputs.cli_version }}` is embedded in a wget URL — an attacker-controlled version string is injected into the shell command. (2) Line 48: `API_TOKEN=${{ inputs.screenly_api_token }}` and `${{ inputs.cli_commands }}` are directly interpolated into a shell command line, allowing arbitrary command injection via the cli_commands input.

Locations:

- `action.yml:37`
- `action.yml:48`

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ matrix.* }} and ${{ env.* }} expressions are directly interpolated inside run: shell commands in release.yml. (1) 'Use Cross' step: `${{ matrix.target }}` is interpolated into echo commands that write to $GITHUB_ENV. (2) 'Show command used for Cargo' step: `${{ env.CARGO }}`, `${{ env.TARGET_FLAGS }}`, `${{ env.TARGET_DIR }}` are interpolated in run: block. (3) 'Build release binary' step: `${{ env.CARGO }}` and `${{ env.TARGET_FLAGS }}` are used directly as shell command. (4) 'Strip release binary' step: `${{ matrix.target }}` is interpolated in a shell path. (5) 'Package' step: `${{ matrix.target }}` and `${{ matrix.build }}` are interpolated in shell commands.

Locations:

- `.github/workflows/release.yml:56`
- `.github/workflows/release.yml:62`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:73`
- `.github/workflows/release.yml:79`

### script-injection (severity: high)

Sub-rule (a): `${{ matrix.rust }}` is directly interpolated inside run: shell commands in rust.yml. The matrix value is used as a Docker image tag in shell commands: `rust:${{ matrix.rust }}` appears unquoted in docker run commands, allowing shell metacharacter injection if the matrix value were attacker-influenced.

Locations:

- `.github/workflows/rust.yml:38`
- `.github/workflows/rust.yml:46`

### github-env-injection (severity: high)

In the 'Use Cross' step of release.yml, the value `${{ matrix.target }}` is written directly to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). This allows newline injection into the GitHub environment file: `echo "TARGET_FLAGS=--target ${{ matrix.target }}" >> $GITHUB_ENV` and `echo "TARGET_DIR=./target/${{ matrix.target }}" >> $GITHUB_ENV`.

Locations:

- `.github/workflows/release.yml:59`
- `.github/workflows/release.yml:60`

### unpinned-uses (severity: high)

action.yml references `actions/upload-artifact@v4` — a mutable tag ref instead of a pinned SHA digest. This is vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `action.yml:60`

### unpinned-uses (severity: high)

actions.yml references `actions/checkout@v4` and `screenly/cli@master` — both are mutable tag/branch refs instead of pinned SHA digests.

Locations:

- `.github/workflows/actions.yml:10`
- `.github/workflows/actions.yml:12`

### unpinned-uses (severity: high)

docs.yml references multiple unpinned actions: `actions/checkout@v4`, `dorny/paths-filter@v3`, `actions/checkout@v3`, and `dtolnay/rust-toolchain@master` — all mutable tag/branch refs instead of pinned SHA digests.

Locations:

- `.github/workflows/docs.yml:20`
- `.github/workflows/docs.yml:21`
- `.github/workflows/docs.yml:34`
- `.github/workflows/docs.yml:37`

### unpinned-uses (severity: high)

fmt.yml references `actions/checkout@v4` and `dtolnay/rust-toolchain@nightly` — both are mutable tag/branch refs instead of pinned SHA digests.

Locations:

- `.github/workflows/fmt.yml:13`
- `.github/workflows/fmt.yml:16`

### unpinned-uses (severity: high)

lint.yml references `actions/checkout@v4` and `actions-rs/clippy-check@v1.0.7` — both are mutable tag refs instead of pinned SHA digests.

Locations:

- `.github/workflows/lint.yml:22`
- `.github/workflows/lint.yml:29`

### unpinned-uses (severity: high)

nix.yml references `actions/checkout@v4`, `DeterminateSystems/nix-installer-action@v8`, and `DeterminateSystems/flakehub-cache-action@main` — all mutable tag/branch refs instead of pinned SHA digests.

Locations:

- `.github/workflows/nix.yml:30`
- `.github/workflows/nix.yml:33`
- `.github/workflows/nix.yml:36`

### unpinned-uses (severity: high)

release.yml references multiple unpinned actions: `actions/checkout@v3`, `dtolnay/rust-toolchain@master`, `softprops/action-gh-release@v1`, `actions/attest-build-provenance@v1`, `docker/login-action@v3` — all mutable tag/branch refs instead of pinned SHA digests.

Locations:

- `.github/workflows/release.yml:47`
- `.github/workflows/release.yml:50`
- `.github/workflows/release.yml:86`
- `.github/workflows/release.yml:92`
- `.github/workflows/release.yml:107`
- `.github/workflows/release.yml:116`

### unpinned-uses (severity: high)

rust.yml references `actions/checkout@v4` and `actions/cache@v4` — mutable tag refs instead of pinned SHA digests.

Locations:

- `.github/workflows/rust.yml:24`
- `.github/workflows/rust.yml:27`

### unpinned-uses (severity: high)

sbom.yml references `actions/checkout@v4`, `sbomify/github-action@master`, and `actions/attest-build-provenance@v1` — all mutable tag/branch refs instead of pinned SHA digests.

Locations:

- `.github/workflows/sbom.yml:14`
- `.github/workflows/sbom.yml:17`
- `.github/workflows/sbom.yml:28`

### missing-permissions (severity: medium)

actions.yml has no top-level `permissions:` key and the single job `test-github-action-workflow` has no job-level `permissions:` key. The workflow runs with default (broad) permissions.

Locations:

- `.github/workflows/actions.yml:1`

### missing-permissions (severity: medium)

rust.yml has no top-level `permissions:` key and the job `build_and_test` has no job-level `permissions:` key. The workflow runs with default (broad) permissions.

Locations:

- `.github/workflows/rust.yml:1`

### missing-permissions (severity: medium)

release.yml has no top-level `permissions:` key. The `build-release` job has job-level permissions, but the `build-docker-image` job has no `permissions:` key and therefore runs with default (broad) permissions.

Locations:

- `.github/workflows/release.yml:100`

### missing-permissions (severity: medium)

docs.yml has no top-level `permissions:` key. The `check_files_changed` and `docs-help-md` jobs have job-level permissions, but the `trigger-developer-portal-deploy` job has no `permissions:` key and therefore runs with default (broad) permissions.

Locations:

- `.github/workflows/docs.yml:56`

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

Fixed all findings across 8 files:

**action.yml**: Moved ${{ inputs.cli_version }}, ${{ inputs.screenly_api_token }}, and ${{ inputs.cli_commands }} from inline run: blocks to env: maps. Pinned actions/upload-artifact@v4 to SHA ea165f8d65b6e75b540449e92b4886f43607fa02.

**actions.yml**: Added top-level permissions: {} and job-level permissions: contents: read. Pinned actions/checkout@v4 (SHA 34e114876b0b11c390a56381ad16ebd13914f8d5) and screenly/cli@master (SHA 5dad863ccdc36263eeb22880ceb67923bea7ef98).

**docs.yml**: Added permissions: {} to trigger-developer-portal-deploy job. Pinned actions/checkout@v4, dorny/paths-filter@v3, actions/checkout@v3, and dtolnay/rust-toolchain@master to SHAs.

**fmt.yml**: Pinned actions/checkout@v4 and dtolnay/rust-toolchain@nightly to SHAs.

**lint.yml**: Pinned actions/checkout@v4 and actions-rs/clippy-check@v1.0.7 to SHAs.

**nix.yml**: Pinned actions/checkout@v4, DeterminateSystems/nix-installer-action@v8, and DeterminateSystems/flakehub-cache-action@main to SHAs.

**release.yml**: Added permissions: contents: read to build-docker-image job. Moved all ${{ matrix.* }} and ${{ env.* }} expressions to env: blocks. Sanitized GITHUB_ENV writes with printf/tr to prevent newline injection. Pinned all 5 unpinned actions to SHAs.

**rust.yml**: Added top-level permissions: {} and job-level permissions: contents: read. Moved ${{ matrix.rust }} to env: blocks as RUST_VERSION. Pinned actions/checkout@v4 and actions/cache@v4 to SHAs.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all four findings:
1. action.yml line 52 (script-injection): Quoted `$CLI_COMMANDS` as `"$CLI_COMMANDS"` to prevent shell metacharacter injection from unquoted expansion.
2. action.yml line 57 (github-env-injection): Sanitized CLI output with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT to prevent newline injection.
3. docs.yml line 68 (script-injection): Moved `${{ secrets.DEVELOPER_PORTAL_REPO_TOKEN }}` to the step's `env:` block and referenced it as `$DEVELOPER_PORTAL_REPO_TOKEN` in the curl command.
4. release.yml line 77 (script-injection): Replaced unquoted `$CARGO_CMD ... $TARGET_FLAGS_VAL` with a bash array approach using `read -ra` to split TARGET_FLAGS_VAL into separate arguments, with `"$CARGO_CMD"` properly quoted.


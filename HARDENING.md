<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.1.1** was hardened automatically. 9 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple ${{ inputs.* }} expressions are directly interpolated into run: shell blocks in action.yml. Specifically: ${{ inputs.cli_version }} is interpolated into a wget URL (line 36), ${{ inputs.screenly_api_token }} is used as an inline env assignment (line 47), and ${{ inputs.cli_commands }} is passed directly as shell arguments (line 47). An attacker controlling these inputs can inject arbitrary shell commands.

Locations:

- `action.yml:36`
- `action.yml:47`

### github-env-injection (severity: high)

The 'Run CLI' step in action.yml writes CLI command output to $GITHUB_OUTPUT without sanitization: `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"`. The content of command_cleaned_output.txt is derived from user-controlled ${{ inputs.cli_commands }} and could contain newlines that inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:54`

### script-injection (severity: high)

Rule (a): Multiple ${{ ... }} expressions are directly interpolated into run: shell blocks in release.yml. Offending lines include: `run: ${{ env.CARGO }} build --verbose --release ${{ env.TARGET_FLAGS }}` (the entire run: value is an expression), `echo "TARGET_FLAGS=--target ${{ matrix.target }}" >> $GITHUB_ENV`, `echo "TARGET_DIR=./target/${{ matrix.target }}" >> $GITHUB_ENV`, `cd target/${{ matrix.target }}/release`, `if [[ "${{ matrix.build }}" == windows* ]]`, `zip ../../../screenly-cli-${{ matrix.target }}.zip`, and `tar czvf ../../../screenly-cli-${{ matrix.target }}.tar.gz`. These allow matrix-controlled values to be injected into shell commands before the shell parses them.

Locations:

- `.github/workflows/release.yml:62`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:75`
- `.github/workflows/release.yml:81`
- `.github/workflows/release.yml:85`
- `.github/workflows/release.yml:88`

### github-env-injection (severity: high)

In release.yml, the 'Use Cross' step writes ${{ matrix.target }} directly to $GITHUB_ENV without sanitization: `echo "TARGET_FLAGS=--target ${{ matrix.target }}" >> $GITHUB_ENV` and `echo "TARGET_DIR=./target/${{ matrix.target }}" >> $GITHUB_ENV`. A matrix value containing newlines could inject additional environment variables.

Locations:

- `.github/workflows/release.yml:63`
- `.github/workflows/release.yml:64`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks. Unpinned references found:
- action.yml: `actions/upload-artifact@v4`
- .github/workflows/actions.yml: `actions/checkout@v4`, `screenly/cli@master`
- .github/workflows/docs.yml: `actions/checkout@v4`, `dorny/paths-filter@v3`, `actions/checkout@v3`, `dtolnay/rust-toolchain@master`
- .github/workflows/fmt.yml: `actions/checkout@v4`, `dtolnay/rust-toolchain@nightly`
- .github/workflows/lint.yml: `actions/checkout@v4`, `actions-rs/clippy-check@v1.0.7`
- .github/workflows/nix.yml: `actions/checkout@v4`, `DeterminateSystems/nix-installer-action@v8`, `DeterminateSystems/flakehub-cache-action@main`
- .github/workflows/release.yml: `actions/checkout@v3`, `dtolnay/rust-toolchain@master`, `softprops/action-gh-release@v1`, `actions/attest-build-provenance@v1`, `actions/checkout@v3`, `docker/login-action@v3`
- .github/workflows/rust.yml: `actions/checkout@v4`, `actions/cache@v4`
- .github/workflows/sbom.yml: `actions/checkout@v4`, `sbomify/github-action@master`, `actions/attest-build-provenance@v1`

Locations:

- `action.yml:63`
- `.github/workflows/actions.yml:9`
- `.github/workflows/actions.yml:11`
- `.github/workflows/docs.yml:21`
- `.github/workflows/docs.yml:22`
- `.github/workflows/docs.yml:36`
- `.github/workflows/docs.yml:40`
- `.github/workflows/fmt.yml:13`
- `.github/workflows/fmt.yml:16`
- `.github/workflows/lint.yml:24`
- `.github/workflows/lint.yml:31`
- `.github/workflows/nix.yml:31`
- `.github/workflows/nix.yml:34`
- `.github/workflows/nix.yml:37`
- `.github/workflows/release.yml:52`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:93`
- `.github/workflows/release.yml:97`
- `.github/workflows/release.yml:107`
- `.github/workflows/release.yml:117`
- `.github/workflows/rust.yml:27`
- `.github/workflows/rust.yml:30`
- `.github/workflows/sbom.yml:14`
- `.github/workflows/sbom.yml:16`
- `.github/workflows/sbom.yml:26`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs, meaning they run with the default (potentially broad) token permissions:
- .github/workflows/actions.yml: no permissions defined at top-level or job level
- .github/workflows/rust.yml: no permissions defined at top-level or job level

Locations:

- `.github/workflows/actions.yml:1`
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

Fixed all findings across action.yml and .github/workflows/*.yml:

1. script-injection / static-inline-injection (action.yml): Moved ${{ inputs.cli_version }}, ${{ inputs.screenly_api_token }}, and ${{ inputs.cli_commands }} out of run: blocks into env: maps; shell scripts reference plain env vars.

2. github-env-injection (action.yml): Sanitized CLI output before writing to $GITHUB_OUTPUT using `printf '%s' ... | tr -d '\n\r'`.

3. script-injection / github-env-injection (release.yml): Moved ${{ matrix.target }} and ${{ matrix.build }} to env: blocks in 'Use Cross', 'Strip release binary', and 'Package' steps. Sanitized matrix.target with tr -d '\n\r' before writing to $GITHUB_ENV.

4. unpinned-uses: Pinned all 15 unpinned action references to full 40-character commit SHAs across action.yml, actions.yml, docs.yml, fmt.yml, lint.yml, nix.yml, release.yml, rust.yml, and sbom.yml.

5. missing-permissions: Added `permissions: contents: read` to actions.yml and rust.yml.

### Iteration 2

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed 7 findings across 4 files:

1. release.yml (script-injection, lines 70+75): Moved ${{ env.CARGO }}, ${{ env.TARGET_FLAGS }}, ${{ env.TARGET_DIR }} into env: blocks for 'Show command used for Cargo' and 'Build release binary' steps; referenced as plain shell vars.

2. docs.yml (script-injection, line 75): Moved ${{ secrets.DEVELOPER_PORTAL_REPO_TOKEN }} into an env: block (DEVELOPER_PORTAL_REPO_TOKEN) and referenced as $DEVELOPER_PORTAL_REPO_TOKEN in the curl -H Authorization header.

3. rust.yml (script-injection, lines 44+52): Moved ${{ matrix.rust }} into env: blocks (MATRIX_RUST) for both Build and Run tests steps; Docker image tag is now "rust:${MATRIX_RUST}" (quoted).

4. action.yml (script-injection, line 52): Fixed unquoted $CLI_COMMANDS by splitting into a bash array with `read -ra cli_args <<< "$CLI_COMMANDS"` and passing "${cli_args[@]}" to prevent shell metacharacter injection.

5. docs.yml (missing-permissions, line 62): Added `permissions: contents: read` to the trigger-developer-portal-deploy job.

6. release.yml (missing-permissions, line 100): Added `permissions: contents: read` to the build-docker-image job.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the 'Build release binary' step in .github/workflows/release.yml: (1) Added `shell: bash` to enable bash-specific features; (2) Quoted `$CARGO` as `"$CARGO"` to prevent shell metacharacter injection; (3) Replaced unquoted `$TARGET_FLAGS` with a bash array (`read -ra target_flags_array <<< "$TARGET_FLAGS"`) expanded as `"${target_flags_array[@]}"` to safely handle the multi-word flag value while keeping each token properly quoted.


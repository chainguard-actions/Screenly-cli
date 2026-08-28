<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v26.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v26.8.0** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are directly interpolated inside run: shell command strings in action.yml. In the 'Download CLI' step, `${{ inputs.cli_version }}` is embedded in a wget URL. In the 'Run CLI' step, `${{ inputs.screenly_api_token }}` is used as an environment variable value inline and `${{ inputs.cli_commands }}` is passed directly as shell arguments. These inputs are attacker-controlled and allow command injection.

Locations:

- `action.yml:35`
- `action.yml:44`

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are directly interpolated inside run: shell command strings in release.yml. The 'Use Cross' step writes `${{ matrix.target }}` directly into shell commands and to $GITHUB_ENV. The 'Show command used for Cargo' step interpolates `${{ env.CARGO }}`, `${{ env.TARGET_FLAGS }}`, `${{ env.TARGET_DIR }}`. The 'Build release binary' step runs `${{ env.CARGO }} build ... ${{ env.TARGET_FLAGS }}` directly. The 'Strip release binary' step uses `${{ matrix.target }}`. The 'Package' step uses `${{ matrix.target }}` and `${{ matrix.build }}` in shell conditionals and commands. The 'Package MCP Bundle' step uses `${{ matrix.build }}`, `${{ matrix.target }}`, and `${{ matrix.mcpb_platform }}` throughout.

Locations:

- `.github/workflows/release.yml:75`
- `.github/workflows/release.yml:81`
- `.github/workflows/release.yml:86`
- `.github/workflows/release.yml:90`
- `.github/workflows/release.yml:94`
- `.github/workflows/release.yml:100`
- `.github/workflows/release.yml:108`
- `.github/workflows/release.yml:116`

### script-injection (severity: high)

Sub-rule (a): `${{ matrix.rust }}` is directly interpolated inside run: shell command strings in rust.yml. In the 'Build' and 'Run tests' steps, the matrix value is embedded directly in the docker image reference (e.g., `rust:${{ matrix.rust }}`), allowing the matrix value to influence the shell command without quoting or sanitization.

Locations:

- `.github/workflows/rust.yml:40`
- `.github/workflows/rust.yml:48`

### github-env-injection (severity: high)

In the 'Use Cross' step of release.yml, `${{ matrix.target }}` is written directly to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). Lines such as `echo "TARGET_FLAGS=--target ${{ matrix.target }}" >> $GITHUB_ENV` and `echo "TARGET_DIR=./target/${{ matrix.target }}" >> $GITHUB_ENV` allow newline injection into the environment file.

Locations:

- `.github/workflows/release.yml:75`

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use `uses:` references pinned to mutable tags, branches, or version strings instead of full 40-character commit SHAs. Failing references include:
- action.yml: `actions/upload-artifact@v4`
- actions.yml: `actions/checkout@v4`, `screenly/cli@master`
- docs.yml: `actions/checkout@v4`, `dorny/paths-filter@v3`, `actions/checkout@v3`, `dtolnay/rust-toolchain@master`
- fmt.yml: `actions/checkout@v4`, `dtolnay/rust-toolchain@nightly`
- lint.yml: `actions/checkout@v4`, `actions-rs/clippy-check@v1.0.7`
- nix.yml: `actions/checkout@v4`, `DeterminateSystems/nix-installer-action@v22`, `DeterminateSystems/flakehub-cache-action@main`
- release.yml: `actions/checkout@v3`, `dtolnay/rust-toolchain@master`, `softprops/action-gh-release@v1`, `actions/attest-build-provenance@v1`, `actions/checkout@v3`, `docker/login-action@v3`
- rust.yml: `actions/checkout@v4`, `actions/cache@v4`
- sbom.yml: `actions/checkout@v4`, `sbomify/github-action@master`, `actions/attest-build-provenance@v1`

Locations:

- `action.yml:57`
- `.github/workflows/actions.yml:14`
- `.github/workflows/actions.yml:16`
- `.github/workflows/docs.yml:22`
- `.github/workflows/docs.yml:23`
- `.github/workflows/docs.yml:34`
- `.github/workflows/docs.yml:38`
- `.github/workflows/fmt.yml:13`
- `.github/workflows/fmt.yml:17`
- `.github/workflows/lint.yml:24`
- `.github/workflows/lint.yml:31`
- `.github/workflows/nix.yml:30`
- `.github/workflows/nix.yml:33`
- `.github/workflows/nix.yml:36`
- `.github/workflows/release.yml:63`
- `.github/workflows/release.yml:66`
- `.github/workflows/release.yml:131`
- `.github/workflows/release.yml:136`
- `.github/workflows/release.yml:143`
- `.github/workflows/release.yml:152`
- `.github/workflows/rust.yml:27`
- `.github/workflows/rust.yml:30`
- `.github/workflows/sbom.yml:15`
- `.github/workflows/sbom.yml:18`
- `.github/workflows/sbom.yml:28`

### missing-permissions (severity: medium)

The workflow file actions.yml has no top-level `permissions:` key and the single job `test-github-action-workflow` also has no `permissions:` key. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/actions.yml:1`

### missing-permissions (severity: medium)

The workflow file rust.yml has no top-level `permissions:` key and the job `build_and_test` has no `permissions:` key. This means the workflow runs with the default (potentially broad) token permissions.

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

Fixed all findings across action.yml and 6 workflow files:

1. action.yml: Moved inputs.cli_version, inputs.screenly_api_token, and inputs.cli_commands from run: blocks to env: blocks. Used xargs-based tokenization for cli_commands (a list input). Pinned actions/upload-artifact to SHA ea165f8d65b6e75b540449e92b4886f43607fa02.

2. release.yml: Moved all matrix.target, matrix.build, matrix.mcpb_platform, matrix.mcpb_alias expressions to env: blocks. Replaced ${{ env.CARGO/TARGET_FLAGS/TARGET_DIR }} with direct shell env var references. Sanitized MATRIX_TARGET with tr -d '\n\r' before writing to $GITHUB_ENV. Pinned all 6 action references to full SHAs.

3. rust.yml: Moved ${{ matrix.rust }} to env: block (MATRIX_RUST) in Build and Run tests steps. Added top-level permissions: contents: read. Pinned actions/checkout and actions/cache to full SHAs.

4. actions.yml: Added permissions: {} at top level. Pinned actions/checkout@v4 and screenly/cli@master to full SHAs.

5. docs.yml: Pinned actions/checkout@v4, dorny/paths-filter@v3, actions/checkout@v3, dtolnay/rust-toolchain@master to full SHAs.

6. fmt.yml: Pinned actions/checkout@v4 and dtolnay/rust-toolchain@nightly to full SHAs.

7. lint.yml: Pinned actions/checkout@v4 and actions-rs/clippy-check@v1.0.7 to full SHAs.

8. nix.yml: Pinned actions/checkout@v4, DeterminateSystems/nix-installer-action@v22, DeterminateSystems/flakehub-cache-action@main to full SHAs.

9. sbom.yml: Pinned actions/checkout@v4, sbomify/github-action@master, actions/attest-build-provenance@v1 to full SHAs.

### Iteration 2

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed three findings across two workflow files:
1. docs.yml (script-injection): Moved `${{ secrets.DEVELOPER_PORTAL_REPO_TOKEN }}` from the `run:` shell string into a step-level `env:` block as `DEVELOPER_PORTAL_REPO_TOKEN`, and referenced it as `$DEVELOPER_PORTAL_REPO_TOKEN` in the curl command.
2. docs.yml (missing-permissions): Added `permissions: {}` to the `trigger-developer-portal-deploy` job, since it only makes an outbound HTTP request and requires no GitHub token permissions.
3. release.yml (missing-permissions): Added `permissions: contents: read` to the `build-docker-image` job, which checks out the repository and pushes Docker images to DockerHub using stored secrets.


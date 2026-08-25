<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.2.2** was hardened automatically. 15 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are directly interpolated inside run: shell commands in action.yml. In the 'Download CLI' step, `${{ inputs.cli_version }}` is embedded in a wget URL. In the 'Run CLI' step, `${{ inputs.screenly_api_token }}` is set as an inline env var and `${{ inputs.cli_commands }}` is passed directly as shell arguments. An attacker controlling these inputs can inject arbitrary shell commands.

Locations:

- `action.yml:36`
- `action.yml:46`
- `action.yml:46`

### github-env-injection (severity: high)

The 'Run CLI' step in action.yml writes `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"` where the file content is derived from executing the CLI with attacker-controlled `${{ inputs.cli_commands }}`. No sanitization (`printf '%s' ... | tr -d '\n\r'`) is applied before the write, allowing newline injection to set arbitrary GITHUB_OUTPUT variables.

Locations:

- `action.yml:53`

### script-injection (severity: high)

Rule (a): Multiple ${{ matrix.* }} and ${{ env.* }} expressions are directly interpolated inside run: shell commands in release.yml. Affected steps include: 'Use Cross' (matrix.target written into GITHUB_ENV and shell strings), 'Show command used for Cargo' (env.CARGO, env.TARGET_FLAGS, env.TARGET_DIR), 'Build release binary' (env.CARGO, env.TARGET_FLAGS), 'Strip release binary' (matrix.target), 'Package' (matrix.target, matrix.build), and 'Package MCP Bundle' (matrix.build, matrix.target, matrix.mcpb_platform, matrix.mcpb_alias). These expressions are substituted by the Actions runner before the shell sees them, enabling injection via matrix values.

Locations:

- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:74`
- `.github/workflows/release.yml:79`
- `.github/workflows/release.yml:83`
- `.github/workflows/release.yml:87`
- `.github/workflows/release.yml:93`
- `.github/workflows/release.yml:107`
- `.github/workflows/release.yml:113`

### github-env-injection (severity: high)

The 'Use Cross' step in release.yml writes `echo "TARGET_FLAGS=--target ${{ matrix.target }}" >> $GITHUB_ENV` and `echo "TARGET_DIR=./target/${{ matrix.target }}" >> $GITHUB_ENV` without sanitization. A matrix value containing newlines could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/release.yml:69`
- `.github/workflows/release.yml:70`

### script-injection (severity: high)

Rule (a): `${{ matrix.rust }}` is directly interpolated inside run: shell commands in rust.yml. It is embedded in a docker image tag reference (`rust:${{ matrix.rust }}`) in both the 'Build' and 'Run tests' steps. A matrix value containing shell metacharacters would be injected into the shell command before quoting.

Locations:

- `.github/workflows/rust.yml:42`
- `.github/workflows/rust.yml:51`

### script-injection (severity: high)

Rule (a): `${{ secrets.DEVELOPER_PORTAL_REPO_TOKEN }}` is directly interpolated inside a run: shell command in docs.yml, embedded in a curl -H "Authorization: Bearer ..." header. Any GitHub Actions expression inside a run: block is substituted before the shell processes it, making this a script-injection risk.

Locations:

- `.github/workflows/docs.yml:72`

### unpinned-uses (severity: high)

action.yml references `actions/upload-artifact@v4` using a mutable tag instead of a pinned 40-character SHA digest. This allows the referenced action to be silently replaced with malicious code.

Locations:

- `action.yml:58`

### unpinned-uses (severity: high)

Multiple workflow files use mutable tag or branch refs instead of pinned 40-character SHA digests. Failing references include: actions.yml: `actions/checkout@v4`, `screenly/cli@master`; docs.yml: `actions/checkout@v4`, `dorny/paths-filter@v3`, `actions/checkout@v3`, `dtolnay/rust-toolchain@master`; fmt.yml: `actions/checkout@v4`, `dtolnay/rust-toolchain@nightly`; lint.yml: `actions/checkout@v4`, `actions-rs/clippy-check@v1.0.7`; nix.yml: `actions/checkout@v4`, `DeterminateSystems/nix-installer-action@v22`, `DeterminateSystems/flakehub-cache-action@main`; release.yml: `actions/checkout@v3`, `dtolnay/rust-toolchain@master`, `softprops/action-gh-release@v1`, `actions/attest-build-provenance@v1`, `docker/login-action@v3`; rust.yml: `actions/checkout@v4`, `actions/cache@v4`; sbom.yml: `actions/checkout@v4`, `sbomify/github-action@master`, `actions/attest-build-provenance@v1`.

Locations:

- `.github/workflows/actions.yml:14`
- `.github/workflows/actions.yml:16`
- `.github/workflows/docs.yml:22`
- `.github/workflows/docs.yml:23`
- `.github/workflows/docs.yml:34`
- `.github/workflows/docs.yml:36`
- `.github/workflows/fmt.yml:12`
- `.github/workflows/fmt.yml:15`
- `.github/workflows/lint.yml:22`
- `.github/workflows/lint.yml:30`
- `.github/workflows/nix.yml:30`
- `.github/workflows/nix.yml:32`
- `.github/workflows/nix.yml:35`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:130`
- `.github/workflows/release.yml:135`
- `.github/workflows/release.yml:143`
- `.github/workflows/rust.yml:28`
- `.github/workflows/rust.yml:30`
- `.github/workflows/sbom.yml:14`
- `.github/workflows/sbom.yml:16`
- `.github/workflows/sbom.yml:22`

### missing-permissions (severity: medium)

actions.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. This means the workflow runs with the default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/actions.yml:1`

### missing-permissions (severity: medium)

rust.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. This means the workflow runs with the default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/rust.yml:1`

### missing-permissions (severity: medium)

release.yml has no top-level `permissions:` key. The `build-docker-image` job has no job-level `permissions:` key (only `build-release` does), so it inherits default broad permissions.

Locations:

- `.github/workflows/release.yml:143`

### missing-permissions (severity: medium)

docs.yml has no top-level `permissions:` key. The `trigger-developer-portal-deploy` job has no job-level `permissions:` key (only `check_files_changed` and `docs-help-md` do), so it inherits default broad permissions.

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

1. action.yml: Moved inputs.cli_version, inputs.screenly_api_token, and inputs.cli_commands to env: blocks. Used xargs-based array tokenization for cli_commands (list input). Added tr -d '\n\r' sanitization before writing to GITHUB_OUTPUT. Pinned actions/upload-artifact to SHA ea165f8d65b6e75b540449e92b4886f43607fa02.

2. release.yml: Added top-level permissions: {} and job-level permissions for build-docker-image (contents: read). Moved all matrix.target, matrix.build, matrix.mcpb_platform, matrix.mcpb_alias expressions to env: blocks. Replaced ${{ env.CARGO/TARGET_FLAGS/TARGET_DIR }} in run: blocks with plain shell vars. Added sanitization for GITHUB_ENV writes. Pinned all 5 action references to SHAs.

3. rust.yml: Added top-level permissions: contents: read. Moved ${{ matrix.rust }} to env: blocks as RUST_VERSION in both Build and Run tests steps. Pinned actions/checkout@v4 and actions/cache@v4 to SHAs.

4. docs.yml: Moved ${{ secrets.DEVELOPER_PORTAL_REPO_TOKEN }} to env: block. Added permissions: {} to trigger-developer-portal-deploy job. Pinned all 4 action references to SHAs.

5. actions.yml: Added top-level permissions: {}. Pinned actions/checkout@v4 and screenly/cli@master to SHAs.

6. fmt.yml: Pinned actions/checkout@v4 and dtolnay/rust-toolchain@nightly to SHAs.

7. lint.yml: Pinned actions/checkout@v4 and actions-rs/clippy-check@v1.0.7 to SHAs.

8. nix.yml: Pinned actions/checkout@v4, DeterminateSystems/nix-installer-action@v22, and DeterminateSystems/flakehub-cache-action@main to SHAs.

9. sbom.yml: Pinned actions/checkout@v4, sbomify/github-action@master, and actions/attest-build-provenance@v1 to SHAs.


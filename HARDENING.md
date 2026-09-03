<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v26.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v26.9.0** was hardened automatically. 21 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Download CLI' run: block directly interpolates ${{ inputs.cli_version }} inside a shell URL string. The 'Run CLI' run: block directly interpolates ${{ inputs.screenly_api_token }} and ${{ inputs.cli_commands }} inside a shell command. Any of these inputs can be attacker-controlled and allow command injection.

Locations:

- `action.yml:37`
- `action.yml:49`

### script-injection (severity: high)

Sub-rule (a): Multiple run: blocks in the build-release job directly interpolate ${{ matrix.target }}, ${{ matrix.build }}, ${{ matrix.mcpb_platform }}, ${{ matrix.mcpb_alias }}, ${{ env.CARGO }}, and ${{ env.TARGET_FLAGS }} inside shell command strings. The 'Use Cross' step writes ${{ matrix.target }} directly into echo commands. The 'Build release binary' step uses ${{ env.CARGO }} and ${{ env.TARGET_FLAGS }} as the run: command itself. The 'Package' and 'Package MCP Bundle' steps interpolate matrix values into shell conditionals and paths.

Locations:

- `.github/workflows/release.yml:77`
- `.github/workflows/release.yml:78`
- `.github/workflows/release.yml:82`
- `.github/workflows/release.yml:86`
- `.github/workflows/release.yml:88`
- `.github/workflows/release.yml:90`

### script-injection (severity: high)

Sub-rule (a): The 'Build' and 'Run tests' run: blocks directly interpolate ${{ matrix.rust }} inside a docker image tag reference (rust:${{ matrix.rust }}), allowing a matrix value to influence the shell command string.

Locations:

- `.github/workflows/rust.yml:38`
- `.github/workflows/rust.yml:46`

### github-env-injection (severity: high)

The 'Use Cross' run: block writes ${{ matrix.target }} directly to $GITHUB_ENV via echo without sanitization (no 'printf | tr -d newlines' step). Lines: 'echo "TARGET_FLAGS=--target ${{ matrix.target }}" >> $GITHUB_ENV' and 'echo "TARGET_DIR=./target/${{ matrix.target }}" >> $GITHUB_ENV'. A matrix value containing newlines could inject arbitrary environment variables.

Locations:

- `.github/workflows/release.yml:77`
- `.github/workflows/release.yml:78`

### github-env-injection (severity: high)

The 'Run CLI' run: block writes command output derived from unsanitized ${{ inputs.cli_commands }} to $GITHUB_OUTPUT via 'echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"' without sanitization. An attacker-controlled cli_commands input could produce output containing newlines that inject arbitrary output variables.

Locations:

- `action.yml:53`

### unpinned-uses (severity: high)

action.yml references actions/upload-artifact@v4 (tag, not a full 40-character SHA commit hash). This is vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `action.yml:59`

### unpinned-uses (severity: high)

Workflow uses unpinned action references (tags/branches instead of full SHA): actions/checkout@v4, screenly/cli@master. None are pinned to a 40-character hex SHA.

Locations:

- `.github/workflows/actions.yml:14`
- `.github/workflows/actions.yml:16`

### unpinned-uses (severity: high)

Workflow uses unpinned action references (tags/branches instead of full SHA): actions/checkout@v4, dorny/paths-filter@v3, actions/checkout@v3, dtolnay/rust-toolchain@master. None are pinned to a 40-character hex SHA.

Locations:

- `.github/workflows/docs.yml:22`
- `.github/workflows/docs.yml:23`
- `.github/workflows/docs.yml:33`
- `.github/workflows/docs.yml:36`

### unpinned-uses (severity: high)

Workflow uses unpinned action references (tags/branches instead of full SHA): actions/checkout@v4, dtolnay/rust-toolchain@nightly. None are pinned to a 40-character hex SHA.

Locations:

- `.github/workflows/fmt.yml:13`
- `.github/workflows/fmt.yml:14`

### unpinned-uses (severity: high)

Workflow uses unpinned action references (tags/branches instead of full SHA): actions/checkout@v4, actions-rs/clippy-check@v1.0.7. None are pinned to a 40-character hex SHA.

Locations:

- `.github/workflows/lint.yml:22`
- `.github/workflows/lint.yml:25`

### unpinned-uses (severity: high)

Workflow uses unpinned action references (tags/branches instead of full SHA): actions/checkout@v4, DeterminateSystems/nix-installer-action@v22, DeterminateSystems/flakehub-cache-action@main. None are pinned to a 40-character hex SHA.

Locations:

- `.github/workflows/nix.yml:30`
- `.github/workflows/nix.yml:33`
- `.github/workflows/nix.yml:36`

### unpinned-uses (severity: high)

Workflow uses unpinned action references (tags/branches instead of full SHA): actions/checkout@v3, dtolnay/rust-toolchain@master, softprops/action-gh-release@v1, actions/attest-build-provenance@v1, docker/login-action@v3. None are pinned to a 40-character hex SHA.

Locations:

- `.github/workflows/release.yml:65`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:133`
- `.github/workflows/release.yml:137`
- `.github/workflows/release.yml:148`
- `.github/workflows/release.yml:163`

### unpinned-uses (severity: high)

Workflow uses unpinned action references (tags/branches instead of full SHA): actions/checkout@v4, actions/cache@v4. None are pinned to a 40-character hex SHA.

Locations:

- `.github/workflows/rust.yml:24`
- `.github/workflows/rust.yml:27`

### unpinned-uses (severity: high)

Workflow uses unpinned action references (tags/branches instead of full SHA): actions/checkout@v4, sbomify/github-action@master, actions/attest-build-provenance@v1. None are pinned to a 40-character hex SHA.

Locations:

- `.github/workflows/sbom.yml:13`
- `.github/workflows/sbom.yml:16`
- `.github/workflows/sbom.yml:26`

### missing-permissions (severity: medium)

The workflow has no top-level permissions: block and the only job (test-github-action-workflow) has no job-level permissions: block. This means the job runs with the default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/actions.yml:1`

### missing-permissions (severity: medium)

The workflow has no top-level permissions: block and the only job (build_and_test) has no job-level permissions: block. This means the job runs with the default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/rust.yml:1`

### missing-permissions (severity: medium)

The workflow has no top-level permissions: block. The build-release job has job-level permissions, but the build-docker-image job has no permissions: block, so it runs with default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/release.yml:145`

### missing-permissions (severity: medium)

The workflow has no top-level permissions: block. The check_files_changed and docs-help-md jobs have job-level permissions, but the trigger-developer-portal-deploy job has no permissions: block, so it runs with default (potentially broad) GITHUB_TOKEN permissions.

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

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across action.yml and 7 workflow files:

1. action.yml - script-injection/static-inline-injection: Moved inputs.cli_version, inputs.screenly_api_token, and inputs.cli_commands to env: blocks. Used xargs-based tokenization for cli_commands (list-style input). Fixed github-env-injection by sanitizing response output with tr -d '\n\r' before writing to GITHUB_OUTPUT. Pinned actions/upload-artifact to SHA.

2. release.yml - script-injection: Moved all matrix.* and env.* expressions to env: blocks in each step. github-env-injection: Sanitized matrix.target with printf/tr before writing to GITHUB_ENV. missing-permissions: Added 'permissions: contents: read' to build-docker-image job. Pinned all 6 action references to full SHAs.

3. rust.yml - script-injection: Moved matrix.rust to env: RUST_VERSION in Build and Run tests steps. missing-permissions: Added top-level 'permissions: {}' and job-level 'permissions: contents: read'. Pinned actions/checkout@v4 and actions/cache@v4 to SHAs.

4. actions.yml - missing-permissions: Added top-level 'permissions: {}' and job-level 'permissions: contents: read'. Pinned actions/checkout@v4 and screenly/cli@master to SHAs.

5. docs.yml - missing-permissions: Added 'permissions: {}' to trigger-developer-portal-deploy job. Pinned all 4 action references to SHAs.

6. fmt.yml - Pinned actions/checkout@v4 and dtolnay/rust-toolchain@nightly to SHAs.

7. lint.yml - Pinned actions/checkout@v4 and actions-rs/clippy-check@v1.0.7 to SHAs.

8. nix.yml - Pinned actions/checkout@v4, DeterminateSystems/nix-installer-action@v22, and DeterminateSystems/flakehub-cache-action@main to SHAs.

9. sbom.yml - Pinned actions/checkout@v4, sbomify/github-action@master, and actions/attest-build-provenance@v1 to SHAs.


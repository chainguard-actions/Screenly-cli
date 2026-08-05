<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.2.0** was hardened automatically. 6 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ inputs.* }} expressions are interpolated directly inside run: shell command strings in action.yml, violating sub-rule (a). (1) Line 34: `${{ inputs.cli_version }}` is embedded directly in a wget URL string — an attacker-controlled version string could break out of the URL context. (2) Line 43: `API_TOKEN=${{ inputs.screenly_api_token }}` and `${{ inputs.cli_commands }}` are both interpolated directly into the shell command line — inputs.cli_commands in particular allows arbitrary shell command injection (e.g. a value of `; malicious-command` would execute). These must be moved to env: variables and then referenced as quoted shell variables (e.g. "$CLI_COMMANDS").

Locations:

- `action.yml:34`
- `action.yml:43`

### github-env-injection (severity: high)

The 'Run CLI' step writes unsanitized CLI output to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Specifically, line 49: `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"`. The file /tmp/screenly_cli_command_output.txt is produced by executing the user-controlled `${{ inputs.cli_commands }}` command, so its content is attacker-controlled. A newline embedded in the output could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting other outputs consumed by downstream steps.

Locations:

- `action.yml:49`

### unpinned-uses (severity: high)

The step 'Upload artifacts of failed screenly cli command' references `uses: actions/upload-artifact@v4`, which uses a mutable version tag rather than a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, creating a supply-chain risk. It should be pinned to a full SHA, e.g. `actions/upload-artifact@65c4c4a1ddee5b72f698fdd19549f0f0fb45cf08 # v4`.

Locations:

- `action.yml:57`

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

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed action.yml with the following changes:
1. Download CLI step: moved `${{ inputs.cli_version }}` to env: block as CLI_VERSION, referenced as ${CLI_VERSION} in the wget URL.
2. Run CLI step: moved `${{ inputs.screenly_api_token }}` to env: block as API_TOKEN and `${{ inputs.cli_commands }}` to env: block as CLI_COMMANDS. CLI_COMMANDS is tokenized into a bash array using xargs (quote-aware) so multi-word commands are properly split without injection risk.
3. GITHUB_OUTPUT write: sanitized the response with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT to prevent newline injection.
4. Pinned actions/upload-artifact@v4 to full SHA ea165f8d65b6e75b540449e92b4886f43607fa02 with # v4 comment.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 14 findings across 7 workflow files:

1. **script-injection (release.yml)**: Moved matrix.target, matrix.build, env.CARGO, env.TARGET_FLAGS, env.TARGET_DIR out of run: shell commands into env: blocks; shell scripts now reference $MATRIX_TARGET, $MATRIX_BUILD, $CARGO, $TARGET_FLAGS, $TARGET_DIR.

2. **script-injection (rust.yml)**: Moved matrix.rust out of docker run commands into env: blocks as MATRIX_RUST.

3. **github-env-injection (release.yml)**: The 'Use Cross' step now sanitizes matrix.target with `printf '%s' "$MATRIX_TARGET" | tr -d '\n\r'` before writing to $GITHUB_ENV.

4. **unpinned-uses**: Pinned all mutable tag/branch references to full 40-character SHA digests across actions.yml, docs.yml, fmt.yml, lint.yml, nix.yml, release.yml, rust.yml, and sbom.yml.

5. **missing-permissions**: Added `permissions: {}` to actions.yml (top-level); added `permissions: {}` top-level and `permissions: contents: read` job-level to rust.yml; added `permissions: contents: read` to the build-docker-image job in release.yml.

### Iteration 3

**Fixes applied:** script-injection, missing-permissions

**Notes:**

In .github/workflows/docs.yml, for the `trigger-developer-portal-deploy` job: (1) Added `permissions: {}` at the job level to restrict default token permissions; (2) Moved `${{ secrets.DEVELOPER_PORTAL_REPO_TOKEN }}` out of the `run:` curl command into a step-level `env:` block as `DEVELOPER_PORTAL_REPO_TOKEN`, and updated the shell script to reference it as the plain environment variable `$DEVELOPER_PORTAL_REPO_TOKEN`.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed the 'Build release binary' step in .github/workflows/release.yml. Changed from `run: $CARGO build --verbose --release $TARGET_FLAGS` (unquoted, injection-prone) to a bash script that: (1) adds `shell: bash`, (2) tokenizes TARGET_FLAGS into a bash array using xargs for quote-aware splitting (prevents metacharacter injection), (3) quotes $CARGO as "$CARGO", and (4) expands the array as "${flags[@]}" to pass each flag as a separate argument safely.


<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.2.0** was hardened automatically. 7 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): The 'Download CLI' step directly interpolates `${{ inputs.cli_version }}` inside a `run:` shell command string. Before the shell executes the command, GitHub Actions performs template substitution, allowing an attacker-controlled value to break out of the URL string and inject arbitrary shell commands. Offending line: `"https://github.com/Screenly/cli/releases/download/${{ inputs.cli_version }}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz"`

Locations:

- `action.yml:35`

### script-injection (severity: high)

Rule (a): The 'Run CLI' step directly interpolates `${{ inputs.screenly_api_token }}` and `${{ inputs.cli_commands }}` inside a `run:` shell command string. `inputs.cli_commands` is especially dangerous — it is passed verbatim as shell arguments, enabling arbitrary command injection by any caller of this action. Offending line: `API_TOKEN=${{ inputs.screenly_api_token }} RUST_LOG=debug /tmp/screenly ${{ inputs.cli_commands }} >> /tmp/screenly_cli_command_output.txt`

Locations:

- `action.yml:45`

### github-env-injection (severity: high)

The 'Run CLI' step writes the CLI command output to $GITHUB_OUTPUT without sanitization. The output is derived from `${{ inputs.cli_commands }}` (attacker-controlled). A newline character in the output can inject arbitrary key=value pairs into GITHUB_OUTPUT. The write `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"` is missing the required `printf '%s' ... | tr -d '\n\r'` sanitization step before the write.

Locations:

- `action.yml:50`

### unpinned-uses (severity: high)

The step 'Upload artifacts of failed screenly cli command' uses `actions/upload-artifact@v4`, which is pinned to a mutable version tag (`@v4`) rather than an immutable 40-character commit SHA. A compromised or altered tag could introduce malicious code into the action. It should be pinned to a full SHA, e.g. `actions/upload-artifact@65c4c4a1ddee5b72f698fdd19549f0f0fb45cf08 # v4`.

Locations:

- `action.yml:54`

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

Fixed all 7 findings in hardened/action/action.yml:
1. Download CLI step: moved `${{ inputs.cli_version }}` to env block as CLI_VERSION, referenced as ${CLI_VERSION} in shell.
2. Run CLI step: moved `${{ inputs.screenly_api_token }}` to env block as API_TOKEN and `${{ inputs.cli_commands }}` to env block as CLI_COMMANDS; both referenced as plain env vars in shell.
3. GITHUB_OUTPUT write: added `safe_response=$(printf '%s' "$(cat /tmp/command_cleaned_output.txt)" | tr -d '\n\r')` sanitization before writing to $GITHUB_OUTPUT.
4. Pinned actions/upload-artifact@v4 to full SHA ea165f8d65b6e75b540449e92b4886f43607fa02 # v4.

### Iteration 2

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all 4 findings across 8 workflow files:

**unpinned-uses**: Pinned all action references to full 40-char SHAs:
- actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
- actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26
- screenly/cli@master → @3d6a92abedc446b3b89e68ea134a33f0025ae564
- dorny/paths-filter@v3 → @d1c1ffe0248fe513906c8e24db8ea791d46f8590
- dtolnay/rust-toolchain@master → @2c7215f132e9ebf062739d9130488b56d53c060c
- dtolnay/rust-toolchain@nightly → @4fd1da8b0805d2d2e936788875a7d65dbd677dc2
- actions-rs/clippy-check@v1.0.7 → @b5b5f21f4797c02da247df37026fcd0a5024aa4d
- DeterminateSystems/nix-installer-action@v22 → @ef8a148080ab6020fd15196c2084a2eea5ff2d25
- DeterminateSystems/flakehub-cache-action@main → @77c6bddd7d747943530aaa578c57f233ee5d920e
- softprops/action-gh-release@v1 → @de2c0eb89ae2a093876385947365aca7b0e5f844
- actions/attest-build-provenance@v1 → @ef244123eb79f2f7a7e75d99086184180e6d0018
- docker/login-action@v3 → @c94ce9fb468520275223c153574b00df6fe4bcc9
- actions/cache@v4 → @0057852bfaa89a56745cba8c7296529d2fc39830
- sbomify/github-action@master → @c65a2b9fc24fc69f3376be690c430571be49c173

**missing-permissions**: Added permissions blocks to actions.yml (top-level + job), rust.yml (top-level + job), docs.yml (top-level + trigger job), release.yml (build-docker-image job).

**script-injection**: Moved all ${{ matrix.* }} and ${{ env.* }} expressions in run: blocks to env: blocks, referencing them as plain shell variables.

**github-env-injection**: In release.yml 'Use Cross' step, sanitized matrix.target with `printf '%s' "$MATRIX_TARGET" | tr -d '\n\r'` before writing to GITHUB_ENV.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:
1. hardened/action/action.yml line 54: Replaced unquoted `${CLI_COMMANDS}` expansion with `read -ra cli_args <<< "${CLI_COMMANDS}"` and `"${cli_args[@]}"` to safely split CLI arguments into an array, preventing shell metacharacter injection while preserving multi-argument support.
2. hardened/action/.github/workflows/docs.yml line 87: Moved `${{ secrets.DEVELOPER_PORTAL_REPO_TOKEN }}` out of the run block into an `env:` block (`DEVELOPER_PORTAL_REPO_TOKEN: ${{ secrets.DEVELOPER_PORTAL_REPO_TOKEN }}`), then referenced it as `$DEVELOPER_PORTAL_REPO_TOKEN` in the curl command.


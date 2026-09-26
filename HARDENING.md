<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.2.2** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ inputs.* }} expressions are interpolated directly inside run: shell command strings. In the 'Download CLI' step, ${{ inputs.cli_version }} is embedded directly in a URL string (e.g. `"https://github.com/.../download/${{ inputs.cli_version }}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz"`). In the 'Run CLI' step, both ${{ inputs.screenly_api_token }} and ${{ inputs.cli_commands }} are interpolated directly into the shell command line (`API_TOKEN=${{ inputs.screenly_api_token }} RUST_LOG=debug /tmp/screenly ${{ inputs.cli_commands }}`). An attacker-controlled value for cli_commands (e.g. `; malicious-command`) can execute arbitrary shell commands. These must be moved to env: variables and the shell expansions must be double-quoted.

Locations:

- `action.yml:38`
- `action.yml:46`

### github-env-injection (severity: high)

The 'Run CLI' step writes to $GITHUB_OUTPUT using `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"`. The content of command_cleaned_output.txt is derived from the CLI output produced by running ${{ inputs.cli_commands }} without sanitization. An attacker-controlled cli_commands value could inject newlines into the output file, which would then be written to GITHUB_OUTPUT without the required `printf '%s' ... | tr -d '\n\r'` sanitization step, enabling GITHUB_OUTPUT injection.

Locations:

- `action.yml:50`

### unpinned-uses (severity: high)

The step 'Upload artifacts of failed screenly cli command' uses `actions/upload-artifact@v4`, which is a mutable tag reference rather than a pinned full 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved. It should be pinned to a specific commit SHA (e.g. `actions/upload-artifact@65c4c4a1ddee5b72f698fdd19549f0f0fb45cf08 # v4`).

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

Fixed all 6 findings in action.yml:
1. Moved ${{ inputs.cli_version }} to env: block (CLI_VERSION) in 'Download CLI' step - fixes static-inline-injection
2. Moved ${{ inputs.screenly_api_token }} to env: block (API_TOKEN) in 'Run CLI' step - fixes script-injection and static-inline-injection
3. Moved ${{ inputs.cli_commands }} to env: block (CLI_COMMANDS) in 'Run CLI' step, with xargs-based tokenization into a bash array to safely split CLI arguments while preventing shell injection - fixes script-injection and static-inline-injection
4. Added printf/tr-based newline sanitization before writing to $GITHUB_OUTPUT - fixes github-env-injection
5. Pinned actions/upload-artifact@v4 to full SHA ea165f8d65b6e75b540449e92b4886f43607fa02 # v4 - fixes unpinned-uses


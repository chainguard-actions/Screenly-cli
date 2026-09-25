<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.0.5** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ inputs.* }} expressions are directly interpolated inside run: shell command strings, violating rule (a). This allows an attacker to inject arbitrary shell commands via the action inputs.

- Line 35: `${{ inputs.cli_version }}` is interpolated directly into a wget URL: `"https://github.com/Screenly/cli/releases/download/${{ inputs.cli_version }}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz"`
- Line 43: `${{ inputs.screenly_api_token }}` and `${{ inputs.cli_commands }}` are both interpolated directly into the shell command: `API_TOKEN=${{ inputs.screenly_api_token }} RUST_LOG=debug /tmp/screenly ${{ inputs.cli_commands }}`

All three expressions should be moved to `env:` variables and then referenced as double-quoted shell variables (e.g. `"$CLI_COMMANDS"`).

Locations:

- `action.yml:35`
- `action.yml:43`

### github-env-injection (severity: high)

The 'Run CLI' step writes the CLI command output directly to $GITHUB_OUTPUT without sanitization (line 49): `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"`. The CLI was invoked with the attacker-controlled `${{ inputs.cli_commands }}` expression directly interpolated (line 43), making the CLI output untrusted. An attacker could craft a cli_commands value whose output contains newlines that inject additional key=value pairs into GITHUB_OUTPUT. The value must be sanitized with `printf '%s' ... | tr -d '\n\r'` before being written to the special environment file.

Locations:

- `action.yml:49`

### unpinned-uses (severity: high)

The step 'Upload artifacts of failed screenly cli command' references `actions/upload-artifact@v4` using a mutable tag (`v4`) instead of a full 40-character commit SHA. This is vulnerable to supply-chain attacks where the tag could be moved to point to a different, potentially malicious commit. It should be pinned to a specific SHA, e.g. `actions/upload-artifact@65c4c4a1ddee5b72f698fdd19549f0f0fb45cf08 # v4`.

Locations:

- `action.yml:55`

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

Fixed all findings in hardened/action/action.yml:
1. script-injection / static-inline-injection: Moved ${{ inputs.cli_version }} to env: CLI_VERSION in 'Download CLI' step; moved ${{ inputs.screenly_api_token }} to env: API_TOKEN and ${{ inputs.cli_commands }} to env: CLI_COMMANDS in 'Run CLI' step. cli_commands is an args-style list input so it's tokenized via xargs into a bash array and expanded as "${cli_args[@]}".
2. github-env-injection: Sanitized the GITHUB_OUTPUT write with `safe_response=$(printf '%s' "$(cat /tmp/command_cleaned_output.txt)" | tr -d '\n\r')` before echoing to $GITHUB_OUTPUT.
3. unpinned-uses: Pinned actions/upload-artifact@v4 to full SHA ea165f8d65b6e75b540449e92b4886f43607fa02 with # v4 comment.


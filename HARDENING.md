<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.2.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Download CLI' step directly interpolates `${{ inputs.cli_version }}` inside a `run:` shell command URL string. This allows an attacker-controlled value to be injected into the shell command before the shell ever parses it. Offending line: `"https://github.com/Screenly/cli/releases/download/${{ inputs.cli_version }}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz"`

Locations:

- `action.yml:37`

### script-injection (severity: high)

Sub-rule (a) and (b): The 'Run CLI' step directly interpolates `${{ inputs.screenly_api_token }}` and `${{ inputs.cli_commands }}` inside a `run:` shell command string without quoting. Both values are attacker-controlled and are substituted into the shell command before the shell parses them, enabling arbitrary command injection. Offending line: `API_TOKEN=${{ inputs.screenly_api_token }} RUST_LOG=debug /tmp/screenly ${{ inputs.cli_commands }} >> /tmp/screenly_cli_command_output.txt`

Locations:

- `action.yml:49`

### github-env-injection (severity: high)

The 'Run CLI' step writes command output to $GITHUB_OUTPUT without sanitization. The content originates from the CLI command driven by the untrusted `${{ inputs.cli_commands }}` input. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing newline injection to poison the GitHub output context. Offending line: `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:53`

### unpinned-uses (severity: high)

The step 'Upload artifacts of failed screenly cli command' uses `actions/upload-artifact@v4`, which is a mutable tag reference rather than a pinned 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved to a malicious commit.

Locations:

- `action.yml:59`

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
1. Download CLI step: moved ${{ inputs.cli_version }} to env: block as CLI_VERSION, referenced as ${CLI_VERSION} in the URL string.
2. Run CLI step: moved ${{ inputs.screenly_api_token }} to env: block as API_TOKEN and ${{ inputs.cli_commands }} to env: block as CLI_COMMANDS. Since cli_commands is a list-style input, used xargs-based tokenization into a bash array (cli_args) with an empty-value guard, then expanded as "${cli_args[@]}" to preserve argument boundaries.
3. github-env-injection: Added safe_response=$(printf '%s' "$(cat /tmp/command_cleaned_output.txt)" | tr -d '\n\r') before writing to $GITHUB_OUTPUT.
4. Pinned actions/upload-artifact@v4 to full SHA ea165f8d65b6e75b540449e92b4886f43607fa02 with # v4 comment.


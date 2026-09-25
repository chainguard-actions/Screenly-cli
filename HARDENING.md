<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.0.3** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Download CLI' run: block directly interpolates ${{ inputs.cli_version }} inside a shell command (embedded in a URL string). Any expression inside a run: block is a script-injection risk because YAML template substitution happens before the shell ever sees the value. Offending line: `"https://github.com/Screenly/cli/releases/download/${{ inputs.cli_version }}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz"`

Locations:

- `action.yml:36`

### script-injection (severity: high)

Sub-rule (a): The 'Run CLI' run: block directly interpolates ${{ inputs.screenly_api_token }} and ${{ inputs.cli_commands }} inside a shell command. The inputs.cli_commands value is passed directly as shell arguments with no quoting or sanitization, allowing an attacker to inject arbitrary shell commands (e.g. by passing '; malicious-command'). Offending line: `API_TOKEN=${{ inputs.screenly_api_token }} RUST_LOG=debug /tmp/screenly ${{ inputs.cli_commands }} >> /tmp/screenly_cli_command_output.txt`

Locations:

- `action.yml:47`

### github-env-injection (severity: high)

The 'Run CLI' step writes command output to $GITHUB_OUTPUT using `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"`. The content of that file is derived from the execution of ${{ inputs.cli_commands }} (an untrusted input) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`) applied before the write. A newline in the output could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:52`

### unpinned-uses (severity: high)

The step 'Upload artifacts of failed screenly cli command' uses `actions/upload-artifact@v4`, which is pinned to a mutable version tag (`v4`) rather than an immutable 40-character commit SHA. A compromised or altered tag could introduce malicious code into the action. It should be pinned to a full SHA, e.g. `actions/upload-artifact@65c4c4a1ddee5b72f698fdd19549f0f0fb45cf08 # v4`.

Locations:

- `action.yml:58`

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

Fixed all 7 findings in action.yml:
1. Download CLI step: moved ${{ inputs.cli_version }} to env: block as CLI_VERSION, referenced as ${CLI_VERSION} in the shell.
2. Run CLI step: moved ${{ inputs.screenly_api_token }} to env: block as API_TOKEN and ${{ inputs.cli_commands }} to env: block as CLI_COMMANDS. Used xargs-based tokenization into a bash array (cli_args) to safely pass CLI_COMMANDS as separate arguments without shell injection.
3. github-env-injection: Added `safe_response=$(printf '%s' "$(cat /tmp/command_cleaned_output.txt)" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT to strip newlines that could inject additional key=value pairs.
4. unpinned-uses: Pinned actions/upload-artifact@v4 to full SHA ea165f8d65b6e75b540449e92b4886f43607fa02 with # v4 comment.


<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v26.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v26.8.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: ${{ inputs.cli_version }} is directly interpolated inside a run: shell command string in the 'Download CLI' step. An attacker-controlled input value is embedded verbatim into the wget URL before the shell ever sees it, enabling command injection. Example offending line: `"https://github.com/Screenly/cli/releases/download/${{ inputs.cli_version }}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz"`

Locations:

- `action.yml:36`

### script-injection (severity: high)

Rule (a) violation: ${{ inputs.screenly_api_token }} and ${{ inputs.cli_commands }} are directly interpolated inside a run: shell command string in the 'Run CLI' step. Both are attacker-controllable inputs embedded verbatim into the shell command, enabling command injection. Offending line: `API_TOKEN=${{ inputs.screenly_api_token }} RUST_LOG=debug /tmp/screenly ${{ inputs.cli_commands }} >> /tmp/screenly_cli_command_output.txt`

Locations:

- `action.yml:47`

### unpinned-uses (severity: high)

The step 'Upload artifacts of failed screenly cli command' references `actions/upload-artifact@v4`, which uses a mutable version tag (@v4) instead of a pinned 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved to point to a different commit.

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

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses

**Notes:**

Fixed all findings in hardened/action/action.yml:
1. 'Download CLI' step: moved ${{ inputs.cli_version }} to env: block as CLI_VERSION, referenced as ${CLI_VERSION} in the wget URL.
2. 'Run CLI' step: moved ${{ inputs.screenly_api_token }} to env: block as API_TOKEN and ${{ inputs.cli_commands }} as CLI_COMMANDS. CLI_COMMANDS is tokenized via xargs into a bash array (cli_args) to safely handle the argument list without injection. API_TOKEN is passed as an environment variable assignment.
3. Pinned actions/upload-artifact@v4 to full commit SHA ea165f8d65b6e75b540449e92b4886f43607fa02 with # v4 comment.


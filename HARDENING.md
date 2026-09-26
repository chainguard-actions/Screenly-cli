<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.2.1** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings. In the 'Download CLI' step, `${{ inputs.cli_version }}` is embedded directly in a wget URL string. In the 'Run CLI' step, `${{ inputs.screenly_api_token }}` is used as an inline env-var assignment and `${{ inputs.cli_commands }}` is passed directly as shell arguments — an attacker-controlled value that enables arbitrary command injection. All three must be moved to an env: block and the shell variables must be double-quoted.

Locations:

- `action.yml:37`
- `action.yml:47`

### github-env-injection (severity: high)

The 'Run CLI' step writes CLI output to $GITHUB_OUTPUT via `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"`. The file content is derived from the execution of `${{ inputs.cli_commands }}` — an untrusted input — and is written to the special environment file without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`) applied immediately before the write. A newline in the output could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:52`

### unpinned-uses (severity: high)

The step 'Upload artifacts of failed screenly cli command' references `actions/upload-artifact@v4`, which uses a mutable version tag instead of a full 40-character SHA commit hash. This is vulnerable to supply-chain attacks if the tag is moved to a different (potentially malicious) commit. Pin to a specific SHA, e.g. `actions/upload-artifact@65c4c4a1ddee5b72f698fdd19549f0f0fb45cf08 # v4`.

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

Fixed all findings in action.yml: (1) Moved ${{ inputs.cli_version }} to env: CLI_VERSION in 'Download CLI' step, referenced as ${CLI_VERSION} in the wget URL. (2) Moved ${{ inputs.screenly_api_token }} to env: API_TOKEN and ${{ inputs.cli_commands }} to env: INPUT_CLI_COMMANDS in 'Run CLI' step; cli_commands is tokenized via xargs into a bash array (args) to preserve argument boundaries while preventing injection. (3) Sanitized the CLI output before writing to $GITHUB_OUTPUT using `printf '%s' ... | tr -d '\n\r'` to prevent newline injection. (4) Pinned actions/upload-artifact@v4 to full SHA ea165f8d65b6e75b540449e92b4886f43607fa02 # v4.


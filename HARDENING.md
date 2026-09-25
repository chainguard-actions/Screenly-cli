<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.1.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ inputs.* }} expressions are directly interpolated inside run: shell command strings in action.yml, violating sub-rule (a). This allows an attacker who controls the calling workflow's inputs to inject arbitrary shell commands.

1. Line 36: `"https://github.com/Screenly/cli/releases/download/${{ inputs.cli_version }}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz"` — inputs.cli_version is interpolated directly into the wget URL.
2. Line 46: `API_TOKEN=${{ inputs.screenly_api_token }} RUST_LOG=debug /tmp/screenly ${{ inputs.cli_commands }}` — both inputs.screenly_api_token and inputs.cli_commands are interpolated directly into the shell command. inputs.cli_commands in particular allows full shell command injection (e.g. a value of `; malicious-command` would execute arbitrary code).

Fix: Move all inputs into env: variables and reference them as double-quoted shell variables (e.g. "$CLI_VERSION", "$CLI_COMMANDS").

Locations:

- `action.yml:36`
- `action.yml:46`

### github-env-injection (severity: high)

Line 52 writes the CLI command output to $GITHUB_OUTPUT without sanitization: `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"`. The content of /tmp/command_cleaned_output.txt is derived from the execution of ${{ inputs.cli_commands }} (an untrusted input), so a malicious input could embed newlines in the output to inject additional key=value pairs into $GITHUB_OUTPUT. The required sanitization step (`printf '%s' ... | tr -d '\n\r'`) is absent before the write.

Locations:

- `action.yml:52`

### unpinned-uses (severity: high)

The step 'Upload artifacts of failed screenly cli command' uses `actions/upload-artifact@v4`, which is a mutable tag reference. If the tag is moved (e.g. by a supply-chain compromise of the actions/upload-artifact repository), the action will silently execute different code. It must be pinned to a full 40-character commit SHA, e.g. `actions/upload-artifact@65c4c4a1ddee5b72f698fdd19549f0f0fb45cf08 # v4`.

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

Fixed all findings in action.yml:
1. script-injection / static-inline-injection: Moved ${{ inputs.cli_version }}, ${{ inputs.screenly_api_token }}, and ${{ inputs.cli_commands }} into env: blocks (CLI_VERSION, API_TOKEN, CLI_COMMANDS) and referenced them as shell variables. For cli_commands (a list-style input), used the xargs-based tokenization pattern to safely split it into an array (opts) preserving argument boundaries.
2. github-env-injection: Added sanitization with `printf '%s' ... | tr -d '\n\r'` before writing the CLI output to $GITHUB_OUTPUT.
3. unpinned-uses: Pinned actions/upload-artifact@v4 to full SHA ea165f8d65b6e75b540449e92b4886f43607fa02 with a # v4 comment.


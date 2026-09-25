<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.1.1** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings. In the 'Download CLI' step, `${{ inputs.cli_version }}` is embedded directly in a wget URL string (line 36). In the 'Run CLI' step, `${{ inputs.screenly_api_token }}` and `${{ inputs.cli_commands }}` are interpolated directly into a shell command (line 46). An attacker controlling these inputs can inject arbitrary shell commands. All three must be moved to env: variables and then double-quoted in the shell script.

Locations:

- `action.yml:36`
- `action.yml:46`

### github-env-injection (severity: high)

The 'Run CLI' step writes to $GITHUB_OUTPUT without sanitization (line 51): `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"`. The content of command_cleaned_output.txt is derived from executing `${{ inputs.cli_commands }}` — an untrusted caller-controlled input — and is written to GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A malicious input containing newlines could inject arbitrary key=value pairs into the GitHub output context.

Locations:

- `action.yml:51`

### unpinned-uses (severity: high)

The step 'Upload artifacts of failed screenly cli command' references `actions/upload-artifact@v4` (line 57), which uses a mutable tag instead of a pinned 40-character commit SHA. If the tag is moved or the referenced commit is changed, the action will silently execute different code, enabling supply-chain attacks. It should be pinned to a full SHA, e.g. `actions/upload-artifact@65c4c4a1ddee5b72f698fdd19549f0f0fb45cf08 # v4`.

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

Fixed all findings in action.yml:
1. script-injection / static-inline-injection: Moved ${{ inputs.cli_version }} to env var CLI_VERSION in 'Download CLI' step; moved ${{ inputs.screenly_api_token }} to SCREENLY_API_TOKEN and ${{ inputs.cli_commands }} to INPUT_CLI_COMMANDS in 'Run CLI' step. Shell scripts now reference these env vars instead of inline expressions.
2. cli_commands is a list-style input, so it is tokenized with xargs into a bash array (quote-aware, guarded with [ -n ]) and expanded as "${cli_args[@]}" to preserve argument boundaries.
3. github-env-injection: Sanitized the GITHUB_OUTPUT write using `printf '%s' ... | tr -d '\n\r'` to strip newlines before writing the response key.
4. unpinned-uses: Pinned actions/upload-artifact@v4 to full commit SHA ea165f8d65b6e75b540449e92b4886f43607fa02 with a # v4 comment.


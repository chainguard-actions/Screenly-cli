<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v26.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v26.9.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Download CLI' step directly interpolates the expression `${{ inputs.cli_version }}` inside a `run:` shell command string (embedded in a wget URL). This allows an attacker who controls the `cli_version` input to inject arbitrary shell commands. Offending line: `"https://github.com/Screenly/cli/releases/download/${{ inputs.cli_version }}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz"`

Locations:

- `action.yml:35`

### script-injection (severity: high)

Sub-rule (a): The 'Run CLI' step directly interpolates `${{ inputs.screenly_api_token }}` and `${{ inputs.cli_commands }}` inside a `run:` shell command string. Both are attacker-controlled inputs injected verbatim into the shell, enabling arbitrary command injection. Offending line: `API_TOKEN=${{ inputs.screenly_api_token }} RUST_LOG=debug /tmp/screenly ${{ inputs.cli_commands }} >> /tmp/screenly_cli_command_output.txt`

Locations:

- `action.yml:46`

### github-env-injection (severity: high)

The 'Run CLI' step writes CLI command output directly to $GITHUB_OUTPUT without sanitization: `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"`. The content of the file is derived from executing user-controlled `${{ inputs.cli_commands }}`, and may contain newline characters that inject additional key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' ... | tr -d '\n\r'`) is absent.

Locations:

- `action.yml:51`

### unpinned-uses (severity: high)

The step 'Upload artifacts of failed screenly cli command' uses `actions/upload-artifact@v4`, which is pinned to a mutable tag (`v4`) rather than an immutable 40-character SHA commit hash. This is vulnerable to supply-chain attacks if the tag is moved to a malicious commit.

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
1. Download CLI step: moved ${{ inputs.cli_version }} to env block as CLI_VERSION, referenced as ${CLI_VERSION} in the wget URL.
2. Run CLI step: moved ${{ inputs.screenly_api_token }} to env block as API_TOKEN and ${{ inputs.cli_commands }} to env block as CLI_COMMANDS. Used xargs-based tokenization into a bash array (cli_args) for CLI_COMMANDS since it is a list of arguments.
3. github-env-injection: Added sanitization with `printf '%s' ... | tr -d '\n\r'` before writing the response to $GITHUB_OUTPUT.
4. Pinned actions/upload-artifact@v4 to immutable SHA ea165f8d65b6e75b540449e92b4886f43607fa02 with # v4 comment.


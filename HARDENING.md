<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v26.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v26.11.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Download CLI' step puts ${{ inputs.cli_version }} straight into the run: script: "https://github.com/Screenly/cli/releases/download/${{ inputs.cli_version }}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz". Fix: pass it through an env: variable and use it double-quoted.

Locations:

- `action.yml:37`

### script-injection (severity: high)

Rule (a): the 'Run CLI' step puts ${{ inputs.screenly_api_token }} and ${{ inputs.cli_commands }} straight into the run: script, unquoted: "API_TOKEN=${{ inputs.screenly_api_token }} RUST_LOG=debug /tmp/screenly ${{ inputs.cli_commands }}  >> /tmp/screenly_cli_command_output.txt". Shell metacharacters in either value get executed.

Locations:

- `action.yml:49`

### github-env-injection (severity: high)

The 'Run CLI' step writes the CLI output to $GITHUB_OUTPUT with: echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT". That output comes from a command run with workflow-supplied inputs. The value is not passed through printf '%s' ... | tr -d '\n\r' first (only tr '\n' ' ' runs, so carriage returns stay), and it is not written between random heredoc delimiters.

Locations:

- `action.yml:53`

### unpinned-uses (severity: high)

A composite step uses a tag, not a commit SHA: 'uses: actions/upload-artifact@v4'.

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

I fixed all seven findings in action.yml:

- **Download CLI step:** `inputs.cli_version` now comes in through `env:` as `CLI_VERSION` and is used inside the double-quoted URL.
- **Run CLI step:**
  - The API token now goes through the `INPUT_API_TOKEN` env variable. It is passed to the CLI as `API_TOKEN="$INPUT_API_TOKEN"`.
  - `cli_commands` now goes through the `INPUT_CLI_COMMANDS` env variable. Upstream pasted the expression straight into the script, so the shell honoured quotes in the value. To keep that behaviour, the value is split into an array with the quote-aware xargs method. That includes the guard for an empty value, `xargs -r`, and the `xargs -r true` check that fails the step on an unmatched quote. The array is then passed as `"${args[@]}"`.
- **Output written to `$GITHUB_OUTPUT`:** the cleaned output is now read into a variable first, using `|| true`, because upstream's `echo` also hid a failure here. Newlines and carriage returns are stripped with `printf '%s' | tr -d '\n\r'` before the value is written.
- **upload-artifact:** pinned to `actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4`.

Nothing was run or tested in this pass. Two behaviour changes to know about:
- A `cli_commands` value with an unmatched quote now fails the step instead of passing through.
- Carriage returns are now removed from the `response` output; upstream left them in.


<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v26.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v26.8.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Download CLI' step puts ${{ inputs.cli_version }} straight into the run script: "https://github.com/Screenly/cli/releases/download/${{ inputs.cli_version }}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz". Fix: pass the value in through an env: variable and use it double-quoted.

Locations:

- `action.yml:37`

### script-injection (severity: high)

Rule (a): the 'Run CLI' step puts ${{ inputs.screenly_api_token }} and ${{ inputs.cli_commands }} straight into the run script, unquoted: `API_TOKEN=${{ inputs.screenly_api_token }} RUST_LOG=debug /tmp/screenly ${{ inputs.cli_commands }}  >> /tmp/screenly_cli_command_output.txt`. Shell metacharacters in either input would run as commands.

Locations:

- `action.yml:49`

### github-env-injection (severity: high)

The 'Run CLI' step writes the output of a CLI command built from inputs.cli_commands to $GITHUB_OUTPUT (`echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"`). The required sanitization step (printf '%s' ... | tr -d '\n\r') is missing. The earlier `tr '\n' ' '` does not remove carriage returns, and no random heredoc delimiter is used.

Locations:

- `action.yml:53`

### unpinned-uses (severity: high)

action.yml uses a tag instead of a full commit SHA: `uses: actions/upload-artifact@v4`.

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 9fa4ef7..ecc8f7f 100644
--- a/action.yml
+++ b/action.yml
@@ -1,4 +1,3 @@
----
 # action.yml
 name: "Screenly"
 description: "GitHub Actions for Screenly."
@@ -20,48 +19,71 @@ inputs:
   cli_version:
     description: "Screenly CLI version."
     default: "v26.8.0"
-
 outputs:
   cli_commands_response:
     description: "The response from the Screenly CLI command(s)."
     value: ${{ steps.run-cli.outputs.response }}
-
 runs:
   using: "composite"
   steps:
     - name: Download CLI
       id: download-cli
+      env:
+        CLI_VERSION: ${{ inputs.cli_version }}
       run: |
         pushd /tmp
         wget -q -O screenly-cli.tar.gz \
-          "https://github.com/Screenly/cli/releases/download/${{ inputs.cli_version }}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz"
+          "https://github.com/Screenly/cli/releases/download/${CLI_VERSION}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz"
         tar xfz screenly-cli.tar.gz
         chmod +x screenly
         popd
       shell: bash
-
     - name: Run CLI
       id: run-cli
       continue-on-error: true
+      env:
+        API_TOKEN: ${{ inputs.screenly_api_token }}
+        CLI_COMMANDS: ${{ inputs.cli_commands }}
       run: |
         set -o pipefail
 
-        API_TOKEN=${{ inputs.screenly_api_token }} RUST_LOG=debug /tmp/screenly ${{ inputs.cli_commands }}  >> /tmp/screenly_cli_command_output.txt
+        cli_args=()
+        if [ -n "$CLI_COMMANDS" ]; then
+          while IFS= read -r -d '' t; do cli_args+=("$t"); done \
+            < <(printf '%s' "$CLI_COMMANDS" | xargs printf '%s\0')
+        fi
+
+        API_TOKEN="$API_TOKEN" RUST_LOG=debug /tmp/screenly "${cli_args[@]}" >> /tmp/screenly_cli_command_output.txt
 
         # Filter logs from github output, while still saving them as artifact. Filtered lines are like: 2024-10-16T13:16:13.974Z DEBUG [reqwest::connect] ...
         grep -Ev '^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}\.[0-9]{3}Z[[:space:]](DEBUG|INFO|WARNING|ERROR)' /tmp/screenly_cli_command_output.txt | tr '\n' ' ' > /tmp/command_cleaned_output.txt
         echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"
         cat /tmp/command_cleaned_output.txt
       shell: bash
-
     - name: Upload artifacts of failed screenly cli command
       if: ${{ steps.run-cli.outcome != 'success' }}
       uses: actions/upload-artifact@v4
       with:
         name: run-cli-output
         path: |
           /tmp/screenly_cli_command_output.txt
-
     - name: Fail if cli command failed
       shell: bash
       if: ${{ steps.run-cli.outcome != 'success' }}

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unpinned-uses, prior-fix

**Notes:**

I fixed all the findings in action.yml, re-applying and extending the previous harden. Nothing was run or tested.

- **Download CLI step:** `inputs.cli_version` now comes in through the env var `CLI_VERSION` and is used double-quoted in the wget URL.
- **Run CLI step:** the API token and the CLI commands now come in as env vars (`API_TOKEN`, `CLI_COMMANDS`). Upstream pasted the commands straight into the script, so the shell honoured quotes in them. To keep that behaviour, the commands are split into an array with the quote-aware xargs method: `xargs -r true` first, so an unmatched quote fails the step, then the shell's built-in printf through `sh -c` to do the split. Shell characters in the input (`;`, `|`, `$(...)`, backticks) now reach the CLI as plain text and are never run.
- **Step output:** the value written to `$GITHUB_OUTPUT` is now read into a variable first (with `|| true`, because upstream's echo hid any failure there), then has both `\n` and `\r` removed with `tr -d '\n\r'` before it is written.
- **upload-artifact:** pinned to `actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4`.

One behaviour difference from the previous harden: its split used `xargs printf`, so a command list like `--version` could be taken as an option by the external printf and dropped. The new split uses the shell's built-in printf, which avoids that.


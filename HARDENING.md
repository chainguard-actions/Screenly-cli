<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v26.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v26.9.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Download CLI' step puts an expression directly inside the run: script: "https://github.com/Screenly/cli/releases/download/${{ inputs.cli_version }}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz". Move inputs.cli_version into an env: variable and reference it double-quoted ("${CLI_VERSION}").

Locations:

- `action.yml:37`

### script-injection (severity: high)

Rule (a): the 'Run CLI' step puts expressions directly inside the run: script: `API_TOKEN=${{ inputs.screenly_api_token }} RUST_LOG=debug /tmp/screenly ${{ inputs.cli_commands }}  >> /tmp/screenly_cli_command_output.txt`. Whoever sets inputs.cli_commands or the token can make the shell run any command. Pass both through env: variables and expand them double-quoted.

Locations:

- `action.yml:49`

### github-env-injection (severity: high)

The 'Run CLI' step writes the CLI output to $GITHUB_OUTPUT with `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"`. That output comes from commands set by inputs.cli_commands. The script only changes newlines to spaces (`tr '\n' ' '`). It never runs the required `printf '%s' ... | tr -d '\n\r'` step, and carriage returns are left in. It also does not use a heredoc with a random delimiter.

Locations:

- `action.yml:53`

### unpinned-uses (severity: high)

The composite step uses a tag instead of a full commit SHA: `uses: actions/upload-artifact@v4`.

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
index 1b4f37b..a94aa4d 100644
--- a/action.yml
+++ b/action.yml
@@ -1,4 +1,3 @@
----
 # action.yml
 name: "Screenly"
 description: "GitHub Actions for Screenly."
@@ -20,48 +19,72 @@ inputs:
   cli_version:
     description: "Screenly CLI version."
     default: "v26.9.0"
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
-        echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"
+        safe_response=$(printf '%s' "$(cat /tmp/command_cleaned_output.txt)" | tr -d '\n\r')
+        echo "response=${safe_response}" >> "$GITHUB_OUTPUT"
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

All fixes are in action.yml. I haven't run the action to test them.

- **Download CLI step:** `inputs.cli_version` now comes in through `env:` as `CLI_VERSION` and is used double-quoted in the download URL.
- **Run CLI step, token:** the API token now comes in through `env:` as `INPUT_API_TOKEN` and is passed to the CLI as `API_TOKEN="$INPUT_API_TOKEN"`.
- **Run CLI step, commands:** `inputs.cli_commands` now comes in through `env:` as `CLI_COMMANDS`. Upstream pasted the expression straight into the script, so the shell honoured quotes inside it. To keep that, the value is split with the quote-aware xargs method: an empty-value guard, a `xargs -r true` check that stops on an unmatched quote, then a split into a bash array using the shell's own `printf` through `sh -c ... sh`. Nothing in the value is run as a command.
- **Response output:** the response is read into a variable first. `|| true` keeps upstream's behaviour of not stopping the step if reading the file fails. Newlines and carriage returns are stripped with `printf '%s' ... | tr -d '\n\r'` before the value is written to `$GITHUB_OUTPUT`.
- **Upload step:** `actions/upload-artifact@v4` is now pinned to commit `ea165f8d65b6e75b540449e92b4886f43607fa02`, with a `# v4` comment.

These changes cover everything the previous harden did. Two parts are stricter than that patch: the commands split now catches an unmatched quote, and it uses the shell's built-in `printf` instead of the external one.


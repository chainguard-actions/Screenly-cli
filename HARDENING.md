<!-- markdownlint-disable -->

# Hardening Report: Screenly--cli/v1.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Screenly--cli/v1.0.3** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Download CLI' step directly interpolates `${{ inputs.cli_version }}` inside a `run:` shell command (inside a wget URL string). An attacker-controlled value can inject shell metacharacters before the shell ever sees the string. Offending line: `"https://github.com/Screenly/cli/releases/download/${{ inputs.cli_version }}/screenly-cli-x86_64-unknown-linux-gnu.tar.gz"`

Locations:

- `action.yml:35`

### script-injection (severity: high)

Sub-rule (a): The 'Run CLI' step directly interpolates `${{ inputs.screenly_api_token }}` and `${{ inputs.cli_commands }}` inside a `run:` shell command. Both are attacker-controllable inputs that are template-substituted before the shell parses the command, enabling command injection. Offending line: `API_TOKEN=${{ inputs.screenly_api_token }} RUST_LOG=debug /tmp/screenly ${{ inputs.cli_commands }} >> /tmp/screenly_cli_command_output.txt`

Locations:

- `action.yml:44`

### github-env-injection (severity: high)

The 'Run CLI' step writes `echo "response=$(cat /tmp/command_cleaned_output.txt)" >> "$GITHUB_OUTPUT"` without sanitizing newlines. The file content is derived from executing user-controlled `${{ inputs.cli_commands }}`, so an attacker can craft CLI output containing newline characters to inject arbitrary key=value pairs into GITHUB_OUTPUT. The required sanitization (`printf '%s' ... | tr -d '\n\r'`) is absent.

Locations:

- `action.yml:48`

### unpinned-uses (severity: high)

The step 'Upload artifacts of failed screenly cli command' references `actions/upload-artifact@v4`, which uses a mutable tag (`v4`) instead of a pinned 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved. It should be pinned to a full SHA, e.g. `actions/upload-artifact@65c4c4a1ddee5b72f698fdd19549f0f0fb45cf08 # v4`.

Locations:

- `action.yml:53`

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
1. 'Download CLI' step: moved `${{ inputs.cli_version }}` to env block as CLI_VERSION, referenced as ${CLI_VERSION} in the wget URL.
2. 'Run CLI' step: moved `${{ inputs.screenly_api_token }}` to env block as API_TOKEN and `${{ inputs.cli_commands }}` to env block as CLI_COMMANDS; both referenced as plain env vars in the shell script.
3. 'Run CLI' step: sanitized GITHUB_OUTPUT write by capturing cleaned output into a variable and stripping newlines with `printf '%s' ... | tr -d '\n\r'` before echoing to $GITHUB_OUTPUT.
4. 'Upload artifacts' step: pinned `actions/upload-artifact@v4` to full SHA `ea165f8d65b6e75b540449e92b4886f43607fa02 # v4`.

### Iteration 2

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all findings across 7 workflow files and action.yml:

1. unpinned-uses: Pinned all action references to full SHA hashes with tag comments:
   - actions/checkout@v2 → @0717577d45739eb3c851188b29f50ed6c0b2194e
   - actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
   - screenly/cli@master → @3d6a92abedc446b3b89e68ea134a33f0025ae564
   - dorny/paths-filter@v3 → @d1c1ffe0248fe513906c8e24db8ea791d46f8590
   - dtolnay/rust-toolchain@master → @2c7215f132e9ebf062739d9130488b56d53c060c
   - actions-rs/clippy-check@v1 → @b5b5f21f4797c02da247df37026fcd0a5024aa4d
   - actions-rust-lang/rustfmt@v1 → @4066006ec54a31931b9b1fddfd38f2fdf2d27143
   - DeterminateSystems/nix-installer-action@v8 → @07b8bcba1b22d847db7ee507180c33e115499665
   - DeterminateSystems/magic-nix-cache-action@main → @55718cd9ebf1737a061487d5ada21abf31e99a98
   - softprops/action-gh-release@v1 → @de2c0eb89ae2a093876385947365aca7b0e5f844
   - actions/attest-build-provenance@v1 → @ef244123eb79f2f7a7e75d99086184180e6d0018
   - docker/login-action@v1 → @dd4fa0671be5250ee6f50aedf4cb05514abda2c7
   - sbomify/github-action@master → @c65a2b9fc24fc69f3376be690c430571be49c173

2. missing-permissions: Added top-level `permissions: {}` to actions.yml, docs.yml, nix.yml, release.yml, rust.yml. Added job-level permissions to previously uncovered jobs.

3. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks in release.yml and docs.yml. Fixed unquoted $CLI_COMMANDS in action.yml using bash array expansion.

4. github-env-injection: Fixed Use Cross step in release.yml to sanitize matrix.target with printf/tr before writing to $GITHUB_ENV.


<!-- markdownlint-disable -->

# Hardening Report: ctfer-io--ctfd-setup/v1.8.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ctfer-io--ctfd-setup/v1.8.3** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ github.ref_name }}` is directly interpolated inside a `run:` shell script in the 'Git commit date' step. The expression is substituted by the GitHub Actions template engine before the shell executes it, allowing a specially crafted ref name (e.g. a tag like `v1.0; malicious-command`) to inject arbitrary shell commands. Offending line: `version=${{ github.ref_name }}`

Locations:

- `.github/workflows/docker.yaml:44`

### github-env-injection (severity: high)

The variable `$version` — derived directly from `${{ github.ref_name }}` without sanitization — is written to `$GITHUB_OUTPUT` via `echo "version=$version" >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization step is applied before the write, so a newline-containing ref name could inject additional key=value pairs into the output context.

Locations:

- `.github/workflows/docker.yaml:46`

### unpinned-uses (severity: high)

Two workflow files reference `slsa-framework/slsa-github-generator` reusable workflows at the mutable tag `@v2.1.0` instead of a full 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling a supply-chain attack.
- `.github/workflows/docker.yaml`: `uses: slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@v2.1.0`
- `.github/workflows/release.yaml`: `uses: slsa-framework/slsa-github-generator/.github/workflows/generator_generic_slsa3.yml@v2.1.0`

Locations:

- `.github/workflows/docker.yaml:65`
- `.github/workflows/release.yaml:47`

### missing-permissions (severity: medium)

`.github/workflows/lint.yaml` has no top-level `permissions:` key and its only job (`lint`) also has no `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be broader than necessary (e.g. `contents: write` on some repositories).

Locations:

- `.github/workflows/lint.yaml:1`

### broad-permissions (severity: medium)

`.github/workflows/scoreboard.yaml` sets `permissions: read-all` at the top level. This grants read access to all available GitHub token scopes (contents, packages, pull-requests, issues, etc.) rather than the minimal specific scopes actually required by the workflow.

Locations:

- `.github/workflows/scoreboard.yaml:7`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, broad-permissions

**Notes:**

Fixed all 5 findings: (1) script-injection in docker.yaml 'Git commit date' step - moved ${{ github.ref_name }} into env block as REF_NAME; (2) github-env-injection - added printf/tr sanitization before writing version to GITHUB_OUTPUT; (3) unpinned-uses - pinned both slsa-framework/slsa-github-generator workflow references to full SHA f7dd8c54c2067bafc12ca7a55595d5ee9b75204a # v2.1.0 in both docker.yaml and release.yaml; (4) missing-permissions - added 'permissions: contents: read' top-level block to lint.yaml; (5) broad-permissions - replaced 'permissions: read-all' with 'permissions: contents: read' in scoreboard.yaml (job-level permissions already specify the needed write scopes).

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Generate subject' step in .github/workflows/release.yaml (line 42). The fix captures the base64-encoded hashes into a variable, then sanitizes it with `printf '%s' "$hashes" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT. This prevents newline injection from a workflow-controllable step output (steps.run-goreleaser.outputs.artifacts) from poisoning the GITHUB_OUTPUT file. Also fixed the unquoted `$checksum_file` variable to `"$checksum_file"` to prevent word splitting.


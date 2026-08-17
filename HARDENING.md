<!-- markdownlint-disable -->

# Hardening Report: ctfer-io--ctfd-setup/v1.8.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ctfer-io--ctfd-setup/v1.8.5** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Git commit date' run: block in docker.yaml directly interpolates `${{ github.ref_name }}` into the shell script as `version=${{ github.ref_name }}`. A crafted tag name containing shell metacharacters (e.g., `;`, `$(...)`) would be executed by the shell before any quoting can protect it.

Locations:

- `.github/workflows/docker.yaml:50`

### github-env-injection (severity: high)

The 'Git commit date' step in docker.yaml writes the value of `${{ github.ref_name }}` (stored in the shell variable `version`) to `$GITHUB_OUTPUT` via `echo "version=$version" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled tag name containing newlines could inject arbitrary key-value pairs into the GitHub output context.

Locations:

- `.github/workflows/docker.yaml:52`

### unpinned-uses (severity: high)

Two workflow files reference `slsa-framework/slsa-github-generator` reusable workflows pinned to a mutable version tag (`@v2.1.0`) instead of a full 40-character commit SHA. This exposes the workflow to supply-chain attacks if the tag is moved. Failing references: (1) `slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@v2.1.0` in docker.yaml; (2) `slsa-framework/slsa-github-generator/.github/workflows/generator_generic_slsa3.yml@v2.1.0` in release.yaml.

Locations:

- `.github/workflows/docker.yaml:72`
- `.github/workflows/release.yaml:52`

### missing-permissions (severity: medium)

The lint.yaml workflow has no top-level `permissions:` key and its only job (`lint`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents, packages, etc.).

Locations:

- `.github/workflows/lint.yaml:1`

### broad-permissions (severity: medium)

The scoreboard.yaml workflow sets `permissions: read-all` at the top level. This grants read access to all available scopes (contents, packages, pull-requests, issues, etc.) rather than the minimal specific permissions required by the workflow.

Locations:

- `.github/workflows/scoreboard.yaml:9`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, broad-permissions

**Notes:**

Fixed all 5 findings: (1) script-injection in docker.yaml: moved `${{ github.ref_name }}` into an env var `REF_NAME` and referenced it as `$REF_NAME` in the shell; (2) github-env-injection in docker.yaml: added `printf '%s' "$version" | tr -d '\n\r'` sanitization before writing to GITHUB_OUTPUT; (3) unpinned-uses in both docker.yaml and release.yaml: pinned slsa-framework/slsa-github-generator workflows to full SHA `f7dd8c54c2067bafc12ca7a55595d5ee9b75204a` (v2.1.0) with tag preserved as comment; (4) missing-permissions in lint.yaml: added top-level `permissions: contents: read`; (5) broad-permissions in scoreboard.yaml: replaced `permissions: read-all` with `permissions: contents: read` (job-level already has specific write permissions needed).


<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.72.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.72.0** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are interpolated directly inside run: shell command strings, violating sub-rule (a). Step 1 ('Sleep a bit') uses `sleep ${{ inputs.sleep-time }}` — an attacker-controlled input injected directly into a shell command. Step 2 ('Wait for deployment to be ready and get URL') embeds ${{ inputs.cloudflare-account-id }}, ${{ inputs.cloudflare-project-name }}, ${{ inputs.cloudflare-api-token }}, ${{ inputs.wait-till-ready-seconds }}, ${{ inputs.deployment-poll-seconds }}, and ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }} directly into shell commands (curl URL/headers, arithmetic, sleep, and jq filter strings). Any of these values can contain shell metacharacters that execute arbitrary commands before the shell ever sees them.

Locations:

- `action.yml:34`
- `action.yml:40`

### unpinned-uses (severity: high)

The composite action step uses `altinukshini/deployment-action@releases/v1`, which is pinned to a mutable branch ref (`releases/v1`) rather than an immutable 40-character commit SHA. This means the action can be silently updated or compromised without any change to this file, creating a supply-chain risk.

Locations:

- `action.yml:62`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sleep-time }}" appears directly in run: block of step "Sleep a bit to wait for the deployment to exist"; move to env: map

Locations:

- `action.yml:35`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cloudflare-account-id }}" appears directly in run: block of step "Wait for deployment to be ready and get URL"; move to env: map

Locations:

- `action.yml:44`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cloudflare-project-name }}" appears directly in run: block of step "Wait for deployment to be ready and get URL"; move to env: map

Locations:

- `action.yml:44`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cloudflare-api-token }}" appears directly in run: block of step "Wait for deployment to be ready and get URL"; move to env: map

Locations:

- `action.yml:44`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.wait-till-ready-seconds }}" appears directly in run: block of step "Wait for deployment to be ready and get URL"; move to env: map

Locations:

- `action.yml:47`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deployment-poll-seconds }}" appears directly in run: block of step "Wait for deployment to be ready and get URL"; move to env: map

Locations:

- `action.yml:52`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deployment-poll-seconds }}" appears directly in run: block of step "Wait for deployment to be ready and get URL"; move to env: map

Locations:

- `action.yml:53`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses

**Notes:**

Fixed all script injection findings by moving all ${{ }} expressions from run: shell strings into env: blocks. The cloudflare-account-id, cloudflare-project-name, cloudflare-api-token, wait-till-ready-seconds, deployment-poll-seconds, and sleep-time inputs are now passed as environment variables. The commit SHA expression (github.event_name == 'pull_request' && ...) is also moved to an env var (COMMIT_SHA) and passed to jq via --arg to avoid injection into the jq filter string. Pinned altinukshini/deployment-action from mutable branch ref 'releases/v1' to immutable SHA fe2fba14b486343ce6893cff73b80e153063917a.


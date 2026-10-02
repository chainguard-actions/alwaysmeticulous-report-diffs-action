<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.80.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.80.0** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are directly interpolated into run: shell command strings (sub-rule a), allowing script injection. Affected lines include:
- Line 33: `run: sleep ${{ inputs.sleep-time }}` — input injected directly into shell
- Line 44: `curl ... "${{ inputs.cloudflare-account-id }}" ... "${{ inputs.cloudflare-project-name }}" ... "${{ inputs.cloudflare-api-token }}"` — inputs injected into curl command
- Line 45: `jq ... "${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}"` — github context injected into jq argument
- Line 48: `if [[ $SLEPT -gt ${{ inputs.wait-till-ready-seconds }} ]]` — input injected into arithmetic comparison
- Line 51: `sleep ${{ inputs.deployment-poll-seconds }}` — input injected into shell
- Line 52: `SLEPT=$((SLEPT+${{ inputs.deployment-poll-seconds }}))` — input injected into arithmetic expansion
All of these should be moved to env: variables and referenced as quoted shell variables (e.g., "$VAR").

Locations:

- `action.yml:33`
- `action.yml:44`
- `action.yml:45`
- `action.yml:48`
- `action.yml:51`
- `action.yml:52`

### github-env-injection (severity: high)

Lines 55 and 56 write values to $GITHUB_OUTPUT that are derived from untrusted github context expressions (${{ github.event.pull_request.head.sha }}, ${{ github.sha }}) embedded directly in jq filter strings inside backtick command substitutions, without the required sanitization step (printf '%s' ... | tr -d '\n\r') before the write. Specifically:
- `echo "url=`...jq ... "${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}"...`" >> $GITHUB_OUTPUT`
- `echo "environment=`...jq ... "${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}"...`" >> $GITHUB_OUTPUT`
These writes to GITHUB_OUTPUT must be preceded by sanitization to prevent newline injection attacks.

Locations:

- `action.yml:55`
- `action.yml:56`

### unpinned-uses (severity: high)

The composite action step uses `altinukshini/deployment-action@releases/v1`, which references a branch name (`releases/v1`) rather than a full 40-character commit SHA. This means the action can be silently updated or compromised without any change to this file, creating a supply-chain attack risk. It should be pinned to a specific commit SHA, e.g., `altinukshini/deployment-action@<40-char-sha> # releases/v1`.

Locations:

- `action.yml:58`

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

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all findings in action.yml:
1. Moved all ${{ }} expressions from run: blocks into env: maps (SLEEP_TIME, CF_ACCOUNT_ID, CF_PROJECT_NAME, CF_API_TOKEN, COMMIT_HASH, WAIT_TILL_READY_SECONDS, DEPLOYMENT_POLL_SECONDS). The COMMIT_HASH is also passed to jq via --arg to avoid injection in the jq filter string.
2. Sanitized GITHUB_OUTPUT writes using printf '%s' "$var" | tr -d '\n\r' before writing url and environment values.
3. Pinned altinukshini/deployment-action@releases/v1 to @fe2fba14b486343ce6893cff73b80e153063917a # releases/v1.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted expansions of input-derived env vars in the run: script at action.yml line 57: (1) quoted $WAIT_TILL_READY_SECONDS in the [[ ]] conditional: `[[ $SLEPT -gt "$WAIT_TILL_READY_SECONDS" ]]`; (2) quoted $DEPLOYMENT_POLL_SECONDS in the arithmetic expansion: `$(("$SLEPT"+"$DEPLOYMENT_POLL_SECONDS"))`. Both variables are sourced from inputs.* and must be double-quoted wherever expanded in shell.


<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.79.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.79.0** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple GitHub Actions expressions (`${{ ... }}`) are directly interpolated inside `run:` shell command strings (sub-rule a), allowing script injection. Affected expressions include:
- `run: sleep ${{ inputs.sleep-time }}` (line 35) — an attacker-controlled input is injected directly into the shell command.
- `LAST_RESULT=$(curl ... "https://api.cloudflare.com/.../accounts/${{ inputs.cloudflare-account-id }}/pages/projects/${{ inputs.cloudflare-project-name }}/..." -H "Authorization: Bearer ${{ inputs.cloudflare-api-token }}" ...)` (line 45) — three inputs are interpolated directly into a curl command string, enabling command injection via shell metacharacters.
- `STATUS=$(echo "$LAST_RESULT" | jq -c '... == "${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}" ...' ...)` (line 46) — github context values are injected into a jq filter string inside a shell command.
- `if [[ $SLEPT -gt ${{ inputs.wait-till-ready-seconds }} ]]` (line 48) — input injected into arithmetic comparison.
- `sleep ${{ inputs.deployment-poll-seconds }}` (line 52) and `SLEPT=$((SLEPT+${{ inputs.deployment-poll-seconds }}))` (line 53) — input injected into sleep and arithmetic expansion.
All of these must be moved to `env:` variables and referenced as double-quoted shell variables (e.g., `"$INPUT_VAR"`) instead.

Locations:

- `action.yml:35`
- `action.yml:45`
- `action.yml:46`
- `action.yml:48`
- `action.yml:52`
- `action.yml:53`

### github-env-injection (severity: high)

The `run:` block writes values to `$GITHUB_OUTPUT` where the jq filter strings contain directly interpolated `${{ github.event.pull_request.head.sha || github.sha }}` expressions (lines 57–58). Although `tr -d '\n'` is applied to the jq output value before the write, the `${{ }}` expression is raw-interpolated into the shell script string itself — an attacker who controls `github.event.pull_request.head.sha` could inject shell metacharacters or newlines into the jq filter, breaking out of the string context and injecting arbitrary content into `$GITHUB_OUTPUT`. The fix requires moving the SHA values into `env:` variables and sanitizing them with `printf '%s' "$VAR" | tr -d '\n\r'` before use.

Locations:

- `action.yml:57`
- `action.yml:58`

### unpinned-uses (severity: high)

The composite action step uses `altinukshini/deployment-action@releases/v1`, which is pinned to a mutable branch ref (`releases/v1`) rather than an immutable 40-character commit SHA. This means the referenced action can be silently changed by the upstream repository owner, enabling a supply-chain attack. It must be pinned to a full SHA, e.g. `altinukshini/deployment-action@<40-char-sha> # releases/v1`.

Locations:

- `action.yml:60`

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

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, static-inline-injection

**Notes:**

Fixed all security findings in hardened/action/action.yml:

1. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: blocks to env: variables. inputs.sleep-time → SLEEP_TIME, inputs.cloudflare-account-id → CLOUDFLARE_ACCOUNT_ID, inputs.cloudflare-project-name → CLOUDFLARE_PROJECT_NAME, inputs.cloudflare-api-token → CLOUDFLARE_API_TOKEN, inputs.wait-till-ready-seconds → WAIT_TILL_READY_SECONDS, inputs.deployment-poll-seconds → DEPLOYMENT_POLL_SECONDS, and the SHA expression → COMMIT_SHA. The SHA is passed to jq via --arg (not string interpolation) to prevent injection into the jq filter.

2. **github-env-injection**: The COMMIT_SHA env var is sanitized with `printf '%s' "$COMMIT_SHA" | tr -d '\n\r'` before use. The url and environment values are captured in shell variables and written to GITHUB_OUTPUT separately, preventing newline injection.

3. **unpinned-uses**: Pinned altinukshini/deployment-action from mutable branch ref `releases/v1` to full commit SHA `fe2fba14b486343ce6893cff73b80e153063917a` with the branch ref preserved as a comment.


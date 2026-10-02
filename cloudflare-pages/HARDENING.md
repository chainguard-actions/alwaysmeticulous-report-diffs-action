<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.77.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.77.0** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ }}` expressions are interpolated directly inside `run:` shell command strings, allowing script injection. Step 1 (line 33): `run: sleep ${{ inputs.sleep-time }}` — the `inputs.sleep-time` value is injected directly into the shell command. Step 2 (lines 43–54): `${{ inputs.cloudflare-account-id }}`, `${{ inputs.cloudflare-project-name }}`, and `${{ inputs.cloudflare-api-token }}` are interpolated directly into a `curl` command; `${{ inputs.wait-till-ready-seconds }}` and `${{ inputs.deployment-poll-seconds }}` are interpolated into arithmetic and `sleep` commands; and `${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}` is interpolated into jq filter strings (including those that write to $GITHUB_OUTPUT). All of these bypass shell quoting and allow an attacker-controlled value to inject arbitrary shell commands.

Locations:

- `action.yml:33`
- `action.yml:43`
- `action.yml:44`
- `action.yml:46`
- `action.yml:49`
- `action.yml:50`
- `action.yml:53`
- `action.yml:54`

### github-env-injection (severity: high)

Lines 53–54 write values to `$GITHUB_OUTPUT` that are derived from `${{ github.event.pull_request.head.sha }}` and `${{ github.sha }}` (attacker-controllable via pull requests). The `${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}` expression is interpolated directly into the jq filter string embedded in the shell command, and the resulting jq output (URL and environment strings) is written to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`) applied to the full value before the write. The `tr -d '\n'` inside the backtick substitution only strips newlines from the jq output value, but does not protect against newline injection from the `${{ }}` expression itself being expanded into the shell script before execution.

Locations:

- `action.yml:53`
- `action.yml:54`

### unpinned-uses (severity: high)

The composite action step uses `altinukshini/deployment-action@releases/v1`, which is pinned to a mutable branch ref (`releases/v1`) rather than an immutable 40-character commit SHA. This means the referenced action can be silently changed by the upstream repository owner, enabling a supply-chain attack.

Locations:

- `action.yml:56`

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

Fixed all findings in action.yml:
1. Moved all ${{ }} expressions from run: shell strings into env: blocks (SLEEP_TIME, CF_ACCOUNT_ID, CF_PROJECT_NAME, CF_API_TOKEN, WAIT_TILL_READY_SECONDS, DEPLOYMENT_POLL_SECONDS, COMMIT_HASH). The commit hash expression used inside jq filters is now passed safely via jq's --arg mechanism instead of being interpolated into the filter string.
2. Sanitized GITHUB_OUTPUT writes with `printf '%s' "$val" | tr -d '\n\r'` before writing url and environment values.
3. Pinned altinukshini/deployment-action@releases/v1 to immutable SHA fe2fba14b486343ce6893cff73b80e153063917a with the original branch ref preserved as a comment.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Added double quotes around `$WAIT_TILL_READY_SECONDS` in the arithmetic comparison at line 58 of action.yml: changed `if [[ $SLEPT -gt $WAIT_TILL_READY_SECONDS ]]; then` to `if [[ $SLEPT -gt "$WAIT_TILL_READY_SECONDS" ]]; then`. This ensures the workflow-controllable input value is properly quoted in the shell script.


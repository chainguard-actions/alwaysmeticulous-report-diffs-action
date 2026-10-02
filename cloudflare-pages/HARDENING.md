<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.78.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.78.0** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are directly interpolated inside run: shell command strings, violating rule (a). Step 1 (line 33): `run: sleep ${{ inputs.sleep-time }}` — inputs.sleep-time is injected directly into the shell command. Step 2 (lines 43–55): inputs.cloudflare-account-id, inputs.cloudflare-project-name, inputs.cloudflare-api-token, inputs.wait-till-ready-seconds, inputs.deployment-poll-seconds, github.event_name, github.event.pull_request.head.sha, and github.sha are all interpolated directly into shell commands (curl URL/headers, jq filter strings, arithmetic, sleep arguments). An attacker-controlled input value could inject arbitrary shell commands.

Locations:

- `action.yml:33`
- `action.yml:43`
- `action.yml:44`
- `action.yml:47`
- `action.yml:50`
- `action.yml:51`
- `action.yml:54`
- `action.yml:55`

### github-env-injection (severity: high)

Lines 54–55 write values to $GITHUB_OUTPUT using backtick command substitution that embeds ${{ github.event_name }}, ${{ github.event.pull_request.head.sha }}, and ${{ github.sha }} directly inside jq filter strings. These untrusted github context values are interpolated into the shell command before execution and the resulting output is written to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A malicious commit SHA or event name containing newlines could inject additional key=value pairs into $GITHUB_OUTPUT.

Locations:

- `action.yml:54`
- `action.yml:55`

### unpinned-uses (severity: high)

The step 'Create GitHub deployment from Cloudflare Pages deployment' references `altinukshini/deployment-action@releases/v1`, which uses a mutable branch ref (`releases/v1`) rather than a pinned 40-character commit SHA. This exposes the action to supply-chain attacks if the upstream repository is compromised or the branch is force-pushed.

Locations:

- `action.yml:57`

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

Rewrote action.yml to fix all findings:
1. Moved all ${{ }} expressions from run: shell strings into env: blocks for both steps. Shell scripts now reference plain env vars ($SLEEP_SECONDS, $CF_ACCOUNT_ID, $CF_PROJECT_NAME, $CF_API_TOKEN, $WAIT_TILL_READY_SECONDS, $DEPLOYMENT_POLL_SECONDS, $EVENT_NAME, $PR_HEAD_SHA, $GITHUB_SHA_INPUT).
2. Replaced the inline GitHub expression-based commit SHA selection in jq filter strings with shell logic that computes COMMIT_HASH from env vars, then passes it safely to jq via --arg.
3. Sanitized $GITHUB_OUTPUT writes: values are captured with tr -d '\n\r' and written with printf instead of backtick command substitution.
4. Pinned altinukshini/deployment-action@releases/v1 to full SHA fe2fba14b486343ce6893cff73b80e153063917a with a # releases/v1 comment.
5. Also corrected inputs.sleep-time (undefined) to inputs.sleep-seconds (the actual defined input name).


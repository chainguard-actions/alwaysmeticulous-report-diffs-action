<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.81.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.81.0** was hardened automatically. 11 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): the step 'Sleep a bit to wait for the deployment to exist' puts an input expression straight into run: `sleep ${{ inputs.sleep-time }}`. (The input inputs.sleep-time is not even declared; the declared one is sleep-seconds.)

Locations:

- `action.yml:34`

### script-injection (severity: high)

Sub-rule (a): the step 'Wait for deployment to be ready and get URL' puts many ${{ }} expressions straight into its run script, including `curl ... "https://api.cloudflare.com/client/v4/accounts/${{ inputs.cloudflare-account-id }}/pages/projects/${{ inputs.cloudflare-project-name }}/deployments" -H "Authorization: Bearer ${{ inputs.cloudflare-api-token }}"`, `${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}`, `if [[ $SLEPT -gt ${{ inputs.wait-till-ready-seconds }} ]]`, `sleep ${{ inputs.deployment-poll-seconds }}` and `SLEPT=$((SLEPT+${{ inputs.deployment-poll-seconds }}))`. These values should be passed through env: and double-quoted.

Locations:

- `action.yml:44`
- `action.yml:45`
- `action.yml:47`
- `action.yml:52`
- `action.yml:53`
- `action.yml:57`
- `action.yml:58`

### github-env-injection (severity: high)

The step 'Wait for deployment to be ready and get URL' writes `url=...` and `environment=...` to $GITHUB_OUTPUT. The values come from an API response that was fetched using inputs.* and filtered with an inline ${{ github.* }} expression. The write only uses `tr -d '\n'`, not the required `printf '%s' ... | tr -d '\n\r'` step, so carriage returns are not removed. The write lines also contain inline github.* expressions.

Locations:

- `action.yml:57`
- `action.yml:58`

### unpinned-uses (severity: high)

A composite step references an action by branch instead of a 40-character commit SHA: `uses: altinukshini/deployment-action@releases/v1`.

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

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unpinned-uses

**Notes:**

All changes are in action.yml. Nothing has been run or tested.

- **Sleep step:** `inputs.sleep-time` now goes through `env: SLEEP_TIME` and runs as `sleep "$SLEEP_TIME"`. I kept the name upstream used, even though the declared input is `sleep-seconds`.
- **Polling step:** the account ID, project name, API token, commit SHA expression, wait time and poll interval are all passed through `env:` and used double-quoted. The commit SHA now reaches jq with `--arg sha` instead of being pasted into the jq filter.
- **New number check:** the step now fails at the start if `wait-till-ready-seconds` or `deployment-poll-seconds` is not a whole number of zero or more. Bash runs maths on whatever text sits inside `[[ -gt ]]` and `$(( ))`, and that can run commands. This is stricter than upstream, which accepted any value.
- **Outputs:** the url and environment values are captured first with `|| true`, because upstream ran jq inside backticks in `echo`, so a jq failure never stopped the step. Each value then goes through `printf '%s' | tr -d '\n\r'` before being written to the quoted "$GITHUB_OUTPUT".
- **Pinning:** `altinukshini/deployment-action@releases/v1` now points to fe2fba14b486343ce6893cff73b80e153063917a, with a `# releases/v1` comment.

Problem in upstream, not fixed: `sleep-time` is not a declared input. If a caller doesn't pass it, the value is empty, the step's `if: != '0'` check passes, and `sleep` fails with an empty argument. That was already true before hardening (only the error message differs). The fix is probably to switch to `sleep-seconds`, but I left the behaviour as upstream had it.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Changed the 'Wait for deployment' step so it uses the env vars that were already defined but never read (CF_ACCOUNT_ID, CF_PROJECT_NAME, CF_API_TOKEN, WAIT_TILL_READY_SECONDS, DEPLOYMENT_POLL_SECONDS). Every reference is quoted. COMMIT_SHA now reaches jq through --arg. Bash can run commands hidden in a value used in arithmetic, so the step now checks that both timing inputs are whole numbers before using them. A value that isn't a number fails the step right away. The url= and environment= values are each saved to a variable first, stripped of \n and \r, and written to a quoted "$GITHUB_OUTPUT". Upstream ran those jq calls inside echo, which hid a jq failure; the new captures end in '|| true' to keep that behaviour. The other steps were left as they were. Not tested: I made these edits by reading the code only.


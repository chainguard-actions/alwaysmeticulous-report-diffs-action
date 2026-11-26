<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.82.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.82.0** was hardened automatically. 11 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the step 'Sleep a bit to wait for the deployment to exist' puts an expression straight into the run script: `run: sleep ${{ inputs.sleep-time }}`. Pass the value in through env: and expand it in double quotes.

Locations:

- `action.yml:35`

### script-injection (severity: high)

Rule (a): the step 'Wait for deployment to be ready and get URL' puts many expressions straight into the run script. Examples: `curl ... "https://api.cloudflare.com/client/v4/accounts/${{ inputs.cloudflare-account-id }}/pages/projects/${{ inputs.cloudflare-project-name }}/deployments" -H "Authorization: Bearer ${{ inputs.cloudflare-api-token }}"`, `if [[ $SLEPT -gt ${{ inputs.wait-till-ready-seconds }} ]]`, `sleep ${{ inputs.deployment-poll-seconds }}`, `SLEPT=$((SLEPT+${{ inputs.deployment-poll-seconds }}))`, and `${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}` inside the jq filters. Move each one into env: variables and expand them in double quotes.

Locations:

- `action.yml:45`
- `action.yml:46`
- `action.yml:48`
- `action.yml:53`
- `action.yml:54`
- `action.yml:58`
- `action.yml:59`

### github-env-injection (severity: high)

The step 'Wait for deployment to be ready and get URL' writes 'url=' and 'environment=' to $GITHUB_OUTPUT (unquoted) from the Cloudflare API response. That response is fetched with workflow-controlled inputs (cloudflare-account-id, cloudflare-project-name) and filtered with github.* expressions. The values pass only through `tr -d '\n'`, not the required `printf '%s' ... | tr -d '\n\r'` step, and the write happens inside backtick command substitution.

Locations:

- `action.yml:58`
- `action.yml:59`

### unpinned-uses (severity: high)

A composite step references a branch instead of a 40-character commit SHA: `uses: altinukshini/deployment-action@releases/v1`.

Locations:

- `action.yml:61`

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

I fixed all four finding types, in action.yml only. Nothing was run, so this hasn't been tested.

- **Script injection:** every `${{ }}` expression is out of the two `run:` scripts and into each step's `env:` block. That covers the sleep time, the Cloudflare account ID, project name and API token, the wait limit, the poll interval and the commit SHA. The scripts now use them as double-quoted variables.
- **Commit SHA in the jq filters:** the SHA is now passed with `jq --arg sha "$COMMIT_SHA"` instead of being pasted into the filter text.
- **Outputs:** the `url` and `environment` values are now captured into variables first (with `|| true`, because upstream's echo never failed the step when jq failed). Newlines and carriage returns are then stripped with `printf '%s' | tr -d '\n\r'`, and the values are written to a quoted `"$GITHUB_OUTPUT"`. The backtick substitution is gone.
- **Unpinned action:** `altinukshini/deployment-action@releases/v1` is now pinned to `fe2fba14b486343ce6893cff73b80e153063917a # releases/v1`. I got the SHA from the lookup tool.

One upstream bug I left as it was: the sleep step reads `inputs.sleep-time`, but the input is actually called `sleep-seconds`. So the sleep value always comes through empty and the `sleep-seconds` setting is ignored. An empty value probably makes `sleep` fail with a missing-operand error, which would fail that step. If so, upstream has the same problem.


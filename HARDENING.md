<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action/v1.81.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action/v1.81.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The Docker action image uses a mutable tag instead of a SHA digest: `image: "docker://ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1"`.

Locations:

- `action.yml:67`

### unpinned-uses (severity: high)

A composite step uses a branch/tag ref instead of a 40-character commit SHA: `uses: altinukshini/deployment-action@releases/v1`.

Locations:

- `cloudflare-pages/action.yml:60`

### script-injection (severity: high)

Rule (a): the step 'Sleep a bit to wait for the deployment to exist' puts an expression straight into the shell command, unquoted: `run: sleep ${{ inputs.sleep-time }}`.

Locations:

- `cloudflare-pages/action.yml:34`

### script-injection (severity: high)

Rule (a): the step 'Wait for deployment to be ready and get URL' puts many expressions straight into the run script. Examples: `curl ... "https://api.cloudflare.com/client/v4/accounts/${{ inputs.cloudflare-account-id }}/pages/projects/${{ inputs.cloudflare-project-name }}/deployments" -H "Authorization: Bearer ${{ inputs.cloudflare-api-token }}"`; `${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}` inside jq filters; `if [[ $SLEPT -gt ${{ inputs.wait-till-ready-seconds }} ]]`; `sleep ${{ inputs.deployment-poll-seconds }}`; `SLEPT=$((SLEPT+${{ inputs.deployment-poll-seconds }}))`. These values should be passed through env: variables and expanded in double quotes.

Locations:

- `cloudflare-pages/action.yml:38`
- `cloudflare-pages/action.yml:44`
- `cloudflare-pages/action.yml:45`
- `cloudflare-pages/action.yml:47`
- `cloudflare-pages/action.yml:52`
- `cloudflare-pages/action.yml:53`
- `cloudflare-pages/action.yml:57`
- `cloudflare-pages/action.yml:58`

### github-env-injection (severity: high)

The step 'Wait for deployment to be ready and get URL' writes `url=...` and `environment=...` to $GITHUB_OUTPUT. Both values are built from a curl response (the URL comes from inputs.cloudflare-account-id and inputs.cloudflare-project-name) and filtered by jq using interpolated github.* expressions. The required sanitization `printf '%s' ... | tr -d '\n\r'` is not applied: only `tr -d '\n'` is used, inside backticks, and $GITHUB_OUTPUT is unquoted.

Locations:

- `cloudflare-pages/action.yml:57`
- `cloudflare-pages/action.yml:58`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

action.yml: pinned the docker:// image to releases-v1@sha256:e76bc17d.... cloudflare-pages/action.yml: pinned altinukshini/deployment-action to fe2fba14b486343ce6893cff73b80e153063917a (# releases/v1). Moved every ${{ }} expression in the run steps into env:. The commit SHA now reaches jq with --arg instead of being pasted into the filter. The poll and wait inputs are checked to be integers before they are used in arithmetic. The url and environment outputs now pass through printf | tr -d '\n\r' and are written to a quoted "$GITHUB_OUTPUT". The upstream reference to the undefined inputs.sleep-time is unchanged on purpose.


<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action/v1.82.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action/v1.82.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

cloudflare-pages/action.yml step 'Sleep a bit to wait for the deployment to exist' breaks rule (a): an expression is placed straight into run: `run: sleep ${{ inputs.sleep-time }}`. Fix: pass it through env: and use "$SLEEP_TIME" in quotes.

Locations:

- `cloudflare-pages/action.yml:34`

### script-injection (severity: high)

cloudflare-pages/action.yml step 'Wait for deployment to be ready and get URL' breaks rule (a): it puts many expressions straight into the run: script. Examples: `curl -s -X GET "https://api.cloudflare.com/client/v4/accounts/${{ inputs.cloudflare-account-id }}/pages/projects/${{ inputs.cloudflare-project-name }}/deployments" -H "Authorization: Bearer ${{ inputs.cloudflare-api-token }}"`, `if [[ $SLEPT -gt ${{ inputs.wait-till-ready-seconds }} ]]`, `sleep ${{ inputs.deployment-poll-seconds }}`, `SLEPT=$((SLEPT+${{ inputs.deployment-poll-seconds }}))`, plus `${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}` inside the jq filters. Fix: move all of them into env: variables and use them in double quotes.

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

cloudflare-pages/action.yml step 'Wait for deployment to be ready and get URL' writes `url=...` and `environment=...` to $GITHUB_OUTPUT (unquoted). The values come from a Cloudflare API lookup that is built from inputs.* and filtered with a github.* expression placed in the script. They only pass through `tr -d '\n'`, not the required `printf '%s' ... | tr -d '\n\r'`, so a carriage return is not removed. Fix: put each value in a variable, clean it with printf '%s' "$VAL" | tr -d '\n\r', and write to "$GITHUB_OUTPUT".

Locations:

- `cloudflare-pages/action.yml:57`
- `cloudflare-pages/action.yml:58`

### unpinned-uses (severity: high)

cloudflare-pages/action.yml uses an action by mutable branch ref instead of a 40-character commit SHA: `uses: altinukshini/deployment-action@releases/v1`.

Locations:

- `cloudflare-pages/action.yml:60`

### unpinned-uses (severity: high)

The root action.yml is a docker action whose image uses a mutable tag instead of a sha256 digest: `image: "docker://ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1"`.

Locations:

- `action.yml:72`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

I fixed all five findings in action.yml and cloudflare-pages/action.yml.

- **Docker image:** pinned the root action.yml image to `docker://ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1@sha256:4b1043b5...`, keeping the docker:// prefix and the tag. That digest is whatever `releases-v1` points to today. It may not be exactly the v1.82.0 build, because there is no `v1.82.0` image tag (the lookup returned 404).
- **Pinned action:** pinned `altinukshini/deployment-action` to commit `fe2fba14b486343ce6893cff73b80e153063917a`, with `# releases/v1` kept as a comment.
- **Script injection:** in cloudflare-pages/action.yml, every `${{ }}` expression in the two run steps now goes through env: (`SLEEP_TIME`, `CF_ACCOUNT_ID`, `CF_PROJECT_NAME`, `CF_API_TOKEN`, `COMMIT_SHA`, `WAIT_TILL_READY_SECONDS`, `DEPLOYMENT_POLL_SECONDS`) and is used in double quotes. The commit SHA reaches the jq filters through `--arg sha` instead of being pasted into the filter text.
- **Output injection:** the url and environment values are now saved to variables first, cleaned with `printf '%s' | tr -d '\n\r'`, and written to the quoted `"$GITHUB_OUTPUT"`. I added `|| true` to those two lookups because upstream ran them inside echo, so a jq failure didn't stop the step there either.

Upstream bug I left alone: the sleep step reads `inputs.sleep-time`, but the input is actually named `sleep-seconds`. The value is always empty, so the sleep step will probably fail just as it does upstream. I didn't rename it because that would change behaviour. A one-word fix if you want it.


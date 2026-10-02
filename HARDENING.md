<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action/v1.77.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action/v1.77.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ ... }} expressions are directly interpolated inside run: shell commands in cloudflare-pages/action.yml, violating rule (a). Offending lines include:
- `run: sleep ${{ inputs.sleep-time }}` (line 34)
- `LAST_RESULT=$(curl ... "${{ inputs.cloudflare-account-id }}" ... "${{ inputs.cloudflare-project-name }}" ... "${{ inputs.cloudflare-api-token }}" ...)` (line 44)
- `STATUS=$(... "${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}")` (line 45)
- `if [[ $SLEPT -gt ${{ inputs.wait-till-ready-seconds }} ]]` (line 47)
- `sleep ${{ inputs.deployment-poll-seconds }}` (line 52)
- `SLEPT=$((SLEPT+${{ inputs.deployment-poll-seconds }}))` (line 53)
All of these allow an attacker-controlled value to be injected directly into the shell command string before the shell ever parses it.

Locations:

- `cloudflare-pages/action.yml:34`
- `cloudflare-pages/action.yml:44`
- `cloudflare-pages/action.yml:45`
- `cloudflare-pages/action.yml:47`
- `cloudflare-pages/action.yml:52`
- `cloudflare-pages/action.yml:53`

### github-env-injection (severity: high)

In cloudflare-pages/action.yml, the run: block writes values derived from ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }} directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Specifically:
- `echo "url=`...jq ... '${{ github.event_name == ... }}'...`" >> $GITHUB_OUTPUT`
- `echo "environment=`...jq ... '${{ github.event_name == ... }}'...`" >> $GITHUB_OUTPUT`
A newline injected into the github context value could allow an attacker to inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `cloudflare-pages/action.yml:57`
- `cloudflare-pages/action.yml:58`

### unpinned-uses (severity: high)

The following references use mutable tags or branch names instead of immutable full 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks:
1. cloudflare-pages/action.yml: `uses: altinukshini/deployment-action@releases/v1` — uses a branch/tag ref `releases/v1`, not a SHA.
2. action.yml: `image: "docker://ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1"` — uses a mutable tag `releases-v1` instead of a SHA digest (e.g. `@sha256:<64-hex-char-digest>`).

Locations:

- `cloudflare-pages/action.yml:60`
- `action.yml:67`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all three findings:

1. **script-injection** (cloudflare-pages/action.yml): Moved all ${{ }} expressions from `run:` shell commands into `env:` blocks. Shell script now uses plain env vars ($SLEEP_TIME, $CF_ACCOUNT_ID, $CF_PROJECT_NAME, $CF_API_TOKEN, $COMMIT_SHA, $WAIT_TILL_READY_SECONDS, $DEPLOYMENT_POLL_SECONDS). The jq queries now use `--arg sha "$COMMIT_SHA"` to pass the commit SHA safely as a jq variable instead of string interpolation.

2. **github-env-injection** (cloudflare-pages/action.yml): Values written to $GITHUB_OUTPUT are now captured into local variables with `tr -d '\n\r'` sanitization applied before writing (safe_url, safe_env).

3. **unpinned-uses** (two locations):
   - cloudflare-pages/action.yml: Pinned `altinukshini/deployment-action` to full SHA `fe2fba14b486343ce6893cff73b80e153063917a` with `# releases/v1` comment.
   - action.yml: Pinned docker image to `docker://ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1@sha256:6e9fbfd74c94bf1190020ab88b08de77a10a9eff982eaa05459228d69792c4cf` (preserving the `docker://` scheme and tag).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansion in cloudflare-pages/action.yml line 56: changed `if [[ $SLEPT -gt $WAIT_TILL_READY_SECONDS ]]` to `if [[ $SLEPT -gt "$WAIT_TILL_READY_SECONDS" ]]`. The variable was already correctly sourced via the step's env: block; only the quoting inside the conditional was missing.


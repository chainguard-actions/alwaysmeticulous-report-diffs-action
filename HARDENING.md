<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action/v1.78.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action/v1.78.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

cloudflare-pages/action.yml contains multiple ${{ }} expressions directly interpolated inside run: shell blocks (sub-rule a). This allows an attacker to inject arbitrary shell commands via controlled inputs or GitHub context values.

Violating lines include:
- `run: sleep ${{ inputs.sleep-time }}` — direct interpolation of an input value into a shell command
- `LAST_RESULT=$(curl ... "https://api.cloudflare.com/.../accounts/${{ inputs.cloudflare-account-id }}/pages/projects/${{ inputs.cloudflare-project-name }}/..." -H "Authorization: Bearer ${{ inputs.cloudflare-api-token }}" ...)` — three inputs interpolated directly into a curl command
- `STATUS=$(echo "$LAST_RESULT" | jq -c '... == "${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}"...')` — github context interpolated into a jq shell command
- `if [[ $SLEPT -gt ${{ inputs.wait-till-ready-seconds }} ]]` — input interpolated into shell arithmetic
- `sleep ${{ inputs.deployment-poll-seconds }}` and `SLEPT=$((SLEPT+${{ inputs.deployment-poll-seconds }}))` — input interpolated into shell arithmetic

All of these should be moved to env: variables and referenced as quoted shell variables (e.g., "$VAR").

Locations:

- `cloudflare-pages/action.yml:33`
- `cloudflare-pages/action.yml:43`
- `cloudflare-pages/action.yml:44`
- `cloudflare-pages/action.yml:47`
- `cloudflare-pages/action.yml:50`
- `cloudflare-pages/action.yml:51`

### unpinned-uses (severity: high)

Two unpinned action/image references found:

1. action.yml uses `image: "docker://ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1"` — this is a mutable tag reference, not a SHA digest. A supply-chain attacker could push a malicious image to this tag. It should be pinned to a SHA digest, e.g. `docker://ghcr.io/alwaysmeticulous/report-diffs-action@sha256:<64-hex-char-digest>`.

2. cloudflare-pages/action.yml uses `altinukshini/deployment-action@releases/v1` — this is a mutable branch/tag reference, not a 40-character commit SHA. It should be pinned to a full commit SHA, e.g. `altinukshini/deployment-action@<40-hex-char-sha> # releases/v1`.

Locations:

- `action.yml:72`
- `cloudflare-pages/action.yml:68`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed three issues across two files:

1. action.yml: Pinned docker image `ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1` to SHA digest `sha256:6e9fbfd74c94bf1190020ab88b08de77a10a9eff982eaa05459228d69792c4cf`, keeping the tag inline.

2. cloudflare-pages/action.yml (unpinned-uses): Pinned `altinukshini/deployment-action@releases/v1` to commit SHA `fe2fba14b486343ce6893cff73b80e153063917a` with the original ref as a comment.

3. cloudflare-pages/action.yml (script-injection): Moved all ${{ }} expressions from run: shell blocks into env: blocks. The inputs for cloudflare-account-id, cloudflare-project-name, cloudflare-api-token, sleep-time, wait-till-ready-seconds, and deployment-poll-seconds are now referenced as shell env vars. The GitHub SHA expression is also moved to an env var (COMMIT_SHA), and the jq queries were updated to use `--arg sha "$COMMIT_SHA"` for safe variable passing instead of string interpolation.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in cloudflare-pages/action.yml in the 'Wait for deployment to be ready and get URL' step:

1. Added integer validation at the start of the run script using `printf '%d'` to coerce `$WAIT_TILL_READY_SECONDS` and `$DEPLOYMENT_POLL_SECONDS` into validated integer variables (`WAIT_TILL_READY_SECONDS_INT` and `DEPLOYMENT_POLL_SECONDS_INT`). If either value is not a valid integer, the script exits with an error.

2. Replaced all uses of the unvalidated variables with the validated integer versions:
   - `[[ $SLEPT -gt $WAIT_TILL_READY_SECONDS ]]` → `[[ "$SLEPT" -gt "$WAIT_TILL_READY_SECONDS_INT" ]]`
   - `$((SLEPT+DEPLOYMENT_POLL_SECONDS))` → `$(( SLEPT + DEPLOYMENT_POLL_SECONDS_INT ))`
   - `sleep "$DEPLOYMENT_POLL_SECONDS"` → `sleep "$DEPLOYMENT_POLL_SECONDS_INT"`

This prevents bash arithmetic expansion from executing arbitrary commands via array subscript expressions like `a[$(malicious_cmd)]`, since `printf '%d'` rejects any non-integer value before it reaches the arithmetic context.


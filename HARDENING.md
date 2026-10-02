<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action/v1.79.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action/v1.79.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple ${{ ... }} expressions are directly interpolated inside run: shell command strings in cloudflare-pages/action.yml. This allows an attacker to inject arbitrary shell commands via inputs or github context values before the shell ever sees them.

Line 34: `run: sleep ${{ inputs.sleep-time }}` — inputs.sleep-time interpolated directly into shell.
Line 44: `LAST_RESULT=$(curl ... ${{ inputs.cloudflare-account-id }} ... ${{ inputs.cloudflare-project-name }} ... ${{ inputs.cloudflare-api-token }} ...)` — three inputs interpolated directly into a curl command.
Line 45: `STATUS=$(echo "$LAST_RESULT" | jq ... "${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}" ...)` — github context interpolated directly.
Line 47: `if [[ $SLEPT -gt ${{ inputs.wait-till-ready-seconds }} ]]` — input interpolated into arithmetic comparison.
Line 52: `sleep ${{ inputs.deployment-poll-seconds }}` — input interpolated directly.
Line 53: `SLEPT=$((SLEPT+${{ inputs.deployment-poll-seconds }}))` — input interpolated into arithmetic.

Locations:

- `cloudflare-pages/action.yml:34`
- `cloudflare-pages/action.yml:44`
- `cloudflare-pages/action.yml:45`
- `cloudflare-pages/action.yml:47`
- `cloudflare-pages/action.yml:52`
- `cloudflare-pages/action.yml:53`

### github-env-injection (severity: high)

Untrusted github context values (github.event.pull_request.head.sha and github.sha) are written to $GITHUB_OUTPUT without sanitization. The values are interpolated directly into the jq command string via ${{ ... }} expressions and the output is piped through `tr -d '\n'` (which only strips newlines from the jq output, not from the injected expression itself). The ${{ ... }} substitution happens before the shell runs, so a malicious value could inject newlines or key=value pairs into GITHUB_OUTPUT before tr ever runs.

Line 57: `echo "url=`...jq ... '${{ github.event.pull_request.head.sha || github.sha }}' ...`" >> $GITHUB_OUTPUT`
Line 58: `echo "environment=`...jq ... '${{ github.event.pull_request.head.sha || github.sha }}' ...`" >> $GITHUB_OUTPUT`

Locations:

- `cloudflare-pages/action.yml:57`
- `cloudflare-pages/action.yml:58`

### unpinned-uses (severity: high)

Two unpinned action/image references found:

1. action.yml uses `runs.image: "docker://ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1"` — a mutable tag (`releases-v1`), not a SHA digest. A supply-chain attacker could push a malicious image to this tag.

2. cloudflare-pages/action.yml uses `altinukshini/deployment-action@releases/v1` — a branch/tag ref, not a 40-character commit SHA. This is vulnerable to tag mutation attacks.

Locations:

- `action.yml:76`
- `cloudflare-pages/action.yml:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all three findings:

1. **unpinned-uses (action.yml line 76)**: Pinned `docker://ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1` to `docker://ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1@sha256:6e9fbfd74c94bf1190020ab88b08de77a10a9eff982eaa05459228d69792c4cf`.

2. **unpinned-uses (cloudflare-pages/action.yml line 60)**: Pinned `altinukshini/deployment-action@releases/v1` to `altinukshini/deployment-action@fe2fba14b486343ce6893cff73b80e153063917a # releases/v1`.

3. **script-injection (cloudflare-pages/action.yml lines 34,44,45,47,52,53)**: Moved all `${{ inputs.* }}` and `${{ github.* }}` expressions into `env:` blocks (`CF_ACCOUNT_ID`, `CF_PROJECT_NAME`, `CF_API_TOKEN`, `COMMIT_SHA`, `WAIT_TILL_READY_SECONDS`, `DEPLOYMENT_POLL_SECONDS`, `SLEEP_TIME`). The jq queries now use `--arg sha "$COMMIT_SHA"` to pass the SHA as a jq variable instead of interpolating it into the filter string.

4. **github-env-injection (cloudflare-pages/action.yml lines 57,58)**: The COMMIT_SHA is now an env var (not interpolated into the shell). The jq output values are captured into shell variables and sanitized with `tr -d '\n\r'` before being written to `$GITHUB_OUTPUT` using the `echo "key=value" >> "$GITHUB_OUTPUT"` pattern.


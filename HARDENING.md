<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action/v1.80.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action/v1.80.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

cloudflare-pages/action.yml has multiple ${{ }} expressions directly interpolated inside run: shell command strings, violating sub-rule (a). This allows script injection via attacker-controlled inputs or github context values.

Line 34: `run: sleep ${{ inputs.sleep-time }}` — inputs.sleep-time interpolated directly into shell.
Line 44: `LAST_RESULT=$(curl ... ${{ inputs.cloudflare-account-id }} ... ${{ inputs.cloudflare-project-name }} ... ${{ inputs.cloudflare-api-token }} ...)` — three input expressions interpolated into a curl command.
Line 45: `STATUS=$(... ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }} ...)` — github context expression interpolated into shell.
Line 47: `if [[ $SLEPT -gt ${{ inputs.wait-till-ready-seconds }} ]]` — input expression interpolated into shell arithmetic.
Line 52: `sleep ${{ inputs.deployment-poll-seconds }}` — input expression interpolated into shell.
Line 53: `SLEPT=$((SLEPT+${{ inputs.deployment-poll-seconds }}))` — input expression interpolated into shell arithmetic.

All of these should be moved to env: variables and then referenced as double-quoted shell variables (e.g., "$VAR").

Locations:

- `cloudflare-pages/action.yml:34`
- `cloudflare-pages/action.yml:44`
- `cloudflare-pages/action.yml:45`
- `cloudflare-pages/action.yml:47`
- `cloudflare-pages/action.yml:52`
- `cloudflare-pages/action.yml:53`

### github-env-injection (severity: high)

cloudflare-pages/action.yml writes values derived from github.event.pull_request.head.sha and github.sha (via direct ${{ }} interpolation) to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker could inject newlines into the output to poison subsequent steps.

Line 57: `echo "url=`...jq ... ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }} ...`" >> $GITHUB_OUTPUT`
Line 58: `echo "environment=`...jq ... ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }} ...`" >> $GITHUB_OUTPUT`

Locations:

- `cloudflare-pages/action.yml:57`
- `cloudflare-pages/action.yml:58`

### unpinned-uses (severity: high)

Two unpinned action/image references were found:

1. cloudflare-pages/action.yml line 60: `uses: altinukshini/deployment-action@releases/v1` — uses a branch name (`releases/v1`) instead of a full 40-character commit SHA. This is vulnerable to supply-chain attacks if the branch is force-pushed.

2. action.yml (runs.image): `image: "docker://ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1"` — uses a mutable image tag (`releases-v1`) instead of a SHA digest (e.g., `@sha256:<64-hex-char-digest>`). A tag can be silently redirected to a different image.

Locations:

- `cloudflare-pages/action.yml:60`
- `action.yml:72`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed cloudflare-pages/action.yml: (1) Moved all ${{ inputs.* }} and ${{ github.* }} expressions from run: shell strings into env: blocks, referencing them as $VAR_NAME shell variables to prevent script injection. (2) Sanitized values written to $GITHUB_OUTPUT using printf '%s' ... | tr -d '\n\r' to prevent newline injection. (3) Pinned altinukshini/deployment-action@releases/v1 to full SHA @fe2fba14b486343ce6893cff73b80e153063917a. Fixed action.yml: pinned docker://ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1 to @sha256:6e9fbfd74c94bf1190020ab88b08de77a10a9eff982eaa05459228d69792c4cf while preserving the docker:// scheme and tag.


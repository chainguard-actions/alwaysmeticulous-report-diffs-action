<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action/v1.76.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action/v1.76.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: Multiple ${{ }} expressions are directly interpolated inside run: shell command strings in cloudflare-pages/action.yml. This allows an attacker who controls the calling workflow's inputs or the GitHub event context to inject arbitrary shell commands. Offending lines include:
- `run: sleep ${{ inputs.sleep-time }}` (line 34)
- `LAST_RESULT=$(curl ... "${{ inputs.cloudflare-account-id }}" ... "${{ inputs.cloudflare-project-name }}" ... "Bearer ${{ inputs.cloudflare-api-token }}" ...)` (line 44)
- `STATUS=$(echo ... | jq -c '... == "${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}" ...')` (line 45)
- `if [[ $SLEPT -gt ${{ inputs.wait-till-ready-seconds }} ]]` (line 48)
- `sleep ${{ inputs.deployment-poll-seconds }}` (line 51)
- `SLEPT=$((SLEPT+${{ inputs.deployment-poll-seconds }}))` (line 52)
- Two GITHUB_OUTPUT writes with `${{ github.event.pull_request.head.sha || github.sha }}` interpolated into jq filter strings (lines 56-57)
All inputs.* and github.* values must be passed via env: variables and then double-quoted in the shell script.

Locations:

- `cloudflare-pages/action.yml:34`
- `cloudflare-pages/action.yml:44`
- `cloudflare-pages/action.yml:45`
- `cloudflare-pages/action.yml:48`
- `cloudflare-pages/action.yml:51`
- `cloudflare-pages/action.yml:52`
- `cloudflare-pages/action.yml:56`
- `cloudflare-pages/action.yml:57`

### github-env-injection (severity: high)

The run: block in cloudflare-pages/action.yml writes values derived from github context expressions (${{ github.event.pull_request.head.sha || github.sha }}) to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Specifically, lines 56-57 write `url=` and `environment=` values to $GITHUB_OUTPUT using backtick command substitution that embeds the github SHA expression directly into a jq filter string. Although tr -d '\n' is applied to the jq output, the ${{ }} expression is interpolated into the shell command before execution, meaning a newline-containing value could still inject additional GITHUB_OUTPUT key-value pairs. The sanitization must be applied to the value being written, not just the jq output.

Locations:

- `cloudflare-pages/action.yml:56`
- `cloudflare-pages/action.yml:57`

### unpinned-uses (severity: high)

Two unpinned action/image references were found:
1. action.yml uses `image: "docker://ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1"` — a mutable tag instead of a SHA digest (e.g., `@sha256:<64-hex-char-digest>`). A supply-chain attacker could push a malicious image to this tag.
2. cloudflare-pages/action.yml uses `uses: altinukshini/deployment-action@releases/v1` — a branch/tag ref instead of a full 40-character commit SHA. This is vulnerable to tag mutation attacks.

Locations:

- `action.yml:63`
- `cloudflare-pages/action.yml:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

1. action.yml: Pinned docker image 'ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1' to '@sha256:6e9fbfd74c94bf1190020ab88b08de77a10a9eff982eaa05459228d69792c4cf'. 2. cloudflare-pages/action.yml: All ${{ inputs.* }} and ${{ github.* }} expressions moved from run: shell strings into env: blocks (CF_ACCOUNT_ID, CF_PROJECT_NAME, CF_API_TOKEN, WAIT_TILL_READY_SECONDS, DEPLOYMENT_POLL_SECONDS, COMMIT_SHA, SLEEP_TIME). The COMMIT_SHA is passed to jq via --arg to avoid shell injection in jq filter strings. 3. cloudflare-pages/action.yml: GITHUB_OUTPUT writes now sanitize values with printf '%s' | tr -d '\n\r' before writing. 4. cloudflare-pages/action.yml: Pinned 'altinukshini/deployment-action@releases/v1' to full SHA 'fe2fba14b486343ce6893cff73b80e153063917a' with '# releases/v1' comment.


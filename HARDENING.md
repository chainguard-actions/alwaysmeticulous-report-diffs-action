<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action/v1.72.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action/v1.72.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

cloudflare-pages/action.yml directly interpolates GitHub Actions expressions into run: shell commands, violating rule (a). Multiple inputs and github context values are embedded verbatim in shell: (1) Line 34: `run: sleep ${{ inputs.sleep-time }}` — attacker-controlled input injected directly into a shell command. (2) Lines 41–55 (the 'Wait for deployment' run block): `${{ inputs.cloudflare-account-id }}`, `${{ inputs.cloudflare-project-name }}`, `${{ inputs.cloudflare-api-token }}`, `${{ inputs.wait-till-ready-seconds }}`, `${{ inputs.deployment-poll-seconds }}`, and `${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}` are all interpolated directly into shell command strings. Any of these values could contain shell metacharacters enabling command injection.

Locations:

- `cloudflare-pages/action.yml:34`
- `cloudflare-pages/action.yml:41`

### github-env-injection (severity: high)

cloudflare-pages/action.yml writes values derived from github context expressions directly to $GITHUB_OUTPUT without sanitization. Specifically, the 'url' and 'environment' outputs are written using backtick command substitution that embeds `${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}` in a jq filter string piped to $GITHUB_OUTPUT. Although `tr -d '\n'` is applied to the jq output, the github context expression is interpolated into the shell command itself before execution, and no `printf '%s' ... | tr -d '\n\r'` sanitization is applied to the written value before the >> $GITHUB_OUTPUT write.

Locations:

- `cloudflare-pages/action.yml:53`
- `cloudflare-pages/action.yml:54`

### unpinned-uses (severity: high)

cloudflare-pages/action.yml references `altinukshini/deployment-action@releases/v1` — a mutable branch/tag ref rather than a pinned 40-character commit SHA. This allows the upstream repository to silently change the code that runs in this action, enabling a supply-chain attack.

Locations:

- `cloudflare-pages/action.yml:57`

### unpinned-uses (severity: high)

action.yml uses a Docker image with a mutable tag: `image: "docker://ghcr.io/alwaysmeticulous/report-diffs-action:releases-v1"`. The tag `releases-v1` is not immutable; the image it points to can be changed at any time. The image reference should use a SHA digest (e.g. `@sha256:<64-hex-char-digest>`) to ensure reproducibility and prevent supply-chain attacks.

Locations:

- `action.yml:75`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all four findings:
1. Pinned Docker image in action.yml to sha256:6e9fbfd74c94bf1190020ab88b08de77a10a9eff982eaa05459228d69792c4cf (keeping releases-v1 tag inline).
2. Fixed script-injection in cloudflare-pages/action.yml by moving all ${{ inputs.* }} and ${{ github.* }} expressions into env: blocks (CF_ACCOUNT_ID, CF_PROJECT_NAME, CF_API_TOKEN, WAIT_TILL_READY_SECONDS, DEPLOYMENT_POLL_SECONDS, COMMIT_SHA, SLEEP_TIME). The jq filter now uses --arg sha "$COMMIT_SHA" to pass the SHA safely.
3. Fixed github-env-injection by capturing raw jq output into variables, then sanitizing with printf '%s' | tr -d '\n\r' before writing to $GITHUB_OUTPUT.
4. Pinned altinukshini/deployment-action@releases/v1 to full SHA fe2fba14b486343ce6893cff73b80e153063917a with # releases/v1 comment.


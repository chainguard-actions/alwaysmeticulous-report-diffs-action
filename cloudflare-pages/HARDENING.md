<!-- markdownlint-disable -->

# Hardening Report: alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.76.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **alwaysmeticulous--report-diffs-action--cloudflare-pages/v1.76.0** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ ... }} expressions are directly interpolated inside run: shell command strings (sub-rule a). This allows an attacker to inject arbitrary shell commands via controlled inputs or github context values. Affected expressions include: `${{ inputs.sleep-time }}` (line 35), `${{ inputs.cloudflare-account-id }}`, `${{ inputs.cloudflare-project-name }}`, `${{ inputs.cloudflare-api-token }}` embedded in a curl command (line 43), `${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}` embedded in jq arguments (lines 44, 54, 55), `${{ inputs.wait-till-ready-seconds }}` in an arithmetic comparison (line 46), and `${{ inputs.deployment-poll-seconds }}` in sleep/arithmetic (lines 50–51). All of these must be moved to env: variables and then double-quoted in the shell script.

Locations:

- `action.yml:35`
- `action.yml:43`
- `action.yml:44`
- `action.yml:46`
- `action.yml:50`
- `action.yml:51`
- `action.yml:54`
- `action.yml:55`

### github-env-injection (severity: high)

Lines 54–55 write values derived from the github context expression `${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}` (interpolated into a jq argument and captured via backtick subshell) directly to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`) applied immediately before the write. The `tr -d '\n'` inside the backtick subshell only strips newlines from the jq output, but the outer `echo "url=..." >> $GITHUB_OUTPUT` still writes the unsanitized github context value. Each write must be preceded by sanitization of the value being written.

Locations:

- `action.yml:54`
- `action.yml:55`

### unpinned-uses (severity: high)

The step `uses: altinukshini/deployment-action@releases/v1` references a mutable branch name (`releases/v1`) rather than a full 40-character commit SHA. This means the action could be silently updated or compromised without the consuming workflow noticing. It should be pinned to a specific commit SHA, e.g. `altinukshini/deployment-action@<40-char-sha> # releases/v1`.

Locations:

- `action.yml:57`

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

Fixed all findings in action.yml:

1. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: blocks into env: maps. The inputs (sleep-seconds, cloudflare-account-id, cloudflare-project-name, cloudflare-api-token, wait-till-ready-seconds, deployment-poll-seconds) and the github context expression (github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha) are now all in env: variables. The jq filters were updated to use --arg commit_sha "$COMMIT_SHA" and $commit_sha in the filter instead of string interpolation.

2. **github-env-injection**: Added proper sanitization before writing to $GITHUB_OUTPUT. The COMMIT_SHA is sanitized with printf/tr before use in jq, and the url/environment values from jq are sanitized with printf '%s' ... | tr -d '\n\r' before being written to $GITHUB_OUTPUT. Also fixed $GITHUB_OUTPUT to be double-quoted.

3. **unpinned-uses**: Pinned altinukshini/deployment-action@releases/v1 to the full commit SHA fe2fba14b486343ce6893cff73b80e153063917a with the original branch name preserved as a comment.

Also fixed a bug in the original: the step referenced inputs.sleep-time but the input is named sleep-seconds; corrected to use sleep-seconds consistently.


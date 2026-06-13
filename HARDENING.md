<!-- markdownlint-disable -->

# Hardening Report: pypa--gh-action-pip-audit/v1.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **pypa--gh-action-pip-audit/v1.0.6** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` steps in action.yml directly interpolate `${{ }}` expressions inside shell command strings. In the 'Set up pip-audit' step, `${{ github.action_path }}` is interpolated directly into the `source` command. In the 'Run pip-audit' step, both `${{ github.action_path }}` and `${{ inputs.inputs }}` are interpolated directly into shell commands. The `inputs.inputs` value is attacker-controlled and is passed as a shell argument without being routed through a quoted `env:` variable, allowing shell metacharacter injection. All `${{ ... }}` expressions inside `run:` blocks are script-injection risks regardless of context. The fix is to move these values into `env:` variables and reference them as double-quoted shell variables (e.g., `"$GHA_ACTION_PATH"` and `"$GHA_PIP_AUDIT_INPUTS"`).

Locations:

- `action.yml:62`
- `action.yml:69`
- `action.yml:71`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.inputs }}" appears directly in run: block of step "Run pip-audit"; move to env: map

Locations:

- `action.yml:78`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection findings in action.yml:
1. 'Set up pip-audit' step: moved ${{ github.action_path }} to GHA_ACTION_PATH env var; run: block now uses source "${GHA_ACTION_PATH}/setup/setup.bash"
2. 'Run pip-audit' step: moved ${{ github.action_path }} to GHA_ACTION_PATH env var and ${{ inputs.inputs }} to GHA_PIP_AUDIT_INPUTS env var; run: block now uses source "${GHA_ACTION_PATH}/setup/venv.bash" and "${GHA_ACTION_PATH}/action.py" "$GHA_PIP_AUDIT_INPUTS". The inputs value is passed as a regular double-quoted argument (not ${VAR:+"$VAR"}) because action.py always accesses sys.argv[1] and handles empty strings correctly via .split().


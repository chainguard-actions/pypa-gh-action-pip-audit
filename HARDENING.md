<!-- markdownlint-disable -->

# Hardening Report: pypa--gh-action-pip-audit/v1.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **pypa--gh-action-pip-audit/v1.0.7** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Set up pip-audit' step in action.yml directly interpolates the GitHub Actions expression `${{ github.action_path }}` inside the `run:` shell command: `source "${{ github.action_path }}/setup/setup.bash"`. Any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk because the value is substituted into the shell command string before the shell parses it.

Locations:

- `action.yml:63`

### script-injection (severity: high)

Sub-rule (a): The 'Run pip-audit' step in action.yml directly interpolates two GitHub Actions expressions inside the `run:` shell command block: (1) `source "${{ github.action_path }}/setup/venv.bash"` and (2) `${{ github.action_path }}/action.py "${{ inputs.inputs }}"`. The `inputs.inputs` value is attacker-controlled (it is a user-supplied action input with `required: false`) and is passed directly as a shell argument without being routed through an `env:` variable. An attacker can inject arbitrary shell metacharacters via this input.

Locations:

- `action.yml:71`
- `action.yml:73`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.inputs }}" appears directly in run: block of step "Run pip-audit"; move to env: map

Locations:

- `action.yml:74`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all three script-injection findings in action.yml:
1. 'Set up pip-audit' step (line 63): Moved `${{ github.action_path }}` out of the run: block into an env: variable `ACTION_PATH`, then referenced it as `${ACTION_PATH}` in the shell script.
2. 'Run pip-audit' step (lines 71, 73, 74): Moved `${{ github.action_path }}` to `ACTION_PATH` env var and `${{ inputs.inputs }}` (attacker-controlled optional input) to `GHA_PIP_AUDIT_INPUTS` env var. Used `${GHA_PIP_AUDIT_INPUTS:+"$GHA_PIP_AUDIT_INPUTS"}` for the optional positional argument to avoid passing an empty string argument when the input is absent. All ${{ }} expressions now appear only in env: blocks, not in run: shell strings.


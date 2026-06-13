<!-- markdownlint-disable -->

# Hardening Report: pypa--gh-action-pip-audit/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **pypa--gh-action-pip-audit/v1.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): In the 'Set up pip-audit' step, the expression `${{ github.action_path }}` is interpolated directly inside the `run:` shell command string: `source "${{ github.action_path }}/setup/setup.bash"`. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because YAML template substitution occurs before the shell ever sees the value.

Locations:

- `action.yml:63`

### script-injection (severity: high)

Sub-rule (a): In the 'Run pip-audit' step, multiple `${{ ... }}` expressions are interpolated directly inside the `run:` shell command string. Most critically, `${{ inputs.inputs }}` — an attacker-controlled value — is passed directly as a shell argument: `python "${{ github.action_path }}/action.py" "${{ inputs.inputs }}"`  and `source "${{ github.action_path }}/setup/venv.bash"`. These expressions are substituted by the YAML template engine before the shell processes the command, enabling shell metacharacter injection via the `inputs.inputs` value.

Locations:

- `action.yml:70`
- `action.yml:72`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.inputs }}" appears directly in run: block of step "Run pip-audit"; move to env: map

Locations:

- `action.yml:74`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script-injection findings in action.yml:
1. 'Set up pip-audit' step: moved `${{ github.action_path }}` into env: as GHA_ACTION_PATH; run: block now uses `${GHA_ACTION_PATH}/setup/setup.bash`.
2. 'Run pip-audit' step: moved `${{ github.action_path }}` into env: as GHA_ACTION_PATH and `${{ inputs.inputs }}` into env: as GHA_PIP_AUDIT_INPUTS; run: block uses `${GHA_ACTION_PATH}` for paths and `${GHA_PIP_AUDIT_INPUTS:+"$GHA_PIP_AUDIT_INPUTS"}` for the optional inputs argument (drops the argument entirely when empty, preserving the action's default-to-current-path behavior).


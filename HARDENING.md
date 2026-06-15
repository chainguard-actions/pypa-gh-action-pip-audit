<!-- markdownlint-disable -->

# Hardening Report: pypa--gh-action-pip-audit/v1.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **pypa--gh-action-pip-audit/v1.0.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings in two steps of action.yml.

1. In the "Set up pip-audit" step (line 62): `source "${{ github.action_path }}/setup/setup.bash"` — `github.action_path` is injected directly into a shell `source` command.

2. In the "Run pip-audit" step (lines 70 and 72): `source "${{ github.action_path }}/setup/venv.bash"` and `${{ github.action_path }}/action.py "${{ inputs.inputs }}"` — both `github.action_path` and the attacker-controlled `inputs.inputs` are interpolated directly into shell commands. An attacker who controls `inputs.inputs` can inject arbitrary shell metacharacters (e.g. `; malicious-command`) that will be executed by the runner shell before Python ever sees the value.

Fix: move `github.action_path` into an env var (e.g. `ACTION_PATH: ${{ github.action_path }}`) and reference it as `"$ACTION_PATH"` in the script; move `inputs.inputs` into an env var (e.g. `INPUTS: ${{ inputs.inputs }}`) and pass it as `"$INPUTS"` to the Python script.

Locations:

- `action.yml:62`
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

Fixed script injection in both steps of action.yml:

1. 'Set up pip-audit' step: Moved `github.action_path` into `ACTION_PATH` env var; replaced `"${{ github.action_path }}/setup/setup.bash"` with `"$ACTION_PATH/setup/setup.bash"`.

2. 'Run pip-audit' step: Moved `github.action_path` into `ACTION_PATH` env var and `inputs.inputs` into `INPUTS` env var; replaced `source "${{ github.action_path }}/setup/venv.bash"` with `source "$ACTION_PATH/setup/venv.bash"` and replaced `${{ github.action_path }}/action.py "${{ inputs.inputs }}"` with `"$ACTION_PATH/action.py" ${INPUTS:+"$INPUTS"}`. The `${INPUTS:+"$INPUTS"}` form is used because `inputs.inputs` defaults to empty string, so the argument should be omitted entirely when empty to preserve original behavior.


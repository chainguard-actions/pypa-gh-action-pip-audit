<!-- markdownlint-disable -->

# Hardening Report: pypa--gh-action-pip-audit/v1.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **pypa--gh-action-pip-audit/v1.0.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In the 'Set up pip-audit' step, `${{ github.action_path }}` is interpolated directly into the shell command: `source "${{ github.action_path }}/setup/setup.bash"`. Any ${{ ... }} expression inside a run: block is a script-injection risk as it flows through YAML template substitution before the shell processes it.

Locations:

- `action.yml:63`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In the 'Run pip-audit' step, both `${{ github.action_path }}` and the attacker-controlled `${{ inputs.inputs }}` are interpolated directly into shell commands: `source "${{ github.action_path }}/setup/venv.bash"` and `${{ github.action_path }}/action.py "${{ inputs.inputs }}"`.

The `inputs.inputs` value is fully attacker-controlled (user-supplied input with no sanitization) and is interpolated directly into the shell command line, enabling shell command injection via metacharacters. The safe pattern is to pass inputs only through env vars and double-quote the env var references in the shell script.

Locations:

- `action.yml:72`
- `action.yml:74`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.inputs }}" appears directly in run: block of step "Run pip-audit"; move to env: map

Locations:

- `action.yml:74`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection issues in action.yml:
1. 'Set up pip-audit' step: moved `${{ github.action_path }}` to env var `GHA_ACTION_PATH`, referenced as `${GHA_ACTION_PATH}` in the shell command.
2. 'Run pip-audit' step: moved `${{ github.action_path }}` to env var `GHA_ACTION_PATH` and `${{ inputs.inputs }}` to env var `GHA_PIP_AUDIT_INPUTS`. Shell commands now use `${GHA_ACTION_PATH}` and `"$GHA_PIP_AUDIT_INPUTS"` respectively. Since action.py uses sys.argv[1].split() and handles empty strings correctly, the inputs env var is always passed as a quoted argument.


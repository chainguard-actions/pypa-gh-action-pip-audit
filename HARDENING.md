<!-- markdownlint-disable -->

# Hardening Report: pypa--gh-action-pip-audit/v1.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pypa--gh-action-pip-audit/v1.0.8** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings in action.yml. In the 'Set up pip-audit' step, `${{ github.action_path }}` is embedded directly in the shell command `source "${{ github.action_path }}/setup/setup.bash"`. In the 'Run pip-audit' step, both `${{ github.action_path }}` and `${{ inputs.inputs }}` are embedded directly in shell commands: `source "${{ github.action_path }}/setup/venv.bash"` and `${{ github.action_path }}/action.py "${{ inputs.inputs }}"`.

The `inputs.inputs` value is fully attacker-controlled (it is a user-supplied input with no sanitization) and is interpolated directly into the shell command line before the shell ever sees it, enabling command injection. Even `github.action_path` and `github.*` context values should not be interpolated directly into `run:` blocks — they must be passed via `env:` variables and then referenced as shell variables.

Locations:

- `action.yml:57`
- `action.yml:64`
- `action.yml:66`

### unpinned-uses (severity: high)

All `uses:` references in the workflow files use mutable tag refs instead of full 40-character commit SHA digests, making them vulnerable to supply-chain attacks if the referenced tag is moved or the upstream repository is compromised.

Failing references:
- `.github/workflows/ci.yml`: `actions/checkout@v3`, `actions/setup-python@v4`
- `.github/workflows/selftest.yml`: `actions/checkout@v3` (used in multiple jobs)
- `.github/workflows/semgrep.yml`: `actions/checkout@v3`

All should be pinned to their full 40-character SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:13`
- `.github/workflows/selftest.yml:10`
- `.github/workflows/selftest.yml:24`
- `.github/workflows/selftest.yml:38`
- `.github/workflows/selftest.yml:55`
- `.github/workflows/selftest.yml:68`
- `.github/workflows/semgrep.yml:17`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no individual job within any of these files defines a `permissions:` key either. Without explicit permissions, workflows run with the default repository permissions (which may include `write` access to contents, pull requests, etc.), violating the principle of least privilege.

Affected files: `.github/workflows/ci.yml`, `.github/workflows/selftest.yml`, `.github/workflows/semgrep.yml`.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/selftest.yml:1`
- `.github/workflows/semgrep.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.inputs }}" appears directly in run: block of step "Run pip-audit"; move to env: map

Locations:

- `action.yml:74`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed script injection in action.yml by moving github.action_path to GHA_ACTION_PATH env var in both steps, and inputs.inputs to GHA_PIP_AUDIT_INPUTS env var. Updated action.py to read inputs from os.getenv('GHA_PIP_AUDIT_INPUTS', '') instead of sys.argv[1]. Pinned all unpinned uses: references to full 40-char SHAs (actions/checkout@v3 -> a37ce9120846195fa4ece8f58b268e6043cb2f26, actions/setup-python@v4 -> 7f4fc3e22c37d6ff65e88745f38bd3157c663f7c). Added permissions: contents: read to all three workflow files (ci.yml, selftest.yml, semgrep.yml).


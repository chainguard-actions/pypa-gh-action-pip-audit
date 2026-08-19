<!-- markdownlint-disable -->

# Hardening Report: pypa--gh-action-pip-audit/v1.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pypa--gh-action-pip-audit/v1.0.7** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of ${{ }} inside run: shell command strings. In the 'Run pip-audit' step (line 71), the attacker-controlled input `${{ inputs.inputs }}` is interpolated directly into the shell command: `${{ github.action_path }}/action.py "${{ inputs.inputs }}"`. An attacker can supply a value like `foo; malicious-command` to execute arbitrary shell commands. Additionally, `${{ github.action_path }}` is interpolated directly in run: blocks in both steps (lines 63, 69, 71) — any ${{ }} expression inside a run: block is a script-injection risk regardless of context. The fix is to route all values through env: variables and reference them as quoted shell variables (e.g., `"$GHA_PIP_AUDIT_INPUTS"`).

Locations:

- `action.yml:63`
- `action.yml:69`
- `action.yml:71`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.inputs }}" appears directly in run: block of step "Run pip-audit"; move to env: map

Locations:

- `action.yml:74`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all ${{ }} expression interpolations in run: blocks by moving them to env: variables:
1. 'Set up pip-audit' step: moved ${{ github.action_path }} to env: GHA_ACTION_PATH, referenced as ${GHA_ACTION_PATH} in shell.
2. 'Run pip-audit' step: moved ${{ github.action_path }} to env: GHA_ACTION_PATH and ${{ inputs.inputs }} to env: GHA_PIP_AUDIT_INPUTS. The inputs value is passed as a single quoted argument "$GHA_PIP_AUDIT_INPUTS" to action.py, which uses sys.argv[1].split() to parse the whitespace-separated list — preserving original behavior while eliminating shell injection risk.

### Iteration 2

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

Fixed all three workflow files:

1. **unpinned-uses**: Pinned all mutable tag references to full 40-char SHAs:
   - `actions/checkout@v3` → `actions/checkout@a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3` (applied in ci.yml, selftest.yml x5, semgrep.yml)
   - `actions/setup-python@v4` → `actions/setup-python@7f4fc3e22c37d6ff65e88745f38bd3157c663f7c # v4` (applied in ci.yml)

2. **missing-permissions**: Added `permissions: {}` at the top level of all three workflow files (deny-all default), and added `permissions: { contents: read }` at the job level for each job that needs to check out code.


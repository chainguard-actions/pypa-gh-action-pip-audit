<!-- markdownlint-disable -->

# Hardening Report: pypa--gh-action-pip-audit/v1.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pypa--gh-action-pip-audit/v1.0.6** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate GitHub Actions expressions inside shell command strings. Step 'Set up pip-audit' uses `source "${{ github.action_path }}/setup/setup.bash"` and step 'Run pip-audit' uses `source "${{ github.action_path }}/setup/venv.bash"` and `${{ github.action_path }}/action.py "${{ inputs.inputs }}"`. Any `${{ ... }}` expression interpolated directly in a `run:` block is a script-injection risk — notably `${{ inputs.inputs }}` is fully attacker-controlled and is passed as a shell argument without going through a quoted env var.

Locations:

- `action.yml:48`
- `action.yml:55`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of full 40-character SHA commit digests, making them vulnerable to supply-chain attacks. Failing references: ci.yml — `actions/checkout@v3`, `actions/setup-python@v4`; selftest.yml — `actions/checkout@v3` (multiple jobs); semgrep.yml — `actions/checkout@v3`. All should be pinned to full SHA digests (e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`).

Locations:

- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:13`
- `.github/workflows/selftest.yml:10`
- `.github/workflows/semgrep.yml:17`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and none of their individual jobs define job-level `permissions:` blocks. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad write access). All three files — ci.yml, selftest.yml, and semgrep.yml — are affected.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/selftest.yml:1`
- `.github/workflows/semgrep.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.inputs }}" appears directly in run: block of step "Run pip-audit"; move to env: map

Locations:

- `action.yml:78`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings:

1. **script-injection / static-inline-injection** (action.yml lines 48, 55, 78): Moved `${{ github.action_path }}` into a new `GHA_ACTION_PATH` env var and `${{ inputs.inputs }}` into a new `GHA_PIP_AUDIT_INPUTS` env var in both `run:` steps. Shell scripts now reference `${GHA_ACTION_PATH}` and `"$GHA_PIP_AUDIT_INPUTS"` instead of interpolating expressions directly. The inputs argument is always passed (even when empty) since action.py expects `sys.argv[1]` to always be present.

2. **unpinned-uses** (ci.yml, selftest.yml, semgrep.yml): Pinned `actions/checkout@v3` → `@a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3` and `actions/setup-python@v4` → `@7f4fc3e22c37d6ff65e88745f38bd3157c663f7c # v4` using verified SHAs.

3. **missing-permissions** (ci.yml, selftest.yml, semgrep.yml): Added `permissions: contents: read` top-level block to all three workflow files.


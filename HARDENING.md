<!-- markdownlint-disable -->

# Hardening Report: pypa--gh-action-pip-audit/v1.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pypa--gh-action-pip-audit/v1.0.5** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

action.yml contains ${{ ... }} expressions directly interpolated inside run: shell command strings (sub-rule a). In the 'Set up pip-audit' step, `${{ github.action_path }}` is interpolated directly in the shell: `source "${{ github.action_path }}/setup/setup.bash"`. In the 'Run pip-audit' step, both `${{ github.action_path }}` and the attacker-controlled `${{ inputs.inputs }}` are interpolated directly in the shell: `${{ github.action_path }}/action.py "${{ inputs.inputs }}"`. Any ${{ ... }} expression inside a run: block is a script-injection risk because YAML template substitution occurs before the shell ever sees the value. The `inputs.inputs` value in particular is fully attacker-controlled and passed unquoted as a shell word, enabling command injection.

Locations:

- `action.yml:64`
- `action.yml:73`
- `action.yml:75`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tag refs instead of full 40-character SHA digests, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: ci.yml — `actions/checkout@v3` and `actions/setup-python@v4`; selftest.yml — `actions/checkout@v3` (used in four jobs); semgrep.yml — `actions/checkout@v3`. All should be pinned to their full commit SHA (e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`).

Locations:

- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:13`
- `.github/workflows/selftest.yml:11`
- `.github/workflows/selftest.yml:24`
- `.github/workflows/selftest.yml:38`
- `.github/workflows/selftest.yml:57`
- `.github/workflows/semgrep.yml:17`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no individual job within any of these files defines a `permissions:` key either. Without explicit permissions, workflows run with the default repository token permissions, which may be overly broad (e.g. write access to contents and pull-requests). All three files — ci.yml, selftest.yml, and semgrep.yml — are affected.

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

Fixed all four findings:

1. **script-injection / static-inline-injection** (action.yml): Moved `${{ github.action_path }}` to `GHA_ACTION_PATH` env var in both 'Set up pip-audit' and 'Run pip-audit' steps. Moved `${{ inputs.inputs }}` to `GHA_PIP_AUDIT_INPUTS` env var. Shell run: blocks now reference these as `${GHA_ACTION_PATH}` and `"$GHA_PIP_AUDIT_INPUTS"` respectively, preventing YAML template injection.

2. **unpinned-uses** (ci.yml, selftest.yml, semgrep.yml): Pinned `actions/checkout@v3` to full SHA `a37ce9120846195fa4ece8f58b268e6043cb2f26` and `actions/setup-python@v4` to `7f4fc3e22c37d6ff65e88745f38bd3157c663f7c`. All 5 occurrences across 3 files updated with `# v3` / `# v4` comments.

3. **missing-permissions** (ci.yml, selftest.yml, semgrep.yml): Added top-level `permissions: {}` to all three workflow files and job-level `permissions: { contents: read }` to each job that checks out code, following least-privilege principle.


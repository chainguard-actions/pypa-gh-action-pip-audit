<!-- markdownlint-disable -->

# Hardening Report: pypa--gh-action-pip-audit/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pypa--gh-action-pip-audit/v1.1.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): In action.yml's 'Run pip-audit' step, the expression `${{ inputs.inputs }}` is directly interpolated inside the `run:` shell command string as a positional argument: `python "${{ github.action_path }}/action.py" "${{ inputs.inputs }}"`.

An attacker-controlled value for the `inputs` input is embedded directly into the shell command before the shell processes it, enabling command injection. The `${{ github.action_path }}` expressions in both `source` commands are also direct `${{ }}` interpolations inside `run:` blocks (in the 'Set up pip-audit' and 'Run pip-audit' steps).

Locations:

- `action.yml:48`
- `action.yml:56`
- `action.yml:58`

### unpinned-uses (severity: high)

Multiple `uses:` references across workflow files are pinned to mutable version tags rather than immutable 40-character SHA digests, making them vulnerable to supply-chain attacks if the upstream tag is moved or compromised.

- ci.yml: `actions/checkout@v3`, `actions/setup-python@v4`
- selftest.yml: `actions/checkout@v3` (appears 5 times)
- semgrep.yml: `actions/checkout@v3`

Only `re-actors/alls-green@05ac9388f0aebcb5727afa17fcccfecd6f8ec5fe` in selftest.yml is correctly pinned.

Locations:

- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:14`
- `.github/workflows/selftest.yml:21`
- `.github/workflows/selftest.yml:38`
- `.github/workflows/selftest.yml:55`
- `.github/workflows/selftest.yml:72`
- `.github/workflows/selftest.yml:85`
- `.github/workflows/semgrep.yml:20`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and none of their individual jobs define job-level `permissions:` keys. Without explicit permissions, workflows run with the repository's default token permissions (which may be `write-all` for older repositories), granting unnecessarily broad access to the GITHUB_TOKEN.

Affected files: ci.yml, selftest.yml, semgrep.yml.

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

1. script-injection / static-inline-injection (action.yml): Moved `${{ github.action_path }}` to `GHA_ACTION_PATH` env var in both steps; moved `${{ inputs.inputs }}` to `GHA_PIP_AUDIT_INPUTS` env var and removed it as a positional CLI argument. Updated action.py to read inputs from `os.getenv('GHA_PIP_AUDIT_INPUTS', '')` instead of `sys.argv[1]`.

2. unpinned-uses: Pinned all mutable tag references to full commit SHAs — `actions/checkout@v3` → `@a37ce9120846195fa4ece8f58b268e6043cb2f26` (7 occurrences in ci.yml, selftest.yml, semgrep.yml); `actions/setup-python@v4` → `@7f4fc3e22c37d6ff65e88745f38bd3157c663f7c` (ci.yml). Tag comments preserved for readability.

3. missing-permissions: Added top-level `permissions: {}` to all three workflow files. Added job-level `permissions: contents: read` to all jobs that checkout code, and `permissions: {}` to the all-selftests-pass job.


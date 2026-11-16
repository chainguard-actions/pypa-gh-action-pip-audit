<!-- markdownlint-disable -->

# Hardening Report: pypa--gh-action-pip-audit/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pypa--gh-action-pip-audit/v1.1.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): expressions are placed directly inside a run: block in the 'Set up pip-audit' step: `source "${{ github.action_path }}/setup/setup.bash"`. Move github.action_path into an env var (or use $GITHUB_ACTION_PATH) and double-quote it.

Locations:

- `action.yml:63`

### script-injection (severity: high)

Rule (a): expressions are placed directly inside a run: block in the 'Run pip-audit' step: `source "${{ github.action_path }}/setup/venv.bash"` and `python "${{ github.action_path }}/action.py" "${{ inputs.inputs }}"`. inputs.inputs is set by the caller, so a value containing a double quote plus shell metacharacters can run commands. Pass it through env (e.g. GHA_PIP_AUDIT_INPUTS) and use "$GHA_PIP_AUDIT_INPUTS"; use "$GITHUB_ACTION_PATH" in place of github.action_path.

Locations:

- `action.yml:72`
- `action.yml:74`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.inputs }}" appears directly in run: block of step "Run pip-audit"; move to env: map

Locations:

- `action.yml:74`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.py without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.py b/action.py
index 3eaffbd..cad0da4 100755
--- a/action.py
+++ b/action.py
@@ -53,7 +53,7 @@ def _fatal_help(msg):
     sys.exit(1)
 
 
-inputs = [Path(p).resolve() for p in sys.argv[1].split()]
+inputs = [Path(p).resolve() for p in os.getenv("GHA_PIP_AUDIT_INPUTS", "").split()]
 
 # The arguments we pass into `pip-audit` get built up in this list.
 pip_audit_args = [

```

Locations:

- `action.py`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index e5b3aec..ea4b090 100644
--- a/action.yml
+++ b/action.yml
@@ -57,22 +57,41 @@ outputs:
 runs:
   using: "composite"
   steps:
     - name: Set up pip-audit
       run: |
         # NOTE: Sourced, not executed as a script.
-        source "${{ github.action_path }}/setup/setup.bash"
+        source "${GHA_ACTION_PATH}/setup/setup.bash"
       env:
+        GHA_ACTION_PATH: "${{ github.action_path }}"
         GHA_PIP_AUDIT_VIRTUAL_ENVIRONMENT: "${{ inputs.virtual-environment }}"
       shell: bash
-
     - name: Run pip-audit
       id: pip-audit
       run: |
         # NOTE: Sourced, not executed as a script.
-        source "${{ github.action_path }}/setup/venv.bash"
+        source "${GHA_ACTION_PATH}/setup/venv.bash"
 
-        python "${{ github.action_path }}/action.py" "${{ inputs.inputs }}"
+        python "${GHA_ACTION_PATH}/action.py"
       env:
+        GHA_ACTION_PATH: "${{ github.action_path }}"
+        GHA_PIP_AUDIT_INPUTS: "${{ inputs.inputs }}"
         GHA_PIP_AUDIT_SUMMARY: "${{ inputs.summary }}"
         GHA_PIP_AUDIT_NO_DEPS: "${{ inputs.no-deps }}"
         GHA_PIP_AUDIT_REQUIRE_HASHES: "${{ inputs.require-hashes }}"

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, prior-fix

**Notes:**

Removed every ${{ }} expression from the two run: blocks in action.yml. Each step now gets github.action_path from a GHA_ACTION_PATH env var and uses it double-quoted. inputs.inputs now reaches the script through a GHA_PIP_AUDIT_INPUTS env var and is no longer passed on the command line. action.py now reads GHA_PIP_AUDIT_INPUTS from the environment (default empty) instead of sys.argv[1]. It still splits the value on whitespace with .split(), the same as upstream, so how a multi-path value is handled is unchanged. This also re-applies both prior-fix patches.


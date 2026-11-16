<!-- markdownlint-disable -->

# Hardening Report: pypa--gh-action-pip-audit/v1.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pypa--gh-action-pip-audit/v1.0.8** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Set up pip-audit' step puts a GitHub expression straight into its run: script: `source "${{ github.action_path }}/setup/setup.bash"`. Use the $GITHUB_ACTION_PATH env var instead: `source "$GITHUB_ACTION_PATH/setup/setup.bash"`.

Locations:

- `action.yml:63`

### script-injection (severity: high)

Rule (a): the 'Run pip-audit' step puts GitHub expressions straight into its run: script: `source "${{ github.action_path }}/setup/venv.bash"` and `${{ github.action_path }}/action.py "${{ inputs.inputs }}"`. inputs.inputs can be set by whoever calls the action, and it is substituted into the script before the shell parses it. Pass it through an env var (e.g. GHA_PIP_AUDIT_INPUTS: ${{ inputs.inputs }}) and refer to it as "$GHA_PIP_AUDIT_INPUTS". Use "$GITHUB_ACTION_PATH" for the action path.

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
index dd624ad..a0a4748 100755
--- a/action.py
+++ b/action.py
@@ -49,7 +49,7 @@ def _fatal_help(msg):
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
index 39e9d3d..aef2af4 100644
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
 
-        ${{ github.action_path }}/action.py "${{ inputs.inputs }}"
+        "${GHA_ACTION_PATH}/action.py"
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

action.yml no longer has any ${{ }} expressions inside run: scripts. Both steps now find the action's files through the runner-provided $GITHUB_ACTION_PATH (quoted) instead of ${{ github.action_path }}. inputs.inputs now goes in through the env var GHA_PIP_AUDIT_INPUTS. action.py reads it with os.getenv("GHA_PIP_AUDIT_INPUTS", "").split() instead of sys.argv[1].split(), which is the same change the previous harden made (both prior-fix patches are re-applied). The value is still split on whitespace in Python, as upstream did, so the list behaves the same. Nothing was run or tested.


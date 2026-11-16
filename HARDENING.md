<!-- markdownlint-disable -->

# Hardening Report: pypa--gh-action-pip-audit/v1.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pypa--gh-action-pip-audit/v1.0.7** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Set up pip-audit' step puts a ${{ }} expression straight into its run: script: `source "${{ github.action_path }}/setup/setup.bash"`. Use "$GITHUB_ACTION_PATH" instead.

Locations:

- `action.yml:63`

### script-injection (severity: high)

Rule (a): the 'Run pip-audit' step puts ${{ }} expressions straight into its run: script: `source "${{ github.action_path }}/setup/venv.bash"` and `${{ github.action_path }}/action.py "${{ inputs.inputs }}"`. inputs.inputs comes from the caller, so it is pasted into the shell script before bash runs it. Shell syntax in it, such as a closing quote, can then inject commands. Pass the input through an env: variable and use "$GITHUB_ACTION_PATH" for the path.

Locations:

- `action.yml:72`
- `action.yml:74`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.inputs }}" appears directly in run: block of step "Run pip-audit"; move to env: map

Locations:

- `action.yml:74`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 39e9d3d..d8f378e 100644
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
+        "${GHA_ACTION_PATH}/action.py" "$GHA_PIP_AUDIT_INPUTS"
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

I took every ${{ }} expression out of the run: scripts in action.yml. Both steps now use "${GITHUB_ACTION_PATH}" for the action's own path instead of ${{ github.action_path }}. The pip-audit inputs value now comes in through a new env variable, GHA_PIP_AUDIT_INPUTS, and is still passed as one double-quoted argument, the same as upstream. Upstream already passed it to action.py as one argument, so this doesn't change how action.py receives it. This re-applies what the previous harden fixed. I made no other changes and didn't run anything to test it.


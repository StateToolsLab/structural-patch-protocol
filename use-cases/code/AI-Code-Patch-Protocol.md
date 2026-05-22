# AI Code Patch Protocol

A code-oriented application profile for Structural Patch Protocol (SPP).

## 0. Purpose

This document defines how an AI agent should modify an existing codebase under SPP.

The goal is to prevent:

```text
wrong-file edits
schema hallucination
API misuse
selector mismatch
over-generalized replacement
unverified patch stacking
scope drift
```

## 1. Required Flow

```text
Target Lock
→ Actual Artifact First
→ Probe Patch
→ Verify
→ Failed Patch Withdrawal / Verified Merge
→ Cleanup
→ Stop
```

## 2. Before Editing

The agent must identify:

```text
Target file(s)
Target function / class / component / selector / schema
Reason for change
Expected effect
Verification method
Rollback method
```

## 3. Actual Artifact First

The agent must inspect the actual codebase before editing.

Examples:

```text
ls / find / tree
open target files
read adjacent code
inspect existing tests
inspect actual DB schema
inspect actual API response
inspect actual DOM or CSS selector
```

## 4. Probe Patch

The agent must first apply one minimal patch.

The Probe Patch should be:

```text
small enough to review
small enough to revert
large enough to test the intended effect
inside the locked target
```

## 5. Verification

After the Probe Patch, use at least one suitable verification method:

```text
unit test
integration test
type check
lint
build
manual UI check
computed style inspection
DB query
API response check
git diff review
```

The verification question is:

```text
Did this specific patch cause the expected effect?
```

## 6. Failed Patch Withdrawal

If the Probe Patch fails, the agent must withdraw it before trying another patch.

```text
No second patch before resolving the first failed patch.
```

Allowed actions:

```text
revert the failed patch;
identify the failure cause and create a revised Probe Patch;
retain the patch only with explicit evidence that it is necessary.
```

Forbidden:

```text
keep the failed patch;
add another patch on top;
repeat until something appears to work;
leave unexplained layers behind.
```

## 7. Verified Merge

If the Probe Patch succeeds, the agent may apply the same transformation to structurally matching targets.

Do not apply a transformation merely because another location looks similar.

```text
similar-looking ≠ structurally matching
```

## 8. Cleanup

Before stopping, the agent must clean up:

```text
failed probes
old selectors
superseded CSS
unused functions
duplicate logic
obsolete comments
scattered temporary definitions
unexplained patch layers
```

Cleanup should consolidate effective structure into canonical form.

## 9. Stop

The agent must stop when:

```text
the target is fixed;
verification has passed;
failed patches are withdrawn;
cleanup is complete for the touched scope;
diff is within scope;
further work requires a new task.
```

## 10. Code Review Output

Every AI code patch should report:

```text
Target
Artifact inspected
Patch applied
Verification result
Failed patches withdrawn
Files changed
Remaining issues
Stop condition
```

Use:

```text
templates/spp-run-log.md
```

## 11. Minimal Prompt for AI Coding Agents

```text
Follow SPP.
Before editing, inspect the actual artifact.
Apply one minimal Probe Patch.
Verify whether that specific patch worked.
If it failed, withdraw it before applying another patch.
If it succeeded, merge only verified transformations to structurally matching targets.
Clean up obsolete patch layers.
Stop when the target is stable and report remaining issues separately.
```

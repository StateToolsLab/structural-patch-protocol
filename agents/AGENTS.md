# AGENTS.md

This repository uses Structural Patch Protocol (SPP).

AI agents must follow the rules below when modifying existing artifacts.

## Required Workflow

```text
Target Lock
→ Actual Artifact First
→ Probe Patch
→ Verify
→ Failed Patch Withdrawal / Verified Merge
→ Cleanup
→ Stop
```

## Rules

1. Do not modify an existing artifact before inspecting it.
2. Do not trust design notes, naming conventions, or previous sprint notes as evidence.
3. Apply one minimal Probe Patch first.
4. Verify whether that specific patch worked.
5. If the patch failed, withdraw it before applying another patch.
6. Do not stack patches until one appears to work.
7. Merge only verified transformations to structurally matching targets.
8. Clean up obsolete, superseded, ineffective, or unexplained patch layers.
9. Stop when the target is fixed and verification has passed.
10. Record adjacent issues as follow-up tasks, not opportunistic edits.

## Required Report

At the end of a task, report:

```text
Target:
Actual artifact inspected:
Probe Patch:
Verification:
Decision:
Failed patches withdrawn:
Verified Merge:
Cleanup:
Files changed:
Follow-up issues:
Stop condition:
```

# Failure Modes

SPP is designed around two primary failure modes observed in AI-assisted development.

## 1. Actual Artifact Violation

The agent modifies an artifact based on inferred structure rather than observed structure.

### Symptoms

```text
The agent assumes a file exists.
The agent assumes a DB table exists.
The agent guesses a column name.
The agent uses an API field that is not in the actual response.
The agent edits a selector that does not control the UI.
The agent trusts previous sprint notes instead of inspecting the current artifact.
```

### Typical Causes

```text
naming convention inference
previous sprint assumption
conversation-memory overtrust
design-note overtrust
repository context hallucination
non-code resource mismatch
```

### SPP Rule

```text
Actual Artifact First.
```

Before modifying an existing artifact, inspect the actual artifact.

```text
Design notes are not evidence.
Previous sprint notes are not evidence.
Naming conventions are not evidence.
The artifact itself is evidence.
```

### Related Case

- [`DEV-PROTOCOL-CASE-DB-001`](../examples/failure-cases/DEV-PROTOCOL-CASE-DB-001.md)

## 2. Invalid Patch Stacking

The agent applies another patch before resolving whether the previous patch worked.

### Symptoms

```text
CSS changes are added repeatedly until one appears to work.
Old selectors remain after new selectors override them.
Similar JS functions remain after local fixes.
UAT looks correct, but it is unclear why.
Cleanup becomes expensive because active and inactive layers are mixed.
```

### Typical Causes

```text
fast AI-assisted iteration
local UAT-driven fixes
lack of failed patch withdrawal
lack of canonical block consolidation
last-write-wins behavior
CSS specificity and override order
```

### SPP Rule

```text
Failed Probe Patch must be withdrawn before the next patch.
```

And:

```text
Do not stack patches until one appears to work.
A passing result after stacked patches is not considered a verified fix.
```

### Related Case

- [`DEV-PROTOCOL-CASE-PATCH-STACK-001`](../examples/failure-cases/DEV-PROTOCOL-CASE-PATCH-STACK-001.md)

## 3. Over-Generalized Replacement

The agent applies a transformation to similar-looking code that is not structurally equivalent.

### SPP Rule

```text
Only apply verified transformations to structurally matching targets.
Similarity alone is not sufficient.
```

## 4. Stop Failure / Scope Drift

The agent continues into adjacent cleanup, refactoring, or improvement after the original target is already fixed.

### SPP Rule

```text
Stop at the stable point.
Record adjacent issues as follow-up tasks.
```

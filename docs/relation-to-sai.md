# Relation to SAI

## Reference

SAI Protocol repository:

- [`StateToolsLab/sai-protocol`](https://github.com/StateToolsLab/sai-protocol)


SPP is designed to work with SAI or any equivalent structural-address system.

## Role Division

```text
SAI = what to touch
SPP = how to safely touch it
```

## SAI Before SPP

SAI locks the target.

SPP governs what happens after the target is locked.

```text
SAI:
  Identify the structure.
  Assign or resolve the structural address.
  Prevent target confusion.

SPP:
  Inspect the actual artifact.
  Apply a Probe Patch.
  Verify the effect.
  Withdraw failed patches.
  Merge verified transformations.
  Clean up ineffective layers.
  Stop at a stable point.
```

## Example

```text
SAI target:
  ui/static/style.css::RightPaneAssistMenu.ButtonLayout

SPP operation:
  1. Inspect the actual CSS block and rendered UI.
  2. Apply one minimal Probe Patch.
  3. Verify the computed style or UAT result.
  4. If it fails, withdraw the patch.
  5. If it succeeds, merge into the canonical block.
  6. Remove superseded declarations.
  7. Stop.
```

## Why This Separation Matters

Without SAI, an agent may patch the wrong target.

Without SPP, an agent may patch the right target in the wrong way.

Together:

```text
SAI prevents target confusion.
SPP prevents unsafe modification.
```

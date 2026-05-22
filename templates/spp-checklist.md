# SPP Checklist

## Target

- [ ] Target is explicitly identified.
- [ ] Target is inside the requested scope.
- [ ] Adjacent targets are excluded or marked as follow-up.

## Actual Artifact First

- [ ] Actual file / DB / API / UI / config was inspected.
- [ ] Design notes or previous sprint notes were not treated as evidence.
- [ ] Actual structure was recorded.

## Probe Patch

- [ ] One minimal Probe Patch was applied.
- [ ] Patch was reversible.
- [ ] Patch was inside the locked target.

## Verify

- [ ] Verification method was selected before merge.
- [ ] Verification checked the effect of this specific patch.
- [ ] Result was recorded as Success / Failure / Unknown.

## Failed Patch Withdrawal

- [ ] Failed Probe Patch was reverted or explicitly resolved.
- [ ] No second patch was stacked on unresolved failure.
- [ ] No unexplained patch layer remains.

## Verified Merge

- [ ] Transformation was applied only to structurally matching targets.
- [ ] Similarity alone was not used as justification.
- [ ] Verification still passed after merge.

## Cleanup

- [ ] Superseded patches were removed.
- [ ] Duplicate logic or selectors were removed or justified.
- [ ] Remaining structure is canonical and explainable.

## Stop

- [ ] Target is fixed.
- [ ] Diff is within scope.
- [ ] Follow-up issues are recorded separately.
- [ ] Agent stopped at a stable point.

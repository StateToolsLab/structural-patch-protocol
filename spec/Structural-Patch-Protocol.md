# Structural Patch Protocol

**Version:** 0.1  
**Status:** Draft  
**Repository:** https://github.com/StateToolsLab/structural-patch-protocol

---

## 0. Purpose

Structural Patch Protocol (SPP) is a quality-control protocol for AI-driven changes to existing artifacts.

SPP is designed to prevent AI agents from:

- modifying artifacts before inspecting them;
- relying on inferred structure instead of observed structure;
- applying plausible but erroneous patches;
- stacking ineffective patches until something appears to work;
- applying unverified transformations to similar-looking targets;
- leaving obsolete or unexplained patch layers behind;
- continuing beyond a stable stopping point.

SPP converts AI-driven patching from:

```text
inference-first generation
```

into:

```text
evidence-first modification
```

SPP is not a prompt trick.  
SPP is a protocol for controlling how AI agents modify existing artifacts.

---

## 1. Scope

SPP applies when an AI agent modifies an existing artifact.

The artifact may be:

- source code;
- database schema;
- API integration;
- JSON / CSV schema;
- UI / DOM / CSS;
- configuration file;
- prompt;
- documentation;
- generated artifact;
- design asset.

SPP is domain-neutral.  
It is not limited to software code.

---

## 2. Relation to SAI

SPP is designed to work with SAI-Protocol.

```text
SAI = what to touch
SPP = how to safely touch it
```

SAI identifies and locks the target.  
SPP controls the modification process after the target has been identified.

SAI repository:

https://github.com/StateToolsLab/sai-protocol

SPP can also be used without SAI when a target is clearly identified by another stable addressing method.

---

## 3. Core Flow

The core SPP flow is:

```text
Target Lock
→ Actual Artifact First
→ Probe Patch
→ Verify
→ Failed Patch Withdrawal / Verified Merge
→ Cleanup
→ Stop
```

Each step is mandatory unless explicitly marked as not applicable.

---

## 4. Target Lock

Before modifying anything, the agent must identify the target.

The target may be:

- a file;
- function;
- class;
- component;
- route;
- database table;
- database column;
- schema;
- selector;
- UI block;
- configuration entry;
- prompt section;
- document section;
- generated artifact.

The target must be constrained enough that the agent can answer:

```text
What artifact will be modified?
Where is it located?
Why is this the intended target?
What is outside the target scope?
```

When available, the target should be locked using SAI or an equivalent structural address.

---

## 5. Actual Artifact First Gate

Before modifying an existing artifact, the agent must inspect the actual artifact.

SPP treats the artifact itself as the primary evidence.

```text
Design notes are not evidence.
Previous sprint notes are not evidence.
Naming conventions are not evidence.
The artifact itself is evidence.
```

Examples:

```text
Files:
  ls
  find
  tree
  open the actual file

Database:
  verify DB path
  SHOW TABLES
  DESCRIBE <table>
  inspect actual column names

API:
  inspect actual response
  inspect schema
  verify version
  inspect existing tests

CSV / JSON:
  inspect headers
  inspect keys
  inspect sample rows

UI:
  inspect DOM
  inspect selectors
  inspect rendered behavior

Configuration:
  inspect actual config file
  inspect actual environment variable names
```

The agent must not proceed with a patch based only on assumed structure.

---

## 6. Probe Patch

A Probe Patch is one minimal change used to test whether the intended modification is valid.

A Probe Patch must be:

- small;
- reversible;
- inspectable;
- explainable;
- limited to the locked target;
- designed to test one transformation.

A Probe Patch is not an invitation to accumulate trial-and-error layers.

```text
One Probe, One Verification, One Decision.
```

---

## 7. Verify

After applying a Probe Patch, the agent must verify whether the expected effect occurred.

Verification may include:

- unit test;
- integration test;
- type check;
- lint;
- build;
- database query;
- API response;
- UI confirmation;
- screenshot;
- log inspection;
- diff review;
- human review.

The agent must decide one of the following:

```text
Success → proceed to Verified Merge
Failure → withdraw the patch before trying another one
Unknown → improve verification before continuing
```

An unknown result is not a success.

Unknown is not a success.

If the agent cannot determine whether the Probe Patch produced the expected effect, it must not proceed to Verified Merge.

The agent must either:

- improve the verification method;
- collect additional evidence;
- withdraw the Probe Patch; or
- stop and report the uncertainty.

A Probe Patch with unknown effect must not be carried forward as an active patch layer.


---

## 8. Failed Patch Withdrawal Rule

If a Probe Patch does not produce the expected effect, the agent must withdraw or explicitly resolve the failed patch before applying another patch.

```text
Failed Probe Patch must be withdrawn before the next patch.
```

The following pattern is prohibited:

```text
Patch does not work
→ Add another patch
→ Still unclear
→ Add a stronger patch
→ Later patch wins
→ Artifact appears correct
→ Old patches remain
```

A passing result after stacked patches is not considered a verified fix.

The agent may proceed only if one of the following is true:

- the failed patch has been reverted;
- the failure cause has been identified and the patch has been revised as a new Probe Patch;
- the patch is intentionally retained with explicit evidence that it is necessary.

---

## 9. Verified Merge

A verified transformation may be applied beyond the Probe Patch only when all of the following are true:

- the Probe Patch succeeded;
- the verification result is clear;
- the target is structurally matching;
- the transformation pattern is explicit;
- the scope remains within the task boundary;
- no failed patch layer is being carried forward.

Similarity alone is not sufficient.

```text
Do not generalize from similarity alone.
Only apply verified transformations to structurally matching targets.
```

---

## 10. Cleanup Gate

Before stopping, the agent must remove obsolete, superseded, ineffective, or unexplained patch layers.

Cleanup is not mere deletion.

```text
Cleanup is reconstruction of the currently effective structure into a canonical form.
```

The agent should identify:

```text
What is currently effective?
What was superseded?
What was ineffective?
What can be removed?
What should become canonical?
```

Cleanup is especially important for:

- CSS overrides;
- UI micro-fixes;
- duplicated JavaScript logic;
- scattered i18n text definitions;
- configuration patches;
- temporary compatibility code;
- development-phase labels or comments.

The agent must not treat a working artifact as stable if the effective structure is unclear.

---

## 11. Stop Rule

The agent must stop when:

- the target is fixed;
- verification has passed;
- failed patches have been withdrawn or resolved;
- obsolete patch layers have been cleaned up;
- the diff is within scope;
- further changes would require a new judgment.

Adjacent issues must be recorded as follow-up tasks.

The agent must not continue into unrelated cleanup, refactoring, or improvements unless explicitly requested.

---

## 12. Required Run Log

For traceability, the agent should record:

```text
Target:
Observed artifact:
Probe Patch:
Verification:
Decision:
Failed patch withdrawal:
Verified Merge:
Cleanup:
Files changed:
Remaining issues:
Follow-up tasks:
```

This log may be included in a pull request, issue, commit message, or agent report.

---

## 13. Failure Modes

SPP is designed to prevent two primary failure modes.

### 13.1 Actual Artifact Violation

The agent modifies an artifact based on inferred structure rather than observed structure.

Examples:

- nonexistent database column;
- wrong table name;
- wrong file path;
- guessed API field;
- assumed schema from previous sprint notes;
- guessed selector;
- nonexistent function or class.

SPP response:

```text
Inspect the actual artifact before modifying it.
```

Related case:

```text
examples/failure-cases/DEV-PROTOCOL-CASE-DB-001.md
```

### 13.2 Invalid Patch Stacking

The agent applies another patch before resolving whether the previous patch worked.

Examples:

- CSS override layers;
- repeated UI micro-fixes;
- old selectors left behind;
- duplicated JS logic;
- scattered i18n definitions;
- final UI works, but the effective structure is unclear.

SPP response:

```text
Failed Probe Patch must be withdrawn before the next patch.
```

Related case:

```text
examples/failure-cases/DEV-PROTOCOL-CASE-PATCH-STACK-001.md
```

---

## 14. Domain Gates

Different artifact types require different inspection gates.

### 14.1 Database Gate

Before modifying code that depends on a database:

```text
verify DB file path
list tables
describe relevant tables
confirm column names
confirm sample rows if needed
```

### 14.2 API Gate

Before modifying API integration:

```text
verify endpoint
inspect actual response
inspect schema
verify version
check existing tests
```

### 14.3 UI / CSS Gate

Before modifying UI or CSS:

```text
inspect DOM
inspect selector
identify active CSS block
verify rendered behavior
avoid adding stronger overrides without cleanup
```

### 14.4 Configuration Gate

Before modifying configuration:

```text
inspect actual config file
confirm variable names
confirm runtime loading order
confirm environment-specific overrides
```

### 14.5 Documentation / Prompt Gate

Before modifying documents or prompts:

```text
identify target section
inspect surrounding context
avoid changing adjacent claims
preserve stable terminology
```

---

## 15. Minimal Agent Instruction

A minimal SPP instruction for an AI agent is:

```text
Before modifying an existing artifact:

1. Lock the target.
2. Inspect the actual artifact.
3. Apply one minimal Probe Patch.
4. Verify the effect.
5. If it fails, withdraw it before trying another patch.
6. If the result is unknown, improve verification or withdraw the patch.
7. If it succeeds, merge only verified transformations.
8. Remove obsolete or ineffective layers.
9. Stop when stable.
```

---

## 16. Non-Goals

SPP does not claim to:

- replace tests;
- replace review;
- replace version control;
- prove semantic correctness;
- provide security guarantees;
- prevent all hallucinations;
- automate all cleanup decisions;
- function as a prompt template.

SPP is a quality-control protocol, not a correctness proof.

---

## 17. License

This specification is released under the MIT License.

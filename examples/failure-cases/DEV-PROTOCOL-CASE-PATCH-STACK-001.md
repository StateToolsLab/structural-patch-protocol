# DEV-PROTOCOL-CASE-PATCH-STACK-001

# Patch Stratification Problem in AI-Assisted Cleanup

## 0. Status

Observational failure case.

This case documents a patch accumulation problem observed during the cleanup phase of an AI-assisted development project.

## 1. Context

Project:

```text
AI Deck Reconstructor / DevSP11 Cleanup
```

This cleanup exposed a major issue that was not simply “unused code removal.”

The actual problem was that many local patches created during fast AI-assisted development had accumulated across CSS, JavaScript, and HTML. These patches formed layers of historical intent, making it difficult to determine which declarations, selectors, functions, and text definitions were currently controlling the UI.

A typical sequence looked like this:

```text
Initial implementation
→ UAT fix
→ Visual adjustment
→ Exception handling
→ EN/JP support
→ Dark/light mode support
→ Additional micro-fix
→ New declarations overriding old declarations
```

As a result, the UI appeared to work, but the internal structure contained old development-phase comments, duplicate selectors, overriding CSS blocks, similar functions, and scattered text definitions.

The cleanup cost increased because it became difficult to decide:

```text
What can be safely deleted?
What is still active?
What only appears necessary because of later overrides?
What will break if removed?
```

## 2. Failure Mode

This case illustrates:

```text
Patch Stratification Problem
```

Japanese:

```text
パッチ積層化問題
```

Definition:

```text
A condition where repeated local patches accumulate as historical layers, and later patches override earlier ones without removing them, making the current effective structure difficult to identify.
```

In AI-assisted development, this tends to happen when the agent repeatedly adds patches to solve visible problems without withdrawing ineffective patches or consolidating successful ones into a canonical structure.

## 3. Why This Matters for SPP

This case directly motivates the following SPP rules:

```text
One Probe, One Verification, One Decision.
```

```text
Failed Probe Patch must be withdrawn before the next patch.
```

```text
Stacking patches until one appears to work is prohibited.
```

```text
Cleanup is not deletion.
Cleanup is reconstruction of the currently effective structure.
```

The problem is not merely that the code becomes messy.

The deeper problem is that the causal relationship between patch and effect becomes unclear.

```text
The UI works,
but it is no longer obvious why it works.
```

That state should not be treated as a stable endpoint.

## 4. Use Case 1: CSS Override Stratification

### 4.1 Symptom

Visual changes around buttons, the right pane, and Preview Control did not clearly appear in UAT, even after CSS was modified.

Example:

```css
.btn.action-btn .btn-main {
  display: block;
  font-size: 12px;
  line-height: 1.1;
}

.btn.action-btn .btn-sub {
  display: block;
  font-size: 10px;
  line-height: 1.1;
}
```

The code had changed, but the visible UI changed only slightly or not at all.

### 4.2 Likely Cause

Later CSS blocks, stronger selectors, or overlapping declarations were still controlling the final appearance.

The effective UI was likely determined by several accumulated layers:

```text
Old Dev7 / Dev8 adjustments
+
Dev10 light/dark mode adjustments
+
DevSP11 cleanup adjustments
```

The final effective property was not centralized in one canonical block.

### 4.3 Cleanup Response

The cleanup strategy changed from adding more CSS to identifying the current effective structure.

```text
Identify the final effective property
→ Identify the dominant existing block
→ Remove duplicate or obsolete declarations
→ Consolidate into a canonical block
```

### 4.4 Lesson

In CSS cleanup, the important question is not:

```text
Did the code change?
```

The important question is:

```text
Is this declaration actually controlling the final UI?
```

## 5. Use Case 2: Residual Development Labels

### 5.1 Symptom

Development-phase labels such as `Dev7`, `Dev8`, `dev7`, and `dev8` remained in comments and CSS block names.

These labels were useful during development but became misleading during cleanup.

### 5.2 Cause

AI-assisted development often uses temporary development labels to distinguish implementation phases:

```text
UI created in Dev7
Right pane added in Dev8
Dark mode adjusted in Dev10
```

This is useful for conversation, handoff, branches, issues, and pull requests.

However, when these labels remain inside code, future readers must interpret the code through past work phases rather than current functional responsibility.

### 5.3 Cleanup Response

Development labels were detected using grep and replaced with responsibility-based comments.

Example:

```text
Dev7 style cleanup
```

became:

```text
Theme library panel styles
```

Example:

```text
Dev8 right pane gap
```

became:

```text
Right pane assist menu spacing
```

### 5.4 Lesson

Development labels are useful in temporary coordination artifacts.

They should not remain as permanent code comments or identifiers unless they still describe current structure.

## 6. Use Case 3: Distributed i18n Text Definitions

### 6.1 Symptom

During JP/EN localization, UI labels and status messages became distributed across multiple functions, objects, and DOM update locations.

A sequence of valid local commits accumulated:

```text
Localize assist modal tabs
Localize assist asset edit panel
Localize material settings modal
Localize preview workspace labels
Localize asset edit status messages
```

Each commit was locally correct.

However, the overall structure became increasingly scattered.

### 6.2 Cause

Retrofitting i18n into an existing UI tends to replace hardcoded text in many small locations.

The following categories often get handled at different times:

```text
Button labels
Status messages
Modal titles
Placeholders
Notifications
Error messages
```

This makes text ownership unclear.

### 6.3 Cleanup Response

Remaining hardcoded text was localized incrementally.

However, the future direction should be to centralize UI strings.

Example:

```js
const UI_STRINGS = {
  ja: {},
  en: {}
};
```

Or by feature-level dictionaries:

```text
assetEditMessages
previewWorkspaceLabels
materialSettingsLabels
assistModalLabels
```

### 6.4 Lesson

i18n becomes patch-stratified when it is added after UI growth.

New UI should avoid embedding user-facing text directly into DOM updates. Text should go through dictionaries from the beginning.

## 7. Use Case 4: “Looks Correct, Internally Dirty”

### 7.1 Symptom

The UAT result appeared correct, but the internal code still contained old CSS, unused functions, duplicate logic, and overlapping declarations.

The cleanup repeatedly encountered this state:

```text
It works.
But it is hard to tell why it works.
Removing something might break it.
```

### 7.2 Cause

AI-assisted development tends to patch visible UAT issues locally:

```text
Fix this spacing
Tighten this button
Adjust this modal
Change this text
```

Each local change is fast.

But over time, similar code remains in multiple places.

### 7.3 Cleanup Response

The effective cleanup flow was:

```text
1. Use grep to extract candidates
2. Use diff to constrain the change range
3. Verify the UI in UAT
4. Commit in small units if stable
5. Revert if the visible effect is unclear or risky
```

For CSS, it was especially important to identify:

```text
Which block is actually controlling the final display?
```

### 7.4 Lesson

Cleanup is not simply deletion.

Cleanup is the process of rediscovering the current effective structure and reorganizing code around it.

## 8. Use Case 5: Local Improvements Around the Right Pane and Assist Menu

### 8.1 Symptom

The Assist Menu, Asset Edit, Text Settings, Theme, and Preview Workspace areas were modified across multiple development phases.

The following areas became interdependent:

```text
Theme
Text Settings
Asset Edit
Preview Workspace
Material Settings
Context Menu
```

A local adjustment in one area could affect the appearance or behavior of another.

### 8.2 Cause

Features continued to grow inside the same files:

```text
ui/static/app.js
ui/static/style.css
index.html
```

The implementation grew faster than the file structure was modularized.

### 8.3 Cleanup Response

The cleanup did not immediately split files, because that would have introduced risk.

Instead, it prioritized:

```text
Remove obsolete development labels
Reduce duplicate CSS
Avoid visible UI regressions
Complete JP/EN text handling
Commit in small units
```

### 8.4 Future Direction

After reaching a stable point, the project should consider feature-level splitting.

Possible CSS files:

```text
assist-menu.css
preview-workspace.css
theme-gallery.css
text-settings.css
asset-edit.css
```

Possible JavaScript files:

```text
i18n.js
preview-workspace.js
asset-edit.js
theme-gallery.js
```

### 8.5 Lesson

File splitting should happen at a stable point.

Splitting too early during cleanup can create additional patch risk.

## 9. Use Case 6: From Additional Patches to Canonical Blocks

### 9.1 Symptom

Early development solved problems by adding patches:

```text
Spacing is wrong → add CSS
Color is weak → add CSS
Text is not localized → add JS
Button is cramped → add CSS
```

This is fast during exploration.

However, during cleanup, continuing this approach only adds more layers.

### 9.2 Cleanup Shift

The important transition was:

```text
Fix by adding another patch
↓
Extract the final effective properties
↓
Consolidate them into a canonical block
↓
Remove obsolete patches
```

In short:

```text
patch accumulation
↓
canonical structure
```

### 9.3 Lesson

AI-assisted development should distinguish between exploration and cleanup.

```text
Exploration phase:
Fast iteration is allowed.
Some inconsistency may be acceptable.

Cleanup phase:
The effective structure must be identified, consolidated, and simplified.
```

## 10. SPP Interpretation

This case shows why SPP must include not only Probe Patch and Verify, but also Failed Patch Withdrawal and Cleanup.

A successful-looking UI is not enough.

SPP requires that the agent can explain:

```text
What was changed?
Why was it changed?
How was the effect verified?
Which old patches were removed?
Which remaining structure is canonical?
```

This case supports the following SPP flow:

```text
Target Lock
→ Actual Artifact First
→ Probe Patch
→ Verify
→ Verified Merge
→ Failed Patch Withdrawal
→ Cleanup
→ Stop
```

## 11. General Lesson

This was not ordinary code dirt.

It was a side effect of high-speed AI-assisted development.

```text
AI can rapidly accumulate local patches.
But without periodic structural consolidation, those patches become strata.
```

Japanese:

```text
AIは局所最適パッチを高速に積める。
しかし、人間またはプロトコルが定期的に構造として再統合しないと、パッチは地層化する。
```

The effective cleanup flow was:

```text
grep
→ diff
→ identify final effective properties
→ consolidate into canonical blocks
→ UAT
→ small commit
```

## 12. Recommended Rule Added to SPP

```text
Failed Probe Patch must be withdrawn before the next patch.
```

Expanded:

```text
If a Probe Patch does not produce the expected effect, it must be reverted or explicitly resolved before another patch is applied.
```

And:

```text
Do not stack patches until one appears to work.
A successful result after stacked patches is not considered a verified fix.
```

Japanese:

```text
効かなかったProbe Patchは、次のパッチを当てる前に取り下げる。

効くまでパッチを積み重ねてはならない。
積層後に成功した結果は、検証済み修正とは見なさない。
```

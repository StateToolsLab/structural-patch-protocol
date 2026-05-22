# Structural Patch Protocol: A Structural Modification Protocol for Preventing Inference-Based Erroneous Patches by AI Agents

**Status:** Working Paper  
**Version:** 0.1  
**Repository:** https://github.com/StateToolsLab/structural-patch-protocol

---

## Overview

As AI-assisted development becomes increasingly common, AI agents are now able to rapidly modify a wide range of existing artifacts, including source code, database schemas, APIs, UI components, CSS, configuration files, documentation, prompts, and generated outputs. This significantly accelerates development, but it also exposes a recurring failure mode: an AI agent may infer the structure of an artifact before inspecting the artifact itself, and then apply a plausible but erroneous patch.

This problem is not entirely unaddressed in prior work. In the field of Automated Program Repair (APR), **plausible but incorrect patches** and **patch overfitting** have long been recognized as cases where a patch passes available tests but is not semantically correct. In research on LLM-based code generation, **code hallucination** and **Project Context Conflict** have been reported, including cases where generated code incorrectly refers to repository-specific dependencies, APIs, configuration values, or non-code resources. In addition, benchmarks such as SWE-bench evaluate whether AI systems can generate patches for real GitHub issues in real repositories. In other words, the central challenge is no longer just isolated function generation, but how to safely modify existing artifacts.

However, these discussions mainly focus on the correctness of AI-generated patches, the gap between test-passing behavior and semantic correctness, and failures to reference repository context accurately. This paper additionally focuses on an operational quality problem that emerges during iterative AI-assisted development: ineffective local patches may remain in place, new patches may be stacked on top of them, and eventually it becomes unclear which change is actually responsible for the final behavior. SPP reframes the problems of erroneous patches, hallucination, and context conflict as a practical change-management protocol for AI-driven modification work.

A typical example is an AI agent assuming, before inspection, that “this column name is probably standard,” “this structure should have been completed in the previous sprint,” “this selector should affect this UI,” or “this patch can be applied to similar-looking locations.” Based on such inference, the agent may modify an artifact while assuming nonexistent files, tables, columns, APIs, functions, selectors, or configuration values. This is not merely a code generation mistake. It is a structural work-quality problem in which inference precedes actual artifact inspection.

Another common problem appears in AI-assisted development workflows characterized by rapid iteration, especially in vibe coding-like workflows. An ineffective local patch may not be withdrawn before another patch is applied. In CSS, JavaScript, HTML, UI state management, and configuration files, a later patch may appear to work because of last-write-wins behavior, selector specificity, loading order, or state update order. Yet in such a state, it becomes difficult to determine what is actually effective, which declarations are obsolete, and what will break if removed. In this situation, “it eventually worked” is not equivalent to “the fix has been verified.”

This paper proposes **Structural Patch Protocol (SPP)** as a practical protocol for addressing this problem. SPP is a lightweight quality-control protocol for controlling AI-driven modification of existing artifacts as evidence-first modification rather than inference-first generation. Through target locking, actual artifact inspection, minimal Probe Patch application, effect verification, failed patch withdrawal, verified merge, Cleanup Gate, and Stop Rule, SPP turns AI-driven modification quality into an externally checkable procedure.

The core flow of SPP is as follows:

```text
Target Lock
→ Actual Artifact First
→ Probe Patch
→ Verify
→ Failed Patch Withdrawal / Verified Merge
→ Cleanup
→ Stop
```

First, the **Actual Artifact First Gate** requires an AI agent to inspect the actual artifact before modifying it. Design notes, previous sprint reports, naming conventions, and common implementation patterns are not evidence. For a database, the agent must inspect the actual database file, tables, and columns. For an API, the agent must inspect the actual response, schema, and version. For a UI, the agent must inspect the DOM, selectors, and rendered behavior. For files, the agent must inspect actual file names and locations. In SPP, the basis for modification is not inference, but observed artifacts.

Second, the **Probe Patch** requires the agent to apply one minimal change to a representative target before making broad replacements. A Probe Patch must be small enough to revert, inspect, and explain. The important point is that a Probe Patch is not a mechanism for accumulating trial-and-error layers. It is a calibration step before integration.

Third, the **Failed Patch Withdrawal Rule** requires that if a Probe Patch does not produce the expected effect, the agent must withdraw or explicitly resolve that failed patch before applying another one. SPP prohibits stacking patches until something appears to work. A successful result after stacked patches is not considered a verified fix.

Fourth, **Verified Merge** permits only transformations that have been verified through a Probe Patch to be applied to structurally matching targets. Similarity alone is not sufficient. The before-and-after transformation pattern must be clear, verified, and within the defined target scope.

Fifth, the **Cleanup Gate** requires the agent to remove obsolete, overridden, ineffective, or unexplained patch layers before stopping. In SPP, cleanup is not mere deletion. Cleanup is the process of rediscovering the currently effective structure and reconstructing it into canonical form.

This paper uses two motivating observations to explain the design of SPP. The first is a case where database schemas and column names were inferred before actual artifact inspection, leading to implementation based on structures that did not exist. This case motivates the Actual Artifact First Gate. The second is a case from AI-assisted UI development where local CSS, JavaScript, and HTML patches accumulated, making it difficult to determine which declarations or functions controlled the final UI. This case motivates Failed Patch Withdrawal and Cleanup Gate.

SPP has three differentiating design points. The first is **evidence-first modification**: SPP requires an AI agent to inspect the actual artifact before modifying it, reducing inference-based erroneous changes. The second is **prohibition of unverified patch stacking**: SPP does not reject fast iteration itself, but it prohibits leaving ineffective patches in place while applying additional patches. The third is **integration of verified transformations only**: SPP allows locally verified modifications to be applied only to structurally matching targets, preventing broad replacement based on superficial similarity.

In relation to SAI-Protocol, SAI defines “what to touch” as a structural address, while SPP defines “how to safely touch the locked target.” SAI provides an external anchor for target identification, and SPP controls the modification process after the target has been identified. Used together, SAI and SPP provide a work-quality layer that helps prevent AI agents from touching the wrong target or degrading artifacts through inference-based changes and stacked patches.

SPP is also **domain-neutral**. It can be applied to code modification, database schema reference, API integration, JSON/CSV schema handling, UI/DOM/CSS modification, configuration files, prompts, documentation, generated outputs, and design assets. SPP is not a code-only style guide. It is a quality-control protocol for AI-driven modification work.

This protocol also makes clear what it does not claim. SPP is not a replacement for tests, code review, or version control. It does not provide formal verification or security guarantees. It is not a prompt template or prompt-engineering technique. SPP defines an externally checkable procedure for target confirmation, actual artifact inspection, verification, withdrawal, integration, cleanup, and stopping when AI agents modify existing artifacts.

The contributions of this paper are threefold. First, it formulates inference-based erroneous patches by AI agents as Actual Artifact Violation and presents a change-management protocol that requires actual artifact inspection as a mandatory gate. Second, it formulates Invalid Patch Stacking, where failed Probe Patches are not withdrawn before additional patches are applied, and introduces the Failed Patch Withdrawal Rule as a core mechanism. Third, it positions Cleanup not as mere deletion, but as reconstruction of the currently effective structure into canonical form, thereby presenting SPP as a practical protocol for preserving work quality in AI-assisted development.

The specification (`spec/Structural-Patch-Protocol.md`) and the initial use case (`use-cases/code/AI-Code-Patch-Protocol.md`) are published under the MIT License.

---

## Keywords

Structural Patch Protocol, AI-assisted development, AI agents, inference-based erroneous patch, Actual Artifact First, Probe Patch, Failed Patch Withdrawal, Invalid Patch Stacking, Cleanup Gate, vibe coding, change management, quality-control protocol

---

## References and Related Materials

- Petke, J. et al. “The Patch Overfitting Problem in Automated Program Repair.” FSE 2024.  
  https://research.monash.edu/en/publications/the-patch-overfitting-problem-in-automated-program-repair-practic/

- Zhang, Z. et al. “LLM Hallucinations in Practical Code Generation.” arXiv, 2024.  
  https://arxiv.org/html/2409.20550v1

- Jimenez, C. E. et al. “SWE-bench: Can Language Models Resolve Real-World GitHub Issues?” ICLR 2024.  
  https://arxiv.org/abs/2310.06770

- SAI-Protocol repository.  
  https://github.com/StateToolsLab/sai-protocol

---

## Formal Paper

This summary is a Working Paper. The formal preprint version will be published with a DOI after submission to Preprints.org or an equivalent preprint server.

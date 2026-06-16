# ADR-006a: Choice of GPT-4o as the Primary Generation and Reasoning Model

## Status

Accepted (Revised — narrowed scope from original ADR-006, split into 006a/006b/006c)

## Date

2026-06-16

## Context

The original ADR-006 conflated three distinct model decisions — generation/reasoning, embeddings, and high-volume triage classification — under a single foundation model choice. These have materially different decision drivers (reasoning depth vs. retrieval quality vs. latency/cost at volume) and conflating them weakened the documented rationale for each. This ADR narrows scope to the generation/reasoning model only: the model used for case narrative drafting, evidence summarization, SAR generation, regulatory analysis, and multi-step agent reasoning within the Case Correlation, Regulatory, and SAR Generation agents.

## Decision Drivers

* Reasoning quality on multi-step, evidence-grounded tasks
* Structured output reliability (JSON schema adherence for downstream parsing)
* Latency for interactive (human-reviewed) workflows
* Cost at the volume of escalated cases (not all alerts — see ADR-006c for triage volume)
* Azure-native deployment and governance support

## Options Considered

### Option 1: GPT-4.1

Pros: strongest reasoning and instruction-following of the Azure OpenAI catalog options evaluated; strong structured output fidelity.
Cons: higher cost and latency than GPT-4o; unnecessary depth for the bulk of generation tasks.
Why Rejected as primary: reserved instead as a selective escalation model for the highest-complexity cases (see Decision).

### Option 2: Claude Sonnet (Azure AI Foundry Model Catalog)

Pros: strong document understanding and large context window, useful for long regulatory text and multi-document case files.
Cons: less standardized internal tooling/patterns for a Microsoft-first architecture; introduces a second model family to govern, version, and monitor for marginal benefit at this stage.
Why Rejected: the operational cost of governing two model families (separate eval baselines, separate drift monitoring, separate prompt formats) outweighs the document-understanding benefit at this project's scale.

### Option 3: GPT-4o ← Chosen

Pros: strong reasoning quality at meaningfully lower cost and latency than GPT-4.1; native Azure OpenAI integration; multimodal support if document image evidence is ever incorporated; broad ecosystem familiarity, which matters for the platform's explainability and onboarding story.
Cons: marginally lower reasoning ceiling than GPT-4.1 on the hardest cases.
Tradeoffs Accepted: accept a small reasoning-quality gap on edge cases in exchange for cost and latency suitable for the platform's full generation workload, with GPT-4.1 available as a named escalation path rather than a parallel default.

## Decision

GPT-4o is the default generation/reasoning model for all AFCIP agent narrative generation, evidence summarization, SAR drafting, and regulatory analysis. GPT-4.1 is escalated to selectively, gated by an explicit complexity flag set by the orchestrator (e.g., cases with conflicting evidence across more than N sources, or cases the Case Correlation Agent marks low-confidence), not used as a parallel default. This escalation path is itself logged to the audit store (see ADR-009/ADR-010) so model selection is traceable per case.

## Consequences

### Positive

Lower baseline inference cost; consistent default behavior simplifies eval and monitoring baselines; escalation path preserves access to stronger reasoning without paying for it on every case.

### Negative

Escalation logic adds a small amount of orchestration complexity; the complexity-flag heuristic itself will need tuning and is a candidate for its own eval metric.

### Risks

Risk: escalation heuristic mis-fires (over- or under-escalates), masking the actual cost/quality tradeoff.

Mitigation: track escalation rate and post-hoc human agreement rate with the escalation decision as a Phase 4 monitoring metric.

## Compliance Considerations

### FINTRAC / PCMLTFA

Supports explainable, grounded outputs; escalation logging strengthens traceability of which model produced which case output.

### OSFI E-23

Per-case model selection logging supports model governance and validation requirements (see ADR-010).

### Data Residency Requirements

Inference endpoints deployed within approved Azure regions.

### PIPEDA

Supports encryption, RBAC, and managed identity access to the inference endpoint.

## Review Date

2027-06-16

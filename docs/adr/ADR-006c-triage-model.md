# ADR-006c: Choice of GPT-4o-mini for High-Volume Alert Triage Classification

## Status

Accepted (New — split from original ADR-006)

## Date

2026-06-16

## Context

The Alert Triage Agent processes every incoming alert before any case is escalated to the full multi-agent investigation pipeline. This is, by volume, the highest-frequency inference workload on the platform — every alert is triaged, while only a fraction are escalated to narrative generation, regulatory analysis, or SAR drafting. Running GPT-4o (ADR-006a) on every triage call would make triage the dominant cost line despite being the least complex reasoning task in the pipeline. This was not addressed in the original ADR-006, which named a single model for all workloads.

## Decision Drivers

* Cost at full alert volume (not escalated-case volume)
* Latency, since triage sits on the critical path before any human sees an alert
* Classification accuracy sufficient to make a reliable escalate/suppress decision
* Asymmetric error cost: a false suppress (missed true positive) is materially worse than a false escalate (extra review work), so the model choice must be paired with a conservative decision threshold, not just a cheap model

## Options Considered

### Option 1: GPT-4o (same as generation model)

Pros: one fewer model to govern; consistent reasoning quality.
Cons: cost scales with full alert volume rather than escalated-case volume, which inverts the platform's intended cost structure (the original problem statement is reducing analyst review *cost*, so the triage layer itself becoming the dominant spend undermines the business case).
Why Rejected: cost profile mismatched to the actual workload shape.

### Option 2: A dedicated fine-tuned classifier (non-LLM or small fine-tuned model)

Pros: potentially lowest cost and latency at true production volume; could outperform a general LLM on a narrowly defined classification task given enough labeled data.
Cons: requires a labeled training set the project does not yet have (see ADR-011, golden-set bootstrapping); adds a second model lifecycle to manage (training, retraining, drift) during a 30-day build.
Why Rejected for now: the labeled data dependency makes this infeasible within the build window; revisit once the golden-set/eval dataset (ADR-011) has matured enough to support supervised fine-tuning, and document this as a Phase 4+ candidate.

### Option 3: GPT-4o-mini ← Chosen

Pros: substantially lower cost and latency than GPT-4o, suitable for full-volume triage; sufficient classification quality for a first-pass escalate/suppress decision when paired with structured-output constraints and a conservative threshold; same Azure OpenAI governance surface as the generation model, avoiding a third vendor.
Cons: lower reasoning depth than GPT-4o — acceptable here because triage is a narrower decision (escalate or not) than narrative synthesis.
Tradeoffs Accepted: accept a small classification-quality gap relative to GPT-4o, mitigated by setting the escalation threshold conservatively (biased toward escalating borderline cases to a human-reviewed pipeline rather than suppressing them outright).

## Decision

GPT-4o-mini will perform first-pass alert triage classification for all incoming alerts. The triage agent outputs a structured decision (escalate / suppress / escalate-with-low-confidence) plus a rationale; suppress decisions below a defined confidence threshold are still logged and sampled for human spot-review rather than silently discarded, given the asymmetric cost of false suppressions. A dedicated fine-tuned classifier is documented as a Phase 4+ candidate once sufficient labeled triage outcomes exist.

## Consequences

### Positive

Cost structure now scales with the right variable (alert volume cheaply triaged, generation cost only on escalated cases); preserves a path to an even cheaper fine-tuned classifier later without committing to it before data exists.

### Negative

A second model tier to monitor for drift, separate from the generation model; conservative thresholding means some suppress-worthy alerts still consume a small amount of human spot-review capacity by design.

### Risks

Risk: GPT-4o-mini's classification quality degrades on novel transaction typologies not well represented in its training distribution.

Mitigation: the human spot-review sample on suppressed alerts (above) acts as an early warning signal; track suppress-then-overturned rate as a Phase 4 monitoring metric and treat a rising rate as a drift signal requiring threshold or model review.

## Compliance Considerations

### FINTRAC / PCMLTFA

Conservative suppression threshold and logged rationale on every triage decision (including suppressions) preserves the auditability of the full alert lifecycle, including alerts that did not escalate.

### OSFI E-23

Triage model performance (escalate/suppress accuracy, overturned-suppression rate) is tracked as a named model risk metric under the governance framework in ADR-010.

### Data Residency Requirements

Consistent with ADR-004/ADR-006a Azure region constraints.

### PIPEDA

No additional PII exposure; same access pattern as the generation model's inference endpoint.

## Review Date

2027-06-16

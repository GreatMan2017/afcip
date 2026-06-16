# ADR-011: Evaluation Golden-Set Bootstrapping Strategy and Cost Ceiling

## Status

Accepted (New)

## Date

2026-06-16

## Context

Phase 4 of the project plan calls for an evaluation framework built on a golden set with regression testing, and ADR-010's model risk governance framework depends on objective validation metrics — but AFCIP has no real historical SAR data, no real alert outcomes, and no existing labeled dataset to evaluate against. Separately, no ADR has established a cost ceiling despite GPT-4o/GPT-4o-mini usage (ADR-006a/006c) being identified as the dominant cost driver. Both gaps block Phase 4 work if left undecided, and the cost ceiling specifically affects build decisions as early as Phase 1 (e.g., how much synthetic data volume is generated, how often the escalation path to GPT-4.1 fires).

## Decision Drivers

* An evaluation framework is meaningless without a labeled ground truth to evaluate against
* The golden set must be defensible as realistic, not just internally consistent (i.e., it should reflect recognized AML typology categories, not ad hoc invented patterns)
* A cost ceiling must exist before Phase 2 ingestion volume and Phase 3 agent-call volume are decided, not retrofitted after spend occurs
* The bootstrapping approach must not require external data the builder doesn't have access to (real bank data, licensed datasets)

## Options Considered

### Option 1: No golden set — evaluate qualitatively by spot-checking agent outputs

Pros: no upfront labeling effort.
Cons: produces no metric for ADR-010's validation framework to reference, makes the regression-testing requirement in the original project plan unfulfillable, and gives a hiring-manager conversation nothing concrete to point to ("we checked it and it looked fine" is not an evaluation framework).
Why Rejected: defeats the purpose of having an evaluation framework at all.

### Option 2: Source a public real-world dataset (e.g., a Kaggle fraud/AML dataset) as the sole ground truth

Pros: real-world data lends some external credibility.
Cons: public fraud/AML datasets are typically transaction-classification datasets (fraud vs. not), not case-narrative or SAR-quality datasets, so they don't actually evaluate the parts of AFCIP that are hardest (narrative generation, evidence synthesis, agent reasoning quality) — only the parts that are easiest (binary classification).
Why Rejected as the sole source: useful as a secondary input for grounding the synthetic generator's realism (see Decision), but insufficient alone since it doesn't cover the platform's actual differentiating capability.

### Option 3: Hand-labeled synthetic golden set, with typology design grounded in publicly available regulatory typology guidance, built incrementally from Day 1 rather than only in Phase 4 ← Chosen

Pros: produces a ground truth tailored to exactly what AFCIP needs to evaluate (triage accuracy, narrative quality, escalation correctness, suppression overturn rate); grounding the typology categories in recognized public guidance (without reproducing that guidance's text) gives the dataset external defensibility rather than being arbitrary; building it incrementally from Day 1 (a handful of labeled cases per day as synthetic scenarios are designed) avoids a Phase-4 scramble to retroactively label months of unlabeled output.
Cons: hand-labeling is manual effort that competes with build time; the golden set will be small relative to a real institution's evaluation corpus, which is a stated scope limitation, not a hidden one.
Tradeoffs Accepted: accept a small, manually-curated golden set in exchange for having any objective ground truth at all, and accept the labeling time cost by spreading it across the full 30 days rather than concentrating it in Phase 4.

## Decision

A golden set is built incrementally starting Day 1: each synthetic AML scenario designed (structuring, layering, smurfing, and other typologies aligned to recognized public categorization) is paired with a hand-labeled expected outcome (correct triage decision, key facts the narrative should cite, correct escalation/no-escalation call). By Phase 4, this produces a regression-test corpus built from real build artifacts rather than retrofitted. This golden set is the ground truth referenced by ADR-010's validation metrics (triage overturned-suppression rate, escalation-heuristic agreement rate). A public fraud-pattern dataset may be consulted only to sanity-check that synthetic typology distributions resemble realistic proportions, not as a substitute ground truth.

Cost ceiling: a defined monthly Azure spend ceiling is set before Phase 2 begins, with an Azure Cost Management budget alert configured at Day 1 (not deferred), tiered at warning (e.g., 50% of ceiling) and hard-stop notification thresholds. Model usage is allocated against this ceiling according to the tiering already established in ADR-006a/006c: GPT-4o-mini absorbs full alert-triage volume, GPT-4o is the default for escalated-case generation, and GPT-4.1 is reserved for the explicitly gated escalation path — this tiering exists specifically so the ceiling is achievable without arbitrarily throttling functionality.

## Consequences

### Positive

ADR-010's governance framework now has an actual ground truth to validate against, closing the gap between "we have a validation metric" and "the metric measures something real"; spreading labeling effort across 30 days avoids a Phase 4 crunch; an early, explicit cost ceiling prevents a late-build discovery that spend has run ahead of the design's assumptions.

### Negative

Golden set size and diversity will be modest compared to production-scale evaluation corpora — must be stated as a scope limitation in any portfolio presentation, not implied to be production-equivalent; the cost ceiling may constrain synthetic data volume or escalation-path testing if set too conservatively.

### Risks

Risk: hand-labeled golden set reflects the builder's own assumptions about correct outcomes, with no independent check (related to the role-separation limitation in ADR-010).

Mitigation: where possible, derive expected outcomes from the typology definition itself (e.g., "this scenario is structuring by construction, therefore the correct triage decision is escalate") rather than from subjective judgment after the fact, keeping labels as objective as the synthetic design allows.

Risk: cost ceiling set without real usage data turns out to be miscalibrated (too tight or too loose) once actual Phase 2/3 volumes are known.

Mitigation: treat the Day 1 ceiling as a starting estimate, reviewed and explicitly adjusted (with rationale documented) once Phase 2 ingestion volume and Phase 3 agent-call patterns are observed — this is itself a demonstration of the cost-monitoring discipline named in the original project requirements.

## Compliance Considerations

### FINTRAC / PCMLTFA

Golden-set typology grounding in recognized categorization supports the platform's claim to reflect realistic investigative scenarios rather than arbitrary test cases.

### OSFI E-23

Directly supplies the ground truth that ADR-010's model governance framework requires to be more than a documentation exercise.

### Data Residency Requirements

No impact; golden set is synthetic data with no real customer information.

### PIPEDA

No PII exposure, since all golden-set data is synthetic by construction — worth stating explicitly as a privacy-by-design benefit of not using real data.

## Review Date

2027-06-16

# ADR-010: Model Risk Governance Framework (SR 11-7 / OSFI E-23 Aligned)

## Status

Accepted (New)

## Date

2026-06-16

## Context

Every prior ADR's Compliance Considerations section cites OSFI E-23 as a satisfied or supported requirement, but no ADR has actually defined what model governance process AFCIP follows — what gets validated, by what criteria, on what cadence, and what triggers revalidation. Citing a regulation in passing across six ADRs without ever specifying the governance process itself is a credibility gap: it reads as compliance-by-assertion rather than compliance-by-design. This ADR defines the actual model risk governance framework that the other ADRs' compliance sections are implicitly relying on.

## Decision Drivers

* Every model used in the platform (GPT-4o, GPT-4o-mini, text-embedding-3-large, and any future fine-tuned classifier) needs a defined validation lifecycle, not just a deployment decision
* The framework must be proportionate to a solo-build portfolio project while still being structurally identical in kind (not just in name) to what a real institution's model risk function would require
* Validation criteria must be tied to concrete metrics already named in other ADRs (triage overturned-suppression rate from ADR-006c, escalation heuristic agreement rate from ADR-006a) rather than invented separately
* The framework must specify what triggers revalidation (model version change, drift detection, performance degradation) rather than treating validation as a one-time gate

## Options Considered

### Option 1: Informal review — document model choices in ADRs (as already done) with no further process

Pros: zero additional process overhead.
Cons: this is what every other ADR has implicitly assumed satisfies OSFI E-23, and it does not — a regulator or a model risk reviewer would expect an ongoing validation and monitoring process, not a one-time architecture decision document.
Why Rejected: insufficient to support the compliance claims already made elsewhere in the ADR set; leaves those claims unsubstantiated.

### Option 2: Full SR 11-7-style three-lines-of-defense model risk function (independent validation team, separate from development)

Pros: matches real institutional practice exactly.
Cons: requires multiple distinct roles (developer, independent validator, audit) that don't exist in a one-person build; adopting the full organizational structure verbatim would be performative rather than genuine.
Why Rejected as a literal implementation, retained as a structural template: the three-lines concept (development, independent validation, audit/oversight) is preserved in spirit — see Decision — by separating the validation *criteria and evidence* from the development decision, even though one person executes both roles. This is documented explicitly as a known scope limitation rather than concealed.

### Option 3: Lightweight model risk governance process scoped to a solo build, structurally aligned with SR 11-7 / OSFI E-23 categories (conceptual soundness, ongoing monitoring, outcomes analysis) but executed by one person with explicit role-separation documentation ← Chosen

Pros: gives every model a defined validation record (what it does, why it was chosen, what metric proves it's working, what would trigger re-review) without requiring an organizational structure that doesn't exist; makes the compliance claims in ADR-001 through ADR-009 actually substantiated by something concrete; the explicit acknowledgment of the role-separation limitation is itself a stronger signal to an evaluator than silently implying full institutional rigor.
Cons: a single person executing "development" and "independent validation" is a genuine limitation, not solved by documentation alone.
Tradeoffs Accepted: accept that true independence between development and validation isn't achievable in a solo build, and address this by making the validation criteria objective and metric-based (so the validation step is a check against a predefined threshold, not a subjective self-review) rather than claiming an independence the project doesn't have.

## Decision

Each model in the platform (GPT-4o per ADR-006a, GPT-4o-mini per ADR-006c, text-embedding-3-large per ADR-006b, and any future fine-tuned classifier) has a model risk record containing: intended use and scope (what the model is and is not approved to do — e.g., GPT-4o-mini is approved for triage classification only, not for SAR narrative drafting); the validation metric and threshold defined at adoption (e.g., triage overturned-suppression rate from ADR-006c, escalation-heuristic human-agreement rate from ADR-006a); the monitoring cadence (continuous logging, periodic — e.g., weekly during active build — review against threshold); and explicit revalidation triggers (model version change by Microsoft, observed metric breach, or a defined calendar review date). The conceptual soundness review (does this model choice make sense for this use case) is documented in the relevant ADR-006 variant; the ongoing monitoring and outcomes analysis is what this ADR adds as a continuing process rather than a one-time decision.

The solo-build role-separation limitation is documented openly: the same person performs model selection and validation review. This is mitigated, not eliminated, by making validation criteria objective and pre-committed (defined at adoption time, before observing results) rather than retrospectively justified.

## Consequences

### Positive

The OSFI E-23 / model governance claims made throughout ADR-001–009 now point to an actual process rather than an assertion; metrics already named in other ADRs are unified into a single governance record instead of being scattered; the explicit limitation disclosure is itself a demonstration of mature risk-thinking, which is a stronger interview signal than an inflated compliance claim.

### Negative

Single-person execution of both development and validation is a real, undissolved limitation; this framework adds documentation overhead that must be kept current as models change.

### Risks

Risk: validation criteria are set generously enough at adoption time to be trivially passed, defeating the purpose.

Mitigation: define thresholds with reference to a stated rationale at adoption (e.g., "overturned-suppression rate below X% based on Y reasoning"), not retrofitted after observing results — and call this out explicitly as the discipline this framework requires to be meaningful rather than performative.

Risk: revalidation triggers are defined but never actually exercised during the 30-day build window, leaving the framework untested.

Mitigation: deliberately force at least one revalidation cycle during the build (e.g., the GPT-4o-mini triage threshold review in Phase 4) so the process is demonstrated, not just documented.

## Compliance Considerations

### FINTRAC / PCMLTFA

Provides the substantiating process behind the auditability and traceability claims made in every prior ADR's compliance section.

### OSFI E-23

This is the framework that the phrase "supports model governance" in ADR-001 through ADR-006c was previously referring to without defining; it now exists as a concrete artifact.

### Data Residency Requirements

No additional impact; this is a process control, not a data placement decision.

### PIPEDA

No additional PII exposure; governance records reference model performance metrics, not individual case PII directly.

## Review Date

2027-06-16, and upon any revalidation trigger event prior to that date

# ADR-009: Human-in-the-Loop Gating Mechanism for SAR Generation and Escalated Cases

## Status

Accepted (New)

## Date

2026-06-16

## Context

The platform's core design principles state that human-in-the-loop is required for SAR generation and that every AI decision must be explainable with evidence citations. Neither prior ADR nor the architecture overview specifies *where* in the pipeline this gate physically lives, what state a case sits in while awaiting human review, or how a human override is recorded. Without this decision, "human-in-the-loop" remains a stated principle rather than an architectural control, which is a meaningful gap for a platform whose entire credibility rests on auditability.

## Decision Drivers

* The gate must be a hard architectural stop, not a convention agents are merely instructed to follow (per the design principle that agents are tool-driven, not trusted to self-police)
* Every override (approve, reject, edit-and-approve) must be logged with the human identity, timestamp, and the original AI-proposed output preserved unmodified alongside the edited version
* The case must have a well-defined state machine so a case can't silently sit in limbo or be double-processed
* The mechanism must work with the chosen orchestration framework (Semantic Kernel) and hosting platform (Container Apps, per revised ADR-005) without requiring a separate workflow engine if avoidable

## Options Considered

### Option 1: Soft gate — agent prompt instructs the SAR Generation Agent to "request human approval" as a step in its reasoning

Pros: no additional infrastructure.
Cons: this is precisely the "trust the agent to self-police" pattern the design principles explicitly reject; an LLM instructed to pause is not an enforced control, it's a suggestion the model could fail to follow, especially under prompt drift or edge-case inputs.
Why Rejected: incompatible with the stated "tool-driven, not hallucination-driven" and "no black-box outputs" principles; a regulated SAR workflow needs a deterministic stop, not a probabilistic one.

### Option 2: External workflow engine (e.g., Azure Logic Apps or a dedicated durable-workflow service) managing the full case lifecycle

Pros: purpose-built for human-approval state machines; strong built-in audit trail.
Cons: introduces a second orchestration paradigm alongside Semantic Kernel, duplicating responsibility for "what happens next in a case" across two systems — risks the two falling out of sync on case state.
Why Rejected: unnecessary duplication given Semantic Kernel's orchestrator already owns case sequencing; the gate can be implemented as an explicit state in the existing orchestration rather than handed to a second system.

### Option 3: Explicit case state machine enforced by the orchestrator, with the SAR Generation Agent's output written to a pending-review store that no downstream action can read from until a human decision record exists ← Chosen

Pros: the orchestrator (already the single point of sequencing, per ADR-003's hub-and-spoke pattern) owns the gate, so there's one source of truth for case state; the mechanism is a data/permissions control (the "filed" action literally cannot execute without a decision record existing), not a prompt instruction, satisfying the hard-stop requirement; the human-proposed-vs-final distinction is preserved by storing both versions, supporting explainability.
Cons: requires defining the case state machine explicitly now, which is more upfront design work than the soft-gate option.
Tradeoffs Accepted: accept the upfront cost of defining and enforcing an explicit state machine in exchange for a control that is structurally guaranteed rather than instruction-dependent.

## Decision

Every case follows an explicit state machine: `triaged → escalated → under_investigation → pending_human_review → (approved | rejected | edited_and_approved) → filed | closed`. The transition from `pending_human_review` to any terminal state requires a decision record (human identity, timestamp, decision type, and — if edited — a diff between the AI-proposed narrative and the human-edited final version) to exist before the orchestrator permits the next step (e.g., SAR filing or case closure) to execute. This is enforced as a permission/data-availability check in the orchestrator's state transition logic, not as an instruction to the SAR Generation Agent. The AI-proposed output is never overwritten in place; the human-edited version is stored as a separate, linked record, so the original AI output remains available for the eval/regression dataset (see ADR-011) regardless of what the human ultimately decided.

Maximum retry/escalation handling: if the Compliance/SAR Generation Agent's output is rejected by the human reviewer, the case returns to `under_investigation` with the rejection reason attached as additional context for the next generation attempt, capped at a defined retry limit before mandatory escalation to a senior human reviewer role (modeled even if only one reviewer role exists in this portfolio build, to demonstrate the escalation-path design).

## Consequences

### Positive

Human-in-the-loop becomes a verifiable architectural property (a query against the audit store can prove no case skipped review) rather than a claim resting on prompt wording; preserves both AI-proposed and human-final versions, directly supporting both the explainability principle and the eval/regression dataset bootstrapping problem noted in ADR-011.

### Negative

Requires the orchestrator to carry case-state logic in addition to agent-sequencing logic, slightly increasing its responsibility surface — acceptable since this is still one system rather than two (per the Option 2 rejection).

### Risks

Risk: a bug in the state-transition enforcement code accidentally permits a transition without a decision record, defeating the purpose of the gate.

Mitigation: the "filed" action's data access itself (not just application logic) should require the decision record's existence — e.g., the filing step queries for the decision record and fails closed if absent, rather than relying solely on the orchestrator's control flow to never call it incorrectly. This is a defense-in-depth point worth highlighting in any architecture review.

Risk: retry loop on repeated rejections could cycle indefinitely without the escalation cap.

Mitigation: explicit retry limit defined above, with mandatory senior-reviewer escalation as the terminal path.

## Compliance Considerations

### FINTRAC / PCMLTFA

Directly implements the regulatory expectation that a human, not an algorithm, makes the SAR filing decision; decision records constitute the evidentiary trail regulators would expect to review.

### OSFI E-23

The state machine and fail-closed filing check are concrete operational governance controls suitable for citing in a model risk management review.

### Data Residency Requirements

Decision records stored in the same approved-region data store as other case data (ADR-004).

### PIPEDA

Human reviewer identity is logged per decision; access to decision records scoped via the RBAC model in ADR-007.

## Review Date

2027-06-16

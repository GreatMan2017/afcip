# ADR-005: Choice of Azure Container Apps over AKS and Azure Functions for Agent Hosting (Revised)

## Status

Accepted (Revised — supersedes prior AKS decision)

## Date

2026-06-16

## Context

AFCIP requires hosting multiple AI agents responsible for:

* Alert triage
* Transaction analysis
* Customer risk assessment
* Regulatory research
* Case correlation
* SAR generation

The platform must support long-running workflows, agent orchestration, event-driven processing, horizontal scaling, enterprise networking, secure service-to-service communication, and production-grade observability.

This decision was originally made in favor of AKS based on enterprise scalability and networking control criteria alone, without reference to the project's actual delivery constraints: a single engineer building a 30-day portfolio system with a defined cost ceiling and no existing Kubernetes operational capacity. Revisiting the decision against those constraints changes the outcome.

The hosting platform must still support a credible path to enterprise production scale, since the project's purpose is to demonstrate that judgment — but the *built* artifact and the *documented future-state* artifact do not have to be the same thing.

## Decision Drivers

* Solo-engineer build capacity within a 30-day window
* Defined cost ceiling (see ADR-011 budget note)
* Event-driven scaling trigger already present (Event Hub queue depth)
* Operational maturity required to run the platform unattended during the build
* Credibility of the "how I'd scale this to enterprise" interview narrative
* Avoiding over-engineering as a negative signal to evaluators

## Options Considered

### Option 1: Azure Functions

Description:
Serverless event-driven compute platform.

Pros:

* Lowest operational overhead
* Native Event Hub trigger integration
* Fast to stand up for the enrichment/ingestion layer

Cons:

* Execution duration limits poorly suited to multi-step agent reasoning chains
* Weak fit for stateful, long-running orchestration across five agents
* Debugging distributed agent call chains is difficult without a hosting layer that supports persistent process state

Why Rejected (for the orchestration layer):

Functions remain the correct choice for the enrichment step in the ingestion pipeline (see ADR-008), but are insufficient as the host for the Semantic Kernel orchestrator and agent runtime, which need longer-lived execution contexts and easier multi-agent debugging.

### Option 2: Azure Kubernetes Service (AKS)

Description:
Managed Kubernetes platform for containerized applications.

Pros:

* Maximum scalability and networking control
* Service mesh support, fine-grained traffic policy
* Industry-standard architecture for large-scale production AI platforms
* Strongest signal of "enterprise production readiness" on paper

Cons:

* Cluster lifecycle management (node pools, upgrades, ingress controllers) consumes build time disproportionate to project scale
* No existing operational capacity for unattended cluster management as a solo engineer
* Adds infrastructure surface area that must itself be secured, monitored, and explained — without adding agent capability
* Risk of presenting as over-engineered relative to actual load (a handful of synthetic-data agents, not bank-scale transaction volume)

Why Rejected (for the build, not for the roadmap):

AKS is the right answer at genuine enterprise scale, but choosing it here optimizes for resume keyword presence over demonstrated engineering judgment. A staff-level evaluator is more likely to probe "why AKS for this load" critically than to reward it by default. AKS is retained as the explicit Phase 3+ migration path (see Consequences) rather than discarded.

### Option 3: Azure Container Apps ← Chosen

Description:
Serverless container hosting platform built on Kubernetes primitives (KEDA, Dapr, Envoy) without exposing cluster management.

Pros:

* KEDA-based autoscaling directly on Event Hub queue depth — matches the platform's actual scaling trigger with no extra configuration
* Managed ingress, managed certificates, VNet integration via private endpoints — meets the zero-trust networking requirement without a self-managed control plane
* Supports long-running revisions suitable for the Semantic Kernel orchestrator process
* Dapr sidecar support gives a credible path to service-to-service calling between agents if needed without hand-rolling it
* Operationally sized correctly for a solo build within a 30-day window and a defined budget

Cons:

* Less granular network policy control than AKS (no custom CNI, no NetworkPolicy objects)
* Some advanced Kubernetes-native tooling (service mesh, custom operators) unavailable
* Ceiling exists if true enterprise transaction volume were ever real — addressed via the documented migration path, not by building past current scope

Tradeoffs Accepted:

Accept reduced networking granularity and a future migration step in exchange for a hosting platform whose operational burden matches actual build capacity, while still meeting private-networking and managed-identity requirements today.

## Decision

Azure Container Apps will serve as the primary compute platform for hosting the AFCIP Semantic Kernel orchestrator, the five investigation agents, and supporting APIs through Phase 1–3 of the build. Azure Functions will host the event-driven enrichment step (per ADR-008). AKS is documented as the Phase 4 / enterprise-scale migration target, not built in the current scope.

This reverses the original ADR-005 decision. The original decision is preserved below for audit continuity.

## Consequences

### Positive

* Build effort concentrated on agent logic and RAG quality rather than cluster operations
* Autoscaling trigger matches the real ingestion pattern with minimal configuration
* Still satisfies private endpoint / managed identity / zero-trust requirements
* Produces a stronger interview narrative: a deliberate, constraint-aware infrastructure decision rather than a default to the most enterprise-sounding option
* Preserves a credible, explicitly designed AKS migration path for the "how would you scale this" conversation

### Negative

* Networking control ceiling lower than AKS — acceptable at current scale, would need revisiting if entity-to-entity traffic volume grew materially
* A second migration ADR will eventually be required if the platform is ever extended past portfolio scope

### Risks

Risk:
Container Apps' networking ceiling is reached if synthetic load is scaled up significantly during testing.

Mitigation:
Document the AKS migration trigger conditions explicitly (see Phase 3 roadmap): sustained concurrent case volume beyond a defined threshold, or a requirement for custom network policy / service mesh features Container Apps cannot provide.

Risk (carried over from original AKS decision, now reframed):
Future production deployment would need to revisit this decision against real transaction volume, which this portfolio project cannot generate.

Mitigation:
Maintain the AKS option as a documented ADR addendum (Phase 4) including the specific metrics that would trigger migration, so the decision-making process — not just the destination — is demonstrable.

## Original Decision (Superseded)

Azure Kubernetes Service (AKS) was originally selected on enterprise-scalability grounds without weighing solo-build capacity or an established cost ceiling. That decision is superseded by this revision. The original options analysis (Functions and Container Apps as rejected alternatives) is retained in version control history for audit traceability, consistent with the platform's own auditability principle.

## Compliance Considerations

### FINTRAC / PCMLTFA

Container Apps' managed identity and private endpoint support preserves secure handling of investigation workflows equivalent to the original AKS decision.

### OSFI E-23

Operational resilience and observability requirements are met via Container Apps' built-in revision management, health probes, and Azure Monitor integration; documented AKS migration path addresses long-term operational governance maturity.

### Data Residency Requirements

Container Apps environment deployed within approved Canadian Azure regions, consistent with ADR-004.

### PIPEDA

Supports private networking, managed identity, encryption in transit/at rest, and least-privilege RBAC equivalent to the original decision's compliance posture.

## Review Date

2027-06-16 (or upon any Phase 3 migration trigger condition being met, whichever is sooner)

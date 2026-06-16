# ADR-007: Identity and Secrets Model — Managed Identity First, Key Vault for Residual Secrets

## Status

Accepted (New)

## Date

2026-06-16

## Context

ADR-001 through ADR-006c each reference "managed identities," "workload identities," and "RBAC" in their Compliance Considerations sections, but no ADR has actually decided the identity and secrets pattern itself. This is a gap: a zero-trust design principle was stated up front in the project's core design principles, but never architecturally specified. Every service-to-service call in AFCIP — Functions to ADLS, Container Apps to Azure OpenAI, the orchestrator to AI Search, agents to Key Vault — needs a consistent, decided authentication pattern before Phase 2 build work begins, since retrofitting identity later is expensive and error-prone.

## Decision Drivers

* Zero-trust principle stated in core design principles (no implicit trust between components)
* Elimination of long-lived credentials wherever Azure supports identity-based auth natively
* Auditability: every access must be attributable to a specific identity, not a shared key
* Operational simplicity for a solo build (credential rotation overhead must be minimized)
* Coverage for the one category of secret that cannot avoid being a secret: external API keys for any third-party data source (e.g., a sanctions data feed not available via managed identity)

## Options Considered

### Option 1: Shared connection strings / access keys stored in app settings

Pros: simplest to wire up initially.
Cons: long-lived credentials, no per-identity audit trail, manual rotation, direct violation of the stated zero-trust principle.
Why Rejected: incompatible with the project's own design principles and with OSFI E-23 operational governance expectations.

### Option 2: Service principals with client secrets for every service

Pros: more granular than shared keys, supported everywhere.
Cons: still a credential that must be stored, rotated, and protected; Azure's managed identity model exists specifically to remove this category of risk where supported.
Why Rejected: managed identity is available for every first-party Azure-to-Azure call AFCIP makes; service principals would be a strictly worse choice for those paths with no offsetting benefit.

### Option 3: System-assigned managed identity per compute resource, scoped RBAC roles, Key Vault reserved for non-Azure secrets only ← Chosen

Pros: no credential to store or rotate for any Azure-to-Azure call (Functions→ADLS, Container Apps→Azure OpenAI, Container Apps→AI Search, orchestrator→Key Vault for the residual secrets); every access attributable to a specific resource identity in Azure AD sign-in logs; least-privilege RBAC scoped per resource (e.g., a Function's identity gets Storage Blob Data Contributor on its specific container, not the whole storage account); Key Vault still used, but only for the narrow case of external (non-Azure) API keys.
Cons: system-assigned identities are tied to resource lifecycle (deleting the resource deletes the identity), which requires care if a resource is ever recreated rather than updated in place.
Tradeoffs Accepted: accept the lifecycle coupling of system-assigned identities in exchange for eliminating essentially all stored credentials in the platform.

## Decision

Every Azure compute resource in AFCIP (Functions, Container Apps) uses a system-assigned managed identity. RBAC role assignments are scoped to the narrowest resource the identity needs (specific storage container, specific Key Vault, specific AI Search index where supported) rather than subscription- or resource-group-wide roles. Azure Key Vault is used exclusively for secrets that cannot be replaced by managed identity — concretely, any external (non-Azure) API key such as a third-party sanctions data feed, if one is used instead of the static-snapshot approach in ADR-008. No connection strings or access keys are stored in application configuration anywhere in the platform.

## Consequences

### Positive

No long-lived credential sprawl; every access is attributable in Azure AD audit logs, directly strengthening the auditability claims made throughout ADR-001–006c; rotation burden eliminated for the large majority of service-to-service calls.

### Negative

Local development requires a workaround (e.g., `DefaultAzureCredential` falling back to developer Azure CLI login) since managed identity doesn't exist outside Azure — this needs a documented local-dev pattern so it doesn't become an ad hoc exception that erodes the principle.

### Risks

Risk: a future integration requires a service that doesn't support managed identity, creating pressure to fall back to a stored credential.

Mitigation: any such exception must be documented as an ADR addendum with explicit justification, not silently added to app settings — keeps the zero-trust principle auditable even when an exception is made.

Risk: system-assigned identity lifecycle coupling causes a permissions gap if a resource is deleted and recreated during iteration.

Mitigation: document the RBAC re-assignment step as part of the resource provisioning runbook (Infrastructure as Code should handle this automatically if role assignments are defined alongside the resource, not applied manually after the fact).

## Compliance Considerations

### FINTRAC / PCMLTFA

Per-identity audit trail strengthens evidentiary traceability of every system access touching investigation data.

### OSFI E-23

Directly supports operational resilience and access governance expectations; eliminates a class of operational risk (credential leakage) that would otherwise need separate mitigation.

### Data Residency Requirements

No impact; identity and RBAC are control-plane constructs independent of data region.

### PIPEDA

Least-privilege, per-resource scoped access directly supports PIPEDA's access-control and accountability principles for any PII touched by the platform.

## Review Date

2027-06-16

# ADR-006b: Choice of Azure OpenAI text-embedding-3-large for Retrieval Embeddings

## Status

Accepted (New — split from original ADR-006)

## Date

2026-06-16

## Context

The retrieval layer (ADR-002, Azure AI Search) requires an embedding model to support vector and hybrid search across the case-narrative/precedent index and the rules/sanctions index. Embedding quality directly determines retrieval relevance, which the original ADR-002 already identifies as the primary lever against hallucination. This decision was not previously made explicitly — ADR-006 named GPT-4o as "the foundation model" without distinguishing the embedding model needed for retrieval from the generation model needed for drafting.

## Decision Drivers

* Retrieval relevance quality (directly tied to hallucination risk per ADR-002)
* Consistency of embedding space across both indices (narrative and rules)
* Dimensionality and cost at index scale
* Azure-native integration (avoiding a second vendor for embeddings alone)
* Re-embedding cost if the model is changed later

## Options Considered

### Option 1: text-embedding-3-small

Pros: lower cost and storage footprint per vector.
Cons: lower retrieval quality on the kind of nuanced regulatory and narrative text AFCIP indexes, where near-duplicate typology descriptions need fine-grained separation.
Why Rejected: the cost saving is small relative to the platform's overall token spend (generation dominates cost, not embeddings at this index scale), while retrieval quality directly affects the platform's core hallucination-mitigation claim.

### Option 2: Open-source embedding model (self-hosted)

Pros: no per-call cost, full control.
Cons: requires hosting and operating an inference endpoint, adds an operational component with no Azure-native governance integration, and reintroduces the multi-vendor complexity ADR-001 explicitly rejected.
Why Rejected: operational burden disproportionate to benefit for a solo build; conflicts with the Azure-first architecture established across ADR-001 through ADR-005.

### Option 3: text-embedding-3-large (Azure OpenAI) ← Chosen

Pros: stronger semantic separation for nuanced regulatory/typology text; native Azure OpenAI integration alongside the generation model, simplifying governance and monitoring under one provider; consistent embedding space across both AI Search indices.
Cons: higher per-call and storage cost than the small variant; any future change of embedding model requires a full re-embed of both indices.
Tradeoffs Accepted: accept higher embedding cost in exchange for retrieval quality that materially affects the platform's central explainability/hallucination-mitigation claim.

## Decision

text-embedding-3-large will be used for all vector embeddings across both the case-narrative/precedent index and the rules/sanctions index. The same model will be used for both indices to keep the embedding space consistent and avoid needing cross-model relevance calibration.

## Consequences

### Positive

Consistent retrieval quality across both indices; single embedding provider simplifies cost monitoring and drift tracking; aligns with the Azure-first governance posture.

### Negative

Higher embedding cost at index growth scale; locked into a re-embedding cost if a future model change is warranted.

### Risks

Risk: embedding model deprecation or version change by Microsoft requiring a full re-embed.

Mitigation: store raw source text alongside vectors (not vectors alone) in the curated zone so re-embedding is a recomputation, not a data-loss event; track this explicitly as an operational runbook item.

## Compliance Considerations

### FINTRAC / PCMLTFA

Embeddings are derived from data already classified per ADR-004's zone structure; no new compliance surface beyond the source data's existing classification.

### OSFI E-23

Embedding model versioning is tracked as part of model governance per ADR-010.

### Data Residency Requirements

Embedding inference calls remain within approved Azure regions, consistent with ADR-004.

### PIPEDA

No additional PII exposure beyond what is already present in source documents; access controls inherited from the AI Search index RBAC (ADR-002).

## Review Date

2027-06-16

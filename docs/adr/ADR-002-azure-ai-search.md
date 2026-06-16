# ADR-002: Choice of Azure AI Search as the Vector and Hybrid Search Engine

## Status

Accepted

## Date

2026-06-16

## Context

AFCIP requires retrieval of information from:

* FINTRAC regulations
* Internal AML policies
* Historical investigation reports
* Customer due diligence documentation
* Risk management procedures

The retrieval layer must support semantic search, keyword search, vector search, and hybrid search while maintaining low latency and high relevance.

Poor retrieval quality would directly increase hallucinations and investigation errors.

## Decision Drivers

* Hybrid search support
* Native vector search
* Azure integration
* Scalability
* Security
* Regulatory compliance

## Options Considered

### Option 1: Pinecone

Pros:

* Mature vector database
* Strong vector performance

Cons:

* Additional vendor
* External data management

Why Rejected:
Introduces unnecessary operational complexity and vendor dependency.

### Option 2: Elasticsearch

Pros:

* Powerful search capabilities
* Large ecosystem

Cons:

* Operational overhead
* Complex tuning

Why Rejected:
Higher management burden than required.

### Option 3: Azure AI Search ← Chosen

Pros:

* Native vector search
* Hybrid retrieval
* Semantic ranking
* Azure-native security
* Managed service

Cons:

* Higher cost at scale
* Less flexible than self-managed alternatives

Tradeoffs Accepted:
Higher service cost in exchange for reduced operational complexity.

## Decision

Azure AI Search will be used as the primary retrieval platform supporting vector, keyword, semantic, and hybrid search for all AFCIP RAG workloads.

## Consequences

### Positive

* Reduced hallucinations
* Improved investigation quality
* Simplified operations
* Native Azure integration

### Negative

* Service cost growth with index size
* Vendor lock-in

### Risks

Risk:
Poor chunking strategy reducing retrieval quality.

Mitigation:
Implement evaluation framework and retrieval benchmarking.

## Compliance Considerations

### FINTRAC / PCMLTFA

Supports traceable evidence retrieval.

### OSFI E-23

Supports explainable AI recommendations.

### Data Residency Requirements

Indexes remain within approved Azure regions.

### PIPEDA

Supports encryption and RBAC controls.

## Review Date

2027-06-16

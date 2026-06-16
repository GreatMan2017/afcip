# ADR-006: Choice of GPT-4o as the Primary Foundation Model

## Status

Accepted

## Date

2026-06-16

## Context

AFCIP relies heavily on generative AI capabilities to perform:

* AML investigations
* Regulatory analysis
* Evidence summarization
* SAR report generation
* Agent reasoning
* Multi-agent collaboration
* Customer risk assessment

The chosen foundation model must provide strong reasoning capabilities, reliable structured outputs, low latency, enterprise governance support, and cost efficiency.

Because model usage represents a major operational cost driver, model selection has significant architectural and financial implications.

## Decision Drivers

* Reasoning quality
* Structured output support
* Latency
* Cost efficiency
* Enterprise support
* Azure integration
* Long-term maintainability

## Options Considered

### Option 1: GPT-4.1

Description:
Advanced OpenAI reasoning model available through Azure AI Foundry.

Pros:

* Excellent reasoning performance
* Strong coding capabilities
* High-quality structured outputs
* Advanced instruction following

Cons:

* Higher operational cost
* Increased latency for certain workloads

Why Rejected:

Best suited for specialized complex reasoning tasks rather than primary platform-wide inference.

### Option 2: Claude Sonnet (Azure AI Foundry Model Catalog)

Description:
Anthropic foundation model available through Azure AI Foundry.

Pros:

* Strong reasoning capabilities
* Excellent document understanding
* Large context window

Cons:

* Additional operational complexity
* Less standardized enterprise usage patterns
* Potential model portability concerns

Why Rejected:

Strong candidate, but less aligned with Microsoft-first architecture strategy.

### Option 3: GPT-4o ← Chosen

Description:
Multimodal enterprise foundation model available through Azure OpenAI Service.

Pros:

* Excellent reasoning quality
* Strong structured outputs
* Lower latency
* Cost-effective compared to larger models
* Native Azure integration
* Multimodal support
* Strong ecosystem adoption

Cons:

* Slightly lower reasoning performance than GPT-4.1 in certain complex tasks
* Potential model evolution over time

Tradeoffs Accepted:

Accept marginally lower reasoning performance in exchange for lower cost, faster response times, and broader deployment suitability.

## Decision

GPT-4o will serve as the primary foundation model for AFCIP agent workloads, investigation generation, evidence summarization, SAR drafting, and regulatory analysis.

GPT-4.1 may be selectively used for specialized high-complexity reasoning workflows when required.

## Consequences

### Positive

* Lower inference costs
* Faster response times
* Consistent platform architecture
* Strong Azure integration
* Simplified operations

### Negative

* Some advanced reasoning tasks may require escalation to GPT-4.1

### Risks

Risk:
Future model performance changes or deprecations.

Mitigation:
Implement model abstraction layers and maintain support for alternative models through Azure AI Foundry Model Catalog.

## Compliance Considerations

### FINTRAC / PCMLTFA

Supports explainable and auditable investigation outputs through grounded RAG workflows.

### OSFI E-23

Supports model governance, monitoring, validation, and operational controls.

### Data Residency Requirements

Inference endpoints deployed within approved Azure regions where available.

### PIPEDA

Supports encryption, RBAC, managed identities, and enterprise security controls.

## Review Date

2027-06-16

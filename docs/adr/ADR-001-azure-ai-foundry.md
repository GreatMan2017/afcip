# ADR-001: Choice of Azure AI Foundry as the AI Engineering Platform

## Status

Accepted

## Date

2026-06-16

## Context

The Autonomous Financial Crime Investigation Platform (AFCIP) requires a centralized platform for developing, evaluating, deploying, governing, and monitoring AI-powered agents used in fraud detection, AML investigations, regulatory compliance analysis, and suspicious activity reporting.

The platform must support:

* Agentic AI workflows
* Prompt engineering
* Model evaluation
* Responsible AI controls
* Enterprise governance
* Multi-model deployment
* Integration with Azure services

Financial institutions operate in highly regulated environments where AI systems must be explainable, auditable, secure, and manageable throughout their lifecycle.

An incorrect platform choice could result in fragmented tooling, poor governance, increased operational costs, and challenges meeting regulatory requirements.

## Decision Drivers

* Enterprise AI governance
* Native support for agentic AI systems
* Integration with Azure ecosystem
* Model evaluation and monitoring
* Responsible AI capabilities
* Long-term Microsoft roadmap alignment

## Options Considered

### Option 1: Azure Machine Learning

Description:
General-purpose machine learning platform for model development and deployment.

Pros:

* Mature MLOps capabilities
* Strong model management
* Excellent ML lifecycle support

Cons:

* Limited native agent development experience
* Requires additional orchestration tooling
* Less optimized for generative AI workloads

Why Rejected:
Provides excellent ML capabilities but lacks the integrated AI application development experience required for agentic systems.

### Option 2: Custom OpenAI + Custom Orchestration

Description:
Build custom AI platform using Azure OpenAI APIs and internally developed tooling.

Pros:

* Maximum flexibility
* Full architectural control

Cons:

* High engineering effort
* Significant governance burden
* Limited built-in evaluation capabilities

Why Rejected:
Operational complexity outweighs benefits for a single-engineer implementation.

### Option 3: Azure AI Foundry ← Chosen

Description:
Microsoft's unified platform for enterprise AI application development and operations.

Pros:

* Native AI agent support
* Built-in evaluation framework
* Prompt management
* Responsible AI tooling
* Integrated observability
* Strong Azure ecosystem integration

Cons:

* Platform evolution may introduce breaking changes
* Some advanced features remain relatively new

Tradeoffs Accepted:
Accept dependence on Microsoft's platform roadmap in exchange for accelerated development and enterprise-grade governance.

## Decision

Azure AI Foundry will serve as the primary AI engineering platform for developing, deploying, evaluating, and governing all AI agents within AFCIP.

## Consequences

### Positive

* Unified AI engineering experience
* Faster agent development
* Strong governance capabilities
* Simplified model lifecycle management
* Native Azure integration

### Negative

* Increased platform dependency
* Potential learning curve for new capabilities

### Risks

Risk:
Rapid platform evolution.

Mitigation:
Maintain abstraction layers around AI services and regularly review platform roadmap updates.

## Compliance Considerations

### FINTRAC / PCMLTFA

Supports auditability and traceability of AI-generated investigation outputs.

### OSFI E-23

Supports model governance, monitoring, and operational resilience requirements.

### Data Residency Requirements

Supports deployment within Canadian Azure regions.

### PIPEDA

Supports enterprise identity controls, encryption, and access management.

## Review Date

2027-06-16

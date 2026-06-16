# ADR-005: Choice of Azure Kubernetes Service (AKS) over Azure Container Apps and Azure Functions for Agent Hosting

## Status

Accepted

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

The platform must support:

* Long-running workflows
* Agent orchestration
* Event-driven processing
* Horizontal scaling
* Enterprise networking
* Secure service-to-service communication
* Production-grade observability

The hosting platform must support future growth from a portfolio project into a realistic enterprise deployment model.

Choosing the wrong hosting platform could create scaling limitations, operational constraints, and security challenges.

## Decision Drivers

* Scalability
* Operational maturity
* Enterprise networking
* Security controls
* Long-running workloads
* Multi-agent architecture support
* Future production readiness

## Options Considered

### Option 1: Azure Functions

Description:
Serverless event-driven compute platform.

Pros:

* Low operational overhead
* Cost-effective for small workloads
* Rapid development

Cons:

* Execution duration limits
* Limited support for complex workflows
* Difficult debugging of distributed agents
* Limited orchestration flexibility

Why Rejected:

Functions are suitable for event triggers but not ideal as the primary hosting platform for a multi-agent enterprise AI system.

### Option 2: Azure Container Apps

Description:
Serverless container hosting platform.

Pros:

* Easier management than Kubernetes
* Built-in scaling
* Lower operational burden

Cons:

* Less control over networking
* Less operational flexibility
* Limited advanced Kubernetes capabilities

Why Rejected:

Excellent choice for MVP environments, but insufficient control for enterprise-grade AI platform operations.

### Option 3: Azure Kubernetes Service (AKS) ← Chosen

Description:
Managed Kubernetes platform for containerized applications.

Pros:

* Enterprise-grade scalability
* Advanced networking controls
* Service mesh support
* Fine-grained security controls
* Multi-agent deployment flexibility
* Production AI platform alignment
* Industry-standard architecture

Cons:

* Higher operational complexity
* Greater learning curve
* Increased management overhead

Tradeoffs Accepted:

Accept additional operational complexity in exchange for maximum flexibility, scalability, and production readiness.

## Decision

Azure Kubernetes Service (AKS) will serve as the primary compute platform for hosting AFCIP agents, orchestration services, APIs, and supporting microservices.

## Consequences

### Positive

* Enterprise deployment architecture
* Maximum scalability
* Advanced networking capabilities
* Improved resiliency
* Strong production alignment

### Negative

* Increased operational burden
* Higher infrastructure complexity

### Risks

Risk:
Operational complexity exceeding project requirements.

Mitigation:
Use Infrastructure as Code, GitOps, monitoring automation, and managed Kubernetes services.

## Compliance Considerations

### FINTRAC / PCMLTFA

Supports secure handling of investigation workflows.

### OSFI E-23

Supports operational resilience, observability, and infrastructure governance.

### Data Residency Requirements

AKS clusters deployed in approved Canadian regions.

### PIPEDA

Supports private networking, workload identities, encryption, and least-privilege access controls.

## Review Date

2027-06-16

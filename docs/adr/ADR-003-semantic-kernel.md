# ADR-003: Choice of Semantic Kernel as the Agent Orchestration Framework

## Status

Accepted

## Date

2026-06-16

## Context

AFCIP requires orchestration of multiple specialized agents including:

* Alert Triage Agent
* Customer Risk Agent
* Transaction Intelligence Agent
* Regulatory Agent
* Case Correlation Agent
* SAR Generation Agent

The orchestration framework must support tool calling, memory management, planning, and enterprise integration.

## Decision Drivers

* Microsoft ecosystem alignment
* Multi-agent support
* Enterprise readiness
* Extensibility
* Future Azure AI Agent integration

## Options Considered

### Option 1: LangChain

Pros:

* Large ecosystem
* Community adoption

Cons:

* Rapidly changing APIs
* Higher maintenance burden

Why Rejected:
Enterprise stability concerns.

### Option 2: AutoGen

Pros:

* Strong multi-agent collaboration

Cons:

* Less mature operational ecosystem
* More research-focused

Why Rejected:
Less aligned with long-term enterprise production strategy.

### Option 3: Semantic Kernel ← Chosen

Pros:

* Microsoft-backed
* Native Azure integration
* Enterprise architecture alignment
* Strong planning capabilities

Cons:

* Smaller ecosystem
* Newer framework

Tradeoffs Accepted:
Accept smaller ecosystem in exchange for tighter Azure integration.

## Decision

Semantic Kernel will be used as the primary orchestration framework for agent communication, planning, memory management, and tool execution.

## Consequences

### Positive

* Consistent agent architecture
* Easier Azure integration
* Enterprise supportability

### Negative

* Smaller community ecosystem

### Risks

Risk:
Framework evolution.

Mitigation:
Use abstraction layers around orchestration components.

## Compliance Considerations

### FINTRAC / PCMLTFA

Supports traceable agent reasoning workflows.

### OSFI E-23

Improves AI governance and operational transparency.

### Data Residency Requirements

No impact.

### PIPEDA

Supports secure access controls and identity integration.

## Review Date

2027-06-16

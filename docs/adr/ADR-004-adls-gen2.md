# ADR-004: Choice of Azure Data Lake Storage Gen2 as the Primary Data Store

## Status

Accepted

## Date

2026-06-16

## Context

The Autonomous Financial Crime Investigation Platform (AFCIP) must store and manage large volumes of structured, semi-structured, and unstructured data including:

* Transaction records
* Customer KYC documents
* AML investigation reports
* Suspicious Activity Reports (SARs)
* Regulatory policies
* Audit logs
* Model evaluation datasets
* Agent execution logs

The platform requires a scalable storage solution capable of supporting analytical workloads, AI workloads, regulatory retention requirements, and future integration with Azure Synapse Analytics, Microsoft Fabric, and Azure Machine Learning.

Financial institutions often retain records for 5–10 years or longer. Storage costs, scalability, governance, and auditability are therefore critical considerations.

Selecting the wrong storage platform could significantly increase operational costs, reduce analytical flexibility, and create compliance risks.

## Decision Drivers

* Enterprise scalability
* Cost efficiency
* Support for AI and analytics workloads
* Integration with Synapse and Fabric
* Regulatory retention requirements
* Governance and auditability
* Future data lakehouse architecture support

## Options Considered

### Option 1: Azure SQL Database

Description:
Managed relational database service optimized for transactional workloads.

Pros:

* Strong ACID compliance
* Familiar SQL interface
* Mature security controls
* Excellent transactional consistency

Cons:

* Expensive for large-scale storage
* Poor fit for document repositories
* Limited support for large unstructured datasets
* Not optimized for AI document ingestion

Why Rejected:

The majority of AFCIP data consists of files, documents, logs, and analytical datasets rather than transactional records.

### Option 2: Azure Cosmos DB

Description:
Globally distributed NoSQL database supporting multiple APIs and low-latency access.

Pros:

* Flexible schema
* High scalability
* Excellent performance
* Suitable for agent memory stores

Cons:

* Higher storage costs
* Less suitable for large file repositories
* Not optimized for lakehouse architectures
* Limited analytics capabilities without additional services

Why Rejected:

Cosmos DB is better suited for operational metadata, agent memory, and session state than enterprise-scale financial crime datasets.

### Option 3: Azure Data Lake Storage Gen2 ← Chosen

Description:
Enterprise-scale data lake platform built on Azure Storage.

Pros:

* Extremely low storage cost
* Unlimited scalability
* Native integration with Synapse
* Native integration with Microsoft Fabric
* Supports structured and unstructured data
* Supports AI and ML workloads
* Hierarchical namespace support
* Enterprise-grade security

Cons:

* Not designed for low-latency transactions
* Requires additional services for advanced querying

Tradeoffs Accepted:

Accept slower transactional access in exchange for significantly lower cost, higher scalability, and stronger analytics capabilities.

## Decision

Azure Data Lake Storage Gen2 will serve as the primary enterprise data repository for AFCIP and will store all investigation data, documents, training datasets, model artifacts, audit logs, and AI knowledge assets.

## Consequences

### Positive

* Supports enterprise-scale growth
* Enables future lakehouse architecture
* Simplifies AI and analytics integration
* Reduces long-term storage costs
* Supports regulatory retention requirements

### Negative

* Additional services required for transactional workloads
* More complex access patterns than relational databases

### Risks

Risk:
Poor data organization resulting in governance challenges.

Mitigation:
Implement data zones, naming standards, metadata management, and Microsoft Purview integration.

## Compliance Considerations

### FINTRAC / PCMLTFA

Supports long-term retention of investigation evidence and audit records.

### OSFI E-23

Supports operational resilience and data governance requirements.

### Data Residency Requirements

Storage accounts deployed within Canadian Azure regions.

### PIPEDA

Supports encryption at rest, encryption in transit, RBAC, and private networking.

## Review Date

2027-06-16

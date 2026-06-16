# ADR-008: Event Ingestion Pattern — Event Hubs Partitioning, Functions Trigger Semantics, and Dead-Letter Handling

## Status

Accepted (New)

## Date

2026-06-16

## Context

AFCIP's ingestion layer must move synthetic transaction events from the generator through enrichment (sanctions/KYC join, preliminary risk flagging) into the ADLS Gen2 raw zone (ADR-004) reliably and in an order that preserves per-entity transaction sequencing — the Transaction Pattern Agent's structuring/layering detection logic depends on seeing one entity's transactions in order, not interleaved arbitrarily across a distributed consumer group. No prior ADR has decided the partitioning scheme, the delivery guarantee, or what happens to malformed events, despite these being prerequisites for both the Functions enrichment design and the raw zone partition layout already referenced in ADR-004.

## Decision Drivers

* Per-entity event ordering, required by downstream pattern-detection logic
* At-least-once delivery is acceptable; exactly-once is not required given downstream idempotency (see Decision)
* Malformed or schema-violating events must not block the rest of the stream
* Raw zone partition layout must support efficient targeted reads by the curated-zone batch job
* Solo-build operational simplicity — avoid a delivery-guarantee mechanism that requires complex coordination to maintain

## Options Considered

### Option 1: Single Event Hub partition, global ordering

Pros: simplest possible ordering guarantee — everything is strictly ordered.
Cons: eliminates parallelism entirely; throughput bottlenecked to a single consumer regardless of entity count.
Why Rejected: defeats the purpose of using Event Hubs at all; ordering is only required per-entity, not globally, so this over-constrains the design for no benefit.

### Option 2: Exactly-once delivery via Event Hubs checkpointing + downstream deduplication store

Pros: removes any possibility of duplicate processing.
Cons: requires an additional deduplication store (e.g., a processed-event-ID table) and adds operational complexity disproportionate to the actual risk, since the enrichment step and raw-zone write can both be made idempotent without it.
Why Rejected: the complexity is unnecessary once enrichment and storage writes are designed to be idempotent on the event's natural key (entity ID + transaction ID + timestamp), which Option 3 already requires regardless.

### Option 3: Partition by entity ID, at-least-once delivery, idempotent writes, explicit dead-letter path ← Chosen

Pros: partitioning by entity ID guarantees per-entity ordering within a partition while still allowing parallelism across entities (the actual requirement); at-least-once delivery is the Event Hubs default and requires no extra mechanism; idempotent writes (using entity ID + transaction ID as a natural key, so a duplicate write overwrites identically rather than duplicating) absorb the at-least-once delivery's only real risk cheaply; a dedicated dead-letter path isolates malformed events without halting the stream.
Cons: partition count must be chosen with enough headroom for entity cardinality, or hot partitions become a bottleneck — requires a one-time sizing decision rather than being fully automatic.
Tradeoffs Accepted: accept a one-time partition-count sizing decision and reliance on idempotent downstream writes (rather than a stronger delivery guarantee) in exchange for materially simpler operations.

## Decision

Events are partitioned in Event Hubs by entity ID (using entity ID as the partition key), guaranteeing that all events for a given entity are processed in order by a single consumer, while different entities are processed in parallel across partitions. Delivery is at-least-once (Event Hubs default); the enrichment Function and the raw-zone write are both designed to be idempotent on the natural key (entity ID + transaction ID + timestamp), so duplicate delivery is a no-op rather than a data-quality issue. Events that fail schema validation are written to a separate `raw/_deadletter/` path with the original payload and a validation-failure reason, rather than blocking or dropping the batch. The raw zone partition layout is by date and entity type (not date alone), enabling the curated-zone batch job to do targeted reads without scanning unrelated entity types.

Enrichment Function design (pseudocode, structure only):

```
EnrichmentFunction(event_hub_trigger, partition_key=entity_id)
  → parse_and_validate(event)
      on failure → write_to_deadletter(event, reason) ; return
  → enrich_sanctions(entity_id)      # static snapshot per ADR-008 sanctions note below
  → enrich_kyc(entity_id)            # mock KYC store
  → compute_preliminary_risk_flags(transaction)   # rule-based, not ML, at this stage
  → write_to_raw_zone(record, partition=date/entity_type, key=entity_id+txn_id+timestamp)
      # idempotent: identical key overwrites rather than duplicates
```

Sanctions/KYC data source: a static OFAC-equivalent snapshot and a mock KYC store are used rather than a live third-party API, both because a real-time feed isn't available to an individual builder and because it avoids introducing the external-API-key exception noted in ADR-007. This is documented as a known simplification relative to a production deployment, not hidden as if it were a live integration.

## Consequences

### Positive

Ordering guarantee matches the actual requirement (per-entity, not global) without sacrificing parallelism; idempotent design absorbs the only real risk of at-least-once delivery cheaply; dead-letter isolation keeps one malformed event from stalling the pipeline; raw zone layout supports efficient downstream reads as already assumed in ADR-004.

### Negative

Partition count is a one-time sizing decision that could need revisiting if entity cardinality in the synthetic dataset changes significantly; the static sanctions/KYC snapshot is a known fidelity gap versus a real institution's live data feeds, which should be stated explicitly in any portfolio walkthrough rather than implied to be live.

### Risks

Risk: skewed entity distribution in synthetic data creates a hot partition.

Mitigation: validate partition distribution as part of the Day 2 synthetic data design step; adjust partition count or entity-ID hashing if skew is observed.

Risk: dead-letter volume grows silently if the generator's schema drifts from the validation contract during iteration.

Mitigation: include dead-letter count as a basic monitoring metric from Phase 1 onward, not deferred to Phase 4.

## Compliance Considerations

### FINTRAC / PCMLTFA

Dead-letter path preserves every received event (even invalid ones) for audit purposes rather than silently dropping data, supporting full alert-lifecycle traceability.

### OSFI E-23

Idempotent design and explicit dead-letter handling are documented operational resilience controls, not implicit assumptions.

### Data Residency Requirements

Event Hubs namespace and Functions deployed within approved Azure regions, consistent with ADR-004.

### PIPEDA

Managed-identity-based access per ADR-007; no credentials introduced by this ingestion design.

## Review Date

2027-06-16

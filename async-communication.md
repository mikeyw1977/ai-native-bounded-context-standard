# AINBCAS — Asynchronous Communication

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

---

## Why Async Communication Matters for Cell Architecture

Synchronous API calls create temporal coupling: the calling cell must wait for
the responding cell, and both must be available simultaneously. For most
request-response interactions, this is acceptable. For value stream transitions,
state change notifications, and cross-cell reactions to domain events, it is
the wrong model.

When Cell A calls Cell B synchronously to notify it that something happened,
Cell A is now responsible for Cell B's availability. If Cell B is down, Cell A
must decide whether to retry, fail, or proceed — making Cell A responsible for
Cell B's reliability. This is hidden coupling expressed as an operational
dependency.

Asynchronous communication separates the concern of *recording that something
happened* from the concern of *reacting to it*. Events decouple cells in time.
Contracts decouple them in knowledge. Both are required for genuine cell
independence.

---

## When to Use Events vs Synchronous APIs

| Use synchronous API when | Use events when |
|---|---|
| The caller needs the response to proceed | The caller does not need to know what happens next |
| The interaction is a query (read) | The interaction is a notification (state change) |
| Failure of the callee should fail the caller | The callee's availability should not affect the caller |
| The response must be immediate | The reaction can be eventual |
| One cell needs data from another | One cell's state change is relevant to multiple cells |

A common mistake is using synchronous APIs for notifications because they are
simpler to implement initially. This creates point-to-point coupling that must
be unwound later. If the caller does not use the response, use an event.

---

## The Domain Event Model

### Events are value stream transitions

Every meaningful event in an AINBCAS system corresponds to a transition in
a value stream. An event is not a technical callback — it is a domain occurrence
that other parts of the system may need to react to.

A well-named event is:
- Past tense (it has already happened)
- Business-language (describes the domain occurrence, not the technical mechanism)
- Self-describing (contains enough context for consumers to react without calling back)

Well-named: `BeliefConfidenceUpdated`, `CandidateQualified`, `EvaluationCompleted`,
`ActionApproved`, `ExecutionRecorded`

Poorly named: `CellOutputReady`, `ProcessingDone`, `DataChanged`, `UpdateEvent`

### Event schema standard

Every event must contain:

```json
{
  "eventId":     "string (uuid) — stable, globally unique identifier for this occurrence",
  "eventType":   "string — namespaced event name, e.g. 'belief.confidence_updated'",
  "schemaVersion": "string — e.g. '1.0'",
  "occurredAt":  "string (ISO 8601) — when the domain occurrence happened",
  "producerCell": "string — the cell that produced this event",
  "correlationId": "string (uuid) — links related events in a flow; passed through from initiating request",
  "payload":     "object — event-specific data (see individual event contracts)"
}
```

**`eventId` is immutable.** If the same event is delivered more than once (at-least-once
delivery), the `eventId` is the same. Consumers must deduplicate on `eventId`.

**`occurredAt` is when the domain occurrence happened,** not when the event was
published. These may differ under backpressure or replay.

**`correlationId` links events in a flow.** When Cell A processes an event from
Cell B and produces its own event as a result, the new event carries the same
`correlationId`. This enables end-to-end flow tracing without shared state.

---

## Event Contracts

Events are contracts, subject to the same versioning and governance as API
contracts (Principle 5).

### Where event contracts live

Event schemas are defined in `contracts/events/`. Each event type has its own
schema file. The schema file defines:
- The event type name
- The payload schema with field types and optionality
- The producer cell
- Known consumer cells
- Version history

No cell may publish an event that is not defined in `contracts/events/`. No cell
may consume an event in a way that depends on undocumented fields.

### Versioning

Event schema versioning follows the same rules as API contracts:
- Adding an optional field: no version bump required
- Adding a required field: new version required
- Removing or renaming a field: new version required
- Changing field type or semantics: new version required

During a version migration:
1. Producers publish both old and new schema versions for the migration window
2. Consumers migrate to the new version
3. The old version is deprecated and removed

Consumers must be tolerant of unknown fields (ignore, do not error). This enables
producers to add optional fields without coordinating consumer updates.

---

## Delivery Guarantees

### At-least-once delivery

AINBCAS requires at-least-once delivery as the minimum guarantee. An event must
not be lost under normal operating conditions. It may be delivered more than once.
Consumers must be idempotent — processing the same event twice must produce the
same result as processing it once.

Idempotency is achieved by:
- Checking `eventId` against a consumer-local record of processed events
- Designing consumer logic so that applying the same state change twice is
  a no-op (e.g. setting a value rather than incrementing it)

### At-least-once does not mean ordering

Events from the same producer may arrive out of order under failure and replay
conditions. Consumers must not assume ordering unless the event infrastructure
explicitly provides it for a specific stream.

When ordering matters (e.g. belief confidence updates must be applied in
chronological order), use `occurredAt` to sequence events at the consumer,
and design the payload to carry enough context to detect and handle out-of-order
delivery.

### Exactly-once is aspirational

Exactly-once delivery requires distributed transactions that are complex,
expensive, and infrastructure-dependent. Do not depend on it at the application
level. Design for at-least-once with idempotent consumers.

---

## The Event Bus

### Infrastructure is a platform concern

The event bus is infrastructure. Individual cells do not implement their own
message queues, brokers, or delivery mechanisms. They publish to and consume
from a shared event infrastructure whose implementation is a platform decision.

Acceptable event bus implementations include message brokers (RabbitMQ, Kafka,
NATS), cloud-native queuing services, or in-process event buses for single-process
deployments during early development.

What matters architecturally is not the implementation but the interface:
- Cells publish events by calling a platform-provided publish API
- Cells consume events by registering a handler for a named event type
- The platform is responsible for delivery, retry, and durability

### Bus unavailability

When the event bus is unavailable:
- Cells must not fail their primary function. Bus unavailability is a degraded
  state, not a failure state.
- Cells must buffer events locally if the bus is unavailable, and publish them
  when connectivity is restored (outbox pattern).
- Cells must not use synchronous API calls as a fallback for events during bus
  unavailability. This recreates the coupling that events were designed to prevent.

The outbox pattern: before publishing an event, write it to a local outbox store.
A background process reads the outbox and publishes to the bus, deleting from
the outbox only on confirmed delivery. This guarantees at-least-once delivery
even under bus outages.

---

## Consumer Patterns

### Competing consumers

Multiple instances of the same cell may consume events from the same stream.
The event infrastructure must guarantee that each event is delivered to only one
instance. Do not implement this at the application level.

### Fan-out

When multiple different cells need to react to the same event, each cell has its
own subscription. The event is delivered independently to each consumer. One
consumer's failure does not affect others.

### Event sourcing

AINBCAS does not require event sourcing (rebuilding state from event history)
but is compatible with it. Cells that implement event sourcing internally must
not expose their event streams as inter-cell contracts — that is hidden coupling.
Inter-cell events are domain events, not internal state events.

---

## Tracing and Observability

Every event consumed and every event published must be logged by the cell, with:
- The `eventId` and `eventType`
- The `correlationId`
- The processing outcome (success, failure, deduplicated)
- The resulting state change, if any

This enables end-to-end tracing of a value stream flow across multiple cells
without requiring a distributed tracing infrastructure (though one may complement
it).

A flow that cannot be traced from the initiating event through each cell to its
outcome cannot be diagnosed. Untraceable flows violate Principle 8 (Observability)
and Principle 9 (Observable Transformation).

---

## Relationship to the Standard

### Events and value stream transitions

Events are the mechanism by which value stream transitions become observable.
When a cell advances the value stream — qualifying a candidate, completing an
evaluation, recording an execution — it publishes an event. Other cells that
need to respond to that transition subscribe.

Designing events as value stream transitions, rather than as technical callbacks,
enforces the business-first naming and scoping that Principle 2 requires.

### Events and AI artifacts

AI-derived conclusions that have operational consequence (Principle 11) must be
stored as explicit artifacts before events are published. Do not publish events
that carry AI reasoning inline as their only record. The reasoning is an artifact;
the event is a notification that the artifact exists.

```
AI cell produces reasoning → stores artifact with stable ID → publishes event with artifact reference
```

Consumers follow the artifact reference to access the reasoning. The event
carries the identifier and the summary, not the full reasoning payload. This
keeps events small and separates the transport concern from the storage concern.

### Events and contracts

Events are contracts. They are versioned. They are owned by a producer cell.
They are defined in `contracts/events/`. They are subject to the same governance
as API contracts.

A cell that publishes undocumented events is violating Principle 5 as clearly
as a cell that serves undocumented API routes.

---

## Event Design Checklist

Before publishing a new event type:

- [ ] Event type is defined in `contracts/events/` with a schema and version
- [ ] Event name is past tense and in business language
- [ ] Payload is self-describing (consumers can react without calling back)
- [ ] `eventId` is a stable UUID unique to this occurrence
- [ ] `correlationId` is threaded through from the initiating request
- [ ] Producer cell and known consumers are documented in the contract
- [ ] Consumer idempotency is designed for (duplicate delivery is handled)
- [ ] Outbox pattern is implemented if the bus may be unavailable
- [ ] Event is logged by both producer and all consumers

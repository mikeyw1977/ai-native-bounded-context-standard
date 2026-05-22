# AINBCAS — Patterns

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

---

# Suggested Topology

AINBCAS does not mandate microservices.

A compliant architecture may begin as:

* modular monolith,
* modular monorepo,
* or partially decomposed system.

The fat-cell pattern is explicitly supported: a single deployable unit may implement
the full intended operator workflow while cell boundaries are defined but not yet
extracted. This is not a compromise — it is the correct starting point. Extract
boundaries when fitness-function evidence justifies it.

Service decomposition should evolve based on:

* capability maturity,
* operational requirements,
* and fitness-function evidence.

---

# Interface Language

The naming conventions of any user-facing interface — API endpoints, event schemas,
UI labels, dashboard terminology, or command surfaces — are architectural decisions.

Interface language that reflects the domain model:

* trains operators and developers to think in business capability terms,
* decouples the interface from the implementation topology,
* remains stable as cells are extracted, replaced, or reorganised.

Interface language that reflects implementation:

* leaks runtime module names into the user-facing layer,
* creates hidden coupling between interface and deployment,
* requires interface changes when implementation changes.

This applies regardless of interface type. A REST API endpoint, a UI button, and
an event name all carry the same risk. Naming a capability after its implementing
module rather than its business function is a form of hidden coupling.

The principle is not about aesthetics. An interface that teaches users to think in
implementation terms makes the architecture harder to evolve, because every refactor
must also change what users have learned.

---

# Structural Narrative and Execution Record

AI-native systems that operate on accumulated evidence must explicitly separate two
distinct categories of artifact:

**Structural narratives** are persistent, multi-entity beliefs about the world.
They accumulate evidence across many events and sessions. They have long lifecycles
— weeks to months. They inform subsequent decisions but are not themselves decisions.
They span multiple entities within a domain (accounts, products, merchants, regions).

**Execution records** are specific, single-entity actions derived from structural
narratives. They have short lifecycles — hours to days. They are created, acted
upon, and closed. They reference the structural narrative that informed them.

Examples across domains:

| Domain | Structural Narrative | Execution Record |
|---|---|---|
| Market analysis | "Sector rotation signalling risk-off environment" | "Open defensive positioning review" |
| Fraud detection | "Merchant cluster showing coordinated timing" | "Block transaction #123456" |
| Customer success | "Enterprise account deteriorating post-upgrade" | "Escalate ticket #7890" |
| Healthcare | "Patient population showing early metabolic indicators" | "Schedule screening for patient Y" |

Conflating these produces signal-reactive systems: every new event creates a new
execution record without reference to accumulated belief. The consequences are:

* outcomes cannot be attributed — was the belief wrong, or was the execution wrong?
* the system forgets what it knows between sessions,
* longitudinal learning is impossible because the narrative layer does not exist,
* the same pattern is re-discovered repeatedly rather than recognised.

The lineage chain must be explicit and traceable:

```
Narrative → Decision → Outcome
```

Each layer has its own lifecycle, its own fitness metrics, and its own corrective
action path. Breaking the chain makes system-level learning impossible and outcome
attribution meaningless.

If a structural narrative only exists in prompt context or model memory, it does not
exist as an architectural artifact. It will be lost between sessions, cannot be
audited, and cannot be linked to the decisions it informed.

See also: [Principle 8 — AI Artifacts Are First-Class Versioned Objects](principles.md#8-ai-artifacts-are-first-class-versioned-objects)

---

# Contract Information Hiding

Contracts define what a cell exposes. They also implicitly define what a cell
does not expose. These are equally important decisions.

A cell that has processed a rich internal representation — persona transcripts,
intermediate scoring steps, evidence chains, model reasoning — knows more than
any consumer needs. The contract must expose the minimum sufficient for each
consumer to perform its function at its stage of the value stream.

**Why this matters:**

Exposing more than necessary creates hidden coupling. If an evaluation cell emits
full reasoning transcripts and a downstream action cell begins to depend on their
structure, that action cell now depends on the evaluation cell's internal
representation. When the evaluation cell changes how it generates reasoning, the
action cell breaks — through a mechanism that no contract clause prevents, because
the data was technically available.

The consumer should receive what it needs to act, not everything the provider knows.

**The value stream test:**

At each value stream stage, ask: what is the minimum output from this cell that
allows the next stage to perform its function without loss of correctness?

- evaluation-cell → action-cell: verdicts, confidence score, priority, divergence flag.
  Not reasoning transcripts, not intermediate scoring, not raw model outputs.
- qualification-cell → evaluation-cell: ranked candidates with structural signals.
  Not raw feature data, not intermediate scoring computations.
- belief-cell → qualification-cell: candidate entities for the current scope.
  Not confidence history, not change log, not evidence accumulation detail.

Consumers at later value stream stages accumulate more context because they need
it for governance decisions. Consumers at earlier stages need less. The contract
must reflect the consumer's position, not the provider's richness.

**Progressive disclosure by stage:**

This is not about hiding information for its own sake. It is about right-sizing
the contract surface so that:

- the provider can change its internal representation without breaking consumers,
- the consumer cannot accidentally build logic that depends on unstable internals,
- and each cell's contract teaches its consumers to think at the right level of
  abstraction for their position in the value stream.

A cell that exposes everything it knows has no internal representation — its
internals are its contract, making it unreplaceable.

**Audit access is a separate concern:**

Information hiding at the contract boundary does not mean the internal
representation is unavailable for audit. Observable transformation (Principle 13)
requires cells to store their full intermediate outputs. Those outputs are
accessible through audit mechanisms, not through the runtime contract.

Runtime consumers receive the minimum sufficient. Audit consumers receive everything.
These are different access paths with different purposes.

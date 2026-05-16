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
| Market intelligence | "Nuclear infrastructure entering a capex cycle" | "Enter LEU position at ≤$42" |
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

See also: [Principle 11 — AI Artifacts Are First-Class Versioned Objects](principles.md#11-ai-artifacts-are-first-class-versioned-objects)

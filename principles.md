# AINBCAS — Foundational Principles

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

---

## 1. AI Is a Capability, Not the Architecture

AI models are implementation details.

The architecture must not be centred around:

* a single model,
* a shared prompt,
* or a monolithic reasoning engine.

Instead:

* AI capabilities are deployed inside bounded contexts,
* constrained by contracts,
* governed by deterministic systems,
* and continuously evaluated.

---

## 2. Business Semantics Precede Technical Topology

Architecture must emerge from:

* value streams,
* business capabilities,
* and bounded domains.

Not from:

* prompts,
* agents,
* model APIs,
* or deployment tooling.

AINBCAS decomposition order:

```text
Value Streams
    ↓
Business Capabilities
    ↓
Domains / Bounded Contexts
    ↓
Services / Cells
    ↓
Deployments
```

---

## 3. Cells Are Bounded Runtime Units

A cell is an independently evolvable runtime unit implementing a bounded capability within a domain.

A cell:

* owns a constrained responsibility,
* exposes explicit contracts,
* maintains local reasoning only,
* and cannot bypass governance boundaries.

Cells are NOT:

* arbitrary prompts,
* generic "AI agents,"
* or autonomous authorities.

---

## 4. Deterministic Governance Overrides AI Interpretation

AI systems may:

* interpret,
* summarise,
* classify,
* hypothesise,
* critique,
* or recommend.

AI systems must NOT:

* bypass governance,
* directly mutate protected state,
* or execute privileged actions without deterministic approval.

Governance must remain:

* observable,
* auditable,
* deterministic,
* and testable.

---

## 5. Contracts Are Mandatory

All inter-cell communication must occur through explicit contracts.

Contracts may include:

* APIs,
* event schemas,
* pact contracts,
* versioned payload definitions,
* and typed interfaces.

Cells must not rely on:

* hidden prompt assumptions,
* shared internal state,
* or undocumented behaviour.

---

## 6. Fitness Functions Are First-Class Architecture Components

Architectural principles are insufficient unless continuously validated.

AINBCAS requires measurable fitness functions validating:

* boundary integrity,
* contract compatibility,
* governance enforcement,
* deployment isolation,
* model reliability,
* and operational resilience.

If a principle matters, it should eventually have a fitness function.

---

## 7. Cells Must Be Replaceable

No cell should permanently depend on:

* one model vendor,
* one prompt structure,
* or one implementation strategy.

Cells should be replaceable by:

* different models,
* deterministic systems,
* local inference,
* rules engines,
* or human workflows.

The system architecture must outlive individual model generations.

---

## 8. Observability Is Mandatory

AI-native systems require deeper observability than traditional services.

Cells should expose:

* inputs,
* outputs,
* uncertainty,
* reasoning metadata,
* evidence references,
* execution traces,
* and decision lineage.

Opaque AI behaviour is considered architectural risk.

---

## 9. Observable Transformation Principle

Every bounded context that transforms, filters, enriches, ranks, or suppresses
information must emit observable diagnostic evidence describing:

* what changed,
* why it changed,
* what was discarded,
* and what uncertainty was introduced.

This applies to every stage in any pipeline: ingestion, normalisation, enrichment,
ranking, convergence, governance, and execution.

Silent transformation is architectural corruption.

A cell that receives N items and outputs M, where M < N, must account for the
difference with explicit reasons. A cell that modifies confidence, ranking, or
classification must expose what changed and why. A cell that suppresses information
— by filtering, deduplicating, or ignoring — must record what it suppressed.

This principle exists because:

* a pipeline that loses signal silently cannot be diagnosed,
* a model that adjusts scores without evidence cannot be calibrated,
* and a system that cannot explain its transformations cannot be trusted.

Observable transformation is the prerequisite for independent cell evaluation,
A/B testing, and kill decisions. Without it, "the model is broken" and "the
environment was unfavourable" are indistinguishable.

---

## 10. Implement Intent First, Extract Boundaries Later

Name interfaces for the operator or user's intent, not the runtime module executing
underneath. The intended workflow model is the architecture. Cell boundaries are an
operational decision made later, when evidence justifies decomposition.

A system designed around operator intent can extract boundaries cleanly because the
interface contract is already correct. A system designed around runtime modules must
rebuild its interface contract when it decomposes — creating churn at every layer.

Practically:

* interface names — API endpoints, event names, UI labels, CLI commands — should
  express what the system does for the user, not which module runs,
* extraction happens when fitness function evidence justifies it, not speculatively,
* the interface contract does not change when the runtime topology changes.

The implementation evolves. The intent is stable.

---

## 11. AI Artifacts Are First-Class Versioned Objects

Beliefs, hypotheses, evidence chains, and derived conclusions are architectural
artifacts — not session state, not prompt context, not implicit model memory.

If an AI-derived conclusion has operational consequence, it must be:

* explicitly typed with a defined schema,
* persistently stored with a stable identifier,
* versioned with a change log,
* traceable through a lineage chain to its evidence and downstream decisions.

Keeping conclusions in prompt context makes them unauditable, unreplayable, and
unattributable. When a decision based on an implicit belief turns out to be wrong,
there is no way to determine whether the belief was wrong or the execution was wrong.

Artifacts with operational consequence must exist as objects, not as memory.

---

## 12. Evidence Quality and Conviction Are Distinct Properties

A signal's conviction — its directional certainty about an outcome — is different
from its data quality, completeness, or source reliability.

Sources must not conflate these:

* conviction: "this evidence strongly suggests buying this asset / escalating this
  account / blocking this transaction"
* data quality: "this reading is complete / partial / based on limited history"

Conflating them corrupts synthesis. A signal with low data completeness is not
a low-conviction signal — it is an incomplete signal. The response to incomplete
data is to collect more data. The response to low conviction is to require more
corroboration or apply a lower weight. These are different corrective actions.

Any cell that emits a confidence or conviction field must ensure that field
represents directional certainty about the domain claim, not a proxy for how much
data was available to make it.

---

## 13. Cells Are Deployment Units — Replace, Don't Patch

A running cell is not modified in place. It is versioned and replaced.

Patching cell behaviour by modifying shared prompt context, adjusting global
parameters, or overriding individual outputs accumulates hidden dependencies.
Each patch makes the cell less replaceable and the system harder to reason about.

The correct corrective action is always at the cell boundary:

* when a cell's behaviour degrades, deploy a new cell implementation,
* the new implementation satisfies the same contracts as the old one,
* downstream consumers observe no change,
* the old cell is retired.

Contracts are what make replacement safe. A cell without explicit contracts cannot
be replaced without risk. A cell with explicit contracts can be replaced at any time.

The target deployment pattern is cell immutability: cells are versioned artefacts,
not living systems that accumulate patches.

# AI-Native Bounded Context Architecture Standard (AINBCAS)

> Experimental Architectural Standard — Version 0.2

---

## Purpose

AINBCAS is a governance model for autonomous capability units. The same governance
applies whether a capability is delivered by a team of engineers, an AI system, or
a combination of both.

A **cell** is the unit of governance. It is an autonomous unit of capability with
a declared boundary, explicit contracts, and observable behaviour. A cell may be
any size — a single function, a service, a team's entire domain. What makes it a
cell is not its size or technology but its properties: it declares what it does,
exposes how it behaves, and can be held accountable through externally verifiable
evidence without access to its internal reasoning. A cell operated by an AI system
and a cell operated by a team of engineers are governed identically. That is the
point.

The governing thesis:

* The cell boundary is a trust boundary
* Contracts are behavioural constraints
* Fitness functions are operational accountability
* Observability is evidence
* Replayability is forensic capability
* Governance exists because internal reasoning cannot be trusted

The standard addresses architectural risks emerging in AI-driven systems:

* prompt monoliths and opaque reasoning,
* uncontrolled autonomy and hidden coupling,
* brittle orchestration and context-window dependency,
* unsafe execution behaviour,
* prompt injection as privilege escalation,
* governance degrading into ceremony.

AINBCAS proposes that these risks are managed through bounded capability units with
explicit contracts, deterministic governance gates, fitness functions, and
independently observable behaviour — not through trusting internal AI reasoning.

---

## Contents

| Document | What it covers |
|---|---|
| [principles.md](principles.md) | The 16 foundational principles |
| [cells.md](cells.md) | Cell qualification, cell principles, performance measurement, corrective action |
| [governance.md](governance.md) | Deterministic governance, recommendation-authority boundary, consequence classes, model accountability |
| [patterns.md](patterns.md) | Topology, interface language, structural narrative vs execution record |
| [anti-patterns.md](anti-patterns.md) | 14 explicitly discouraged patterns |
| [security-model.md](security-model.md) | Identity, authority, semantic integrity — the three trust breakdown points |
| [reference.md](reference.md) | EA alignment, suggested lifecycle, standard philosophy |
| [cell-development-model.md](cell-development-model.md) | Bounded AI development context, principle of least privilege, capability map, privilege elevation protocol |
| [deployment-model.md](deployment-model.md) | Repo model, per-cell CI pipelines, versioning, state migration, deployable cell checklist |
| [decomposition-watchdog.md](decomposition-watchdog.md) | Continuous boundary enforcement, structural and semantic drift detection, AI completion bias |
| [evaluation-model.md](evaluation-model.md) | Technical vs business fitness, replay, A/B cell replacement protocol, spec-driven regeneration |
| [async-communication.md](async-communication.md) | Event model, domain events as value stream transitions, delivery guarantees, event contracts |

---

## Principles — Quick Reference

| # | Principle |
|---|---|
| 1 | AI Is a Capability, Not the Architecture |
| 2 | Business Semantics Precede Technical Topology |
| 3 | Implement Intent First, Extract Boundaries Later |
| 4 | Cells Are Bounded Runtime Units |
| 5 | Every Cell Must Declare Its Value Stream Position |
| 6 | Cells Must Be Sized for AI Comprehension |
| 7 | Cell Specifications Must Be Sufficient for AI-Assisted Regeneration |
| 8 | AI Artifacts Are First-Class Versioned Objects |
| 9 | Evidence Quality and Conviction Are Distinct Properties |
| 10 | Contracts Are Mandatory |
| 11 | Deterministic Governance Overrides AI Interpretation |
| 12 | Observability Is Mandatory |
| 13 | Observable Transformation Principle |
| 14 | Fitness Functions Are First-Class Architecture Components |
| 15 | Cells Must Be Replaceable |
| 16 | Cells Are Deployment Units — Replace, Don't Patch |

---

## Anti-Patterns — Quick Reference

1. Prompt Monoliths
2. Hidden Coupling
3. Autonomous AI Authority
4. Semantic-Free Decomposition
5. Shared Prompt Architecture
6. Non-Testable Intelligence
7. Silent Transformation
8. Evidence Inflation
9. Confidence Conflation
10. Premature Boundary Extraction
11. Boundary Absorption
12. In-Context Patching
13. Prompt-Defined Authority
14. Governance Theatre

---

## Status

AINBCAS is currently experimental, being validated through practical implementation
in complex, multi-cell systems that emulate enterprise-scale concerns: multiple
interacting value streams, independently deployable cells with explicit contracts,
AI-assisted reasoning pipelines with deterministic governance gates, and
fitness-function-driven boundary enforcement.

The reference implementation deliberately mirrors the organisational complexity of
a real enterprise — cross-cutting governance, sequential and parallel value stream
stages, cells with distinct data ownership, and an EA layer that governs without
absorbing domain logic. This complexity is the test bed: the standard is considered
sound when it can govern a system of this kind without architectural drift.

Future revisions will evolve based on operational evidence, evaluation data, and
real-world architectural constraints.

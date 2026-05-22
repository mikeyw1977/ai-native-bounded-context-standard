# AI-Native Bounded Context Architecture Standard (AINBCAS)

> Experimental Architectural Standard — Version 0.2

---

## Purpose

AINBCAS defines principles, patterns, and governance mechanisms for building
intelligent systems composed of independently evolvable bounded contexts ("cells").

The standard addresses architectural risks emerging in AI-driven systems:

* prompt monoliths and opaque reasoning,
* uncontrolled autonomy and hidden coupling,
* brittle orchestration and context-window dependency,
* unsafe execution behaviour.

AINBCAS proposes that intelligent systems should be designed using enterprise
semantic architecture, bounded contexts, explicit contracts, deterministic
governance, fitness functions, and independently evolvable cells.

---

## Contents

| Document | What it covers |
|---|---|
| [principles.md](principles.md) | The 16 foundational principles |
| [cells.md](cells.md) | Cell qualification, cell principles, performance measurement, corrective action |
| [governance.md](governance.md) | Deterministic governance, recommendation-authority boundary, consequence classes, model accountability |
| [patterns.md](patterns.md) | Topology, interface language, structural narrative vs execution record |
| [anti-patterns.md](anti-patterns.md) | 11 explicitly discouraged patterns |
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

* Prompt Monoliths
* Hidden Coupling
* Autonomous AI Authority
* Semantic-Free Decomposition
* Shared Prompt Architecture
* Non-Testable Intelligence
* Silent Transformation
* Evidence Inflation
* Confidence Conflation
* Premature Boundary Extraction
* In-Context Patching

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

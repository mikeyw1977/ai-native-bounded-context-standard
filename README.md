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
| [principles.md](principles.md) | The 13 foundational principles |
| [cells.md](cells.md) | Cell qualification, cell principles, performance measurement, corrective action |
| [governance.md](governance.md) | Governance principles and fitness function categories |
| [patterns.md](patterns.md) | Topology, interface language, structural narrative vs execution record |
| [anti-patterns.md](anti-patterns.md) | 11 explicitly discouraged patterns |
| [reference.md](reference.md) | EA alignment, suggested lifecycle, standard philosophy |
| [cell-development-model.md](cell-development-model.md) | Bounded AI development context, principle of least privilege, capability map, privilege elevation protocol |
| [deployment-model.md](deployment-model.md) | Repo model, per-cell CI pipelines, versioning, state migration, deployable cell checklist |
| [decomposition-watchdog.md](decomposition-watchdog.md) | Continuous boundary enforcement, structural and semantic drift detection, AI completion bias |
| [evaluation-model.md](evaluation-model.md) | Technical vs business fitness, replay, A/B cell replacement protocol, spec-driven regeneration |
| [governance.md](governance.md) | Deterministic governance, recommendation-authority boundary, consequence classes, model accountability |
| [async-communication.md](async-communication.md) | Event model, domain events as value stream transitions, delivery guarantees, event contracts |

---

## Principles — Quick Reference

| # | Principle |
|---|---|
| 1 | AI Is a Capability, Not the Architecture |
| 2 | Business Semantics Precede Technical Topology |
| 3 | Cells Are Bounded Runtime Units |
| 4 | Deterministic Governance Overrides AI Interpretation |
| 5 | Contracts Are Mandatory |
| 6 | Fitness Functions Are First-Class Architecture Components |
| 7 | Cells Must Be Replaceable |
| 8 | Observability Is Mandatory |
| 9 | Observable Transformation Principle |
| 10 | Implement Intent First, Extract Boundaries Later |
| 11 | AI Artifacts Are First-Class Versioned Objects |
| 12 | Evidence Quality and Conviction Are Distinct Properties |
| 13 | Cells Are Deployment Units — Replace, Don't Patch |
| 14 | Cells Must Be Sized for AI Comprehension |
| 15 | Every Cell Must Declare Its Value Stream Position |
| 16 | Cell Specifications Must Be Sufficient for AI-Assisted Regeneration |

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

AINBCAS is currently experimental, being explored through practical implementation
using market intelligence systems, probabilistic analysis pipelines, and
fitness-function-driven governance.

Future revisions will evolve based on operational evidence, evaluation data, and
real-world architectural constraints.

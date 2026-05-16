# AINBCAS — Governance

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

---

# Governance Principles

Governance systems should:

* remain deterministic where possible,
* reject unsafe autonomy,
* enforce exposure/risk limits,
* provide audit trails,
* support kill switches,
* and remain independently testable.

AI-generated outputs should be treated as:

* evidence,
* recommendations,
* or probabilistic interpretation.

Never unquestionable authority.

---

# Fitness Function Categories

AINBCAS encourages fitness functions for:

## Boundary Integrity

* import restrictions
* deployment isolation
* shared-state prevention

## Contract Integrity

* schema compatibility
* pact validation
* replay validation

## Governance Integrity

* approval enforcement
* override auditing
* policy compliance

## Reliability

* output validation
* uncertainty emission
* hallucination reduction

## Operational Resilience

* degradation handling
* failure isolation
* circuit-breaker behaviour

## Evolvability

* replaceability
* deployment independence
* model abstraction

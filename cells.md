# AINBCAS — Cells

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

---

# Cell Qualification Criteria

Not every function deserves cell separation.

A capability should generally become a separate cell only when justified by operational or semantic need.

Potential qualification criteria include:

* distinct bounded context,
* independent deployment cadence,
* independent scaling requirements,
* isolated failure domain,
* different runtime requirements,
* security boundary separation,
* experimental volatility,
* or distinct ownership.

Premature cell proliferation is discouraged. See [Anti-Patterns — Premature Boundary Extraction](anti-patterns.md#premature-boundary-extraction).

See also: [Cell Boundary Trade-offs](#cell-boundary-trade-offs) for how to reason about co-location decisions before applying these criteria.

---

# Cell Boundary Trade-offs

The question of whether two capabilities belong in the same cell is frequently answered by default — either by a bias toward co-location ("the domain is still being discovered") or a bias toward separation ("separate concerns"). Neither default is categorically correct. The decision deserves deliberate reasoning.

## Two types of uncertainty that are often conflated

**Implementation uncertainty** — how complex is this? what are the edge cases? how does the internal logic work? — is a legitimate reason to iterate quickly inside a single cell before separating. You learn by building.

**Purpose uncertainty** — what is this capability *for*? what question does it answer? what data does it own? — is a different kind of uncertainty. When you can articulate the functional purpose clearly, the boundary is available. Whether the implementation is five lines or five hundred is irrelevant to whether the boundary should exist.

Invoking implementation uncertainty to justify co-locating capabilities with clearly distinct purposes is the most common route to accumulated coupling.

## The primary decision criterion

Before placing a new capability inside an existing cell, ask:

> Does this capability serve the same functional purpose as what is already here — answering the same questions, owning the same data, serving the same value stream?

If yes: same cell, even if it grows large.  
If no: separate cell, even if it starts simple.

Convenience of co-location is not a reason. Shared runtime is not a reason. The fact that two capabilities interact is not a reason — interaction is what contracts are for.

## When co-location is genuinely appropriate

Co-location within a fat cell is appropriate when the **domain vocabulary itself has not stabilised** — when you do not yet know what the right questions are, let alone who answers them. This applies to genuinely exploratory work where the business purpose of a capability is unknown, not merely underspecified.

A system that frequently invokes "the domain is still being discovered" to justify co-location is accumulating coupling under an exploratory excuse. If you can name what a capability is for, you can bound it.

## The practical challenge

Neither monolith-first nor granular decomposition is categorically correct. At any boundary decision, challenge the choice with three questions:

1. Can I clearly articulate what this capability is for, what question it answers, and what data it owns?
2. Does the answer differ meaningfully from what is already in the candidate cell?
3. If so, what is the cost of enforcing the boundary now versus the cost of extracting it later?

The answers vary by context. The practice is to ask the questions deliberately rather than defaulting to whichever pattern is locally convenient.

---

# Cell Principles

A compliant cell should:

* own a single bounded concern,
* expose explicit contracts,
* avoid direct shared-state mutation,
* maintain local reasoning only,
* be independently testable,
* be observable,
* support replay/evaluation,
* and remain replaceable.

A compliant cell must NOT:

* bypass governance,
* directly orchestrate unrelated domains,
* rely on hidden prompt coupling,
* or assume global system knowledge.

---

# Cell Performance Measurement

Cells are evaluated against two distinct dimensions:

**Functional correctness** — does the cell operate as contracted?

* outputs conform to schema,
* contracts are honoured,
* error conditions are handled as specified,
* observable diagnostics are emitted.

Functional failures indicate the cell is broken. They require immediate corrective
action and are typically binary: the cell either behaves as contracted or it does not.

**Purpose optimisation** — does the cell serve its business capability well?

* does the cell's output lead to good outcomes at the system level?
* is the cell making the right decisions, not just correct ones?
* is the AI component within the cell reasoning well?

Purpose failures indicate the cell's design is wrong. They require evolutionary
response — instructing the cell-bound AI to improve, or replacing the cell entirely.
They are typically continuous: the cell performs better or worse on a spectrum.

These dimensions require different corrective responses and different fitness
functions. Conflating them — treating a purpose failure as a functional failure or
vice versa — produces the wrong corrective action.

---

# Corrective Action Taxonomy

When cell performance degrades, corrective action is always applied at the cell
boundary. The two modes are:

**Instruct** — provide updated guidance to the cell-bound AI component within the
cell's own boundary. Refined context, updated examples, corrected constraints.
The cell interface does not change. The cell's internal reasoning improves.

Instruct is appropriate when:
* the cell is functionally correct but purpose-suboptimal,
* the AI component within the cell is reasoning from outdated or insufficient context,
* the improvement is incremental and the contract is sound.

**Replace** — deploy a new cell implementation that satisfies the same contracts.
The interface is unchanged. Downstream consumers are unaffected. The old cell is
retired.

Replace is appropriate when:
* the cell design is fundamentally wrong,
* a different model, algorithm, or approach is required,
* functional correctness cannot be achieved with the current implementation,
* or the cell has accumulated enough patches that replacement is cleaner.

Replace is the default corrective action. Instruct is a transitional mechanism.
A system that primarily relies on Instruct is accumulating implicit state within
cells rather than evolving through replacement.

Contracts are what make Replace safe. Without explicit contracts, replacement
carries undefined downstream risk.

See also: [Principle 16 — Cells Are Deployment Units](principles.md#16-cells-are-deployment-units--replace-dont-patch)

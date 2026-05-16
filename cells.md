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

See also: [Principle 13 — Cells Are Deployment Units](principles.md#13-cells-are-deployment-units--replace-dont-patch)

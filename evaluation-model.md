# AINBCAS — Evaluation Model

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

---

## Purpose

Cells are not evaluated once at deployment. They are evaluated continuously
against two distinct dimensions: whether they operate correctly, and whether
they serve their purpose well. These require different measurement methods,
different corrective actions, and different feedback loops.

Without a structured evaluation model, cell degradation is invisible until
it manifests as a system-level failure. By then, the cause is unattributable —
it could be the cell, the data, the model, or the environment.

---

## Two Dimensions of Cell Performance

### Functional Correctness

Does the cell operate as contracted?

- Outputs conform to the contracted schema
- Contracts are honoured for all input conditions
- Error conditions are handled as specified
- Observable diagnostics are emitted (Principle 8)
- Transformations are traceable (Principle 9)

Functional failures are binary: the cell either behaves as contracted or it
does not. They require immediate corrective action. They are detectable by
automated tests and fitness functions.

### Purpose Optimisation

Does the cell serve its position in the value stream well?

- Does the cell's output lead to good downstream outcomes?
- Is the cell making the right decisions, not just correct ones?
- Is the AI component reasoning well about its domain?
- Does the cell's behaviour reflect the intent of its value stream position?

Purpose failures are continuous: the cell performs better or worse on a
spectrum. They require evolutionary response — instructing the cell-bound AI
component or replacing the cell. They are detectable only through outcome
tracking and business-level fitness functions.

**Conflating these dimensions produces the wrong corrective action.** A cell
that outputs valid JSON but consistently recommends the wrong action has a
purpose failure, not a functional failure. Patching its schema validation will
not fix it.

---

## Fitness Function Levels

Every cell requires evaluation at all three levels.

### Fitness Functions

Fitness functions validate **architectural invariants** — structural properties
that must hold across any implementation of the cell. They are derived from
*Building Evolutionary Architectures* (Ford, Parsons, Kua): objective, measurable
assessments of architectural characteristics, not correctness tests of specific logic.

A fitness function fails when the architecture degrades, not when a bug is introduced.
If you rewrote the cell from scratch, a fitness function would still need to pass.

Examples:
- The contract surface is fully implemented (watchdog: `contractSurfaceCheck`)
- No implementation file imports across a cell boundary (watchdog: `importBoundaryCheck`)
- All inputs to a transformation are accounted for in the output — no silent drops
- An append-only data store has never had a record removed or modified
- A deterministic cell produces identical outputs for identical inputs on repeated execution
- A safety-critical gate (e.g. kill switch) is always evaluated before any other logic

Fitness functions are **automated, run on every commit, and block the pipeline on violation**.
Each must specify the mechanism that enforces it — a watchdog check, a CI script, or a
framework test. A fitness function with no enforcement mechanism is documentation, not governance.

### Logic Tests

Logic tests validate **implementation correctness** — that the current implementation
matches its logic-tree specification. They are derived directly from `logic-tree.md`:
every decision, formula, and routing rule stated there should have a corresponding
automated test that verifies the code matches the spec.

Logic tests fail when the code diverges from the logic tree, or when the logic tree
was wrong about what the code should do. They are closer to specification tests than
unit tests — the spec is the authority, not the test.

Examples:
- A scoring formula produces the values stated in the logic tree for known inputs
- A status machine rejects the invalid transitions listed in the logic tree
- A routing decision produces the outcome specified for each condition

Logic tests are **automated, run on every commit, and fail the pipeline on violation**.
They are organised by logic-tree section, not by file or function.

### Business Fitness Functions

Validate purpose optimisation against the cell's value stream position.

Examples across domains:

| Value Stream Position | Business Fitness Function |
|---|---|
| Belief formation | Does accumulated evidence lead to higher-quality decisions downstream? |
| Signal qualification | Do qualified signals have a higher outcome hit rate than unqualified ones? |
| Multi-perspective evaluation | Do operators make more confident decisions after seeing the evaluation? |
| Governance | What percentage of flagged items are acted on, and with what outcome? |
| Execution | What is the error rate and recovery time for executed actions? |

Business fitness functions require outcome data. They are measured over time
windows (days, weeks) rather than per-request. They feed back into cell purpose
assessment and are the primary signal for whether a cell needs replacement.

A cell with perfect technical fitness but poor business fitness is not a
functioning cell. It is a well-behaved cell doing the wrong thing.

---

## Replay and Evaluation

### Why Replay Matters

A cell that cannot be replayed cannot be evaluated. If you cannot reconstruct
what a cell decided, on what inputs, under what conditions, you cannot determine
whether a different implementation would have decided better.

Every cell must store enough observable evidence (Principle 8) to replay its
decisions post-hoc. This means:
- Inputs at the time of decision
- Outputs produced
- Internal reasoning metadata where AI is involved
- The model version and configuration in use at that time

Without replay capability, cell replacement is a guess. With it, replacement
is a measurement.

### Evaluation Protocol

When a cell's business fitness functions indicate degradation:

1. **Replay a representative sample** of recent decisions through the current
   implementation and record outputs.

2. **Identify failure patterns** — what classes of input produce poor outcomes?
   Is it systematic (the cell's logic is wrong) or distributional (inputs have
   shifted beyond the cell's designed operating range)?

3. **Determine corrective action** — instruct or replace (see Corrective Action
   Taxonomy in `cells.md`).

4. **For replacement:** generate a candidate cell from the cell's specification
   documents. Replay the same sample through the candidate. Compare outputs and
   fitness function scores.

5. **Promote or discard** the candidate based on measured improvement.

---

## The Cell Replacement Protocol

AINBCAS supports AI-assisted cell replacement as a first-class operational
capability, provided the cell's specification is complete (see Cell Specification
Completeness in `cells.md`).

### Preconditions

A cell can be replaced by AI-assisted generation only when all of the following
are true:

1. The cell's **value stream position** is explicitly documented in its CLAUDE.md
2. The cell's **capability statement** is unambiguous
3. The cell's **logic tree** is documented and current
4. The cell's **contract** is stable (no breaking changes pending)
5. **Technical fitness functions** exist and pass for the current implementation
6. **Business fitness functions** exist with measured baseline scores
7. **Replay data** is available for a representative decision sample

If any precondition is missing, replacement is not a generation exercise — it
is a rewrite with unknown correctness.

### Generation Process

1. Open a fresh AI session scoped to the cell's specification documents only:
   - `CLAUDE.md` (value stream position, capability, boundary)
   - `logic-tree.md` (decision structure, transformation rules)
   - `contracts/apis/{cell}-api.md` (interface specification)
   - `fitness-functions.md` (success criteria)
   - `tests/` (correctness verification)

   The AI session must not have access to the existing implementation.
   The implementation is what is being replaced, not the reference.

2. Instruct the AI to generate a complete implementation satisfying the spec.

3. Run all technical fitness functions against the generated implementation.
   All must pass before proceeding.

4. Replay the representative decision sample through the generated implementation.
   Compare output distributions against the baseline.

### A/B Validation

Before promoting a replacement, validate it under real traffic:

1. Deploy the replacement cell on a secondary endpoint.

2. Route a defined percentage of traffic to both implementations simultaneously
   (shadow mode: the primary implementation's response is used; the replacement
   runs in parallel and its output is recorded but not returned to the caller).

3. Compare outputs against business fitness function criteria over the validation
   window (typically 24–72 hours depending on decision frequency).

4. Promote the replacement if it matches or exceeds the baseline across all
   business fitness dimensions. Retire the original.

5. If the replacement underperforms, discard it. Investigate whether the
   specification is incomplete (logic tree is wrong or missing cases) or whether
   the AI generation approach needs refinement.

### What A/B Validation Requires

- Stable contracts (both implementations share the same interface)
- Observable outputs (Principle 8 — both implementations emit comparable diagnostics)
- A routing layer capable of shadow-mode traffic splitting (EA Cell responsibility)
- Defined business fitness function baselines to compare against

The routing layer is a platform concern, not a cell concern. Cells must not
implement their own A/B logic.

---

## The Feedback Loop

Evaluation data must flow back into the cell's specification, not just into
operational decisions.

When evaluation reveals a systematic purpose failure, the root cause is usually
one of:
- The logic tree is wrong (the cell was designed to make the wrong decisions)
- The logic tree is incomplete (the cell was not designed for inputs it is receiving)
- The value stream position has changed (the business context has shifted)

In each case, the specification must be updated before replacement is attempted.
A replacement generated from a wrong or incomplete spec will reproduce the same
failure.

**The specification is the system's long-term memory.** Implementation is
ephemeral — it is one realisation of the spec at a point in time. The spec
accumulates learning. The implementation is replaced.

---

## Relationship to Observability and Transformation Principles

Replay requires that Principle 8 (Observability) and Principle 9 (Observable
Transformation) are implemented, not aspirational.

A cell that does not emit what it received, what it produced, what it filtered,
and what it reasoned cannot be replayed. It cannot be evaluated. It cannot be
replaced with confidence.

Observability is not an operational nicety in AINBCAS. It is the prerequisite
for the entire evaluation and replacement model to function.

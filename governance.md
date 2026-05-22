# AINBCAS — Governance

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

---

## The Governance Problem in AI-Native Systems

Traditional software governance is primarily about access control, audit trails,
and policy enforcement. These concerns apply to AI-native systems, but are
insufficient.

AI-native systems introduce an additional governance problem: **AI outputs are
probabilistic and context-dependent.** A deterministic rule either fires or it
does not. An AI recommendation might be correct 80% of the time under normal
conditions and 40% of the time under distributional shift — and both cases produce
structurally identical outputs. The system cannot distinguish them without
independent verification.

AINBCAS governance addresses both concerns: the traditional (access, audit, policy)
and the AI-specific (recommendation authority, confidence calibration, human
override, model accountability).

---

## Principle: Deterministic Governance Overrides AI Interpretation

*(Principle 11)*

AI systems may interpret, summarise, classify, hypothesise, critique, or
recommend. AI systems must not:

- bypass governance without deterministic approval,
- directly mutate protected state,
- execute privileged actions without explicit human authority, or
- represent probabilistic outputs as authoritative decisions.

The governance boundary between AI recommendation and human authority must be
explicitly defined for every value stream. Implicit boundaries accumulate hidden
risk — they work until they don't, and failure is undetectable in advance.

---

## The Recommendation-Authority Boundary

Every value stream must define the point at which AI output transitions from
recommendation to action. This boundary is not a technical decision — it is a
business and risk decision that determines how much autonomous AI behaviour is
acceptable for a given consequence class.

A consequence class framework:

| Consequence Class | Description | Governance Model |
|---|---|---|
| Informational | Output is shown to a human. No direct action results. | AI output requires no approval gate |
| Advisory | Output informs a human decision that results in an action. | AI output requires human acknowledgement before action |
| Conditional | Output triggers an action if a deterministic rule is also satisfied. | AI output + rule both required; either can veto |
| Supervised | Output triggers a proposed action that requires explicit human approval. | Human must affirmatively approve before action executes |
| Restricted | Action class is prohibited for AI-initiated flows entirely. | Human-only; AI may provide supporting context but cannot propose |

Consequence classes must be assigned per action type in each value stream, not
per system. The same cell may produce informational output for low-stakes actions
and supervised output for high-stakes ones.

**No cell may self-assign a consequence class lower than the one designated for
its action type.** Consequence class is a governance decision, not a cell design
decision.

---

## Value Stream Governance

Governance applies at the value stream level, not only at individual cell
boundaries.

A value stream is a sequence of capability stages, each consuming the output of
the previous. Governance at the value stream level asks:

- Which stage transitions require human checkpoints?
- What accumulated evidence is required before the stream can advance to the
  next stage?
- What conditions trigger a halt — stopping the stream until a condition is
  resolved?
- What is the maximum autonomous progression before a human review point is
  required?

A system where every individual cell is well-governed but the value stream has
no checkpoints is still an ungoverned system. An action may be reached through
a sequence of small, individually-reasonable AI steps that a human would not
have approved as a whole.

Governance checkpoints must be placed at value stream transitions, not only at
execution points.

---

## Confidence, Evidence, and Decision Authority

### Confidence is not permission

A high-confidence AI output does not grant the AI permission to act autonomously.
Confidence is a property of the output's directional certainty (Principle 9).
Authority to act is determined by the consequence class of the action, which is
a governance decision independent of confidence.

A cell may emit high confidence and still require human approval before its
recommendation becomes an action. These are orthogonal properties.

### Evidence thresholds

Governance may require a minimum evidence threshold before a value stream can
advance. Evidence thresholds are defined in deterministic terms — specific types
of evidence, minimum counts, recency requirements — not in terms of AI confidence
scores.

This is because AI confidence scores can be inflated by [evidence inflation
(Anti-Pattern 8)](anti-patterns.md#8-evidence-inflation) — counting the same
source multiple times — and can be miscalibrated. Evidence thresholds based on identifiable, deduplicated source
records are auditable. Confidence-based thresholds are not.

### Conviction vs completeness

Governance rules must distinguish between:
- **Low conviction:** the evidence suggests the claim may not be true
- **Incomplete evidence:** insufficient evidence has been gathered to assess the claim

These require different governance responses. Low conviction may trigger a
hold — wait for more evidence, or escalate for human review. Incomplete evidence
triggers a data collection action — what additional evidence is needed, from
where, and who is responsible for obtaining it.

Governance systems that treat these as equivalent will generate incorrect
escalations and miss genuine risks.

---

## Deterministic Governance Mechanisms

Governance in AINBCAS must remain:
- **Observable** — every governance decision is logged with its inputs and outputs
- **Auditable** — governance decisions can be reconstructed after the fact
- **Deterministic** — given the same inputs, the same governance decision is always made
- **Testable** — governance rules can be verified with unit tests, independent of AI behaviour
- **Independent** — governance cells do not depend on the cells they govern

### Required mechanisms

**Approval gates:** Explicit checkpoints where human approval is required before
a value stream can advance. Approval gates are implemented as deterministic
state machines — not as AI reasoning. An approval gate must be able to halt a
value stream without AI involvement.

**Override audit:** Every human override of an AI recommendation is logged with:
the original AI recommendation, the override decision, the human identity, the
timestamp, and the stated reason. Overrides without stated reasons are rejected.
Override patterns are monitored as signals of AI purpose failure.

**Kill switches:** The ability to halt a cell's output from propagating through
the value stream, without requiring changes to the cell itself. Kill switches are
operator-controlled, deterministic, and must not require AI interpretation to
activate or deactivate.

**Exposure limits:** Deterministic caps on the cumulative consequence of AI-influenced
actions within a defined window. Once an exposure limit is reached, further
AI-influenced actions are halted regardless of AI recommendation or confidence.
Exposure limits are defined in business terms (not technical terms) and are
enforced by the governance cell, not by individual cells.

---

## AI Model Accountability

### Model identity is an audit requirement

Every AI output stored or acted upon must record:
- The model identifier (including version) that produced it
- The prompt or instruction set version
- The timestamp
- The input hash (to enable replay)

This is required for post-hoc evaluation, for identifying model-version-specific
failure modes, and for managing model deprecation.

### Model deprecation governance

When a model version is deprecated:

1. Identify every cell that uses the deprecated model.
2. For each cell, trigger the cell replacement protocol (see `evaluation-model.md`).
3. Do not patch cells to reference a new model version. Replace the cell with a
   new implementation that satisfies the same specification using the new model.
4. Validate the replacement through A/B testing before promoting.
5. Retire the old cell implementation.

Model deprecation is a cell replacement event, not a configuration change.
Changing a model reference inside a running cell without replacement is
[in-context patching (Anti-Pattern 12)](anti-patterns.md#12-in-context-patching).

---

## Governance at Different Scales

The mechanisms above apply regardless of organisational scale. A single-operator
system and a large enterprise face the same governance problems — the consequence
classes, evidence thresholds, and approval gates are the same — but the
implementation of human oversight differs.

In a single-operator system, the operator is the governance layer. The kill
switch, override audit, and approval gates are invoked by one person. The
governance model defines what that person must do, when, and with what recorded
rationale — even when there is no external auditor.

In a larger organisation, the same mechanisms distribute across roles: approval
gates may require specific role holders, override audits are reviewed by
independent parties, and exposure limits are set by risk functions separate from
the teams that operate the cells.

The standard defines the mechanisms. The scale of the organisation determines
how those mechanisms are staffed.

---

## Governance Fitness Functions

| Category | Fitness Function |
|---|---|
| Approval enforcement | No action in a supervised consequence class executes without a recorded approval event |
| Override audit completeness | 100% of human overrides have a recorded reason |
| Kill switch operability | Kill switch activation halts propagation within the defined SLA |
| Exposure limit enforcement | No action executes when the relevant exposure limit is reached |
| Model accountability | 100% of stored AI outputs have a model identifier and input hash |
| Evidence threshold compliance | No value stream advances past a checkpoint without satisfying its evidence threshold |
| Governance independence | Governance cell has no import dependency on any cell it governs |

---

## Relationship to Other Standard Documents

- **Cells** (`cells.md`) — corrective action taxonomy (instruct vs replace) applies
  when governance failures indicate cell purpose failure
- **Evaluation Model** (`evaluation-model.md`) — business fitness functions and
  A/B replacement are the mechanism for resolving AI model accountability concerns
- **Contracts** (`contracts/`) — governance cells must have explicit contracts;
  governance must not be implemented as shared logic inside non-governance cells
- **Decomposition Watchdog** (`decomposition-watchdog.md`) — governance cells are
  subject to the same boundary checks as all other cells; governance logic must
  not leak into non-governance cells

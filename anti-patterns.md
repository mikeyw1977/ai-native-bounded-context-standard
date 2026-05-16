# AINBCAS — Anti-Patterns

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

AINBCAS explicitly discourages the following patterns.

---

## Prompt Monoliths

Large shared prompts controlling multiple unrelated responsibilities.

---

## Hidden Coupling

Implicit behavioural dependencies between cells.

---

## Autonomous AI Authority

AI systems bypassing deterministic governance.

---

## Semantic-Free Decomposition

Creating services from technical enthusiasm rather than business semantics.

---

## Shared Prompt Architecture

Multiple cells relying on one evolving prompt context.

---

## Non-Testable Intelligence

Reasoning systems without replayability or evaluation.

---

## Silent Transformation

A cell or pipeline stage that transforms, filters, enriches, ranks, or suppresses
information without emitting observable evidence of what changed and why.

Silent transformation makes pipelines undiagnosable, models uncalibratable, and
systems untrustworthy. It is the mechanism by which failure becomes invisible until
it is too expensive to trace.

See [Principle 9 — Observable Transformation](principles.md#9-observable-transformation-principle).

---

## Evidence Inflation

Reprocessing the same source event multiple times without source identity tracking,
causing a single fact to be counted as multiple independent pieces of evidence.

Evidence inflation artificially increases conviction. A system that processes the
same filing, transaction, or document across multiple sessions without deduplication
will accumulate corroboration that does not exist. The corrective action — lowering
a threshold, reducing a position, triggering a review — is applied against a
confidence figure that is factually wrong.

Every evidence source must carry a stable identifier. Any cell that accumulates
evidence must track which source identifiers it has already incorporated and reject
resubmission of the same event.

---

## Confidence Conflation

Expressing data quality, completeness, or source reliability as a conviction score,
causing downstream synthesis to treat low-quality data as low-conviction evidence.

These are different conditions requiring different responses:

* low conviction: require more corroboration, apply lower weight, flag for review,
* low data quality: collect more data, defer decision, mark as incomplete.

Treating them as the same metric produces incorrect corrective actions. A source
must not emit a low confidence score when it means "I have incomplete data." It must
instead emit high-quality uncertainty metadata that distinguishes between "I have
seen this and doubt it" and "I have not seen enough to say."

See [Principle 12 — Evidence Quality and Conviction Are Distinct Properties](principles.md#12-evidence-quality-and-conviction-are-distinct-properties).

---

## Premature Boundary Extraction

Creating a separate cell, service, or deployment unit before operational evidence
justifies the separation.

Premature extraction adds deployment complexity, contract overhead, and operational
burden without the benefits of true cell independence. A cell extracted before its
contract is stable will require repeated breaking changes. A cell extracted before
its failure domain is understood creates operational risk without operational gain.

The fat-cell pattern is explicitly supported as a legitimate architectural starting
point. Extract boundaries when fitness-function evidence shows that the cost of
coupling exceeds the cost of separation.

See [Principle 10 — Implement Intent First, Extract Boundaries Later](principles.md#10-implement-intent-first-extract-boundaries-later).

---

## In-Context Patching

Modifying shared prompt context, global configuration, or session state to correct
the behaviour of a specific cell rather than replacing the responsible cell.

In-context patching accumulates hidden dependencies. Each patch makes the system
harder to reason about, the cell harder to replace, and the boundary between cells
less clear. Over time, the patched context becomes the architecture — undocumented,
untested, and load-bearing.

When a cell misbehaves, the correct response is corrective action at the cell
boundary: instruct the cell-bound AI component with updated guidance within its own
boundary, or replace the cell entirely with a new deployment. The shared context
must not become a repair surface.

See [Principle 13 — Cells Are Deployment Units](principles.md#13-cells-are-deployment-units--replace-dont-patch) and [Corrective Action Taxonomy](cells.md#corrective-action-taxonomy).

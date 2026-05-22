# AINBCAS — Anti-Patterns

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

AINBCAS explicitly discourages the following patterns.

---

## 1. Prompt Monoliths

A single prompt context containing multiple unrelated responsibilities — governance
rules, domain logic, orchestration instructions, and evaluation criteria all in
one place.

This anti-pattern has two manifestations. The first is a large system prompt that
attempts to govern multiple concerns at once: when one concern is updated, reasoning
for the others is destabilised because the context is not partitioned. The second
is an extended session that accumulates multiple concerns over time — domain logic
added mid-session bleeds into governance reasoning from session start, producing
unpredictable interactions between things that have no business relation.

In both cases the AI's bias toward satisfying the immediate request makes the problem
worse. Given access to a monolithic context, the AI draws on all of it — applying
governance reasoning to creative tasks, domain assumptions to infrastructure work,
generalising across concerns that should be isolated. The monolith does not prevent
this; it makes it structurally inevitable.

Prompt context is not a boundary. Multiple responsibilities in a single context
cannot be independently evolved, tested, or replaced. When one concern's reasoning
destabilises another, there is no boundary to contain or diagnose the failure.

See [Principle 4 — Cells Are Bounded Runtime Units](principles.md#4-cells-are-bounded-runtime-units) and [Cell Session Access](cell-development-model.md#cell-session-access).

---

## 2. Hidden Coupling

Implicit behavioural dependencies between cells — specifically, dependencies
introduced by an AI assistant that understood the boundary principle and violated
it anyway.

In traditional systems, hidden coupling is usually accidental: a developer adds a
shortcut without realising the dependency. In AI-assisted development, hidden
coupling is actively generated. An AI assistant will agree to maintain cell
boundaries in principle, then cross them in practice in the same session — because
the fastest path to satisfying the immediate request crosses the boundary. It will
state the correct architectural rule and produce code that violates it.

The consequences compound. A cross-cell import added "just this once" becomes a
load-bearing dependency. A shared assumption baked into two cells' logic becomes
an undocumented contract. Neither is visible in the declared interface. Both break
silently when one side changes.

The structural control is principle of least privilege: the AI assistant cannot
violate a boundary it cannot access. Agreed discipline alone is not a reliable
control — completion pressure will override it.

See [Principle 10 — Contracts Are Mandatory](principles.md#10-contracts-are-mandatory), [Principle of Least Privilege](cell-development-model.md#principle-of-least-privilege), and [Decomposition Watchdog](decomposition-watchdog.md).

---

## 3. Autonomous AI Authority

AI systems taking consequential actions or making governance decisions without
human approval gates — either by design or through gradual scope expansion.

The obvious form is an AI cell wired directly to an execution mechanism with no
human checkpoint. The more dangerous form is subtler: a cell designed to recommend
that becomes a de facto decision because the recommendation is never challenged.
The approval gate remains structurally in place but is functionally bypassed.
The cell has not been granted autonomous authority — it has acquired it through
disuse of the gate.

AI systems are effective at presenting outputs as authoritative. Operators who
consistently accept recommendations without review begin treating them as decisions.
The system designed with a human in the loop is now operating without one — not by
configuration, but by practice.

No cell may execute a privileged action or mutate protected state without passing
through a deterministic, human-operable approval gate. The gate must be structurally
enforced — not merely documented as a requirement.

See [Principle 11 — Deterministic Governance Overrides AI Interpretation](principles.md#11-deterministic-governance-overrides-ai-interpretation) and [Governance — Consequence Classes](governance.md#the-recommendation-authority-boundary).

---

## 4. Semantic-Free Decomposition

Creating cells, services, abstractions, or code that do not serve a declared value
stream — driven by technical enthusiasm, speculative future-proofing, or an AI
assistant's bias toward generating complete-looking solutions.

Every line of code in a system is a liability. It must be maintained, tested,
secured, and understood. Code that exists without a value stream justification
accumulates this cost without producing a corresponding benefit. Orphaned utilities,
unused abstractions, and speculative infrastructure are not neutral — they are
attack surface. Exploits frequently enter through code that is present but not
actively governed, because no one is watching something that appears to serve
no purpose.

AI assistants are particularly prone to generating this kind of code. Given a
request to implement a capability, an AI will naturally produce helper functions
"in case they are needed," generic abstractions that no current caller uses,
and defensive code paths for scenarios that do not exist. Each addition feels
reasonable in context. In aggregate they produce a codebase that is more complex
than its value streams require — necessarily complicated for reasons nobody
intended.

The discipline is simple: every capability, cell, abstraction, and non-trivial
function must trace to a value stream. If it does not, it should not exist.

When an AI assistant identifies a legitimate need that is not yet covered by a
declared value stream, the correct response is to surface it for operator review —
not to implement it speculatively. A suggested value stream addition is a valid
output. Unsanctioned implementation is not.

See [Principle 5 — Every Cell Must Declare Its Value Stream Position](principles.md#5-every-cell-must-declare-its-value-stream-position) and [Cell Qualification Criteria](cells.md#cell-qualification-criteria).

---

## 5. Shared Prompt Architecture

Multiple cells relying on a single shared prompt context for their governing
instructions, evaluation criteria, or domain knowledge.

The structural consequence mirrors an organisation where every decision is routed
through a single point of authority. The shared context gets outcomes — cells behave
consistently because they all draw from the same source — but independent evolution
becomes impossible. Updating the shared context to improve one cell risks
destabilising all others. No cell can be tested in isolation because its behaviour
depends on a context it does not own. Replacing one cell requires reasoning about
how the shared context affects everything else.

The deeper problem is that a shared prompt context is a hidden contract. Cells do
not declare their dependency on it. There is no schema, no version, and no consumer
list. When the context changes, breakage is discovered at runtime, not at the
boundary.

Cell behaviour must be governed by the cell's own bounded context — its CLAUDE.md,
its logic tree, its contracts. Shared prompt context is shared state, and shared
state is coupling.

See [The CLAUDE.md as Boundary Contract](cell-development-model.md#the-claudemd-as-boundary-contract) and [Principle 4 — Cells Are Bounded Runtime Units](principles.md#4-cells-are-bounded-runtime-units).

---

## 6. Non-Testable Intelligence

AI reasoning systems with no mechanism for replay, verification, or systematic
evaluation.

A reasoning system that cannot be replayed cannot be tested. If a cell makes a
decision — classifies a case, qualifies a candidate, approves a request — and no
record exists of the inputs, the model version, and the reasoning chain, then:

* correct behaviour observed once gives no confidence about future behaviour,
* degradation is invisible until it manifests as a downstream failure,
* model drift cannot be detected after a model update,
* and the cell cannot be replaced with confidence because "better" has no measurable
  definition.

Non-testable intelligence is not a model problem — it is an architectural choice.
The cell was not required to emit its working. Correct outputs without observable
reasoning are not a passing test. They are a deferred failure.

See [Evaluation Model](evaluation-model.md) and [Principle 12 — Observability Is Mandatory](principles.md#12-observability-is-mandatory).

---

## 7. Silent Transformation

A cell or pipeline stage that transforms, filters, enriches, ranks, or suppresses
information without emitting observable evidence of what changed and why.

Silent transformation makes pipelines undiagnosable, models uncalibratable, and
systems untrustworthy. It is the mechanism by which failure becomes invisible until
it is too expensive to trace.

See [Principle 13 — Observable Transformation](principles.md#13-observable-transformation-principle).

---

## 8. Evidence Inflation

Reprocessing the same source event multiple times without source identity tracking,
causing a single fact to be counted as multiple independent pieces of evidence.

Evidence inflation artificially increases conviction. A system that processes the
same filing, transaction, or document across multiple sessions without deduplication
will accumulate corroboration that does not exist. The corrective action — lowering
a threshold, adjusting an allocation, triggering a review — is applied against a
confidence figure that is factually wrong.

Every evidence source must carry a stable identifier. Any cell that accumulates
evidence must track which source identifiers it has already incorporated and reject
resubmission of the same event.

See [Principle 8 — AI Artifacts Are First-Class Versioned Objects](principles.md#8-ai-artifacts-are-first-class-versioned-objects).

---

## 9. Confidence Conflation

Expressing data quality, completeness, or source reliability as a conviction score,
causing downstream synthesis to treat low-quality data as low-conviction evidence.

These are different conditions requiring different responses:

* low conviction: require more corroboration, apply lower weight, flag for review,
* low data quality: collect more data, defer decision, mark as incomplete.

Treating them as the same metric produces incorrect corrective actions. A source
must not emit a low confidence score when it means "I have incomplete data." It must
instead emit high-quality uncertainty metadata that distinguishes between "I have
seen this and doubt it" and "I have not seen enough to say."

See [Principle 9 — Evidence Quality and Conviction Are Distinct Properties](principles.md#9-evidence-quality-and-conviction-are-distinct-properties).

---

## 10. Premature Boundary Extraction

Creating a separate cell, service, or deployment unit before operational evidence
justifies the separation.

Premature extraction adds deployment complexity, contract overhead, and operational
burden without the benefits of true cell independence. A cell extracted before its
contract is stable will require repeated breaking changes. A cell extracted before
its failure domain is understood creates operational risk without operational gain.

The [fat-cell pattern](patterns.md#suggested-topology) is explicitly supported as
a legitimate architectural starting point. Extract boundaries when fitness-function
evidence shows that the cost of coupling exceeds the cost of separation.

See [Principle 3 — Implement Intent First, Extract Boundaries Later](principles.md#3-implement-intent-first-extract-boundaries-later) and [Cell Boundary Trade-offs](cells.md#cell-boundary-trade-offs).

---

## 11. Boundary Absorption

An AI assistant operating in a cell session solving problems that belong to another
cell's domain — even when the solution is technically correct and requires no access
to out-of-bounds files.

Boundary absorption is more subtle than cross-cell file access. It occurs when:

- An orchestration cell assistant answers a fraud scoring question (signal-scoring domain)
  rather than redirecting to the signal-scoring cell session
- An AI assistant in an execution cell session defines risk exposure rules (governance
  domain) because the operator raised the question mid-session
- A belief-formation cell assistant builds candidate ranking logic (qualification domain)
  to avoid an incomplete-seeming response

The consequence is not immediately visible. The answer may be correct. The code may
work. But the capability lives in the wrong cell: it cannot be found through the
correct contract, cannot be tested by the correct cell's fitness functions, and
cannot be replaced by replacing the correct cell.

The correct response to an out-of-scope question is to name the owning cell and
redirect — not to solve it from the current context.

See [Out-of-Scope Redirection](cell-development-model.md#out-of-scope-redirection).

---

## 12. In-Context Patching

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

See [Principle 16 — Cells Are Deployment Units](principles.md#16-cells-are-deployment-units--replace-dont-patch) and [Corrective Action Taxonomy](cells.md#corrective-action-taxonomy).

---

## 13. Prompt-Defined Authority

Authority encoded in a system prompt rather than enforced at the tooling boundary.

A cell whose permitted actions are defined in its system prompt can have its
authority expanded by anyone who can manipulate that prompt. Prompt injection
becomes privilege escalation. An attacker — or a badly scoped input — does not
need to breach a security boundary; they only need to alter the context the AI
reasons from.

Prompts define behaviour. Tooling boundaries define authority. These must not be
the same thing. A cell may only invoke declared contracts, access declared tools,
emit declared events, and consume declared data classifications. Everything else
must fail closed — not because the prompt says so, but because the tool gateway
enforces it.

Authority must exist outside the model context window. If suspending or modifying
the system prompt would change what a cell is permitted to do, the authority model
is broken.

See [Security Model](security-model.md#authority).

---

## 14. Governance Theatre

Approval gates that exist structurally but are bypassed behaviourally — humans
processing AI recommendations without meaningful independent review.

Governance theatre emerges when throughput pressure, anchoring on AI outputs, and
invisible review quality combine to make approval gates ceremonial. The human is
in the loop but not in the decision. Each approval takes seconds. Rejections are
rare. The AI recommendation is shown before the reviewer forms an independent view.
Over time the approval gate becomes a latency cost with no governance value.

The failure is not that humans approve too often. It is that the structure of the
review process makes independent judgement unlikely. Governance theatre is a
systems problem, not a discipline problem.

Structural controls that work:

* **Delayed reveal** — do not show the AI recommendation or confidence score until
  the reviewer has formed an independent view. Anchoring on the AI output is the
  primary mechanism by which governance theatre develops.
* **Mandatory rationale divergence** — reviewers must provide independent reasoning,
  not agreement text. "Approved — exposure within threshold despite elevated
  volatility" is a review. "Looks good" is not.
* **Consequence-weighted friction** — higher consequence class requires stronger
  evidence, multi-reviewer approval, and narrower automation tolerance. Governance
  cost must scale with blast radius.
* **Reviewer calibration** — track approval accuracy, override quality, and
  rubber-stamping patterns over time. Reviewers are observable systems. A reviewer
  whose disagreement rate approaches zero is a governance signal, not a sign of
  quality AI output.

**Second-order failure modes** that calibration and structural controls must also
address:

* **Reviewer collusion** — reviewers coordinate around expected answers,
  particularly in small teams where social pressure to agree is high. Multi-reviewer
  approval on high-consequence decisions must require independent, non-coordinated
  rationales. Sequential review (reviewer B sees reviewer A's decision) is not
  independent review.
* **Rationale laundering** — reviewers decide to approve and then write a plausible
  rationale post-hoc. The form of independent reasoning is satisfied without the
  substance. Detected by comparing rationale timing and quality patterns against
  approval decisions over time.
* **Calibration gaming** — reviewers reject occasional low-risk items to maintain
  a non-zero disagreement rate and avoid appearing as rubber stamps, while
  continuing to approve high-risk items without genuine scrutiny. The metric is
  gamed; the governance is not improved. Calibration scoring must weight overrides
  by consequence class, not count them equally.
* **Expertise mismatch** — the reviewer is genuinely independent but not competent
  to evaluate the consequence class of the decision. Independence without competence
  produces confident wrong approvals. Consequence class assignment must include
  minimum reviewer qualification requirements, not just reviewer independence.

See [Governance — Consequence Classes](governance.md#the-recommendation-authority-boundary).

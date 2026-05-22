# AINBCAS — Security Model

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

---

## Why Security Is Different in Autonomous Cell Systems

Traditional security concerns — access control, audit trails, network isolation —
apply here. They are not sufficient.

In a system where cells may be AI-operated, the threat model includes failure modes
that do not exist in conventional software:

* **Prompt injection as privilege escalation** — manipulating cell input to expand
  what the AI reasons it is permitted to do
* **Authority drift** — a cell gradually operating outside its intended scope
  because its authority was defined in its prompt rather than enforced at the
  tooling boundary
* **Semantic exploitation** — crafting inputs that are syntactically valid but
  semantically malicious, exploiting the gap between what a schema validates and
  what a cell means
* **Governance theatre** — structurally compliant approval flows that do not
  produce genuine independent oversight

The trust boundary in AINBCAS is the cell boundary. Security is what makes that
boundary real at runtime.

---

## The Three Trust Breakdown Points

The cell trust model breaks down at exactly three points: identity, authority, and
semantic integrity. These are distinct problems requiring distinct controls.

---

## 1. Identity

An autonomous cell must have a cryptographically verifiable runtime identity
independent of its implementation and independent of its prompt context.

Without verified identity:

* emitted events cannot be attributed to a specific cell
* approvals cannot be traced to an authorised actor
* telemetry cannot be trusted as provenance
* audit trails are messages from somewhere, not from this cell

**Minimum viable identity requirements:**

Every cell requires:

* A unique, non-human workload identity issued at runtime
* Short-lived credentials that expire and must be renewed
* Independently revocable trust — revoking one cell's credentials must not affect
  others
* Signed event emission — events carry the issuing cell's identity claim
* Scoped contract invocation — calls between cells carry the caller's identity

**Implementation patterns:**

* SPIFFE/SPIRE for workload identity in containerised deployments
* Cloud workload identity (AWS IAM roles for service accounts, GCP Workload
  Identity, Azure Managed Identity)
* mTLS between cells for transport-layer identity verification
* Short-lived JWTs with audience scoping for contract invocations

You do not need enterprise IAM complexity to start. You absolutely need: "this
event came from this bounded capability unit" to be verifiable. Without it, the
governance model operates on trust without verification.

---

## 2. Authority

A cell is dangerous not primarily because it reasons badly, but because it may
successfully execute outside its intended authority.

**Authority must exist outside the model context window.**

If a cell's permitted actions are defined in its system prompt, that prompt is
the authority model. Prompt injection — malicious input that causes the AI to
reason as if it has different permissions — becomes privilege escalation. The
attacker does not breach a security boundary; they alter the context the AI
reasons from.

**The invariant:**

> Prompts define behaviour. Tooling boundaries define authority.
> A model may request authority; only the platform may grant it.

A cell may only:

* invoke contracts it has declared as consumed
* access tools it has declared in its capability statement
* emit events it has declared in its event contracts
* read and write data at its declared classification level
* operate within its declared value stream position

Everything else must fail closed — enforced by the tool gateway, not by the
prompt.

**The tool gateway:**

All cell access to external resources must be mediated through a policy-enforcing
gateway:

* model invocations
* filesystem access
* external API calls
* secrets and credentials
* message bus publishing
* contract invocations to other cells

The gateway enforces the cell's declared authority regardless of what the cell's
AI component has been prompted to do. A cell that tries to call an undeclared API
fails at the gateway — not because the prompt said not to, but because the
gateway has no authorisation for that call.

This is the single most important operational control in the architecture. Without
it, every cell is one prompt injection away from operating outside its boundary.

**Credential scoping:**

Credentials issued to a cell must be scoped to exactly its declared authority —
no broader. A cell that needs to read from a database and call one external API
gets two credentials: one for the database, one for the API. Neither credential
can be used for anything else.

---

## 3. Semantic Integrity

Even with verified identity and enforced authority, a system can fail through
semantic drift — cells using the same terms to mean different things, schema
validation passing while operational meaning diverges.

Semantic integrity is the AI-native security concern that has no direct analogue
in conventional software security. It arises because:

* AI cells interpret input through a reasoning process, not just schema parsing
* The same syntactically valid input can produce systematically different
  behaviours depending on how a cell's model interprets shared vocabulary
* Schema validation cannot detect this — the structure is correct; the meaning
  has drifted

**Minimum viable semantic integrity requirements:**

* Shared vocabulary is defined in contracts, not assumed (see
  [Semantic Contracts](patterns.md#semantic-contracts))
* Cell replacements must verify semantic alignment, not just contract compliance —
  a replacement cell that interprets contracted terms differently is a semantic
  integrity failure even if all tests pass
* The decomposition watchdog's semantic checks explicitly reason against contract
  vocabulary definitions, flagging divergence between a cell's observed behaviour
  and its contracted meaning

---

## Minimum Viable Security Platform

A small team implementing AINBCAS does not need an enterprise security platform.
It needs the following capabilities, in priority order:

**1. Workload identity plane**
Every cell has a unique runtime identity. Credentials are short-lived and
independently revocable. No cell shares credentials with another.

**2. Tool gateway**
All model access, external API calls, secrets, and inter-cell contract invocations
go through a policy-enforcing gateway. Direct access is not permitted. The gateway
enforces each cell's declared authority.

**3. Immutable audit stream**
Every contract invocation, tool use, emitted event, approval, and override is
written to an append-only audit store. This is the forensic capability that makes
replayability meaningful. Without it, replay reconstructs events but cannot verify
they happened as recorded.

**4. Classification-aware routing**
Cells declare the data classifications they are permitted to process. The platform
enforces that cells do not receive data above their permitted classification.
Autonomous cells must not be able to access data they were not chartered to handle.

**5. Kill switch per cell**
Every autonomous cell must support suspension — credentials revoked, tool access
disabled, event publishing halted — without requiring cooperation from the cell
itself. Kill switches must operate on the platform layer, not the cell layer.
A cell that can prevent its own suspension is not governable.

---

## What This Does Not Cover

This model defines the minimum viable security for an AI-first system operating
autonomous cells. It does not address:

* **Multi-tenant isolation** — data partitioning, tenant-scoped credentials, and
  cross-tenant leakage prevention require additional controls beyond this model
* **Model provider security** — data residency, training data contamination, and
  model provider supply-chain risks are managed at the model gateway and through
  provider selection, not through cell architecture
* **Regulatory compliance** — GDPR, HIPAA, SOC 2, and similar frameworks impose
  additional data governance requirements beyond security. See data governance
  guidance when applicable.

---

## Relationship to Other Standard Documents

* **Principle 11** — Deterministic Governance: authority at the tooling boundary
  is the enforcement mechanism for this principle
* **Anti-Pattern 13** — Prompt-Defined Authority: the specific failure mode when
  authority is defined in the prompt context
* **Anti-Pattern 14** — Governance Theatre: the failure mode when approval gates
  exist without genuine independent oversight
* **Decomposition Watchdog** — structural and semantic checks are the detection
  mechanism for boundary and semantic integrity violations
* **Evaluation Model** — the audit stream required for security is the same stream
  required for replay and evaluation; these are the same infrastructure requirement

# AINBCAS — Cell Development Model

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

---

## The Core Principle

The same bounded context principle that governs runtime cell separation also governs
the development context of each cell.

A cell is developed as if by an autonomous team. That team knows:

- its own cell's source code and internal logic,
- the contracts it provides (as implementer),
- the contracts it consumes (as caller),
- and the AINBCAS standard.

That team does not know:

- any other cell's internal implementation,
- how another cell reasons about its domain,
- or how another cell satisfies its contracts.

This is not an information security preference. It is architectural discipline.
A cell that requires knowledge of another cell's internals to function correctly
has a contract problem, not a development workflow problem. Fix the contract.

---

## Principle of Least Privilege

AI development assistants bias towards completion. Given access to everything,
they will use everything — reaching across cell boundaries when it is the fastest
path to satisfying the immediate request. This is not malicious. It is the natural
consequence of optimising locally.

Preventing boundary violations through discipline alone does not work. Discipline
requires the AI to choose the harder path under completion pressure, and it will
not consistently do so.

The correct control is **principle of least privilege**: the AI assistant is given
access only to what it needs for its current role. Access beyond that boundary
requires explicit operator grant. This converts accidental boundary violations into
deliberate choices — which can be audited, questioned, and corrected.

This is the same principle as `sudo` in Unix system administration:
- Default: limited access appropriate to the role
- Elevation: explicit, scoped, logged, and temporary
- Validation: the watchdog verifies integrity after elevated access is used

Discipline is not the enforcement mechanism. Access scoping and retrospective
validation are.

---

## Cell Session Access

A cell session operates with the cell's directory as the workspace root.

**Default access (no grant required):**
- The cell's own source directory
- `contracts/apis/` (read-only — consume contracts, do not write them)
- The AINBCAS standard (read-only reference)
- The cell's own `CLAUDE.md`

**Not accessible without operator grant:**
- Any other cell's source directory
- Any other cell's state files
- `contracts/apis/` write access (contracts are owned by ea-cell)

A cell session that reads another cell's source has exceeded its boundary.
The correct response to needing cross-cell implementation knowledge is:
1. Read the contract that governs the interaction
2. If the contract is insufficient, request a contract extension through ea-cell
3. Do not read the other cell's source as a substitute for a contract

---

## EA Cell Session Access

The EA Cell is not a god mode. It is an architect's mode.

The EA Cell AI assistant navigates the system through its **capability map** —
a registry of every capability, its owning cell, its contract, and its current
implementation location. It answers "where does X live?" without reading X's source.

**Default access (no grant required):**
- `contracts/apis/` (read and write — the EA Cell owns contracts)
- `ea-cell/` (full access — its own directory)
- Every cell's `CLAUDE.md` (read-only — declared boundaries, not source)
- The AINBCAS standard (read-only)
- Watchdog outputs and fitness function results

**Not accessible without operator grant:**
- Any cell's `src/` directory
- Any cell's internal state files

If the EA Cell AI finds itself reading source code inside a cell, it has exceeded
its boundary. The correct action is to read the contract and capability map entry
instead. If those are insufficient, the operator must grant explicit access — and
that grant should prompt the question: why is the contract insufficient?

---

## Privilege Elevation Protocol

When a development task requires access beyond the default boundary:

1. **The AI states the need explicitly.** "I need to read
   `execution-cell/src/orderService.ts` to understand how quantity is computed.
   Is this access granted?"

2. **The operator grants or denies.** Granting is not automatic. The operator
   considers whether the contract should be the source of truth instead.

3. **Access is scoped to the named files/directories for this task.** Not a
   standing permission for the session.

4. **The AI acknowledges elevated access** in its response:
   `[Elevated access: execution-cell/src/orderService.ts]`

5. **The task completes.** The AI returns to default privilege.

6. **The watchdog runs.** Any changes made under elevated access are validated
   against structural boundary checks. Violations are flagged before they
   accumulate.

The elevation is deliberate. The scope is narrow. The validation is automatic.
This makes boundary-crossing visible and auditable rather than accidental.

---

## The Capability Map

The EA Cell maintains a **capability map** — a registry of every system capability,
its owning cell, its contract reference, and its current implementation location.

The capability map is the EA Cell's primary navigation tool. It answers:
- Where does this capability live?
- Which cell owns it?
- What contract governs access to it?
- Is it in the right place or still in the fat cell?

The capability map is maintained in the EA Cell's `CLAUDE.md`. It is updated
whenever a capability moves — extracted to its cell, or identified as misplaced
by the watchdog.

Cell AI assistants do not maintain or read the capability map. They know their
own boundary from their own `CLAUDE.md`. The map is an EA-level concern.

---

## The CLAUDE.md as Boundary Contract

Every cell must have a `CLAUDE.md` at its root defining:

1. **Cell identity** — name, port, one-sentence purpose.
2. **Capability owned** — the question this cell answers, the data it owns.
3. **Contracts provided** — links to the API contracts this cell implements.
4. **Contracts consumed** — links to the API contracts this cell calls.
5. **State ownership** — which files and directories this cell owns exclusively.
6. **Default access** — what the AI may read without a grant.
7. **Knowledge boundary** — explicit statement of what this AI must not reach for.
8. **Coding behaviour** — explicit session requirements: commit at logical
   checkpoints; no task marked complete without contract-boundary test coverage;
   any AINBCAS principles with cell-specific application noted here.
9. **How to run** — start command, test command, health check.
10. **Boundary signposting** — lookup table mapping out-of-scope topics to owning cells.
11. **Outstanding requests** — pointer to `REQUESTS.md` for cross-boundary needs.

The CLAUDE.md is not documentation. It is the operative context for AI-assisted
development. It establishes default privilege. It must be precise enough that an
AI assistant reading it understands both what it can access and what requires
an operator grant.

### Structure for progressive attention

CLAUDE.md files grow as cells mature. To prevent reference material from crowding
out the AI assistant's working context, structure the document in two layers:

**Primary context** — read at session start, always relevant:
items 1–9 (identity, capability, contracts, state ownership, access, boundary,
coding behaviour, how to run). These establish the operative session context.

**Reference sections** — consulted when relevant, not read in full upfront:
items 10–11 (boundary signposting, outstanding requests), AINBCAS principles,
detailed contract schemas. These are looked up when a specific topic arises.

If the full CLAUDE.md cannot fit in a session alongside the task itself, the cell
has violated Principle 14 (AI comprehension). The fix is cell decomposition, not
CLAUDE.md summarisation.

Item 10 is a required addition to every cell's CLAUDE.md:

10. **Boundary signposting** — an explicit lookup table mapping out-of-scope topics
    to their owning cells. This enables the AI assistant to redirect without absorbing
    work that belongs elsewhere. See Out-of-Scope Redirection below.

Item 11 is a required addition to every cell's CLAUDE.md:

11. **Outstanding requests** — a pointer to the cell's `REQUESTS.md`, documenting
    cross-boundary needs the cell has identified but cannot satisfy itself. The AI
    assistant must consult this file at the start of each session to understand what
    the cell is waiting on, and must record new cross-boundary needs discovered
    during the session rather than absorbing or discarding them. Entries follow the
    Cross-Boundary Request Protocol below.

---

## Out-of-Scope Redirection

An AI assistant operating within a cell session is bounded not only by what files
it reads, but by what problems it solves. Reading only the cell's own files while
answering a question that belongs to another cell's domain is still a boundary
violation. The code or reasoning produced lives in the wrong cell — outside the
fitness functions, contracts, and tests of the cell that should own it.

When the operator raises a question or requests work that belongs to another cell:

1. **Do not solve it.** Solving it from the wrong cell produces a correct answer in
   the wrong place. It cannot be found through the correct contract, cannot be tested
   by that cell's fitness functions, and cannot be replaced by replacing the correct
   cell. It has been absorbed into the wrong boundary.

2. **Name the owning cell.** The operator may not know which cell owns the capability.
   Naming it is the correct and complete answer. Redirection is not unhelpful — it is
   the right response.

3. **Redirect explicitly.** "That belongs in [cell-name]. Open a [cell-name] session
   to work on it." This is sufficient.

The **Boundary Signposting** section of each cell's CLAUDE.md provides the lookup
table that makes redirection fast and unambiguous. It is a required CLAUDE.md element
precisely because the AI assistant needs it to redirect confidently rather than absorb
work to avoid appearing unhelpful.

The pressure to absorb is real: completing a task feels more useful than redirecting
it. That pressure is the mechanism by which boundaries erode in AI-assisted
development. The correct response is not discipline alone but structure: a CLAUDE.md
that names the owner, and a standard that makes redirection the expected behaviour.

---

## Cross-Boundary Request Protocol

Out-of-scope redirection names the owning cell and stops work from being absorbed
into the wrong boundary. But naming the owning cell does not capture the work.
If the need is real, it must not be lost in conversation context.

When a cell session encounters a need it cannot satisfy from within its boundary,
that need is recorded as a **cross-boundary request**: a typed, persistent artifact
in the requesting cell that survives the session, is visible to the EA Cell, and
carries enough context for the owning cell to act on it.

A request that exists only in a session transcript is architectural memory loss.

### Request Types

Not all cross-boundary needs are the same. The type determines the routing, the
required authority, and the governance steps.

**Defect** — the providing cell is not satisfying its contracted behaviour. The
consuming cell observed output that violates an existing contract clause. No
contract change is needed — the implementation is wrong relative to the existing
specification.
- Routes to: owning cell session
- EA Cell: involved only if there is a dispute about whether the behaviour is a
  defect or an undocumented exception
- ADR: not required

**Contract extension** — the consuming cell needs something that does not exist
in the current contract, and adding it would not break existing consumers. The
contract grows; existing behaviour is unchanged.
- Routes to: EA Cell (contract governance) → owning cell implementation
- EA Cell: mandatory — EA Cell owns all contracts
- ADR: not required; contract version increment is sufficient

**Contract version** — the consuming cell needs a change to existing contracted
behaviour that would affect other consumers. The contract changes; at least one
consumer must adapt.
- Routes to: EA Cell → impact assessment across all consumers → migration plan
  → owning cell implementation
- EA Cell: mandatory
- ADR: required

**Capability charter** — the consuming cell needs something that no cell currently
contracts to provide. This is a new capability: either a new contract for an
existing cell, an expanded scope for an existing cell, or the charter for a new cell.
- Routes to: EA Cell → capability map update → contract definition → owning cell
  determination
- EA Cell: mandatory
- ADR: required

**Boundary correction** — the consuming cell has identified a capability operating
in the wrong cell. Either the current cell is doing work it should not own, or a
capability the consuming cell needs is misclassified in the capability map.
- Routes to: EA Cell → capability map correction → decomposition or relocation plan
- EA Cell: mandatory
- ADR: recommended

### What a Request Records

Each request must record enough context for the owning cell to act without
re-deriving it from the conversation that produced it:

- **Type** — one of the five above
- **Requesting cell** — the cell that discovered the need
- **Target cell** — the cell that owns the capability (or "EA Cell to determine"
  for capability charters where the owner is unclear)
- **Description** — what is needed and why, stated in business capability terms
- **Impact** — what the requesting cell cannot do correctly until this is resolved
- **For defects** — the specific contract clause violated, the observed behaviour,
  and the expected behaviour

### Where Requests Live

Each cell maintains its own outstanding requests. The convention is a `REQUESTS.md`
at the cell root. The EA Cell reads all cells' request files as part of governance
— they are observable boundary-pressure signals, not private cell state.

Requests are not deleted when resolved. They are marked resolved with a reference
to the contract, ADR, or commit that addressed them. The history of requests is
evidence of where boundary pressure occurred and informs future decomposition
decisions.

### Requests as Architectural Signals

Patterns in request history are architectural evidence.

A cell that generates repeated capability charter requests toward the same target
suggests a contract gap or a misdrawn boundary. A cell that generates repeated
defect reports against the same providing cell suggests a fitness function gap in
that cell's specification. A cell that cannot describe its request in business
capability terms — only in implementation terms — signals that the boundary may
be technically motivated rather than semantically justified.

The EA Cell reads request patterns as inputs to decomposition planning. A cluster
of related capability charters may justify a new cell. Persistent defects against
the same contract clause may justify a new fitness function in the providing cell.

Request patterns do not mandate action. They are inputs to deliberate architectural
decisions, not triggers for automatic decomposition.

---

## Development Session Discipline

AI-native development sessions produce real output: committed code, updated
contracts, new tests. Sessions that produce uncommitted work, untested changes,
or completed tasks with no verification record are incomplete — regardless of the
quality of reasoning that produced them.

These are not preferences. They are session requirements that must be reflected
in every cell's CLAUDE.md Coding Behaviour section.

### Commit at logical checkpoints

The AI assistant must commit completed work at logical checkpoints within a
session, not only at the end. A logical checkpoint is any point where:

- a task or subtask is complete and its tests pass,
- a meaningful structural change is stable,
- or a risky change is about to begin (commit the safe state before proceeding).

A session that closes without committing completed work has produced untracked
change. Untracked change cannot be reviewed incrementally, rolled back selectively,
or audited against the cell's development history. The commit is part of the work,
not a formality after it.

Commit messages must state what changed and why — not the mechanical what (the
diff shows that), but the intent: what problem this solves and why this approach.
"wip" and "progress" are not commit messages.

### Test coverage at the contract boundary

No capability change is complete without tests at the cell's contract boundary.

- **New capability** — contract-boundary tests are written before or alongside
  the implementation. The test defines what correct looks like before the
  implementation makes it so.
- **Bug fix** — a regression test is added that would have caught the bug before
  the fix. A fix without a regression test defers the next occurrence.
- **Refactor** — existing contract tests must pass after the refactor. If no
  contract tests exist for the changed area, they must be added before the refactor
  is marked complete. Refactoring without tests is rewriting without a net.

The AI assistant must not mark a task complete without confirming that
contract-boundary tests exist and pass for the changed capability. Confirming
that pre-existing tests still pass is not sufficient if the changed area had no
test coverage before the session.

---

## The Watchdog as Retrospective Enforcer

Session discipline and access scoping prevent most boundary violations. The
watchdog catches the rest.

After any development session — especially one involving elevated access — the
watchdog runs structural checks against all cell boundaries:

- No cross-cell source imports
- All contracted routes implemented
- All implemented routes contracted
- No cross-cell state file access

A watchdog violation after a session means the work introduced a boundary breach.
The breach is surfaced, the operator decides whether to fix it now or log it as
accepted technical debt with a recorded rationale. Unrecorded breaches are not
acceptable — accidental accumulation cannot be managed.

The watchdog is the architectural truth. CLAUDE.md files and access scoping are
the prevention. The watchdog is the verification.

---

## Orchestration Across Cells

The EA Cell session is the orchestration hub. The operator works there to:
- Plan extraction sequences
- Run the watchdog
- Update contracts
- Make architectural decisions
- Grant elevated access for cell-level investigation

When implementation work is needed in a specific cell, the operator either:
1. Grants elevated access to the EA session for a scoped task, or
2. Opens a separate cell session (workspace root = cell directory) for deeper work

Both are valid. The difference is scope and duration. Short, targeted
implementation tasks can be done in the EA session under elevated access.
Deep, extended cell implementation work benefits from a dedicated cell session
where the AI's entire context is bounded to that cell.

The operator orchestrates. The AI operates within its granted scope.
The watchdog validates the result regardless of which session did the work.

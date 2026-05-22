# AINBCAS — Decomposition Watchdog

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

---

## Why This Exists

Architecture drifts towards entropy. This is not a failure of intent — it is a
consequence of how development pressure operates in practice.

Every development session, human or AI-assisted, optimises locally. The immediate
request is completed. The path of least resistance is taken. The path of least
resistance is almost always "add it to what already exists." Over many sessions,
this accumulates. Boundaries erode. Cells absorb responsibilities they were not
chartered to own. Contracts diverge from implementations. The fat cell re-grows
inside the extracted cells.

This force is not eliminated by CLAUDE.md files, cell charters, or good
intentions. Passive guidance does not hold against active completion pressure.

The decomposition watchdog exists to make boundary drift continuously visible
and automatically blocking. Architecture must be enforced, not just stated.

---

## What It Detects

The watchdog operates at two levels: structural and semantic.

### Structural Checks (deterministic, fast, run on every commit)

These checks are binary. They either pass or fail. They require no AI reasoning.

**1. Import boundary violation**

A cell must not import source code from another cell's `src/` directory.
Shared types and schemas must come from `contracts/` only.

```
VIOLATION: hypothesis-cell/src/server.ts imports from market-data-cell/src/
VIOLATION: debate-cell/src/runDebate.ts imports from ../market-explorer/src/debate/
```

Detection: static analysis of import statements across all cell directories.

**2. Undocumented route**

A cell must not expose HTTP routes that are not defined in its contract.
Every route that another cell or the operator calls must be contracted.

```
VIOLATION: market-data-cell serves GET /internal/cache-stats (not in market-data-api.md)
           and ea-cell calls it
```

Detection: extract routes from server.ts, compare against contracted endpoints.

**3. Missing contracted route**

A cell must implement every route defined in its contract.

```
VIOLATION: debate-api.md defines POST /debate/{theme} but debate-cell has no handler
```

Detection: extract contracted routes from API contract markdown, verify against
server implementation.

**4. Cross-cell state access**

A cell must not read or write another cell's state directory directly.
All cross-cell data access must go through the contract API.

```
VIOLATION: portfolio-cell reads ../hypothesis-cell/data/hypotheses.json directly
VIOLATION: debate-cell writes to market-data-cell/data/structural-candidates-*.json
```

Detection: scan file system access calls (fs.readFile, fs.writeFile) in each
cell, verify paths are within that cell's declared state directory.

**5. Contract consumer not calling contracted endpoint**

A cell that declares it consumes a contract must call the contracted endpoint,
not an internal function or direct file read.

```
VIOLATION: debate-cell declares it consumes hypothesis-api.md
           but calls hypothesisStore.getAll() directly instead of GET /hypotheses
```

Detection: compare declared consumed contracts against actual HTTP calls in source.

---

### Semantic Checks (AI-assisted, run on pull request or scheduled)

These checks require reasoning about purpose and intent. They use the EA Cell's
AI assistant, scoped to governance context only — contracts, cell charters,
the AINBCAS standard. The AI does not read cell internals to perform these checks;
it reads the diff and reasons against the chartered boundaries.

**6. Responsibility scope violation**

New code added to a cell is analysed against that cell's chartered capability.
Flag when the new code answers a question that belongs to a different cell.

```
WARNING: ea-cell/server.ts — new endpoint GET /api/portfolio/positions
         This capability is chartered to portfolio-cell (portfolio-api.md).
         Adding it to ea-cell creates an undeclared duplicate implementation.

WARNING: market-data-cell — new function computePersonaDebateContext()
         Debate context assembly is chartered to debate-cell.
         This function's purpose belongs across the boundary.
```

The AI does not block on these — it flags and requires operator decision.
Sometimes a temporary implementation in the wrong cell is the right tradeoff
(transition period, urgency). But the decision must be explicit, not accidental.

**7. Fat cell re-growth**

After a cell is extracted, the fat cell must not re-absorb its capability.
Flag when new code in the fat cell duplicates logic that an extracted cell owns.

```
WARNING: market-explorer/src/api/server.ts — new route POST /api/orders/buy
         execution-cell owns order placement (execution-api.md).
         This appears to duplicate execution-cell's chartered capability.
         If this is a proxy, declare it. If it is a re-implementation, remove it.
```

**8. Contract drift**

Flag when an implemented response shape diverges from the contracted schema.
This requires comparing actual serialised response objects against the contract
field definitions.

```
WARNING: market-data-api.md defines structural.atrPct as number | null
         market-data-cell returns structural.atrPct as string in some responses
         Contract and implementation have diverged.
```

---

## Where It Lives

The decomposition watchdog is an EA Cell capability.

It is not part of any individual cell. Individual cells cannot override or bypass
it. It reports to the operator through the EA Cell's governance interface.

The watchdog has two operational modes:

**CI mode** — structural checks only, runs on every commit, fails the pipeline on
any violation. Fast. Deterministic. No AI required. Every cell's CI pipeline
includes a watchdog step.

**Governance mode** — full checks including semantic analysis, runs on pull request
or on a scheduled cadence. Flags violations for operator review. Does not
automatically block — the operator decides whether a semantic violation is
acceptable in context. Records the decision.

---

## On AI-Assisted Development and Drift

AI development assistants bias towards completion. Given access to everything,
they will use everything — reaching across cell boundaries when it is the fastest
path to satisfying the immediate request.

The CLAUDE.md establishes context at session start. It does not prevent mid-session
drift under completion pressure. Discipline alone is not a reliable control.

The correct architecture is **principle of least privilege** (see Cell Development
Model): the AI assistant is given access only to what its role requires. Boundary
violations require elevated access that the operator must explicitly grant. This
converts accidental violations into deliberate choices.

The watchdog is the retrospective validation layer — the check that runs after
every session to verify that access controls held and no violations slipped through.
It is not a substitute for access scoping. It is the verification that access
scoping worked.

**The discipline the watchdog enforces:**

Every boundary violation must be a deliberate decision with a recorded rationale.
"Added to ea-cell temporarily — hypothesis-cell extraction pending, ADR-012" is
acceptable. "It ended up in ea-cell and nobody noticed" is architectural entropy.

The watchdog converts the second case into the first by making violations visible
before they accumulate.

The watchdog converts accidental drift into deliberate choice. Deliberate choices
can be revisited and corrected. Accidental accumulation cannot, because it has
no recorded rationale to reason against.

---

## Implementation

The structural checks are implemented as scripts in `ea-cell/src/watchdog/`:

```
ea-cell/
  src/
    watchdog/
      importBoundaryCheck.ts      ← check 1
      contractSurfaceCheck.ts     ← checks 2, 3
      stateOwnershipCheck.ts      ← check 4
      consumerComplianceCheck.ts  ← check 5
      index.ts                    ← runs all structural checks, exits non-zero on violation
```

Each check emits structured output:

```json
{
  "check": "importBoundaryCheck",
  "status": "VIOLATION",
  "cell": "debate-cell",
  "file": "src/runDebate.ts",
  "detail": "imports from ../market-explorer/src/debate/personas.ts",
  "severity": "BLOCKING"
}
```

Semantic checks are implemented as a separate script that calls the EA Cell's
AI assistant with governance context:

```
ea-cell/
  src/
    watchdog/
      semanticCheck.ts    ← check 6, 7, 8 — AI-assisted, non-blocking
```

The CI integration:

```bash
# In each cell's CI pipeline:
pnpm --filter ea-cell watchdog:structural --cell hypothesis-cell

# In the EA Cell's governance pipeline (scheduled or PR-triggered):
pnpm --filter ea-cell watchdog:semantic --diff HEAD~1..HEAD
```

---

## Relationship to Fitness Functions

The decomposition watchdog is a category of fitness function (Principle 14).

It validates:
- boundary integrity (structural checks)
- contract compatibility (surface checks)
- deployment isolation (state ownership checks)

These overlap with the fitness function categories in `governance.md`. The
distinction is operational: fitness functions validate system-level properties
(the system behaves correctly). The watchdog validates structural properties
(the system is built correctly).

Both are mandatory. Neither substitutes for the other.

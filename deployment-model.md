# AINBCAS — Deployment Model

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

---

## Principle

A cell is a deployment unit. Its deployment lifecycle must be independent of every
other cell. This is not a preference — it is the operational expression of
Principle 16 and Principle 4. A cell that cannot be deployed without coordinating
with another cell has a hidden dependency that the contract surface does not
express.

---

## Repository Model

AINBCAS does not mandate polyrepo or monorepo. Both are compliant if cell
deployment independence is preserved. The decision is operational, not
architectural.

### Monorepo (recommended during incubation)

All cells live in a single repository as top-level directories. The contracts
package is a local workspace reference. CI pipelines are path-scoped so changes
to one cell's directory trigger only that cell's pipeline.

```
ecosystem/
  contracts/          ← shared workspace package, versioned internally
  ea-cell/
    CLAUDE.md
    package.json
    .github/workflows/ea-cell.yml
  hypothesis-cell/
    CLAUDE.md
    package.json
    .github/workflows/hypothesis-cell.yml
  market-data-cell/
    CLAUDE.md
    package.json
    .github/workflows/market-data-cell.yml
  ...
```

This is the correct starting point. Cell boundaries are actively being proven.
Extraction is still happening. Polyrepo overhead is not yet justified.

### Polyrepo (target state for ACTIVE cells)

Each cell is its own git repository. The contracts package is published to a
registry (npm, private, or a git submodule) and pinned by version in each cell's
`package.json`. A breaking contract change requires a version bump, a published
release, and a coordinated update across all consuming cell repositories.

Polyrepo graduation is required when a cell reaches **ACTIVE** status in the
capability map — meaning the capability is fully implemented in the cell with no
fat-cell fallbacks. An ACTIVE cell that remains in the monorepo indefinitely is a
governance concern: its boundary is stable enough to enforce physically, but the
enforcement is not in place.

Polyrepo graduation may be deferred beyond ACTIVE if a stated reason is recorded
in the capability map entry. Unstated deferral is not compliant.

Additional criteria that independently justify graduation:
- the cell has a distinct deployment cadence from other cells,
- or the cell has a distinct operator or team.

Polyrepo is not justified by preference for isolation alone. The operational
overhead of published contracts and coordinated versioning is real. It is earned
by boundary stability — ACTIVE status is that bar.

### Hybrid (graduated)

Cells incubate in the monorepo. When a cell reaches ACTIVE status, it graduates
to its own repository. The contracts package graduates at the same time — from
workspace reference to published package.

This is the expected and required long-term trajectory for a system that begins
as a fat cell and decomposes progressively. Graduation is not optional once the
ACTIVE threshold is met.

---

## Per-Cell CI Pipeline

Every cell must have an independent CI pipeline. In a monorepo, this is a
path-scoped workflow. In a polyrepo, it is the repository's primary workflow.

A compliant cell CI pipeline must:

1. **Trigger only on changes to that cell's directory** (monorepo) or on any push
   (polyrepo). Cross-cell changes must not trigger another cell's pipeline.

2. **Validate the contract surface.** Run the decomposition watchdog contract
   checks against this cell's implemented routes. Fail the pipeline if a
   contracted route is missing or an undocumented route exists.

3. **Run the cell's own tests.** Unit tests, integration tests, and any pact
   contract tests for contracts this cell provides or consumes.

4. **Check import boundaries.** Fail if this cell imports from another cell's
   `src/` directory. Imports from `contracts/` are permitted.

5. **Build and tag a container image.** Each cell produces its own independently
   deployable container image tagged with its own version.

6. **Emit a health check result.** The built image must respond correctly to
   `GET /health` before the pipeline passes.

### Example: path-scoped workflow (monorepo)

```yaml
# .github/workflows/hypothesis-cell.yml
on:
  push:
    paths:
      - 'hypothesis-cell/**'
      - 'contracts/**'   # contract changes affect all cells

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pnpm install
      - run: pnpm --filter hypothesis-cell typecheck
      - run: pnpm --filter hypothesis-cell test
      - run: pnpm --filter hypothesis-cell watchdog:contracts
      - run: pnpm --filter hypothesis-cell watchdog:imports
      - run: docker build -t hypothesis-cell:${{ github.sha }} hypothesis-cell/
```

Note: contract changes trigger all cell pipelines, because a contract change is a
potential breaking change for every cell that references it.

---

## Versioning

Each cell is versioned independently. Version numbers reflect the cell's own
release history, not the ecosystem's.

```
hypothesis-cell@1.2.0
market-data-cell@1.0.3
debate-cell@2.0.0    ← major bump: breaking contract change
execution-cell@1.1.1
```

### Contract versioning

Contracts are versioned separately from the cells that implement them.

A contract version increment is required when:
- a field is removed or renamed,
- a field's type changes in a breaking way,
- a required field is added,
- or an endpoint's behaviour changes in a way that breaks existing callers.

A contract version increment is not required when:
- an optional field is added,
- a new endpoint is added (existing callers are unaffected),
- or documentation is updated without schema change.

When a contract version increments:
1. The new version is defined alongside the old (e.g. `v2/structural-candidates/{theme}`)
2. The provider implements both versions during the migration window
3. All consumers update to the new version
4. The old version is deprecated and eventually removed

No cell may deploy a breaking contract change without identifying all consumers
and verifying their migration readiness. This is the EA Cell's responsibility to
enforce.

---

## State Migration

When a cell is extracted from a fat cell, its state must migrate cleanly.

### Principles

- State migration is a one-time, planned operation — not an ongoing dual-write.
- The fat cell and the extracted cell must not share a state directory after
  migration is complete.
- A transition period of dual-read (fat cell reads from extracted cell's API) is
  acceptable. Dual-write (both cells write to the same file) is not.
- The migration plan must be documented in an Architecture Decision Record before
  extraction begins.

### Transition pattern

```
Phase 1: Fat cell implements the cell's contract internally
         (routes exist, no new process)

Phase 2: New cell process starts, owns its state directory
         Fat cell proxies to new cell for all reads and writes
         Shared filesystem access is removed

Phase 3: Consumers update to call new cell directly
         Fat cell proxy is removed

Phase 4: Fat cell no longer contains the extracted capability
```

The execution-cell extraction followed this pattern.
`MARKET_EXPLORER_DIR` was the transition-period shared path reference.
Phase 3 is not yet complete — the fat cell still proxies journal reads.

---

## What a Deployable Cell Must Have

A cell is deployable when it has all of the following:

| Artefact | Purpose |
|----------|---------|
| `package.json` | Independent dependency management and scripts |
| `CLAUDE.md` | AI development boundary contract |
| `server.ts` | HTTP entry point on its declared port |
| `GET /health` | Liveness endpoint returning `{ cell, port, status }` |
| `Dockerfile` | Container image definition |
| CI workflow | Path-scoped pipeline (monorepo) or repo pipeline (polyrepo) |
| Contract reference | Pointer to the contracts it provides and consumes |
| State directory | Declared and exclusively owned data path |

A cell directory that exists but lacks any of the above is an incomplete
extraction, not a deployed cell.

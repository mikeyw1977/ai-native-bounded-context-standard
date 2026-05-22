# AINBCAS — Threat Model

> Part of the [AI-Native Bounded Context Architecture Standard](README.md)

---

## The Thesis

Traditional enterprise security is designed for human-speed attacks. AI-assisted
adversaries operate at machine speed with increasing contextual intelligence.
That gap is not a temporary condition. It is structural, and it is widening.

Organisations that built their security posture around perimeter defences, patch
management cycles, and on-demand scanning were making rational decisions for the
threat environment that existed. That environment no longer exists.

---

## How AI Changes the Attack

A human attacker probing an enterprise system is constrained by time, attention,
and the limits of what they can hold in context. They find one vulnerability, they
exploit it, they move carefully to avoid detection. The attack is sequential and
relatively slow.

An AI-assisted attacker operates differently on three dimensions:

**Speed.** Probing, brute-forcing, and vulnerability discovery happen at machine
speed. What a skilled human attacker might do in days, an AI-assisted attack
can do in minutes. The window between vulnerability disclosure and active
exploitation — which was once measured in weeks — collapses.

**Context richness.** A human attacker who gains a foothold has limited ability
to rapidly understand what they have access to and how to use it. An AI-assisted
attacker can quickly map the system, infer the architecture from observable
behaviour, identify the most valuable targets, and chain exploits intelligently.
A breach that previously yielded limited value becomes significantly more
damaging because the attacker can bring far more analytical capability to what
they find.

**Persistence and adaptation.** AI-assisted attacks can probe continuously,
learn from failed attempts, adapt strategy in real time, and operate at a scale
that overwhelms human-monitored alert systems. The attacker does not get tired.
The attack does not require sleep.

---

## Why Traditional Security Cannot Keep Up

Enterprise security has three structural problems that AI-assisted attacks
directly exploit.

**The patch backlog.** Most organisations carry a backlog of unresolved
vulnerabilities. Patching is time-consuming, requires testing, and competes
with feature delivery for deployment slots. AI-assisted attackers are aware of
published CVEs the moment they are disclosed and can target organisations that
have not yet patched them at scale. The backlog is not a minor operational
inconvenience — it is a continuously expanding attack surface.

**The deployment constraint.** Most enterprises cannot deploy more than once a
day. Many cannot deploy more than once a week for critical systems. A detected
vulnerability requires a fix, a test cycle, an approval, and a deployment window
before it is addressed. At machine-speed attack rates, that timeline is not a
response — it is a delay during which the vulnerability is being actively
exploited.

**On-demand defences.** Vulnerability scanning, penetration testing, and security
reviews are periodic activities. They find the state of the system at a point in
time. In a world where the attack surface changes continuously and attackers probe
continuously, point-in-time defences are asymmetrically outmatched.

The response time mismatch is the existential problem. Attackers operate at AI
speed. Defenders operate at human-deployment speed. No amount of additional
tooling built on the same reactive, deploy-based model closes that gap.

---

## Unnecessary Code Is Attack Surface

There is a direct connection between architectural discipline and security posture.

Every line of code that exists without a value stream justification is potential
attack surface that nobody is actively governing. Unused endpoints, orphaned
utilities, speculative abstractions, and dead code paths all represent places
where an exploit can enter or operate without being immediately detected.

This is not a theoretical concern. Exploits frequently enter through code that
appears to serve no current purpose — because no fitness function monitors it,
no test covers it, and no team owns it.

The principle is simple: less unnecessary code means less attack surface. A system
where every capability traces to a declared value stream, and where the watchdog
enforces that nothing undeclared operates at the cell boundary, is structurally
harder to exploit than a system that has accumulated capabilities without governance.

See [Anti-Pattern 4 — Semantic-Free Decomposition](anti-patterns.md#4-semantic-free-decomposition).

---

## What a Defensible Architecture Looks Like

The response to AI-assisted attacks must operate on the same timescale as the
attacks themselves. That requires three properties that traditional security
architectures do not have:

**Continuous, not periodic.** Defences must operate as always-on capabilities,
not as scheduled scans or triggered reviews. The detection surface must be live.

**AI-backed, not human-speed.** Threat detection, anomaly identification, and
initial response must be automated and intelligent. Human review is for decisions,
not for detection. An analyst reviewing logs after the fact is not a defence against
machine-speed attacks — it is forensics.

**Fast isolation, not fast patching.** The correct response to a compromised
component is not to patch it faster. It is to isolate it immediately and replace
it from a known-good specification. The deployment cycle is not fast enough for
the threat environment. Isolation and replacement must operate below the deployment
cycle.

---

## How AINBCAS Architecture Properties Address This

AINBCAS was designed to govern AI capabilities, not to be a security framework.
However, the architectural properties that make AI capabilities governable are
the same properties that make a system more defensible against AI-assisted attacks.
This is not coincidental — both problems require the same answer: externally
verifiable behaviour, bounded authority, and fast isolation.

**Short-lived, cell-scoped credentials** limit the value of a credential
compromise. A credential that expires in minutes and is scoped to one cell's
declared authority does not give an attacker persistent, broad access. They
would need to continuously re-compromise credentials to maintain access — a
significantly harder attack to sustain and hide.

**Per-cell kill switches** operate at the platform layer, not the deployment
layer. A compromised cell can be isolated — credentials revoked, tool access
disabled, event publishing halted — in seconds, without a deployment cycle. The
response time is no longer constrained by patch approval and deployment windows.

**Cell replacement rather than patching** changes the remediation model
for cell-level vulnerabilities. A compromised cell is not patched — it is
regenerated from its specification documents in a clean environment. The cell
implementation patch backlog does not accumulate because the answer to any
compromised cell implementation is replacement, not repair.

The distinction is important:

> **Replace compromised cells. Patch compromised substrates.**

Cell replacement eliminates the implementation patch backlog. It does not
eliminate the need to patch the platform, runtime, base images, shared libraries,
and model gateway that cells depend on. Vulnerabilities in the shared substrate —
the foundation that all cells run on — still require traditional remediation.
Vulnerabilities in a cell's own implementation are replaced away. Both are
necessary; they are not the same problem.

**Capability-scoped authority** limits blast radius. A compromised cell in a
monolith can access everything the process can access. A compromised cell in an
AINBCAS system can only access what its declared authority permits — the contracts
it is authorised to call, the tools it is authorised to use, the data
classifications it is authorised to process. The breach is real, but the damage
is bounded.

**Observable behaviour as the detection surface.** AI-assisted attacks behave
differently from legitimate cell operation. Unusual call patterns, unexpected
data access, anomalous event emission — all of these are detectable against the
cell's declared behavioural contract. The immutable audit stream is not just
forensic capability; it is the continuous detection surface that a security
cell can monitor for deviation from contracted behaviour.

---

## Security as a Continuous Cell

In an AINBCAS system, security is not a periodic review or an external tooling
layer. It is a cell — or a set of cells — that continuously observes the
behaviour of other cells and acts on anomalies without requiring a deployment
cycle to respond.

A security cell in this model:

* Monitors the audit stream continuously for deviations from contracted
  behaviour — unexpected call patterns, access to undeclared resources, unusual
  event volumes
* Compares observed cell behaviour against declared fitness functions in real time
* Can trigger cell isolation (kill switch invocation) autonomously on confirmed
  anomalies, within defined consequence classes
* Escalates to human review for decisions above its autonomous authority threshold
* Operates on the same replacement model as any other cell — if the security
  cell itself is compromised, it is replaced, not patched

This is the AI-backed, continuous defence model that the threat environment
requires. The security cell is governed by the same standard as every other cell:
declared boundary, explicit contracts, observable behaviour, and deterministic
governance gates for actions above its authority class.

The governance model is the security model. They are not separate concerns.

---

## What This Standard Does Not Cover

AINBCAS defines architectural properties that improve a system's defensive
posture. It is not a security framework and does not cover:

* **Web application firewall configuration** and perimeter defence
* **Network security**, segmentation, and ingress/egress controls
* **Vulnerability management** beyond the cell replacement model
* **Threat intelligence** and external feed integration
* **Compliance frameworks** (SOC 2, ISO 27001, NIST CSF) — these impose
  additional requirements beyond what AINBCAS addresses
* **Supply chain security** for model providers and third-party dependencies

These remain important concerns. The argument here is not that AINBCAS replaces
security practice — it is that the architectural discipline AINBCAS requires
produces a system that is structurally more defensible than one without it, and
that the response model (continuous observation, fast isolation, replacement not
patching) is better suited to the AI-assisted threat environment than traditional
reactive approaches.

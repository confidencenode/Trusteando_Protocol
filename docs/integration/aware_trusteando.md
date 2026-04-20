# AWARE + Trusteando Integration

This document explains how:

- **Trusteando** publishes *certifiable facts* and produces an immutable, offline-verifiable audit trail.
- **AWARE** compiles *canonical structure* into executable semantics (preconditions, calculations, transitions).

They remain independent by design. The only coupling point is the **bridge** that turns **Publication Layer (Folder Language)** into an **AWARE seed** (or equivalent input).

---

## What This Enables

- A human publishes a folder with `steps/`, `since/`, `[firmante @...]`
- AWARE compiles that folder into executable modules (preconditions, calculations, transitions)
- Each action during execution is logged to Trusteando with a cryptographic signature
- A regulator receives a dossier that proves what happened, without access to the live system
- The dossier can be verified offline — the issuing authority does not need to be online

**Key property:** AWARE does not need to know about Trusteando to execute. Trusteando does not need to know about AWARE to audit. The bridge is the only coupling point.

---

## Concrete Example: Housing Benefit AI Assistant

| Step | Layer | What happens |
|------|-------|--------------|
| 1 | Publication Layer | Municipality publishes folder: `steps/01-verify-id`, `steps/02-calculate-income`, `[firmante @municipality.es]`, `since/2026-04-01` |
| 2 | AWARE | Compiles folder into OCG: preconditions (income ≤ threshold), calculations (annualized income), transitions (approved → notified) |
| 3 | Execution (AWARE) | AI assistant processes application. Each step invokes the corresponding handler. |
| 4 | Audit (Trusteando) | Each action is logged: `{action: "calculate-income", result: "approved", citizen_id: "123", safety_level: "high"}` → signed, immutable, offline-verifiable |
| 5 | Dossier / View | Extractor tool generates ZIP for EU AI Act compliance, HTML for municipality, JSON for automated verification |

**The citizen, the municipality, and the regulator all operate over the same explicit reality — but at different levels of abstraction and with different verification needs.**

---

## Why This Matters for Europe

European institutions face two fragmentation problems:

| Fragmentation | Solved by |
|---------------|-----------|
| **Technical** (same concept defined differently across backend, client, storage) | AWARE OCG → one canonical model |
| **Institutional** (each country has its own BOE, registry, DSL) | Trusteando Publication Layer → one folder structure that wraps different DSLs |

**Interoperability does not come from a common language. It comes from a common structure that wraps different languages.**

- AWARE ensures that a system's internal model is consistent across surfaces (Python, Dart, SQL).
- Trusteando ensures that different organizations can refer to each other's certifications without bilateral agreements.

Together, they enable cross-border, cross-sector verifiable workflows without centralization.

---

## Current Status and Next Steps (April 2026)

| Component | Status |
|----------|--------|
| AWARE OCG | ✅ Public, documented, compiles to Python/Dart/SQL |
| AWARE OIG + commits | 🔄 Next chapter (Actors, Functions, Commits) |
| Trusteando Core protocol | ✅ v0.2 specified |
| Trusteando Publication Layer | ✅ Specified (Folder Language) |
| Bridge (Folder → AWARE seed) | ✅ Implemented in `ctx-robotics` (`plan_source: "folder"`) |
| Trusteando worker (audit logs) | ✅ Implemented (HMAC/Ed25519, offline verification) |
| Dossier / View extractor | ⚠️ Pattern defined, tooling pending |
| Full integration documentation | 🔄 This document |

**Immediate next step:** Build the standalone Dossier/View extractor tool that reads Trusteando logs and generates regulator-specific dossiers without modifying the worker.

---

## Related Documents

- **Trusteando: A Layered Architecture for Verifiable Systems** — `../architecture/layers.md`
- **Trusteando Protocol Whitepaper** — `../../whitepaper/trusteando_whitepaper_v021_en.md`
- **AWARE: Object Config Graph** — `https://aware.run`
- **AWARE: Shared Digital Reality (Thesis)** — `https://aware.run/publications`

---

*Part of the Trusteando and AWARE documentation ecosystems*

*April 2026*

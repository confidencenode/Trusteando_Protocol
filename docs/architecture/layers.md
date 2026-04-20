# Trusteando: A Layered Architecture for Verifiable Systems

**Foundations document — v0.1**

*confidencenode.org/protocolos/trusteando/layers*

---

## Abstract

Trusteando has a **minimal core protocol** (20 lines of code) plus a **stack of optional layers** that work together. Each layer solves a distinct problem and offers a clear interface to the layer above. The analogy is TCP/IP: each layer has its purpose, does not contaminate the others, and the whole is greater than the sum of its parts.

| Layer | Purpose |
|-------|---------|
| **Human Bridge** | Capture human intent → deterministic contract |
| **Publication Layer (Folder Language)** | Auditable certification for humans and the law |
| **AWARE** | Complete semantic execution |
| **Trusteando Core** | Immutable audit + offline verification |
| **Notified Body Kit** | Structured evidence for regulators |
| **Dossier / View** | Presentation (MVC View) — independent extractor tool |

---

## 1. The Fundamental Problem

Current systems mix layers. A mission plan contains business logic, certification metadata, implementation details, and audit evidence all together. This mixing causes three problems:

1. **Certification is fragile** — changing the implementation invalidates the certification
2. **Audit is dependent** — verification requires understanding the implementation
3. **Interoperability is bilateral** — each pair needs specific agreements

---

## 2. The Layers

### Layer 1: Human Bridge — Intent → Contract

| | |
|---|---|
| **Input** | Natural language (system description from a process engineer) |
| **Output** | Bridge Output v1 (deterministic JSON with entities, states, handlers, projections, security, tests) |
| **Guarantee** | The JSON is compilable to AWARE without additional human intervention |
| **Non-goals** | Executable code, AWARE ontologies, implementation decisions |

*Why it's a layer:* The human does not need to know what OCG, OPG, OIG, projections, lanes, commits, or branches are.

---

### Layer 2: Publication Layer (Folder Language) — Auditable Certification

| | |
|---|---|
| **Input** | Folder structure published under a controlled URL |
| **Output** | A set of signed facts: `since/`, `until/`, `[firmante @...]`, `steps/`, `extern/` |
| **Guarantee** | The existence of the folder, signed by the domain controller, is the certification act |
| **Non-goals** | Conditional logic (if/else), loops, calculations, error handling |

*Publication under a controlled URL is the act.* An auditor, judge, or regulator can read the folder without special tools. Certification does not change when the implementation changes.

---

### Layer 3: AWARE — Complete Semantic Execution

| | |
|---|---|
| **Input** | Bridge Output v1 (or manual specification) |
| **Output** | Compiled modules (OCG, OPG, OIG) with executable handlers |
| **Guarantee** | Execution respects the defined preconditions, postconditions, and invariants |
| **Non-goals** | Certification (lives in Publication Layer), audit (lives in Trusteando Core) |

*AWARE is the "code".* It can be complex, change frequently, and be optimized. Certification (Publication Layer) does not change with every commit.

---

### Layer 4: Trusteando Core — Immutable Audit

| | |
|---|---|
| **Input** | Published facts (folders) or execution logs (optional streaming) |
| **Output** | Cryptographically chained immutable records |
| **Guarantee** | Offline verification: a parent can derive a child's key and disappear; the child remains verifiable forever |
| **Non-goals** | Consensus (no blockchain), real-time (clock skew ±5 minutes), business logic execution |

*Two modes:*
- **Static publication:** any entity with a URL publishes facts (e.g., "Juan is a professor since 2021")
- **Log streaming:** an optional worker receives signed logs (HMAC for internal append-only integrity, Ed25519 for externally verifiable attestations)

*Administrative time (minutes to months), not real-time.* A degree verified in 2 minutes is 2,000× faster than the current 6-8 weeks.

---

### Layer 5: Notified Body Kit — Evidence for Regulators

| | |
|---|---|
| **Input** | Trusteando logs, system specification, regulatory requirements mapping |
| **Output** | Structured dossier (optional: ZIP, HTML, PDF) with verifiable evidence |
| **Guarantee** | A notified body can verify compliance without access to the original system |
| **Non-goals** | The original system, real-time logs, access to internal implementation |

*Optional.* Not required to operate the system. Only for regulatory certification (EU AI Act, MDR, etc.).

---

### Layer 6: Dossier / View — Presentation Layer (MVC)

| | |
|---|---|
| **Input** | Raw Trusteando logs (via API or export) |
| **Output** | Formatted dossier (ZIP, JSON, HTML, PDF) for the target regulator |
| **Guarantee** | The extractor tool is independent of the worker; changing the format does not affect the audit |
| **Non-goals** | Audit logic, signature verification, log storage |

*MVC Analogy:*
- **Model** — Trusteando Core (immutable logs)
- **Controller** — Worker (receives, signs, stores)
- **View** — Extractor tool (generates dossier)

The Trusteando server must not have formatting logic, HTML templates, or ZIP generation. Separating them allows multiple views for different regulators (AI Act, MDR, GDPR) without redeploying the worker.

---

## 3. Threat Model (What Each Layer Guarantees)

| Layer | Guarantees | Does not guarantee |
|-------|-----------|-------------------|
| Human Bridge | The JSON contract is deterministic and compilable | That the human correctly described the system |
| Publication Layer | The certification is authentic (signed by the domain controller) | That the implementation respects the certification |
| AWARE | Execution respects preconditions and postconditions | That execution is correct (safety) — that requires testing |
| Trusteando Core | The log is immutable and offline-verifiable | That the log reflects physical reality (sensor spoofing, etc.) |
| Notified Body Kit | The dossier contains verifiable evidence | That the regulator accepts the dossier as sufficient |
| Dossier / View | The format is correct for the target regulator | That the original data is correct |

---

## 4. Why Layers Are Independent

| Change | Affected layers | Unaffected layers |
|--------|----------------|-------------------|
| Business logic changes (AWARE) | AWARE | Publication Layer, Trusteando, Dossier |
| Plan signer changes | Publication Layer | AWARE (executes same way), Trusteando |
| Audit algorithm changes | Trusteando | AWARE, Publication Layer |
| Dossier format changes | Dossier (extractor tool) | Trusteando, AWARE, Publication Layer |
| Regulation changes (AI Act v2) | Notified Body Kit, Dossier | Trusteando (logs remain valid) |

**The principle:** Each layer offers a stable interface to the layer above. Internal implementation can change without breaking contracts.

---

## 5. The Local-Global Tension, Resolved by Layers

| Level | Local (each country, each domain) | Global (interoperable) |
|-------|-------------------------------|----------------------|
| **Certification** | BOE, BOP, property registry, chamber of commerce | Publication Layer (same folder structure) |
| **Execution** | AWARE adapted to each national DSL | `extern/` (references to implementations) |
| **Audit** | Trusteando (signed logs) | Universal offline verification |
| **View** | Dossier adapted to each regulator | Configurable extractor tool |

**Interoperability does not come from a common language. It comes from a common structure that wraps different languages.**

---

## 6. End-to-End Example (Housing Benefit AI)

```
1. Human Bridge
   "I want an AI assistant that evaluates housing benefit applications"
   → After 8 questions → Bridge Output v1 JSON

2. Publication Layer
   municipality.es/trusteando/mandates/housing-assistant/
   ├── since/2026-04-01/
   ├── [firmante @municipality.es]/
   └── steps/
       ├── 01-verify-id/
       ├── 02-calculate-income/
       └── 03-notify-result/

3. AWARE
   Compiles the JSON + folder into executable modules:
   - preconditions (income ≤ threshold)
   - calculations (annualized income)
   - transitions (approved → notified)

4. Trusteando Core (streaming)
   Each AI action is logged:
   POST /log { action: "verify-id", result: "ok", citizen_id: "123", safety_level: "high" }
   → Immutable log, offline-verifiable

5. Notified Body Kit
   For AI Act audit: dossier with logs, requirements mapping, verification protocol

6. Dossier / View
   Extractor tool generates:
   - ZIP for the notified body
   - HTML for the municipality
   - JSON for automated verification
```

---

## 7. Conclusion

Trusteando has a **minimal core protocol** (20 lines) plus a **stack of optional layers**. Each layer does one thing and does it well:

| Layer | One sentence |
|-------|-------------|
| Human Bridge | Translates human intent to a deterministic contract |
| Publication Layer | Certifies for humans and the law via URL folders |
| AWARE | Executes the complete semantics |
| Trusteando Core | Audits immutably with offline verification |
| Notified Body Kit | Packages evidence for regulators |
| Dossier / View | Presents without contaminating the model |

The analogy to TCP/IP is not accidental. The internet works because its layers are independent. Digital trust will work when the layers of certification, execution, audit, and presentation are also independent.

---

*confidencenode.org/protocolos/trusteando/layers*

*Author: confidencenode.org/members/confidencenode0*

*April 2026 — v0.1*

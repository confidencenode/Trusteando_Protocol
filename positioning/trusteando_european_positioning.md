# Trusteando and European Digital Sovereignty

**A positioning document for European institutions, research funding bodies, and public administrations**

*This document accompanies the Trusteando Protocol Whitepaper v0.2*
*confidencenode.org/protocolos/trusteando*

## Related Documents

- **[Trusteando: A Layered Architecture for Verifiable Systems](../docs/architecture/layers.md)** — Conceptual foundation: why certification, execution, audit, and presentation must be separate layers. Includes the TCP/IP analogy and the resolution of the local-global tension (BOE vs interoperability).

---

## 1. The European Context

Europe has been pursuing digital sovereignty for years — reducing dependency on infrastructure controlled from outside (US, China). Initiatives such as GAIA-X, the eIDAS Regulation, the European Digital Identity Wallet, and Next Generation Internet investments all point in the same direction:

> **European infrastructure, European control, European values.**

Trusteando is not a response to this agenda. It is a protocol designed from first principles that happens to align naturally with it — because good protocol design and European values share the same foundations: decentralisation, transparency, privacy by design, and open standards.

---

## 2. The Problem Trusteando Solves

European institutions do not fail because they lack data. They fail because they lack a *verifiable, interoperable, cross-border* way to publish facts that other parties can check without manual processes, platform lock-in, or bilateral integrations.

Trusteando’s core move is simple: **folder hierarchy = key hierarchy**. Any entity that controls a URL can publish verifiable facts by creating a `trusteando/` folder on its own web server.

**Cross-border degree verification still takes weeks.** A graduate applies to an employer in another country; HR requests verification; the university processes it manually; the result arrives in 6–8 weeks. Trusteando reduces this to administrative minutes: the university publishes once under its own domain, and any verifier can check the cryptographic proof in ~2 minutes — without calling the university, without an API contract, and without a central database.

**Healthcare data exchange fails at the edges.** Hospitals and regional systems often cannot share referrals, discharge summaries, or lab results without bespoke integration projects. Trusteando shifts the burden from integration to publication: if two parties publish the same field schema under the same URL patterns, they become interoperable by construction. The protocol is the interface.

**SMEs struggle to be recognised abroad.** A small company can comply locally yet still be treated as “unknown” by foreign banks, insurers, or procurement portals because recognition depends on platform enrollment or manual paperwork. Trusteando replaces onboarding with URL control: an SME publishes a `trusteando/` folder under its own domain, signed with existing credentials, and becomes verifiable immediately.

**Public procurement is still PDF-and-stamp heavy.** Tendering processes frequently require manual verification of PDFs, signatures, stamps, and scans. Trusteando turns verification into reading structure: the verifier checks a folder schema and signatures, with validity windows (`since/` and `until/`) and revocation signals (`revoked/`) published as facts — no PDFs, no email chains, no “please confirm”.

---

## 3. Alignment with European Initiatives

| Initiative | Concrete integration points with Trusteando |
|------------|--------------------------------------------|
| **eIDAS 2.0** | Use qualified certificates as roots for high-assurance nodes (`t9`). Bind legal identity to a cryptographic publishing key; publish mandates, roles, and validity periods as signed facts under institutional domains. |
| **European Digital Identity Wallet** | Use Trusteando as a lightweight verification layer: credentials remain verifiable without contacting the issuer; wallets can verify offline, cache proofs, and validate time-bounded facts (`since/…`, `until/…`) without platform dependencies. |
| **GAIA-X** | Publish participant identities, roles, and service claims under participant domains; enable machine-verifiable trust graphs without bilateral onboarding; allow ecosystems to validate provenance and organisational relationships with offline verifiability. |
| **Next Generation Internet** | Deploy decentralised identity and verifiable knowledge graphs on existing web infrastructure; build open-source reference implementations (wallet + verifier + crawler) and reusable blueprints for education, healthcare, and public administration. |
| **Once-Only / Cross-border interoperability** | Replace repeated “please provide the same document again” flows with a single published fact that any authorised party can verify; reduce duplication while keeping issuers as the source of truth under their own jurisdiction. |

Optional compatibility note: Trusteando does not require semantic-web tooling to be useful. If an institution already operates linked-data infrastructure, Trusteando folders can be exposed as derived views without changing the underlying protocol.

---

## 4. The Official Gazette as Legal Anchor

In Europe, legal authority often comes from publication. Spain’s BOE (Boletín Oficial del Estado), France’s Journal Officiel, Germany’s Bundesanzeiger, and Italy’s Gazzetta Ufficiale publish acts with binding effect. Trusteando uses the same pattern for identity anchoring: publication is enough.

An institution can reference its official gazette entry from its `trusteando/` folder (or publish the reference inside the gazette). This creates **dual anchoring**:

- **Cryptographic anchor:** the node’s key proves authorship of the published facts.
- **Legal anchor:** the gazette reference ties the node to a legally recognised entity.

Neither replaces the other. Together, they allow strong verification without creating a central registry and without requiring new legislation: the protocol uses existing publication mechanisms and existing legal identity frameworks.

Concrete syntax example:

```
boe.es/trusteando/
[boe-ref BOE-A-2027-XXXX]/
```

---

## 5. Integration with eIDAS and the European Digital Identity Wallet

Trusteando does not replace eIDAS. eIDAS defines legal identity assurance and signature validity; Trusteando defines how those identities publish verifiable facts as a decentralised, auditable graph under domain control.

| eIDAS level | Trusteando equivalent |
|-------------|----------------------|
| Low | b9 (self-declaration) |
| Substantial | v9 (email + phone) |
| High / Qualified | t9 (DNIe / FNMT certificate) |

A qualified electronic signature can be the **root of a `t9` node**: the institution (or citizen) uses an existing qualified certificate to anchor the root key, then publishes roles, mandates, credentials, and validity periods as signed facts under a URL it controls. The result is a portable, verifiable identity surface that does not depend on a vendor, a platform, or a live issuer endpoint.

Critically, citizens do not need new credentials: this approach reuses what already exists (e.g., DNIe and national qualified certificates such as FNMT).

For the European Digital Identity Wallet, Trusteando provides a pragmatic verification layer:

- **Offline verifiability:** verifiers can validate proofs without contacting the issuer (critical for administrative workflows and long-lived credentials).
- **Time-bounded facts:** `since/` and `until/` folders express validity windows in a way that verifiers can check deterministically.
- **Selective disclosure pattern:** sensitive data can live under `private/` and be revealed via explicit grants, while public facts remain globally verifiable.

---

## 6. What Trusteando Offers European Institutions

| Real European problem | Trusteando solution |
|----------------------|----------------------|
| Cross-border degree verification takes 6–8 weeks | University publishes once. Any employer verifies in ~2 minutes. |
| Hospitals cannot share patient data between regions | Same schema → interoperable by construction. No integration project required. |
| SMEs cannot prove themselves to foreign banks | Folder under their own domain, signed with existing credentials. |
| Public tenders require manual document verification | Verifier reads folder structure and signatures. No PDFs, no stamps. |
| Cross-organisational verification requires contacting the issuer every time | Trusteando enables offline verification without contacting the issuer. |

---

## 7. The EU AI Act: Compliance by Construction

The EU AI Act establishes that high-risk AI systems — including those used in public administration, education, employment, and access to essential services — must be transparent, traceable, and subject to human oversight.

Trusteando provides the infrastructure for compliance with these requirements by design, not by addition:

**Traceability by construction.** Every action taken by an AI agent through a Trusteando-enabled process is recorded in the graph with the agent's identity, the model version, the mandate reference, and a hash of the action. The audit trail is permanent, tamper-evident, and readable by any authorised auditor without special tooling.

**Human oversight with a structural kill-switch.** The mandate model ensures that a human principal is always identifiable behind any AI action. Revoking the agent's authority is a single publication in the principal's own node — no platform intermediary, no vendor cooperation required. The human always has the switch.

**Accountability chain to the citizen.** The AI agent mandate traces every machine action back to the human who authorised it, via a cryptographically signed chain that includes the identity level of the principal (t9 for DNIe-backed mandates). The accountability chain is not declared — it is structural.

**Sector-specific compliance.** The parallel pathway model allows administrations to define different audit levels for different risk categories — consistent with the AI Act's tiered approach to risk. High-risk AI actions (affecting benefits, permits, legal status) can require enhanced audit and mandate verification; low-risk actions (information queries) can run with lighter controls.

Concrete example: an AI assistant used by a municipality to pre-process housing benefit applications can publish, for each decision-relevant step, the agent identity, the exact model version, the mandate URL, and an action hash under the municipality’s domain. An auditor can verify the chain later without asking the vendor for logs.

---

## 8. Verification at Administrative Speed

Trusteando operates on **administrative time**: minutes to months, not real time. This is a feature, not a limitation. It enables offline verification, long-lived audit trails, and institutional memory without introducing real-time revocation races and fragile always-online dependencies. Clock skew tolerance (±5 minutes) is a deliberate design parameter, not an operational constraint.

A degree verified in ~2 minutes is not “slow” — it is roughly **2,000× faster** than the 6–8 weeks that cross-border verification often takes today.

---

## 9. A Concrete Pilot Proposal

Pilot: **cross-border academic credentials**, designed to demonstrate verification in minutes without bilateral agreements or a central platform.

- **Participants:** 3 universities (Spain, Germany, France) + 1 European employer (multinational engineering firm)
- **Scope:** publish degree credentials and staff/student roles as Trusteando folder structures under each university domain; implement a verifier used by the employer’s HR process
- **Budget:** €150k reference implementation, €200k pilot deployment, €50k documentation (total €400k)
- **Duration:** 12 months
- **Expected outcome:** degrees verifiable in minutes, offline-capable verification, no bilateral integrations, no central registry, auditable issuance and validity windows

Budget figures are indicative estimates; the final cost depends on scope, number of credential types, and integration constraints of participating institutions.

Success is measured operationally: time-to-verify, reduction in manual verification requests, and a reproducible blueprint other universities can adopt without negotiations.

---

## 10. What Trusteando Is Not

- It is not another publicly funded European project
- It is not a replacement for eIDAS
- It is not a blockchain
- It is not a closed system controlled by any institution
- It is not yet a live network — it is a complete specification ready for implementation
- It is not a real-time authentication protocol
- It is not a database or a central registry
- It is not a token, a coin, or a financial instrument

It is an open protocol — like HTTP or SMTP — that any institution can implement, extend, and build upon. The specification is public. The algorithm is public. No entity can close it or revoke access to it.

---

## 11. Resistance to Institutional Capture

A legitimate concern: could a government or institution attempt to control Trusteando?

The design makes this structurally difficult:

- Multiple roots from day one — no single entity controls the network
- The root executes a public algorithm with no discretion — it cannot grant or deny access arbitrarily
- Open source — the code cannot be "closed"
- The network can grow without permission — any entity can publish a trusteando/ folder

The more an institution attempts to control it, the more evident its decentralisation becomes. The protocol is antifragile with respect to institutional capture.

---

## 12. Funding Pathways and Timeline

Trusteando is a natural fit for:

- **NGI (Next Generation Internet)** — decentralised identity infrastructure for the next web
- **Horizon Europe** — open-source protocol for cross-border interoperability
- **Digital Europe Programme** — deployment support for public administrations
- **EIC (European Innovation Council)** — protocol as infrastructure for the European digital economy

The reference implementation (wallet), institutional adoption support, and sector-specific blueprints (education, healthcare, public administration) are areas where European funding could accelerate adoption without compromising the open nature of the protocol.

---

| Phase | Timeline | Activity |
|------|-------|-----------|
| 1: Specification | Current (v0.2) | Protocol published; executable reference implementation and test vectors available in the implementation guide |
| 2: Pilot | 6–12 months | 2–3 universities, 1 ministry, 1 employer |
| 3: Cross-border | 12–18 months | Add institutions from 3 EU countries |
| 4: Production | 18–24 months | Open to any EU entity, wallet available |
| 5: Polycentric | 24–36 months | Multiple independent roots, European network |

---

## 13. How to Get Involved / Call to Action

- **Public administrations:** publish a `trusteando/` folder (level 1: static files) under your official domain; start with public roles, mandates, and validity windows.
- **Universities:** act as identity roots for staff and students; publish verifiable roles and degree credentials; participate in the cross-border pilot in Section 9.
- **SMEs:** claim your domain and publish your `trusteando/` folder; use it as a verifiable identity surface for banks, insurers, and procurement.
- **Developers:** implement the reference wallet and verifier; build open-source tooling (libraries, verifier CLI, crawlers) and reusable sector blueprints.
- **Funders:** support the academic pilot (Section 9) and reusable blueprints for education, healthcare, and public administration.

---

*confidencenode.org/protocolos/trusteando*
*Author: confidencenode.org/members/confidencenode0*
*Companion to: Trusteando Protocol Whitepaper v0.2*

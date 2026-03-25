# Trusteando and European Digital Sovereignty

**A positioning document for European institutions, research funding bodies, and public administrations**

*This document accompanies the Trusteando Protocol Whitepaper v0.2*
*confidencenode.org/protocolos/trusteando*

---

## The European Context

Europe has been pursuing digital sovereignty for years — reducing dependency on infrastructure controlled from outside (US, China). Initiatives such as GAIA-X, the eIDAS Regulation, the European Digital Identity Wallet, and Next Generation Internet investments all point in the same direction:

> **European infrastructure, European control, European values.**

Trusteando is not a response to this agenda. It is a protocol designed from first principles that happens to align naturally with it — because good protocol design and European values share the same foundations: decentralisation, transparency, privacy by design, and open standards.

---

## Alignment with European Initiatives

| Initiative | Connection with Trusteando |
|------------|---------------------------|
| **eIDAS 2.0** | Trusteando signatures can carry legal weight when linked with qualified identities under eIDAS. The two are complementary: eIDAS provides the legal framework, Trusteando provides the cryptographic evidence. |
| **European Digital Identity Wallet** | Trusteando could serve as a verification layer for the wallet — any credential issued under the protocol is verifiable without contacting the issuer. |
| **GAIA-X** | Trusteando provides a trust graph for GAIA-X participants: any entity publishing a trusteando/ folder becomes verifiable by any other participant without bilateral agreements. |
| **Next Generation Internet** | NGI funds explicitly support decentralised, open-source technologies for the next web. Trusteando is exactly that: a decentralised protocol running on existing web infrastructure, fully open source. |

---

## What Trusteando Offers European Institutions

| European Need | How Trusteando Addresses It |
|---------------|---------------------------|
| Independence from big tech | Identity does not depend on Google, Apple, or Microsoft |
| Data control | Each entity publishes in its own domain, under its own jurisdiction |
| GDPR compliance | Privacy by design, minimal data, selective disclosure |
| Interoperability | Universal folder schema — any system respecting it is interoperable by construction |
| Sovereignty | Designed for multiple roots in different European jurisdictions — architecture supports this from day one |
| Resilience | If one root fails, others sustain the network |
| Democratic values | Transparency, auditability, decentralisation |

---

## Natural Entry Points for European Adoption

**Public universities** can act as identity roots for their staff, students, and alumni — without depending on any external platform.

**Ministries and public administrations** can publish their trusteando/ folder and become verifiable nodes. Publication in official gazettes (BOE, Journal Officiel, Bundesanzeiger, etc.) provides the legal anchor for the cryptographic identity — two independent layers of verification.

**SMEs** can have verifiable identity without depending on big tech platforms. A small business can publish its trusteando/ folder on its existing web hosting and become part of the trust network immediately.

**Citizens** can control their own identity through wallets that implement the protocol — without surrendering data to centralised identity providers.

---

## What Trusteando Is Not

- It is not another publicly funded European project
- It is not a replacement for eIDAS
- It is not a blockchain
- It is not a closed system controlled by any institution
- It is not yet a live network — it is a complete specification ready for implementation

It is an open protocol — like HTTP or SMTP — that any institution can implement, extend, and build upon. The specification is public. The algorithm is public. No entity can close it or revoke access to it.

---

## Resistance to Institutional Capture

A legitimate concern: could a government or institution attempt to control Trusteando?

The design makes this structurally difficult:

- Multiple roots from day one — no single entity controls the network
- The root executes a public algorithm with no discretion — it cannot grant or deny access arbitrarily
- Open source — the code cannot be "closed"
- The network can grow without permission — any entity can publish a trusteando/ folder

The more an institution attempts to control it, the more evident its decentralisation becomes. The protocol is antifragile with respect to institutional capture.

---

## The Narrative

Europe has been searching for digital sovereignty. Trusteando is not another European project funded with public money — it is an open protocol that **enables** European institutions to recover control of their own identity and their relationships, without depending on external platforms.

The protocol does not adapt to European values. European values are what good protocol design looks like.

---

## Trusteando and the EU AI Act

The EU AI Act establishes that high-risk AI systems — including those used in public administration, education, employment, and access to essential services — must be transparent, traceable, and subject to human oversight.

Trusteando provides the infrastructure for compliance with these requirements by design, not by addition:

**Traceability by construction.** Every action taken by an AI agent through a Trusteando-enabled process is recorded in the graph with the agent's identity, the model version, the mandate reference, and a hash of the action. The audit trail is permanent, tamper-evident, and readable by any authorised auditor without special tooling.

**Human oversight with a structural kill-switch.** The mandate model ensures that a human principal is always identifiable behind any AI action. Revoking the agent's authority is a single publication in the principal's own node — no platform intermediary, no vendor cooperation required. The human always has the switch.

**Accountability chain to the citizen.** The AI agent mandate traces every machine action back to the human who authorised it, via a cryptographically signed chain that includes the identity level of the principal (t9 for DNIe-backed mandates). The accountability chain is not declared — it is structural.

**Sector-specific compliance.** The parallel pathway model allows administrations to define different audit levels for different risk categories — consistent with the AI Act's tiered approach to risk. High-risk AI actions (affecting benefits, permits, legal status) can require enhanced audit and mandate verification; low-risk actions (information queries) can run with lighter controls.

The EU AI Act requires that organisations deploying high-risk AI systems be able to demonstrate oversight. Trusteando makes that demonstration a graph query, not a manual audit.

## Funding and Development Opportunities

Trusteando is a natural fit for:

- **NGI (Next Generation Internet)** — decentralised identity infrastructure for the next web
- **Horizon Europe** — open-source protocol for cross-border interoperability
- **Digital Europe Programme** — deployment support for public administrations
- **EIC (European Innovation Council)** — protocol as infrastructure for the European digital economy

The reference implementation (wallet), institutional adoption support, and sector-specific blueprints (education, healthcare, public administration) are areas where European funding could accelerate adoption without compromising the open nature of the protocol.

---

*confidencenode.org/protocolos/trusteando*
*Author: confidencenode.org/members/confidencenode0*
*Companion to: Trusteando Protocol Whitepaper v0.2*

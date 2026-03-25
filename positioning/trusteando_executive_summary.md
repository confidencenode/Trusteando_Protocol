# Trusteando — Executive Summary

**For public institutions, administrations, and senior decision-makers**

*Companion to the Trusteando Protocol Whitepaper v0.2.1*
*confidencenode.org/protocolos/trusteando*

---

## The problem in one paragraph

Every organisation that needs to verify a fact about another organisation — a degree, a licence, an authorisation, a membership — currently solves this problem bilaterally: a phone call, an API integration, a PDF with a stamp, a database query to a central registry. Each pair of organisations that needs to exchange verified information must negotiate an agreement, build a connector, and maintain it. With N organisations in a sector, this requires O(N²) agreements. Public administration in Spain alone has thousands of bodies that need to exchange verified data. The current model does not scale. It never has.

---

## What Trusteando does

Trusteando is an open protocol that allows any organisation to publish verifiable facts about itself and its members — on a web server it already controls, under a domain it already owns, using folder structures that any system administrator can read with standard tools.

The core idea is simple enough to state in one sentence:

> **A folder that exists at a path, signed by whoever controls that domain, is a cryptographically verifiable fact.**

A university that publishes `university.es/trusteando/professors/juan-ruiz/since/2021-09-01/` has created a verifiable credential. No certificate authority. No bilateral agreement. No shared database. Any verifier who can reach that URL can confirm that the university says Juan Ruiz has been a professor since September 2021 — and that this fact has not been altered since publication.

---

## Why this is different from everything that came before

| Approach | The problem |
|---|---|
| Bilateral APIs | Requires a new agreement and a new integration for every pair of organisations |
| Central registries | A single point of control, failure, and political dependency |
| Blockchain | Designed for millisecond financial transactions — not for human-scale institutional facts |
| W3C DIDs / Verifiable Credentials | Complex infrastructure, unresolved governance, low adoption |
| PDF with stamp | Not machine-readable, not verifiable at scale, not auditable |

Trusteando's approach is different because it requires **no new infrastructure**. Organisations already have web servers. They already have domains. They already publish information online. Trusteando only asks them to structure that information in a standard way — the same way HTML standardised web pages without requiring new servers.

---

## What "verifiable" means in practice

Three properties — and only three — are guaranteed by the protocol:

**Integrity.** A fact that has been published cannot be silently altered. Any modification invalidates the cryptographic signature, and the alteration is detectable by any verifier.

**Authorship.** Only the entity that controls the domain could have published a fact under that domain. `ministerio.es/trusteando/autorizaciones/empresa-xyz/` can only be published by whoever controls `ministerio.es`.

**Temporality.** Every fact carries a `since/` date that is part of the signed content. The historical record is immutable: what was true in 2020 remains verifiable as having been true in 2020, even if the fact changed in 2023.

What the protocol does **not** guarantee: whether the fact is true. A university can sign a credential for a degree that was never awarded. The protocol makes this visible and traceable — it does not make it impossible. Auditors, oversight bodies, and competing institutions provide the veracity layer that the protocol deliberately leaves open.

---

## Designed for institutional time

Public institutions operate at human scale: days, weeks, months, years. A ministerial appointment takes effect 30 days after publication in the BOE. A court gives a tenant 30 days to respond to an eviction notice. A professional licence is renewed annually.

This is exactly the scale at which Trusteando operates — and this is a security property, not a limitation. At human scale:

- A credential that takes minutes to propagate across the network is effectively instant relative to how often institutional facts change.
- The period between publication and effect (vacatio legis) is a structural security buffer: an adversary who wants to insert a fraudulent fact must sustain the deception across an entire institutional response cycle.
- The historical record accumulates over months and years, making retroactive falsification progressively harder.

Systems designed for millisecond financial transactions require consensus mechanisms, distributed locks, and real-time settlement — complexity that is irrelevant and counterproductive for institutional credentials. Trusteando deliberately avoids this complexity. The result is a protocol that a public institution can adopt without retooling its IT infrastructure.

---

## What adoption looks like

**Minimum viable adoption (one afternoon):**

A webmaster publishes a `trusteando/` folder on the institution's existing domain. No new servers. No new software. No cryptographic infrastructure. The institution is immediately on the graph and discoverable by other nodes.

```
ministerio.es/trusteando/
├── identity/
│   └── [name "Ministerio de Educación"]/
├── universidades/
│   └── universidad-malaga/since/1972-01-01/
└── [state verifiado]/
```

**Full adoption (one developer, one week):**

A 50-line server script adds cryptographic challenge-response — the ability for members to prove their membership without contacting the institution for each verification. Credentials become verifiable offline, at scale, without load on the institution's systems.

**Interoperability by construction:**

Two hospitals that publish the same folder schema for prescriptions are interoperable — any pharmacy that can read one can read the other, without any bilateral agreement between the hospitals. This is the protocol's most powerful property for public administration: the schema is the contract.

---

## The audit trail is the folder structure

Any system administrator with SSH access to a web server can audit a Trusteando node directly — no proprietary software, no special client, no vendor dependency. The credential is a folder. The history is a directory structure. The signature is a file.

This matters for public institutions: auditability should not require purchasing a licence from the vendor of the verification system. In Trusteando it never does.

---

## What independent observers provide

Independent nodes — other universities, public agencies, professional associations — can index and archive the graph. Once an observer has seen a fact and published its own signed attestation, the original publisher cannot delete that fact without the deletion being visible. The distributed archive of observers is the practical guarantee of historical permanence — stronger than any single database, because it requires compromising every independent observer simultaneously.

---

## Relation to existing legal frameworks

**eIDAS:** Trusteando signatures can carry legal weight when linked with qualified identities under eIDAS. The two are complementary: eIDAS provides the legal framework, Trusteando provides the cryptographic evidence and the graph structure.

**BOE and official gazettes:** A BOE publication that includes the institution's Trusteando node URL and public key provides a two-layer identity anchor — cryptographic and legal — that any citizen, court, or administration can verify independently.

**GDPR:** Privacy is built into the protocol's structure. Facts that should not be public are placed under `private/` folders with access control. The existence of a private relation is visible; its content is not. Correction of errors adds new facts rather than altering old ones — preserving the right to rectification while maintaining the integrity of the historical record.

---

## For the evaluating institution

Three questions determine whether Trusteando is appropriate for a given use case:

**1. Does this domain operate at human scale?**
If the relevant facts change over days, weeks, or months — yes. If sub-second consistency is required — the protocol is not the right tool for that layer.

**2. Is the information already public, or does it need to be verifiable by third parties?**
If yes to either — Trusteando adds cryptographic verifiability to what you already publish, at near-zero marginal cost.

**3. Do you need bilateral agreements to share this information today?**
If yes — Trusteando eliminates the O(N²) agreement problem. Any institution that adopts the same schema becomes automatically interoperable with any other, without negotiation.

---

## Trusteando as governance infrastructure for the age of automation

Public administrations face a structural challenge that no current digital identity system addresses: AI agents can now act with the speed and scale of machines while claiming the rights of human citizens. An administration that cannot distinguish between a human filing a tax return and an agent filing ten thousand returns per minute is one script away from an administrative denial of service.

Trusteando provides the infrastructure for the administration to survive machine speed without sacrificing citizen rights. Three concrete capabilities:

**1. Protection against administrative service collapse.**
The parallel pathway model (§7.17) gives AI agents their own lane with explicit rate limits, enhanced audit, and independent suspension. Human services continue unaffected when an agent is suspended. The administration does not have to choose between serving humans and controlling machines — it does both simultaneously, in the same graph, with the same protocol.

**2. Legal compliance without new legislation.**
The AI agent mandate builds on the existing electronic representation framework of the FNMT and Cl@ve system. The apoderamiento electrónico already exists in Spanish administrative law — Trusteando extends it to declare that the representative is an AI agent. A public official does not need to ask whether the law permits this. It already does. The technical implementation changes; the legal basis does not.

**3. Algorithmic auditability.**
Requiring `model-version` and action hash in the agent pathway makes mass audits possible. If a bias or error is discovered in a specific AI model, the administration can identify every decision influenced by that model — "review all submissions made by agent X running model Y between January and March" — from the graph, without asking anyone, without special tooling. The audit trail is the folder structure.

**The human-scale detection window.**
Because the protocol operates at institutional time — days and weeks, not milliseconds — an identity attack has a detection window that a human can act within. A citizen who receives a notification that their agent submitted something unexpected has time to publish `revoked/` in their mandate before the administrative process — which takes days — completes. The calendar is the security mechanism. This is not a limitation of the protocol. It is a property that makes it suitable for governance in a way that millisecond systems are not.

This positions Trusteando not as a digital version of existing bureaucracy but as the infrastructure layer that allows institutions to maintain meaningful control over processes that are increasingly executed by machines.

## Next steps

- **Whitepaper** — full technical specification: `trusteando_whitepaper_v021_en.md`
- **Quickstart** — Level 1 node in an afternoon: `trusteando_quickstart.md`
- **Cookbook** — domain examples (university, hospital, court, public administration): `trusteando_cookbook.md`
- **European positioning** — alignment with eIDAS, GAIA-X, NGI: `positioning/trusteando_european_positioning.md`

The protocol is open source, licensed under GNU GPL v3. No entity — including the founding author — can close it, restrict access to it, or charge for its use.

---

*confidencenode.org/protocolos/trusteando*
*Author: confidencenode.org/members/confidencenode0*
*March 2026 — v0.2.1*

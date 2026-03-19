# Trusteando Protocol
## Research Directions & Future Proposals

**v0.1 — companion document to the Trusteando Protocol Whitepaper v0.2**
**Author:** confidencenode.org/members/confidencenode0

---

## About this document

This document collects improvement proposals and research directions for future versions of the Trusteando protocol. The ideas contained here are open lines, not committed specifications. Some are natural extensions of the base protocol; others introduce external dependencies or complexity that requires deeper evaluation. The criterion for separation from the main whitepaper is: if a proposal requires new infrastructure, external dependencies, or implies significant design changes, it belongs here until it is mature enough to be incorporated into the specification.

---

## P1 — Hybrid Anchoring with IPFS for DNS Attack Resilience

**Problem:** A web domain can be confiscated or expire, making credentials published under it inaccessible.

**Proposal:** When an entity updates its trusteando/ space, it calculates the Merkle root hash of the entire tree and publishes it in two places: its own URL and an immutable decentralised network such as IPFS, under its autonomous identifier (entity_id). A verifier who doubts whether a server has been compromised can compare the hash published on the web with the one registered in IPFS. If they do not match, the web server is not trustworthy. This architecture means that compromising an entity's identity requires acting simultaneously on the DNS, the web server and the IPFS network, significantly raising the cost of an attack.

*Consideration: introduces dependency on external infrastructure (IPFS). Requires evaluation of stability and permanence before being incorporated into the specification.*

---

## P2 — Blueprints: Interoperability Templates by Domain

**Problem:** The open vocabulary of the protocol can generate semantic fragmentation — different communities using different terms for the same concept, reducing the interoperability the protocol promises.

**Proposal:** A Blueprint is a node in the graph that defines a standard folder structure and vocabulary for a sector. For example, blueprint/university/v1 defines what folders a university should expose and what terms to use for its relations. Entities declare adherence: [adheres-to blueprint/university/v1]. A search engine for professors will know exactly which folders to look in if the node follows the educational sector blueprint. Blueprints are graph nodes like any other — verifiable, versionable, and navigable.

*Low complexity, high impact on interoperability. Natural candidate for v0.3.*

---

## P3 — Fast-Track Revocation: High-Priority Revocation Channel

**Problem:** The observer replica model introduces an inconsistency window between revocation and propagation. For real-time access control this window can be dangerous.

**Proposal:** Replicas synchronise entries in registry/compromised/ with maximum priority over any other update. A Bloom Filter or compact signed revocation list (CRL) issued by the emitter allows replicas to verify revocations locally without an online query. The combination of priority propagation and signed CRL reduces the inconsistency window from minutes to seconds, making the system suitable for physical or logical access control in real time.

*Requires specification of CRL format and priority synchronisation mechanism.*

---

## P4 — ZK-Proofs for Verification Chain Aggregation

**Problem:** Verifying a fact at the end of a deep chain — root, university, faculty, department, professor — requires multiple HTTP requests and sequential processing of HMACs and signatures, penalising devices with limited resources.

**Proposal:** The emitter generates a ZK proof (zk-SNARK) that demonstrates the existence of a valid chain from a recognised root to the fact, without revealing the intermediate steps. The verifier checks a single small mathematical proof, regardless of chain depth. A more immediate solution without ZK is for each node to publish a precomputed Merkle proof of its position in the chain, enabling local verification without additional queries.

*Very high complexity for ZK-SNARKs. The precomputed Merkle solution is more immediate and can be developed in parallel.*

---

## P5 — Proof-of-Stake for Observer Replicas

**Problem:** The observer replica model mitigates Sybil attacks through institutional reputation, but without economic cost reputation alone may not be sufficient in adversarial environments.

**Proposal:** To act as a replica, a node deposits a stake managed by multiple roots. If a replica signs a false state in conflict with the majority, its stake is burned (slashing) and its reputation decreases. Vote weight combines seniority, reputation and stake. An attacker would need to stake sufficient capital to control the quorum, making the Sybil attack economically unviable.

*Introduces tokenisation and possibly blockchain dependency, in tension with Trusteando's design principle of not requiring blockchain. Open research.*

---

## P6 — Smart Contracts as Arbitrators for Programmable Disputes

**Problem:** Dispute resolution depends on external arbitrators and legal frameworks whose integration is not defined. For purely logical disputes, human arbitration introduces latency and cost.

**Proposal:** Parties to an agreement can include in the plan/ an automated arbitration clause. If the fact execution/milestone-X/ is published and signed by the corresponding party, a smart contract automatically executes the resolution — for example, releasing a held payment. For higher-level disputes requiring human interpretation, the result of automated arbitration serves as cryptographic evidence that a court can accept as expert testimony.

*Introduces blockchain dependency for contract execution. Applies only to disputes with algorithmically verifiable rules.*

---

## P7 — Periodic Hash Anchoring in Official Registries

Complement to the subscription examples via BOE and Federal Register (Appendix F.5 and F.6 of the whitepaper). Entities with critical roles could periodically publish the Merkle root hash of their trusteando/ space in the national official registry. This would create a chain of legally backed anchors that would allow detection of retroactive alterations even without IPFS.

---

> **Note on blockchain dependencies:** Proposals P5 and P6 introduce blockchain or smart contract dependencies. The Trusteando protocol is explicitly designed to not require blockchain. P5 and P6 are open research, not committed directions.

---

*confidencenode.org/protocolos/trusteando — companion document v0.1*

---

## P8 — Validation Experiment: HN Multi-Root Network

This proposal describes a concrete experiment to validate the polycentric bootstrapping mechanism described in section 7.10 of the whitepaper, using Hacker News as the test platform.

**Setup:** Create three independent root nodes for HN, operated by different people without prior coordination:

```
trusteando/hn/root1/   (operated by Alice)
trusteando/hn/root2/   (operated by Bob)
trusteando/hn/root3/   (operated by Carol)
```

Each root has its own secret. When a user requests to join, each root independently generates a verification/private key pair for that user. The user publishes all three verification keys in their HN profile. Any verifier that trusts at least 2 of the 3 roots can verify the user's identity:

```python
class HNCrowd:
    def __init__(self):
        self.roots = [
            TrusteandoNode(key_root1),
            TrusteandoNode(key_root2),
            TrusteandoNode(key_root3)
        ]
        self.threshold = 2

    def verify_user(self, user_hn, context, proof):
        votes = sum(
            1 for root in self.roots
            if root.verify_child_authorship(user_hn + ":private", context, proof)
        )
        return votes >= self.threshold

    def get_verification_keys(self, user_hn):
        return [root.grant_key(user_hn + ":verify") for root in self.roots]
```

**What the experiment would demonstrate:**

- **Resilience** — if one root is compromised or disappears, the other two sustain the network
- **Independence** — roots operated by different people without coordination can coexist
- **Simplicity** — the consensus mechanism is trivial (2-of-3) but effective
- **Uniformity** — no special cases: `HNCrowd` is built entirely on `TrusteandoNode`
- **Transparency** — all code is public, anyone can audit

**Why HN is the right first platform:** the community understands cryptography, can audit the code themselves, the concept "publish this string in your about" is familiar (Keybase did it), and the 2-of-3 threshold is easy to explain. Roots can be well-known community members.

**Natural next steps:** if it works on HN, the same mechanism applies to GitHub (publish in a gist), Twitter (publish in a tweet), Reddit (publish in the profile), LinkedIn (publish in a post). The parallel network starts to populate.

*This experiment is proposed to accompany the publication of whitepaper v0.2.*

---

*confidencenode.org/protocolos/trusteando — companion document v0.1*

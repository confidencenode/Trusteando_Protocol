# Trusteando as a Semantic Layer for AI Systems

**A positioning document for AI researchers, developers, and institutions**

*Companion to the Trusteando Protocol Whitepaper v0.2*
*confidencenode.org/protocolos/trusteando*

---

## The Problem with LLMs Today

Large language models are powerful but blind to structure. They know how to read text, but cannot distinguish:

- What is a fact vs. an opinion
- What source has authority vs. any anonymous page
- What relation is verifiable vs. hallucinated
- What identity is real vs. fabricated

They learn statistical patterns from text, not truth. That is why they hallucinate. That is why they cannot distinguish an official page from any blog.

---

## What Trusteando Provides

Trusteando converts the web into a graph of structured, verifiable facts. Instead of unstructured HTML:

```
GET uma.es/trusteando/profesores/juan/rol     → "catedrático"
GET uma.es/trusteando/profesores/juan/desde   → "2015-09-01"
GET uma.es/trusteando/registry/profesores/    → list of verified professors
```

Each fact is:
- **Atomic** — subject, relation, value, timestamp
- **Signed** — ECDSA signature by the entity with authority
- **Timestamped** — since/until conventions give full temporal history
- **Traceable** — the authority chain is navigable

This is an implicit API over the entire web. AI systems do not have to guess — they follow the structure.

---

## Trusteando as Training Data for AI

LLMs need training data with clear semantic structure. A web populated with Trusteando nodes provides:

**Atomic verified facts** — billions of subject-relation-object triples with provenance. Not scraped text, but structured facts signed by authoritative sources.

**Authority hierarchies** — who can say what about whom. An AI that understands this does not confuse a tweet with an official document, or a self-declaration with an institutionally verified credential.

**Temporal relations** — facts with since/until. The model learns what was true when — critical for reasoning about time-sensitive information.

**Cryptographic proofs** — not just text, but evidence that the fact is authentic. The model can learn to distinguish verified from unverified claims.

---

## Real-Time Verification for AI Agents

An AI agent with access to a Trusteando network can:

1. **Generate a claim**: "Juan is a professor at UMA"
2. **Verify it**: query `uma.es/trusteando/profesores/juan`
3. **Return confidence**: "claim verified (source: UMA, 2026-03-19)"
4. **Or self-correct**: "I cannot verify that claim against authoritative sources"

AI systems stop hallucinating verifiable facts. They learn to distinguish what they know from what they can check.

---

## A Practical Example

```
User: "Who is the longest-serving computer science professor at UMA?"

AI agent:
1. Translates: entity=uma.es, relation=profesores, filter=informatica, sort=since
2. Queries: GET uma.es/trusteando/profesores/?dept=informatica&sort=since
3. Gets: juan.perez@uma.es, since 2001-09-01 (signed by UMA)
4. Responds: "Juan Pérez, Full Professor since 2001 (source: UMA, cryptographically verified)"
```

This is more powerful than any current LLM for factual queries — because it does not guess. It knows, and it can prove it.

---

## The Virtuous Cycle

| Layer | Function |
|-------|----------|
| Trusteando | Verifiable facts, semantic structure, authority chains |
| AI systems | Comprehension, generation, reasoning |
| Users | Natural language queries |
| Web | Human content + machine-readable verified facts |

AI systems trained on Trusteando learn the **meaning** of relations, not just their statistical frequency. When deployed, they can query the live network to verify facts in real time.

---

## Connection with Trustworthy AI

The EU AI Act and European AI strategy emphasise transparency, explainability, and verifiability. An AI system that can say "I verified this against official sources" has a structural advantage over systems that only approximate.

Trusteando provides exactly this: a layer of verified, structured, auditable facts that AI systems can consult and cite. Not scraped text. Not hallucinated patterns. Cryptographically signed facts from authoritative sources.

---

## For Researchers and Developers

**Open questions this enables:**

- How do LLMs change when trained on structured verified facts vs. raw web text?
- Can RAG (Retrieval-Augmented Generation) systems use Trusteando as a knowledge base?
- How do AI agents navigate trust hierarchies to assess source reliability?
- Can Trusteando provide the "ground truth" layer that current AI benchmarks lack?

**What can be built today:**

- A Trusteando-aware search engine for AI agents
- A verification plugin for LLM outputs against Trusteando nodes
- A dataset generator that produces structured fact triples from existing Trusteando nodes
- A RAG system that uses Trusteando as a primary knowledge source with cryptographic provenance

---

*confidencenode.org/protocolos/trusteando*
*Author: confidencenode.org/members/confidencenode0*
*Companion to: Trusteando Protocol Whitepaper v0.2*

---

## The Public Network is Already There

A key insight about adoption: **80% of the value comes from data that is already public.** University directories, official registries, open government data, public platform APIs — all of this already exists and is already published. Trusteando does not need to convince anyone to share new data. It only needs entities to adopt the folder schema to structure what is already available.

Today:
```
uma.es/directorio/profesores/juan-perez.html
```
Free-format HTML, difficult to parse, impossible to verify cryptographically.

With Trusteando:
```
uma.es/trusteando/profesores/juan-perez/
├── [departamento informatica]/
├── since/2015-09-01/
└── [state trusteando]/
```
Same university, same public data — but structured, signed, and machine-readable.

### What LLMs Gain Immediately

If 100 European universities published their directories in Trusteando format tomorrow:

```python
# An AI agent could query directly
universities = ["uma.es", "upm.es", "ub.edu", ...]
facts = []
for uni in universities:
    professors = fetch(uni + "/trusteando/profesores/")
    for prof in professors:
        facts.append({
            "subject": prof,
            "relation": "professor_at",
            "object": uni,
            "since": fetch(prof + "/since"),
            "department": fetch(prof + "/departamento"),
            "verified_by": uni  # cryptographically implicit
        })
```

That is **high-quality training data** — facts, not opinions, with clear provenance. An LLM trained on this does not hallucinate about who is a professor at which university, because it can verify.

### The public/private distinction

The public network provides the foundation. Private nodes (`private/`) are the extension for sensitive relations, personal credentials, and controlled access. But **the network does not need private nodes to be useful**. It grows from public to private, not the other way around.

- **Public**: university directories, official registries, company data, public credentials → 80% of value, 20% of effort
- **Private**: personal relations, sensitive credentials, access control → the remaining 20% that makes the system complete

The practical implication: AI systems can start consuming and benefiting from Trusteando today, based purely on publicly available structured data, without waiting for private node adoption.

# TRUSTEANDO
## A Decentralised Verifiable Knowledge Graph Protocol

**Author:** confidencenode.org/members/confidencenode0
**URL:** confidencenode.org/protocolos/trusteando
**Date:** March 18, 2026 — v0.2

---



## Abstract

Trusteando is an open protocol for a decentralised, cryptographically verifiable Knowledge Graph. Its core is a class with five functions — fewer than twenty lines of code — from which the entire system emerges:


```
class TrusteandoNode:
def __init__(self, key): self.key = key
    @staticmethod
def reduce_hash(seed, elements):
    for e in elements: seed = hash(seed + e)
    return seed
def grant_key(self, child):
    return hash(self.key + child)
def respond_to_challenge(self, ctx):
    return self.reduce_hash(self.key, ctx)
def verify_child_authorship(self, child, ctx, proof):
    return self.reduce_hash(self.grant_key(child), ctx) == proof

```

From this simplicity the entire protocol emerges: identity, authentication, authorisation, hierarchies, privacy and portability. Nodes are entities identified by the hash of their public URL or by an autonomously generated identifier. Relations are expressed as folder structures published by each entity in its own web space — the folder structure is the credential. Trust chains are established via hierarchically derived keys: each node receives its key from its parent via grant_key, and any interaction is proven with respond_to_challenge without revealing keys. The graph is dynamic with full verifiable history via since/until and plan/execution conventions. Identity is portable via an emergency key mechanism and old_identities tables published by receiving entities. The system is polycentric by design: any sufficiently reputable node can act as a root by executing the same public algorithm, without requiring blockchain. When adopted by public administrations, the folder schema acts as a universal interoperability contract: any system that exposes its data respecting the schema becomes interoperable by construction with any other, without bilateral agreements or intermediary platforms. Membership precedes recognition. Contextual trust suffices for most verification. Cryptographic verification handles the rest.




---

# 1. Introduction

Semantic interoperability — the ability for independent systems to exchange data and understand its meaning without prior negotiation — has been the holy grail of information technology for thirty years. Every attempt has fallen short:

| Approach | Why it failed |
|---|---|
| EDI | Required bilateral agreements between every pair of participants |
| XML schemas | Demanded centralised governance and prior negotiation |
| RDF / OWL | Too complex for mass adoption — never left academic and research contexts |
| REST API standards | Scale as O(n²) — every new participant must negotiate with every existing one |
| Linked Data | Separated from existing web infrastructure — required parallel adoption |

The result is a world of silos: hospital systems that cannot talk to each other, banks that require proprietary apps, public administrations that cannot share citizen data across borders, and identity systems controlled by a handful of private platforms. The European Union's PSD2 regulation spent years attempting bank interoperability through bilateral APIs and regulatory mandate — and achieved only partial results at enormous coordination cost.

Trusteando's central insight is that a folder structure, published on a web server you already control, under a domain you already own, is sufficient to achieve interoperability by construction. No negotiation. No integration project. No shared database. No bilateral agreement. Just a folder. Any organisation that publishes its structure in the same schema becomes automatically interoperable with any other that does the same — the schema is the contract. Two banks that publish the same `transfer/fields {}` schema are interoperable without any agreement between them. Two hospitals that publish the same `appointment/fields {}` schema can share patients without a data exchange project.

Existing digital identity systems share a structural flaw: they require identity to be granted by a central authority before it can exist. X.509 certificates depend on commercial Certification Authorities. Federated identity systems such as OAuth delegate identity to private providers — Google, Apple, Microsoft — which can revoke it unilaterally. Existing Knowledge Graphs like Wikidata or the Google Knowledge Graph are centralised or demand consensus from a central editor to write.

Trusteando begins from a different observation: the identity of a real entity does not need to be granted — it already exists. A university exists. It has a URL. It has staff. It has real authority over that staff. The relationships between entities — who belongs to what, since when, with what role — also exist. What is missing is not a mechanism to create that identity but a protocol to express it as a verifiable, decentralised knowledge graph with temporal memory.

Trusteando is that protocol. The graph's nodes are entities identified by the hash of their public URL. The edges are relationships expressed as folder structures published by each entity in its own web space. Each entity's cryptographic signature over its own space guarantees integrity. The trust chain between levels enables verification without relying on any central server in everyday usage.

What if trust worked like version control? What if every claim was a commit, every authority a repository, every verification a checkout? In Git, the history is immutable — each commit is a fact, the current state is the projection of that history. Trusteando applies this model to knowledge: every published folder is a commit, the state of an entity is the sum of the facts it has published, and verifying a credential is reconstructing the history of a branch of authority. There is no central database to query. There is a history to read.

The concepts the protocol uses — hierarchy, dates, signatures, delegation — are concepts humans already understand and already use. Trusteando does not need to be learned. It needs to be recognised.


## 1.1 A Parallel Network on the Same Web

Trusteando does not compete with the web—it overlays it. Every entity that publishes a trusteando/ folder on its domain adds a node to a parallel network that runs on the same HTTP infrastructure, the same servers, the same URLs. The difference lies in what that folder contains: not HTML for human consumption but folder structures that express cryptographically verifiable facts.

The web says: this document exists. Trusteando says: this relation exists, and it is verified by the authority over it. That distinction lets you accurately represent any organizational structure—universities with their faculties, companies with their departments, states with their administrations—and control at a granular level who can certify what about whom. The folder hierarchy mirrors the real hierarchy, and cryptography guarantees that only those with authority can issue credentials within their domain.

Both layers use the same infrastructure. Only the rules change.


## 1.2 Adoption Levels

Trusteando is a layered specification. Organisations and individuals can participate at the level that matches their technical capacity — there is no minimum requirement beyond publishing a folder.

**Level 1 — Publish a static trusteando/ folder**

Any webmaster can do this in an afternoon. No server-side code, no cryptographic infrastructure. The entity publishes its structure as static files. This is sufficient for membership in the graph, discovery by other nodes, and basic interoperability.

```
# Minimum viable Trusteando node
university.es/trusteando/
├── identity/[name "University of Example"]/
├── professors/
│   └── juan-ruiz/since/2021-09-01/
└── [state verifiado]/
```

**Level 2 — Add the reference server (50 lines)**

A single Python or Node script adds cryptographic verification — `respond_to_challenge`, `verify_child_authorship`, `grant_key`. A junior developer can deploy this in a day. This enables verified credentials, challenge-response authentication, and the HN bootstrapping mechanism.

**Level 3 — Implement the full DSL**

The complete specification — type system, `fields {}`, `extern/`, `on/` event handlers, `when` guards, `steps/` workflows — is for teams building infrastructure on top of the protocol. This level is optional for most organisations and not required to participate in the network.

The rest of this document describes the complete specification. Readers interested only in Level 1 or Level 2 can focus on sections 1–4 and the reference server implementation. The DSL sections (2.14 onwards) and appendices are reference material for implementers.

### A note on the grammar vs the schema

The protocol's grammar is simple — four syntactic patterns, twenty lines of code. Designing structures that faithfully represent real-world authority relationships is not trivial. It requires thinking about who has authority over what, where each fact lives, who can sign it, and how verifiers will navigate the structure. That is schema design, not syntax.

This is analogous to SQL: the grammar fits on a few pages, but designing a normalised database schema for a complex domain is a skill learned through practice. The protocol inherits this same tension.

The canonical examples (sections 7.10–7.13, Appendix G) exist to reduce this learning curve. They show how to model common patterns: authority chains, bilateral agreements, stateless verification, and multi-party consent. Implementors are encouraged to use them as templates and adapt them to their domains.

For complex new domains, an LLM prompted with these canonical examples can generate correct structures more reliably than from the grammar alone. The cookbook of practical examples is not optional documentation — it is the primary tool for learning to model correctly.

### Protocol core vs ecosystem

The complexity of this document should not be mistaken for the complexity of the protocol. The protocol core is four functions and a folder structure — it fits in twenty lines of code and can be implemented in an afternoon. Everything beyond that is ecosystem: conventions, vocabulary, optional patterns, and extension points.

The boundary is deliberate and fixed:

**Protocol core** — immutable, the minimum required to participate:
- `TrusteandoNode` and its four functions
- Folder hierarchy as key hierarchy
- `since/until` for temporal facts
- `private/` for access control
- The three conformity states (b9, v9, t9)

**Ecosystem** — optional, can be adopted incrementally:
- Type system and `fields {}` schemas
- `on/` event handlers and `when` guards
- `steps/` workflow convention
- `extern/` references and `procedures/` patterns
- Sector-specific vocabulary

This boundary will not move. Future versions of the protocol may add hooks — articulation points that allow the ecosystem to extend behaviour — but will not add complexity to the core. The power of the system emerges from composing simple primitives, not from enriching them.

---

# 2. Design Principles


## 2.1 The URL as Identity Anchor

An entity's identifier in the system is the cryptographic hash of its canonical URL:


```
entity_id = SHA-256(canonical_url)

```

The entity does not need to register with any prior system. Its URL already exists and is public. Trusteando recognizes it as an identity anchor without adding friction. Two different entities have different URLs and therefore different identifiers—SHA-256 collisions are practically impossible.

The canonical URL is defined as the entity's homepage with https scheme, no trailing slash, and lowercase letters. For example: https://universidad.es

**The URL is a convenience, not a constraint.** The protocol separates two independent concepts: the *identity anchor* (what makes an entity uniquely identifiable) and the *publication point* (where it publishes its graph). For the common case these coincide — an organisation uses its domain as both. But the protocol also supports autonomous identity, where the anchor is a locally generated hash with no dependency on DNS:

```
object.id          = hash("universidad-de-málaga")
object.secret_base = hash("secret phrase known only to the university")
```

In autonomous mode, the URL is only where the node chooses to publish — not what defines its identity. The identity can exist before having a URL, survive the loss of a domain, and persist even if the original root node disappears. Registration can be anchored by electronic signature, email verification, phone verification, or any other means that demonstrates control over the identity. The URL-based mode is the pragmatic default for entities that already have a web presence. The autonomous mode is the foundation for entities that do not, or that require identity independent of any infrastructure.

## 2.2 Membership Precedes Recognition

If a university publishes on its own website—for example at https://universidad.es/trusteando—the hash of a professor together with their public key, that professor already belongs to that university's identity space. The trust chain exists before the root node formally validates it.

The root node does not create the relationship between the university and its professor—it elevates it to cryptographically verifiable trust for third parties that do not know either party. For a verifier who already knows and trusts the university, the publication at its URL is enough. Section 2.15 develops the full gradient from self-declaration to institutional recognition, showing how the same mechanism — placing a node in a folder — operates at every level of trust.


## 2.3 Contextual Trust Suffices for Everyday Use

Seeing Juan Escobar's hash published at universidad.es/profesores already conveys trust to any verifier who knows that university. The root node does not need to certify it. The university has real authority—its URL is its identity, and its institutional reputation is its endorsement.

Full cryptographic verification resolves the cases where contextual trust falls short: when the verifier does not know the entity, when verification must be automatic, or when there is a dispute about the authenticity of a credential.


## 2.5 Authority is Demonstrated by Controlling the URL

La autoridad de una entidad para emitir credenciales no se declara — se demuestra controlando la URL bajo la que publica. Nadie puede publicar bajo universidad.es/profesores/ sin controlar universidad.es. El DNS y el servidor web son la prueba de control, exactamente igual que en HTTPS.

This answers the question of how a verifier knows that an entity has authority to issue a credential: because the credential is published under a URL that the entity controls. No additional declaration is required. Control is authority.


```
confidencenode.org                               ← ConfidenceNode controls this
/protocolos/trusteando                         ← therefore controls this
/members/confidencenode0                      ← and this
/members/confidencenode0/externals/github
→ github.com/ConfidenceNode0                ← linked external presence

universidad.es                                   ← the university controls this
/profesores/juan-ruiz                         ← therefore controls this
/old_identities/                              ← and this

```

The externals folder allows an entity to link from its own space to any presence on external systems it does not control—GitHub, ORCID, LinkedIn, rankings, official registries. The entity signs the link itself, so the authority over the link is its own. But externals is protocol semantics, not just a naming convention: it explicitly declares that the entity has no authority over what lies at the destination. A verifier who reads externals/github knows the entity is linking to something it does not control and therefore cannot guarantee its content—only that the link is authentic.

This distinction stands in direct contrast with the rest of the folders, where the absence of externals implies control:


```
/members/confidencenode0/externals/github
→ la entidad declara que NO controla el destino

/protocolos/trusteando
→ the entity implicitly declares that it DOES control this

Other valid examples:
/members/confidencenode0/externals/orcid
/members/confidencenode0/externals/linkedin
universidad.es/externals/ministerio-educacion
universidad.es/externals/rankings/qs-world

```

In Trusteando the URL structure declares the authority. The externals folder is the only explicit exception—and its existence makes the general rule even clearer.

This mechanism illustrates a real case: confidencenode.org/members/confidencenode0/externals/github points to github.com/ConfidenceNode0, where the protocol's code lives. The own domain is the canonical identity; GitHub is the current hosting for the code. When the self-host is up, the code will migrate—but the canonical identity does not change.


## 2.6 The Golden Rule — The Folder Structure is the Credential

All the information that an entity must provide about credentials is stored in its public folder structure. There are no hidden databases, no fields in separate documents, no stateful APIs that a verifier has to query.

A verifier can check any credential by following only public URLs. If the information is not in the folder structure, it does not exist as a credential in the protocol. This rule is not a convention—it is a condition of validity.

The practical consequences are profound: total verifiability without the issuer's cooperation, complete auditability of any entity's identity space, and resilience to outages—a static copy of the folders is enough to reconstruct all the information.

A detailed style guide in `trusteando_style_guide.md` documents the UML-inspired naming and relation conventions that keep this grammar consistent across implementations.


## 2.7 Trusteando as a Decentralised Knowledge Graph

The set of entities and relationships the protocol expresses forms a Knowledge Graph with three properties that existing systems do not combine:

- Decentralized — each node writes in its own space. No one needs permission from a central editor to publish relationships.
- Cryptographically verifiable — each edge is signed by whoever emits it. There is no need to trust any central editor—the signature proves it.
- Dynamic with a complete history — the since/and until/ folders give the graph a temporal dimension. It is not a static snapshot but a graph that evolves with an integral and verifiable history.
Credentials are the edges of the graph. Entities are the nodes. The trust chain is the topology. The identity protocol is a particular case of this more general graph.


## 2.8 The Protocol is Open and Irrevocably Public

The protocol format, the ability to run your own nodes, and manual verification by consulting URLs directly are free and remain so. No entity—including the Trusteando root node—can stop anyone from implementing the protocol, verifying credentials, or publishing their own identity space.

This openness is not a concession—it is what gives the system value. A protocol that can be revoked by its creator is not infrastructure: it is a service with terms of use.


## 2.9 Interoperability by Design

A structural consequence of anchoring identity in public URLs and expressing relationships as a folder schema is that any system—database, REST API, microservice—that exposes its data while respecting that schema becomes interoperable by default with any other doing the same. There is no need to negotiate bilateral agreements or develop specific connectors: the folder structure is the interoperability contract. This has especially relevant implications for public administrations and agencies, where current interoperability demands complex intermediary platforms and point-to-point agreements. The use case is developed in section 7.8.


## 2.10 The Graph as Immutable Historical Record

Facts published in the graph cannot be modified. This is not a semantic restriction but a direct cryptographic consequence: each credential is covered by an ECDSA signature. Modifying any field of an already published fact invalidates the signature covering it, making the alteration detectable by any verifier. The graph's past is, by construction, immutable.

When a fact contains an error, the correction does not erase the original—it adds a new fact that supplements or corrects it, signed by whoever has authority to do so. The graph keeps the complete history: the original mistake, the correction, who issued it, and when. Any verifier can reconstruct exactly what was valid at any point in the past.

This property distinguishes Trusteando from traditional registry systems, where modifying a field is operationally possible and often undetectable. In Trusteando, any attempt to alter the past is cryptographically visible. Trust in the system does not rely on no one wanting to tamper with the records—it relies on tampering being impossible without leaving evidence.

The since/until convention and the plan/execution model of A.24 are the temporal expression of this principle: since/ and until/ record when something happened, plan/ declares the intention, and execution/ accumulates the actual facts as they occur. The plan may change; execution facts already recorded do not.

Any published fact must be treated as immutable. Modifying it breaks the signature covering it and destroys the trust chain that depends on it.

### The graph as an event store

Trusteando is, at its foundation, a pure event store. Each folder is an event — an immutable fact published by whoever has authority over that path. The current state of any node is the projection of all events published about it. There is no hidden state, no mutable fields, no background database. The graph is the state.

This maps directly to the operations of traditional systems:

| Traditional operation | Trusteando equivalent |
|---|---|
| `INSERT` | Publish a new folder |
| `UPDATE` | Publish a new fact that supersedes the previous one |
| `DELETE` | Publish `revoked/` or `until/` |
| Audit log | The graph itself — no separate log needed |

The difference from a database is not just technical — it is epistemic. A database administrator can silently modify a record. In Trusteando, any modification invalidates the signature and is immediately detectable by any verifier. Trust does not depend on the operator's honesty — it depends on the cryptographic impossibility of undetected alteration.

Comparison with systems that share this property:

| System | Model | How facts are superseded |
|---|---|---|
| SQL database | Mutable | `UPDATE set revoked=true` — history lost |
| Blockchain | Immutable | New transaction — prior block unchanged |
| Git | Immutable | New commit — history intact |
| Trusteando | Immutable | New folder (`revoked/`, `until/`) — prior fact intact |

### Why `revoked/` is a child folder, not a flag

This immutability explains a design choice that might otherwise seem counterintuitive. Revocation is not a flag that modifies the node — it is a new event appended to the node's history:

```
juanito/
├── identity/
│   └── since/2026-03-23/       ← event: identity created
└── revoked/                     ← event: identity revoked
    ├── since/2026-03-24/
    ├── [by t9:12312312A]/
    └── [reason "key-compromised"]/
```

A verifier reads both folders and computes the current state: identity exists AND revocation exists — therefore the node is currently revoked. The original identity fact is not erased — it remains as evidence of what existed and when.

### The golden rule

> **In Trusteando, facts are never modified. Only new facts are added — facts that complement, supersede, or annul prior ones.**

This is what makes the system auditable without trusting the operator, recoverable without losing history, and verifiable by any third party at any point in time.

### Undoing a revocation — `unrevoked/`

If a self-revocation was published in error, it can be superseded by a new fact — not by deleting `revoked/`, but by publishing `unrevoked/`:

```
juanito/
├── identity/since/2026-03-23/
├── revoked/since/2026-03-24/       ← original revocation, preserved
└── unrevoked/since/2026-03-25/     ← new fact: revocation undone
    └── [by t9:12312312A]/
```

The history is complete: the node existed, was revoked, and was reinstated. No fact is erased. A verifier reading the full sequence can reconstruct exactly what was valid at any moment.

The authority question applies here too: who can publish `unrevoked/`? By the same reasoning as revocation — the parent, not the node itself, has binding authority. A node that self-revoked can signal its own reinstatement via `unrevoked/`, but a cautious verifier should seek confirmation from the parent before accepting it. `unrevoked/` is a reserved folder name with the same advisory status as `revoked/` when published by the node itself.

### Why immutability — the distributed systems foundation

Trusteando is not immutable by aesthetic preference. It is immutable because immutability is the only foundation that makes a distributed system reliably auditable.

Rich Hickey, the designer of Clojure, stated the principle precisely: *"State is the history of values an identity has had over time."* In Clojure, that history lives in memory during runtime, managed by atoms and refs. In Trusteando, that history lives in the folder structure permanently, signed by the entities that produced it.

The parallel with Erlang is equally direct. Erlang has no shared mutable state. Processes communicate by messages. If a process fails, the supervisor restarts it — no locks, no transactions, no coordination overhead. Trusteando has no shared mutable state. Nodes communicate by references (`extern/`). If a node fails, the reference remains as a historical fact — no locks, no transactions, no coordination overhead.

Both languages were designed for distributed systems. And both arrived at the same conclusion: mutation is the origin of complexity in distributed systems. If a state can change, you need to coordinate the change. If you need to coordinate the change, you need locks, transactions, consensus. If you need consensus, you introduce latency, complexity, and points of failure.

Trusteando extends this principle from program execution to inter-organisational coordination:

| Concept | Clojure / Erlang | Trusteando |
|---|---|---|
| Data | Immutable by default | Append-only folders |
| State | History of transitions | `since/`, `until/`, `revoked/` |
| Identity | Value, not variable | Hash of URL, not mutable |
| Communication | Messages between processes | `extern/` references |
| Failure | "Let it crash" | Absent node leaves reference intact |

The immutability is not a constraint. It is the property that makes the system auditable without trusting the operator, verifiable without coordination, and resilient without a central authority. A fact, once published, is permanent. The past cannot be rewritten — it can only be extended.


## 2.11 Public by Default, Privacy Declared

Everything that exists in the graph is public except for what lies under private/. Privacy is declared explicitly; publicity is the natural state. This inversion of the traditional model—where you have to declare what you share—makes the graph auditable by default and privacy a conscious choice, not a consequence of silence.

The private/ folder resolves three levels of privacy. First, private content with a public relationship: the node is visible but its properties remain hidden. Second, a private relation with the existence of public privacy: the private/ folder is visible but not its contents—a verifier knows something exists there without knowing what. Third, total privacy under an opaque folder: not even the type of relationship is visible. In all three cases, accessing the content requires raising a query to the agent controlling the folder, who decides whether to grant access via grantReveal().

Applied to a university: universidad.es/personal/profesores/private/ hides the identities of certain professors. An external verifier sees that hidden entries exist under that folder, but not who they are or how many. To access it, they must raise a query to the profesores/ agent, who decides whether to grant access. This solves the privacy problem of the relationship graph described in section 12.7: the existence of the edge can be hidden, not just its content.

Nodes can also be modeled as objects with methods and values, following the object-oriented programming analogy. The folder is the object, the subfolders are methods or collections, and the brackets are properties or values—immutable once signed. An agreement between parties illustrates this structure:


```
acuerdo-xyz/                         ← object
├── [state trusteado]/              ← object.state
├── participantes/                   ← object.collection
│   ├── [empresaA proveedor]/         ← object.property (immutable)
│   └── [empresaB cliente]/           ← object.property (immutable)
├── plan/                            ← object.method (intention)
│   ├── [objetivo "servicios 2026"]/
│   └── [state approved]/
├── execution/                       ← object.method (accumulated facts)
│   └── firma/since/2026-03-18/       ← fact (history, immutable)
└── private/             ← private, requires grantReveal()
```

The analogy with object-oriented programming is illustrative but incomplete: unlike a class with private state, there is no default encapsulation here. Everything is public except what is explicitly hidden. It is closer to an immutable record in a functional language: signed properties are not modified, corrections add new records, and the complete history is always accessible.


## 2.12 The Folder Hierarchy is the Key Hierarchy

The protocol has a structural property that connects its visible organization with its cryptographic foundation: the folder hierarchy and the key hierarchy are isomorphic. Each folder is a node, and each node has a key derived directly from its position in the tree. Traversing the folder tree is verifying the cryptographic chain. There is no separate public key infrastructure—the folder structure is the cryptographic infrastructure.

This isomorphism has an important practical consequence: any entity that controls a folder automatically controls the key associated with it, and can delegate keys to subfolders via grant_key. Authority over a branch of the tree is simultaneously authority over the information published in that branch and authority over the cryptographic keys that protect it. There is no possibility for these two authorities to diverge. Section 4.11 formalizes this principle with the TrusteandoNode class.

## 2.13 Node Conformity States (Verifiado, Trusteado, Brokenado)

The protocol is not an agnostic pipe. To ensure universal interoperability, it defines three mandatory semantic states that determine how a verifier must process a node or a branch of the graph:

* **Verifiado (Verified) — alias `v9`:** The base state of existence. A node is **Verifiado** if its cryptographic signature is valid and its URL is accessible. It represents a technically correct identity but without sufficient history or endorsements (e.g., a new user on a social network or a recently registered merchant). It is the "entry point" to the system.
* **Trusteado (Trusted) — alias `t9`:** The state of full operation. A node reaches a **Trusteado** state when, in addition to being *Verifiado*, it has an explicit endorsement from its superior node or has passed the consensus threshold of observers (see section 4.3). In terms of reputation, it is equivalent to a node in "good standing" or with "positive karma."
* **Brokenado (Broken) — alias `b9`:** The state of invalidity or alert. A node enters **Brokenado** mode when its cryptographic integrity fails (corrupt signature) or when the superior node publishes an explicit revocation for malfunction. A **Brokenado** node halts the propagation of trust: any credential or sub-node hanging from a branch marked as **Brokenado** is ignored by verifiers by default.

The short aliases — `b9`, `v9`, `t9` — are used in paths, type annotations, and code. The full names are used in prose. **T10** (uppercase, ten letters) refers exclusively to the protocol name and root path — it is not a trust level.

### Optionality and Contextual Interpretation

While the protocol strictly enforces a binary distinction between technically valid (**Verifiado**) and invalid (**Brokenado**) nodes, the transition to a **Trusteado** state is often an optional, context-dependent layer.

In many cases, the **Trusteado** state serves as a reputation signal—representing social value, "karma," or a history of good standing. This flexibility allows the protocol to scale from simple identity checks to complex decentralized ecosystems where trust is earned, not just cryptographically proven. The infrastructure provides the status; the community provides the meaning.

---

# 2.14 Trusteando as a DSL

Trusteando is a Domain-Specific Language for declaring verifiable organisational structures, access control hierarchies, and workflow definitions over distributed web infrastructure. It is not a general-purpose programming language — it has no standard library, no arithmetic functions, no string manipulation. What it has is deep expressiveness within its domain, analogous to SQL for data queries, HTML for document structure, or Terraform for infrastructure declaration. Like UML, it describes structures and relationships. Unlike UML, it is executable — a server can read the graph and make cryptographically verifiable decisions based on it.

## 2.14.1 The Five Syntactic Patterns

Every element in a Trusteando path belongs to one of five syntactic categories, each with unambiguous meaning:

```
subfolder-name/              OBJECT     — has its own key, controls its subtree
[field:value]/               IDENTIFIER — unique key, indexable, primary key
[field "value"]/             ATTRIBUTE  — descriptive string, not unique
[field value]/               PROPERTY   — numeric or enumerated value
extern/path/to/node/         REFERENCE  — link to data living elsewhere
fields { ... }               SCHEMA     — structured local data declaration
```

The colon distinguishes identifiers from attributes: `[client-id:C-123456]` is unique and indexable; `[name "Juan Ruiz"]` is descriptive. This convention is consistent throughout the protocol — `[instance:2]`, `[quorum:3]`, `[dni:12312312A]` all use the colon for values that identify or quantify precisely.

## 2.14.2 Objects vs Properties — Control vs Data

The most important distinction in the grammar is between objects and properties. An object placed in a folder has controlling power over its content — it has its own key derived by `grant_key`, can publish its own children, can respond to challenges, and propagates trust level. A property is simply data belonging to the parent node — no key, no control, no trust propagation.

```
professors/juan-ruiz/          ← OBJECT: juan-ruiz has identity, controls subtree
    └── since/2021/            ← juan-ruiz published this, he controls it

transfer/
    └── [from:C-123456]/       ← PROPERTY: data of the transfer, no control
    └── [amount 100]/          ← PROPERTY: data of the transfer, no control
```

The same identifier can be an object in one context and a property in another. The container defines the semantics, not the identifier itself.

## 2.14.3 The Type System

The grammar includes a formal type system for field values, expressed in BNF notation:

```bnf
<type>        ::= <primitive> | <verified> | <composite>
<primitive>   ::= "string" | "decimal" | "integer" | "boolean" | "date"
<verified>    ::= <primitive> "-length:" <integer>
               |  <primitive> "-pattern:" <regex>
               |  <domain-type>
<domain-type> ::= "iban" | "dni-es" | "nie-es" | "nif-es"
               |  "phone-e164" | "email" | "url" | "isbn"
<composite>   ::= "select-one-from" "{" <options> "}"
               |  "select-subset-from" "{" <options> "}"
               |  <type> "|" <type>
```

Domain types carry built-in validation — `iban` validates the ISO 13616 checksum, `dni-es` validates the Spanish modulo-23 check digit. A wallet that reads a `fields {}` declaration knows how to validate inputs before signing them.

## 2.14.4 The extern/ Pattern — Single Source of Truth

The `extern/` prefix references data that lives in another node, following the DRY principle — each fact has one canonical source:

```
transfer/
├── [from extern/bank/santander/accounts/[client-id:C-123456]]/
├── [to extern/bank/bbva/accounts/[client-id:B-789012]]/
├── [amount 100]/
└── [currency EUR]/
```

The transfer does not copy the client's name or IBAN — it references the authoritative source. If the client updates their IBAN, every transfer that references them reflects the change automatically. `extern/` is to Trusteando what `<a href>` is to HTML — a typed link to another node in the graph.

## 2.14.5 The Trust Type System — Monadic Propagation

The three conformity states — `b9` (brokenado), `v9` (verifiado), `t9` (trusteado) — form a type system with monadic properties. A function applied to a `b9` node can only return a `b9` result — the trust level cannot be elevated without additional verification. The result of any composition is the minimum trust level of all participating nodes:

```
t9(student) → passes through b9 function → b9(result)
v9(student) → passes through t9 function → v9(result)
```

This is not a policy rule — it is a structural property of the type system, analogous to the `Maybe` monad in Haskell. You cannot escape `b9` without genuine verification, just as you cannot escape `Nothing` without handling it explicitly.

## 2.14.6 Effects — The IO Boundary

The graph is pure and immutable — an append-only record of facts. Side effects (notifications, webhooks, push messages) are separated into an explicit `on/` namespace, analogous to the IO monad in Haskell or event handlers in HTML:

```
group/trusteando/
├── posts/                     ← pure graph — immutable facts
└── on/                        ← effects boundary (like onclick, onload in HTML)
    ├── on-new-post/
    │   └── [notify members/]/
    └── on-new-member/
        └── [webhook "https://api.example.com/hook"]/
```

Conditions within effect handlers use `when` rather than `if` — a deliberate choice inspired by Erlang guards. `when` predicates are intentionally limited to a closed set of simple comparisons: equality, numeric comparison, temporal comparison, existence checks, and quorum state. No arbitrary function calls, no external fetches, no side effects within conditions. This guarantees termination, auditability, and security.

```
on-new-state/
└── [when state=brokenado]/
    └── [action revoke-access courses/]/
```

Allowed in `when`: `state=X`, `level<N`, `since>=date`, `extern/node/exists`, `quorum-reached`.
Not allowed: function calls, arithmetic expressions, external fetches.

## 2.14.7 The Parallel with Other DSLs and Modelling Languages

Trusteando shares concepts with both programming languages and modelling languages. The table below maps Trusteando concepts to their closest equivalents — not to claim equivalence, but to make the DSL immediately accessible to developers and architects familiar with these tools:

| Programming concept | Trusteando equivalent |
|---|---|
| Class | Root folder node |
| Instance | Named subfolder |
| Attribute | `[field "value"]` |
| Primary key | `[field:value]` |
| Method | `_verify`, `_grant`, `_challenge` endpoints |
| Inheritance | `[is-a type]` |
| Composition | `[composes child]` |
| Aggregation | `[aggregates child]` / `extern/` |
| Type | `is-type iban`, `select-one-from {}` |
| Visibility | `private/` |
| Namespace | URL domain |
| Event handler | `on/on-new-post/` |
| Guard | `[when condition]` — limited by design |
| Pure function | Graph operations (immutable) |
| IO monad | `on/` namespace (effects) |
| Monad type | `b9/v9/t9` trust levels |

What Trusteando has that programming languages do not: time as a native dimension (`since/until`), cryptographic verifiability on every node, and distribution — objects live on independent servers across the web.

### The Rust parallel — ownership and borrowing

The analogy with Rust is particularly precise because Rust's ownership system and Trusteando's key hierarchy share the same structural invariant: **at any point in time, exactly one entity controls a given resource**.

In Rust, a value has exactly one owner. In Trusteando, a folder has exactly one controlling node — the one whose key was derived by `grant_key` for that path. Control cannot be duplicated or transferred without an explicit protocol act.

| Rust concept | Trusteando equivalent |
|---|---|
| `mod` | Folder |
| `struct` | Folder with `[field value]` properties |
| `pub fn` | `procedures/` subfolder with `fields {}` schema |
| `impl` | `on/` event handlers |
| `trait` | `fields {}` schema declaring required fields |
| `Result<T, E>` | `plan/` (intention) + `execution/` (outcome) |
| `Option<T>` | `private/` — may exist, access requires `grantReveal()` |
| Ownership | Controlling a folder via derived key |
| Borrowing (`&T`) | `extern/` — reference without control transfer |
| Move semantics | Key migration via `old-identities/` |
| Lifetime `'a` | `since/until` — the reference is valid for this period |

The borrowing parallel is especially clean: `extern/` references data that lives in another node without claiming control over it, exactly as a Rust reference borrows a value without taking ownership. The referenced node retains full authority over its content — the referencing node only holds a typed pointer.

Rust expresses these constraints in its type system, enforced at compile time. Trusteando expresses them in its folder structure, enforced by cryptography at verification time. Both systems make authority explicit and non-duplicable by construction — the difference is the enforcement mechanism and the distribution across servers.

### Where the analogy breaks — no move semantics, no mutation

The ownership analogy holds for control — who controls what — but breaks for data movement. In Rust, ownership can be transferred: `fn consume(x: T)` moves `x` into the function, and the caller can no longer use it. In Trusteando, ownership never moves. Data does not travel between nodes. Every `extern/` is a perpetual borrow — the referenced node retains full ownership indefinitely.

A bank transfer in Trusteando is not a movement of data. It is a signed declaration that a transfer occurred, published by whoever has authority to declare it. The balance of each account remains under the owning bank's URL. The transfer node references both accounts via `extern/` — it does not consume them.

This means there is no equivalent to `&mut T` or move semantics in the protocol. Nodes are append-only. No node can modify data published by another. This is not a limitation to be fixed — it is a deliberate design decision that preserves the most valuable property of the system: the graph is an immutable, auditable record. Every fact, once signed, is permanent evidence.

Adding mutation would require distributed consensus mechanisms to guarantee consistency — which would make the protocol fundamentally more complex and destroy the property that makes it trustworthy. The protocol is closer to **event sourcing** than to Rust: each node publishes immutable facts signed by its owner. The graph does not execute logic — it certifies results.

**The boundary is fixed.** Future versions may add hooks — optional articulation points for the ecosystem — but will not add mutation or distributed computation to the core. The power of the system emerges from composing simple immutable primitives, not from enriching them.

### The Datalog parallel — a distributed fact base

The analogy with Datalog is perhaps the deepest of all, because Trusteando and Datalog share the same foundational model: immutable facts and rules of inference.

| Datalog concept | Trusteando equivalent |
|---|---|
| Fact | Published folder |
| Rule | Relationship implied by folder structure |
| Query | Path traversal and verification |
| Recursion | Parent → child derivation chain |
| Negation as failure | `revoked/`, `until/`, absence of a folder |

In Datalog, a fact is a ground atom: `professor(juan, university)`. In Trusteando, the equivalent fact is the existence of a folder: `university.es/trusteando/professors/juan/`. The folder's presence is the assertion; its absence is the negation. Rules are implicit in the structure — if Juan is in `professors/` and in `doctors/`, the combined fact `professor_doctor(juan)` is derivable by any verifier who traverses both paths.

The recursion parallel is precise: the parent → child key derivation chain is exactly Datalog's recursive rule. Authority propagates downward through the tree the same way a Datalog rule propagates through recursive definitions.

**The critical difference** is distribution. Datalog assumes a centralised fact base and a single query engine. Trusteando distributes facts across independent servers — each node publishes its own facts under its own URL. There is no central query engine and no central fact base. Inference happens at the verifier, not at a server.

This is a strength: no single point of failure, no single authority over the complete fact base, no query bottleneck. It is also a constraint: aggregation, optimisation, and complex inference require an external layer that the protocol does not provide.

**What the protocol provides vs what implementers build:**

| Layer | Responsibility | Who provides it |
|---|---|---|
| Facts | Publish immutable signed folders | Each node — protocol core |
| Integrity | Cryptographic verification of facts | Protocol core |
| Simple inference | Path traversal, existence checks | Protocol core |
| Complex queries | Joins, aggregation, Datalog rules | Implementer layer |
| Optimisation | Indexing, caching, query planning | Implementer layer |

A query engine built over Trusteando could express rules like `active_professor(X) :- university/professors/X, not X/revoked/` and evaluate them by traversing the distributed graph. The protocol does not specify how such an engine works — but the fact model is fully compatible with Datalog semantics, and implementers building query layers over the graph are encouraged to look to Datalog as the natural formal foundation for their rule systems.

---

## 2.15 Identity as an Aggregate of Relations

A node's identity is not a single authoritative fact — it is the sum of all the relations that point to it. This principle, implicit throughout the protocol, merits explicit statement.

**Declaration as origin.** A node is born by declaring itself. The simplest possible identity is a hash of a self-description:

```
node7/trusteando/identity/
└── [declared-hash hash("person who says dni 12312312A")]/
```

This is a b9-level identity — the node says something about itself, and no one else has confirmed it. The hash is stable: it is the anchor that other nodes can reference without ambiguity.

**Corroboration as the base mechanism.** Adding a node to a folder is corroborating it. No special keyword is needed — the folder structure is the corroboration:

```
juan.es/trusteando/friends/
├── node7/since/2026-03-18/     ← Juan says node7 is his friend
├── node8/since/2026-03-18/
└── node9/since/2026-03-18/
```

Juan exercises his authority over his own space `juan.es/trusteando/`. His signature over his structure guarantees authenticity. Any verifier can check `juan.es/trusteando/friends/node7/` and confirm that Juan placed that node there. The trust level of this corroboration is Juan's trust level — no higher.

**The identity gradient.** From self-declaration to state recognition, the same mechanism operates at every level:

| Level | Example | Who signs | What it proves |
|---|---|---|---|
| Self-declaration | `node7/[declared-hash H]/` | node7 | "I say I am X" |
| Personal relation | `juan/friends/node7/` | Juan | "Juan says node7 is his friend" |
| Institutional relation | `university/professors/node7/` | university | "The university says node7 is a professor" |
| Official identity | `state/citizens/node7/` | state | "The state says node7 exists legally" |

Each level adds corroboration from a source with more established authority. The verifier decides which evidence is sufficient for their use case. For a friend of Juan, Juan's word is enough. For a bank, the state level is required.

**The graph is the registry.** There is no central database of identities — only nodes and signed edges. Node7's identity is the aggregate of everything the graph says about it: its own declaration, its friends' corroborations, its institutional memberships, its official recognitions. Identity is not stored — it is computed from the graph at query time.

---

## 2.16 Security Levels and Trust Subtrees

The three conformity levels — b9, v9, t9 — are not only semantic distinctions. They map to structurally isolated subtrees in the graph, each with its own key derivation chain and cryptographic parameters.

```
T10/
├── b9/    ← SHA-256 chain, isolated
│   └── [email:juan@example.com]/
└── t9/    ← SHA-512 chain, isolated
    └── [dni:12312312A][email:juan@example.com]/
```

T10 grants keys to both subtrees using SHA-512 in both cases — the root is never the weak link. Below that point each subtree is homogeneous and self-contained. A verifier reading a path under `T10/b9/` knows by structure — not by declaration — that it is dealing with a b9-level identity. The path is the proof.

### The weak link principle

Cryptographic strength is only meaningful up to the strength of the identity verification it protects. A system is as strong as its weakest link.

In b9, the weak link is not the hash function — it is the absence of identity verification. A node that declares "I am Pedro with DNI 123" without any supporting evidence is a self-assertion. SHA-256 or SHA-512 equally protect the integrity of that assertion — neither can make it true. Upgrading the hash function above a reasonable threshold adds no real security when the identity anchor is unverified.

Cryptographic strength should therefore be proportional to the strength of the identity verification process:

| Level | Identity verification | Hash function | Justification |
|---|---|---|---|
| b9 | Self-declaration | SHA-256 | The weak link is identity, not cryptography. SHA-256 is sufficient for integrity and attribution. |
| v9 | Email + phone verified | SHA-256 | Verification adds trust but the identity anchor remains soft. SHA-256 still sufficient. |
| t9 | DNI + electronic signature | SHA-512 | The identity anchor is legally strong. Protecting the full chain with SHA-512 is now meaningful. |

### Isolation as a security property

The subtree separation has a consequence beyond cryptography: what happens in b9 stays in b9. A t9 node never inherits or is contaminated by activity in the b9 subtree. This is expected and by design — the barrier to entry in b9 is low, there will be false identities and throwaway accounts, and the protocol does not prohibit this. It contains it. The separation is the system's hygiene mechanism.

### Separate identities, optional link

A node in `T10/b9/` and a node in `T10/t9/` are distinct identities. If Juan has both, the connection is not automatic — he declares it explicitly via `old-identities/` if he chooses, or keeps them separate. The fact that someone has a b9 node reveals nothing about whether they also have a t9 node.

### Implementation recommendation for t9 — hardware-secured keys

The protocol specifies SHA-512 for the t9 key derivation chain but cannot enforce where a private key is stored — that is outside the protocol's reach by construction, as it is with TLS. For t9 nodes, hardware-secured key storage is strongly recommended: cryptographic smart cards, TPM modules, or the secure enclave of a mobile device.

The FNMT (Fábrica Nacional de Moneda y Timbre), Spain's national certificate authority, demonstrates that this model works in production. FNMT smart cards generate keys internally — the private key never leaves the device. This is the reference implementation for t9 key security. Section 2.17 develops the full integration model.

### Gradual quantum migration

This structure provides a natural path for post-quantum cryptography adoption. When quantum-resistant algorithms become necessary, only t9 subtrees need to migrate first — because only there is the identity anchor strong enough to make the upgrade meaningful. b9 subtrees can remain on SHA-256 indefinitely: their security ceiling is set by identity verification, not by the hash function. Migration is gradual, scoped, and does not require coordinated action across the entire network.

---

## 2.17 Federated Nodes and External Roots

Not all nodes derive their key from T10. A federated node is one whose root key is chosen autonomously — not derived from T10 via `grant_key`. T10 can recognise the existence of such a node and vouch for it in the graph, but cannot verify its internal credentials. The distinction is declared explicitly:

```
uma.es/trusteando/
└── [external-root]/    ← key not derived from T10
```

The analogy is email: `@uma.es` belongs to the global namespace, but `uma.es` controls its domain independently. To verify a UMA professor's credential you ask UMA, not T10. T10 vouches that UMA exists and is recognised — not that any specific professor is valid.

### The FNMT as a canonical external root

The most important case of a federated node is an existing certificate authority. The FNMT already has a legally recognised identity verification infrastructure, a hardware-secured key system, and millions of issued certificates. Trusteando does not replace this — it integrates with it.

A citizen who already holds an FNMT certificate or DNIe has everything needed to be a t9 node from day one, without any action by T10:

```
juan.es/trusteando/identity/
├── [level t9]/
├── [hash-function sha512]/
├── [certificate-sha256 hash(cert)]/
├── [signature base64-signed-with-dnie]/
└── public_key/
```

Juan publishes his FNMT certificate at his node and signs the node with the private key associated with that certificate. The verification flow for a relying party is:

```
1. Fetch juan.es/trusteando/identity/
2. Verify Juan's signature against the certificate public key
3. Verify the certificate chain against the FNMT root
4. Check certificate revocation via OCSP
5. Extract DNI from the certificate subject
→ Juan is t9. T10 did not participate.
```

No new infrastructure is needed. No re-registration. No action by T10. The existing state-issued identity is sufficient.

### T10 does not add bureaucracy on top of existing bureaucracy

This is the key consequence: T10 recognises existing verified identities rather than replacing them. Any citizen with a DNIe, any company with an FNMT certificate, any professional with an accreditation from a recognised body — all are t9 from day one. Trusteando gives them a way to publish that identity at their own URL, add verifiable relationships, manage privacy with `private/`, and port their identity if their domain changes. The verification infrastructure already exists. Trusteando adds the graph layer on top.

### Two verification paths for t9

A t9 node can be verified via either of two independent paths:

| Path | Who vouches | Mechanism |
|---|---|---|
| Via T10 | T10 root | `verify_child_authorship`, SHA-512 chain |
| Via external root | FNMT, eIDAS CA, or equivalent | X.509 certificate chain + OCSP |

Both paths produce a t9 result. A verifier that trusts the FNMT does not need T10. A verifier that trusts T10 does not need the FNMT. The two paths are independent and mutually reinforcing — a node vouched for by both carries the strongest possible evidence.

### Scope of T10's vouching for federated nodes

When T10 recognises a federated node, it declares existence — not content:

```
T10/spain/recognised/
└── uma.es/[external-root]/    ← T10 says: uma.es exists and is recognised
```

T10 cannot verify UMA's internal credentials because it did not derive UMA's keys. A verifier who wants to confirm that Juan is a UMA professor must query UMA directly. T10's recognition is a pointer, not a guarantee of content. This distinction is always declared explicitly via `[external-root]` — its absence means the key derives from T10 and T10 can verify the full chain.

---

# 3. Credential Structure

A Trusteando credential has three fields:

Field

Description

public_key

Issuer's public key. Identifies the credential issuer in a verifiable manner.

hash(mensaje)

SHA-256 hash of the asserted content. The content can remain private—only the hash is needed for verification.

hash_verificacion

HMAC-SHA-256 del hash(mensaje) usando la clave compartida entre el firmante y su nivel superior. Solo el nivel superior puede verificarlo — confirma que la credencial pertenece a alguien reconocido por ese nivel.

auth_signature

ECDSA signature of msg_hash with the signer’s private key. Only the holder of that private key can generate it. Any third party can verify it with the signer’s public key. The root keeps a signed commitment at registration that links this key with the signer’s real identity, supporting dispute resolution with legal backing.


La credencial no es un documento — es la presencia de un nodo en una carpeta del espacio de identidad de su superior. La estructura de carpetas es la credencial. Por ejemplo:


```
universidad.es/profesores/juan-escobar          ← Juan is a Professor
universidad.es/doctores/juan-escobar            ← Juan is a Doctor
universidad.es/profesores/since/2021-09-01/juan-escobar   ← onboarding
universidad.es/profesores/until/2024-06-30/juan-escobar   ← departure

```

The temporal dimension is encoded in the folder structure itself—under the university's control, not Juan's. Juan cannot change the join or exit dates. Revoking a specific credential is simply removing the node from the corresponding folder without affecting the others.

Adicionalmente, cada credencial puede incluir un bloque de metadatos firmados:


```
{
"issuer":         "SHA-256(url_del_emisor)",
"subject":        "SHA-256(url_o_identificador_del_sujeto)",
"public_key":     "base64(clave_publica_del_emisor)",
"msg_hash":       "SHA-256(contenido_de_la_credencial)",
"hmac":           "HMAC-SHA-256(msg_hash, clave_compartida)",
"auth_signature": "ECDSA.sign(msg_hash, clave_privada_firmante)",
"timestamp":      1710000000
}

```

The message whose hash is included can have any structure. The protocol does not impose a schema. Examples of valid messages:


```
{ "role": "Profesor Asociado", "department": "Informatica" }
{ "fact": "email_sent", "from": "a@uni.es", "to": "b@uni.es", "date": "2026-03-17" }
{ "claim": "age_over_18", "verified_by": "SHA-256(universidad.es)" }
{ "presence": "SHA-256(https://venue.es)", "timestamp": 1710000000 }

```


---

# 4. The Chain of Trust


## 4.1 General Structure

El sistema organiza la confianza en niveles. Cada nivel puede emitir credenciales sobre los niveles que reconoce. La cadena tiene la siguiente forma general:


```
Trusteando root
└─ reconoce a universidad.es
└─ reconoce a juan@universidad.es (Profesor Asociado)
└─ juan firma credenciales verificables

Trusteando root
└─ reconoce a empresa.com
└─ reconoce a correo.empresa.com (servidor de correo)
└─ correo certifica comunicaciones entre empleados

```

The number of levels is not limited by the protocol. Each entity decides how deeply it organizes its internal identity space.


## 4.2 Shared Key Establishment

Each adjacent pair of levels shares a symmetric key established through an elliptic curve Diffie-Hellman exchange (ECDH). This is the same mechanism TLS uses for each HTTPS connection—proven, standardized, and well understood.

El proceso de establecimiento ocurre una sola vez por par:


```
# The lower entity generates its ephemeral key pair
privada_inferior, publica_inferior = ECDH.generate_keypair(curve=P-256)

# The upper level generates its ephemeral pair for this exchange
privada_superior, publica_superior = ECDH.generate_keypair(curve=P-256)

# They exchange public keys (authenticated channel)

# Cada parte deriva la misma clave compartida independientemente
clave_compartida = ECDH.derive(privada_inferior, publica_superior)
clave_compartida = ECDH.derive(privada_superior, publica_inferior)
# → mismo resultado en ambos lados

# The shared key is used to generate and verify the HMAC
hash_verificacion = HMAC-SHA-256(msg_hash, shared_key)

```

The fundamental property of the Diffie-Hellman exchange is that neither party knows the other's private key, and no one observing the exchange of public keys can derive the shared key. The superior level cannot impersonate the inferior because it does not have that private key.

The root does not know the shared key between the university and Juan. The university does not know the shared key between the root and other entities. Each pair has a key that is exclusively theirs.


## 4.3 Chain Verification

To verify that a credential signed by Juan as Associate Professor is valid, the verifier follows these steps:

```
1. Fetch Juan’s credential:
(public_key_juan, msg_hash, hmac, auth_signature)

2. Check universidad.es/trusteando— is public_key_juan registered?
→ yes: Juan belongs to the university’s identity space

3. Verify AUTHENTICITY—ask the university to verify the hmac:
the university checks: HMAC(msg_hash, shared_key_juan_uni) == hmac
→ the credential comes from someone recognized by the university

4. Verify AUTHORSHIP—using Juan’s public key:
ECDSA.verify(msg_hash, auth_signature, public_key_juan) == valid
→ only Juan could have generated this signature

5. Is universidad.es a recognized agent?
→ consult the root’s public registry
→ root signed: SHA-256(universidad.es) → public_key_universidad
→ valid

6. Credential verified—authenticity and authorship confirmed.

```

The verifier does not need to know the shared key between Juan and the university—it only needs the university to confirm the HMAC's validity. The university does not reveal the shared key: it merely answers yes or no to the verification query.

Note on scalability. Step 3 requires that the superior node—the university in the example—have a verification endpoint available for each query. In organizations with many members and high verification volume this can become a bottleneck. There are two complementary mechanisms to mitigate it.

The first is signed caching. The superior node periodically publishes a signed document that lists the valid HMACs for a given period along with the corresponding TTL. The verifier downloads that document once and verifies locally throughout the TTL's duration without querying the superior for each individual verification. This mechanism is already covered by the cache/ convention of the protocol.

The second is batch verification via a Merkle tree. The superior node periodically publishes the root of a Merkle tree built over all the valid HMACs for the period. The member node can then prove that its HMAC is included in that tree by presenting only its proof branch—a logarithmic set of hashes that the verifier can check locally without needing an online query to the superior and without the superior revealing the complete set of HMACs. Both mechanisms are compatible with each other and with the verification flow described in the previous steps.

A third mechanism, complementary to the previous ones, is the observer replica model with sum consensus. N independent nodes—reputed entities with verifiable identity within the graph itself: universities, public agencies, professional associations—maintain copies of the identity spaces they observe. The verifier queries K of those replicas and receives binary votes: 1 if the credential is valid according to that replica, 0 if not. The sum of votes determines the result.

The issuer publishes its folder structure without needing to know anything about the consensus. Replicas are passive observers that synchronize via cryptographic diffs—minimal deltas that describe only what has changed—instead of replicating the entire tree. This makes propagation efficient even with modest connections, and the diffs act as a native change log: storing them is equivalent to keeping the complete history of how an entity's trust has evolved.

Propagation is staggered: the fastest replicas receive the diff first and can answer queries almost immediately, while the slower ones update in the background. To avoid ambiguity during the inconsistency window, the protocol defines a hysteresis threshold: only a result that exceeds the configured threshold (for example, 80 out of 100 replicas agree) is accepted as valid. A result below the threshold is an undefined state—not partial truth—that the verifier treats as inconclusive and retries after an interval. Permanently outdated replicas can request a full synchronization from the fast replicas via pull-sync.

The analogy is an identity CDN: replicas distribute certainty just as a content distribution network distributes data. The computational load of each query is minimal—a hash arithmetic operation—which makes the system accessible to resource-constrained devices. Adding replicas increases resilience without penalizing the speed of individual queries. The cost of a query is equivalent to a standard HTTP request.

The main risk is a Sybil attack: if an attacker controls a majority of replicas, they can validate relationships that do not exist. The mitigation is structural: replicas are not anonymous nodes but entities with verifiable identity within the graph itself. Observers with opposing interests—universities from different countries, agencies from different jurisdictions—make collusion socially unlikely as well as cryptographically expensive. The formal specification of this mechanism will be addressed in future versions. Each replica's verification operation is directly expressible in terms of the TrusteandoNode class defined in section 4.11: verify_child_authorship encapsulates exactly the calculation each replica executes to validate a credential.


## 4.4 Separation Between Authenticity and Authorship

The introduction of auth_signature solves an important structural problem: without it, the superior level could impersonate the inferior. It knows the shared key, can generate a valid HMAC, and could construct a credential that appears legitimately attributed to Juan. With auth_signature that is impossible—the private key of Juan never leaves his device.

Las dos propiedades son verificables independientemente:

- Authenticity — verified by the superior level via the HMAC. Answers: does this credential come from someone I recognize?
- Authorship — verified by any third party using the signer's public key via ECDSA. Answers: was it specifically this person and not someone else?
This separation makes the trust chain robust against internal compromises: even if a superior level is malicious or compromised, it cannot fabricate credentials attributable to its lower agents. Section 4.11 formalizes this mechanism in terms of the TrusteandoNode model: respond_to_challenge guarantees authorship because only the key holder can produce the correct response, and verify_child_authorship allows the parent to verify without the child's participation.


## 4.5 Non-repudiation and Dispute Resolution

The ECDSA signature establishes the technical evidence of authorship. However, full non-repudiation—the impossibility of denying authorship even when neither party wants to assume responsibility—requires more than cryptography.

At registration, the signer submits a responsibility commitment to the root: they accept that the registered private key belongs to them and that they assume responsibility for the credentials signed with it. The root safeguards that commitment.

In the event of a dispute where neither party wants to assume authorship:


```
1. The root presents the commitment signed by Juan at registration
2. The cryptographic evidence links that commitment to the disputed credential
   via auth_signature verifiable with the registered public key
3. Responsibility is established even if Juan refuses to acknowledge it

```

This mechanism explicitly acknowledges that non-repudiation is not just a cryptographic problem. Cryptography provides the evidence—the legal framework gives it binding force. It is the same model used by qualified electronic signature systems under the eIDAS regulation in the European Union. The protocol provides the evidence; the institutional framework gives it force. Both layers are necessary and neither replaces the other.


## 4.6 The Role of the Root Node

The root has three responsibilities:

- Once per agent: establish the shared key with the entity via ECDH and publish in its registry the association between the entity_id of the entity and its public key.
- Maintain a signed public registry of recognized agents, updated periodically with a TTL that allows verifiers to cache it locally.
- Safeguard the registered emergency keys and approve identity migrations when presented with valid evidence.
The registration of emergency keys with the root is asynchronous—a node can publish its hash_publico on its own web space and operate normally before the root records it. The root processes registrations according to its capacity. The system does not block in the meantime.


## 4.7 Identity Resilience — the Emergency Key

Each node generates at creation time an emergency key that it never shares with its superior or with any other node. With it, it computes its hash_publico:


```
clave_emergencia  ← generated locally, never shared with the superior
hash_publico = hash(entity_id, clave_emergencia)

```

hash_publico is published on the node's own website as part of its public identity, at a standard path that any protocol implementation knows to look for:


```
universidad.es/profesores/juan-escobar/hash_publico/a3f9e2b1c4d7...

```

Any third party can see this value. Only Juan can prove he generated it, because only he knows the clave_emergencia that produces it. hash_publico is anchored in two independent places: the node's own website and, when the root registers it, the central registry. If either one fails, the other sustains the chain.

Voluntary migration to a new superior

When Juan wants to migrate to a new university, the process is:


```
1. Juan reveals his clave_emergencia to universidad-b.es

2. Universidad-b verifies against the hash_publico published by Juan:
   hash(entity_id_juan, clave_emergencia) == hash_publico
   → the adoption is legitimate and voluntary—Juan handed over the key

3. Universidad-b informs the root that it knows Juan's clave_emergencia.
   Root verifies: hash(entity_id_juan, clave_emergencia) == hash_publico
   → valid equation: migration confirmed. There is no discretion.

4. Universidad-b creates Juan's entry in its space:
   universidad-b.es/profesores/juan-escobar/

5. Universidad-b records the link in its own old_identities table,
   under its control—not Juan's:
   universidad-b.es/old_identities/
   hash_nuevo → [ hash_nuevo, hash_antiguo_1, hash_antiguo_2, ... ]

   The complete chain of previous identities is published in full.
   Universidad-b received it from universidad-a during the adoption process,
   adds the new entry, and signs it with its own key.

6. Juan generates a new clave_emergencia and publishes a new hash_publico.
   The previous key is invalidated.

```

The old_identities table is an institutional record of the university, not of Juan. Juan cannot modify it or omit entries. It is signed by the university with the same authority as any other credential it issues.

```
The role of the root in this process is purely algorithmic: it executes the public function hash(entity_id, clave_emergencia) == hash_publico and returns true or false. It has no discretion to deny a migration whose equation holds, nor to approve one whose equation does not hold. Any node executing the same function on the same data would obtain an identical result. The root does not hold any privileged state—it only executes a public algorithm.
```

Publishing the entire chain—not just the immediate previous link—allows any agent in the system to verify Juan's complete history with a single query, without having to jump from server to server following each link one by one. Each migration extends the chain by an element; the receiving university inherits it, extends it, and publishes it entirely signed.

Temporary adoption by the root when the superior disappears

If Juan's superior disappears—the university closes, loses the domain, or is revoked—Juan can identify himself to the root with his clave_emergencia:


```
1. Juan presents his clave_emergencia to the root
2. Root verifies: hash(entity_id_juan, clave_emergencia) == registered hash_publico
3. Root welcomes Juan as an orphan node under its direct guardianship
4. Juan operates normally while he finds a new superior
5. When a new superior adopts him, he follows the standard migration process

```

This property guarantees that a node's identity survives the disappearance of its superior. It does not start from zero—it continues with its history, its previously verifiable credentials, and its accumulated reputation. It is analogous to telephone number portability, but for cryptographic identity: the holder controls portability, not the operator.

Graceful degradation

If the root is saturated or unavailable, existing nodes continue operating normally. Verifications do not require the root—they consult the entities' websites directly. Only new registrations and migrations are blocked, not the system's everyday use.


## 4.9 Distributed Trust Roots

Over time, entities with sufficient public recognition become natural trust roots. A verifier who knows and trusts a university can verify credentials of its professors without consulting the Trusteando root.

The system evolves from hierarchical to polycentric organically. The Trusteando root does its job well when it stops being necessary for everyday use—when the trust network has enough nodes with their own reputation to sustain itself without a single central authority.


## 4.10 Multiple Roots and Identity Independence

This section extends the protocol's cryptographic model to incorporate witness redundancy and the decoupling of identity from the web infrastructure. It does not replace the base model—it complements it. An entity can have URL-derived identity, standalone identity with its own secrets, or both linked through old_identities/.

4.10.1 Identifier independence

In the base model the entity_id is derived from the canonical URL. This pragmatic decision anchors identity in a pre-existing web presence but creates dependency on DNS infrastructure and domain continuity. For entities requiring maximum resilience, the protocol allows an alternative mode: autonomous identity. The entity generates its identifier locally and a secret that only it knows:


```
object.id          = hash("universidad-de-málaga")
object.secret_base = hash("frase secreta conocida solo por la universidad")
```

The URL becomes the publication anchor—where the entity publishes its data—but not the identity anchor. Identity can exist before there is a URL, can survive domain loss and DNS blocking, and can persist even if the original root disappears.


4.10.2 Multiple secrets for multiple contexts

An entity can generate different secrets for different purposes while maintaining their isolation:


```
secret_identidad  = hash("frase para identidad principal")
secret_academic  = hash("phrase for academic credentials")
secret_emergencia = hash("frase para migraciones y rescate")
```

Compromising one secret does not affect the others. Each secret generates its own cryptographic derivation chain. The same entity can have a civil identity recognized by governmental roots, a professional identity recognized by collegiate roots, and an operational identity recognized by technical roots, without the compromise of one affecting the others.


4.10.3 Multiple root network

Since the root is a trivial public algorithm, any node with sufficient reputation can act as root. There is no fundamental distinction between a root and a high-reputation node: both execute the same public algorithm. This allows an ecosystem with thousands of potential roots. The entity randomly chooses a set of N roots (for example 5) and sends its request to each one. For each root i and each secret j a specific public key is derived:

```
public_key[i][j] = hash(object.id + secret_j + root[i].secret)
```

The result is a verification matrix of dimensions roots × secrets. Each cell is an independent proof that the entity knows secret j and has been recognized by root i. There is no mandatory root. The geographical, jurisdictional, and organizational diversity of the chosen roots determines resilience: an attacker controlling one jurisdiction or compromising one organization only affects the roots in that scope.


4.10.4 Validation quorum

For sensitive operations—issuing high-value credentials, migrating identity, resolving disputes—the entity can require a quorum of K roots (with K ≤ N). An operation is valid if it presents valid signatures from at least K independent roots. Using the protocol grammar:


```
operacion/[quorum 3]/
[testigo root-23]/
[testigo root-45]/
[testigo root-78]/
```

A verifier who finds the [quorum 3] marker knows they must check at least 3 signatures from different roots before accepting the operation. The same mechanism applies to intermediate certifier nodes replicated across multiple instances: modifying a critical field—such as the issuance date of a degree—requires the consensus of the instance quorum, not the control of a single one.

4.10.5 Resulting properties

- Local compromise resilience. An attacker who compromises a root can falsify that root's keys but not the others. If the required quorum is 3 and there are 5 roots, the attacker needs to compromise 3 independent roots, each with its own security, governance, and location.

- Infrastructure independence. Identity survives domain changes, web server loss, DNS blocking, and the disappearance of the original root.

- Scalability of trust. Any node with reputation can offer itself as a root. There is no upper limit. Entities can choose their roots at random, by geography, institutional scope, or any other criterion.

- Context separation. The different secrets allow the same entity to have identities in different institutional contexts without the compromise of one affecting the others.

This extension turns Trusteando into a polycephalic system by design from the moment of registration. There is no single point of failure, no single authority on which an identity's existence depends. Identity belongs to the subject, backed by a quorum of independent witnesses executing a public algorithm. The result is a system that inherits the robustness of distributed consensus systems without needing a blockchain, because the proof of existence is the multiplicity of independent signatures, not a shared immutable chain.

### 4.10.6 Dual institutional key — independent verification layers

An extension of the multiple-roots model for federated institutional nodes (section 2.17). When an institution (UMA, a bank, a public agency) issues a credential to a child node, that child can receive two independent keys from two independent sources:

```python
K_institutional = institution_node.grant_key("path/to/child")
K_T10           = T10_node.grant_key("institution.es/path/to/child")
```

The institution is legally responsible for `K_institutional` — it derives from the institution's own key chain. `K_T10` is an independent verification layer derived from T10's key chain using the full institutional path as the derivation segment. Neither key reveals the other.

The two keys are declared in the child node's identity:

```
uma.es/trusteando/professors/juan-ruiz/
├── [key-institutional hash(K_institutional)]/
├── [key-t10 hash(K_T10)]/
└── since/2021-09-01/
```

A verifier can check either key independently. The trust outcome depends on which sources the verifier trusts and which keys are valid:

| K_institutional | K_T10 | Result |
|---|---|---|
| Valid | Valid | Maximum trust — two independent sources agree |
| Valid | Invalid | Valid with notice — T10 does not recognise the node |
| Invalid | Valid | Suspicious — institution key fails, T10 sees something the institution does not |
| Invalid | Invalid | Rejected |

This is analogous to a document bearing both the issuer's signature and an independent notarial seal. The document is valid with only the issuer's signature — the notarial seal adds an independent verification layer that is meaningful precisely because it is independent.

**Relation to federated nodes.** For nodes with `[external-root]` (section 2.17), T10 cannot verify internal credentials. The dual key model is the mechanism by which T10 can provide an independent verification layer even for federated nodes, without requiring those nodes to derive their keys from T10's chain. The institution retains full sovereignty; T10 adds a parallel witness.

**Open design questions.** The synchronisation process between `K_institutional` and `K_T10` — how T10 learns about a new child node issued by a federated institution and derives the corresponding `K_T10` — is an open design question. Two candidate approaches are: (a) the institution notifies T10 when issuing a new credential and T10 derives and publishes the corresponding hash; (b) T10 derives `K_T10` on demand when a verifier queries it, without prior notification. Approach (b) is stateless and requires no synchronisation, but requires T10 to be online for every new verification. This will be formalised in v0.3.

---

# 4.11 The Protocol Core: Four Functions

Trusteando is built on a premise as simple as it is profound: each folder in the graph is a node, and each node has a private key derived directly from its path in the hierarchy. There is no separate public key infrastructure. There are no certificates. There are no external authorities. The folder structure is the cryptographic infrastructure. The entire protocol boils down to four functions that fit in a few lines of code. Although they seem simple, they solve all the problems raised without special cases or loose ends.


```
class TrusteandoNode:
def __init__(self, key):
    self.key = key

@staticmethod
def reduce_hash(seed, elements):
    """Primitive operation: left fold hash"""
    result = seed
    for element in elements:
    result = hash(result + element)
    return result

def grant_key(self, child_path_segment):
    """El padre otorga una clave al hijo"""
    return hash(self.key + child_path_segment)

def respond_to_challenge(self, context_elements):
    """El hijo responde a un reto sin revelar su clave"""
    return self.reduce_hash(self.key, context_elements)

def verify_child_authorship(self, child_path_segment, context_elements, proof):
    """Only the parent can verify a child's authorship"""
    child_key = self.grant_key(child_path_segment)
    return self.reduce_hash(child_key, context_elements) == proof

```


## 4.11.1 reduce_hash: the Primitive Operation

reduce_hash is the only cryptographic operation the protocol needs. It takes a seed and a list of elements and returns a hash that depends on all of them in order. It is deterministic (same inputs produce the same output), one-way (the output does not allow recovering any input), order-sensitive (reduce_hash(s, [a,b]) differs from reduce_hash(s, [b,a])), and composable (it can be verified in stages).


## 4.11.2 grant_key: the Parent Grants Keys to the Child

When a parent node incorporates a child with a path segment, it grants it a derived key: child_key = hash(parent_key + child_path_segment). The parent delivers child_key to the child over a secure channel. From that moment, the child knows its key, the parent can recompute it when necessary, and no one else knows it. The child never learns the parent's key—the direction of trust is strictly downward.


## 4.11.3 respond_to_challenge: the Child Responds Without Revealing its Key

When a node wants to prove its identity for a specific context, it generates a proof: proof = reduce_hash(self.key, [verifier_id, hash(content), timestamp]). The node does not reveal its key; it only demonstrates that it knows it. The proof is specific to that verifier, that content, and that moment—it cannot be reused or transferred.

Example: a professor posts a comment on an external forum with their professor_id. The moderator wants to verify that it really is that professor. The moderator sends the professor: [moderator_id, hash(comment), timestamp]. The professor calculates respond_to_challenge([moderator_id, hash(comment), timestamp]) and sends the response. The moderator forwards it to the parent node—the one in the professor_id path—along with the context. The parent verifies.


## 4.11.4 verify_child_authorship: Only the Parent Verifies

The external verifier does not choose arbitrarily whom to ask—it always queries the parent node of the child that issued the proof. If the identifier is uma.es/profesores/juan, the verification request goes to uma.es/profesores. The parent recalculates the child's key with grant_key and verifies the proof with reduce_hash. If they match, the proof is authentic. The child does not participate in the verification.


## 4.11.5 Security Properties Emerging from Simplicity

Non-repudiation: only the key holder can produce a correct respond_to_challenge. Non-reusable: context_elements includes verifier_id and timestamp, unique per interaction. Non-transferable: the response is valid only for that specific set of elements. No key disclosure: reduce_hash is one-way; the response does not allow recovering the key. Parent verification: verify_child_authorship recalculates without needing the child. Trust hierarchy: grant_key is the only way to create a valid child node, always from parent to child. External verification: any system—forums, mail, documents—can use the same mechanism by sending a challenge and verifying the response with the parent. There are no special cases. There is no hidden state. The entire system is deterministic, auditable, and verifiable by construction.

**Verification is O(1) in chain depth.** A common misconception is that verifying a deeply nested node requires traversing the entire chain from root to leaf. This is incorrect. `verify_child_authorship` requires exactly two interactions regardless of chain depth: the verifier receives the proof from the child node, and makes a single HTTP request to the immediate parent. The parent computes `grant_key(child_path)` and `reduce_hash(child_key, context)` locally — two hash operations — and returns the result. The depth of the hierarchy (root → country → university → faculty → professor) is irrelevant to verification cost. The only case where chain depth matters is initial trust discovery — establishing that a previously unknown entity is recognised by a root — and this is resolved once per entity with a cacheable signed result (section 4.3).

### The three verification cost cases

The actual cost of verification depends on what the verifier already knows. There are three distinct cases:

**Case 1 — Known parent (cost: near zero).** The verifier already trusts `universidad.es`. Verifying a student's credential is one HTTP request to the university and two hash operations. This covers the vast majority of everyday interactions — local or single-level hierarchies where prior trust exists.

**Case 2 — Unknown parent, known root (cost: logarithmic, one-time).** The verifier does not know `universidad.es` but knows `ministerio.es`. They must walk the chain upward until reaching a node they trust. This traversal is O(log n) in hierarchy depth, but it happens only once per unknown entity — the result is cached with a signed TTL. Subsequent verifications of the same entity cost the same as Case 1.

**Case 3 — Autonomous node (cost: trust establishment).** A node that generated its own key has no parent to verify against. Trust must be established through the quorum mechanism (section 4.10) or by a recognised root publishing the node in its registry. Once established, verification returns to Case 1.

The practical consequence: for 90% of interactions — those within a known domain or between entities with prior trust — verification cost is a single HTTP request and two hash operations. The chain traversal cost only appears on first contact with an unknown entity, and only once. This is the same model used by TLS certificate chains: the chain is verified once, the session is cached.


---

# 5. Two Levels of Existence

An entity can operate at two distinct levels in the system:

Level

Description

Declared identity

The entity publishes its hashes and keys on its own URL. The chain is coherent and verifiable by anyone who knows the entity. There is no external guarantee for verifiers who do not know it. Useful for internal systems, closed communities, or for building the chain before requesting formal recognition.

Recognized identity

The root has established a shared key with the entity and published its agent certificate. The chain is verifiable by any third party without prior knowledge of the entity. Trust is transitive to the lower levels.

An entity can operate at level 1 indefinitely. Level 2 adds verifiability for unknown third parties—it is not required for the system to function.


---

# 6. Privacy and Selective Disclosure


## 6.1 Privacy by Default

The verifier sees only the verification result: the credential is valid or it is not. The complete trust chain—what entities are involved, what the original message states—remains hidden unless the credential holder decides to reveal it explicitly.

The message hash allows verifying integrity without revealing the content. The issuer can prove the message has not been altered without showing the message itself.


## 6.2 Selective Disclosure

The credential holder can choose what to reveal in each context:

- Only the result: the credential is valid—without revealing who issued it or what it states.
- The issuer but not the content: this credential comes from universidad.es—without revealing the specific role.
- The entire chain: universidad.es certifies that Juan Escobar is Associate Professor of the Computer Science Department.

This property enables use cases such as proving majority age without revealing the birthdate, or proving membership in an organization without revealing the position.


## 6.3 What the System Does Not Hide

The fact that an entity is registered as an agent in the root is public—it is part of the recognized agents registry. The entity_id (hash of the URL) of an entity is derivable by anyone who knows its URL. The protocol is not designed to hide the existence of participating entities, only to protect the content of the credentials they issue.


## 6.4 Dynamic Privacy via Manager

The private/ folder is a static mechanism: the information exists on the server but is not exposed by default. For cases requiring dynamic access control—where the decision to reveal depends on the context of each request—the protocol allows a manager model. The manager is software that receives the request, identifies who is asking, decides whether to grant access, and serves the information only on demand. In Trusteando this manager is the agent that controls the private/ folder, implemented via requestReveal() and grantReveal(). The static URL is the model for public information; the manager is the model for private information on demand. The two models are complementary and can coexist within the same node.


---

# 7. Use Cases

The protocol is agnostic to the content of credentials. The following use cases illustrate the generality of the mechanism—they are not the only possible ones.


## 7.1 Academic and Professional Credentials

A university registers its professors in its identity space. A medical college registers its members. A law firm registers its members. Any verifier can check the credential without calling a central server — by consulting the issuer's URL and verifying the signature.

The student does not receive a copy of the credential — they receive a key that grants access to it. The university publishes the credential under a `private/` folder. Only the student (who holds the key) can reveal it via `grantReveal()`. Different verifiers can be granted access to different subsets of the student's academic record — a prospective employer sees the degree, a library sees only enrolment status — and access can be revoked at any time. The student's control over their own data is not an add-on; it is built into the folder structure from the start.

This also means interoperability requires no bilateral agreement. If a recruiter's platform implements a verifier that reads `uma.es/trusteando/degrees/[hash]/` and verifies the UMA's signature, it works with any university that publishes in the same schema. No contract, no API negotiation, no custom integration — only convergence on the same folder structure.


## 7.2 Presence-Verified Reviews

A venue registers temporary presence tokens—for example using a rotating QR code available only on-site. A user who was physically present can obtain a presence token and use it to sign a review. The review carries a presence credential that anyone can verify: this person was at this place.

This is the use case that motivated the initial development of the protocol. Presence verification is a particular case of a verifiable credential—not a separate mechanism.


## 7.3 Communication Verification

A company's mail server, registered as a delegated agent by the company, can issue credentials certifying that an email was sent between two parties on a specific date. The email's content can remain private—only the message hash and the sending credential are needed for verification.


## 7.4 Condition Verification Without Revealing Data

An authority over personal data—a civil registry, a university, a health system—can issue condition credentials: over 18, university graduate, registered healthcare professional. The verifier receives confirmation of the condition without accessing the underlying data.


## 7.5 Delegation of Authority Within Organizations

A company can register its divisions, departments, or internal systems as delegated agents. Each can issue credentials within its scope. The delegation chain is externally verifiable without exposing the organization's internal structure.


## 7.6 Verifiable Authorship of Content

An author can sign a document, article, or any digital content with their Trusteando credential. Any verifier can confirm that the content was produced by that person and has not been altered since, without needing additional trusted third parties.


## 7.7 Multi-Agency Emergency Coordination

Trusteando's hierarchical structure not only serves static identity—it is a natural model for representing command and coordination structures in real time. In incidents involving multiple agencies—police, firefighters, emergency medical services, civil protection—the ability to publish and read operational statuses in structured folders removes the friction of radio communication and the loss of information during handovers.

Each responder is a node. Each team publishes its location, status, and needs under its own URL. The incident command publishes orders using the plan/execution convention: plan/ contains the objective and timeline; execution/ is updated by the teams with their actual progress. The absence of execution/end/ indicates the order remains active. Coordination requests between agencies—for example police requesting that firefighters clear an access—are folder entries that both parties can see, accept, and execute with full traceability.

```
emergencias.madrid.es/incidents/2026-03-18/incendio-xxx/
├── command/
│   ├── orders/order-001/
│   │   ├── plan/      ← objective, deadline, assigned teams  (signature: command)
│   │   └── execution/ ← teams, incidents, end/ when complete  (signature: team)
├── firefighters/
│   ├── deployment/teams/            ← who is where
│   └── requests/police/             ← inter-agency coordination
├── police/
└── medics/
```

The result is full traceability: every action, every order, every status change remains signed and timestamped. Command sees the real-time status of all teams on a single screen without a radio call. During a handover, the incoming team checks the execution/ of the outgoing team and continues without losing information. After the incident, the complete record is verifiable and serves as the basis for forensic reconstruction.

An additional advantage is the speed of organizational design. Faced with a new emergency—a rare incident type, an unfamiliar geographic area, an unusual combination of agencies—a language model can generate in seconds a complete folder structure proposal tailored to the context: which nodes are involved, what folders each needs, and how command relationships and inter-agency requests are expressed. The human team evaluates the proposal, adjusts it if necessary, and the structure is operational immediately. What once required meetings and procedure documents can be resolved in minutes, with the protocol as a common language that both humans and automated systems understand without ambiguity.


## 7.8 Interoperability Between Administrations and Organisations

Interoperability between public administrations is still an unresolved structural problem. Every pair of agencies that needs to exchange data requires a bilateral agreement, a specific exchange format, and a technical gateway that translates between heterogeneous systems. The integration cost grows with the number of actors. Trusteando inverts this dynamic thanks to the protocol's nature: any system—a relational database, an existing API, a legacy registry—can expose its data through an interface that respects the folder schema without needing to migrate or rewrite its internal infrastructure. Once two agencies respect the same schema, they are interoperable by construction. There is no need to negotiate any additional agreement.

The fundamental shift is moving from exchanging documents to publishing facts. An agency does not issue a certificate in PDF—it publishes a verifiable claim under its URL. Any other agency that needs that data does not request a document: it queries the URL directly, verifies the signature, and obtains a fact that can be processed automatically. The read permission is implicit in the publication. There is no need for a specific authorization for each query.

Example: an agency needs to verify that a person has a university degree, a master's, and is a registered professional. Today: three queries to three different systems, three formats, three authorizations, and manual data merging. With Trusteando: three URLs, the same format, the same verification, and an automatically coherent result because each datum is signed by the authority over it.

The adapter layer (section 11) allows agencies with legacy systems to participate without rewriting their infrastructure: the adapter exposes the same Trusteando interface over any internal database. For the external verifier, consulting a municipality's civil registry is identical to consulting a university's credential registry. Internal complexity is encapsulated; the public interface is uniform.

Small agencies without their own technical capacity can delegate publishing to a superior node—a provincial government, an aggregator agency—that acts as an infrastructure provider. The data is published under the delegating agency's URL, signed in a way that allows verifying its origin. Interoperability does not require each actor to have its own infrastructure.


## 7.9 Agreements and Contracts as Graph Nodes

An agreement between two or more parties can be represented as an independent node in the graph. Following the OOP analogy of section 2.11, the agreement is the object, the participants are its properties, and plan/execution are its methods. Each party publishes the agreement in its own space, and the match between both publications constitutes the proof of bilateral consent.


```
acuerdo-servicios-2026-xyz/
├── [state trusteado]/              ← object.state
├── participants/                    ← object.collection
│   ├── [empresaA proveedor]/         ← object.property (immutable)
│   └── [empresaB cliente]/           ← object.property (immutable)
├── plan/                            ← object.method (intention)
│   ├── [objective "consulting services 2026"]/
│   └── [state approved]/
├── execution/                       ← object.method (accumulated facts)
│   ├── firma/since/2026-03-18/
│   └── hito-1/[state completed]/since/2026-06-15/
└── private/             ← economic conditions, private
```

Each party publishes an identical copy in its own space:

```
empresaA.es/trusteando/acuerdos/acuerdo-servicios-2026-xyz/
empresaB.es/trusteando/acuerdos/acuerdo-servicios-2026-xyz/
```

A verifier that consults either of the two URLs can verify that party's signature, confirm that the other party has published the same structure, check that the signing dates match, and conclude that the agreement is bilaterally valid. If the structures differ, there is a dispute. If a party has not published, the agreement is not completed. The protocol does not need an external system to record contracts—the concordance between two nodes is the proof.


---

## 7.10 Individual User Registration

The three conformity levels — b9, v9, t9 — are not only states that a node can reach organically. They can also be the explicit target of a registration flow, where a node operator verifies a user's identity to a defined level and publishes the result in a standard path.

The canonical path structure for individual registration under a national node is:

```
T10/spain/register/
├── b9/
│   └── [email:juan@example.com]/
│       └── since/2026-03-23/
├── v9/
│   └── [email:juan@example.com][phone:+34612345678]/
│       └── since/2026-03-23/
└── t9/
    └── [dni:12312312A][email:juan@example.com][phone:+34612345678][signature:true]/
        └── since/2026-03-23/
```

Each level corresponds to a distinct identity verification process:

| Level | Verification | Identity anchor |
|---|---|---|
| b9 | Email confirmed | Self-declaration — the weak link is identity, not cryptography |
| v9 | Email + phone confirmed | Soft anchor — contact reachable but not legally bound |
| t9 | DNI + electronic signature | Hard anchor — legally bound, non-repudiable |

The fields inside each path segment are the verified data points for that registration. Field ordering follows alphabetical convention (see style guide section 18.3). The node operator controls which fields are required at each level.

### The `[instance:N]` convention for duplicate registrations

When two registrations arrive with identical identifiers — the same email, the same phone — the node assigns an `[instance:N]` suffix to disambiguate. The common case carries no suffix. Only duplicates are numbered:

```
T10/spain/register/b9/[email:juan@example.com]/              ← first (no suffix)
T10/spain/register/b9/[email:juan@example.com]/[instance:2]/ ← second
```

The instance suffix is part of the path segment used to derive the child key via `grant_key`. Two registrations with the same email therefore have cryptographically distinct keys — holding one grants no authority over the other. The node assigns the instance number from an internal counter; the user cannot choose it.

When a second registration is detected, the node should warn the user: another registration exists with the same identifier. This may indicate an error, a duplicate account, or an attempt at impersonation.

### t9 registration via existing certificate infrastructure

For t9 registrations, no new infrastructure is required if the user already holds a state-issued certificate. A citizen with a DNIe or FNMT certificate signs their registration node with that certificate. The node operator verifies the signature against the certificate chain (see section 2.17). T10 does not participate — the FNMT is the external root that vouches for the identity.

This means any citizen with a DNIe is a potential t9 node from day one. Registration is the act of publishing that identity at a URL, not the act of obtaining verification — the verification already exists.

### Subtree isolation by level

Because b9, v9 and t9 registrations live in structurally isolated subtrees (section 2.16), what a user registers at one level does not automatically connect to registrations at other levels. A user who registers at b9 and later at t9 has two independent nodes. The link between them is optional and declared explicitly via `old-identities/` if the user chooses to establish it.

---

## 7.11 The Wallet as a Universal Key Manager

The wallet is not only an identity tool. It is a key manager for any verifiable action — any operation where a node needs to prove it authorised something. Identity is the first use case; it is not the only one.

The pattern is always the same: a node publishes a `fields {}` schema declaring what a verifiable action looks like. The wallet reads the schema, presents the appropriate form to the user, validates the inputs, and signs the completed action with `respond_to_challenge`. The receiving node verifies with `verify_child_authorship`. No bilateral agreement is needed between the wallet and the node — the schema is the contract.

### Canonical example — bank transfer

```
bank/santander/user-juan-ruiz/transfer/fields {
    amount       is-type decimal-positive
    concept      is-type string
    currency     is select-one-from { EUR, USD, GBP }
    date         is-type date
    from-account is-type iban
    to-account   is-type iban
}
```

The user opens the wallet, navigates to `bank/santander/user-juan-ruiz/transfer/`, and the wallet reads the schema. It knows to present an IBAN field for `from-account`, validate the format before signing, and present a currency selector with exactly three options. The user fills in the form and approves. The wallet signs with `respond_to_challenge`. Santander verifies with `verify_child_authorship`.

The structural consequence is significant: two banks that publish the same `transfer/fields {}` schema are interoperable by construction. A wallet that works with Santander works with BBVA without any integration work. The schema is the interoperability contract. No bilateral agreement between banks is required — only adoption of the same schema.

### The interface disappears

Today: app → menus → proprietary form → bank's internal system.

With Trusteando: the wallet reads `bank/santander/user-juan-ruiz/transfer/fields {}`, presents a standard form, and signs. The same interaction works at any bank that publishes the same schema. The interface is generated from the schema — it does not need to be built separately for each institution.

### Other cases

The same pattern applies to any verifiable action:

| Schema path | Action |
|---|---|
| `hospital/cita-previa/fields {}` | Book a medical appointment |
| `ayuntamiento/padron/fields {}` | Register as a resident |
| `universidad/matricula/fields {}` | Enrol in a course |
| `seguridad-social/prestacion/fields {}` | Apply for a benefit |
| `hotel/reserva/fields {}` | Make a hotel booking |
| `notario/contrato/fields {}` | Sign a notarised contract |

In each case: the institution publishes the schema, the wallet reads it, the user signs, the institution verifies. No custom integration. No proprietary app. The wallet is the universal client for any institution that speaks the protocol.

### Connection to interoperability by design

This is the practical expression of section 2.9. The folder structure is the interoperability contract — not at the level of identity declarations, but at the level of executable actions. Any institution that exposes its operations as `fields {}` schemas becomes interoperable with any wallet that reads those schemas, without negotiation, without connectors, without bilateral agreements.

---

## 7.12 Declarative Workflow Engine

The `plan/execution` convention (section A.24) is sufficient to express simple processes. For multi-step workflows with sequential dependencies, error handling, and distributed execution, the `steps/` pattern provides a complete declarative model.

### Structure

```
process-name/
├── plan/
│   ├── [state approved]/
│   └── steps/
│       ├── 01-first-step/
│       ├── 02-second-step/
│       └── 03-third-step/
└── execution/
    └── steps/
        ├── 01-first-step/[state completed]/since/2026-03-22/
        ├── 02-second-step/[state in-progress]/since/2026-03-23/
        └── 03-third-step/[state pending]/
```

`plan/steps/` declares the steps in order. `execution/steps/` records actual progress. Numeric prefixes (`01-`, `02-`, `03-`) imply sequentiality — no explicit dependency declarations are needed. A plan reaches `[state completed]` when all its steps are completed.

### Step lifecycle

Each step moves through a defined set of states:

| State | Meaning | Can transition to |
|---|---|---|
| `[state pending]` | Not yet started | `in-progress` |
| `[state in-progress]` | Currently active | `completed`, `failed`, `aborted` |
| `[state completed]` | Finished successfully | — |
| `[state failed]` | Did not complete — retriable | `completed`, `aborted` |
| `[state aborted]` | Cancelled definitively | — |

If a step has `[state failed]` or `[state aborted]`, subsequent steps do not start. The history of all states is preserved — the graph is append-only, failed states are never deleted.

### Error handling and retry

```
execution/steps/
├── 01-contract/[state completed]/since/2026-03-22/
├── 02-social-security/
│   ├── [state failed]/since/2026-03-23/
│   │   └── [reason "Expired ID document"]/
│   └── [state completed]/since/2026-03-24/   ← successful retry
└── 03-system-access/[state pending]/
```

The failed attempt remains in the graph permanently. The retry is a new `[state completed]` entry — it does not erase the failure. Any auditor can reconstruct exactly what happened and when.

### Canonical example — employee onboarding

```
onboarding-juan-ruiz-2026/
├── plan/
│   ├── [state approved]/
│   └── steps/
│       ├── 01-contract/
│       ├── 02-social-security-registration/
│       ├── 03-system-access/
│       └── 04-welcome/
└── execution/
    └── steps/
        ├── 01-contract/[state completed]/since/2026-03-22/
        ├── 02-social-security-registration/[state completed]/since/2026-03-23/
        ├── 03-system-access/[state in-progress]/since/2026-03-24/
        └── 04-welcome/[state pending]/
```

Each step is signed by whoever completed it. The state of the entire process is readable at any moment without querying any system — the graph is the state.

### Advantages over centralised workflow systems

| Property | Explanation |
|---|---|
| Auditable | The exact state of every step is visible at any moment |
| Verifiable | Each completed step is signed by whoever executed it |
| Distributed | Each step can live on a different node |
| Temporal | `since/` records exactly when each state was reached |
| Resilient | No central server — the graph is the source of truth |
| Append-only | History cannot be altered — failures are permanent record |

### Use cases

The same pattern applies to any multi-step process with sequential dependencies and audit requirements: loan approvals, academic certification, administrative procedures (permits, licences), multi-party contracts with sequential approvals, procurement with multiple validations, clinical trial protocols.

### Automatic notifications with `on/`

Steps can trigger notifications when their state changes:

```
on/
└── on-new-state/
    └── [when state=completed]/
        └── [notify next-step-responsible/]/
```

This connects the workflow engine to the event system (A.26) without adding new primitives — `on/` and `steps/` compose naturally.

---

## 7.13 Canonical Domain Pattern — Authority, Agent, Subject, Resource

The folder structure is an organisational modelling language. Any domain that involves multiple parties with distinct roles and responsibilities can be expressed as a Trusteando structure. This section defines a canonical four-role pattern and illustrates it with a concrete case.

### The four roles

| Role | Responsibility | Publishes |
|---|---|---|
| **Authority** | Accredits agents and defines standard resources | Registry of recognised agents and resources |
| **Agent** | Performs actions within their accredited scope | Actions referencing subjects and resources |
| **Subject** | Receives the effects of actions | References to actions affecting them |
| **Resource** | The object of the action | Its own state and history |

```
authority.es/trusteando/
├── registry/agents/      ← accredited agents
└── registry/resources/   ← standard resource definitions

agent.es/trusteando/
└── actions/[id]/
    ├── plan/             ← declared intention
    │   ├── [subject extern/]/
    │   ├── [resource extern/]/
    │   ├── [valid-from]/
    │   └── [valid-until]/
    └── execution/        ← accumulated facts
        ├── completed/    ← if completed
        ├── cancelled/    ← if cancelled
        └── [domain-specific-state]/

subject.es/trusteando/
└── references/
    └── [ref extern/agent.es/actions/id]/

resource.es/trusteando/
└── history/
    └── [ref extern/agent.es/actions/id]/
```

The same pattern applies across domains:

| Domain | Authority | Agent | Subject | Resource |
|---|---|---|---|---|
| Healthcare | Ministry of Health | Doctor / Pharmacy | Patient | Medication |
| Education | Ministry | University | Student | Degree |
| Banking | Central Bank | Bank | Client | Account |
| Justice | Court | Lawyer | Citizen | Document |
| Logistics | Customs | Carrier | Client | Shipment |

### Canonical example — medical prescription

This example shows the pattern applied to prescription and dispensation, and illustrates how `plan/execution` correctly represents the temporal states of a multi-party process.

**Authorities publish registries:**

```
colegiomedicos.es/trusteando/professionals/
└── [nif:12345678A]/
    ├── [specialty cardiology]/
    └── since/2020-01-01/

sanidad.es/trusteando/medications/
└── [code:M-123456]/
    ├── [name "Paracetamol"]/
    ├── [prescription-required]/
    └── [max-dose 4000mg]/
```

**Doctor issues prescription using `plan/execution`:**

```
doctor.es/trusteando/prescriptions/
└── [id:REC-2026-001]/
    ├── plan/
    │   ├── [patient extern/patient.es/identity/]/
    │   ├── [medication extern/sanidad.es/medications/M-123456]/
    │   ├── [dose 500mg]/
    │   ├── [duration 7 days]/
    │   ├── [valid-from 2026-03-24T10:00:00Z]/
    │   └── [valid-until 2026-04-24T10:00:00Z]/
    └── execution/
        └── dispensed/         ← only published when dispensation occurs
            ├── by/extern/pharmacy.es/dispensations/DIS-2026-001/
            └── since/2026-03-24T15:30:00Z/
```

**Why `plan/execution` and not a flag:**

A flag `[dispensed false]` would require mutation — changing the value when dispensation occurs. That breaks immutability. The `plan/execution` model is append-only: the prescription is created under `plan/`, and `execution/dispensed/` is published only when the event actually happens. The absence of `execution/dispensed/` does not mean "not dispensed" — it means "not yet dispensed". This distinction matters when combined with `valid-from` and `valid-until`.

A verifier evaluates the complete state by reading both folders:

| plan/ | execution/dispensed/ | valid-from | valid-until | State |
|---|---|---|---|---|
| exists | absent | future | — | Not yet valid |
| exists | absent | past | future | Valid, pending |
| exists | exists | past | future | Dispensed |
| exists | absent | past | past | Expired undispensed |
| revoked/ present | — | — | — | Revoked |

**Pharmacy publishes dispensation independently:**

```
pharmacy.es/trusteando/dispensations/
└── [id:DIS-2026-001]/
    ├── [prescription extern/doctor.es/prescriptions/REC-2026-001]/
    ├── [patient extern/patient.es/identity/]/
    ├── [medication extern/sanidad.es/medications/M-123456]/
    └── since/2026-03-24T15:30:00Z/
```

Each party publishes in their own space. The doctor publishes the prescription and later records the dispensation event. The pharmacy publishes the dispensation independently. Any verifier can cross-reference both and confirm consistency without either party cooperating with the verifier directly.

### What this pattern resolves

| Problem | Resolution |
|---|---|
| Forged prescription | Only the doctor can publish under their URL |
| Multiple dispensation | `execution/dispensed/` is present — verifier rejects second dispensation |
| Expired prescription | `valid-until` is past — verifier rejects |
| Unauthorised medication | Reference to `sanidad.es/medications/` — authority controls what is valid |
| Unidentified patient | Reference to verified identity node |
| Full traceability | Every action signed, timestamped, and permanent |

---

## 7.14 Administrative Acts and Chain of Mandate

Public administration is a cascade of delegated authority. The challenge is not only proving who signed an act, but proving that the signer had unrevoked authority at the moment of signing — and that the authority chain from the trusted root to the agent was intact throughout.

### The authority cascade

```
estado.gob/min-urbanismo/           ← root of trust
└── ayuntamiento/malaga/            ← delegated node
    └── employees/
        └── tecnico-01/             ← agent node
            └── acts/
                └── license-99/     ← signed act
                    └── since/2026-03-15/
```

Each level derives its key from its parent via `grant_key`. The agent's authority is not a stored permission — it is a cryptographic consequence of its position in the tree. Any verifier can trace the chain upward from the act to the trusted root without consulting any central registry.

### Validity at the time of the act

When an employee leaves, the organisation publishes `until/` in their employee entry:

```
ayuntamiento/malaga/employees/tecnico-01/
├── since/2024-01-10/
└── until/2026-04-01/      ← authority ends here
```

The validity rule for any act signed by that employee is:

> **An act is valid if `date_of_act < date_in_until`.**

A licence signed on 15 March remains valid after 1 April — the authority existed at the time of signing. A licence signed on 2 April is invalid — the authority had already been revoked. The protocol does not need to invalidate past acts retroactively; it only needs to record when the authority ended.

### What the protocol guarantees — and what it does not

The protocol proves two things: that the signature is cryptographically authentic (it came from the agent's key), and that the agent's authority chain was unrevoked at the date of the act. It does not prove that the content of the act is legally correct, or that the agent acted in good faith.

This distinction matters for high-stakes administrative acts. A building permit signed by a corrupt official with valid authority is cryptographically valid in Trusteando — the protocol records that the official signed it, with full authority, at that moment. Whether the permit should have been issued is a legal question, not a cryptographic one. The protocol provides the evidence; the legal system provides the judgment.

### The administration as its own official gazette

The folder structure of a public administration is, in effect, a real-time official gazette. A citizen who receives a fine or a licence can trace the signature upward through the authority chain — from the signing agent to the municipal department to the ministry — and verify at each level that the authority was unrevoked at the time of the act. No separate lookup in an official bulletin is required. The graph is the bulletin.

---

# 8. Sustainability Model

The protocol is open. The reference implementation is free software. What can be monetized is the service layer over the protocol—not the protocol itself.


## 8.1 Formal recognition as an agent

Registering with the Trusteando root has signaling value, especially in the early phases when the root is the principal established trust root. Formal recognition has a reasonable cost and a public, transparent criteria.


## 8.2 Managed infrastructure

Implementing a full Trusteando node requires technical knowledge that not all entities possess. Trusteando offers managed infrastructure—the node as a service—for entities that want to participate without managing the infrastructure themselves. It is the WordPress model: the protocol is free, but managed hosting does not have to be.


## 8.3 Verification API

Third-party applications that want to integrate Trusteando verification without implementing the full protocol can use a verification API. The price is based on verification volume. The protocol remains free—the API is a convenience, not a requirement.


## 8.4 What remains free

- The complete protocol format
- The ability to run your own nodes
- Manual verification by consulting URLs directly
- Publishing credentials without passing through the root
- Implementing your own verifiers

---

# 9. The Bootstrap Problem

The root's value depends on the reputation of whoever operates it. Trust systems do not arise out of nothing—they need a starting point with enough credibility for the first entities to want to register.

This is the same problem that DNS, Let's Encrypt, and any PKI solved: at the beginning the authority does not come from the system—it comes from the people and organizations behind it. The solution is social, not technical: the first recognized entities are secured through personal relationships, concrete use cases that solve real problems, and the credibility of the founding team in that context.

The correct launch strategy is to publish the protocol openly with established temporal priority before any competitor can claim the same idea. Openness does not weaken the root's position—it strengthens it. A root operating a closed protocol can be replaced by anyone who publishes a better protocol. A root that operates the de facto standard protocol accumulates legitimacy that cannot be copied.

However, a single entity operating the initial root introduces a risk that the technical design alone cannot eliminate: coercion. Although the root has no discretion—it only executes a public algorithm—the entity operating the server can be subject to legal, political, or regulatory pressure. A government can order a foundation to exclude certain nodes. A company can be acquired. An individual can be intimidated. The technical solution to this risk is to apply from day one the same quorum mechanism defined in section 4.10.

The system does not start with a single root but with a founding set: N independent entities—universities, NGOs, technical organizations, public administrations—that sign a genesis agreement and operate their own root node from day one. Any sensitive operation requires the quorum of K of those N founding nodes. No founding node has unilateral authority. The system is born polycentric; it does not become so after a maturation period under a transient central authority.

An additional advantage is that the founding nodes bring diverse institutional reputation from the start. A verifier who trusts any one of them has immediate access to the entire system. Resistance to coercion is proportional to the legal and geographic diversity of the founding set: to silence the system an attacker needs to compromise simultaneously the full quorum of entities across multiple jurisdictions.


## 9.1 Resistance to Embrace, Extend, Extinguish

The strategy known as Embrace, Extend, Extinguish consists of adopting an open standard, adding proprietary extensions that create dependency, and ultimately making the original standard irrelevant. It is the pattern that has eliminated or weakened multiple open protocols throughout Internet history.

Trusteando incorporates resistance to this pattern by design:

- Embrace — anyone can adopt the protocol. Inevitable and desirable.
- Extend — any proprietary extension must be declared under trusteando/extensions/ with justification. Non-standard extensions are not interpreted by other parsers. Fragmentation is visible and auditable.
- Extinguish — the data lives under each entity's URL, not in any central service. There is nothing to shut down. GPL v3 prevents distributing closed versions. The root stores no state—it cannot be monopolized.

The most effective defense against this pattern is to cover the functional space so completely that any proprietary extension is marginal or redundant. The extensive vocabulary of standard folders—features/, payments/, messaging/, hooks/, media/—is not unnecessary complexity: it is territory that cannot be used as a wedge.


## 9.2 Impact on Intermediary Services

A side effect of the protocol is that the structured information of any business—schedules, location, features, verified reputation—becomes public and directly accessible from its URL. This allows building search, comparison, and aggregation services over open data without relying on intermediaries that currently centralize and monetize that information.

The protocol does not eliminate those services—it simply shifts their value to the presentation and user experience layer, not to exclusive access to the data. The protocol's hooks and proxies also ensure that the business is never tied to a service provider for technical reasons—changing providers is always a business decision, never a technical migration.


---

# 10. Two layers — registration and process

The protocol explicitly distinguishes two layers that are conflated in current systems:


## 10.1 The registration layer — the Knowledge Graph

The registration layer is the permanent graph of verified facts. What each node is and what relationships it has. Each entry is signed by the entity that publishes it, immutable once established, and auditable by any third party without anyone's cooperation.


```
universidad.es/profesores/juan-ruiz          ← verified fact, permanent
estados/spain/ciudadanos/juan-ruiz          ← verified fact, permanent
colegios/medicos/colegiados/juan-ruiz        ← verified fact, permanent

```

Redundancy in this layer is a positive property—multiple independent nodes certifying the same thing about Juan adds resilience. If two high-grade nodes say contradictory things, that inconsistency is itself a meaningful signal.


## 10.2 The process layer — the flows between nodes

The process layer consists of the information exchanges between nodes that lead to establishing facts in the registration layer. They are ephemeral, operational, and not part of the permanent graph. They represent the process, not the result.


```
University asks the state: does juan-ruiz have a valid ID?     ← flow, ephemeral
State responds: yes, issued 2015                               ← flow, ephemeral
University adds juan-ruiz to /professors/since/2026-03-17/    ← enters the graph

```

The protocol can define the standard format of these flows to facilitate interoperability between implementations. But the flows are not the graph—they are the operational edges that allow building it. An external verifier never needs to see the flows, only the result in the graph.


## 10.3 Civil identity as a minimum condition

A node can technically exist without verifiable civil identity. But for a node to represent a real natural person, the protocol requires at least one credential issued by an entity with recognized civil authority—a state, an official registry—that links the node to a legal identity.

The natural structure for natural persons is:


```
estados/spain/ciudadanos/juan-ruiz     ← civil identity anchor
→ universidad certifies: juan-ruiz is Professor
→ colegio certifies: juan-ruiz is registered
→ empresa certifies: juan-ruiz is an employee

```

The civil identity node is the anchor. Credentials are the graph edges pointing to that node from different entities. A person can have multiple credentials from multiple entities—all pointing to the same civil identity node.


---

# 11. Declarative specification and functional implementation

The protocol distinguishes two levels that are independent and complementary:


## 11.1 The declarative specification — the folders

Folders define what exists and what it means. It is the public contract of the protocol—the WHAT. Anyone can read it, verify it, and audit it without special tools or knowledge of the internal implementation. The folder structure is permanent, signed, and auditable.


## 11.2 The functional implementation — the database

The database, the API, the adapter define how it is stored and retrieved internally—the HOW. It is an implementation detail invisible to the external verifier. Any functional implementation is valid if it produces results indistinguishable from reading the folders directly.


```
# Declarative access — static web
GET turismo.gob.es/establecimientos/casapepe.es/classification/stars
→ 3

# Functional access — database
query({ path: 'establecimientos/casapepe.es/classification/stars' })
→ 3

# The result is identical — the parser neither knows nor cares how it is stored

```

It is the same principle that separates an interface from its implementation in object-oriented programming. The interface is declarative—it defines what exists and what it returns. The implementation is functional—it defines how it does so internally.


## 11.3 The adapter system

An agency such as a ministry has its data in an existing database. It does not need to rewrite it to participate in the protocol—it only needs to publish an adapter function that responds to protocol routes with the same values that a folder would have.

The agency publishes on its node:


```
turismo.gob.es/trusteando/adapters/clasificacion_hotelera/
name           ← 'clasificacion_hotelera'
description    ← 'Verifies official establishment classification'
input_schema   ← which protocol fields it needs as input
output_schema  ← which fields it returns to the protocol
version
cache/ttl      ← how often to refresh

```

The adapter function has a standard signature:


```
async function query(path) {
// path: protocol route
// 'establecimientos/casapepe.es/classification/stars'

// internally query its database
// SELECT stars FROM establecimientos WHERE url = 'casapepe.es'

// return as if it were a folder
return { value: 3, timestamp: '2024-01-15' }
}

```


## 11.4 The central adapter repository

If the agency does not publish its adapter, the ConfidenceNode central repository maintains community adapters for known public sources. When the agency publishes its own, it automatically takes precedence—the parser always prefers the adapter of the node itself.


```
confidencenode.org/trusteando/adapters/
turismo.gob.es/clasificacion_hotelera/   ← maintained by the community
guia.michelin.com/estrellas/              ← maintained by the community

```


## 11.5 Gradual Migration

Un organismo puede moverse a su ritmo sin interrumpir el servicio:


```
Fase 1 — sin adaptador
Parser reads static folders if they exist, or uses community adapter

Fase 2 — adaptador sobre base de datos existente
Organisation publishes query() over its current DB — without changing anything else

Fase 3 — web nativa del protocolo
Organismo publica carpetas reales — el adaptador ya no es necesario
La ruta funciona igual para todos los parsers existentes

```

The declarative specification is the destination. The functional implementation is the path. The protocol works in all three states without changing anything in the parsers that already use it.


---

# 12. Open Questions

The following issues are identified but deliberately out of scope for this version of the protocol. They are documented here to acknowledge their existence and to guide future work.


## 12.1 Anchor entity failure

If an entity's URL disappears abruptly—sudden bankruptcy, domain loss—the credentials it issued lose their anchor. For orderly disappearances, the protocol facilitates transferring records to a successor. For abrupt disappearances, the solution lies outside the protocol—institutional or legal. The protocol honestly inherits that limitation.


## 12.2 Scope of issued credentials

The protocol does not impose restrictions on what a recognized agent can certify. Mechanisms to declare and verify an agent's scope of competence will be designed in v0.3.


## 12.3 Advanced privacy via zero-knowledge proofs

A zero-knowledge proof–based implementation would allow demonstrating the validity of a credential without revealing any element of the chain. This is a research direction for future versions.


## 12.4 Standard format for the transaction layer

The protocol defines the registration layer but not the format of the inter-node flows in the transaction layer. An interoperability standard for those flows will be addressed in v0.2.


## 12.5 Compatibility Certification System

A testing tool that evaluates whether a node uses standard fields for standard functions or has invented equivalents without justification. Nodes can declare justified extensions under trusteando/extensions/ with a rationale and a certifying body. A system of evaluation bodies and dispute resolution over extensions is planned for v0.4.


## 12.6 Adapter Sandboxing

Adapters execute third-party code. The protocol must specify a restricted execution environment—no network access except to the agency’s URL, no side effects, with a maximum timeout. Web Workers and Content Security Policy are the technical basis. The formal specification will be addressed in v0.2.


## 12.7 Graph Relationship Privacy

Selective disclosure defined in section 6 protects the content of credentials, but the existence of the edge in the graph is public by default: that universidad.es/profesores/juan-ruiz is a public URL reveals that Juan has a relationship with that university. This problem is solved via the private/ folder defined in section 2.11. Publishing an entry under universidad.es/personal/profesores/private/ hides the node’s identity—a verifier knows something exists there without knowing what. To access it, they must raise a query to the agent that controls the folder. The existence of the edge can be hidden, not just its content. Zero-knowledge proofs (section 12.3) will complement this mechanism in future versions for cases where even the existence of private/ should not be visible.

This is not a flaw in the protocol — it is a characteristic of a public-by-default model. The privacy analysis has three distinct layers:

**Layer 1 — The tool exists.** The protocol provides `private/` as a semantic firewall. Placing sensitive nodes under `private/` rather than under descriptive public paths hides both the content and the identity of what lies beneath. The style guide (section 5.1b) makes the correct pattern explicit: `private/` as high as possible in the hierarchy, not at the leaf level.

**Layer 2 — The responsibility is the implementor’s.** The protocol cannot enforce good modelling decisions. An implementor who publishes `universidad.es/profesores/juan-ruiz/` when they should have published `universidad.es/private/profesores/juan-ruiz/` has leaked a relationship by design choice, not by protocol failure. This is why the note on schema design in section 1.2 exists — the grammar is simple, but designing structures that faithfully protect privacy requires domain modelling skill.

**Layer 3 — The ceiling requires ZKP.** Even with correct use of `private/`, the existence of a `private/` folder at a given path signals that something is being hidden there. For cases where even that signal is too much — where the existence of the relationship itself must be invisible to anyone without prior access — zero-knowledge proofs are the necessary extension. This is a research direction for future versions (section 12.3). The current protocol handles the common cases; ZKP handles the edge cases where even metadata must be invisible.


## 12.8 Legal Framework for Dispute Resolution

Appendix B defines a dispute mechanism with cryptographic evidence and increasing costs, but it does not specify how the resolutions of the root or arbitrators are linked to the real judicial system. The hash_disputa is proof of identity before the arbitrator, but the arbitrator needs a procedural framework to take binding decisions about the meaning of a credential or the legitimacy of a relationship. How Trusteando integrates with concrete legal frameworks—beyond using eIDAS for signature—is an open question that will require collaboration with legal experts in each jurisdiction.


## 12.9 Semantic Vocabulary Fragmentation

The route grammar defines the syntax but not the vocabulary. Opening the vocabulary of operators and properties—any natural-language term is valid—is an expressive strength but can generate semantic fragmentation: different communities or domains might use different terms for the same concept, reducing the interoperability the protocol promises. A node that uses [en] and another that uses [located-in] for the same geographic relation are not automatically interoperable. The long-term solution lies in domain ontologies or term registries consensused by the community, something the protocol facilitates but does not specify. The reserved vocabulary in Appendix A is the common core; its coordinated extension is a governance matter more than a technical specification. A concrete direction is to publish domain ontologies in distributed repositories hosted on multiple roots, where communities of practice agree on the standard terms for their sector. Analytical tools can warn a node operator when they use a term that diverges from the majority vocabulary in their domain, encouraging convergence without imposition. The formal specification of this mechanism will be addressed in future versions.


## 12.10 Name Discovery and Registry

The protocol identifies entities by URL hash or autonomous identifier, not by human-readable name. The association between a name — "Universidad de Málaga" — and its cryptographic identifier raises a problem with three layers of complexity.

**Layer 1 — The entity publishes its own name (partial solution)**

The node can publish its readable name in its `identity/` folder:

```
uma.es/trusteando/identity/
├── [official-name "Universidad de Málaga"]/
├── [acronym "UMA"]/
└── [also-known-as "University of Malaga"]/
```

For any verifier who already knows the URL `uma.es`, the association is direct and verifiable — the node declares it, signed with its own key. This resolves the problem for anyone who already has the URL.

**Layer 2 — Reverse discovery (name → identifier)**

The real problem is discovery: a user who only knows the name "Universidad de Málaga" needs a mechanism to find its identifier. The protocol does not formally specify this mechanism. Three non-exclusive options exist:

*Option A — Centralised registry at the root node.* The root maintains a registry of recognised names: `T10/names/universidad-de-malaga/ → uma.es`. Each entry is signed by the publishing node.

*Option B — Sector blueprints with standard vocabulary.* For specific domains — education, healthcare, public administration — canonical paths are defined: `T10/blueprints/education/spain/universities/uma.es/`. Nodes publish their presence in these blueprints; a search engine can index them without depending on a single central registry.

*Option C — External search engines (outside the protocol).* The simplest solution compatible with the protocol's philosophy: delegate discovery to external services that index the graph, the same way Google indexes the web. The protocol defines how data is published so it can be indexed; it does not need to specify how indices are built.

**Layer 3 — Name conflict resolution**

Two nodes could claim the same readable name. Without an authoritative registry, the protocol alone cannot resolve which node has the right to the name. The solution relies on the quorum system (section 4.10): a name is considered established when a quorum of reputable nodes publish the same association. Disputes are resolved through the dispute resolution mechanism (Appendix B), where the weight of evidence — which nodes support each version — determines the outcome.

**Direction for future versions**

The formal name system specification will have three components: local declaration by the node, third-party attestation by reputable nodes, and quorum-based resolution. The resulting system is decentralised, dispute-resistant, and compatible with the rest of the protocol without adding new infrastructure. The concrete implementation will be addressed in v0.3.

## 12.11 Active Authentication with Key Rotation

The protocol defines the emergency key for migrations but does not specify an interactive active authentication mechanism. A natural extension is atomic rotation: the wallet generates a new key and publishes it on the web before revealing the old one to the verifier. The flow is: (1) wallet generates clave_2 and publishes hash_publico_2, (2) wallet waits for confirmation that hash_publico_2 is active, (3) wallet reveals clave_1 to the verifier, (4) verifier checks hash(entity_id, clave_1) == hash_publico_1. Each authentication consumes the revealed key, turning the system into a one-time password mechanism cryptographically anchored in the node's identity. The formal specification of this mechanism will be addressed in future versions.


## 12.12 Metadata Leakage in Path Structure

The position of `private/` in the hierarchy determines what metadata leaks to external observers. Consider these two paths:

```
# Leaks metadata — the existence of a mergers process is visible
empresa.com/trusteando/mergers-acquisitions/private/

# No metadata leak — only the existence of private/ is visible
empresa.com/trusteando/private/mergers-acquisitions/
```

In the first pattern, an external observer can see that a mergers-and-acquisitions process exists, even though its content is protected. In the second, they see only that a `private/` folder exists — nothing about what lies beneath.

The design principle is: **place `private/` as high as possible in the hierarchy**. It acts as a semantic firewall — hiding not just the content but the existence of what lies beneath. If the existence of a folder is itself sensitive information, it belongs inside `private/`, not the other way around.

```
# ✅ Good — private/ as firewall
trusteando/private/mergers-acquisitions/
trusteando/private/litigation/
trusteando/private/hr/dismissals/

# ❌ Bad — leaks structure
trusteando/mergers-acquisitions/private/
trusteando/hr/dismissals/private/
```

This converts what might appear to be a weakness of the protocol into a non-weakness, provided implementations follow this convention. The Style Guide makes this explicit.


## 12.13 Latency in Critical Security Revocation

The observer replica model with a hysteresis threshold (section 4.3) introduces a window of inconsistency between the moment a credential is revoked and the moment all replicas reflect that revocation. For low-criticality use cases this latency is acceptable. For real-time access control—physical access to facilities, authorizations for critical systems—it can be dangerous. A solution direction is a high-priority revocation channel: replicas synchronize revocations published under registry/compromised/ with maximum priority over any other update, reducing the inconsistency window from minutes to seconds. The formal specification of this mechanism will be addressed in future versions.


## 12.14 Social Identity Recovery

The secret custody mechanism via collaborating wallets (section F.1) addresses key loss for the protocol author but does not specify a generic mechanism for any entity. Without social recovery, a node's emergency key loss is irreversible: its history remains orphaned permanently. The natural direction is Shamir's Secret Sharing: fragment the key among N trusted guardians so that any subset of M of them can reconstruct it. The guardians are entities from the graph itself—university, employer, official registry—named by the node using the [agent role] convention. The formal specification of this mechanism will be addressed in future versions.


## 12.15 Verification Load in Deep Chains

Verifying a fact at the end of a long chain—root, university, faculty, department, professor—requires multiple HTTP requests and processing several HMACs and ECDSA signatures in sequence. This penalizes resource-constrained devices or slow connections. The solution direction is proof aggregation via zero-knowledge proofs (section 12.3): a single ZK proof demonstrating the validity of the entire chain without revealing intermediate steps reduces the verification load to a single operation, regardless of chain depth. A more immediate solution without ZK is for each node to publish a precomputed Merkle proof of its position in the chain, allowing local verification without additional queries.



---

# 13. Design Trade-offs and Intentional Constraints

Trusteando makes choices. Every architectural decision that gives the protocol one property takes away another. This section names those choices explicitly — not as apologies, but as principles. A protocol that does not acknowledge its trade-offs has not thought them through.

## 13.1 Facts vs Transactions — the "move semantics" argument

Trusteando is an event store at the scale of a knowledge graph, not a general-purpose state machine. It models facts — immutable, signed statements about what exists or has existed — not transactions that transfer ownership or mutate state.

In traditional systems and in blockchain, an asset moves from A to B: if B has it, A no longer does. In Trusteando, there is no movement. There is only a new signed fact: A declares that something involving B occurred. The "transfer" is a fact that references prior facts. The balance of an account is not a mutable field — it is the projection of all facts published about that account by the authority that controls it.

This means Trusteando cannot directly model: atomic swaps, double-spend prevention, or any operation that requires a globally consistent state change. These require distributed consensus — which requires coordination overhead that the protocol deliberately avoids.

What Trusteando can model: that a transfer was declared, by whom, when, and with what authority. The downstream consequences of that declaration — whether money actually moved, whether a system actually updated — are the responsibility of the systems that implement the action, not of the protocol that records it.

This is not a limitation to be fixed. It is the property that makes the graph auditable without a central coordinator.

## 13.2 Web Identity vs Abstract Identity

The protocol anchors identity in URLs. This is a deliberate choice — URLs are human-readable, already exist for most organisations, and tie identity to a verifiable public presence. Anyone can confirm that `universidad.es` controls `universidad.es/trusteando/`.

The cost is DNS dependency. If an entity loses its domain abruptly, its credentials lose their anchor. The protocol mitigates this with the emergency key mechanism (section 4.7) and autonomous identity mode (section 2.1), which allow identity to survive domain loss. But the default mode inherits the web's infrastructure dependency.

The alternative — fully abstract identity with no URL anchor — maximises independence but requires the holder to manage cryptographic keys with no fallback, no human-readable anchor, and no institutional backing. Trusteando chooses the web as the foundation because the web is already the infrastructure of institutional identity. Entities that need fully abstract identity can use autonomous mode; they trade the convenience of URL anchoring for full key sovereignty.

## 13.3 Transparency by Default vs Privacy by Design

The graph is public by default. Every fact published in it is visible to any verifier. Privacy is not the default state — it is an explicit choice made by placing content under `private/`.

This inversion is deliberate. A system where privacy is the default and publication is the exception produces graphs that are not auditable, not verifiable, and not useful as a knowledge substrate. The transparency-first model makes the graph useful precisely because anyone can verify anything that has been published.

The cost is that implementors must actively choose privacy. An entity that publishes `universidad.es/profesores/juan-ruiz/` has made that relationship public — not by accident, but by following the default. Good schema design requires knowing when to use `private/` and at what level of the hierarchy. This is a skill, not a configuration option (section 1.2 and 12.7).

The ceiling of privacy — hiding even the existence of a `private/` folder — requires zero-knowledge proofs, which are a research direction for future versions. The current protocol handles the common cases; ZKP handles the edge cases.

## 13.4 Radical Immutability vs Error Correction

Published facts cannot be modified. There is no delete, no update, no correction in place. This is not a technical limitation — it is a cryptographic consequence that the protocol treats as a guarantee.

When a fact contains an error, the correction is a new fact that supersedes the original. The original remains in the graph permanently. Any verifier can see what was believed at any point in the past, what was corrected, when, and by whom. This is forensic auditability — the graph is a complete and tamper-evident record of everything that was ever published.

The cost is that mistakes are permanent. An entity that publishes incorrect data cannot erase it — only supersede it. For organisations accustomed to being able to silently correct databases, this is a cultural shift. The protocol imposes transparency on error correction the same way it imposes transparency on everything else.

This is the correct trade-off for a system whose primary value is trustworthiness. A system that can silently modify its past cannot be trusted. A system whose past is permanently visible and cryptographically sealed can be.

### Trusteando as a Git of semantic assertions

The most precise analogy for this model is Git. In Git, a commit is a snapshot — you never overwrite history, you add new commits that supersede prior ones. `git revert` does not erase the reverted commit; it adds a new one that undoes its effects. The log is permanent.

Trusteando is Git applied to knowledge claims. Publishing a folder under a path with a `since/` is the equivalent of a commit: "from this moment, this fact is true." Publishing `until/` or `revoked/` is the equivalent of `git revert` — the original fact remains in the graph permanently, and the new fact supersedes it going forward.

| Git | Trusteando |
|---|---|
| Commit | Folder published with `since/` |
| Checkout / HEAD | Current state reconstructed from `since/` and `until/` |
| `git log` traversal | Verifier reconstructing state from `since/` and `until/` |
| `git log` | The complete history of facts under a path |
| `git revert` | Publishing `until/` or `revoked/` |
| `git push --force` | Does not exist — the past cannot be rewritten |

There is no `git push --force` in Trusteando. The derived keys and folder hashes of parent nodes have already sealed the history. You can supersede a fact but you cannot erase it — the record that it existed remains permanently, guaranteeing forensic auditability.

The current state of any node is the reconstruction of its signed facts over time — the HEAD of a branch of knowledge. Trusteando does not manage states; it manages the history of changes to what is known to be true. It is a version control system for human knowledge.

## 13.5 Simplicity of Core vs Richness of Ecosystem

The protocol core is four functions and twenty lines of code. The whitepaper is nearly four thousand lines. That distance is not a contradiction — it is the distinction between what the protocol requires and what the ecosystem recommends.

The core cannot grow. Every primitive added to the core adds complexity that every implementor must handle. The ecosystem — conventions, schemas, patterns, sector vocabularies — can grow indefinitely without burdening the core. An implementor who only needs Level 1 or Level 2 (section 1.2) never needs to read past section 4.

The risk is that ecosystem complexity obscures core simplicity. This section exists partly to name that risk explicitly: the protocol is simple. The world it models is not.

## 13.6 Revocation vs Cached Verification

The protocol supports signed caches to reduce verification latency (sections 4.3, A.11). An issuer publishes a signed list of valid credentials with a TTL. A verifier using that list can answer queries locally without contacting the issuer for each verification.

This creates a trade-off. A revoked credential may appear valid to a verifier using a cached copy that predates the revocation, until the cache expires or is refreshed. The window is bounded by the TTL. For low-stakes verifications — a student discount, a library card — this is acceptable. For high-stakes verifications — employment, professional licensing, financial transactions — the verifier should perform a live check against the issuer's current state.

The protocol does not mandate one mode. It provides both, and the context determines the correct choice. Implementors must choose TTLs appropriate to their domain. The security property is not that revocation is instantaneous — it is that revocation is eventually consistent and that the window is known and bounded.

This trade-off is particularly visible in academic credentials. A university that revokes a degree for academic misconduct may have already issued signed caches that list the degree as valid. Those caches remain valid until their TTL expires. For employment verification, a live check is appropriate. For a library access card, a cached check is sufficient. The same credential, the same revocation — different verification modes for different contexts. See also section 12.13 (Latency in Critical Security Revocation).

---

# 14. Roadmap

Future versions of the protocol are guided by the problems that implementation practice reveals as most urgent. The following roadmap reflects the current state of known priorities—it is not a commitment of dates but a declaration of intent.

v0.1 — current

Protocol fundamentals: URL as identity anchor, separation of authenticity/authorship, emergency key and portability, reserved vocabulary, business layer, two levels of existence, resistance to Embrace–Extend–Extinguish.

v0.2

Semantic extensions: route grammar with relational operators in brackets, plan/execution convention for active processes. Expanded use cases: multi-agency emergency coordination, administrative interoperability. HMAC scalability: batch verification via Merkle trees. Formal specification of adapter sandboxing and an interoperability standard for the transaction layer. Reference wallet: a basic user agent that manages keys securely, displays credentials graphically, and allows sharing and verifying credentials without technical knowledge. The wallet is not part of the protocol—it is the application layer that makes it accessible to the end user, analogous to a web browser for HTTP.

v0.3

Credential scope mechanisms: declaration and verification of an agent's competence domain. Advanced privacy via zero-knowledge proofs for verification without online interaction. Reference implementations in at least two languages.

v0.4

System of evaluation bodies and dispute resolution over extensions. Formal mechanism for root rotation or replacement by consensus of high-grade nodes. Roadmap toward quantum-resistant cryptography.

v1.0

Stable protocol. Ecosystem of tools: public validator, assisted generator, interactive tutorials. Network with enough high-grade nodes to operate polycentrically without dependence on the root in everyday use.



---

# FAQ


## Why URLs and not DIDs or W3C Verifiable Credentials?

The W3C DIDs (Decentralised Identifiers) require additional infrastructure—blockchain registries, specific DID methods—and are not anchored in any preexisting real presence. W3C Verifiable Credentials define the document format but not the trust system nor how the issuer's authority is established. Trusteando uses URLs because the entities already exist with their own URLs—you do not need to create new infrastructure to identify them. The URL is the identity, not a pointer to it.


## What happens if an entity loses its domain?

The protocol has two anchoring levels. At the basic level, identity is anchored in the URL, and its abrupt loss is a real risk, mitigable with mirrors and alternate domains. At the advanced level (section 4.10), the entity can fully decouple its identity from the URL via autonomously generated secrets. In that case the URL is only the publication place: the identity survives domain changes, server outages, and DNS blocking. The two modes are compatible and linkable via old_identities/.


## Can anyone create a node?

Yes. Any entity with a URL can publish its identity space following the protocol without anyone's permission. This creates a level-1 node—declared identity, verifiable by context. To reach level 2—recognized identity with cryptographic verifiability by unknown third parties—the recognition of the root or another sufficiently trusted node is required.


## What is the difference between this and PGP Web of Trust?

PGP Web of Trust is a network of signatures among people where trust propagates through acquaintances. It lacks hierarchical structure, is not anchored in public URLs, lacks a verifiable temporal dimension, and does not distinguish between types of relationships. Trusteando is a structured Knowledge Graph where entities are nodes with their own URLs, relationships have explicit semantics expressed in the folder structure, and the full history is auditable. PGP proves that someone knows someone else. Trusteando proves what each entity is and since when.


## Is the Trusteando root a centralised point of control?

The root is the bootstrap node with the highest initial reputation. It stores no state—it only executes algorithmic verifications over information that the entities themselves publish. If the root goes down, any high-grade node can perform the same verifications because the algorithm is public. Over time, entities with sufficient reputation become independent trust roots. The root is necessary to start the system—not for it to operate once established.


## How does Trusteando relate to eIDAS?

eIDAS is the European regulation for digital identity—it defines the legal requirements for an electronic signature to have legal validity. Trusteando establishes the cryptographic evidence; eIDAS gives it legal force in European jurisdictions. They are complementary. A Trusteando node backed by an entity recognized under eIDAS automatically inherits its legal validity.


## Does verification require consulting the parent every time? Does it scale?

The base design requires consulting the superior to verify the HMAC, but there are three scalability mechanisms. Signed cache: the superior publishes signed listings of its members with TTL, allowing offline verification during that period. Merkle trees: the superior publishes a Merkle root of all valid HMACs; the member presents a proof with its credential, enabling verification without online queries. Multiple roots and quorum: for high-value credentials, validation can require a quorum of independent roots (section 4.10.4), distributing the load. All three mechanisms are described in section 4.3. The underlying verification operation in all of them is the verify_child_authorship function of the TrusteandoNode class (section 4.11).


## A small business cannot manage keys and folders. How is adoption handled?

There are three adoption paths. Managed hosting: just like WordPress, the protocol is free but anyone can hire someone to manage their node (section 8.2). LLM-assisted migration: the user describes where their data resides and a language model automatically generates the complete structure; the human only reviews and validates it. Adapters: agencies with existing databases can publish an adapter that translates protocol queries to their internal system without migrating data (section 11). In all three cases, the technical complexity stays out of the end user’s reach.


## Isn't everything public? What about privacy?

The protocol offers three levels of selective privacy (section 6). Message hash: only the hash is published, not the content; you can verify integrity without revealing the message. Selective disclosure: the holder can choose to reveal only the result, only the issuer, or the full chain depending on the context. private/ folders: information exists but is not exposed by default, accessible only via grantReveal(). Privacy is not an add-on layer—it is designed into the structure from the start.


## What if a root is compromised? Can it forge identities?

A compromised root can falsify the keys it generates for new entities but cannot falsify the past: already issued credentials are signed by the entities, not the root; already established relationships are published at the nodes, not at the root; and the quorum of multiple roots makes compromising just one insufficient for consensus-required operations. Moreover, a compromised root enters state brokenado—a term from the protocol’s reserved vocabulary—visible to all participants, and its signatures no longer count toward the quorum.


## The protocol seems very ambitious. Can it work in practice?

Trusteando does not aim to replace all existing identity systems overnight. Its strategy is organic and layered. Level 1 declared: any entity can publish its space without permission, useful as proof of concept or for closed communities. Level 2 recognised: entities that need verifiability by unknown third parties request recognition from roots. Sectoral adoption: the protocol can start in concrete niches — hospitality, emergencies, universities — and grow from there. The protocol does not need to conquer the world to be useful. It is enough to solve real problems in concrete domains, and that is already possible today with the current specification.

That said, the honest answer is that a protocol's success is determined by four things — and only the first three are within the protocol's control.

**Easy.** The barrier to entry must be low enough that a single developer can implement Level 1 in an afternoon and Level 2 in a day. This document tries to make that true.

**Useful.** It must solve a real problem that people already have and are already paying to solve. Verified reviews, academic credentials, administrative interoperability — these are not invented problems. The value proposition is real.

**Adaptable.** It must fit into existing infrastructure without requiring organisations to rewrite everything. The adapter model, the coexistence with existing web infrastructure, the compatibility with FNMT certificates — these are deliberate choices to lower the adoption cost.

**Lucky.** A protocol also needs the right moment, the right early adopters, and the compounding effect of a community that grows before it dies. This cannot be designed. What can be done is to make the first three conditions as good as possible, and then let the network do its work.

The technology is the least uncertain part. What will decide the protocol's fate is whether a few concrete use cases gain enough traction to make it self-sustaining — and whether the community that forms around those early uses is large enough to carry it forward.



## What if two sources say contradictory things about the same node?

This is not an edge case — it is the normal state of the world. A university may say a professor is active. A professional body may say the same person is suspended. A court may have issued a disqualification that neither institution has published yet. These contradictions exist in the real world today; what changes with Trusteando is that they become visible rather than hidden.

The graph is built from assertions. An assertion is a signed statement by an entity that has authority over its own space. The protocol guarantees that the assertion is authentic — that it really was published by that entity, at that time, and has not been altered. It does not and cannot guarantee that the assertion is true.

**The protocol makes contradictions visible. It does not resolve them.**

When a verifier finds two contradictory assertions about the same node — the university says active, the college says suspended — the protocol surfaces both facts with their sources, dates, and signing authorities. What to do with that contradiction is a human decision, not an algorithmic one. The right answer depends on context: which source has more authority for this specific purpose, how recent each assertion is, whether one explicitly supersedes the other, what the stakes of the decision are.

This is not a weakness of the protocol — it is an honest reflection of reality. The world contains deception, error, institutional inertia, conflicts of interest, and deliberate omission. No algorithm can resolve that. What the protocol can do is raise the cost of maintaining contradictions silently:

- The `since/` timestamp records when each assertion was published. An institution that knew about a disqualification but delayed publishing it leaves a visible gap between the fact and its publication.
- The append-only model means that once a fact is published, it cannot be quietly retracted. Corrections add new facts; they do not erase the original.
- Contradictions between high-reputation nodes are themselves a signal worth examining. Two trusted sources disagreeing is more informative than one trusted source speaking alone.

The protocol's role is to make the evidence as complete, as traceable, and as tamper-resistant as possible. The judgment of what that evidence means — in a hiring decision, a medical context, a legal proceeding — belongs to the humans and institutions involved, not to the infrastructure that carries the claims.

A graph built from human assertions reflects human reality, including its contradictions. That is not a weakness of the design — it is an accurate model of how information actually works.

Opaque systems do not eliminate errors — they hide them. A disqualified professional who continues practising, a fraudulent degree that passes unchecked, an institution that knows of a problem and delays publishing it — these failures exist today, in every sector, precisely because the information is fragmented, siloed, and invisible to the parties who need it.

Trusteando does not promise to make the world honest. It promises to make inconsistencies harder to sustain in silence. When assertions are public, signed, and timestamped, contradictions become visible to anyone who looks. In many cases that visibility is enough: organisations confronted with a public discrepancy between their records and another institution's have a reason to act that they did not have before. The protocol does not resolve disputes automatically. It makes them harder to ignore indefinitely — and that, in practice, is often the most important step.

---

# Appendix A — Reserved Protocol Vocabulary

The following folders have specific semantics in the protocol. Any implementation must respect these names and their meanings. The list is extensible in future versions — no reserved name can be redefined by a particular implementation.

### Master reserved folder list

| Folder | Binding | Section | Meaning |
|---|---|---|---|
| `trusteando/` | Required | A.0 | Protocol root — everything lives here |
| `identity/` | Required | A.2 | Node identity declaration |
| `registry/` | Recommended | A.3 | Recognised entities registry |
| `externals/` | Optional | A.4 | Links to uncontrolled external systems |
| `since/` | Core | A.5 | Start of validity |
| `until/` | Core | A.5 | End of validity |
| `private/` | Core | 2.11 | Content requiring explicit authorisation |
| `plan/` | Core | A.24 | Declared intention |
| `execution/` | Core | A.24 | Accumulated facts |
| `steps/` | Core | 7.12 | Sequential workflow steps |
| `on/` | Core | A.26 | Effects namespace — event handlers |
| `extern/` | Core | A.25 | Reference to canonical data in another node |
| `cache/` | Optional | A.11 | Cached signed results with TTL |
| `docs/` | Optional | Style Guide §8 | Human-readable documentation |
| `status/` | Optional | A.9 | Current operational status |
| `features/` | Optional | A.15 | Capabilities declaration |
| `registry/compromised/` | Optional | A.22 | Keys declared compromised by the issuer |
| `revoked/` | Optional | A.22 | Self-revocation signal published by the node |
| `unrevoked/` | Optional | 2.10 | Reinstatement signal — supersedes a prior revoked/ |
| `old-identities/` | Optional | 4.7 | Chain of previous identities after migration |

Reserved folder names cannot be used as entity or node names. An implementation that receives a path where a reserved name appears in a non-protocol position must reject it or treat it as a protocol element — never as user data.


## A.0 The Root Folder — trusteando/

Everything the protocol needs lives under a single root folder: trusteando/. It is the only folder an entity needs to create, move, or upload to participate in the protocol. If the entity changes hosting while keeping the same domain, it moves the trusteando/ folder to the new server and everything keeps working—the entity_id does not change because it is calculated over the normalized domain, not the infrastructure.


```
casapepe.es/trusteando/     ← the entire protocol lives here
identity/
registry/
transactions/

```


## A.1 URL Normalisation for entity_id Calculation

The entity_id is calculated externally by any verifier—the entity cannot declare or influence it. This is a direct consequence of the golden rule: the structure declares information but cannot impose protocol fields.

Normalization is minimal and only removes technical fields from the URL scheme—never paths, never semantic components. This is critical to avoid collisions: if paths were removed, two different entities could end up with the same entity_id.


```
normalizar(url):
1. lowercase
2. remove www prefix
3. drop standard port (443 for https, 80 for http)
4. remove trailing slash
5. remove query params (?...)
6. remove fragments (#...)
7. keep the full path  ← never drop paths

entity_id = SHA-256(normalizar(url))

Examples:
https://WWW.CasaPepe.ES:443/  →  https://casapepe.es
https://casapepe.es/?ref=google  →  https://casapepe.es
https://casapepe.es/#inicio  →  https://casapepe.es
https://casapepe.es/santander/  →  https://casapepe.es/santander
https://casapepe.es/mio/  →  https://casapepe.es/mio

```

Documented edge case: if normalization removed paths, 'banco.es/santander/' and 'banco.es/mio/santander/' could collapse into the same entity_id. The rule of keeping full paths eliminates that possibility. Two entities with different paths are always distinct entities.


## A.2 trusteando/identity/

Contains everything the node declares about itself:


```
trusteando/identity/
public_key          ← node's public key (base64, P-256 curve)
hash_migracion      ← hash(entity_id, clave_emergencia_1)
used for voluntary migration to a new superior
renewed after each completed migration
hash_disputa        ← hash(entity_id, clave_emergencia_2)
used for dispute resolution
never revealed during normal migrations
entity_type         ← universidad | empresa | persona | servidor | ...
scope               ← declared scope of credentials it issues
root_certificate    ← certificate signed by the root (if any)
dispute_level       ← maximum level to which this node can escalate disputes

```


## A.3 trusteando/registry/

Contains everything the node certifies about others:


```
trusteando/registry/
old_identities/     ← historical chain of previous identities
maintained by the receiving superior, not by the node
contains the complete chain, not just the immediate link
since/              ← incorporation dates of recognized nodes
until/              ← departure dates of recognized nodes
externals/          ← links to uncontrolled external systems

```


## A.4 The externals/ Folder

Any folder under externals/ declares that the node links to something it does not control. The semantics are opposite to the rest of the folders—outside externals/, controlling the URL implies controlling the content. Inside externals/, the node only guarantees the authenticity of the link, not the destination's content.

### A.4.1 private/externals/ — Private references to third-party facts

The combination of `private/` and `externals/` is a distinct and valuable pattern: a private reference to a fact whose authority resides in another node.

```
person/trusteando/private/externals/
└── hacienda/
    └── tax-debt-status/ → https://aeat.es/expedientes/12345
```

The person is declaring four things simultaneously: a fact exists (Hacienda considers them a debtor), the authority over that fact is Hacienda, not the person (`externals/`), the evidence is at a specific URL, and the person does not want this reference to be publicly visible (`private/`).

This pattern solves a classic problem in verifiable identity systems:

- **Without Trusteando:** the person must request a certificate from Hacienda each time
- **With Trusteando:** the person publishes the reference once and reuses it as many times as needed, controlling who can see it via `grantReveal()`

The authority over the fact remains with the third party. The person certifies nothing — they only point to where the certification lives. This is honest: `externals/` explicitly declares that the destination is not under the person's control.

Use cases: expedients and registries the person does not control but can reference; initiating verification chains without exposing the reference publicly; demonstrating facts like "I am paying a tax plan" or "my criminal record is clean" by granting temporary access to the authoritative source.

## A.5 The since/ and until/ Convention

Credential validity dates are expressed as subfolders with ISO 8601 format (YYYY-MM-DD). Presence in since/ without a corresponding entry in until/ indicates that the credential remains valid. Both folders are under the superior's control, never the certified node's. The convention extends with two additional variants. from/ indicates a future activation condition whose exact moment is not a known date but an event: from/[time secure-wallet-is-ready]/ activates the folder when the wallet is ready, not on a fixed date. calculate/maximum/ and calculate/minimum/ are compound conditions: maximum requires that all subfolders are present (logical AND), minimum requires that at least one is present (logical OR). These conventions are especially useful to express launch conditions and conditional activation of roles.


## A.6 trusteando/identity/contact/ — optional, recommended for businesses

Public contact fields. If the folder exists, the following fields are the recognized standard—using non-standard fields for the same function reduces interoperability:


```
trusteando/identity/contact/
email                 ← RECOMMENDED
phone                 ← RECOMMENDED
web                   ← contact URL or form
address/              ← structured postal address
street
number
city
region
country             ← ISO 3166-1 alpha-2 (ES, FR, DE...)
postal_code
social/               ← social presences
twitter
linkedin
mastodon
languages             ← languages served

```


## A.7 trusteando/identity/location/ — optional, recommended for physical businesses


```
trusteando/identity/location/
lat                   ← REQUIRED if location/ exists
lon                   ← REQUIRED if location/ exists
place_id/
externals/          ← IDs in external systems — not controlled
google_maps
osm               ← OpenStreetMap
foursquare
area/
radius_km           ← service area for deliveries etc.

```


## A.8 trusteando/identity/classification/ — optional

Official classification of the business. If certified_by points to a Trusteando node, the verification is cryptographic. If it points to an external URL, verifyExternal() is used. If it resides under externals/, it is only a link without automatic verification.


```
trusteando/identity/classification/
[organismo]/          ← one entry per certifying agency
stars               ← integer 1-5
category            ← alternative alphanumeric scheme
certified_by        ← URL of the certifying node or page
since               ← date of the classification
until               ← expiration date (if applicable)
calculated/
verified          ← bool — verified against the source
last_checked
cache/ttl

```


## A.9 trusteando/status/ — optional, recommended for businesses with opening hours

Real-time status. Fields under status/ have a short TTL by default. open is always under calculated/—it is derived from open_at_today and close_at_today.


```
trusteando/status/
open_at_today         ← today's opening time (HH:MM) — for today's customers
close_at_today        ← today's closing time (HH:MM) — for today's customers
schedule/             ← regular schedule — for future bookings
monday/ ... sunday/
open_at
close_at
exceptions/         ← holidays, vacations, events
[YYYY-MM-DD]/
open_at
close_at
closed          ← explicit bool
note            ← 'Semana Santa', 'Vacaciones agosto'
occupancy/
current             ← current occupancy (0-100%)
capacity            ← maximum capacity
cache/ttl           ← short — real-time data
queue/
current_number      ← number currently being served
last_issued         ← last number issued
wait_minutes        ← estimated wait
cache/ttl
special/              ← active offers or events
[offer_id]          ← reference to transactions/trusteando/offers/
calculated/
open                ← bool: now >= open_at_today && now < close_at_today
open/cache/
ttl               ← seconds until the next state change

```


## A.10 trusteando/notifications/ — optional

Notification channel for parsers and aggregators. Allows staying synchronized without traversing the entire tree.


```
trusteando/notifications/
last_updated          ← timestamp of the last change
changelog/
[timestamp]/
path              ← which folder changed
action            ← created | updated | deleted | calculated
summary
cache/ttl             ← short — parsers check frequently

```


## A.11 The cache/ Convention — applicable to any folder

Any folder can declare its caching behavior with a cache/ subfolder. If it does not exist, the protocol’s default TTL is applied based on the content type.


```
property/cache/
ttl                   ← seconds until expiration (0 = do not cache)
strategy              ← no_cache | short | normal | long

Default TTL by content:
status/calculated/    → no_cache (0s)
status/occupancy/     → short (60s)
transactions/offers/  → short (300s)
identity/             → long (86400s)
registry/             → normal (3600s)
notifications/        → short (120s)

```


## A.12 trusteando/adapters/ — optional

Adapters that allow agencies with their own databases to expose their data through the same interface as protocol folders. The agency publishes the function, the parser calls it with protocol paths, and receives values as if they were folders.


```
trusteando/adapters/
[function_name]/
name
description
input_schema        ← protocol fields it needs
output_schema       ← fields it returns
version
cache/ttl

```


## A.13 Mandatory, Recommended and Optional Fields

MANDATORY — without these fields the node is not valid for the protocol:

- trusteando/identity/public_key
- trusteando/identity/hash_migracion
- trusteando/identity/hash_disputa
- trusteando/identity/entity_type
- trusteando/identity/location/lat and lon — if location/ exists
RECOMMENDED — optional but improves interoperability:

- trusteando/identity/scope
- trusteando/identity/root_certificate — if formal recognition exists
- trusteando/identity/contact/ — for businesses
- trusteando/identity/location/ — for physical businesses
- trusteando/status/ — for businesses with schedules
- trusteando/notifications/ — for nodes with frequent changes
OPTIONAL — free, but if implemented it must follow the standard schema:

- trusteando/identity/classification/
- trusteando/status/queue/
- trusteando/status/occupancy/
- trusteando/routing/
- trusteando/adapters/
- trusteando/features/
- trusteando/payments/
- trusteando/messaging/
- trusteando/media/
- trusteando/hooks/
Before creating a new field, verify whether an existing standard field already fulfills that function. Using standard fields ensures that parsers, aggregators, and third-party applications can interpret your node without additional configuration. Non-standard fields are ignored by implementations that do not know them—which is appropriate—but interoperability is lost. If you create a non-standard field that brings differentiating value, declare it under trusteando/extensions/ with justification.


## A.14 Presence-Based Boolean Convention

In the protocol, the presence of a folder represents true—its absence represents false. There are no fields with an explicit bool value. This simplifies verification—a parser checks whether the folder exists, not whether a value is true or false. And it simplifies publishing—the business creates or removes folders, not editing values.


```
trusteando/payments/accepted/
cash/          ← exists → accepts cash
card/          ← exists → accepts cards
google_pay/    ← exists → accepts Google Pay
← bizum/ does not exist → does not accept bizum

trusteando/features/
parking/       ← exists → has parking
wifi/          ← exists → has wifi
← pool/ does not exist → no pool

```


## A.15 trusteando/features/ — optional

Verifiable features of the business—structured facts that parsers and aggregators can use to filter and compare. It is not marketing—it is interoperable information. Some fields can have a certified_by if an agency verifies them.


```
trusteando/features/
parking/                ← bool by presence
type/                 ← free/ | paid/ | valet/
spaces                ← number of spots
wifi/                   ← bool by presence
speed_mbps
accessibility/          ← bool by presence
wheelchair/           ← bool by presence
elevator/             ← bool by presence
certified_by          ← optional certifying agency
pets/                   ← bool by presence
conditions            ← free text
pool/                   ← bool by presence
type/                 ← indoor/ | outdoor/ | both/
checkin/
from                  ← HH:MM
until                 ← HH:MM
checkout/
until                 ← HH:MM
restaurant/             ← bool by presence
cuisine               ← type of cuisine
smoking/                ← bool by presence — smoking allowed
air_conditioning/       ← bool by presence
heating/                ← bool by presence

```


## A.16 trusteando/payments/ — optional

Accepted payment methods and hook to the payment provider. The protocol does not implement payments—it defines where they connect. Changing providers is modifying a single field.


```
trusteando/payments/
accepted/               ← accepted methods — booleans by presence
cash/
card/
contactless/
bizum/
google_pay/
apple_pay/
invoice/              ← accepts invoice
agent/                  ← hook to the payment provider
provider              ← URL of the provider node
endpoint              ← where the transaction starts
externals/
stripe
redsys
paypal

```


## A.17 trusteando/messaging/ — optional

Hook to the messaging provider. The protocol does not implement messaging—it defines where it connects. Open protocols like Matrix are recommended over proprietary solutions.


```
trusteando/messaging/
agent/
provider              ← URL of the provider node
externals/
whatsapp
telegram
matrix              ← open protocol recommended
email               ← universal fallback

```


## A.18 trusteando/hooks/ — optional

General connection point for external services. Hooks follow a deliberate principle: the business should never be tied to a specific provider for technical reasons. The protocol defines the connection points—the market decides who implements them best. Changing provider should always be a business decision, never a technical migration.


```
trusteando/hooks/
payments/agent/provider     ← payment provider
messaging/agent/provider    ← messaging provider
booking/agent/provider      ← booking provider
delivery/agent/provider     ← delivery provider
analytics/agent/provider    ← analytics provider

```


## A.19 trusteando/media/ — optional

Verifiable images and multimedia content. Hashes guarantee that the content has not been altered. The actual storage may live on any CDN—the protocol only records the hash and the reference.


```
trusteando/media/
photos/
[photo_id]/
hash              ← SHA-256 of the file
url               ← where the image is stored
caption           ← description
taken_at          ← date
certified_by      ← who verified it is real (optional)
logo/
hash
url

```


## A.20 trusteando/rescue/ — optional but critical

The resilience folder declares where the node's operational copies are located if the primary server falls. It has a special caching property: it is the only folder in the protocol with mandatory cache priority.

The paradox of rescue/ is that if it is only read when the server falls, it is already too late to read it. That is why every parser visiting a node must cache rescue/ on its first visit and refresh it periodically—regardless of the rest of the tree's TTL. A parser without rescue/ cached cannot provide resilience during outages.


```
trusteando/rescue/
cache/
ttl                 ← 604800s (7 days) by default
priority            ← max — cached before any other folder
strategy            ← always — never skipped even if the rest are not cached
mirrors/              ← node copies on other servers
[mirror_id]/
url               ← URL of the mirror
last_synced       ← timestamp of the last synchronization
operator          ← who operates the mirror
verified/         ← bool by presence — mirror verified
fallback/
ttl                 ← how long the cached copy remains valid
backup/
frequency           ← how often a backup is made
last_backup         ← timestamp of the last backup
externals/
github            ← repo where the backup is published
ipfs              ← if using IPFS for distribution
emergency/
contact/
primary           ← primary contact
secondary         ← alternate contact
protocol          ← URL of the procedure document
escalation/
level_1           ← first escalation if primary does not respond
level_2           ← second escalation
root              ← escalate to the root as a last resort
status_page         ← URL where the node's status is published
hook/               ← wildcard for emergency management systems
provider          ← URL of the external system
endpoint          ← where the issue is reported
severity_levels/  ← severity levels handled
externals/        ← if using known systems
pagerduty
opsgenie
victorops

```

For use cases with critical availability requirements, the emergency/hook/ hook allows the protocol to connect with any external incident management system. The protocol does not specify that system—it only defines the connection point. The extensions required for those cases are declared under trusteando/extensions/ following the standard process.

The recommended cache order for any parser:


```
1. rescue/         ← ALWAYS — first visit and periodic refresh
2. identity/       ← high priority — rarely changes
3. notifications/  ← to detect subsequent changes
4. registry/       ← normal
5. status/         ← short or no_cache
6. rest            ← according to its own declared cache/ttl

```


## A.21 trusteando/safety/ — optional, recommended for public venues

Safety information for the establishment. It has two complementary layers: local basic information—available from the start, published by the establishment itself—and references to external agents with real authority over the location’s safety.

The presence of fully_implemented/ alongside an agent indicates that the agent has a complete and specific implementation for this establishment on its own node. Its absence indicates that the local information may be the only one available.


```
trusteando/safety/

← Local basic information
emergency_exits/
[exit_id]/
location          ← description or coordinates
floor
accessible/       ← bool by presence
assembly_point        ← external meeting point
capacity/
max                 ← legal maximum capacity
certified_by
defibrillator/        ← bool by presence
location
first_aid/            ← bool by presence
location
evacuation/
plan_url

← References to agents with real authority
authorities/
fire/
node              ← firefighters' node URL with specific entry
fully_implemented/ ← bool by presence — verifiable complete plan
externals/phone   ← fallback while no complete node exists
police/
node
fully_implemented/
externals/phone
medical/
node
fully_implemented/
externals/phone
food_safety/
node
fully_implemented/
building/
node              ← town hall — licenses and building regulations
fully_implemented/
insurance/
node              ← insurer — active policy
fully_implemented/

```

Safety for a venue is not what the venue says about itself—it is the network of authorities that have implemented something specific for it. The local information under safety/ is always useful as a starting point and redundancy. The presence of fully_implemented/ alongside an agent indicates that the authoritative source lives on the external node and that the prior work is complete and verifiable.

## A.22 trusteando/registry/compromised/ — optional

List of public keys or public hashes that the issuer declares compromised or invalid. It allows proactive revocation by the issuer without waiting for the subject to initiate a migration.

The main use case is ecosystem protection when the issuer has evidence that a key was breached before the subject can act: a university that detects stolen credentials can publish the compromised hash_publico in this folder immediately. Any verifier who checks registry/compromised/ knows that key must not be accepted even if the signature is cryptographically valid.

```
trusteando/registry/compromised/
{hash_publico_comprometido}/    ← bool by presence: this key must not be accepted
since/2026-03-18T10:30:00Z   ← compromise detection time
```

This folder complements the subject migration mechanism (section 4.7) and the state brokenado grammar. Together they cover the three revocation vectors: the subject migrates voluntarily, the issuer revokes proactively, and the network observes and declares state brokenado via quorum.

### Revocation authority hierarchy

Revocation is not symmetric — different actors have different authority over a node's validity. Consider a chain A → B → C where C is a person (Paco):

| Who revokes | Mechanism | Binding | Use case |
|---|---|---|---|
| Parent (B) | `registry/compromised/` or `until/` in B's space | Yes — B is the credential issuer | B detects Paco's key is stolen |
| Node itself (C) | `revoked/` subfolder in C's own space | Signal only — not binding | Paco notices his own key is compromised |
| Network | `[state brokenado]` by quorum | Yes — for nodes without accessible parent | Distributed detection of malicious behaviour |

The critical case is key theft: if Paco's key has been stolen, the attacker holding the key will not self-revoke — they will continue impersonating Paco. Self-revocation by C is therefore not a security mechanism. It is a voluntary signal, useful when Paco acts first, but insufficient when the attacker acts instead.

**The parent is the authority.** A verifier who wants to know whether Paco is still valid should not ask Paco — they should ask whoever issued Paco's credential (B). If B publishes Paco's hash in `registry/compromised/`, the chain is broken regardless of what Paco's own node says. B's revocation is binding; Paco's self-declaration is advisory.

A friend of Paco who holds a credential issued by Paco cannot verify its continued validity by consulting Paco's node alone. They must consult B. This is not a limitation — it is a direct consequence of the trust hierarchy: credentials derive their validity from the issuer, not from the subject.

### The `revoked/` subfolder — advisory self-revocation

A node may publish a `revoked/` subfolder to signal that it considers its own identity no longer valid:

```
paco.es/trusteando/
└── revoked/
    ├── since/2026-03-24/
    ├── [by self]/
    └── [reason "key-compromised"]/
```

`revoked/` is a reserved folder name. Its presence is a signal that verifiers may choose to honour, but it is not binding without confirmation from the parent. When present, a cautious verifier should treat the node as suspect and seek confirmation from the issuer before accepting any credential.

`revoked/` is distinct from `registry/compromised/`: the former is published by the subject about itself; the latter is published by the issuer about its subjects. Only the latter is binding.


The protocol defines a dispute resolution hierarchy analogous to an administrative appeal. Disputes are resolved at the lowest possible level. Elevating a dispute carries increasing cost to discourage frivolous use.


## B.1 Resolution Hierarchy


```
Level 1 — immediate superior
Juan has a dispute with the university
→ the university resolves it using Juan's hash_disputa
Cost: low

Level 2 — appeal
Juan does not accept the university's resolution
→ elevates to the university's superior node
→ that node resolves with its own authority
Cost: medium

...

Level N — the root as the highest court
Only for disputes that have exhausted lower levels
Or for disputes between nodes without a common superior
Cost: high

```


## B.2 The Cost of Dispute

Each level has an increasing cost—both economic and reputational. Elevating a dispute to the root without passing through the lower levels is not allowed except for specified exceptional cases. Losing a dispute elevated unnecessarily has consequences for the node that raised it. The model is analogous to court fees: they exist so that challenges are proportional to the seriousness of the matter.


## B.3 The Role of hash_disputa

hash_disputa—derived from clave_emergencia_2, never revealed in normal migrations—is the proof of identity before the arbitrator. In a dispute, the node demonstrates that it knows the key that produces that hash without revealing it. The arbitrator verifies the proof and resolves. If the dispute escalates, the hash_disputa accompanies the record—it is the cryptographic chain of custody for the disputant’s identity.


## A.23 Route grammar: definitive grammar

Route segments come in three types. The parser distinguishes them by the presence of brackets and whether there is whitespace inside them. The only reserved characters in the protocol are /, [ and ].

Type 1 — canonical identifier (without brackets)

```
hospital-la-paz     restaurante-casa-robles     universidad-malaga
Graph node with its own identity and authority over its space. The naming convention is type-name: the type is part of the identifier, making it canonical and self-explanatory out of context. Two consecutive nodes separated by / imply a hierarchical relationship with authority transfer.
```

Type 2 — Relation (brackets, without inner space)

```
[en]     [accredited-by]     [funded-by]     [agreement-with]
```

Semantic operator that connects nodes. It expresses a contextual relationship without authority transfer: the node preceding the operator does not certify the one that follows. The vocabulary is open—any natural-language term is valid.

Type 3 — Property with value (brackets, with inner space)

```
[camas 150]     [fundado 1964]     [estrellas 2]     [temperatura 23.5]
```

Attribute of the preceding node. The value is an immutable literal—it is not a navigable entity nor does it have its own URL. The space inside the brackets is the semantic marker that distinguishes this type from the previous one.

Rutas completas:

```
hospitales/[en]/madrid/hospital-la-paz/[camas 150][fundado 1964]
restaurantes/[en]/sevilla/restaurante-casa-robles/[estrellas 2][fundado 1952]
obras/[funded-by]/ue/obra-autovia-a7/[km 45][budget_euros 4500000]
sensor-42/[installed-in]/edificio-a/[planta 3][temp_celsius 85]
```

El parser:

```
def parse_segment(s):
if s.startswith('[') and s.endswith(']'):
inner = s[1:-1]
if ' ' in inner:
key, value = inner.split(' ', 1)
return {'type': 'property', 'key': key, 'value': value}
return {'type': 'relation', 'value': inner}
return {'type': 'node', 'value': s}

def parse_path(path):
return [parse_segment(s) for s in path.strip('/').split('/')]
```

This grammar does not replace the since/until convention for temporal facts nor the externals/ folder for uncontrolled links. It is an orthogonal extension that allows expressing relationships and properties directly in the path in a self-documenting way, without adding burden to the base protocol.

Reserved semantic conventions

On top of the base grammar, the following semantic conventions are defined. They do not add new segment types or reserved characters—they are conventional values within type 3 that the protocol interprets in standardized ways.

```
Reciprocal relations: mutual key
When a property's key is mutual, the value expresses the relation type and the counterpart. The relation is reciprocal: for it to be considered verified, the counterpart must publish an equivalent statement. There are two forms:
pedro/[mutua amistad juan]/  ← bilateral: pedro declares mutual friendship with juan
juan/[mutua amistad pedro]/  ← juan confirms: relation verified bilaterally
For relationships among more than two nodes the reserved value among-members is used on a collective node. The node declares the type of relation that exists among all its members:
grupo-amigos-madrid/[mutua amistad among-members]/  ← mutual friendship relation among all members
grupo-amigos-madrid/[miembro pedro]/
grupo-amigos-madrid/[miembro juan]/
grupo-amigos-madrid/[miembro ana]/
```


Critical distinction: attribute vs subordination

The choice between type 1 and type 3 to express a node's relationship with its members has precise semantic consequences:

```
grupo/miembro/juan/  ← juan is subordinated to the group. The group has authority over him.
grupo/[miembro juan]/  ← juan is an attribute of the group. There is no subordination nor authority.
```

For peer groups—friends, partners, collaborators—the attribute form is always used. The subordination form is reserved for real hierarchical relationships where the superior node has authority over the inferior.

Reserved state vocabulary

The protocol reserves three terms as canonical states of a node. They are extendable in future versions but not re-definable by particular implementations:

```
trusteando   ← full trust, verified with solid evidence
verifiado    ← meets minimums, basic approval
brokenado    ← partially compromised node: its local key has been breached but its identity remains valid within the remaining root quorum. It is not equivalent to invalid.
```

Convention [indicates] for source-backed claims

```
To express that a state or property is asserted by an external entity, use the relational operator [indicates] followed by the source. This allows attributing each claim to its origin, having multiple sources for the same node, and decoupling the protocol from specific infrastructure. The protocol does not designate official sources: any node can act as a source by indicating states about other nodes.
restaurante-casapepe/[state trusteado]/[indicates]/[source michelin.es]/
root-23/[state brokenado]/[indicates]/[observer root-45]/[observer root-78]/
restaurante-casapepe/[state trusteado]/[indicates]/[source michelin.es]/[source tripadvisor.com]/
```

Syntactic sugar: [X by Y] for states

```
To make human writing easier, the parser may transform [X by Y] into [state X][context Y] if and only if X is a term from the reserved state vocabulary (currently trusteando, verifiado, brokenado). This condition is essential: it ensures that parsers that do not implement the sugar encounter a predictable degradation—they see X as a recognizable part of the vocabulary—instead of silently misinterpreting it.
[brokenado by reputation]   →  [state brokenado][context reputation]
[trusteado by node-state]   →  [state trusteado][context node-state]
[reserva by ventana]        →  ordinary property (reserva is not a reserved state)
This convention does not introduce new reserved words into the grammar, only into the semantic vocabulary. The three-type grammar remains intact. by is free text in any context except when it precedes a value and is preceded by a reserved state.
```

Note: The Trusteando grammar can be understood by analogy with object-oriented programming. Folders are objects. Subfolders are methods (actions) or collections. Brackets are properties or values. And as section 2.10 states, once signed, values inside brackets are immutable—just like the fields of an immutable object in a functional language. The fundamental difference is that in Trusteando there is no encapsulation: everything is public by default except what is declared under private/. The graph is a space of verifiable objects, not private states.

## A.24 plan/execution convention for active processes

The since/until convention describes completed facts: periods whose start and end are already known. It is not sufficient to represent active processes where there is a planned intention and an execution that may differ. For those cases the protocol defines the plan/execution convention.

Any action, order, task, or process can contain two subfolders:

```
accion/
├── plan/        ← intention, signed by whoever orders or plans
│   └── [free]   ← start/, end/, resources/, priority/... whatever the domain needs
└── execution/  ← reality, signed by whoever executes
└── [free]   ← start/, end/, reports/, incidents/... whatever the domain needs
```

The contents of each subfolder are free: the protocol does not prescribe which folders go inside. Each domain defines the vocabulary it needs. This freedom does not sacrifice interoperability—the distinction between intention and reality is universal; the internal vocabulary is specific to the context.

The presence-based boolean semantics apply directly: the absence of execution/end/ indicates the process is ongoing. Its presence indicates completion. There is no explicit state field—the state is read from the structure.

Access control follows the protocol's general rule: whoever controls the URL under which content is published has authority over that content. plan/ is signed by whoever orders; execution/ is signed by whoever executes. An external verifier can compare both subfolders and derive the process's real state—delays, compliance, deviation—without needing to consult either party.

Examples of application:

```
— Emergencies: orden-001/plan/end/ is the deadline; orden-001/execution/end/ is when it was completed.
— Construction: cimentacion/plan/end/ is the planned date; cimentacion/execution/porcentaje/ is the real progress.
— Administrative procedure: solicitud/plan/resolucion/ is the legal term; solicitud/execution/resolucion/ is the actual date.
— Software: sprint/plan/story-points/ is the estimate; sprint/execution/completados/ is what was delivered.

```

## A.25 extern/ — external reference (link to canonical source)

The `extern/` prefix references data that lives in another node. It is the single-source-of-truth pattern — data is not duplicated, it is linked.

```
transfer/
├── [from extern/bank/santander/accounts/[client-id:C-123456]]/
└── [to extern/bank/bbva/accounts/[client-id:B-789012]]/
```

`extern/` differs from `externals/` (section A.4): `externals/` declares that an entity has a presence on an external platform it does not control. `extern/` is a typed reference to a specific node within the Trusteando graph.

### Lifetime of an extern/ reference

An `extern/` reference is valid for as long as the destination node exists and is reachable. This is the analogy to Rust lifetimes: the reference is borrowed from the destination node, and its validity is tied to the destination's existence.

If the destination node disappears — domain loss, deliberate deletion, `until/` expiry — the reference becomes a dangling pointer. The referencing node retains the link but the destination no longer responds. A verifier encountering an unresolvable `extern/` must treat the referencing fact as unverifiable, not as invalid. The fact that the transfer referenced account `C-123456` is still true — the account may simply no longer exist.

Implementations should distinguish three states when resolving an `extern/` reference:

| State | Meaning | Verifier action |
|---|---|---|
| Resolves and valid | Destination exists and is reachable | Accept |
| Resolves but expired (`until/` past) | Destination existed, now closed | Accept as historical fact |
| Does not resolve | Destination unreachable | Treat as unverifiable — do not reject, flag for review |

The third case is not an error in the referencing node — it is a state of the network. A node that published a valid `extern/` reference at signing time is not responsible for the destination disappearing later.

## A.26 on/ — effects namespace

The `on/` folder is the explicit boundary between the pure graph and side effects. Everything inside `on/` may have effects (notifications, webhooks). Everything outside is pure and immutable.

```
node/trusteando/on/
├── on-new-child/
│   └── [notify parent/]/
├── on-new-state/
│   └── [when state=brokenado]/
│       └── [action revoke-access]/
└── on-until/
    └── [action notify-expiry]/
```

Reserved event names: `on-new-child`, `on-new-state`, `on-key-revoked`, `on-quorum-reached`, `on-since`, `on-until`.

## A.27 [only-valid-for] and [example-not-valid-for]

Declare the intended scope of a credential. These two properties are mutually exclusive:

- `[only-valid-for X Y Z]` — closed scope: everything not listed is excluded
- `[example-not-valid-for X Y Z]` — open scope with illustrative exclusions only

```
uma.es/trusteando/students/b9/
├── [only-valid-for events-discounts festivals]/
└── [state brokenado]/
```

## A.28 [liability], [legal-terms], [max-value-per-use]

Declare legal responsibility and usage limits for a node or credential:

- `[liability url]` — entity responsible for this node's actions
- `[legal-terms url]` — link to machine-readable terms
- `[max-value-per-use EUR:50]` — maximum transaction value

```
uma.es/trusteando/students/b9/
├── [liability uma.es]/
├── [legal-terms uma.es/trusteando/legal/student-b9]/
└── [max-value-per-use EUR:50]/
```

## A.29 Trust level aliases

Short aliases for the three conformity states, consistent with the 9/10-letter naming convention:

| Alias | Full name | Meaning |
|---|---|---|
| `b9` | brokenado | Unverified, basic trust |
| `v9` | verifiado | At least one verified data point |
| `t9` | trusteado | Full verification |

Note: `T10` (uppercase) refers to the protocol name and root path, not a trust level.

## A.30 [external-root] — federated node declaration

Declares that a node's key was not derived from T10 via `grant_key` — it was chosen autonomously by the node operator. T10 can vouch for the existence of such a node but cannot verify its internal credentials.

```
uma.es/trusteando/
└── [external-root]/    ← key not derived from T10
```

The absence of `[external-root]` implies the key derives from T10 and T10 can verify the full chain. The distinction is always explicit — never inferred. See section 2.17 for the full federated node model.

## A.31 Workflow states — `[state failed]` and `[state aborted]`

Two additional states for the `steps/` workflow convention (see A.24):

- `[state failed]` — the step did not complete successfully. The step can be retried by publishing a new `[state completed]` or `[state aborted]`. The failed state is preserved in the history — it is never deleted.
- `[state aborted]` — the step was cancelled definitively. It cannot be retried. Subsequent steps in the same workflow do not start.

```
execution/steps/
├── 01-contract/[state completed]/since/2026-03-22/
├── 02-social-security/
│   ├── [state failed]/since/2026-03-23/      ← preserved in history
│   │   └── [reason "DNI caducado"]/
│   └── [state completed]/since/2026-03-24/   ← successful retry
└── 03-system-access/[state pending]/
```

If a step has `[state failed]` or `[state aborted]`, subsequent steps do not start. The history of failures is append-only and is never removed.

## A.32 [reason] — failure description

Optional property on a failed or aborted state. Provides a human-readable description of why a step failed. Not machine-parsed — for audit and human review only.

```
[state failed]/
└── [reason "DNI caducado"]/
```

`[reason]` only appears as a child of `[state failed]` or `[state aborted]`. It has no meaning in other contexts.



---

# Appendix C — Minimal Protocol API

The API defines how to interact with a node. It is independent of the implementation—a node can be a static web server, a REST API, or a microservice, as long as it exposes these methods with the same semantics. The API is a convenience to facilitate interoperability—what is not here can be implemented freely without breaking the protocol.


## C.1 Verification


```
verify(url, folder)
→ is this node in this folder now?
→ devuelve: { valid: bool, timestamp: ISO8601, issuer: url }

getIdentity(url)
→ obtener trusteando_identity/ de un nodo
→ devuelve: { public_key, hash_migracion, hash_disputa,
entity_type, scope, root_certificate }

```


## C.2 Lifecycle


```
create(url, entity_type, scope)
→ crear un nuevo nodo en el sistema
→ genera claves, publica trusteando_identity/

certify(subject_url, folder)
→ add a node to a registry folder
→ equivale a emitir una credencial

revoke(subject_url, folder)
→ eliminar un nodo de una carpeta del registro
→ equivalent to revoking a specific credential

```


## C.3 Migration


```
migrate(new_superior_url, emergency_key_1)
→ request migration to a new parent
→ el nodo revela emergency_key_1 al nuevo superior

adoptNode(subject_url, emergency_key_1)
→ the parent accepts the migration
→ verifica emergency_key_1 contra hash_migracion publicado
→ publica old_identities/ con cadena completa
→ aprueba ante el root

```


## C.4 Contextual Privacy


```
requestReveal(subject_url, field, context)
→ el receptor solicita acceso a un campo protegido
→ field: ruta de la carpeta private/
→ context: declared purpose of the request
→ devuelve: { request_id, status: pending }

grantReveal(request_id, disclosure_level)
→ el sujeto concede acceso al campo solicitado
→ disclosure_level: RESULT_ONLY | ISSUER_ONLY |
CONTENT | FULL_CHAIN
→ transferable: bool  ← si el receptor puede demostrarlo a terceros

denyReveal(request_id)
→ el sujeto deniega la solicitud


```


---

# Appendix D — Business Layer — transactions/trusteando/

The Trusteando protocol is general—it serves identity, certifications, and any verifiable fact. The business layer is an optional extension that standardizes how verifiable transactions and reputation are expressed. Any node that wants to participate in the verifiable transactions ecosystem adopts the transactions/trusteando/ convention.

The convention is voluntary but strategically important: any system that wants to index, aggregate, or verify transactions and reputation interoperably must speak this language. The folder is the declaration of participation—no central registry or special API is required.


## D.1 Structure of transactions/trusteando/


```
casapepe.es/transactions/trusteando/

visits/                              ← verified visits
since/YYYY-MM-DD/[visitor_hash]

reviews/                             ← review collection
review/                            ← review object (primary data)
set_id                           ← unique identifier
type                             ← enumerated integer (see D.2)
content_hash                     ← SHA-256 of the text
rating                           ← numerical rating
visit_ref                        ← reference to visit (if type >= 2)
payment_ref                      ← reference to payment (if type >= 3)
private/             ← optional private fields
calculated/                        ← automatically derived indexes
verified_visit/                  ← reviews with type >= 2
verified_paid/                   ← reviews with type >= 3
verified_identity/               ← reviews with type >= 1
by_rating/                       ← index by rating

offers/
[offer_id]/
terms
valid/since/until/
eligibility/

reputation/
calculated/
score                            ← calculated aggregate
total_reviews                    ← count of unique set_id
total_verified_visits            ← count of type >= 2
total_verified_paid              ← count of type >= 3

```
## D.2 Review Enumerated Type

The type field is an enumerated integer where the order reflects the verification level—the higher the number, the greater the cryptographic guarantee:

type

name

guarantee

```
0
```

unverified

no verification—anyone can review

```
1
```

verified_identity

author's identity verified

```
2
```

verified_visit

physical presence verified

```
3
```

verified_paid

purchase verified via POS or payment token

```
4
```

verified_visit_and_paid

physical presence and purchase verified

The order matters for queries: reviews with type ≥ 2 return only reviews with verified physical presence. Counting unique reviews always means counting distinct set_id values under reviews/review/—the indexes in calculated/ are derived views, not duplicates.


## D.2b Publishing Facts vs Publishing an Index of Others' Facts

A critical semantic distinction in the business layer: the restaurant does not invent reviews — it publishes an *index* of reviews that exist in other nodes. The `[source]` property is the corroboration mechanism:

```
mi-restaurante.es/transactions/trusteando/
├── offers/                         ← the restaurant publishes its own offers
│   └── menu-del-dia/
│       ├── [price 15]/
│       └── valid/until/2026-03-31/
└── reviews/                        ← an index of reviews received
    ├── pepe/
    │   ├── [rating 4]/
    │   └── [source pepe.es/reviews/mi-restaurante]/  ← original review lives here
    └── ana/
        ├── [rating 5]/
        └── [source ana.es/reviews/mi-restaurante]/
```

The restaurant is saying: "I have seen this review and I include it in my index — here is the verifiable source." The original review is signed by Pepe at `pepe.es/` — the restaurant cannot forge it. A verifier can cross-reference both sources.

This is analogous to how Google shows TripAdvisor reviews: Google indexes them, not invents them. The `[source]` property makes this explicit — the restaurant does not claim authorship of the review, only that it has indexed it.

Three types of content coexist in `transactions/trusteando/`:

| Folder | What the restaurant publishes | Authority |
|---|---|---|
| `offers/` | Its own commercial offers | The restaurant |
| `reviews/` | An index of received reviews | Original authors via `[source]` |
| `reputation/calculated/` | Its own computed score | The restaurant (signed, auditable) |

**`calculated/` is a declaration, not a magic state.** The restaurant signs its own calculation. If it lies, anyone can verify by consulting the source reviews. Lying has a reputational cost. An external aggregator can ignore the restaurant's `calculated/` and compute its own metric independently.

## D.3 The calculated/ Folder

Everything under calculated/ is automatically derived from the primary data and independently verifiable by any third party. It is never written by hand. The protocol defines two functions to manage calculated fields:


```
updateIndex(entity_url, collection)
→ recalculates the indices of a collection
→ se llama tras cada create/update/delete de un objeto
→ idempotente — ejecutarlo dos veces da el mismo resultado

checkIndex(entity_url, collection)
→ verifies that indices match primary data
→ devuelve: { valid: bool, mismatches: [...] }
→ any third party can run it — autonomous public audit

```


## D.4 The routing/ Folder

Explicitly declares who an entity sends data to and for what purpose. It is active transparency—any third party can see exactly which processors are authorized. Compatible with the GDPR’s record of processing activities.


```
trusteando/routing/
processors/           ← entities authorized to process my data
confidencenode.org  ← the root — verifications
analytics.es        ← authorized analytics processor
subscribers/          ← entities receiving event notifications
malaga.org          ← subscribed regional aggregator
webhooks/             ← endpoints that automatically receive events

```


## D.5 Semantics of Reserved Prefixes

folder

semantics

```
externals/
```

link to something not controlled by the node

```
calculated/
```

automatically derived, verifiable with checkIndex()

```
private/
```

exists but is not exposed by default—grantReveal() for access

```
routing/
```

processors and subscribers authorized to receive data

```
fully_implemented/
```

bool by presence—the referenced external node has a complete and specific implementation. Its absence indicates a partial or in-progress implementation—the local information may be the only one available.



## D.6 The Central Property

A review under transactions/trusteando/reviews/review/ is not an opinion—it is an edge in the graph that connects a verified node with a verified visit or purchase. The content may be subjective but the presence is objective. Any verifier can confirm it by consulting public URLs.

This property destroys fake reviews by construction, not by moderation. And the audit is public and autonomous—anyone can execute checkIndex() and confirm that the reputation indexes are consistent with the primary data.


## D.7 Strategic Value of the Name

The transactions/trusteando/ convention associates the name Trusteando with the protocol’s verifiable transactions standard. Any implementation that adopts this convention strengthens that link. The domain trusteando.app is the reference implementation—the natural entry point for anyone looking to understand what this folder means or how to implement it.



---

# Appendix E — Intentionality Dictionary — Inspiration from Web Mechanisms

Trusteando draws inspiration from established web mechanisms. The following dictionary helps understand the intentionality of each folder—it does not imply technical equivalence or direct compatibility. Where the web solved similar problems, Trusteando adds cryptographic signatures, identity semantics, and public verifiability.

Trusteando

Inspired by

Key difference

```
trusteando/redirect/
```

HTTP 301/302

Signed, part of the identity graph, with date and historical chain

```
trusteando/notifications/
```

HTTP headers / ETag

Declarative in a folder, verifiable, without header negotiation

```
notifications/changelog/
```

RSS / Atom

No feed server, direct pull over public folders

```
trusteando/routing/
```

robots.txt / CORS

With signed authorization semantics compatible with GDPR

```
trusteando/registry/
```

sitemap.xml

With cryptographic signatures and credential semantics

```
calculated/
```

Cache / ETag

Publicly auditable with checkIndex()—not just for performance

```
externals/
```

<a href> externo

Explicitly declares lack of control over the destination

```
routing/processors/
```

GDPR registro tratamiento

Cryptographically verifiable, not just declarative

```
private/
```

— (sin equivalente)

Contextual privacy with selective disclosure—uncommon on the web


The trusteando/redirect/ folder deserves special mention. Unlike an HTTP 301, a redirect in Trusteando is a signed declaration of identity migration—with date, reference to the emergency key used, and as part of the node's historical graph. It is not just a technical routing instruction but a verifiable fact with the same guarantees as any other credential in the protocol.


```
trusteando/redirect/
target          ← destination URL (new identity)
permanent       ← bool — definitive or temporary migration
since           ← migration date
migration_ref   ← reference to the migration process in old_identities/


```


---

# Appendix F — Author Identity, Version Chain and Governance Model

This appendix applies the protocol itself to establish the author's identity, the authenticity chain between whitepaper versions, and the initial governance model. It is both a specification and a demonstration: the author uses Trusteando to prove that they are the author of Trusteando.

## F.1 Author's canonical identity

The author's identity uses the autonomous mode defined in section 4.10. The canonical identifier h1 directly incorporates the protocol grammar, making it semantically verifiable by anyone who understands the specification:


```
h1 = hash("ConfidenceNode0/[author-of Trusteando Protocol]")
h2 = hash(h1 + secret_v02)         ← public hash of this version
h3 = hash(h1 + secret_disputes)    ← clave de disputas, independiente
```

h1 is permanent and public: anyone can compute it by applying SHA-256 to the literal string. h2 changes with each version and only the author can generate it because it requires secret_v02, which is never published. h3 is independent of h2—compromising one does not compromise the other.

Secret management does not require any human to memorize them. The reference wallet (planned in v0.2) generates them, stores them in the device's secure enclave, and distributes encrypted copies to trusted collaborating wallets using the protocol itself. The author grants custody roles by publishing under their own URL—no one can self-proclaim a custodian:


```
author/s-w-c/
├── [w-c-1 secret-wallet-collaborator]/
│   ├── since/2026-03-18/             ← custody start
│   └── [state trusteado]/          ← active and verified
├── [w-c-2 secret-wallet-collaborator]/
│   ├── since/2026-03-18/
│   └── [state trusteado]/
├── [w-c-3 secret-wallet-collaborator]/
│   ├── since/2026-03-18/
│   └──                              ← empty until verified
└── [quorum 2]/                      ← minimum to reconstruct the secret
```

If a custody is revoked or compromised, the author removes or updates the entry under s-w-c/ and the state becomes brokenado. The full lifecycle—granting, activation, revocation—uses exclusively the protocol’s standard grammar. There are no special APIs or external permission tables.

## F.2 Version chain

Each whitepaper version publishes its h2 and a migration_ref linking it to the previous version. Only someone who knew the previous version’s secret can generate a valid migration_ref. The chain is unidirectional: version N accredits version N+1, never the reverse.


```
confidencenode.org/protocolos/trusteando/
├── v0.1/
│   ├── h1/                       ← author's canonical identity
│   └── h2/                       ← v0.1 public hash
├── v0.2/
│   ├── h1/                       ← same permanent h1
│   ├── h2/                       ← v0.2 public hash
│   ├── migration_ref/            ← hash(h2_v01 + h2_v02)
│   └── [acredited-by]/v0.1/      ← only the previous version accredits
└── v0.N/  ← each future version extends the chain likewise
```

A reader who finds the whitepaper in any repository can verify its authenticity by checking the chain on confidencenode.org—there is no need to trust the repository. A document claiming to be v0.3 without a valid migration_ref linking it to v0.2 is not authentic.

## F.3 Initial governance model and transition to polycentrism

The author does not activate their root role until a set of launch conditions are simultaneously met—conditions expressed in the protocol itself using calculate/maximum. This decision is deliberate: the author will not assume institutional responsibility before the system has the necessary tools and endorsements. Accrediting new roots uses the [agent role] convention: the key is the agent identifier, the value is its role in the ecosystem.


```
author/
├── h1/
├── [author-of Trusteando Protocol]/
├── [confidencenode0 root-founding-author]/confidencenode0/
│   ├── [state verifiado]/          ← identity established, root not yet active
│   └── from/calculate/maximum/      ← all launch conditions must be met
│       ├── [time secure-wallet-is-ready]/
│       ├── group-of-legal-experts-selected-for-launch/execution/[state ready]/
│       ├── group-of-security-experts-selected-for-launch/execution/[state ready]/
│       └── group-of-computer-science-experts-selected-for-launch/execution/[state ready]/
├── [spain root-national-agent]/spain-root-national-agent/
├── [france root-national-agent]/france-root-national-agent/
├── [eu root-supranational-agent]/eu-root-agent/
└── [quorum 3]/               ← minimum roots for sensitive operations
```

The [agent role] convention is infinitely extensible. Any entity can be accredited with any role. The current roles are root-founding-author, root-national-agent, and root-supranational-agent, but the protocol does not restrict the vocabulary. Any verifier can discover all active roots by looking for nodes with the suffix -root- in their role—without consulting any central registry.

The transition to polycentrism is irreversible by design: once the national agents are accredited and operating with their own quorum, the system no longer depends on the author’s signature to function. The author cannot regain unilateral control. Accreditation is a public, signed, and permanent act in the graph.

Each expert group has its own internal structure. The author names the group and assigns it the approve-launch-conditions role by publishing under their own URL. The group names its members and controls when it declares approval by publishing under its own URL. Neither party can fake the other’s action:


```
author/[group-of-computer-science-experts-selected-for-launch approve-launch-conditions]/
← the author names the group and assigns the role

group-of-computer-science-experts-selected-for-launch/
├── [expert-1 is-member]/    ← the group names its members
├── [expert-2 is-member]/
├── [expert-3 is-member]/
├── [quorum 3]/              ← minimum for the group's approval
├── plan/[state approved]/        ← the group approves the plan
└── execution/[state ready]/    ← the group declares conditions satisfied
```

The separation of authorities is total: the author controls who the experts are, the experts control when they approve. When the three groups have published [state ready] under their own URLs, and the wallet is ready, calculate/maximum will evaluate all conditions as satisfied and the root will become active automatically—without any additional act by the author.

## F.4 Illustrative example of an internal structure for a national agent (non-normative)

The internal governance decisions of each national agent are sovereign. The protocol imposes no model. The following example is purely illustrative—a possible structure a state could adopt as a starting point, freely adapting it to its legal and institutional framework.


```
spain-root-national-agent/
├── [acredited-by]/author/          ← accredited by the founding author
├── [quorum 3]/                    ← internal quorum for sensitive operations
├── [sepe root-domain-agent]/sepe-root/   ← domain agent: employment
├── [mec root-domain-agent]/mec-root/    ← domain agent: education
├── [aeat root-domain-agent]/aeat-root/  ← domain agent: taxation
├── regions/
│   ├── [andalucia root-regional-agent]/andalucia-root/
│   └── [catalunia root-regional-agent]/catalunia-root/
└── [since 2027-01-01]/               ← national agent activation date
```

In this example, the Spanish national agent delegates sectoral domains to ministries and territorial domains to autonomous communities, each with its own node and role. The hierarchy is whatever the state decides—the protocol only provides the grammar. An external verifier querying spain-root-national-agent can automatically discover all subordinate agents without any central directory.


F.5 Illustrative example: Spain's subscription to the protocol via BOE publication (non-normative)

The Boletín Oficial del Estado is Spain’s official legal publicity mechanism. Any act published in the BOE has legal validity in the Spanish legal system. Its use as the anchoring point for protocol subscription solves the institutional bootstrap problem in one step: the publication in the BOE is the official declaration of adherence and the public proof of the national agent’s key, verifiable by any citizen, company, or administration.

The illustrative flow would be as follows. The Government approves adhesion to the Trusteando protocol through a ministerial order. The order is published in the BOE with the minimum necessary content: the national root node’s URL, the agent’s public key, and the public hash that confirms the identity. From that moment, any verifier can consult the BOE and the node directly:


```
boe.es/trusteando/
├── identity/
│   ├── public_key/                  ← national agent’s public key
│   ├── hash_publico/               ← autonomous identity proof
│   └── [boe-ref BOE-A-2027-XXXX]/    ← reference to the official publication
├── [acredited-by]/author/         ← accredited by the founding author
├── [state trusteado]/            ← active and verified node
├── since/2027-01-01/
├── registry/                      ← register of nodes recognized by Spain
│   ├── [uma root-domain-agent]/university-malaga/
│   ├── [ucm root-domain-agent]/university-complutense/
│   └── [aeat root-domain-agent]/aeat-root/
```

The reference [boe-ref BOE-A-2027-XXXX] links the digital node with the official legal act. A citizen, company, or court can consult the BOE, find the ministerial order, and cryptographically verify that the node at boe.es/trusteando is exactly the one the Government declared on that date. The identity proof has two independent layers: the cryptographic (hash_publico) and the legal (publication in the BOE). Compromising one does not compromise the other.

The same mechanism applies to any country with an official register equivalent to the BOE: the Official Journal of the European Union, the Journal Officiel in France, the Bundesanzeiger in Germany. The national official publication is the bridge between the preexisting legal system and the protocol. There is no need to create new legal infrastructure—just use what already exists.


## F.6 Illustrative example: United States subscribing via the Federal Register (non-normative)

The American equivalent of the BOE is the Federal Register, the official journal of the federal government of the United States. However, the American institutional system has a relevant particularity for the protocol: authority is distributed from the origin among independent federal agencies with specific technical competencies. This makes the American model naturally polycentric from the start—rather than a single national agent, several agencies can act as independent domain roots from day one.


```
federalregister.gov/trusteando/
├── identity/
│   ├── public_key/
│   └── [fr-ref FR-2027-XXXXX]/        ← reference to the Federal Register
├── [acredited-by]/author/
├── [state trusteado]/
├── since/2027-01-01/
├── [quorum 3]/                    ← minimum quorum among agencies
└── registry/
├── [nist root-domain-agent]/nist-root/     ← technical standards
├── [ftc root-domain-agent]/ftc-root/       ← consumer protection
├── [ed root-domain-agent]/dept-education-root/  ← academic credentials
└── [mit root-domain-agent]/mit-root/       ← example: academic institutions
```

The structural difference with respect to the Spanish model illustrates how the protocol adapts to different institutional systems. Spain centralizes in the BOE and delegates downward. The United States distributes authority from the origin among independent agencies. In both cases the publication mechanism is the same—the official register URL as the legal anchor—but the resulting graph topology reflects each country’s real institutional architecture.



---

# References

- ECDH — Elliptic Curve Diffie-Hellman: RFC 7748, Bernstein et al., 2016
- HMAC — Hash-based Message Authentication Code: RFC 2104, Krawczyk et al., 1997
- SHA-256 — Secure Hash Algorithm: FIPS PUB 180-4, NIST, 2015
- W3C Verifiable Credentials Data Model: W3C Recommendation, 2022
- W3C Decentralised Identifiers (DIDs): W3C Recommendation, 2022
- TLS 1.3 — Transport Layer Security: RFC 8446, Rescorla, 2018
- eIDAS — Regulation (EU) 910/2014 on electronic identification, 2014
- GNU General Public License v3: Free Software Foundation, 2007

---

# Appendix G — The Authority Pattern: A Unifying Principle

The protocol's domain-specific examples — banking, healthcare, education, identity, software development — are not separate use cases. They are instances of the same underlying pattern.

**The pattern:** authority over a fact resides with one entity, but verification is needed by another. The pre-Trusteando solution to this pattern is always some combination of bilateral agreements, trusted intermediaries, proprietary APIs, or repeated verification requests to the authority. The cost grows as O(n²) with the number of participants.

Trusteando makes this pattern explicit by design:

- **Authority** is demonstrated by controlling a URL — no declaration needed, no registration required
- **Publication** is the act of making a fact verifiable — the folder structure is the credential
- **Reference** (`extern/`) is how one node points to another's authority without duplicating the fact
- **Verification** is a direct query to the authority's URL — no intermediary, no prior agreement

The pattern is isomorphic across domains:

| Domain | Authority | Fact | Verifier | Pre-Trusteando friction |
|---|---|---|---|---|
| Banking | Bank A | Account, balance | Bank B | SWIFT, bilateral APIs |
| Healthcare | Hospital | Medical record | Another hospital | Fax, proprietary systems |
| Education | University | Degree | Employer | Request to university, paper copy |
| Professional licences | College | Active membership | Any service | Query to college per request |
| Identity | State | Legal identity | Any institution | Certificate issued on demand |
| Rust traits | Crate defining the type | Trait implementation | Crate using the trait | Orphan rules — type owner must implement every trait |

The Rust case deserves a note. The orphan rule exists to guarantee coherence: only one implementation of a trait for a given type can exist, and it must live in one of the two defining crates. This prevents ambiguity but creates a bottleneck — the author of the type must implement every trait the ecosystem might want to use, or the ecosystem must wait.

Trusteando's model resolves this structurally. Any entity can publish a credential about any node it controls. A crate that wants `Matrix` to be serialisable with `nextserde` publishes that fact under its own URL — not under the type author's URL, but under the URL of the crate asserting the compatibility. The verifier queries the asserting crate directly. There are no orphan rules because there is no central registry that enforces uniqueness — each authority publishes its own view, and verifiers decide which authorities they trust.

The cost is that coherence is no longer guaranteed by construction — two crates could publish contradictory trait implementations for the same type. But as section 2.10 notes, this is the general condition of a distributed fact base: contradictions are visible, timestamped, and traceable. The verifier decides which source to trust, exactly as they do when two institutions publish contradictory facts about a person.

### The stateless verifier node

A further application of the authority pattern is a node that verifies paths without storing any record. The node holds only its secret key. The existence of a path is proven by presenting the correct hash — derived from the node's secret and the path string. The node stores nothing; it verifies on demand.

```python
# The node derives the path key on demand
key_path = hash(node.secret + "event/2026-03-24/attendee/juan/")

# The attendee presents (path, key_path) to a verifier
# The verifier asks the node: does this path exist?
node.verify_child_authorship(path, context, key_path)
# → yes, because only the node could have derived key_path
```

This is useful whenever the verifying authority should not accumulate records — temporary event access, presence verification, one-time credentials. The node is a pure function: given a path, it can confirm or deny its validity without ever having written anything to disk.

Combined with signed declarations from the parties involved, this pattern produces a triple proof: the node recognises the path, and the parties have each signed their participation. Each element is independently verifiable. The node's statelessness is a privacy property, not a limitation — it cannot be compelled to produce records it never held.

The operational consequences are significant:

- **No storage cost.** The node accumulates no records. There is no database to protect, back up, scale, or make GDPR-compliant. The only thing the node needs to custody is its secret key.
- **Zero marginal cost per verification.** Each verification is a hash operation — microseconds, no I/O, no database query. The cost does not grow with the number of verifications performed historically.
- **Trivial horizontal scaling.** Any replica that holds the key can verify. There is no shared state to synchronise between replicas.
- **Resistance to extraction attacks.** An attacker who compromises the server obtains no record of who verified what, because those records do not exist.

This inverts the usual assumption about verification systems — that verifying requires a database. In this model, the database is replaced by a mathematical property: only the node that holds the secret can confirm that a given path is valid. The secret is the database.

### Multi-party commutative hash

When two or more parties must each contribute to a joint proof — and the result must be independent of the order in which they present — the combination function must be commutative. XOR has this property but is reversible: knowing the result and one input reveals the other. The correct construction is XOR followed by a hash:

```python
def joint_hash(h1, h2):
    return hash(h1 ^ h2)   # XOR for commutativity, hash for non-reversibility
```

The XOR step combines the two inputs without imposing an order. The hash step makes the result non-reversible — knowing `hash(h1 ^ h2)` and `h1` does not reveal `h2`. Both properties are necessary; neither is sufficient alone.

This extends naturally to N parties:

```python
def joint_hash_n(*hashes):
    xor_combined = hashes[0]
    for h in hashes[1:]:
        xor_combined ^= h
    return hash(xor_combined)
```

The result is the same regardless of the order in which parties contribute. A verifier recomputes the same function and checks equality — no ordering protocol required, no coordinator needed.

| Element | Standard Trusteando | Stateless verifier |
|---|---|---|
| Parent node | University | Event node |
| Child | Professor Juan | Attendee path |
| Child key | `hash(uni.secret + path)` | `hash(node.secret + path)` |
| Proof of authority | `verify_child_authorship` | Presentation of `key_path` |
| Additional credential | Degree certificate | Signed declaration of participation |
| Node storage | Registry entries | Nothing — key only |

In each case, the authority publishes once. Verification is direct and permanent — not a request that the authority must answer each time, but a query to a public URL that anyone can make at any moment.

```
# The generic structure
authority.es/trusteando/registry/subject/
└── [role]/since/date/         ← published once by the authority

# Verification — no cooperation required
GET authority.es/trusteando/registry/subject/
→ fact exists, signed, timestamped, permanent
```

The protocol does not prescribe domain-specific solutions. It provides a grammar for expressing authority relationships. Domains that adopt the same schema become interoperable by construction — not because they negotiated an agreement, but because they published their data in a common structure that any verifier already knows how to read.

A reader who reaches this appendix after the use cases in sections 7.1–7.13 may notice that all those examples share this structure. That is not a coincidence. The use cases are not a collection of recipes — they are manifestations of a single structural principle. The protocol is a language for expressing authority relationships, and that language is general enough to cover any domain where authority and verification are held by different parties.

> *Trusteando does not solve the problem of authority. It makes authority explicit — and that, in practice, is often enough.*

### Multilateral consent — independent nodes referencing a shared document

A final application of the authority pattern is multilateral consent: several independent parties need to record that they all agreed to something, without any of them being a child of the others.

The canonical structure uses a single neutral node as the shared anchor — a notary, a registry, or any trusted third party — not owned by any of the parties, but referenced by all of them:

```
# The canonical document — lives under the neutral node
notary.es/trusteando/consents/[id:C-2026-001]/
├── [ana consent]/           ← notary certifies ana's authorisation
├── [juan consent]/          ← notary certifies juan's authorisation
└── since/2026-03-24/

# Ana declares her consent, referencing the canonical document
ana.es/trusteando/private/
└── [juan consent extern/notary.es/trusteando/consents/[id:C-2026-001]]/

# Juan declares his consent, referencing the same document
juan.es/trusteando/private/
└── [ana consent extern/notary.es/trusteando/consents/[id:C-2026-001]]/
```

The document `[id:C-2026-001]` is the shared key that links three independent nodes without making any of them a child of another. The notary certifies both parties' authorisation in its own space. Ana and Juan each declare their consent in their own private space, pointing to the same anchor.

**The parent folder is the signer.** `notary/[ana consent]` means: the notary (the parent folder) certifies Ana's authorisation. `ana/[juan consent extern/...]` means: Ana declares her consent to Juan, with the evidence at the notary's URL. The authority chain is encoded in the structure — no special syntax required.

**The canonical document is the anchor, not the agreement.** It does not need to contain the full terms — it is the shared reference that makes all three nodes point to the same fact. The content lives in each party's space; the anchor makes them converge.

**Finding the right structure takes practice.** Even with a clear grammar, deciding where each fact should live requires domain modelling skill. The examples in this appendix and in sections 7.10–7.13 are reference patterns, not illustrations. When modelling a new domain, start from the closest canonical example.

---

# References

- ECDH — Elliptic Curve Diffie-Hellman: RFC 7748, Bernstein et al., 2016
- HMAC — Hash-based Message Authentication Code: RFC 2104, Krawczyk et al., 1997
- SHA-256 — Secure Hash Algorithm: FIPS PUB 180-4, NIST, 2015
- W3C Verifiable Credentials Data Model: W3C Recommendation, 2022
- W3C Decentralised Identifiers (DIDs): W3C Recommendation, 2022
- TLS 1.3 — Transport Layer Security: RFC 8446, Rescorla, 2018
- eIDAS — Regulation (EU) 910/2014 on electronic identification, 2014
- GNU General Public License v3: Free Software Foundation, 2007

---

*Author: confidencenode.org/members/confidencenode0 — confidencenode.org/protocolos/trusteando*

*March 2026 — v0.2 — ConfidenceNode — confidencenode.org*

*Trusteando is free software licensed under the GNU GPL v3.*

*The protocol is open. Anyone can implement it, verify it, and build on top of it.*

*trusteando.app — github.com/ConfidenceNode*

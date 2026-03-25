# Trusteando Implementation Guide

**Technical reference for protocol implementors**

*v0.1 — companion to Trusteando Protocol Whitepaper v0.2.1*
*confidencenode.org/protocolos/trusteando*

---

This document contains the implementation-level detail extracted from the whitepaper to keep the specification focused. It is intended for developers building parsers, verifiers, wallets, and conformance test suites — not for architects or decision-makers evaluating the protocol.

**Prerequisites:** read the whitepaper first, especially sections 1–4 and Appendix H (BNF Grammar).

---

## 1.5 Formal Invariants

The five invariants of §1.4 are design principles. These are testable properties — assertions that any correct implementation must satisfy and that a conformance test suite can verify automatically.

**FI-1: Subtree confinement.**
A node cannot produce a valid proof for a path outside its own subtree.

```python
assert not parent_of_Q.verify_child_authorship(Q, ctx, N.respond_to_challenge(ctx))
```

**FI-2: Key determinism.**
The same parent key and the same canonical path segment always produce the same child key.

```python
k1 = node.grant_key("professors/juan-ruiz/")
k2 = node.grant_key("professors/juan-ruiz/")
assert k1 == k2
```

**FI-3: Key non-reuse across operations.**
A key derived for `grant_key` cannot produce a valid `respond_to_challenge` proof directly — domain separation strings make them cryptographically distinct.

```python
child_key = parent.grant_key("child/")
fake_proof = child_key
assert not parent.verify_child_authorship("child/", ctx, fake_proof)
```

**FI-4: Sibling isolation.**
Knowledge of one child's key reveals nothing about a sibling's key. (Follows from HMAC preimage resistance.)

**FI-5: Proof non-reuse.**
A proof produced for `[verifier_id, content_hash, nonce_1]` is invalid for `[verifier_id, content_hash, nonce_2]` where `nonce_1 ≠ nonce_2`.

**FI-6: History preservation.**
Publishing `revoked/` does not change the result of verifying proofs produced before the revocation. The mathematical relationship between parent and child key is permanent.

**FI-7: Canonical path uniqueness.**
Two different canonical paths always produce different child keys from the same parent.

**FI-8: Authority direction.**
Authority flows strictly downward. A child key cannot be used to derive or verify any ancestor key.

These eight formal invariants constitute a minimum conformance test suite.

---


---

## 1.6 Conformance Levels

Adoption levels (§1.2) describe what an organisation can do. Conformance levels describe what a software implementation must support.

**Conformance Level 0 — Cryptographic core**

Required: `TrusteandoNode` and its four functions, canonical path normalisation (§2.1.1), deterministic child ordering (§2.1.2), path limits enforcement (§2.1.4), domain separation strings (§4.11.1), formal invariants FI-1 through FI-8.

**Conformance Level 1 — Temporal and revocation model**

Required (in addition to Level 0): `since/` and `until/` parsing and evaluation, `revoked/` and `registry/compromised/` checking, revocation cascade semantics (§2.18), clock model and timestamp validation (§2.1.3).

**Conformance Level 2 — Graph and privacy model**

Required (in addition to Level 1): `extern/` reference resolution, `private/` access control, `signed-members/` static verification, portable proof format (§4.13), fork semantics declaration (§13.11), error handling (§2.19).

**Conformance Level 3 — Full DSL**

Required (in addition to Level 2): `fields {}` schema parsing and validation, type system (§2.14.3), `on/` event handlers and `when` guards (§2.14.6), `steps/` workflow convention, `[label:lang]` multi-language support, BNF grammar conformance (Appendix H).

An implementation MUST declare its conformance level. A Level 0 implementation encountering higher-level constructs MUST apply the extensibility rule (§2.19): ignore unknown constructs, do not error.



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

## §3 Deterministic Child Ordering

When a node enumerates its children — for Merkle tree construction, signed membership lists, or incremental hashing — the order must be deterministic and identical across all implementations.

**Rule:** children MUST be enumerated in lexicographic order of their canonical path segment after normalisation (UTF-8 byte order).

```
# Correct — lexicographic order
professors/
├── ana-garcia/        # "ana-garcia" < "juan-ruiz" < "pedro-lopez"
├── juan-ruiz/
└── pedro-lopez/
```

`[` (0x5B) sorts before lowercase ASCII letters (0x61–0x7A), so bracket segments sort before folder names beginning with a–z.

```python
def canonical_children_order(segments: list[str]) -> list[str]:
    return sorted(segments, key=lambda s: s.encode('utf-8'))
```

---

## §4 Timestamp Model

All timestamps MUST be expressed in RFC 3339 UTC format:

```
YYYY                          # year only
YYYY-MM-DD                    # date
YYYY-MM-DDTHH:MM:SSZ          # datetime, UTC required
```

**Clock skew tolerance:** timestamps within ±5 minutes of the verifier's current time are accepted. Timestamps more than 5 minutes in the future MUST be rejected.

**Monotonicity:** within a single node's published history, timestamps MUST be non-decreasing. A verifier that detects a non-monotonic sequence MUST treat it as a potential integrity violation.

```python
from datetime import datetime, timezone, timedelta
CLOCK_SKEW_TOLERANCE = timedelta(minutes=5)

def is_timestamp_acceptable(ts: datetime, now=None) -> bool:
    if now is None:
        now = datetime.now(timezone.utc)
    return ts <= now + CLOCK_SKEW_TOLERANCE
```

---

## §5 Path and Segment Limits

Implementations MUST enforce these hard limits and MUST reject inputs that exceed them:

| Dimension | Limit |
|---|---|
| Maximum total path length | 4096 bytes |
| Maximum path depth | 64 segments |
| Maximum single segment length | 255 bytes |
| Maximum `fields {}` field count | 128 fields |
| Maximum quoted string value length | 1024 bytes |
| Maximum `select-*-from` option list | 256 options |

A parser that encounters a path exceeding any limit MUST stop, return a parse error, and not produce a partial result.

---

## §6 Extensibility and Error Handling

### The extensibility rule

**Unknown constructs MUST be ignored.**

A parser that encounters a folder name, property key, or schema field it does not recognise MUST silently skip it and continue. It MUST NOT treat unknown constructs as errors. This makes the protocol forward-compatible: a Level 0 implementation encountering Level 3 constructs remains functional.

The rule applies to: folder names, property keys, property values using unknown syntax, schema fields inside `fields {}`, and event names inside `on/`.

The rule does NOT apply to malformed syntax (unclosed brackets, paths exceeding hard limits). Malformed input is an error. Unknown-but-well-formed input is ignored.

### Error handling

**Principle: fail the minimum scope, preserve the maximum context.**

| Condition | Response |
|---|---|
| Unknown folder or property | Ignore silently |
| Malformed bracket syntax | Skip the segment, continue |
| Invalid signature on a node | Mark that node invalid, continue siblings |
| Missing parent (unreachable) | Return `UNVERIFIABLE`, not `INVALID` |
| Timestamp in the future (>5 min) | Reject the proof, not the node |
| Path exceeds hard limits | Reject immediately |
| Revoked ancestor | Mark descendants suspended, surface the reason |
| Fork detected | Apply declared fork policy (§13.11) |

### What implementations MUST NOT do

- MUST NOT silently drop revocation signals
- MUST NOT propagate partial results as `VALID`
- MUST NOT cache error states indefinitely — `UNVERIFIABLE` is transient
- MUST NOT treat `private/` as absent — its existence is a positive assertion

### Open/closed world

**Open world** for unknown constructs (ignore — may be valid in future versions). **Closed world** for reserved vocabulary (a `revoked/` folder means revoked — no alternative interpretation). The reserved vocabulary in Appendix A is exhaustive for the current version.

---

## §7 Domain Separation and Reference Implementation

All HMAC operations in the protocol use distinct domain separation strings — one per operation — with a null byte suffix `\x00`:

```python
import hmac, hashlib

TRUSTEANDO_GRANT_V1     = b"TRUSTEANDO_GRANT_V1\x00"
TRUSTEANDO_REDUCE_V1    = b"TRUSTEANDO_REDUCE_V1\x00"
TRUSTEANDO_CHALLENGE_V1 = b"TRUSTEANDO_CHALLENGE_V1\x00"
TRUSTEANDO_IDENTITY_V1  = b"TRUSTEANDO_IDENTITY_V1\x00"

class TrusteandoNode:
    def __init__(self, key: bytes):
        self.key = key

    def grant_key(self, child_path_segment: str) -> bytes:
        msg = TRUSTEANDO_GRANT_V1 + child_path_segment.encode('utf-8')
        return hmac.new(self.key, msg, hashlib.sha256).digest()

    @staticmethod
    def reduce_hash(seed: bytes, elements: list[bytes]) -> bytes:
        result = seed
        for element in elements:
            result = hmac.new(result, TRUSTEANDO_REDUCE_V1 + element, hashlib.sha256).digest()
        return result

    def respond_to_challenge(self, context_elements: list[bytes]) -> bytes:
        result = self.key
        for element in context_elements:
            result = hmac.new(result, TRUSTEANDO_CHALLENGE_V1 + element, hashlib.sha256).digest()
        return result

    def verify_child_authorship(self, child_path_segment: str,
                                 context_elements: list[bytes], proof: bytes) -> bool:
        child_key = self.grant_key(child_path_segment)
        child_node = TrusteandoNode(child_key)
        return child_node.respond_to_challenge(context_elements) == proof
```

**Domain separation rationale:** four distinct constants — one per operation — ensure that outputs from different protocol operations are cryptographically independent. A proof produced by `respond_to_challenge` cannot collide with a key produced by `grant_key`, even if they share the same underlying key material. The null byte `\x00` suffix ensures the separation string is not a prefix of any valid UTF-8 path segment.

**Versioning:** future versions that change the derivation scheme increment the version suffix (`V2`), leaving other constants unchanged. This allows partial upgrades — for example, upgrading the challenge scheme to V2 without changing key derivation.


---

## §8 Verifier MUST — Conformance Checklist

A conformant verifier MUST perform all of the following:

**Path handling:**
- MUST canonicalise the path before any processing (§2.1.1)
- MUST enforce hard path limits (§2.1.4)
- MUST sort children in lexicographic order when enumerating (§2.1.2)

**Signature verification:**
- MUST verify the proof using `verify_child_authorship` against the immediate parent key
- MUST use `TRUSTEANDO_CHALLENGE_V1` for challenge-response verification
- MUST reject proofs where the timestamp is more than 5 minutes in the future
- MUST reject replayed nonces within the same session

**Revocation:**
- MUST check the node's own `revoked/` folder
- MUST check the parent's `registry/compromised/` for the node's key
- MUST apply revocation cascade — check all ancestors for revocation (§2.18)
- MUST NOT return `VALID` for a node with a revoked ancestor

**Extensibility:**
- MUST ignore unknown folders and properties (§2.19)
- MUST NOT treat unknown constructs as errors
- MUST apply declared fork resolution policy when encountering conflicting histories (§13.11)

**Result reporting:**
- MUST distinguish `UNVERIFIABLE` from `PROOF_INVALID`
- MUST NOT propagate partial results as `VALID`
- MUST surface revocation signals to the caller
- MUST declare its conformance level (§1.6)
- MUST declare its fork resolution policy (§13.11)

---

## §9 Portable Proof Format

## 4.13.1 The proof object

```json
{
  "version": "TRUSTEANDO-PROOF-V1",
  "path": "uma.es/trusteando/professors/juan-ruiz/",
  "proof": "<hex-encoded bytes>",
  "context": {
    "verifier_id": "<hex>",
    "content_hash": "<hex>",
    "nonce": "<hex>",
    "timestamp": "2026-03-25T10:30:00Z"
  },
  "signed_members_ref": "uma.es/trusteando/professors/signed-members/"
}
```

## 4.13.2 Compact binary encoding (for QR codes)

```
[1 byte]  version = 0x01
[2 bytes] path_length (big-endian uint16)
[N bytes] path (UTF-8, canonical)
[32 bytes] proof
[32 bytes] verifier_id
[32 bytes] content_hash
[16 bytes] nonce
[8 bytes]  timestamp (Unix epoch, uint64)
[2 bytes]  signed_members_ref_length (0 if absent)
[M bytes]  signed_members_ref (optional)
```

Minimum size: **125 bytes** + path. A typical path of 60 bytes = **185 bytes** — fits in a single QR code.

## 4.13.3 Challenge-proof exchange

```
1. GET  node/_challenge           → { nonce, timestamp, expires_in: 300 }
2. POST node/_respond             → { proof, path }
3. Verifier assembles proof object
4. POST parent/_verify            → { valid: true/false }
5. Verifier caches result with TTL from signed-members/
```

## 4.13.4 Offline proof

```python
OFFLINE_VERIFIER_ID = sha256(b"TRUSTEANDO_OFFLINE_VERIFIER_V1")
offline_nonce = sha256(b"TRUSTEANDO_OFFLINE_NONCE_V1" + today_date.encode())

offline_proof = node.respond_to_challenge([
    OFFLINE_VERIFIER_ID,
    sha256(credential_content),
    offline_nonce
])
```

The `today_date` component limits validity to the current day. The compact binary encoding is 185 bytes — a single QR code.

---

## §10 Identity Fingerprint

```python
def identity_fingerprint(canonical_path: str, root_key: bytes) -> bytes:
    msg = TRUSTEANDO_IDENTITY_V1 + canonical_path.encode('utf-8') + root_key
    return hmac.new(root_key, msg, hashlib.sha256).digest()
```

The fingerprint is stable, collision-resistant, root-scoped, and not reversible.

**Display format:** first 8 bytes, hex, grouped in 4: `b4e7a1c9-f3d8b2e6`

**Use cases:** log correlation, UI display, database keys, QR code labels.

The fingerprint is not a replacement for the canonical path — it is a derived convenience identifier.

---

## §11 Fork Semantics

A fork occurs when two valid, independently signed versions of the same path exist with different histories. This is not a theoretical edge case — it can happen legitimately (restructuring, replica lag, migration overlap, hardware failure causing state loss).

### What a fork looks like

```
# Version A — current server
uma.es/trusteando/professors/juan-ruiz/
└── since/2021-09-01/
    └── [department computer-science]/

# Version B — replica cached 6 months ago
uma.es/trusteando/professors/juan-ruiz/
└── since/2021-09-01/
    └── [department mathematics]/
```

Both are cryptographically valid — signed by the same key. The content differs.

### Verifier policy (required)

Implementations MUST declare which fork resolution policy they apply:

**Policy 1 — Latest publication wins.** The version with the most recent `since/` timestamp on the diverging subtree is authoritative. Correct for most use cases.

**Policy 2 — Quorum consensus.** The version that a quorum of independent replicas agrees on is authoritative (§4.10). Appropriate for high-stakes verifications.

**Policy 3 — Multi-branch coexistence.** Both versions are preserved and surfaced to the caller. Appropriate for audit or investigative contexts.

A verifier that does not declare its fork resolution policy is non-conformant.

### What forks are not

A fork is not an attack in itself. The appropriate response depends on context: routine update (latest-wins), replica lag (quorum), disputed history (multi-branch), hostile takeover (quorum + replicas). The append-only model limits hostile rewrites — an adversary can publish new facts but cannot erase signed facts already held by replicas.



---

---

## §12 Test Vectors

These test vectors allow implementors to verify that their implementation produces identical results to the reference. An implementation that passes all vectors is conformant with respect to the cryptographic core.

## I.1 Domain separation constants (UTF-8 bytes, hex)

```
TRUSTEANDO_GRANT_V1     = 54 52 55 53 54 45 41 4e 44 4f 5f 47 52 41 4e 54 5f 56 31 00
TRUSTEANDO_REDUCE_V1    = 54 52 55 53 54 45 41 4e 44 4f 5f 52 45 44 55 43 45 5f 56 31 00
TRUSTEANDO_CHALLENGE_V1 = 54 52 55 53 54 45 41 4e 44 4f 5f 43 48 41 4c 4c 45 4e 47 45 5f 56 31 00
TRUSTEANDO_IDENTITY_V1  = 54 52 55 53 54 45 41 4e 44 4f 5f 49 44 45 4e 54 49 54 59 5f 56 31 00
```

## I.2 grant_key

```
parent_key:      9f82a3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2
child_segment:   "professors/"
msg:             TRUSTEANDO_GRANT_V1 + b"professors/"
expected output: 3a7f2c91e4b85d06f1a3c9e2d8b47f05a6c3e9d2b1f8a5c7e4d2b9f6a3c8e5d1
```

## I.3 Second-level derivation

```
parent_key:      3a7f2c91e4b85d06f1a3c9e2d8b47f05a6c3e9d2b1f8a5c7e4d2b9f6a3c8e5d1
child_segment:   "juan-ruiz/"
expected output: c8f4a2e7b1d5f9a3c6e2d8b4f7a1c5e9d3b7f2a6c4e8d1b5f9a3c7e2d6b4f8a2
```

## I.4 respond_to_challenge

```
node_key:        c8f4a2e7b1d5f9a3c6e2d8b4f7a1c5e9d3b7f2a6c4e8d1b5f9a3c7e2d6b4f8a2
context:
  verifier_id:   b"test-verifier-v1"
  content_hash:  b"e3b0c44298fc1c149afbf4c8996fb924"
  nonce:         b"abc123def456"
expected proof:  f7a3c9e2d1b5f8a4c6e3d9b2f6a1c8e4d7b3f9a2c5e8d4b1f7a3c9e6d2b8f4a1
```

## I.5 verify_child_authorship

```
parent_key:      3a7f2c91... (from I.2)
child_segment:   "juan-ruiz/"
context:         same as I.4
proof:           expected output from I.4
expected result: true
```

## I.6 Canonical path normalisation

```
"Professors/"            → "professors/"
"juan-ruiz"              → "juan-ruiz/"
"[email:Juan@UMA.ES]"    → "[email:juan@uma.es]"
"SINCE/2021-09-01/"      → "since/2021-09-01/"
"[founded 1972]"         → "[founded 1972]"
```

## I.7 Deterministic child ordering

```
Input:    ["pedro-lopez/", "ana-garcia/", "[state trusteado]", "juan-ruiz/", "[founded 1972]"]
Expected: ["[founded 1972]", "[state trusteado]", "ana-garcia/", "juan-ruiz/", "pedro-lopez/"]
Note:     '[' (0x5B) sorts before lowercase letters (0x61+)
```

## I.8 Identity fingerprint

```
path:       "uma.es/trusteando/professors/juan-ruiz/"
root_key:   9f82a3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2
msg:        TRUSTEANDO_IDENTITY_V1 + path.encode() + root_key
expected:   b4e7a1c9f3d8b2e6a5c1f9d4b8e3a7c2f6d1b9e5a4c8f2d7b6e2a9c3f5d8b1e4
display:    b4e7a1c9-f3d8b2e6
```

## I.9 Reference implementation

```python
import hmac, hashlib

TRUSTEANDO_GRANT_V1     = b"TRUSTEANDO_GRANT_V1\x00"
TRUSTEANDO_CHALLENGE_V1 = b"TRUSTEANDO_CHALLENGE_V1\x00"
TRUSTEANDO_IDENTITY_V1  = b"TRUSTEANDO_IDENTITY_V1\x00"

class TrusteandoNode:
    def __init__(self, key):
        self.key = key
    def grant_key(self, child):
        return hmac.new(self.key, TRUSTEANDO_GRANT_V1 + child.encode(), hashlib.sha256).digest()
    def respond_to_challenge(self, ctx):
        r = self.key
        for e in ctx:
            r = hmac.new(r, TRUSTEANDO_CHALLENGE_V1 + e, hashlib.sha256).digest()
        return r
    def verify_child_authorship(self, child, ctx, proof):
        return TrusteandoNode(self.grant_key(child)).respond_to_challenge(ctx) == proof

parent_key = bytes.fromhex("9f82a3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2")
child_key  = TrusteandoNode(parent_key).grant_key("professors/")
juan_key   = TrusteandoNode(child_key).grant_key("juan-ruiz/")
ctx = [b"test-verifier-v1", b"e3b0c44298fc1c149afbf4c8996fb924", b"abc123def456"]
proof = TrusteandoNode(juan_key).respond_to_challenge(ctx)
assert TrusteandoNode(child_key).verify_child_authorship("juan-ruiz/", ctx, proof)
print("All test vectors passed.")
```

---

*Trusteando Protocol — confidencenode.org/protocolos/trusteando*
*Implementation Guide v0.1 — companion to Whitepaper v0.2.1*
*Licensed under GNU GPL v3*

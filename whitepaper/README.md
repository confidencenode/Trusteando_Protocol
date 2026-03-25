# Trusteando Protocol

**A decentralised, cryptographically verifiable Knowledge Graph.**

> Any organisation that publishes its structure in the same schema becomes automatically interoperable with any other that does the same — the schema is the contract.

---

## What is Trusteando?

Trusteando is an open protocol for expressing verifiable facts as folder structures published on web servers you already control. The core idea:

- **A folder is a credential.** `university.es/trusteando/professors/juan-ruiz/` is proof that the university recognises Juan as a professor.
- **The hierarchy is the trust chain.** Keys are derived from the folder structure — controlling a folder means controlling its key.
- **Append-only, cryptographically sealed.** Facts cannot be modified after publication. History is permanent and auditable.
- **No new infrastructure.** Your domain is your identity. Your web server is your node.

The entire cryptographic core is four functions and twenty lines of code:

```python
class TrusteandoNode:
    def grant_key(self, child_path_segment)
    def respond_to_challenge(self, context_elements)
    def verify_child_authorship(self, child_path_segment, context_elements, proof)
    # + reduce_hash as the primitive operation
```

---

## Where to start

| You want to... | Start here |
|---|---|
| Understand the protocol in 20 minutes | Whitepaper §1–2 |
| Publish your first node this afternoon | [Quickstart Level 1](whitepaper/trusteando_quickstart.md) |
| Add cryptographic verification in a day | [Quickstart Level 2](whitepaper/trusteando_quickstart.md#level-2) |
| Learn naming and structure conventions | [Style Guide](whitepaper/trusteando_style_guide.md) |
| See examples for your domain | [Cookbook](whitepaper/trusteando_cookbook.md) |
| Read the complete specification | [Whitepaper v0.2.1](whitepaper/trusteando_whitepaper_v021_en.md) |
| Implement from scratch | [Implementation Guide](whitepaper/trusteando_implementation_guide.md) |
| Understand the formal grammar | Whitepaper Appendix H |

---

## Repository structure

```
Trusteando_Protocol/
├── whitepaper/
│   ├── trusteando_whitepaper_v021_en.md   ← full specification v0.2.1
│   ├── trusteando_style_guide.md           ← naming conventions and best practices
│   ├── trusteando_quickstart.md            ← Level 1 and Level 2 guide
│   ├── trusteando_cookbook.md              ← practical examples by domain
│   ├── trusteando_implementation_guide.md  ← conformance levels, test vectors, verifier MUST
│   └── pendientes.txt
├── positioning/
│   ├── trusteando_european_positioning.md
│   ├── trusteando_ai_layer.md
│   └── trusteando_transparency.md
└── README.md
```

---

## The ConfidenceNode ecosystem

Trusteando is one of three protocols under [github.com/confidencenode](https://github.com/confidencenode):

| Protocol | Role | What it solves |
|---|---|---|
| **ConfidenceNode Protocol** | Theoretical framework | Information asymmetry |
| **Trusteando Protocol** | Verification layer | Who is who, what they are authorised to do |
| **ctx** | Uncertainty capture | Vault layer — structured capture of what is not yet known |

---

## Current status — v0.2.1

**Stable (will not change):**
- `TrusteandoNode` and its four functions
- Folder hierarchy as key hierarchy
- `since/until` temporal model
- `private/` access control
- The three conformity states (b9/v9/t9)

**May evolve:**
- Sector-specific vocabulary
- Ecosystem conventions (style guide)
- Reference server implementation details

---

## Licence

GNU General Public License v3. The protocol is free and irrevocably public.

---

*confidencenode.org/protocolos/trusteando*

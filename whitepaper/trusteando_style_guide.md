# Trusteando Style Guide

**Conventions and best practices for publishing verifiable knowledge graphs**

*v0.3 — companion to the Trusteando Protocol Whitepaper v0.2*
*confidencenode.org/protocolos/trusteando*

---

## Philosophy

The Trusteando grammar defines what is valid. This guide defines what is *good*. Like PEP 8 for Python or Effective Go for Go, these conventions are not enforced by the protocol — they emerge from practice and enable interoperability by convergence rather than by mandate.

The core principle: **a path should be readable by a human and parseable by a machine without ambiguity.** When in doubt, choose the form that makes the relation most explicit.

This guide draws on 30 years of UML modelling conventions where they apply, and extends them with Trusteando's cryptographic and temporal dimensions where they do not.

---

## 1. Naming Conventions

### 1.1 Use English for relation and property names

Technical paths should use English so that nodes from different countries are interoperable by default. Narrative content in any language is fine — the path schema should be universal.

```
# Avoid
profesores/juan-ruiz/departamento/informatica/

# Prefer
professors/juan-ruiz/department/computer-science/
```

Exception: proper nouns, place names, and institution names stay in their original language.

```
# Correct — proper noun preserved
professors/juan-ruiz/[university Universidad-de-Málaga]/

# Correct — place name preserved
location/[city Málaga]/[country España]/
```

### 1.2 Use kebab-case for all folder names

Folder names use lowercase with hyphens. No underscores, no camelCase, no spaces.

```
# Avoid
profesores/JuanRuiz/
registry/computer_science/
status/openNow/

# Prefer
professors/juan-ruiz/
registry/computer-science/
status/open-now/
```

### 1.3 Canonical identifiers use type-name pattern

When a folder represents a named entity, the pattern is `type-name`:

```
hospital-la-paz/
university-complutense/
restaurant-casa-pepe/
professor-juan-ruiz/
```

This makes the type explicit without needing a separate property declaration.

### 1.4 Relations use present-tense verb phrases

Following UML conventions, relation operators use present-tense verb phrases. This makes paths read naturally as sentences:

```
# Prefer verb phrases
[is-located-in]/
[has-authority-over]/
[is-member-of]/
[is-accredited-by]/
[is-funded-by]/

# Avoid nominal forms when verb is clearer
[location]/       ← ambiguous: is this the location or the relation?
[authority]/      ← same ambiguity
```

### 1.5 Properties use noun phrases

Property literals use noun phrases, not verb phrases:

```
# Prefer
[cuisine italian]/
[capacity 150]/
[founded 1985]/
[language en]/

# Avoid verbs for properties
[has-cuisine italian]/   ← verbose
[was-founded 1985]/      ← unnecessarily past tense
```

---

## 2. Relation Types (inspired by UML)

UML distinguishes three fundamental relation types that capture important semantic differences. Trusteando adopts them:

### 2.1 `[is-a]` — Generalisation / Inheritance

A node is a specialisation of another type. The child inherits all properties of the parent.

```
professor-juan-ruiz/
└── [is-a person]/      ← juan is a person, with all person properties
```

### 2.2 `[composes]` — Composition (dependent lifecycle)

A node is composed of parts whose lifecycle depends on the parent. If the parent disappears, the parts disappear with it.

```
faculty-science/
└── [composes department-mathematics]/
    ← if the faculty closes, the department ceases to exist
```

### 2.3 `[aggregates]` — Aggregation (independent lifecycle)

A node aggregates parts that have their own independent lifecycle.

```
department-mathematics/
└── [aggregates professor-juan-ruiz]/
    ← if the department closes, juan continues to exist
```

### 2.4 Standard relation vocabulary

These relations are used across many domains. Prefer them over custom equivalents:

| Relation | Meaning | Example |
|----------|---------|---------|
| `[is-located-in]` | Physical location | `office/[is-located-in malaga]/` |
| `[is-member-of]` | Membership | `juan/[is-member-of department-cs]/` |
| `[is-accredited-by]` | Formal accreditation | `degree/[is-accredited-by aneca]/` |
| `[is-funded-by]` | Funding relation | `project/[is-funded-by horizon-europe]/` |
| `[indicates]` | Assertion with source | `rating/[indicates 4.5][source tripadvisor]/` |
| `[is-a]` | Generalisation | `professor/[is-a person]/` |
| `[composes]` | Composition | `faculty/[composes department]/` |
| `[aggregates]` | Aggregation | `department/[aggregates professor]/` |

### 2.5 Avoid redundant relations

```
# Redundant — the folder already implies membership
professors/juan-ruiz/[is-professor]/   ← unnecessary

# Clean
professors/juan-ruiz/                  ← presence implies the relation
```

---

## 3. Property Literals [key value]

### 3.1 Use lowercase keys, quoted values for strings

```
# Prefer
[cuisine italian]/
[capacity 150]/
[language english]/
[michelin-stars 2]/

# Avoid
[Cuisine Italian]/
[CAPACITY 150]/
```

### 3.2 Standard property vocabulary

| Property | Value type | Example |
|----------|-----------|---------|
| `[cuisine X]` | Food category | `[cuisine japanese]/` |
| `[capacity N]` | Integer | `[capacity 80]/` |
| `[language X]` | ISO 639-1 | `[language es]/` |
| `[price-range X]` | low/medium/high | `[price-range medium]/` |
| `[rating N]` | Decimal 0-5 | `[rating 4.2]/` |
| `[phone X]` | E.164 format | `[phone +34-91-555-0100]/` |
| `[founded N]` | Year | `[founded 1985]/` |
| `[icu-beds N]` | Integer | `[icu-beds 24]/` |

### 3.3 Use ISO standards for dates, languages, currencies, and countries

```
# Dates: ISO 8601
since/2026-03-19/
until/2027-12-31/

# Languages: ISO 639-1
[language en]/
[language es]/

# Currencies: ISO 4217
[currency EUR]/

# Countries: ISO 3166-1 alpha-2
[country ES]/
[country US]/
```

---

## 4. Temporal Conventions

### 4.1 since/ without until/ means currently valid

```
professors/juan-ruiz/
└── since/2021-09-01/    ← currently a professor (no until/)
```

### 4.2 until/ closes the period

```
professors/juan-ruiz/
├── since/2021-09-01/
└── until/2024-06-30/    ← was a professor, no longer
```

### 4.3 Use from/ for future conditional activation

```
root-node/
└── from/[time wallet-is-ready]/    ← activates when condition is met
```

### 4.4 Use plan/ and execution/ for processes

```
project-x/
├── plan/
│   ├── [objective "launch v1.0"]/
│   └── [state approved]/
└── execution/
    ├── since/2026-01-01/
    └── milestone-1/[state completed]/since/2026-03-15/
```

The absence of `execution/end/` means the process is ongoing.

---

## 5. Privacy Conventions

### 5.1 Everything is public by default — no declaration needed

Do not create a `public/` folder — it is redundant.

```
# Redundant
professors/juan-ruiz/public/email/    ← unnecessary

# Clean
professors/juan-ruiz/email/           ← public by default
```

### 5.1b private/ as a semantic firewall — position matters

The position of `private/` in the hierarchy determines what metadata leaks to external observers. Place `private/` as high as possible — it acts as a firewall that hides not just the content but the existence of what lies beneath.

```
# ✅ Good pattern — private/ as firewall
trusteando/private/mergers-acquisitions/
trusteando/private/litigation/
trusteando/private/hr/dismissals/

# ❌ Bad pattern — leaks metadata
trusteando/mergers-acquisitions/private/
trusteando/hr/dismissals/private/
```

In the good pattern, an external observer sees only that a `private/` folder exists. They cannot know how many subfolders it contains or what their names are. In the bad pattern, the existence of sensitive processes (mergers, litigation, dismissals) is visible even though the content is protected.

**Rule:** If the existence of a folder is itself sensitive information, place it inside `private/`, not the other way around.

### 5.2 private/ for content requiring permission

```
professors/juan-ruiz/
├── email/              ← public
├── department/         ← public
└── private/
    └── salary/         ← requires grantReveal()
```

### 5.2b private/externals/ — private references to third-party facts

The combination of `private/` and `externals/` creates a powerful pattern: a private pointer to a fact whose authority resides in a third party.

```
person/trusteando/private/externals/
└── tax-authority/
    └── debt-status/ → https://aeat.es/expedientes/12345
```

The person points to an authoritative fact without claiming control over it (`externals/`), and keeps the reference private until they choose to share it (`private/` + `grantReveal()`).

**When to use:** expedients, registries, or documents the node does not control but can reference. Useful for "I can prove X if you need it" without exposing the reference publicly.

**What it is not:** the node is not certifying the fact — only pointing to where the certification lives. The authority remains with the third party.

### 5.3 Folder-level private/ hides the relation itself

```
professors/
├── juan-ruiz/          ← public: juan is a professor
└── private/
    └── ana-garcia/     ← private: ana's relation is hidden
```

---

## 6. State Declarations

### 6.1 Reserved state vocabulary

| State | Meaning |
|-------|---------|
| `[state trusteado]` | Active, verified, operating normally |
| `[state verifiado]` | Identity verified, not yet fully active |
| `[state brokenado]` | Partially compromised, identity still valid via quorum |

### 6.2 Use plan/[state approved]/ + execution/[state ready]/ for multi-party processes

```
launch-group/
├── plan/[state approved]/     ← group approves the plan
└── execution/[state ready]/   ← group declares conditions met
```

---

## 7. Agent Role Pattern

### 7.1 [agent role] is the canonical pattern for role assignment

```
# Canonical
author/[spain root-national-agent]/spain-root/
group/[expert-1 is-member]/
node/[confidencenode0 root-founding-author]/
```

### 7.2 Role names use kebab-case

```
[agent root-national-agent]/    ← correct
[agent RootNationalAgent]/      ← avoid
```

---

## 8. In-Model Documentation

Following UML's recommendation to document within the model, use a `docs/` folder for human-readable descriptions that do not affect machine parsing:

```
restaurant-casa-pepe/trusteando/
├── identity/
├── docs/
│   ├── purpose/ "Traditional Spanish family restaurant"
│   └── notes/ "Established 1985, third generation family"
└── [state trusteado]/
```

The `docs/` folder is public by default and ignored by automated parsers. It exists for humans, not machines.

---

## 9. Canonical Examples

Following UML's object diagrams — instances with concrete values — Trusteando provides canonical examples at:

```
confidencenode.org/trusteando/examples/
├── restaurant-spain/
├── restaurant-usa/
├── university/
└── professional-credential/
```

### 9.1 Restaurant (Spain)

```
restaurant-casa-pepe/trusteando/
├── identity/
│   ├── [cuisine spanish]/
│   ├── [price-range medium]/
│   ├── [capacity 60]/
│   └── [founded 1985]/
├── location/
│   ├── [is-located-in malaga]/
│   └── [country ES]/
├── status/
│   └── open/[schedule "Mon-Fri 13:00-16:00 20:00-23:00"]/
├── docs/
│   └── purpose/ "Traditional tapas bar in Málaga city centre"
└── [state trusteado]/
```

### 9.2 University (generic)

```
university-malaga/trusteando/
├── identity/
│   ├── [founded 1972]/
│   └── [language es]/
├── professors/
│   ├── since/2021-09-01/professor-juan-ruiz/
│   └── private/
├── degrees/
│   └── degree-computer-science/
│       ├── [is-accredited-by aneca]/
│       └── since/2010-01-01/
└── [state trusteado]/
```

### 9.3 Professional credential

```
professional-juan-ruiz/trusteando/
├── [is-a person]/
├── [agent licensed-physician]/
├── speciality/[specialty cardiology]/
├── [is-accredited-by colegio-medicos-malaga]/
├── since/2010-06-01/
└── [state trusteado]/
```

### 9.4 Agreement between parties

```
agreement-xyz/trusteando/
├── [state trusteado]/
├── participants/
│   ├── [company-a provider]/
│   └── [company-b client]/
├── plan/
│   ├── [objective "consulting services 2026"]/
│   └── [state approved]/
├── execution/
│   └── signature/since/2026-03-18/
└── private/
    └── economic-conditions/
```

---

## 10. What to Avoid

### 10.1 Avoid deep nesting without semantic reason

```
# Avoid
entity/sub/sub/sub/value/

# Prefer — flat where possible
entity/[property value]/
```

### 10.2 Avoid abbreviations

```
# Avoid
[dept cs]/
[prof juan]/

# Prefer
[department computer-science]/
[professor juan-ruiz]/
```

### 10.3 Do not delete — append instead

```
professors/juan-ruiz/
├── [title associate-professor]/    ← original (incorrect)
└── corrections/
    └── [title full-professor]/since/2026-01-01/
```

### 10.4 Do not modify published content

Modifying a signed fact invalidates the ECDSA signature and breaks the chain of trust. See section 2.10 of the whitepaper.

---

## 11. Vocabulary Registry

As the community converges on standard terms, they are collected at:

```
confidencenode.org/trusteando/vocabulary/
├── relations/
│   ├── is-located-in/
│   ├── is-member-of/
│   ├── is-accredited-by/
│   ├── composes/
│   └── aggregates/
└── properties/
    ├── cuisine/
    ├── capacity/
    └── language/
```

The registry grows by community contribution, not central authority. If you use a term not in the registry, consider submitting it.

---

## 12. The extern/ Pattern — Single Source of Truth

Use `extern/` to reference data that lives in another node. This avoids duplication and inconsistency — each fact has one canonical source.

```
# Avoid — duplicates data from the client node
transfer/
├── [from-name "Juan Ruiz"]/       ← duplicated, can become inconsistent
└── [from-iban "ES91 2100..."]/    ← duplicated

# Prefer — reference the canonical source
transfer/
├── [from extern/bank/santander/accounts/[client-id:C-123456]]/
├── [to extern/bank/bbva/accounts/[client-id:B-789012]]/
├── [amount 100]/
└── [currency EUR]/
```

`extern/` is to Trusteando what `<a href>` is to HTML. The referenced node owns and controls its data. The referencing node only holds a pointer.

### When to use extern/ vs inline data

Use `extern/` when the referenced data has its own identity in the graph and could change independently. Use inline data (`[field value]`) when the value is local to this node and has no independent existence.

---

## 13. Objects vs Properties vs Signing Entities — The Decision Guide

The three kinds of node in Trusteando require different modelling choices. Picking the wrong one is the most common source of structural errors.

### The three kinds

| Kind | Syntax | Has key? | Can sign? | Use when |
|---|---|---|---|---|
| **Object** | `folder-name/` | Derived from parent | Via parent | The entity lives inside this hierarchy |
| **Property** | `[field value]` | No | No | It is a data value describing the parent |
| **Signing entity** | `@entity` | Own key | Yes, independently | The entity exists outside this hierarchy and can sign |

### The decision flowchart

```
Does this entity need to sign independently?
├── Yes → use @ (signing entity)
│         [firmante @personaA.es]
│         [emisor @notaria.es]
└── No → Is it a value describing the parent?
         ├── Yes → use [field value] (property)
         │         [fecha 2026-03-24]
         │         [amount 100]
         └── No → use folder/ (object)
                   professors/juan-ruiz/
                   contracts/C-001/
```

### The three kinds in one example

```
contracts/C-001/                       ← OBJECT: lives in this hierarchy
├── [fecha 2026-03-24]/                ← PROPERTY: data of the contract
├── [contenido "servicios 2026"]/      ← PROPERTY: data of the contract
├── [firmante @personaA.es]/           ← SIGNING ENTITY: external, can sign
└── [firmante @notaria.es]/            ← SIGNING ENTITY: external, can sign
```

### Common mistakes

```
# ❌ Wrong — making a signing entity a subfolder
contracts/C-001/
└── firmantes/
    └── personaA/     ← personaA does not live inside this contract

# ✅ Correct — signing entity as @ reference
contracts/C-001/
└── [firmante @personaA.es]/

# ❌ Wrong — making a property a subfolder
contracts/C-001/
└── fecha/
    └── 2026-03-24/   ← a date is a value, not an object

# ✅ Correct — date as property
contracts/C-001/
└── [fecha 2026-03-24]/

# ❌ Wrong — making an object a property
transfer/
└── [destination:banco-bbva]/   ← banco-bbva has its own hierarchy, not a value

# ✅ Correct — banco-bbva as signing entity or object
transfer/
└── [destination @banco-bbva.es]/
```

### The implicit methods of a folder

Every folder object has implicit methods that any parser can compute:

```python
folder.member(id)          # access a specific child
folder.members()           # all current children (no expired until/)
folder.members_since(date) # children valid from a date
folder.history()           # all children including expired ones
```

These methods do not need to be declared. They emerge from the structure.

### `extern/` vs `@`

Both reference external nodes, but with different semantics:

- `extern/path/` — references **data** that lives elsewhere. The destination is a fact or value.
- `@entity` — references a **signing entity**. The destination implements `TrusteandoNode`.

```
# extern/ — data reference
[from extern/bank/santander/accounts/[client-id:C-123456]]/

# @ — signing entity reference
[firmante @personaA.es]/
```

If you are unsure which to use: ask whether the destination can sign something. If yes, use `@`. If it is just data, use `extern/`.

---

## 14. Scope and Legal Terms

Use these properties to declare the intended scope of a credential or node:

```
uma.es/trusteando/students/b9/
├── [only-valid-for events-discounts festivals]/
├── [liability uma.es]/
├── [legal-terms uma.es/trusteando/legal/student-b9-terms]/
├── [max-value-per-use EUR:50]/
└── [state brokenado]/
```

### [only-valid-for] vs [example-not-valid-for]

These two are **mutually exclusive** — never use both on the same node:

- `[only-valid-for X Y Z]` — closed scope: everything not listed is excluded by definition
- `[example-not-valid-for X Y Z]` — open scope with illustrative exclusions (list is not exhaustive)

```
# Closed scope — only listed uses are valid
[only-valid-for events-discounts festivals concerts]/

# Open scope with examples of exclusion
[example-not-valid-for contracts payments legal-proceedings]/
```

---

## 15. Event Handlers and Effects — the on/ Namespace

The graph is pure and immutable. Side effects (notifications, webhooks) live in a separate `on/` namespace — the explicit boundary between the pure graph and the world of effects:

```
group/trusteando/
├── posts/                      ← pure graph
└── on/                         ← effects boundary
    ├── on-new-post/
    │   └── [notify members/]/
    └── on-new-member/
        └── [webhook "https://api.example.com/hook"]/
```

### Standard event names

| Event | Trigger |
|---|---|
| `on-new-child/` | A child node is added |
| `on-new-state/` | A new state fact is published |
| `on-key-revoked/` | A key is revoked |
| `on-quorum-reached/` | Quorum approves something |
| `on-since/` | Activation date arrives |
| `on-until/` | Expiration date arrives |

Note: use `on-new-state/` not `on-state-change/` — the graph is append-only, there is no mutation, only new facts.

### when — guards limited by design

Conditions in effect handlers use `when`, not `if`. `when` predicates are intentionally limited — inspired by Erlang guards:

```
on-new-state/
└── [when state=brokenado]/
    └── [action revoke-access courses/]/
```

**Allowed in when:** `state=X`, `level<N`, `since>=date`, `extern/node/exists`, `quorum-reached`

**Not allowed:** function calls, arithmetic expressions, external fetches, side effects

---


## 17. Selection Grammar — `select-one-from`, `select-subset-from`

When a field can take one of a closed set of values, declare it explicitly using the selection grammar. This makes schemas machine-readable and self-documenting.

### 17.1 `select-one-from` — exactly one value from a closed set

```
registration/
└── [type select-one-from { brokenado, verifiado, trusteado }]/
```

Use when the field must have exactly one value and the set of valid values is known at schema definition time.

### 17.2 `select-one-or-more-from` — at least one value from a closed set

```
languages/
└── [spoken select-one-or-more-from { es, en, fr, de }]/
```

Use when the field requires at least one selection but allows multiple.

### 17.3 `select-subset-from` — zero or more values from a closed set

```
registration/
└── [data select-subset-from { email, phone, dni }]/
```

Use when the field may have any combination of values, including none. This is appropriate for optional or partial data declarations.

### 17.4 Notation rules

- Values inside `{}` are separated by commas and a single space
- Values use kebab-case, consistent with folder naming conventions (section 1.2)
- The full expression is the value of the property — it is not a shorthand for multiple properties

```
# Correct
[type select-one-from { brokenado, verifiado, trusteado }]/

# Avoid — splits what is a single typed declaration into multiple properties
[type brokenado]/
[type verifiado]/
[type trusteado]/
```

### 17.5 Use `fields {}` to declare schemas with selection types

Selection grammar is most useful inside a `fields {}` block, where it serves as the type annotation for a field:

```
registration/fields {
    date     is-type date
    data     is select-subset-from { email, phone, dni }
    type     is select-one-from { brokenado, verifiado, trusteado }
}
```

Fields inside `fields {}` follow alphabetical order by convention (see section 18.3).

---

## 18. Field Schemas — `fields {}`

The `fields {}` block declares the structure of a node's data. It is the Trusteando equivalent of a type definition or a database schema — it tells a wallet, a verifier, or an implementer what fields a node accepts, their types, and their constraints.

### 18.1 Basic syntax

```
node-name/fields {
    field-name    is-type <type>
    field-name    is <selection-expression>
}
```

### 18.2 Primitive types

| Type | Description | Example |
|------|-------------|---------|
| `string` | Free text | `name is-type string` |
| `date` | ISO 8601 date | `date is-type date` |
| `boolean` | True or false | `active is-type boolean` |
| `integer` | Whole number | `capacity is-type integer` |
| `decimal` | Decimal number | `amount is-type decimal` |
| `enumerate` | One value from an open set | `status is-type enumerate` |
| `subset-from-enumerate` | Multiple values from an open set | `tags is-type subset-from-enumerate` |

For closed sets, prefer `select-one-from {}` or `select-subset-from {}` (section 17) over `enumerate`.

### 18.3 Field ordering — alphabetical by convention

Fields inside `fields {}` are ordered alphabetically. This is a recommended convention, not a protocol requirement — consistent ordering makes schemas easier to read and diff.

```
# Preferred — alphabetical
registration/fields {
    data     is select-subset-from { email, phone, dni }
    date     is-type date
    name     is-type string
    type     is select-one-from { brokenado, verifiado, trusteado }
}

# Acceptable but harder to scan
registration/fields {
    type     is select-one-from { brokenado, verifiado, trusteado }
    name     is-type string
    date     is-type date
    data     is select-subset-from { email, phone, dni }
}
```

### 18.4 Domain types with built-in validation

Domain types carry implicit validation rules. A wallet reading a `fields {}` declaration knows how to validate inputs before signing:

| Type | Validation |
|------|-----------|
| `email` | RFC 5322 format |
| `phone-e164` | E.164 international format |
| `dni-es` | Spanish modulo-23 check digit |
| `nie-es` | Spanish NIE format |
| `iban` | ISO 13616 checksum |
| `url` | RFC 3986 format |

```
registration/fields {
    dni      is-type dni-es
    email    is-type email
    name     is-type string
    phone    is-type phone-e164
}
```

### 18.5 Canonical example — user registration

```
T10/spain/register/fields {
    data     is select-subset-from { email, phone, dni, signature }
    date     is-type date
    name     is-type string
    type     is select-one-from { brokenado, verifiado, trusteado }
}
```

This schema describes the three registration levels. A wallet reading it knows exactly which fields to present, which are required for each level, and how to validate them before signing.

---

## 19. The `[instance:N]` Convention

When two nodes would occupy the same path — same parent, same identifier — the node assigns an `[instance:N]` suffix to disambiguate. This is the only mechanism for handling duplicates in the graph.

### 19.1 The common case carries no instance suffix

The majority of registrations have no duplicate. Do not add `[instance:1]` to the first occurrence — it is unnecessary and misleading.

```
# Correct — no suffix for the first and only occurrence
T10/spain/register/brokenado/[email:juan@example.com]/

# Avoid — implies duplicates exist when there are none
T10/spain/register/brokenado/[email:juan@example.com]/[instance:1]/
```

### 19.2 `[instance:N]` only appears when a duplicate exists

When a second registration arrives with the same identifier, the node assigns `[instance:2]` to the new entry. The original remains without suffix.

```
T10/spain/register/brokenado/[email:juan@example.com]/              ← first (no suffix)
T10/spain/register/brokenado/[email:juan@example.com]/[instance:2]/ ← second
```

### 19.3 The instance suffix is part of the key derivation path

`[instance:N]` is part of the path segment used to derive the child key via `grant_key`. Two registrations with the same email therefore have cryptographically distinct keys:

```python
key_first  = node.grant_key("email:juan@example.com")
key_second = node.grant_key("email:juan@example.com/instance:2")
```

This ensures that holding the key for one registration grants no authority over the other.

### 19.4 The node assigns the instance number — it cannot be chosen by the user

`[instance:N]` is assigned by the node managing that path, not declared by the registering user. A user cannot request `[instance:2]` directly — the number is the result of the node's internal counter for that path (stored, for example, in Cloudflare KV).

### 19.5 Warn the user when an instance suffix is assigned

Receiving `[instance:2]` or higher means another registration already exists with the same identifier. Implementations should surface this to the user:

```
⚠ Another registration exists with this email address.
  Your registration is valid, but you are not the only one using this identifier.
  If this is unexpected, verify that your email has not been used without your knowledge.
```

---

## 20. Property Syntax — Three Distinct Forms

The three bracket syntaxes have distinct semantics. Choosing the right form makes paths unambiguous for both humans and parsers.

| Syntax | Name | Semantics | Example |
|---|---|---|---|
| `[field:value]` | Identifier | Unique, indexable, primary key | `[client-id:C-123456]` |
| `[field "value"]` | Attribute | Descriptive string, not unique | `[name "Juan Ruiz"]` |
| `[field value]` | Property | Numeric or enumerated value | `[amount 100]`, `[currency EUR]` |

The colon carries the uniqueness semantics implicitly — no `is-unique` declaration is needed in `fields {}`. The parser knows by syntax what is an identifier and what is a descriptor.

```
# Identifier — unique, indexable
[client-id:C-123456]/
[dni:12312312A]/
[instance:2]/

# Attribute — descriptive string
[name "Juan Ruiz"]/
[reason "Expired document"]/
[objective "Launch v1.0"]/

# Property — numeric or enumerated
[amount 100]/
[currency EUR]/
[capacity 80]/
[rating 4.2]/
```

The same entity can appear as an identifier in one context and as a property in another. The container defines the semantics, not the value itself:

```
professors/juan-ruiz/          ← object — juan-ruiz has identity and controls subtree
transfer/[from:juan-ruiz]/     ← identifier — juan-ruiz is a participant role in the transfer
transfer/[name "Juan Ruiz"]/   ← attribute — descriptive, not unique
```

---

## 21. Verifiable Action Pattern — `fields {}` with Signature

Any operation that requires cryptographic authorisation follows the same pattern: the node publishes a `fields {}` schema, the wallet reads it, validates inputs, and signs with `respond_to_challenge`. The receiving node verifies with `verify_child_authorship`.

```
# Schema declaration — published by the institution
bank/santander/transfer/fields {
    amount       is-type decimal-positive
    concept      is-type string
    currency     is select-one-from { EUR, USD, GBP }
    date         is-type date
    from-account is-type iban
    to-account   is-type iban
}
```

**Rule:** any `fields {}` schema that declares a verifiable action should include the fields needed to make the action non-repudiable — at minimum a `date` and enough context to identify the action uniquely.

Two institutions that publish identical `fields {}` schemas for the same action are interoperable by construction. A wallet that works with one works with the other without any integration work. This is the practical expression of interoperability by design (whitepaper section 2.9).

### The `procedures/` convention for multi-operation domains

When an institution publishes several related operations, group them under a `procedures/` folder. This is a recommended convention, not a reserved name:

```
banco-santander.es/trusteando/procedures/
├── transfer/fields { ... }
├── account-opening/fields { ... }
├── loan-application/fields { ... }
└── kyc/fields { ... }
```

Any institution in the same sector that adopts the same `procedures/` structure becomes interoperable with any wallet or client that knows the standard path. This is particularly valuable in regulated sectors where a common schema acts as a machine-readable regulatory standard:

| Sector | Standard procedures path |
|---|---|
| Banking | `trusteando/procedures/transfer/`, `kyc/`, `account/` |
| Healthcare | `trusteando/procedures/prescription/`, `appointment/`, `referral/` |
| Public administration | `trusteando/procedures/registration/`, `permit/`, `certificate/` |
| Education | `trusteando/procedures/enrolment/`, `degree/`, `transcript/` |

The `procedures/` folder does not need to be declared as reserved — its value comes from convergence across implementations in the same sector, not from protocol enforcement.

---

## 22. Workflow Error Handling — `steps/` with `failed` and `aborted`

When a workflow step fails, append the failure as a new state — never modify or delete the existing record.

```
execution/steps/
├── 01-contract/[state completed]/since/2026-03-22/
├── 02-social-security/
│   ├── [state failed]/since/2026-03-23/         ← preserved permanently
│   │   └── [reason "Expired ID document"]/
│   └── [state completed]/since/2026-03-24/      ← successful retry
└── 03-system-access/[state pending]/
```

### Rules

- `[state failed]` — the step did not complete. It can be retried by appending a new `[state completed]` or `[state aborted]`.
- `[state aborted]` — definitive cancellation. No retry. Subsequent steps do not start.
- `[reason "text"]` — optional child of a failed or aborted state. Human-readable, not machine-parsed.
- If any step has `[state failed]` or `[state aborted]`, subsequent steps remain `[state pending]`.
- The complete failure history is append-only — it can never be removed.

### Avoid modifying published states

```
# Wrong — modifies existing state, breaks the signature
execution/steps/02-social-security/[state failed]/ → DELETE

# Correct — appends new state, history preserved
execution/steps/02-social-security/[state completed]/since/2026-03-24/
```

---

## 23. Sector Schemas and Interoperability Patterns

The protocol's power as an interoperability standard emerges when independent organisations adopt the same `fields {}` schemas for the same operations. This section documents recommended patterns for common sectors.

### 23.1 When to define a sector schema

Define a sector schema when two or more independent organisations need to perform the same operation and a third party (wallet, verifier, regulator) needs to interact with both without custom integration.

The schema should be the minimum necessary — only the fields that are universal across all implementations in the sector. Organisation-specific fields go under `private/` or as optional extensions.

### 23.2 Schema stability

A published `fields {}` schema is a public contract. Changing it breaks existing wallets. Follow these rules:

- **Add fields** by publishing a new schema version under a versioned path: `procedures/transfer/v2/fields {}`
- **Never remove or rename** fields from a published schema version
- **Deprecate** old versions with `until/` rather than deleting them

```
bank/santander/trusteando/procedures/
├── transfer/
│   ├── v1/fields { ... }     ← deprecated, still valid
│   │   └── until/2027-01-01/
│   └── v2/fields { ... }     ← current
```

### 23.3 Cross-sector example — medical prescription

```
hospital-la-paz.es/trusteando/procedures/prescription/fields {
    diagnosis-code   is-type string          ← ICD-10 code
    drug-name        is-type string
    dosage           is-type string
    duration-days    is-type integer
    patient-id       is-type dni-es
    prescriber-id    is-type string
    date             is-type date
}
```

Any pharmacy that reads this schema can verify a prescription issued by any hospital that publishes the same structure — without a bilateral agreement between the hospital and the pharmacy.

---

## 16. Considerations for Style Guide v0.3

These UML concepts merit further study for a future version of this guide:

- **Explicit multiplicities** — `[multiplicity 0..*]` for cardinality constraints
- **Layered views** — `conceptual/`, `logical/`, `implementation/` separation
- **OCL-like constraints** — a `constraints/` folder for machine-verifiable rules
- **Namespace conventions** — `country/region/type/identifier` path hierarchy

---

## 24. Static Verification — Where Verifiers Find Proof

A common question for implementors: where does a verifier look for cryptographic proof when the server is static (Level 1) and cannot respond to real-time challenges?

### The two verification modes

**Dynamic verification (Level 2+):** The verifier sends a challenge to the child node, which responds with `respond_to_challenge`. The parent verifies with `verify_child_authorship`. This requires a live server.

**Static verification (Level 1):** The parent periodically publishes a signed list of its valid children with a TTL. The verifier downloads this list once and verifies locally during the TTL period. No live challenge required.

### The `signed-members/` convention

A parent node that wants to support static verification publishes a signed membership list:

```
professors/
├── juan-ruiz/since/2021/
├── ana-garcia/since/2023/
└── signed-members/
    ├── [valid-until 2026-03-25T00:00:00Z]/
    ├── [signed-by @uma.es]/
    └── members.json          ← signed list of valid child HMACs
```

The `members.json` file contains:

```json
{
  "parent": "uma.es/trusteando/professors/",
  "valid-until": "2026-03-25T00:00:00Z",
  "members": [
    { "path": "juan-ruiz", "hmac": "a3f9e2b1..." },
    { "path": "ana-garcia", "hmac": "c7d4f891..." }
  ],
  "signature": "3045022100..."
}
```

A verifier that finds `signed-members/` can verify any child's membership locally without a live challenge. The TTL in `[valid-until]` bounds the window of potential inconsistency — see section 13.6 of the whitepaper for the revocation latency trade-off.

### What a verifier should look for

When a verifier encounters a node and wants to verify its membership:

1. **Check for `signed-members/`** in the parent — if present, verify against the signed list (static mode)
2. **If absent**, send a live challenge to the parent's server (dynamic mode)
3. **If the parent is unreachable**, treat the node as unverifiable — not invalid, but unverifiable

The absence of `signed-members/` is not an error — it means the parent only supports dynamic verification.

### For binary files and documents

When a node contains a binary file (PDF, image, executable), publish its hash as a property to maintain graph integrity:

```
degrees/juan-ruiz/
├── since/2021-06-15/
├── diploma.pdf
└── [file:diploma.pdf]/
    ├── [sha256 "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"]/
    ├── [mime "application/pdf"]/
    └── [size-bytes 1048576]/
```

The `[file:filename]` property anchors the binary to the graph. Any verifier can check the file's integrity by computing its SHA-256 and comparing it to the published hash. The file itself is not part of the Trusteando graph — the hash is.

---

## 25. Large Collections — Semantic Pagination

When a node has thousands of children (a university with 50,000 students, a bank with millions of accounts), flat structure becomes impractical for parsers and verifiers.

### Partition by time

```
students/
├── by-year/
│   ├── 2024/
│   │   ├── juan-ruiz/
│   │   └── ana-garcia/
│   └── 2025/
│       └── pedro-lopez/
└── signed-members/       ← covers all partitions
```

### Partition by identifier prefix

```
students/
├── by-surname/
│   ├── a-f/
│   │   └── ana-garcia/
│   ├── g-m/
│   │   └── juan-ruiz/
│   └── n-z/
│       └── pedro-lopez/
└── signed-members/
```

### Rules for partitioned collections

- The `signed-members/` list covers the **entire collection** across all partitions — not per partition
- Partition folders (`by-year/`, `by-surname/`) are **containers**, not objects — they have no keys of their own
- Partition names are **structural** — they do not appear in the child's identity path. `juan-ruiz`'s canonical path is `students/juan-ruiz/`, not `students/by-surname/g-m/juan-ruiz/`
- Use partitions when a collection exceeds ~1000 members

---

*This is a living document. Conventions evolve with practice.*
*confidencenode.org/protocolos/trusteando — Style Guide v0.3*

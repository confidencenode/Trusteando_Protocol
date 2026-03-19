# Trusteando Style Guide

**Conventions and best practices for publishing verifiable knowledge graphs**

*v0.1 — companion to the Trusteando Protocol Whitepaper v0.2*
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

### 5.2 private/ for content requiring permission

```
professors/juan-ruiz/
├── email/              ← public
├── department/         ← public
└── private/
    └── salary/         ← requires grantReveal()
```

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

## 12. Considerations for Style Guide v0.2

These UML concepts merit further study for a future version of this guide:

- **Explicit multiplicities** — `[multiplicity 0..*]` for cardinality constraints
- **Layered views** — `conceptual/`, `logical/`, `implementation/` separation
- **OCL-like constraints** — a `constraints/` folder for machine-verifiable rules
- **Namespace conventions** — `country/region/type/identifier` path hierarchy

---

*This is a living document. Conventions evolve with practice.*
*confidencenode.org/protocolos/trusteando — Style Guide v0.1*

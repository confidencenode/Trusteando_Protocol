# Trusteando Cookbook

**Practical examples for common domains.**

Each recipe shows a complete folder structure for a real use case, with annotations explaining the modelling decisions. All examples follow the [Style Guide](trusteando_style_guide.md).

---

## Recipe 1 — University and its professors

```
university-malaga/trusteando/
├── identity/
│   ├── [official-name "Universidad de Málaga"]/
│   ├── [label:es "Universidad de Málaga"]/
│   ├── [label:en "University of Malaga"]/
│   ├── [acronym "UMA"]/
│   ├── [founded 1972]/
│   └── [language es]/
├── professors/
│   ├── juan-ruiz/
│   │   ├── since/2021-09-01/
│   │   ├── [cpi:prof-uma-2010-0042]/
│   │   └── [email:juan@uma.es]/
│   └── signed-members/
│       ├── [valid-until 2026-04-01T00:00:00Z]/
│       └── [signed-by @uma.es]/
├── degrees/
│   └── computer-science/
│       ├── [is-accredited-by aneca]/
│       └── since/2010-01-01/
└── [state trusteado]/
```

**Notes:** `signed-members/` enables static verification without a live challenge. `[cpi]` gives the professor a portable identifier that survives institutional path changes.

---

## Recipe 2 — Hospital and medical staff

```
hospital-la-paz/trusteando/
├── identity/
│   ├── [official-name "Hospital Universitario La Paz"]/
│   └── [is-accredited-by msc.gob.es]/
├── physicians/
│   ├── maria-lopez/
│   │   ├── since/2018-06-01/
│   │   ├── [specialty cardiology]/
│   │   └── [license:col-28-12345]/
│   └── signed-members/
│       └── [valid-until 2026-04-01T00:00:00Z]/
├── procedures/
│   └── prescription/fields {
│       date             is-type date
│       diagnosis-code   is-type string
│       drug-name        is-type string
│       dosage           is-type string
│       duration-days    is-type integer
│       patient-id       is-type dni-es
│       prescriber-id    is-type string
│   }
└── [state trusteado]/
```

**Notes:** `procedures/prescription/fields {}` is the sector schema. Any pharmacy that reads this can verify prescriptions from any hospital publishing the same structure — no bilateral agreement.

---

## Recipe 3 — Bank and verifiable transfers

```
banco-santander/trusteando/
├── identity/
│   └── [swift:BSCHESMM]/
├── procedures/
│   ├── transfer/fields {
│   │   amount        is-type decimal
│   │   concept       is-type string
│   │   currency      is select-one-from { EUR, USD, GBP }
│   │   date          is-type date
│   │   from-account  is-type iban
│   │   to-account    is-type iban
│   }
│   └── kyc/fields {
│       date          is-type date
│       dni           is-type dni-es
│       name          is-type string
│       type          is select-one-from { basic, enhanced }
│   }
├── accounts/
│   └── private/
└── [state trusteado]/
```

**Notes:** the protocol records the *declaration* of a transfer, not the settlement. The bank's internal systems handle the actual movement of funds (whitepaper §13.8).

---

## Recipe 4 — Individual professional credential

```
dr-maria-lopez/trusteando/
├── identity/
│   ├── [cpi:person-maria-lopez-1985]/
│   └── [is-a person]/
├── credentials/
│   ├── medical-license/
│   │   ├── since/2018-06-01/
│   │   ├── [issued-by @colegio-medicos-madrid.es]/
│   │   └── [specialty cardiology]/
│   └── university-degree/
│       ├── since/2012-06-15/
│       └── [issued-by @ucm.es]/
└── [state trusteado]/
```

**Notes:** the person does not certify themselves — they point to where their certifications live. `[issued-by @entity]` references the licensing body as a signing entity.

---

## Recipe 5 — Bilateral agreement

```
# Both parties publish identical structures in their own spaces
empresa-a/trusteando/agreements/consulting-2026-xyz/
├── since/2026-03-18/
├── [firmante @empresa-a.es]/
├── [firmante @empresa-b.es]/
├── plan/
│   ├── [objective "IT consulting services 2026"]/
│   └── [state approved]/
├── execution/
│   └── signature/since/2026-03-18/
└── private/
    └── economic-conditions/
```

**Notes:** concordance between both parties' published structures is the proof of bilateral consent. If the structures diverge, there is a dispute (whitepaper §7.9).

---

## Recipe 6 — Public administration (citizen registration)

```
T10/spain/register/
├── b9/
│   └── [email:juan@example.com]/
│       └── since/2026-03-23/
├── v9/
│   └── [email:juan@example.com][phone:+34612345678]/
│       └── since/2026-03-23/
└── t9/
    └── [dni:12312312A][email:juan@example.com]/
        └── since/2026-03-23/
```

**Notes:** the three subtrees (b9/v9/t9) are cryptographically isolated — a b9 key cannot produce a valid proof for a t9 path. The path is the proof of level.

---

## Recipe 7 — Workflow with steps

```
empresa-x/trusteando/onboarding/employee-juan-ruiz-2026/
├── plan/
│   ├── [objective "Onboarding Juan Ruiz"]/
│   └── [state approved]/
├── steps/
│   ├── 01-contract/
│   │   └── [state completed]/since/2026-03-22/
│   ├── 02-social-security/
│   │   ├── [state failed]/since/2026-03-23/
│   │   │   └── [reason "Expired ID document"]/
│   │   └── [state completed]/since/2026-03-24/
│   └── 03-system-access/
│       └── [state pending]/
└── [firmante @juan-ruiz.es]/
```

**Notes:** failed steps are never deleted — preserved permanently with `[reason]` and the retry appended beneath (style guide §22).

---

## Recipe 8 — Multilateral consent

```
# Canonical document at neutral node
notario.es/trusteando/consents/[id:C-2026-001]/
├── [patient consent]/
├── [hospital-a consent]/
├── [hospital-b consent]/
└── since/2026-03-24/

# Patient declares consent (private)
patient-juan/trusteando/private/
└── [data-sharing consent extern/notario.es/trusteando/consents/[id:C-2026-001]]/

# Hospital references same document
hospital-la-paz/trusteando/patient-consents/
└── [juan-ruiz consent extern/notario.es/trusteando/consents/[id:C-2026-001]]/
```

**Notes:** three independent references converging on one canonical document is the proof of multilateral consent. The patient keeps their reference under `private/`.

---

*Trusteando Protocol — confidencenode.org/protocolos/trusteando*
*Style Guide, Quickstart, and Whitepaper in the same directory.*

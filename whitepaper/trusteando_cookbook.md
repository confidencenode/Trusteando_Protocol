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

## Recipe 9 — AI Agent Mandate (citizen authorising a tax agent)

**Domain:** public administration
**Entities:** citizen (principal), AI tax agent, AEAT
**Key questions:** is this agent authorised to file on behalf of this citizen, within what scope and limits?

### Step 1 — Citizen grants the mandate

```
juan-ruiz.es/trusteando/mandates/ai-agents/
└── [id:mandate-2026-001]/
    ├── since/2026-03-25T10:00:00Z/
    ├── until/2026-12-31T23:59:59Z/
    ├── [agent @agente-fiscal.es]/
    ├── [principal @juan-ruiz.es]/
    ├── [scope "trusteando/procedures/tax-*"]/    ← path-pattern
    ├── [allows-machine-pace true]/
    ├── [limit-value "EUR:0"]/
    ├── [mandate-ref "FNMT-APODERAMIENTO-2026-ES-12345"]/
    ├── [firmante @juan-ruiz.es]/
    └── [firmante @agente-fiscal.es]/
```

### Step 2 — AEAT declares its agent pathway

```
aeat.es/trusteando/procedures/
├── tax-filing/
│   ├── fields {
│   │   date      is-type date
│   │   dni       is-type dni-es
│   │   modality  is select-one-from { renta, sociedades, iva }
│   │   }
│   └── [requires-human-pace]/
│
└── tax-filing-agent/
    ├── fields {
    │   agent-id       is-type url
    │   date           is-type date
    │   mandate-ref    is-type string
    │   model-version  is-type string
    │   modality       is select-one-from { renta, sociedades, iva }
    │   principal-dni  is-type dni-es
    │   }
    ├── [agent-pathway true]/
    ├── [accepts-ai-mandate]/
    ├── [required-scope-matches "trusteando/procedures/tax-*"]/
    ├── [max-submissions-per-hour 10]/
    └── [audit-level enhanced]/
```

### Step 3 — AEAT verifies the mandate chain

```
1. Fetch juan-ruiz.es/trusteando/mandates/ai-agents/[id:mandate-2026-001]/
2. Verify [firmante @juan-ruiz.es] — citizen signed with DNIe/Cl@ve
3. Verify since/ <= today <= until/
4. Confirm [scope tax-filings] covers this operation
5. Confirm [limit-value "EUR:0"] — no financial commitment
6. Confirm [allows-machine-pace true]
7. Optionally verify FNMT-APODERAMIENTO-2026-ES-12345 in FNMT registry
8. Process — record verification in audit log
```

### Step 4 — If something goes wrong

```
aeat.es/trusteando/agent-suspensions/
└── [agent @agente-fiscal.es]/
    ├── [reason "anomalous-submission-pattern"]/
    ├── since/2026-04-01T14:00:00Z/
    └── [principal-unaffected true]/
```

Juan retains full access to the human pathway. The agent is suspended pending review.

**Modelling notes:**
- The mandate lives under **Juan's node**, not the agent's — mirrors legal convention for powers of attorney.
- `principal-id` uses `t9-identity` type — the mandate is only legally meaningful if the principal is t9-verified (DNIe or FNMT certificate). A b9 mandate is a self-declaration; a t9 mandate has institutional backing.
- `scope` as `path-pattern` mirrors how powers of attorney work: "authorised for my tax affairs" (`trusteando/procedures/tax-*`) rather than a fixed enumeration. The institution checks that the submitted path matches the declared pattern.
- `[limit-value "EUR:0"]` is the safe default for informational filings. A higher limit requires a new explicit signature from the citizen.
- The `model-version` field enables forensic traceability: if a specific model version misinterprets a rule, AEAT can identify all filings made by that version and review systematically.
- The FNMT `mandate-ref` provides a second independent verification layer — cryptographic plus legal, the same dual anchor as a BOE publication.
- Two parallel pathways (`tax-filing` and `tax-filing-agent`) allow the institution to apply different controls to human and machine actors without conflating them.


---

---

## Recipe 10 — Journalism and photographic provenance

**Domain:** digital media and press
**Entities:** camera (autonomous signing device), journalist, newspaper, reader/verifier
**Key question:** was this photo taken by this camera, published unaltered by this outlet, and is it still endorsed?

### The structure

```
periodico-el-pais.es/trusteando/noticias/2026/03/25/cumbre-clima/
├── [headline "World leaders reach climate agreement"]/
├── [author @maria-gomez.es]/
├── since/2026-03-25T14:32:00Z/
├── assets/
│   └── foto-001/
│       ├── [hash "sha256:a3f9e2b1c4d7..."]/   ← hash of original file
│       ├── [format jpeg]/
│       ├── [firmante @camara-canon-78.es]/       ← camera as autonomous signing entity
│       ├── [firmante @maria-gomez.es]/            ← journalist countersigns
│       └── since/2026-03-25T14:31:47Z/           ← timestamp from camera clock
└── [firmante @el-pais.es]/
```

The camera is registered as an autonomous `@` signing entity:

```
camara-canon-78.es/trusteando/
├── identity/
│   ├── [device-type camera]/
│   ├── [model "Canon EOS R5"]/
│   ├── [serial "083000012847"]/
│   └── [owner @maria-gomez.es]/
├── [is-registered-with @canon.com]/
└── [state trusteado]/
```

### Verification flow

```
1. Verifier fetches periodico.es/.../foto-001/
2. Checks [hash] matches the image file bytes
3. Verifies [firmante @camara-canon-78.es] — camera's key signed the hash
4. Verifies [firmante @maria-gomez.es] — journalist countersigned
5. Verifies [firmante @el-pais.es] — outlet published and endorsed
6. Checks camara-canon-78.es/trusteando/ — registered device, known owner
7. Checks since/ timestamps — camera timestamp precedes journalist and outlet signatures
```

A verifier who completes all seven steps has cryptographic proof that the image bytes have not changed since leaving the camera, that the journalist who took it endorsed it, and that the outlet published it in that state.

**Modelling notes:**
- The camera is an autonomous `@` entity — its key is generated in its secure enclave at manufacture, not derived from any parent. No parent compromise can produce a valid camera signature.
- `[hash]` commits to the exact image bytes. Any post-processing (crop, filter, compression) changes the hash and breaks the camera's signature — detectable immediately.
- If the outlet later discovers an error (wrong caption, wrong event), they publish a correction node as a new sibling — the original remains in the graph with its original signatures intact (§13.4).
- The `since/` on the asset is the camera's embedded timestamp. The `since/` on the article node is the publication timestamp. The gap between them is the editorial process — auditable.

---

## Recipe 11 — IoT supply chain with cold chain monitoring

**Domain:** logistics and food safety
**Entities:** port authority, container, temperature sensor, logistics operator, receiving party
**Key question:** was the cold chain maintained, and if not, at exactly which node did it break?

### The structure

```
puerto-valencia.es/trusteando/terminal-a/
├── contenedor-[id:MAEU-4521873]/
│   ├── since/2026-03-20T08:00:00Z/
│   ├── [cargo "frozen fish"]/
│   ├── [required-temp-max "-18"]/
│   ├── [operator @maersk-logistics.es]/
│   ├── sensors/
│   │   └── [id:sensor-TMP-447]/
│   │       ├── [device-type temperature]/
│   │       ├── [firmante @sensor-TMP-447.iot]/  ← sensor as autonomous signing entity
│   │       └── readings/
│   │           ├── [id:read-001]/
│   │           │   ├── [temp "-20.3"]/
│   │           │   ├── since/2026-03-20T09:00:00Z/
│   │           │   └── [firmante @sensor-TMP-447.iot]/
│   │           ├── [id:read-002]/
│   │           │   ├── [temp "-12.1"]/           ← breach
│   │           │   ├── since/2026-03-20T14:22:00Z/
│   │           │   ├── [state failed]/
│   │           │   ├── [reason "refrigeration-unit-failure"]/
│   │           │   └── [firmante @sensor-TMP-447.iot]/
│   │           └── [id:read-003]/
│   │               ├── [temp "-19.8"]/            ← restored
│   │               └── since/2026-03-20T16:05:00Z/
│   └── [firmante @puerto-valencia.es]/
└── [state trusteado]/
```

### What the graph proves

When the receiving party inspects the container, they fetch the sensor readings subtree. The signed readings show:

- Every temperature record is signed by the sensor itself — not by the operator, not by the port. The operator cannot retroactively alter readings.
- The breach at `read-002` is a permanent, signed, timestamped fact. It cannot be removed.
- The exact node (`terminal-a/contenedor-MAEU-4521873/sensors/sensor-TMP-447/readings/read-002/`) identifies where in the hierarchy the failure occurred — which terminal, which container, which sensor, at what time.

**Modelling notes:**
- Sensors are autonomous `@` entities with keys in their hardware secure enclave. An operator who controls the port's root key cannot forge sensor readings — sensor keys are independent.
- `[state failed]` on a reading does not revoke the container — it marks the specific reading as a breach event. The graph preserves full history; downstream parties decide what threshold is acceptable for their use case.
- For high-value cargo, the receiving party can require a `signed-members/` snapshot of the readings subtree, enabling offline verification of the complete cold chain history without querying the port's live server.
- The same pattern applies to any IoT monitoring scenario: pharmaceutical cold chain, art transport humidity monitoring, industrial equipment vibration logs.

---

## Recipe 12 — AI model training provenance

**Domain:** AI governance and auditability
**Entities:** AI developer, training datasets, model versions, auditor
**Key question:** what data was used to train this model version, is that data licensed, and has anything changed since the audit?

### The structure

```
openmodel-org.es/trusteando/models/
└── [id:gpt-openmodel-v3]/
    ├── since/2026-01-15T00:00:00Z/
    ├── [architecture "transformer"]/
    ├── [parameter-count "7B"]/
    ├── training-data/
    │   ├── [id:dataset-001]/
    │   │   ├── [name "CommonCrawl-2024-Q1"]/
    │   │   ├── [hash "sha256:f7c3a1..."]/      ← hash of dataset at time of use
    │   │   ├── [license "CC-BY-4.0"]/
    │   │   ├── [source extern/commoncrawl.org/trusteando/datasets/2024-q1/]/
    │   │   └── since/2025-11-01/
    │   ├── [id:dataset-002]/
    │   │   ├── [name "curated-legal-es"]/
    │   │   ├── [hash "sha256:3b9d72..."]/
    │   │   ├── [license "proprietary"]/
    │   │   ├── [license-ref extern/boe.es/trusteando/licenses/ai-training-2025-001/]/
    │   │   └── since/2025-10-15/
    │   └── [total-tokens "3.2T"]/
    ├── evaluations/
    │   ├── [id:eval-bias-2026-01]/
    │   │   ├── [method "winogender"]/
    │   │   ├── [result "0.87"]/
    │   │   ├── [firmante @auditor-aenor.es]/
    │   │   └── since/2026-01-20/
    │   └── [id:eval-harmful-2026-01]/
    │       ├── [method "ToxiGen"]/
    │       ├── [result "0.03"]/
    │       ├── [firmante @auditor-aenor.es]/
    │       └── since/2026-01-20/
    ├── [firmante @openmodel-org.es]/
    └── [state trusteado]/
```

### What the graph proves

```
1. Auditor fetches openmodel-org.es/trusteando/models/[id:gpt-openmodel-v3]/training-data/
2. For each dataset:
   a. Verifies [hash] matches the dataset archive — bytes have not changed since training
   b. Follows extern/ reference to the dataset's own node — checks its license is current
   c. Verifies license covers AI training use for the declared date range
3. Checks evaluations/ — each signed by an independent auditor (@auditor-aenor.es)
4. Verifies the model node is signed by the developer — they cannot disown it
```

**Modelling notes:**
- `[hash]` on each dataset commits to the exact bytes used during training. If a dataset is later found to contain problematic content, the hash identifies whether this specific model used the exact version in question or a different one.
- `extern/` references to dataset nodes enable live license checking — if a license expires or is revoked, the reference chain reflects it without the model node needing to be updated.
- Independent auditor signatures (`[firmante @auditor-aenor.es]`) on evaluation results are the key governance mechanism — the developer cannot self-certify bias or safety results for regulated contexts.
- Under the EU AI Act, high-risk AI systems require documented training data provenance. This structure provides exactly that: a signed, append-only, externally verifiable record of what data was used, under what license, with what evaluation results, audited by whom.
- New model versions are new nodes (`[id:gpt-openmodel-v4]`) — the old version's record is never modified. An investigator can always recover the exact training configuration of any past version.

---

## Recipe 13 — Verifiable sale receipt

**Domain:** retail and e-commerce
**Entities:** merchant, transaction, customer, verifier (returns desk, auditor, tax authority)
**Key question:** is this receipt genuine, was it issued by this merchant, and has it been altered or reused?

### The structure

```
tienda-garcia.es/trusteando/receipts/
└── [id:tx-2026-03-25-00847]/
    ├── since/2026-03-25T11:42:00Z/
    ├── [total "EUR:127.50"]/
    ├── [vat "EUR:22.50"]/
    ├── [payment-method card]/
    ├── [terminal "POS-03"]/
    ├── items/
    │   ├── [id:item-001]/
    │   │   ├── [description "Zapatillas Nike Air Max"]/
    │   │   ├── [sku "NK-AM-42-BLK"]/
    │   │   ├── [qty 1]/
    │   │   └── [price "EUR:105.00"]/
    │   └── [id:item-002]/
    │       ├── [description "Calcetines pack x3"]/
    │       ├── [sku "SOC-3PK-M"]/
    │       ├── [qty 1]/
    │       └── [price "EUR:22.50"]/
    └── [firmante @tienda-garcia.es]/
```

The merchant publishes the receipt at a stable URL. The QR on the physical or digital receipt encodes that URL plus the transaction hash:

```
QR content:
  tienda-garcia.es/trusteando/receipts/[id:tx-2026-03-25-00847]/
  hash: sha256:c4f1a3...
```

### Verification flow

```
1. Customer scans QR — fetches receipt node
2. Verifier checks [firmante @tienda-garcia.es] — merchant signed this
3. Verifier checks since/ timestamp — matches point-of-sale time
4. Verifier checks QR hash matches receipt content — bytes unchanged
5. Verifier traces authority chain:
   tienda-garcia.es → [is-registered-with @camara-comercio-madrid.es] → t9
6. Receipt is VERIFIED: genuine, unaltered, issued by this merchant
```

For a return:

```
tienda-garcia.es/trusteando/receipts/[id:tx-2026-03-25-00847]/
└── return/
    ├── [id:return-2026-03-28-00112]/
    ├── since/2026-03-28T10:15:00Z/
    ├── [items-returned "item-001"]/
    ├── [refund "EUR:105.00"]/
    ├── [method card-reversal]/
    └── [firmante @tienda-garcia.es]/
```

The return is a child node of the original receipt — permanently linked, signed by the merchant, auditable by any verifier. The original receipt is never modified.

**Modelling notes:**
- The receipt URL is the receipt. No paper, no PDF, no shared database between merchant and customer — just a public URL that both parties can verify independently at any time.
- The QR encodes both the URL and a content hash. A merchant cannot silently alter a receipt after issuing it — the hash would no longer match. This eliminates the most common form of receipt fraud.
- The authority chain (`tienda-garcia.es → Cámara de Comercio`) is what a Trust Badge (§16.5) would display to a customer before they complete a purchase: "This merchant is registered with the Madrid Chamber of Commerce." The receipt and the badge use the same underlying graph — the receipt is a node in the merchant's trust structure, not a separate document.
- For the tax authority, the `receipts/` subtree is a signed, append-only ledger. An auditor can verify any transaction by fetching its URL — no special access, no API key, no bilateral agreement with the merchant's accounting software.
- `[terminal "POS-03"]` enables the merchant to trace which physical terminal issued which receipts — useful for detecting compromised hardware or employee fraud without exposing customer data.

---

*Trusteando Protocol — confidencenode.org/protocolos/trusteando*
*Style Guide, Quickstart, and Whitepaper in the same directory.*

# Trusteando and Institutional Transparency

**A positioning document for public institutions, civil society, and investigative journalism**

*Companion to the Trusteando Protocol Whitepaper v0.2*
*confidencenode.org/protocolos/trusteando*

---

## The Transparency Problem

Public institutions already publish a great deal of information — organisational charts, competency frameworks, registries, and official directories. But this information is scattered across PDFs, inconsistent HTML, disconnected databases, and formats that are difficult to query or verify.

The result is a structural asymmetry: institutions know their own structure well, but citizens, journalists, and oversight bodies struggle to navigate it. Not because the information is secret, but because it is inaccessible in practice.

Trusteando addresses this not by demanding new disclosures, but by providing a standard schema for information that is already public.

---

## What Changes with Trusteando

Today, a citizen who wants to know which ministry has authority over a specific area must navigate multiple websites, read legislation, and cross-reference documents. With Trusteando:

```
GET ministerio.es/trusteando/competencias/agua/
→ "Ministerio para la Transición Ecológica"

GET transicion.gob.es/trusteando/delegaciones/madrid/
→ verified delegation exists, since 2020-01-01
```

The same information — already public — but structured, verifiable, and queryable. A citizen, a journalist, or an AI agent can navigate institutional structures in minutes instead of days.

---

## The Reputational Incentive

Trusteando does not oblige any institution to publish anything. It only allows those who already publish to do so in a verifiable, structured way.

The incentive mechanism is reputational, not coercive. Institutions that adopt the protocol gain:

- **Verified credibility** — their structure and competencies are queryable and auditable
- **Interoperability** — they become automatically connectable with other Trusteando nodes
- **Trust** — any claim they make about their own structure is cryptographically backed

Institutions that do not adopt it are not penalised. But over time, absence from the graph becomes information in itself — the same way that not having a website was first normal, then unusual, then a signal worth noting.

---

## For Investigative Journalism

Journalists investigating institutional structures today rely on manual research, freedom of information requests, and source leaks. Trusteando creates a complementary layer of structured, verifiable, public data:

- Organisational hierarchies: who depends on whom, since when
- Competency boundaries: which body has authority over which domain
- Inter-institutional relations: partnerships, delegations, shared authorities

This is not a replacement for investigative journalism — it is infrastructure that makes certain factual questions answerable without sources, because the answer is already public and structured.

---

## For Civil Society

Citizens and civil society organisations can use Trusteando-structured data to:

- Verify which institution is responsible for a specific issue before contacting it
- Track changes in organisational structure over time (the graph is append-only — history is preserved)
- Cross-reference declared competencies with actual actions
- Identify gaps in coverage — areas where no institution has declared responsibility

---

## The Neutrality of Structure

Trusteando is technically neutral — it publishes structures, not judgements. But structural transparency has political implications in any context where opacity currently serves established interests.

The protocol does not make this judgement. It provides the infrastructure. What society does with verified, structured, public information is a political and social question, not a technical one.

---

## Adoption Path for Public Institutions

The minimum adoption for a public institution is straightforward:

1. Publish a `trusteando/` folder on the existing domain
2. Structure the existing public directory and competency information using the folder schema
3. Sign the content with the institution's existing cryptographic infrastructure

No new data is required. No new legislation is needed. The institution simply structures what it already publishes.

---

## What is Not Changed

Trusteando does not:

- Require institutions to publish information they are not already required to publish
- Override privacy protections for personal data
- Create new legal obligations
- Depend on any central authority for operation

It is an open protocol — like HTTP or SMTP — that any institution can implement independently, at its own pace, and to the degree it chooses.

---

*confidencenode.org/protocolos/trusteando*
*Author: confidencenode.org/members/confidencenode0*
*Companion to: Trusteando Protocol Whitepaper v0.2*

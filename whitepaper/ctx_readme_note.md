## Relation to the ConfidenceNode Protocol

`ctx` implements **Layer 3 (Vault)** of the [ConfidenceNode Protocol](https://github.com/confidencenode/protocol).

The ConfidenceNode Protocol addresses information asymmetry — the structural problem underlying trust failures in markets, institutions, and coordination. It defines three layers:

- **Layer 1 — Signal:** what is observable
- **Layer 2 — Verification:** what is cryptographically provable (→ [Trusteando Protocol](https://github.com/confidencenode/Trusteando_Protocol))
- **Layer 3 — Vault:** what is known but not yet published — structured capture of uncertainty (→ this repository)

`ctx` is the vault layer: it provides a structured way to capture, version, and eventually disclose information that an agent holds privately. Where Trusteando publishes verifiable facts to the graph, `ctx` manages the facts that are not yet ready to be published — drafts, uncertainties, private assessments, and conditional disclosures.

The three layers are independently usable. They are designed to compose: a `ctx` vault entry can reference a Trusteando path as its eventual disclosure target, and a Trusteando node can reference a `ctx` vault as a `private/externals/` pointer to information it controls but does not yet expose.

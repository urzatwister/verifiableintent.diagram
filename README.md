# Verifiable Intent — Delegation Chain Diagram

An interactive, field-level annotated diagram visualising the **Verifiable Intent (VI)** cryptographic delegation chain for AI agent authorization in commerce.

> **Spec Reference:** This diagram is based on [agent-intent/verifiable-intent](https://github.com/agent-intent/verifiable-intent) — the open specification for cryptographic agent authorization in commerce. Full documentation is available at [verifiableintent.dev](https://verifiableintent.dev).

---

## What This Diagram Shows

This single-page HTML diagram provides a **visual, annotated breakdown** of the VI credential anatomy — specifically how each SD-JWT layer cryptographically binds the next via `cnf.jwk` (RFC 7800), with selective disclosure per RFC 9901.

Two execution modes are supported and can be toggled interactively:

| Mode | Layers | Use Case |
|------|--------|----------|
| **Autonomous** | L1 → L2 → L3 (split) | User delegates to agent; agent acts without user presence |
| **Immediate** | L1 → L2 (terminal) | User is present and authorises exact checkout + payment details at runtime |

---

## The Delegation Chain

### Layer 1 — Identity (SD-JWT)
- Signed by the **Credential Provider**
- Binds the **user's public key** via `cnf.jwk`
- Contains identity claims (e.g. `email`), `pan_last_four`, and `scheme`
- Lifetime: ~1 year

### Layer 2 — Mandate (KB-SD-JWT)
- Signed by the **User** using the key confirmed in L1's `cnf.jwk`
- Anchors to L1 via `sd_hash`
- Contains purchase constraints (amount range, approved merchants, line items)
- **Autonomous:** Includes `cnf.jwk` = Agent Key (24 h–30 d lifetime)
- **Immediate:** No `cnf` — terminal credential (~15 min lifetime)

### Layer 3 — Execution (Split KB-SD-JWTs) *(Autonomous only)*
Both L3 credentials are signed by the **Agent** using the key confirmed in L2's `cnf.jwk`:

| L3a — Payment Mandate | L3b — Checkout Mandate |
|-----------------------|------------------------|
| Sent to payment **network** | Sent to **merchant** |
| Final payment values | Final checkout (`checkout_jwt`) |
| `transaction_id` | `checkout_hash` |
| `payment_instrument` | — |

**Integrity guarantee:** `L3a.transaction_id == L3b.checkout_hash` — cryptographic proof that the payment mandate references the exact same checkout the user approved.

---

## Diagram Features

- **Toggle** between Autonomous (3-layer) and Immediate (2-layer) modes
- **Field encoding legend** — distinguish always-visible vs. selectively disclosable claims at a glance
- **Required / Optional badges** on every claim
- **Interactive field links** (toggle on/off) — SVG overlay lines showing:
  - `cnf.jwk` → signs the next layer
  - `sd_hash` = hash of the prior layer
  - `kid` ↔ `cnf.kid` match
  - Cross-reference equality constraints
- **Signature & Binding Summary** bar — a concise one-line-per-layer summary of signing authority, binding mechanism, credential type, and lifetime

---

## File Structure

```
verifiableintent_diagram/
└── index.html   # Self-contained interactive diagram (HTML + CSS + JS)
```

No build step, no dependencies, no server required.

---

## Usage

Open `index.html` directly in any modern browser:

```
# Windows
start index.html

# macOS
open index.html

# Linux
xdg-open index.html
```

---

## About Verifiable Intent

**Verifiable Intent** is an open specification maintained by Mastercard, open to multi-stakeholder contribution, that defines a layered SD-JWT credential format creating a tamper-evident chain of cryptographic evidence that an AI agent's actions were within the scope explicitly delegated by a human user.

### The Problem
Today's payment infrastructure assumes human presence at the point of transaction. AI agents break that assumption — without a binding mechanism, every stakeholder (merchant, network, issuer) carries unquantifiable risk that the agent acted outside the user's wishes.

### What VI Solves
- **Verifiable delegation** — Any party can cryptographically verify that an agent's actions trace back to explicit user authorization
- **Minimal disclosure** — Each party sees only the claims required for its role
- **Constraint enforcement** — Quantitative constraints (amount, payee, merchant) are machine-enforceable; descriptive fields provide informational context
- **Protocol agnostic** — Works across payment protocols, agent frameworks, and commerce platforms
- **Standards aligned** — Built on SD-JWT, JWS, JWK, and RFC 7800; no novel cryptography

### Specification Documents
| Document | Description |
|----------|-------------|
| [spec/README.md](https://github.com/agent-intent/verifiable-intent/blob/main/spec/README.md) | Architecture, trust model, conformance |
| [spec/credential-format.md](https://github.com/agent-intent/verifiable-intent/blob/main/spec/credential-format.md) | Credential format, claim tables, serialization |
| [spec/constraints.md](https://github.com/agent-intent/verifiable-intent/blob/main/spec/constraints.md) | Constraint types and validation rules |
| [spec/security-model.md](https://github.com/agent-intent/verifiable-intent/blob/main/spec/security-model.md) | Security analysis, threats, key management |

---

## Credits

- **Diagram drawn by** CY Tan
- **Specification** — [Verifiable Intent v0.2](https://github.com/agent-intent/verifiable-intent) · Draft · Maintained by Mastercard
- **Typography** — [Outfit](https://fonts.google.com/specimen/Outfit) & [JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono) via Google Fonts
- **Standards** — [RFC 7800](https://www.rfc-editor.org/rfc/rfc7800) (Key Confirmation), [RFC 9901](https://www.rfc-editor.org/rfc/rfc9901) (SD-JWT)

---

## License

This diagram is provided for reference and educational purposes. The underlying Verifiable Intent specification is licensed under [Apache 2.0](https://github.com/agent-intent/verifiable-intent/blob/main/LICENSE).

# Fides Protocol

**Built for trust.**

---

## What is Fides?

Fides is an open protocol that makes public spending integrity verifiable and enforceable. It defines the minimum technical rules to ensure that no public payment can be executed without a prior, immutable, and verifiable decision record.

**No record, no payment.**

---

## The Problem

Every government has systems to track spending. None of them **block** irregular payments before they happen. They record, report, and audit — but the money is already gone.

Fides inverts this: the payment system itself refuses to execute without valid authorization.

---

## Core Principle

> A payment is only authorized if a valid Decision Record exists, is immutable, and has not been revoked.

This is not transparency. This is not auditing. This is a **mechanical lock**.

---

## How It Works

```
1. Authority creates a Decision Record (DR)
2. System validates structure and rules
3. Append-only record stores immutably with hash chain
4. External anchor publishes state hash
5. Payment executor queries: isPaymentAuthorized()?
6. System returns true or false
7. Payment executes or blocks — no override
```

---

## Key Properties

| Property | Description |
|----------|-------------|
| **Binary verification** | `true` or `false` — no interpretation |
| **Append-only ledger** | No UPDATE, no DELETE, no rewrite |
| **External anchor** | Hash published to independent medium |
| **Open source** | AGPLv3 — no capture possible |
| **No single controller** | Separation of functions is mandatory |

---

## Specification

- [Fides Protocol v0.1](spec/FIDES-v0.1.md) — The core specification (frozen)

**Canonical hash (SHA-256):**
```
fad49b01b8f263e179987dc6781bf83cf0efcbce9b20e6beaf8f182a3bffc544
```

## Reference Implementation

- [Architecture](reference/architecture.md)
- [Data Model](reference/data-model.md)
- [Algorithms](reference/algorithms.md)

---

## Why "Fides"?

*Fides* is Latin for **trust, faith, loyalty**. In ancient Rome, Fides was the goddess of trust — the personification of the given word.

The Fides Protocol makes trust possible by making integrity structural.

---

## License

AGPLv3 — This protocol belongs to everyone.

See [LICENSE](LICENSE) for details.

---

## Contributing

This protocol has no single author. See [CONTRIBUTORS.md](CONTRIBUTORS.md).

Contributions are welcome for documentation, tooling, and non-normative materials.
The core specification (v0.1) is frozen.

---

## Contact

For security reports, compatibility questions, or protocol discussions:

**fides-protocol@proton.me**

---

*Integrity as infrastructure.*

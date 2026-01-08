# Origin of the Fides Protocol

**Historical document** — Record of the conceptual evolution that led to the creation of the protocol.

---

## The Journey

### Phase 1: The Transparency App

The initial idea was simple: create an application to show how much each politician spends. Pure transparency — open data, visualizations, rankings.

**Problem identified:** Transparency doesn't prevent misuse. The money has already left. Showing it afterward doesn't solve the problem.

### Phase 2: The Tracking Law

Evolution: propose a law requiring the tracking of every cent of tax, from collection to final expenditure.

**Problem identified:** Tracking is still looking backward. The diversion happens, then you discover it. Enormous complexity, doubtful results.

### Phase 3: The Moment of Decision

Crucial insight: the real point of power is not the payment, it's the **administrative decision** that authorizes the expenditure (commitment, contract, amendment).

If you control that moment, you control everything that follows.

**The question that changed everything:** What if the payment system simply refused to pay without a valid record?

### Phase 4: The Thesis + Proof

The project stopped being "a system" and became something bigger:

1. **A thesis**: It is technically possible to block misuse before the money leaves
2. **A proof**: Reference implementation that demonstrates the thesis
3. **An implication**: If such a mechanism is possible, its absence becomes a legitimate question

> If it is provably possible, the question becomes: why is it not adopted?

---

## The Name

### Evolution

1. Initial explorations: Turnstile, GATE, Airlock, Clarion, Veritas, Candor...
2. **Final choice:** **Fides**

### Why Fides?

*Fides* is Latin for **trust, faith, loyalty**.

In ancient Rome, Fides was the goddess of trust — the personification of the given word. Contracts were sealed in her temple.

The protocol doesn't create trust in people. It makes trust possible by making integrity structural.

### Tagline (non-normative)

**"Built for trust."**

Rejected taglines that sounded paternalistic ("When systems are honest, people can be too" — as if people would only be honest if controlled).

"Built for trust" is neutral: built so that trust is possible.

---

## Core Principle

> **No valid record, no payment.**

This is not transparency. This is not auditing. This is a **mechanical lock**.

The payment system simply refuses to execute without valid authorization. There is no override. There is no operational exception. There is no "release it this time."

---

## What Fides Is NOT

| What it seems | What it is |
|---------------|------------|
| Transparency system | Execution lock |
| Auditing tool | Payment prerequisite |
| Spending dashboard | Authorization protocol |
| Commercial product | Open technical standard |
| AI solution | Simple binary verification |

---

## Success Criterion

> A malicious agent, even with internal access, cannot execute an irregular payment without leaving a verifiable trace.

If this is true, the protocol works.

---

## What This Repository Is

**Fides is ONLY the protocol.**

- Not a system
- Not an application
- Not a product
- Not an implementation

It is a **technical specification** — a document that defines rules. Any government, company, or developer can read and implement it.

### Deliverables in this repository:

1. `spec/FIDES-v0.1.md` — The specification
2. `reference/*.md` — Technical reference documents
3. `LICENSE` — AGPLv3
4. `README.md` — Public explanation

**No code.** Code belongs in implementations.

---

*This document records decisions made during the conception of the project. It serves as a historical reference and should not be retroactively altered.*

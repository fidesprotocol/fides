# Fides Protocol - Threat Model

This document describes known attack vectors against the Fides Protocol and explains how they are addressed, exposed, or intentionally left outside the protocol's scope. These are deliberate boundaries, not oversights.

## Purpose

Security protocols benefit from explicit threat documentation. This document:

1. Lists plausible attacks against Fides
2. Explains why each is not prevented
3. Describes what Fides **does** provide against each attack
4. Helps implementers understand the protocol's guarantees and limits

---

## Attack Categories

### 1. Content-Based Attacks

#### 1.1 Toxic Record Attack

**Description:** A corrupt decider creates a formally valid DR authorizing a fraudulent expenditure. All fields are correct, signatures are valid, but the decision itself is illegitimate.

**Why not fully prevented:** Fides verifies structure, not merit. Evaluating decision content would require subjective judgment, destroying binary verification.

**What Fides provides:**
- The fraudulent DR is **public**
- Deciders are **identified** by `deciders_id`
- The record is **immutable** — cannot be hidden or altered
- The fraudster must **sign their own fraud**

**Consequence:** Fraud is possible, but deniability is not.

---

#### 1.2 Fragmentation Attack

**Description:** A large irregular expenditure (e.g., $1 billion) is split into many smaller DRs (e.g., 1000 x $1 million), each formally valid.

**Why not fully prevented:** Fides validates individual records, not spending patterns. Pattern detection requires analytics outside the protocol scope.

**What Fides provides:**
- All 1000 DRs are **public**
- All have **identified deciders**
- The pattern is **visible** to any third party
- Aggregation is **computable** by anyone

**Consequence:** Fragmentation cannot hide — only distribute attribution.

---

### 2. Infrastructure Attacks

#### 2.1 Anchor Capture Attack

**Description:** The Record Operator and Immutability Guardian collude to publish false anchors, hiding retroactive alterations.

**Why not fully prevented:** If both separated roles collude, the protocol's structural separation is violated. Fides cannot prevent violations of its own prerequisites.

**What Fides provides:**
- The protocol explicitly **requires** separation (Section 8.6)
- Violation is **detectable** by any third party recalculating hashes
- If detected, the protocol enters **fail-closed state** (Section 8.7)

**Consequence:** Collusion is possible, but detectable and invalidating.

---

#### 2.2 Total System Compromise

**Description:** An attacker gains control of all three roles (Record Operator, Immutability Guardian, Technical Auditor).

**Why not fully prevented:** Complete institutional capture is outside technical mitigation scope. No protocol survives total compromise.

**What Fides provides:**
- Assumes the existence of **at least one honest role** or independent external verifier
- All data remains **publicly verifiable**
- External parties can **independently audit** at any time

**Consequence:** Fides assumes adversarial environment, not omnipotent adversary.

---

### 3. Governance Attacks

#### 3.1 Political Circumvention

**Description:** Authorities refuse to adopt Fides, or adopt it partially, excluding certain payment types.

**Why not fully prevented:** Fides is a technical protocol, not a law. It cannot force adoption.

**What Fides provides:**
- Clear **compliance checklist** (Appendix C)
- Binary **compatibility definition** — partial adoption is not Fides-compatible
- Public **specification** enabling civil society pressure

**Consequence:** Adoption is political; the protocol only defines what compliance means.

---

#### 3.2 Exception Abuse

**Description:** Authorities create many SDRs (Special Decision Records) to bypass normal DR requirements.

**Why not fully prevented:** The protocol cannot prevent legitimate exception use. It can only make exceptions costly.

**What Fides provides:**
- Exceptions require **typed categories** — no generic "urgent" (Section 9.3)
- Exceptions require **more deciders** (Section 9.4)
- Exceptions require **expiration dates** (Section 9.4)
- All exceptions are **public and highlighted**

**Consequence:** Exception abuse is visible and attributable.

---

### 4. Identity Attacks

#### 4.1 Fake Decider Identity

**Description:** Attacker creates DRs using fabricated or stolen decider identities.

**Why not fully prevented:** Fides defines what fields a DR must have, not how identity systems work. Identity infrastructure is implementation-specific.

**What Fides provides:**
- `deciders_id` field is **required**
- `signatures` field enables **cryptographic verification**
- Implementation Notes specify format is **jurisdiction-defined**

**Consequence:** Identity security is an implementation concern, not a protocol concern.

---

### 5. Timing Attacks

#### 5.1 Anchor Window Attack

**Description:** Between anchor publications, alterations are made and then "corrected" before the next anchor.

**Why not fully prevented:** Fides requires frequent anchoring but doesn't specify exact frequency. Trade-off between cost and security window.

**What Fides provides:**
- Explicit guidance: "Rare anchor = fraud window" (Section 8.4)
- Append-only requirement makes alterations **structurally difficult**
- Multiple independent anchors can be used

**Consequence:** Anchor frequency is an implementation decision with explicit trade-off documentation.

---

## Summary Table

| Attack | Prevented? | Detectable? | What Fides Provides |
|--------|------------|-------------|---------------------|
| Toxic Record | No | Yes | Public, attributed, immutable trace |
| Fragmentation | No | Yes | Visible, aggregatable pattern |
| Anchor Capture | No | Yes | Detectable, invalidating |
| Total Compromise | No | Depends | Assumes non-omnipotent adversary |
| Political Circumvention | No | N/A | Clear compliance definition |
| Exception Abuse | No | Yes | Costly, visible, expiring |
| Fake Identity | No | Partially | Required fields, implementation-defined |
| Anchor Window | No | Partially | Explicit trade-off guidance |

---

## Design Philosophy

Fides follows a specific security philosophy:

> **Make attacks expensive, visible, and attributable — not impossible.**

The protocol does not attempt to:
- Prevent all fraud
- Replace human judgment
- Guarantee legal validity
- Survive total institutional capture

The protocol does guarantee:
- No payment without record
- Immutable, public history
- Identified deciders on every decision
- Verifiable by anyone, anytime

---

## Conclusion

> Fides does not stop corruption. It stops corruption from being silent.

This threat model documents the boundaries of that promise. Attacks outside these boundaries require solutions outside this protocol.

---

*Document version: 1.0*
*Companion to: FIDES-v0.1.md*

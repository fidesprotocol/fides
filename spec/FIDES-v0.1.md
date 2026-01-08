# Fides Protocol

**Version:** 0.1
**Status:** Frozen
**Nature:** Open technical standard
**License:** AGPLv3

---

## 1. Definition

The Fides Protocol is a minimal set of technical rules that conditions the execution of any public expenditure on the prior existence of a recorded, immutable, and verifiable decision.

**No valid record, no payment.**

---

## 2. Objective

Block the misuse of public funds **before** the money leaves, acting at the real point of power: the decision that authorizes the expenditure, not subsequent oversight.

Fides does not investigate, interpret, or judge. It permits or blocks execution.

---

## 3. Scope

This protocol defines exclusively:

- What constitutes a valid expenditure decision
- When a payment is formally authorized
- The minimum requirements for immutability and verifiability
- The logical interface for binary verification (yes/no)

**Nothing beyond this.**

### 3.1 Out of Scope (intentionally)

Fides does **not** address:

- Advanced transparency or visualizations
- Dashboards, rankings, or scores
- Artificial intelligence
- Analysis of merit, pricing, or opportunity
- Marketing, public narrative, or engagement
- National scale or massive integration

These elements are not necessary to block misuse.

---

## 4. Inviolable Principles

1. **No record, no payment**
2. **Expenditure becomes public at the decision, not at payment**
3. **Records are immutable and verifiable**
4. **Verification is binary and objective**
5. **Exceptions are rare, typed, and costly**
6. **Open source mandatory (AGPLv3)**
7. **Hard separation of functions and minimal access**
8. **Governance that prevents capture**

If any of these principles falls, the protocol fails.

---

## 5. Decision Record (DR)

### 5.1 Definition

A Decision Record (DR) is the technical artifact that potentially authorizes the future execution of a public expenditure.

- Without a valid DR, no payment can be executed
- The DR is created at the moment of administrative decision, never at payment

### 5.2 Nature of the Record

The Decision Record is:

- **Unique** — identified by `decision_id`
- **Immutable** — append-only
- **Chained** — hash of the previous record
- **Verifiable** — publicly accessible
- **Irrevocable by edit** — only by a new revocation record

Any attempt at retroactive alteration invalidates the record.

### 5.3 Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `decision_id` | UUID v4 | Unique identifier of the decision |
| `authority_id` | String | Unique identifier of the authority/agency |
| `deciders_id` | Array | IDs of the decision makers (non-empty) |
| `act_type` | Enum | `commitment`, `contract`, `amendment` |
| `maximum_value` | Decimal | Positive numeric value |
| `beneficiary` | String | Tax ID or entity identifier |
| `legal_basis` | String | Explicit legal reference |
| `decision_date` | DateTime | Date and time of the decision |
| `previous_record_hash` | String | Cryptographic hash of the previous record |
| `record_timestamp` | DateTime | Date and time of the record |
| `signatures` | Array | Identifiers of the signatories |

### 5.4 Validity Rules

A Decision Record is valid if, and only if:

1. All required fields are present
2. `maximum_value` > 0
3. `decision_date` <= `record_timestamp`
4. `previous_record_hash` corresponds to the last valid record
5. The record has not been revoked
6. The record respects the format and types defined in this protocol

**Failure in any item = invalid record.**

### 5.5 Immutability

It is **PROHIBITED** to:

- Update fields of an existing DR
- Delete records
- Correct records via editing

Corrections must occur exclusively through:

- A new Decision Record
- An explicit revocation record, referencing the original `decision_id`

**History is never erased.**

### 5.6 Cryptographic Chaining

Each Decision Record must contain:

```
previous_record_hash = HASH(immediately_previous_record)
```

- The hash function must be public, deterministic, and collision-resistant
- Breaking the chain invalidates all subsequent records

### 5.7 Publicity

Every Decision Record must be:

- Made public immediately after recording
- Available in machine-readable format (JSON)
- Accompanied by its hash

**A hidden record is an invalid record.**

---

## 6. Payment Authorization

### 6.1 Central Principle

The decider decides once. Execution only verifies.

After a valid DR exists:

- No decider "authorizes" payment
- No operator "releases" payment
- The system only verifies validity

### 6.2 Authorization Conditions

A payment P is authorized if, and only if, all conditions below are true:

1. A `decision_id` is provided
2. The corresponding DR is valid
3. The DR precedes the payment
4. The accumulated amount paid <= `maximum_value` of the DR
5. The beneficiary of the payment = beneficiary of the DR
6. The DR is not revoked
7. The payment is within the temporal and material scope of the DR

**Failure in any item = payment not authorized.**

### 6.3 Nature of Verification

Verification is:

- **Binary** — `true` / `false`
- **Deterministic** — same input, same output
- **Without interpretation** — does not evaluate merit
- **Without implicit exception** — there is no "almost valid"

### 6.4 Verification Interface

The protocol requires only one logical function:

```
isPaymentAuthorized(decision_id, payment) -> true | false
```

Implementations may expose this as an API, internal service, or local routine. The protocol does not define technology, only behavior.

### 6.5 Consequence

If verification returns `false`:

- The payment **MUST** be blocked
- There is no manual override
- There is no operational exception
- There is no human fallback

Any payment executed against a `false` response is proven irregular execution.

---

## 7. Immutability and External Anchor

### 7.1 Principle

The past cannot be altered, not even by those who control the system.

### 7.2 Structural Rule (Append-Only)

The system implementing Fides must operate in:

- Append-only mode
- No UPDATE
- No DELETE
- No overwrite

Any implementation that allows retroactive alteration is not compatible with Fides.

### 7.3 External Anchor

To prevent silent sabotage:

- The system must periodically publish a state hash to an external and independent medium
- Acceptable examples: public repository, DNS, public blockchain, independent timestamp service

Anchor requirements:

- External to the system
- Public
- Verifiable by third parties

### 7.4 Anchor Frequency

- The anchor must be published at regular intervals
- Rare anchor = fraud window
- Frequent anchor = high attack cost

### 7.5 Independent Verification

Any third party must be able to:

1. Recalculate hashes locally
2. Compare with public anchors
3. Detect any historical divergence

This does not require trust, only computation.

### 7.6 Technical Separation of Powers

No agent should simultaneously accumulate:

- Control of the database **AND**
- Control of the external anchor mechanism

This separation is mandatory.

### 7.7 Consequence

If immutability or chaining is broken:

- All authorizations become invalid until verifiable reconstruction

**Fail closed. Always.**

---

## 8. Typed Exceptions

### 8.1 Principle

Exception is permitted. Improvisation is not.

Fides admits exceptions, but only if they are:

- Pre-defined
- Recorded before payment
- Public
- Costly
- Auditable

**Exception without cost becomes rule.**

### 8.2 Special Decision Record (SDR)

An exception occurs exclusively as a Special Decision Record (SDR), created **before** payment.

Without a valid SDR, there is no exception.

### 8.3 Permitted Types (examples)

Each exception must belong to an explicit type:

- Public calamity
- Court order
- Health emergency
- Essential service continuity

**Generic types ("exceptional", "urgent", "special") are prohibited.**

### 8.4 Additional SDR Fields

In addition to the standard DR fields, every exception must contain:

| Field | Description |
|-------|-------------|
| `exception_type` | Typed category of the exception |
| `formal_justification` | Public text justifying |
| `maximum_term` | Expiration date (required) |
| `reinforced_deciders` | >= minimum deciders of common DR |

### 8.5 Structural Cost

Every exception must impose at least one additional cost:

- More deciders
- Highlighted publicity
- Short term
- Mandatory subsequent review
- Explicit marking as exception

### 8.6 Prohibitions

It is **PROHIBITED**:

- Verbal exception
- Retroactive exception
- Exception without term
- Exception without identified decider
- "Operational" exception in the database
- Exception outside the record

---

## 9. Revocations

### 9.1 Principle

Nothing is erased. Decisions are revoked, not corrected.

### 9.2 Revocation Record (RR)

A decision is revoked exclusively by a Revocation Record (RR), which:

- References the original `decision_id`
- Is chained and immutable
- Is public
- Has valid deciders

### 9.3 Effect of Revocation

After a valid RR:

- New payments based on the revoked DR are prohibited
- Already executed payments are not erased
- History remains intact

---

## 10. Minimum Governance

### 10.1 Principle

No person, position, or entity can control the entire protocol.

Fides does not depend on virtue, it depends on structure.

### 10.2 Mandatorily Separated Functions

Every implementation compatible with Fides must separate:

**1. Record Operator**
- Records decisions and exceptions
- Does not control immutability
- Does not control external anchor

**2. Immutability Guardian**
- Maintains the chaining mechanism
- Publishes external anchors
- Does not record decisions
- Does not execute payments

**3. Technical Auditor**
- Verifies history integrity
- Recalculates hashes
- Publishes independent reports
- Does not alter records

**No person or entity may accumulate more than one function.**

### 10.3 Access

All privileged access:

- Has defined scope
- Has validity period
- Is recorded

Permanent access is prohibited. Emergency access generates a public record.

### 10.4 Open Source

Every implementation of Fides must:

- Be published under AGPLv3 license
- Make complete source code available
- Make public any modifications

**Closed implementation is not compatible with Fides.**

### 10.5 Prohibitions

It is **PROHIBITED**:

- Single administrator
- Master key without distributed custody
- Dependence on exclusive vendor
- Opaque infrastructure control

### 10.6 Consequence

If governance fails:

- The protocol enters a state of operational distrust
- Execution must be suspended until verifiable recomposition

**Fail closed. Always.**

---

## 11. Nature of the Protocol

Fides:

- Is not a product
- Is not a platform
- Is not a transparency system
- Does not depend on specific people
- Does not require trust

It is a public technical rule, implementable by any entity, auditable by any person, and resistant to internal or external capture.

---

## 12. Success Criterion

The protocol is considered successful if:

> A malicious agent, even with internal access, cannot execute an irregular payment without leaving a verifiable trace.

Nothing beyond this defines success.

---

## 13. Evolution

This document defines the minimum version necessary to guarantee integrity in execution.

Any future evolution cannot violate the principles established here.

---

## 14. Normative Freeze Clause

This document constitutes the immutable normative core of the Fides Protocol (v0.1).

Any retroactive alteration to its principles, sections, rules, or invariants is prohibited.

Any modification, extension, or clarification may only occur through a new numbered version, preserving this text as a permanent historical reference.

Implementations, adaptations, or interpretations that contradict this core are not compatible with Fides, even if they use its name.

---

## Appendix A: Decision Record Schema (JSON)

```json
{
  "decision_id": "uuid-v4",
  "authority_id": "string",
  "deciders_id": ["string"],
  "act_type": "commitment | contract | amendment",
  "maximum_value": "decimal",
  "beneficiary": "string (tax ID / entity identifier)",
  "legal_basis": "string",
  "decision_date": "ISO 8601",
  "previous_record_hash": "string (SHA-256)",
  "record_timestamp": "ISO 8601",
  "signatures": ["string"]
}
```

---

## Appendix B: Verification Function (Pseudocode)

```
function isPaymentAuthorized(decision_id, payment):

    dr = findDecisionRecord(decision_id)

    if dr == null:
        return false

    if dr.revoked == true:
        return false

    if dr.valid == false:
        return false

    if payment.date < dr.decision_date:
        return false

    if payment.beneficiary != dr.beneficiary:
        return false

    total_paid = sumPreviousPayments(decision_id)

    if (total_paid + payment.value) > dr.maximum_value:
        return false

    return true
```

---

## Appendix C: Invariants (Compliance Checklist)

An implementation is compatible with Fides if:

- [ ] No payment is executed without a valid DR
- [ ] No retroactive alteration is possible
- [ ] No authorization is subjective
- [ ] No exception exists outside of record
- [ ] No decision exists without an identified decider
- [ ] No power is concentrated in a single role
- [ ] Source code is public (AGPLv3)
- [ ] A verifiable external anchor exists

---

**Fides Protocol**
*Built for trust.*

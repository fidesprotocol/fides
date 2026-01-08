# Fides Protocol — Architecture Reference

This document describes the architectural components and their relationships for implementations compatible with Fides Protocol v0.1.

---

## 1. Overview

Fides is a **protocol**, not a system. This document describes the logical architecture that any compliant implementation must follow.

```
┌─────────────────────────────────────────────────────────────────┐
│                        FIDES ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │   RECORD     │    │ IMMUTABILITY │    │   TECHNICAL  │      │
│  │   OPERATOR   │    │   GUARDIAN   │    │   AUDITOR    │      │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘      │
│         │                   │                   │               │
│         ▼                   ▼                   ▼               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    RECORD STORE                          │   │
│  │                   (Append-Only)                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│         │                   │                   │               │
│         ▼                   ▼                   ▼               │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │  DECISION    │    │   EXTERNAL   │    │   PUBLIC     │      │
│  │  RECORDS     │    │   ANCHOR     │    │   API        │      │
│  └──────────────┘    └──────────────┘    └──────────────┘      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Core Components

### 2.1 Record Store

The central data repository. Must be:

- **Append-only** — No UPDATE, no DELETE
- **Ordered** — Records maintain insertion sequence
- **Chained** — Each record contains hash of previous
- **Durable** — Data survives system failures

**Technology agnostic.** Can be implemented with:
- Append-only database
- Event log
- Blockchain
- Immutable file system

### 2.2 Record Operator

The component that creates records. Responsibilities:

- Accept decision data from authorized sources
- Validate required fields
- Calculate hash chain
- Insert into Record Store
- Return confirmation with record hash

**Constraints:**
- Cannot modify existing records
- Cannot delete records
- Cannot control external anchor

### 2.3 Immutability Guardian

The component that ensures tamper-evidence. Responsibilities:

- Maintain hash chain integrity
- Publish state hashes to external anchor
- Detect chain breaks
- Alert on integrity violations

**Constraints:**
- Cannot create decision records
- Cannot execute payments
- Must be independent from Record Operator

### 2.4 Technical Auditor

The component that verifies integrity. Responsibilities:

- Recalculate all hashes independently
- Compare with external anchors
- Publish audit reports
- Detect historical divergence

**Constraints:**
- Read-only access to Record Store
- Cannot modify anything
- Must be independent from other roles

### 2.5 External Anchor

An independent, public medium for integrity proof. Requirements:

- External to the system
- Publicly accessible
- Immutable or verifiable
- Time-stamped

**Examples:**
- Public Git repository
- DNS TXT records
- Public blockchain
- Independent timestamp authority

### 2.6 Public API

Interface for verification queries. Must expose:

```
isPaymentAuthorized(decision_id, payment) -> boolean
getDecisionRecord(decision_id) -> DR | null
verifyChainIntegrity(from, to) -> boolean
```

---

## 3. Separation of Functions

The protocol **mandates** separation. No single entity controls everything.

| Function | Can Do | Cannot Do |
|----------|--------|-----------|
| Record Operator | Create records | Modify records, control anchor |
| Immutability Guardian | Publish anchors | Create records, execute payments |
| Technical Auditor | Verify integrity | Modify anything |
| Payment Executor | Execute authorized payments | Create records, modify anchor |

### 3.1 Minimum Viable Separation

At minimum, two independent parties:

1. **Party A:** Record Operator + Payment Executor
2. **Party B:** Immutability Guardian + Technical Auditor

Party B must be truly independent — different organization, different infrastructure.

### 3.2 Recommended Separation

For high-value implementations:

1. **Government Agency:** Record Operator
2. **Independent Authority:** Immutability Guardian
3. **Civil Society / Press:** Technical Auditor
4. **Financial Institution:** Payment Executor

---

## 4. Data Flow

### 4.1 Decision Recording

```
1. Decider submits decision data
2. Record Operator validates fields
3. Record Operator calculates previous_record_hash
4. Record Operator inserts into Record Store
5. Record Operator returns decision_id + record_hash
6. Record becomes publicly accessible
```

### 4.2 Payment Authorization

```
1. Payment request arrives with decision_id
2. System calls isPaymentAuthorized(decision_id, payment)
3. Function retrieves DR from Record Store
4. Function validates all conditions (see spec section 6.2)
5. Function returns true or false
6. If false: payment BLOCKED (no override)
7. If true: payment proceeds
```

### 4.3 Anchor Publication

```
1. Immutability Guardian calculates state hash
2. State hash = HASH(all records up to current)
3. Guardian publishes to external anchor
4. Publication includes timestamp
5. Anyone can verify by recalculating
```

### 4.4 Integrity Audit

```
1. Technical Auditor fetches all records
2. Auditor recalculates hash chain
3. Auditor compares with external anchors
4. Auditor publishes report
5. Any divergence = integrity violation
```

---

## 5. Failure Modes

### 5.1 Chain Break Detected

**Cause:** Hash mismatch in record chain.

**Response:**
- All authorizations suspended
- Alert all parties
- Investigation required
- No payments until resolution

### 5.2 Anchor Mismatch

**Cause:** Recalculated hash differs from published anchor.

**Response:**
- Same as chain break
- External anchor is source of truth
- System data is suspect

### 5.3 Guardian Unavailable

**Cause:** Immutability Guardian offline.

**Response:**
- Record creation can continue (temporarily)
- Anchor publication paused
- Time window recorded
- Must resume before window exceeds policy

### 5.4 Governance Failure

**Cause:** Single entity gains control of multiple functions.

**Response:**
- Protocol enters distrust state
- Execution suspended
- Requires governance recomposition

---

## 6. Security Considerations

### 6.1 Attack Vectors

| Attack | Mitigation |
|--------|------------|
| Record tampering | Hash chain + external anchor |
| Anchor tampering | Multiple anchors + public verification |
| Insider collusion | Function separation + independent auditor |
| Key compromise | Distributed custody + rotation policy |
| System takeover | Open source + multiple implementations |

### 6.2 Cryptographic Requirements

- Hash function: SHA-256 minimum (or equivalent security)
- Signatures: Ed25519 or RSA-2048 minimum
- All cryptographic choices must be public

### 6.3 Access Control

- No permanent privileged access
- All access time-limited
- All access logged
- Emergency access requires public record

---

## 7. Implementation Notes

### 7.1 Technology Freedom

Fides specifies **behavior**, not technology. Implementations may use:

- Any programming language
- Any database (if append-only enforced)
- Any cloud or on-premise infrastructure
- Any cryptographic library (if standards met)

### 7.2 Interoperability

Different implementations should:

- Use same JSON schema for records
- Use same hash algorithm
- Expose compatible verification API
- Publish anchors in verifiable format

### 7.3 Testing Compliance

An implementation is compliant if it passes:

1. **Immutability test:** Cannot modify past records
2. **Chain test:** Break detection works
3. **Authorization test:** False conditions block payment
4. **Anchor test:** External verification succeeds
5. **Separation test:** Functions are truly independent

---

## 8. Reference Diagram

```
                    EXTERNAL WORLD
                         │
    ┌────────────────────┼────────────────────┐
    │                    │                    │
    ▼                    ▼                    ▼
┌────────┐        ┌────────────┐        ┌─────────┐
│DECIDER │        │  PAYMENT   │        │ PUBLIC  │
│        │        │  SYSTEM    │        │ AUDITOR │
└───┬────┘        └─────┬──────┘        └────┬────┘
    │                   │                    │
    │ submit            │ verify             │ audit
    │ decision          │ payment            │ integrity
    ▼                   ▼                    ▼
┌─────────────────────────────────────────────────┐
│                   FIDES LAYER                   │
│  ┌─────────────────────────────────────────┐   │
│  │            RECORD STORE                 │   │
│  │         (append-only chain)             │   │
│  └─────────────────────────────────────────┘   │
│                      │                         │
│                      ▼                         │
│  ┌─────────────────────────────────────────┐   │
│  │          EXTERNAL ANCHOR                │   │
│  │    (public, independent, verifiable)    │   │
│  └─────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

---

*This document is part of Fides Protocol reference materials.*

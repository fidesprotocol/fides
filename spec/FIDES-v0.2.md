# Fides Protocol

**Version:** 0.2
**Status:** Active
**Nature:** Open technical standard
**License:** AGPLv3
**Supersedes:** v0.1

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

## 4. Limitations and Non-Goals

This section explicitly documents what the Fides Protocol does **not** do. These are not oversights — they are deliberate design boundaries.

### 4.1 What Fides Does NOT Do

**Does not validate decision content**

Fides verifies that a Decision Record exists and is structurally valid. It does not evaluate whether the decision itself is wise, legal, or legitimate. A formally valid DR authorizing a fraudulent expenditure will pass verification.

**Does not replace human judgment**

The protocol does not decide what should be funded, at what price, or for what purpose. These remain political and administrative decisions made by humans before the DR is created.

**Does not guarantee legal validity**

Technical immutability does not automatically confer legal standing. Courts and legal systems in each jurisdiction determine whether cryptographic records constitute valid evidence.

**Does not prevent collusion**

If all separated roles (Record Operator, Immutability Guardian, Technical Auditor) collude, the protocol can be subverted. Fides assumes at least one honest actor or vigilant third party. No technical system can guarantee integrity under total collusion and absence of independent verification.

**Does not eliminate fraud**

Fraud can still occur. What Fides guarantees is that fraud leaves a **permanent, public, attributed trace**. The fraudster must sign their own fraud.

### 4.2 Deliberate Trade-offs

| Trade-off | Rationale |
|-----------|-----------|
| No merit analysis | Keeps verification objective and binary |
| No human override | Prevents "emergency" exceptions from becoming routine |
| No content inspection | Avoids interpretive disputes and subjective blocking |
| Depends on adoption | Political acceptance is required; protocol cannot force itself |

### 4.3 What Fides DOES Guarantee

Despite its limitations, the protocol provides hard guarantees:

1. **No payment without record** — Structurally enforced
2. **Immutable history** — Past cannot be altered
3. **Public attribution** — Every decision has identified deciders
4. **Verifiable by anyone** — No trust required, only computation
5. **Detectable tampering** — Any chain break is visible

### 4.4 The Core Promise

> Fides does not stop corruption. It stops corruption from being silent.

Any irregular payment executed under Fides leaves a permanent, verifiable, attributed record. The protocol makes **deniability impossible**, not fraud impossible.

---

## 5. Inviolable Principles

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

## 6. Decision Record (DR)

### 6.1 Definition

A Decision Record (DR) is the technical artifact that potentially authorizes the future execution of a public expenditure.

- Without a valid DR, no payment can be executed
- The DR is created at the moment of administrative decision, never at payment

### 6.2 Nature of the Record

The Decision Record is:

- **Unique** — identified by `decision_id`
- **Immutable** — append-only
- **Chained** — hash of the previous record
- **Verifiable** — publicly accessible
- **Irrevocable by edit** — only by a new revocation record

Any attempt at retroactive alteration invalidates the record.

### 6.3 Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `decision_id` | UUID v4 | Unique identifier of the decision |
| `authority_id` | String | Unique identifier of the authority/agency |
| `deciders_id` | Array | IDs of the decision makers (non-empty) |
| `act_type` | Enum | Type of administrative act (see 6.3.1) |
| `currency` | String | ISO 4217 currency code (e.g., USD, EUR, BRL) |
| `maximum_value` | Decimal | Positive numeric value in the specified currency |
| `beneficiary` | String | Tax ID or entity identifier (format varies by jurisdiction) |
| `legal_basis` | String | Explicit legal reference |
| `decision_date` | DateTime | Date and time of the decision (ISO 8601) |
| `previous_record_hash` | String | Cryptographic hash of the previous record (SHA-256) |
| `record_timestamp` | DateTime | Date and time of the record (ISO 8601), externally attested |
| `signatures` | Array | Cryptographic signatures (see 6.3.2) |

#### 6.3.1 Act Types (Mandatory Taxonomy)

The `act_type` field MUST be one of the following:

| Type | Description |
|------|-------------|
| `commitment` | Initial budget commitment |
| `contract` | Formal contract execution |
| `amendment` | Modification to existing contract (requires `references` field) |
| `purchase_order` | Direct purchase order |
| `grant` | Grant or subsidy allocation |
| `payroll` | Personnel payment authorization |
| `reimbursement` | Expense reimbursement |
| `transfer` | Inter-agency transfer |

**Amendment constraint:** An `amendment` type DR MUST include a `references` field pointing to the original `decision_id` and cannot increase `maximum_value` by more than 25% of the original without a new `contract` type DR.

Jurisdictions MAY define additional types, but MUST publish their extended taxonomy publicly. Extended types MUST NOT overlap semantically with the mandatory types above.

#### 6.3.2 Cryptographic Signatures (Mandatory)

The `signatures` field MUST contain cryptographic signatures that are:

- **Verifiable** — Can be validated using public key cryptography
- **Non-repudiable** — Signer cannot deny having signed
- **Timestamped** — Signature includes timestamp

Acceptable algorithms:
- Ed25519 (recommended)
- ECDSA with P-256 or P-384
- RSA-PSS with minimum 2048-bit key

Each signature MUST include:

```json
{
  "signer_id": "string (matches deciders_id element)",
  "public_key": "string (base64)",
  "algorithm": "string (Ed25519 | ECDSA-P256 | ECDSA-P384 | RSA-PSS)",
  "signature": "string (base64)",
  "signed_at": "ISO 8601"
}
```

**Binding Rule:** Every identifier listed in `deciders_id` MUST correspond to exactly one valid cryptographic signature in `signatures`. Unmatched deciders, invalid signatures, or unverifiable signatures invalidate the record.

### 6.4 Validity Rules

A Decision Record is valid if, and only if:

1. All required fields are present
2. `maximum_value` > 0
3. `decision_date` <= `record_timestamp`
4. **`record_timestamp` - `decision_date` <= 72 hours** (maximum registration delay)
5. `previous_record_hash` corresponds exactly to the last valid record
6. The record has not been revoked
7. The record respects the format and types defined in this protocol
8. `authority_id` refers to an authority that exists at `decision_date`
9. All signatures are cryptographically valid
10. `record_timestamp` is attested by an independent time authority

**Failure in any item = invalid record.**

**Registration Delay Rule (6.4.4):** The maximum allowed time between `decision_date` and `record_timestamp` is 72 hours. Decisions not registered within this window require a Special Decision Record (SDR) with `exception_type: late_registration` and formal justification. This prevents silent backdating of decisions.

### 6.5 Immutability

It is **PROHIBITED** to:

- Update fields of an existing DR
- Delete records
- Correct records via editing

Corrections must occur exclusively through:

- A new Decision Record
- An explicit revocation record, referencing the original `decision_id`

**History is never erased.**

### 6.6 Cryptographic Chaining

Each Decision Record must contain:

```
previous_record_hash = SHA-256(canonical_serialization(immediately_previous_record))
```

#### 6.6.1 Canonical Serialization

Records MUST be serialized using the following canonical format before hashing:

1. JSON format
2. UTF-8 encoding
3. Keys sorted alphabetically (recursive)
4. No whitespace between elements
5. No trailing newline
6. Numbers without unnecessary precision (no trailing zeros)
7. Dates in ISO 8601 format with UTC timezone (Z suffix)

Example:
```json
{"authority_id":"BR-GOV-001","currency":"BRL","decision_date":"2025-01-15T10:00:00Z","decision_id":"550e8400-e29b-41d4-a716-446655440000","deciders_id":["CPF-12345678901"],"legal_basis":"Lei 8.666/1993 Art. 24","maximum_value":10000.00,"previous_record_hash":"abc123...","record_timestamp":"2025-01-15T14:30:00Z","signatures":[...]}
```

- The hash function MUST be SHA-256
- Breaking the chain invalidates all subsequent records
- Implementations MUST provide a canonical serialization function for verification

#### 6.6.2 Chain Integrity

There MUST be exactly ONE authoritative chain. To prevent chain divergence:

1. The Immutability Guardian MUST maintain a single append-only ledger
2. Each new record MUST reference the immediately previous record's hash
3. External anchors MUST include a chain height (record count) alongside the state hash
4. Any divergence between chain height and record count indicates tampering

### 6.7 Publicity

Every Decision Record must be:

- Made public immediately after recording
- Available in machine-readable format (JSON)
- Accompanied by its hash

**A hidden record is an invalid record.**

**Definition of Public:** Public means accessible without authentication, free of charge, and readable using open standards.

### 6.8 Finality

Once a Decision Record is recorded and publicly anchored, it is final.

Its legal, administrative, or political contestation does not suspend its technical validity unless a formal Revocation Record exists.

**No limbo. No provisional state. Recorded means final.**

### 6.9 Timestamp Attestation

The `record_timestamp` MUST be attested by an independent time authority. Acceptable methods:

1. **RFC 3161 Timestamp** — From a trusted TSA (Time Stamping Authority)
2. **Blockchain timestamp** — Bitcoin, Ethereum, or other public blockchain with sufficient confirmation
3. **Multiple independent NTP sources** — Minimum 3 sources, median value used

The Record Operator CANNOT be the sole authority on `record_timestamp`. Attestation proof MUST be stored alongside the record.

#### 6.9.1 Attestation Proof Formats (Mandatory)

The `timestamp_attestation.proof` field MUST follow the exact format for each method:

**RFC 3161 Format:**
```json
{
  "method": "rfc3161",
  "proof": {
    "tsa_url": "string (URL of the TSA)",
    "tsa_certificate": "string (base64 DER-encoded X.509 certificate)",
    "timestamp_token": "string (base64 DER-encoded TimeStampToken per RFC 3161)",
    "hash_algorithm": "SHA-256",
    "message_imprint": "string (hex-encoded hash of the record)"
  },
  "sources": ["string (TSA identifier)"]
}
```
**Validation:** Verify the TimeStampToken signature against the TSA certificate, and verify the message_imprint matches the canonical hash of the record.

**Blockchain Format:**
```json
{
  "method": "blockchain",
  "proof": {
    "chain": "bitcoin | ethereum | other",
    "network": "mainnet | testnet",
    "block_number": "integer",
    "block_hash": "string (hex)",
    "transaction_id": "string (hex)",
    "merkle_proof": ["string (hex, proof path)"],
    "data_hash": "string (hex, hash embedded in transaction)",
    "confirmations_at_record": "integer (minimum 6 for Bitcoin, 12 for Ethereum)"
  },
  "sources": ["string (block explorer URLs for verification)"]
}
```
**Validation:** Verify the transaction exists in the specified block, verify the merkle proof, verify the data_hash matches the canonical hash of the record, verify minimum confirmations.

**NTP Consensus Format:**
```json
{
  "method": "ntp_consensus",
  "proof": {
    "samples": [
      {
        "server": "string (NTP server hostname)",
        "stratum": "integer (1-15)",
        "timestamp": "ISO 8601",
        "offset_ms": "integer (offset from local clock)",
        "delay_ms": "integer (round-trip delay)"
      }
    ],
    "consensus_timestamp": "ISO 8601 (median of samples)",
    "max_divergence_ms": "integer (maximum divergence between samples)"
  },
  "sources": ["string (NTP server hostnames)"]
}
```
**Validation:** Verify minimum 3 samples from different stratum-1 or stratum-2 servers, verify max_divergence_ms <= 1000 (1 second), verify consensus_timestamp is the median.

**TSA Requirements:** For RFC 3161, the TSA MUST be external to the implementing jurisdiction. Government-controlled TSAs from the same jurisdiction are NOT acceptable. Acceptable TSAs include: commercial TSA services, TSAs from other countries, or decentralized timestamp services.

#### 6.9.2 Timestamp Validation Procedures (Mandatory)

Implementations MUST perform rigorous validation of timestamp attestations. Weak validation defeats the purpose of external timestamping.

##### 6.9.2.1 RFC 3161 Validation Procedure

For RFC 3161 timestamp tokens, implementations MUST:

1. **Parse TimeStampToken** — Decode the base64 DER-encoded token
2. **Extract signer certificate** — Get the TSA certificate from the token
3. **Verify certificate chain:**
   - Build chain up to a trusted root CA
   - Verify all certificates in chain are not expired
   - Check Certificate Revocation List (CRL) or OCSP for each certificate
   - Verify TSA certificate has Extended Key Usage = `id-kp-timeStamping` (OID 1.3.6.1.5.5.7.3.8)
4. **Verify token signature** — Using the TSA's public key
5. **Extract and verify message imprint:**
   - Extract `messageImprint` from the token
   - Compute `SHA256(canonical_serialization(record))`
   - Verify they match exactly
6. **Verify genTime** — The timestamp MUST be within ±24 hours of current time at verification (allows for network delays and clock drift)
7. **Verify TSA identity** — TSA MUST NOT be controlled by the implementing jurisdiction

**Failure in any step = attestation INVALID.**

##### 6.9.2.2 Blockchain Validation Procedure

For blockchain timestamps, implementations MUST:

1. **Verify block existence** — Query at least 2 independent block explorers or full nodes
2. **Verify transaction inclusion:**
   - Transaction ID exists in specified block
   - Merkle proof is valid (reconstruct merkle root from proof path)
3. **Verify data hash:**
   - Extract embedded data from transaction (OP_RETURN for Bitcoin, input data for Ethereum)
   - Verify `data_hash` matches `SHA256(canonical_serialization(record))`
4. **Verify confirmations:**
   - Bitcoin: minimum 6 confirmations
   - Ethereum: minimum 12 confirmations
   - Other chains: as specified in implementation documentation
5. **Verify block timestamp:**
   - Block timestamp MUST be <= `record_timestamp`
   - Block timestamp MUST be >= `record_timestamp - 24 hours`
6. **Handle reorgs:**
   - If block is orphaned after initial verification, attestation becomes PENDING
   - Re-verify when chain stabilizes

**Failure in any step = attestation INVALID.**

##### 6.9.2.3 NTP Consensus Validation Procedure

For NTP consensus timestamps, implementations MUST:

1. **Verify sample count** — Minimum 3 samples required
2. **Verify server diversity:**
   - All samples from different NTP servers
   - At least 2 different network operators
   - No samples from servers controlled by implementing jurisdiction
3. **Verify stratum:**
   - All samples from stratum-1 or stratum-2 servers
   - Stratum > 2 samples are rejected
4. **Verify divergence:**
   - Maximum divergence between any two samples <= 1000ms (1 second)
   - If divergence exceeds limit, NTP attestation is INVALID
5. **Verify consensus calculation:**
   - `consensus_timestamp` MUST equal median of all sample timestamps
   - Recalculate median and verify match
6. **Verify offset reasonableness:**
   - All `offset_ms` values should be within ±100ms of each other
   - Large offset variance indicates potential manipulation

**Failure in any step = attestation INVALID.**

**NTP Security Note:** NTP consensus is the weakest attestation method. Implementations SHOULD prefer RFC 3161 or blockchain when available. NTP is acceptable only when stronger methods are unavailable.

---

## 7. Payment Authorization

### 7.1 Central Principle

The decider decides once. Execution only verifies.

After a valid DR exists:

- No decider "authorizes" payment
- No operator "releases" payment
- The system only verifies validity

### 7.2 Authorization Conditions

A payment P is authorized if, and only if, all conditions below are true:

1. A `decision_id` is provided
2. The corresponding DR is valid
3. The DR precedes the payment (`decision_date` < payment date)
4. The accumulated amount paid <= `maximum_value` of the DR
5. The payment beneficiary = DR beneficiary
6. The payment currency = DR currency
7. The DR has not been revoked
8. **If DR is an SDR: `payment_date` <= `maximum_term`**

**Failure in any item = payment not authorized.**

**Payment Atomicity:** Every payment, partial or total, is subject to independent verification under this protocol. Splitting payments does not bypass `maximum_value` constraints.

**Definition of Payment Date:** For the purpose of this protocol, payment date refers to the moment of irreversible execution of funds.

### 7.3 Nature of Verification

Verification is:

- **Binary** — `true` / `false`
- **Deterministic** — same input, same output
- **Without interpretation** — does not evaluate merit
- **Without implicit exception** — there is no "almost valid"

### 7.4 Verification Interface

The protocol requires only one logical function:

```
isPaymentAuthorized(decision_id, payment) -> true | false
```

Implementations may expose this as an API, internal service, or local routine. The protocol does not define technology, only behavior.

### 7.5 Consequence

If verification returns `false`:

- The payment **MUST** be blocked
- There is no manual override
- There is no operational exception
- There is no human fallback

Any payment executed against a `false` response is proven irregular execution.

### 7.6 Payment Serialization (Concurrency Control)

Payments against the same `decision_id` MUST be processed serially to prevent race conditions.

Implementation requirements:

1. Acquire exclusive lock on `decision_id` before verification
2. Verify payment authorization
3. Record payment if authorized
4. Release lock

**No parallel processing of payments against the same DR.**

Implementations MAY use database transactions, distributed locks, or other serialization mechanisms, but MUST guarantee that `sumPreviousPayments()` is always accurate at verification time.

### 7.7 Payment Ledger

Every Payment Executor MUST maintain an immutable, append-only payment ledger that records ALL payment authorization requests, whether authorized or rejected.

#### 7.7.1 Payment Ledger Entry Fields

| Field | Type | Description |
|-------|------|-------------|
| `payment_id` | UUID v4 | Unique identifier for this payment attempt |
| `decision_id` | UUID v4 | Reference to the DR being paid against |
| `payment_amount` | Decimal | Amount requested |
| `payment_currency` | String | ISO 4217 currency code |
| `payment_beneficiary` | String | Intended recipient identifier |
| `request_timestamp` | DateTime | When authorization was requested (ISO 8601) |
| `authorization_result` | Boolean | Result of `isPaymentAuthorized()` |
| `rejection_reason` | String | If rejected, the specific rule that failed (nullable) |
| `execution_timestamp` | DateTime | When payment was executed (nullable if rejected) |
| `payment_executor_id` | String | ID of the Payment Executor |
| `previous_payment_hash` | String | SHA-256 hash of previous payment ledger entry |
| `signatures` | Array | Cryptographic signatures of Payment Executor |

#### 7.7.2 Payment Ledger Requirements

1. **Append-only** — No UPDATE or DELETE operations permitted
2. **Chained** — Each entry includes hash of previous entry
3. **Complete** — Both authorized AND rejected payments MUST be recorded
4. **Public** — Ledger MUST be publicly accessible (excluding sensitive beneficiary details if jurisdiction requires)
5. **Machine-readable** — JSON format required
6. **Timestamped** — All entries MUST have externally attested timestamps (same methods as DR)

#### 7.7.3 Rejected Payment Recording

Rejected payments are critical for audit purposes. They prove that:
- An attempt was made
- The system correctly blocked it
- The rejection reason is documented

Without rejected payment records, there is no proof that the system is actually blocking unauthorized payments.

#### 7.7.4 Payment Ledger Reconciliation

The payment ledger MUST be reconcilable against the Decision Record ledger:

```
For each decision_id:
    sum(authorized_payments) <= dr.maximum_value
```

Any violation indicates either:
- Payment Executor malfunction
- Ledger tampering
- Race condition failure

See Section 11.9 for Technical Auditor reconciliation procedures.

---

## 8. Immutability and External Anchor

### 8.1 Principle

The past cannot be altered, not even by those who control the system.

### 8.2 Structural Rule (Append-Only)

The system implementing Fides must operate in:

- Append-only mode
- No UPDATE
- No DELETE
- No overwrite

Any implementation that allows retroactive alteration is not compatible with Fides.

### 8.3 External Anchor

To prevent silent sabotage:

- The system must periodically publish a state hash to an external and independent medium
- Acceptable examples: public blockchain, independent timestamp service, multiple DNS TXT records

Anchor requirements:

- External to the system
- Public
- Verifiable by third parties
- Outside the administrative control of the Record Operator
- Includes chain height (record count)

### 8.4 Anchor Frequency

- The anchor MUST be published at minimum every 24 hours
- The anchor SHOULD be published after every 100 records or 1 hour, whichever comes first
- Rare anchor = fraud window
- Frequent anchor = high attack cost

**Maximum anchor interval: 24 hours.** Any implementation with longer intervals is not Fides-compatible.

### 8.5 Independent Verification

Any third party must be able to:

1. Recalculate hashes locally using canonical serialization
2. Compare with public anchors (including chain height)
3. Detect any historical divergence

This does not require trust, only computation.

### 8.6 Technical Separation of Powers

No agent should simultaneously accumulate:

- Control of the database **AND**
- Control of the external anchor mechanism

This separation is mandatory.

### 8.7 Consequence

If immutability or chaining is broken:

- All authorizations become invalid until verifiable reconstruction

**Fail closed. Always.**

---

## 9. Typed Exceptions

### 9.1 Principle

Exception is permitted. Improvisation is not.

Fides admits exceptions, but only if they are:

- Pre-defined
- Recorded before payment
- Public
- Costly
- Auditable

**Exception without cost becomes rule.**

### 9.2 Special Decision Record (SDR)

An exception occurs exclusively as a Special Decision Record (SDR), created **before** payment.

Without a valid SDR, there is no exception.

### 9.3 Permitted Exception Types (Exhaustive)

The `exception_type` field MUST be one of the following:

| Type | Description | Maximum Term |
|------|-------------|--------------|
| `public_calamity` | Natural disaster, declared emergency | 90 days |
| `court_order` | Judicial mandate requiring immediate action | As specified by court |
| `health_emergency` | Imminent threat to public health | 30 days |
| `essential_service` | Prevent interruption of critical service | 15 days |
| `national_security` | Classified defense/security matter | 180 days |
| `late_registration` | Decision registered after 72h window | N/A (one-time) |

**Generic types ("exceptional", "urgent", "special", "other") are PROHIBITED.**

**No new exception types may be created by implementations.** New types require a protocol version update.

### 9.4 Additional SDR Fields

In addition to the standard DR fields, every exception must contain:

| Field | Type | Description |
|-------|------|-------------|
| `exception_type` | Enum | One of the permitted types (9.3) |
| `formal_justification` | String | Public text justifying (minimum 100 characters) |
| `maximum_term` | DateTime | Expiration date (required, must respect type limits) |
| `reinforced_deciders` | Array | >= 2x minimum deciders of common DR |
| `oversight_authority` | String | ID of authority that must review this SDR |

### 9.5 Structural Cost

Every exception MUST impose ALL of the following costs:

1. More deciders (minimum 2x normal requirement)
2. Highlighted publicity (marked as exception in all public views)
3. Term limit (cannot exceed type maximum)
4. Mandatory review by `oversight_authority` within 30 days
5. Explicit marking as exception in all systems

### 9.6 Prohibitions

It is **PROHIBITED**:

- Verbal exception
- Retroactive exception (except `late_registration` type)
- Exception without term
- Exception without identified decider
- "Operational" exception in the database
- Exception outside the record
- Consecutive SDRs for the same purpose (must wait 30 days)
- SDR with term exceeding type maximum
- SDR without `oversight_authority`

### 9.7 SDR Verification

The `isPaymentAuthorized` function MUST verify SDR expiration:

```
if dr.is_sdr == true:
    if payment.date > dr.maximum_term:
        return false
```

---

## 10. Revocations

### 10.1 Principle

Nothing is erased. Decisions are revoked, not corrected.

### 10.2 Revocation Record (RR)

A decision is revoked exclusively by a Revocation Record (RR), which:

- References the original `decision_id`
- Is chained and immutable
- Is public
- Has valid deciders with revocation authority

### 10.3 Revocation Authority

A Revocation Record is valid only if signed by deciders who have **revocation authority** over the original DR. Valid revokers are:

1. **Original deciders** — Any decider listed in the original DR's `deciders_id`
2. **Hierarchical superior** — A decider with documented authority over the original deciders (see 10.3.1)
3. **Oversight authority** — The `oversight_authority` specified in an SDR
4. **Judicial authority** — A court with jurisdiction (requires `court_order` reference)

#### 10.3.1 Hierarchical Superior Definition

A "hierarchical superior" claim is valid only if:

1. **Public organization chart exists** — The authority (`authority_id`) MUST publish a machine-readable organization chart (JSON format) listing all positions and their reporting relationships
2. **Superior relationship is documented** — The revoker's position MUST be a direct or indirect superior of at least one original decider's position in the published chart
3. **Chart is externally timestamped** — The organization chart MUST have an external timestamp proof (RFC 3161 or blockchain) demonstrating it was published BEFORE the original DR was created
4. **Reference is included** — The RR MUST include an `org_chart_reference` field with:

```json
{
  "org_chart_url": "string (URL to published organization chart)",
  "org_chart_hash": "string (SHA-256 of the chart at time of original DR)",
  "org_chart_timestamp_proof": {
    "method": "rfc3161 | blockchain",
    "proof": "object (same format as timestamp_attestation.proof)"
  },
  "revoker_position": "string (position ID in the chart)",
  "superior_path": ["string (chain of positions from revoker to original decider)"]
}
```

**Validation:**
- The superior relationship MUST be verifiable by traversing the published organization chart
- The `org_chart_timestamp_proof` MUST prove the chart existed before the original DR's `record_timestamp`
- Ad-hoc claims of superiority without documented and timestamped evidence are NOT valid

**Anti-falsification:** The external timestamp requirement prevents retroactive modification of organization charts. Without cryptographic proof of when the chart was published, hierarchical superior claims are invalid.

The RR MUST include:

| Field | Type | Description |
|-------|------|-------------|
| `revocation_id` | UUID v4 | Unique identifier |
| `target_decision_id` | UUID v4 | The DR being revoked |
| `revocation_type` | Enum | `voluntary`, `oversight`, `judicial`, `superseded` |
| `revocation_reason` | String | Public explanation (minimum 50 characters) |
| `revoker_authority` | String | Basis for revocation authority |
| `revoker_id` | Array | IDs of those revoking |
| `signatures` | Array | Cryptographic signatures of revokers |

### 10.4 Effect of Revocation

After a valid RR:

- New payments based on the revoked DR are prohibited
- Already executed payments are not erased
- History remains intact
- Revocation does not retroactively legitimize payments executed before the revocation

### 10.5 Revocation Verification

Before accepting an RR, implementations MUST verify:

1. `target_decision_id` exists
2. At least one `revoker_id` has valid authority (per 10.3)
3. All signatures are cryptographically valid
4. The RR is properly chained

---

## 11. Minimum Governance

### 11.1 Principle

No person, position, or entity can control the entire protocol.

Fides does not depend on virtue, it depends on structure.

### 11.2 Mandatorily Separated Functions

Every implementation compatible with Fides must separate:

**1. Record Operator**
- Records decisions and exceptions
- Does not control immutability
- Does not control external anchor
- Does not execute payments

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
- Does not execute payments

**4. Payment Executor**
- Calls `isPaymentAuthorized()` before any payment
- Executes authorized payments
- Records payment execution in payment ledger
- Does not record decisions
- Does not control immutability
- Does not control external anchor

**No person or entity may accumulate more than one function.**

Control includes direct or indirect authority, contractual dominance, or technical dependency.

**Payment Executor Independence:** The Payment Executor role is critical because it controls actual fund movement. If the Payment Executor is controlled by the same entity as the Record Operator, that entity could:
1. Create a DR
2. Execute payment immediately
3. Revoke the DR

This would defeat the purpose of the protocol. Therefore, Payment Executor MUST be institutionally separate from all other roles.

### 11.3 Separation Verification Criteria

To verify genuine separation, implementations MUST document:

| Criterion | Requirement |
|-----------|-------------|
| Legal independence | Different legal entities |
| Administrative independence | No shared management or board members |
| Technical independence | No shared infrastructure or credentials |
| Financial independence | No funding dependency between roles |
| Contractual independence | No service contracts between roles |

**Annual audit required:** An independent party must verify separation criteria annually. Audit results must be public.

### 11.4 Access

All privileged access:

- Has defined scope
- Has validity period (maximum 1 year)
- Is recorded in an access log
- Requires multi-party authorization for sensitive operations

Permanent access is prohibited. Emergency access generates a public record.

### 11.5 Open Source

Every implementation of Fides must:

- Be published under AGPLv3 license
- Make complete source code available
- Make public any modifications

**Closed implementation is not compatible with Fides.**

Publishing partial source code, mock implementations, or non-functional stubs does not satisfy the open-source requirement.

### 11.6 Compliance Testing

Every implementation MUST pass the official Fides Compliance Test Suite before claiming compatibility.

The test suite verifies:

1. Append-only behavior (no UPDATE/DELETE possible)
2. Canonical serialization correctness
3. Signature verification
4. Chain integrity
5. Payment atomicity
6. SDR expiration enforcement
7. Revocation authority verification
8. Timestamp attestation proof validation

**Test suite repository:** https://github.com/fidesprotocol/compliance-tests

**Test suite status:** The compliance test suite is a normative requirement of this specification. Until the official test suite is published:

1. Implementations MUST self-certify against all items in Appendix F (Invariants)
2. Implementations MUST publish their self-certification methodology
3. Self-certification does NOT constitute full compliance — it is a transitional measure
4. Once the official test suite is available, all implementations MUST re-certify within 90 days

Implementations that do not pass all tests are NOT Fides-compatible, regardless of claims.

### 11.7 Prohibitions

It is **PROHIBITED**:

- Single administrator
- Master key without distributed custody
- Dependence on exclusive vendor
- Opaque infrastructure control

### 11.8 Consequence

If governance fails:

- The protocol enters a state of operational distrust
- Execution must be suspended until verifiable recomposition

**Fail closed. Always.**

### 11.9 Technical Auditor Reconciliation

The Technical Auditor MUST perform full reconciliation at least monthly. This section specifies the exact algorithm.

#### 11.9.1 Reconciliation Inputs

The Technical Auditor receives:
1. **Decision Record ledger** — All DRs, SDRs, and RRs
2. **Payment ledger** — All payment attempts (authorized and rejected)
3. **External anchors** — All published state hashes and chain heights

#### 11.9.2 Reconciliation Algorithm

```
function performAuditReconciliation():

    report = new AuditReport()

    // Phase 1: Verify Decision Record chain
    for each dr in decision_record_ledger:

        // 1a. Verify signature
        if not verifySignatures(dr):
            report.addAlert("INVALID_SIGNATURE", dr.decision_id)

        // 1b. Verify timestamp attestation
        if not verifyTimestampAttestation(dr):
            report.addAlert("INVALID_TIMESTAMP", dr.decision_id)

        // 1c. Verify chain hash
        expected_hash = computeHash(previous_record)
        if dr.previous_record_hash != expected_hash:
            report.addAlert("CHAIN_BREAK", dr.decision_id)

        // 1d. Verify registration delay
        if (dr.record_timestamp - dr.decision_date) > 72_hours:
            if not dr.is_sdr or dr.exception_type != "late_registration":
                report.addAlert("REGISTRATION_DELAY_EXCEEDED", dr.decision_id)

    // Phase 2: Verify payments against DRs
    for each decision_id in unique(payment_ledger.decision_id):

        dr = findDecisionRecord(decision_id)
        payments = findPayments(decision_id)

        // 2a. Check if DR exists
        if dr == null:
            report.addAlert("PAYMENT_WITHOUT_DR", decision_id, payments)
            continue

        // 2b. Calculate total authorized payments
        total_authorized = sum(p.payment_amount for p in payments
                               where p.authorization_result == true)

        // 2c. Verify against maximum_value
        if total_authorized > dr.maximum_value:
            report.addAlert("MAXIMUM_VALUE_EXCEEDED", decision_id,
                           total_authorized, dr.maximum_value)

        // 2d. Check SDR expiration
        if dr.is_sdr:
            for each payment in payments where authorization_result == true:
                if payment.execution_timestamp > dr.maximum_term:
                    report.addAlert("SDR_EXPIRED_PAYMENT", decision_id,
                                   payment.payment_id)

        // 2e. Check beneficiary match
        for each payment in payments where authorization_result == true:
            if payment.payment_beneficiary != dr.beneficiary:
                report.addAlert("BENEFICIARY_MISMATCH", decision_id,
                               payment.payment_id)

        // 2f. Check currency match
        for each payment in payments where authorization_result == true:
            if payment.payment_currency != dr.currency:
                report.addAlert("CURRENCY_MISMATCH", decision_id,
                               payment.payment_id)

        // 2g. Check revocation timing
        rr = findRevocationRecord(decision_id)
        if rr != null:
            for each payment in payments where authorization_result == true:
                if payment.execution_timestamp > rr.record_timestamp:
                    report.addAlert("PAYMENT_AFTER_REVOCATION", decision_id,
                                   payment.payment_id)

    // Phase 3: Verify external anchors
    for each anchor in external_anchors:

        // 3a. Recalculate state hash
        records_at_height = getRecordsUpTo(anchor.chain_height)
        computed_hash = computeStateHash(records_at_height)

        if computed_hash != anchor.state_hash:
            report.addAlert("ANCHOR_MISMATCH", anchor.anchor_id,
                           computed_hash, anchor.state_hash)

        // 3b. Verify anchor frequency
        if (anchor.timestamp - previous_anchor.timestamp) > 24_hours:
            report.addAlert("ANCHOR_FREQUENCY_EXCEEDED", anchor.anchor_id)

    // Phase 4: Generate report
    report.drs_processed = count(decision_record_ledger)
    report.payments_processed = count(payment_ledger)
    report.anchors_verified = count(external_anchors)
    report.alerts_count = count(report.alerts)
    report.audit_timestamp = now()
    report.auditor_signature = sign(report, auditor_private_key)

    return report
```

#### 11.9.3 Alert Types

| Alert | Severity | Description |
|-------|----------|-------------|
| `INVALID_SIGNATURE` | CRITICAL | DR signature verification failed |
| `INVALID_TIMESTAMP` | CRITICAL | Timestamp attestation invalid |
| `CHAIN_BREAK` | CRITICAL | Hash chain integrity violated |
| `PAYMENT_WITHOUT_DR` | CRITICAL | Payment executed without valid DR |
| `MAXIMUM_VALUE_EXCEEDED` | CRITICAL | Payments exceed DR maximum |
| `SDR_EXPIRED_PAYMENT` | HIGH | Payment after SDR expiration |
| `PAYMENT_AFTER_REVOCATION` | HIGH | Payment after DR revocation |
| `BENEFICIARY_MISMATCH` | HIGH | Payment to wrong beneficiary |
| `CURRENCY_MISMATCH` | HIGH | Payment in wrong currency |
| `REGISTRATION_DELAY_EXCEEDED` | MEDIUM | DR registered >72h after decision |
| `ANCHOR_MISMATCH` | CRITICAL | External anchor doesn't match |
| `ANCHOR_FREQUENCY_EXCEEDED` | MEDIUM | >24h between anchors |

#### 11.9.4 Audit Report Publication

The Technical Auditor MUST publish audit reports that include:
1. Audit period (start and end timestamps)
2. Total DRs, payments, and anchors processed
3. All alerts with full details
4. Cryptographic signature of the auditor
5. Hash of previous audit report (for audit chain integrity)

Reports MUST be published within 72 hours of audit completion.

#### 11.9.5 Alert Escalation

For CRITICAL alerts:
1. Immediate notification to oversight authority
2. Payment execution SHOULD be suspended pending investigation
3. Public notice within 24 hours

For HIGH alerts:
1. Notification to oversight authority within 24 hours
2. Investigation required within 7 days

For MEDIUM alerts:
1. Included in monthly audit report
2. Remediation recommended

---

## 12. Nature of the Protocol

Fides:

- Is not a product
- Is not a platform
- Is not a transparency system
- Does not depend on specific people
- Does not require trust

It is a public technical rule, implementable by any entity, auditable by any person, and resistant to internal or external capture.

---

## 13. Success Criterion

The protocol is considered successful if:

> A malicious agent, even with internal access, cannot execute an irregular payment without leaving a verifiable trace.

Nothing beyond this defines success.

---

## 14. Evolution

This document defines the minimum version necessary to guarantee integrity in execution.

Any future evolution cannot violate the principles established here.

Version changes follow semantic versioning:
- **Patch (0.2.x):** Clarifications, typo fixes
- **Minor (0.x.0):** New optional features, extended types
- **Major (x.0.0):** Breaking changes to core rules

---

## 15. Failure Modes & Recovery

This section specifies what implementations MUST do when integrity violations or component failures are detected.

**Core Principle:** When in doubt, fail closed. Deny payment authorization until the issue is resolved.

### 15.1 Timestamp Authority Compromise

If a timestamp authority (TSA) is compromised or suspected of issuing fraudulent timestamps:

1. **Immediate action:**
   - Publish security notice identifying the compromised TSA
   - Flag all DRs attested by that TSA as `ATTESTATION_SUSPECT`

2. **Affected records handling:**
   - DRs are NOT automatically invalidated (historical record preserved)
   - Payments against flagged DRs require manual oversight authorization
   - New DRs MUST NOT use the compromised TSA

3. **Recovery procedure:**
   - Optional: Re-attest affected DRs using a different, trusted TSA
   - Re-attestation does not change original `record_timestamp`
   - Re-attestation proof is stored alongside original proof

4. **Documentation:**
   - Incident report MUST be published within 72 hours
   - List of affected `decision_id` values MUST be made public

### 15.2 Blockchain Reorganization

If a blockchain reorganization (reorg) invalidates previously-confirmed transactions:

1. **Detection:**
   - Monitor block confirmations continuously
   - If confirmations drop below minimum (6 for Bitcoin, 12 for Ethereum), trigger alert

2. **Affected records handling:**
   - Mark affected DRs as `ATTESTATION_PENDING`
   - New payments against pending DRs are DENIED
   - Already-executed payments are NOT reversed (they are flagged for audit)

3. **Recovery procedure:**
   - Wait for chain to stabilize (minimum 24 hours after reorg)
   - Re-verify confirmations for all affected transactions
   - If transaction is still valid, mark DR as `ATTESTATION_CONFIRMED`
   - If transaction is lost, DR attestation becomes INVALID

4. **Contingency:**
   - If attestation becomes invalid, the DR itself is not invalidated
   - But payment authorization returns `false` until re-attestation occurs

### 15.3 Chain Integrity Break

If a hash chain break is detected (hash mismatch between consecutive records):

1. **Severity:** CRITICAL — This indicates tampering or data corruption

2. **Immediate action:**
   - Halt ALL new DR registrations
   - Halt ALL payment authorizations
   - Notify oversight authority immediately
   - Publish public alert

3. **Investigation scope:**
   - All records AFTER the break are suspect
   - Records BEFORE the break are presumed valid

4. **Recovery options:**

   **Option A: Reconstruction (if backup exists)**
   - Restore from last known-good backup
   - Re-register any missing records with new timestamps
   - Document gap in chain with `chain_recovery_record`

   **Option B: Fork (if reconstruction impossible)**
   - Create new chain starting from break point
   - Mark old chain as `INVALIDATED_AT_HEIGHT_N`
   - All DRs after break require re-creation and re-signing

5. **Payment handling:**
   - Payments already executed remain executed
   - New payments are denied until chain is restored
   - All affected payments marked `CHAIN_INTEGRITY_DISPUTE`

### 15.4 Payment Executor Unavailable

If Payment Executor cannot be reached:

1. **Queuing:**
   - Payment authorization requests are queued (with request timestamp)
   - Queue is append-only and preserved

2. **Timeout:**
   - Maximum queue wait time: 24 hours
   - After 24 hours, queued requests are marked `TIMEOUT_REJECTED`

3. **Fallback:**
   - **NO FALLBACK TO ALTERNATE EXECUTOR**
   - Fallback would violate separation of functions
   - The designated Payment Executor MUST be restored

4. **Notification:**
   - Oversight authority notified after 1 hour of unavailability
   - Public notice after 4 hours of unavailability

5. **Recovery:**
   - When executor is restored, process queue in FIFO order
   - Each queued request re-evaluated with current DR state
   - Requests where DR was revoked during queue time are rejected

### 15.5 External Anchor Failure

If external anchor cannot be published:

1. **Grace period:**
   - System continues accepting DRs for up to 24 hours
   - All DRs created during this period are marked `ANCHOR_PENDING`

2. **Hard stop:**
   - After 24 hours without successful anchor, NEW DRs are rejected
   - Payment authorization continues for existing, properly-anchored DRs

3. **Recovery:**
   - Once anchor capability is restored, publish catch-up anchor immediately
   - Catch-up anchor includes all pending records
   - Mark previously pending DRs as `ANCHOR_CONFIRMED`

4. **Maximum outage:**
   - If anchor capability is not restored within 72 hours, enter emergency mode
   - Emergency mode requires oversight authority intervention
   - No new DRs or payments until anchor is restored

### 15.6 Signature Key Compromise

If a decider's signing key is compromised:

1. **Immediate action:**
   - Decider MUST publish key revocation notice
   - Key revocation includes timestamp of compromise (if known)

2. **Affected records:**
   - DRs signed BEFORE suspected compromise time remain valid
   - DRs signed AFTER suspected compromise time are marked `SIGNATURE_SUSPECT`

3. **New signatures:**
   - Decider MUST generate new key pair
   - New public key MUST be registered with authority
   - Old key MUST NOT be used for new signatures

4. **Payment handling:**
   - Payments against suspect DRs require manual oversight authorization
   - Automatic authorization resumes only after oversight review

### 15.7 Recovery Record

All failure events and recovery actions MUST be documented in a Recovery Record:

```json
{
  "recovery_id": "uuid-v4",
  "failure_type": "tsa_compromise | blockchain_reorg | chain_break |
                   executor_unavailable | anchor_failure | key_compromise",
  "detected_at": "ISO 8601",
  "affected_records": ["decision_id", ...],
  "recovery_action": "string (description)",
  "resolved_at": "ISO 8601 (nullable)",
  "oversight_authorization": "string (if required)",
  "previous_record_hash": "string (SHA-256)",
  "signatures": ["...same format as DR..."]
}
```

Recovery Records are chained with DRs and subject to the same immutability rules.

---

## Appendix A: Decision Record Schema (JSON)

```json
{
  "decision_id": "uuid-v4",
  "authority_id": "string",
  "deciders_id": ["string"],
  "act_type": "commitment | contract | amendment | purchase_order | grant | payroll | reimbursement | transfer",
  "currency": "string (ISO 4217)",
  "maximum_value": "decimal",
  "beneficiary": "string",
  "legal_basis": "string",
  "decision_date": "ISO 8601 with Z suffix",
  "previous_record_hash": "string (SHA-256, hex encoded)",
  "record_timestamp": "ISO 8601 with Z suffix",
  "timestamp_attestation": {
    "method": "rfc3161 | blockchain | ntp_consensus",
    "proof": {
      "// For rfc3161:": "",
      "tsa_url": "string",
      "tsa_certificate": "string (base64 DER)",
      "timestamp_token": "string (base64 DER)",
      "hash_algorithm": "SHA-256",
      "message_imprint": "string (hex)",
      "// For blockchain:": "",
      "chain": "bitcoin | ethereum",
      "network": "mainnet | testnet",
      "block_number": "integer",
      "block_hash": "string (hex)",
      "transaction_id": "string (hex)",
      "merkle_proof": ["string (hex)"],
      "data_hash": "string (hex)",
      "confirmations_at_record": "integer",
      "// For ntp_consensus:": "",
      "samples": [{"server": "string", "stratum": "integer", "timestamp": "ISO 8601", "offset_ms": "integer", "delay_ms": "integer"}],
      "consensus_timestamp": "ISO 8601",
      "max_divergence_ms": "integer"
    },
    "sources": ["string"]
  },
  "signatures": [
    {
      "signer_id": "string",
      "public_key": "string (base64)",
      "algorithm": "Ed25519 | ECDSA-P256 | ECDSA-P384 | RSA-PSS",
      "signature": "string (base64)",
      "signed_at": "ISO 8601"
    }
  ],
  "references": "uuid-v4 (optional, required for amendments)"
}
```

**Note:** The `proof` object contains method-specific fields. Only include fields relevant to the chosen `method`. See Section 6.9.1 for detailed format specifications.

---

## Appendix B: Special Decision Record Schema (JSON)

```json
{
  "...all DR fields...": "...",
  "is_sdr": true,
  "exception_type": "public_calamity | court_order | health_emergency | essential_service | national_security | late_registration",
  "formal_justification": "string (min 100 chars)",
  "maximum_term": "ISO 8601",
  "reinforced_deciders": ["string (>= 2x normal)"],
  "oversight_authority": "string"
}
```

---

## Appendix C: Revocation Record Schema (JSON)

```json
{
  "revocation_id": "uuid-v4",
  "target_decision_id": "uuid-v4",
  "revocation_type": "voluntary | oversight | judicial | superseded",
  "revocation_reason": "string (min 50 chars)",
  "revoker_authority": "string",
  "revoker_id": ["string"],
  "revocation_date": "ISO 8601",
  "previous_record_hash": "string (SHA-256)",
  "record_timestamp": "ISO 8601",
  "signatures": ["...same format as DR..."],
  "org_chart_reference": {
    "// Required only for hierarchical superior revocations": "",
    "org_chart_url": "string (URL to published organization chart)",
    "org_chart_hash": "string (SHA-256 of the chart at time of original DR)",
    "org_chart_timestamp_proof": {
      "method": "rfc3161 | blockchain",
      "proof": "object (same format as timestamp_attestation.proof in Appendix A)"
    },
    "revoker_position": "string (position ID in the chart)",
    "superior_path": ["string (chain of positions from revoker to original decider)"]
  }
}
```

**Note:** The `org_chart_reference` field is REQUIRED when `revoker_authority` claims hierarchical superiority. The `org_chart_timestamp_proof` MUST demonstrate the chart was published before the original DR. See Section 10.3.1 for validation requirements.

---

## Appendix D: Verification Function (Pseudocode)

```
function isPaymentAuthorized(decision_id, payment):

    // Acquire exclusive lock for this decision_id
    lock = acquireLock(decision_id)

    try:
        dr = findDecisionRecord(decision_id)

        if dr == null:
            return false

        if dr.revoked == true:
            return false

        if dr.valid == false:
            return false

        if not verifySignatures(dr):
            return false

        if not verifyTimestampAttestation(dr):
            return false

        if payment.date < dr.decision_date:
            return false

        if payment.beneficiary != dr.beneficiary:
            return false

        if payment.currency != dr.currency:
            return false

        // SDR expiration check
        if dr.is_sdr == true:
            if payment.date > dr.maximum_term:
                return false

        total_paid = sumPreviousPayments(decision_id)

        if (total_paid + payment.value) > dr.maximum_value:
            return false

        // Record this payment before releasing lock
        recordPayment(decision_id, payment)

        return true

    finally:
        releaseLock(lock)
```

---

## Appendix E: Canonical Serialization (Pseudocode)

```
function canonicalSerialize(record):
    // Remove computed fields
    obj = copyWithout(record, ["hash", "computed_fields"])

    // Sort keys recursively
    obj = sortKeysRecursive(obj)

    // Serialize to JSON
    json = toJSON(obj, {
        indent: none,
        separators: [",", ":"],
        ensure_ascii: false,
        sort_keys: true
    })

    // Encode as UTF-8
    bytes = encodeUTF8(json)

    return bytes

function computeHash(record):
    bytes = canonicalSerialize(record)
    return SHA256(bytes).toHex()
```

---

## Appendix F: Invariants (Compliance Checklist)

An implementation is compatible with Fides v0.2 if:

- [ ] No payment is executed without a valid DR
- [ ] No retroactive alteration is possible
- [ ] No authorization is subjective
- [ ] No exception exists outside of record
- [ ] No decision exists without an identified decider
- [ ] No power is concentrated in a single role
- [ ] Source code is public (AGPLv3)
- [ ] A verifiable external anchor exists
- [ ] **Signatures are cryptographically verifiable**
- [ ] **Record timestamps are externally attested**
- [ ] **Canonical serialization is implemented correctly**
- [ ] **Payment processing is serialized per decision_id**
- [ ] **SDR expiration is enforced in isPaymentAuthorized**
- [ ] **Revocation authority is verified**
- [ ] **Maximum registration delay (72h) is enforced**
- [ ] **External anchor interval <= 24 hours**
- [ ] **Compliance test suite passes**

---

## Appendix G: Changes from v0.1

See [CHANGELOG.md](../spec/CHANGELOG.md) for detailed changes.

Summary of breaking changes:
1. Cryptographic signatures now mandatory (was implementation-defined)
2. Maximum registration delay of 72 hours (was unlimited)
3. Timestamp attestation required (was implicit)
4. Canonical serialization specified (was undefined)
5. Payment serialization required (was implicit)
6. Exception types enumerated (was examples only)
7. Revocation authority specified (was undefined)
8. Compliance test suite required (was none)

---

## Appendix H: Payment Ledger Entry Schema (JSON)

```json
{
  "payment_id": "uuid-v4",
  "decision_id": "uuid-v4",
  "payment_amount": "decimal",
  "payment_currency": "string (ISO 4217)",
  "payment_beneficiary": "string",
  "request_timestamp": "ISO 8601 with Z suffix",
  "authorization_result": "boolean",
  "rejection_reason": "string (nullable, required if authorization_result == false)",
  "execution_timestamp": "ISO 8601 with Z suffix (nullable, required if authorization_result == true)",
  "payment_executor_id": "string",
  "previous_payment_hash": "string (SHA-256, hex encoded)",
  "timestamp_attestation": {
    "method": "rfc3161 | blockchain | ntp_consensus",
    "proof": "object (same format as DR timestamp_attestation)"
  },
  "signatures": [
    {
      "signer_id": "string (payment_executor_id)",
      "public_key": "string (base64)",
      "algorithm": "Ed25519 | ECDSA-P256 | ECDSA-P384 | RSA-PSS",
      "signature": "string (base64)",
      "signed_at": "ISO 8601"
    }
  ]
}
```

### Rejection Reason Codes

When `authorization_result` is `false`, the `rejection_reason` MUST be one of:

| Code | Description |
|------|-------------|
| `DR_NOT_FOUND` | No Decision Record exists for the given decision_id |
| `DR_REVOKED` | The Decision Record has been revoked |
| `DR_INVALID` | The Decision Record failed validation |
| `SIGNATURE_INVALID` | DR signatures could not be verified |
| `TIMESTAMP_INVALID` | DR timestamp attestation failed |
| `PAYMENT_BEFORE_DECISION` | Payment date precedes decision_date |
| `BENEFICIARY_MISMATCH` | Payment beneficiary does not match DR |
| `CURRENCY_MISMATCH` | Payment currency does not match DR |
| `SDR_EXPIRED` | SDR maximum_term has passed |
| `MAXIMUM_VALUE_EXCEEDED` | Payment would exceed maximum_value |
| `EXECUTOR_UNAVAILABLE` | Payment Executor was unavailable (timeout) |
| `CHAIN_INTEGRITY_FAILURE` | Chain integrity check failed |
| `ATTESTATION_SUSPECT` | DR attestation is under investigation |

---

## Appendix I: Audit Report Schema (JSON)

```json
{
  "audit_id": "uuid-v4",
  "audit_period_start": "ISO 8601",
  "audit_period_end": "ISO 8601",
  "auditor_id": "string",
  "drs_processed": "integer",
  "payments_processed": "integer",
  "anchors_verified": "integer",
  "alerts": [
    {
      "alert_type": "string (see Section 11.9.3)",
      "severity": "CRITICAL | HIGH | MEDIUM",
      "decision_id": "uuid-v4 (if applicable)",
      "payment_id": "uuid-v4 (if applicable)",
      "details": "string",
      "detected_at": "ISO 8601"
    }
  ],
  "alert_summary": {
    "critical_count": "integer",
    "high_count": "integer",
    "medium_count": "integer"
  },
  "audit_timestamp": "ISO 8601",
  "previous_audit_hash": "string (SHA-256, hex encoded)",
  "signatures": [
    {
      "signer_id": "string (auditor_id)",
      "public_key": "string (base64)",
      "algorithm": "Ed25519 | ECDSA-P256 | ECDSA-P384 | RSA-PSS",
      "signature": "string (base64)",
      "signed_at": "ISO 8601"
    }
  ]
}
```

---

## Appendix J: Recovery Record Schema (JSON)

```json
{
  "recovery_id": "uuid-v4",
  "failure_type": "tsa_compromise | blockchain_reorg | chain_break | executor_unavailable | anchor_failure | key_compromise",
  "detected_at": "ISO 8601",
  "detection_method": "string (how failure was detected)",
  "affected_records": ["uuid-v4 (decision_ids)"],
  "affected_payments": ["uuid-v4 (payment_ids)"],
  "severity": "CRITICAL | HIGH | MEDIUM",
  "recovery_action": "string (description of action taken)",
  "recovery_status": "in_progress | resolved | escalated",
  "resolved_at": "ISO 8601 (nullable)",
  "oversight_authorization": {
    "authority_id": "string",
    "authorization_timestamp": "ISO 8601",
    "authorization_reference": "string"
  },
  "previous_record_hash": "string (SHA-256, hex encoded)",
  "record_timestamp": "ISO 8601",
  "signatures": [
    {
      "signer_id": "string",
      "public_key": "string (base64)",
      "algorithm": "Ed25519 | ECDSA-P256 | ECDSA-P384 | RSA-PSS",
      "signature": "string (base64)",
      "signed_at": "ISO 8601"
    }
  ]
}
```

---

**Fides Protocol v0.2**
*Built for trust.*

# Fides Protocol

**Version:** 0.5.7
**Status:** Active
**Nature:** Open technical standard
**License:** AGPLv3
**Supersedes:** v0.4

---

## 1. Definition

The Fides Protocol is a minimal set of technical rules that conditions the execution of any public expenditure on the prior existence of a recorded, immutable, and verifiable decision.

**No valid record, no payment.**

---

## 2. Objective

**Render misuse of public funds undeniable**, acting at the real point of power: the decision that authorizes the expenditure.

Fides does not judge. It exposes.


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

These elements are not necessary to make misuse undeniable.

---

## 4. Limitations and Non-Goals

This section explicitly documents what the Fides Protocol does **not** do. These are not oversights — they are deliberate design boundaries.

### 4.1 What Fides Does NOT Do

**Does not validate decision content (or legal shell companies)**

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
| `currency` | String | ISO 4217 currency code (e.g., USD, EUR) |
| `maximum_value` | String (Fixed-Point) | Positive numeric value in the specified currency (e.g., "100.00") |
| `beneficiary` | String | Tax ID or entity identifier |
| `legal_basis` | String | Explicit legal reference |
| `decision_date` | DateTime | Date and time of the decision (ISO 8601) |
| `previous_record_hash` | String | Cryptographic hash of the previous record (SHA-256) |
| `record_timestamp` | DateTime | Date and time of the record (ISO 8601), externally attested |
| `signatures` | Array | Cryptographic signatures (see 6.3.2) |
| `program_id` | String | **MANDATORY** - Identifier linking related decisions (Format: `{authority_id}:{local_identifier}`) |
| `references` | UUID v4 | Reference to original decision (required for amendments, null otherwise) |
| `delay_justification` | Object | Required if registration delay > 72h (see 6.4.5) |

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
| `genesis` | System initialization (Block 0) |

**Amendment constraint:** An `amendment` type DR MUST include a `references` field pointing to the original `decision_id`. Amendment chains MUST NOT cumulatively exceed 25% of the ORIGINAL contract value. Amendments exceeding this cumulative limit require a new `contract` type DR.

Jurisdictions MAY define additional types, but MUST publish their extended taxonomy publicly. Extended types MUST NOT overlap semantically with the mandatory types above.

#### 6.3.2 Cryptographic Signatures (Mandatory)

The `signatures` field MUST contain cryptographic signatures that are:

- **Verifiable** — Can be validated using public key cryptography
- **Non-repudiable** — Signer cannot deny having signed
- **Timestamped** — Signature includes timestamp

**Approved Algorithms:**
Implementations MUST define a list of approved cryptographic algorithms compliant with local information security regulations.
- Recommended: Ed25519, ECDSA (P-256/P-384), RSA-PSS (>2048-bit).
- Permitted: National standard algorithms (e.g., GOST R 34.10, SM2) if approved by the implementation's Oversight Authority.

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

**Signature Timing Rule:** All `signed_at` timestamps MUST be <= `record_timestamp`. Signatures dated after the record timestamp are INVALID.

**Minimum Deciders:** The `deciders_id` array MUST contain at least 1 identifier. Implementations MAY define a higher minimum per `act_type` or `authority_id` in their configuration.

#### 6.3.4 Program Identifier (Mandatory)

The `program_id` field enables correlation of related Decision Records for audit purposes.

**Purpose:** Detect semantic fragmentation — the practice of splitting a logically unified expenditure into multiple smaller DRs to avoid political or administrative thresholds.

**Usage:**
- The field is **MANDATORY**.
- When present, it MUST be a stable identifier assigned by the authority.
- Multiple DRs MAY share the same `program_id`.
- The Technical Auditor SHOULD aggregate spending by `program_id` in reconciliation reports.

**Audit correlation:** Auditors MUST analyze patterns of:
- Multiple DRs to the same `beneficiary` within short time windows
- Sequential DRs with similar `legal_basis`
- DRs just below known approval thresholds

**Anti-gaming:** The `program_id` is binding. Its absence invalidates the DR.

#### 6.3.3 External PKI Requirements (Critical)

Public key infrastructure for signature verification MUST be external to the implementing jurisdiction to prevent certificate authority manipulation.

**Mandatory Requirements:**

1. **External CA Requirement:** The Certificate Authority (CA) issuing signer certificates MUST NOT be controlled by:
   - The same government implementing the Fides instance
   - Any agency within that government's hierarchy
   - Any entity with financial interest in the decisions being recorded

2. **Acceptable CA Sources:**
   - International commercial CAs (e.g., DigiCert, GlobalSign, Let's Encrypt)
   - Foreign government CAs from jurisdictions with no bilateral agreement with implementer
   - Consortium CAs with multi-stakeholder governance (minimum 3 independent parties)
   - Blockchain-based decentralized identity systems (e.g., ION, Sovrin)

3. **CA Validation:**
   - Implementations MUST publish the list of accepted CAs
   - CAs MUST undergo annual independent security audit
   - CA compromise MUST trigger immediate key revocation and re-issuance
   - Implementations MUST support multiple CAs for redundancy

4. **Key Binding:**
   - Each signer's public key MUST be bound to their identity via external CA certificate
   - Certificate MUST include Extended Key Usage (EKU) for document signing
   - Certificate validity period MUST be checked at signature time

**Rationale:** If the implementing government controls the CA, it can issue fraudulent certificates for non-existent signers, undermining the entire non-repudiation guarantee.

#### 6.3.3.1 CA Identity Validation (Critical)

To prevent fraudulent signing with non-existent identities, every CA MUST implement identity verification:

1. **Official Registry Verification:**
   - Signer identity (National ID, Tax ID, or equivalent) MUST be verified against the official government registry of the corresponding jurisdiction
   - Verification MUST be performed by the CA or an independent third party
   - Verification MUST NOT rely solely on data provided by the implementing agency
   - The registry used for verification MUST be the primary source of truth for the signer's jurisdiction (e.g., National Tax Authority, Civil Registry)
   - **Cross-Verification (Mandatory):** Identity MUST be verified against at least TWO independent sources:
       1. Official government registry (Civil/Tax)
       2. AND at least ONE of: Banking KYC records, Telecom registration, Foreign government records, or Notarized biometric capture.


2. **Certificate Transparency (Mandatory):**
   - All certificates MUST be logged to public CT log
   - Log MUST be searchable by the signer's unique identifier (ID/Tax ID)
   - Search result shows: Name, ID, CA, issue date, expiry
   - Anyone can verify: "Does this certificate really exist?"

3. **Red Flag Detection:**
   - Multiple certificates for same ID = fraud signal
   - Certificate for non-existent ID = immediate rejection
   - Implementer must check CT logs monthly
   - Report suspicious patterns to oversight authority

4. **Annual CA Audit:**
   - Independent auditor verifies:
     * Identity verification procedures work
     * No fraudulent certificates issued
     * CT logs are complete and accurate
   - Public audit report mandatory
   - Audit must be published within 30 days

5. **Rationale:**
   Even with external CA, malicious actors can obtain fake certificates for non-existent people. Identity verification against official registries closes this gap.

### 6.4 Validity Rules

A Decision Record is valid if, and only if:

1. All required fields are present
2. `maximum_value` > 0 (except for `act_type: genesis`, where it MUST be 0)
3. `decision_date` <= `record_timestamp`
4. **`record_timestamp` - `decision_date` <= `MAX_REGISTRATION_DELAY`** (maximum registration delay)
5. `previous_record_hash` corresponds exactly to the last valid record
6. The record has not been revoked
7. The record respects the format and types defined in this protocol
8. `authority_id` refers to an authority that exists at `decision_date`
9. All signatures are cryptographically valid
10. `record_timestamp` is attested by an independent time authority

**Failure in any item = invalid record.**

**Registration Delay Rule (6.4.4):**
- **Standard Window:** 72 hours. Decisions SHOULD be registered within this window.
- **Extended Window (73h - 7 days):** Requires `delay_justification` (Reason + Explanation).
- **Strict Limit:** 7 days. Decisions beyond this limit require a Special Decision Record (SDR) with `exception_type: late_registration`.
- **Absolute Hard Cap:** The parameter `MAX_REGISTRATION_DELAY` MUST NOT exceed 7 days.


#### 6.4.5 Registration Delay Justification

To detect abuse of the registration window, all registrations delayed beyond a reasonable implementation-defined threshold MUST include:

1. **Delay Reason:** A coded reason (e.g., `system_outage`).
2. **Explanation:** A text explanation if the reason is generic.
3. **Approval:** (Optional) Approval by oversight mechanism if required by local policy.

**Rationale:** Systematic delays may conceal backdating. Requiring justification creates an audit trail.

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
{"authority_id":"GOV-001","currency":"USD","decision_date":"2025-01-15T10:00:00Z","decision_id":"550e8400-e29b-41d4-a716-446655440000","deciders_id":["GOV-ID-123"],"legal_basis":"Public Procurement Act","maximum_value":"10000.00","previous_record_hash":"abc123...","record_timestamp":"2025-01-15T14:30:00Z","signatures":[...]}
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

#### 6.6.3 Genesis Record
There MUST be exactly ONE Genesis Record per implementation. Any system presenting multiple Genesis Records is INVALID.

The first record in the chain (Genesis Record) MUST be a valid Decision Record with special characteristics. It acts as the "Root of Trust" and "DNA" of the implementation.

**Requirements:**
- `act_type`: MUST be `genesis`.
- `previous_record_hash`: MUST be 64 zeros (`0000000000000000000000000000000000000000000000000000000000000000`).
- `decision_id`: MUST be `00000000-0000-0000-0000-000000000000` (Nil UUID).
- `authority_id`: MUST be `GENESIS`.
- `maximum_value`: MUST be 0.
- `legal_basis`: MUST cite the specific Law, Decree, or Ordinance that authorizes the implementation of the system.
- `deciders_id`: MUST list the top-level authorities responsible for the deployment (e.g., Minister, CIO, Auditor General).
- `signatures`: MUST contain valid cryptographic signatures of the listed deciders.
- `system_parameters`: MUST be present (see Schema).
- `governance_succession`: MUST specify succession rules for system parameters modification (e.g., "2/3 Cabinet Vote").
- `beneficiary`: MUST be `GENESIS` (placeholder, no payment possible).
- `currency`: MUST be `XXX` (ISO 4217 code for 'no currency').
- `program_id`: MUST be `GENESIS:INIT` (system initialization, no actual program).
- `decision_date`: MUST be the date the authorizing Law/Decree was published or enacted.
- `references`: MUST be `null` (Genesis does not reference any prior decision).
- `delay_justification`: MAY be present if Genesis is registered more than 72h after the authorizing Law was published.
- `is_sdr`: MUST be `false` (Genesis cannot be an exception).

**Purpose:**
This ensures that the system does not start from a "vacuum". It starts from a specific legal act, signed by specific liable individuals, declaring the initial rules of the game.


### 6.7 Publicity

Every Decision Record must be:

- Made public immediately after recording (within 60 seconds; implementations SHOULD target < 5 seconds)
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
3. **Multiple independent NTP sources** - DEPRECATED in v0.3 (See Section 6.9.3)

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
    "network": "mainnet | testnet (WARNING: testnet for dev/test ONLY; production MUST use mainnet)",
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
6. **Verify genTime** — The timestamp MUST be within ±24 hours of `record_timestamp` (allows for network delays and clock drift).
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

##### 6.9.2.3 Monotonicity Rule (All Methods)

The attested timestamp of record N MUST be >= the attested timestamp of record N-1 (monotonic constraint). This applies to both RFC 3161 (`genTime`) and Blockchain (`block_timestamp`) attestation methods, preventing accumulated drift across the chain.

### 6.9.3 Deprecation of NTP (Security Hardening)

The use of legacy NTP consensus (`ntp_consensus`) allowed in previous drafts is **FORMALLY DEPRECATED** and **REMOVED** from Fides v0.3.

**Rationale:**
- **MITM Vulnerability:** Unauthenticated NTP (UDP 123) is trivially spoofable by network attackers.
- **NTS Availability:** Network Time Security (RFC 8915) is not yet universally available in government infrastructure.
- **Strictness:** Fides prioritizes "Fail Closed". If a secure external audit trail (Blockchain/RFC 3161) cannot be generated, the record should not exist. Weak timestamping is worse than no timestamping because it provides false confidence.

**Requirement:**
Implementations MUST reject any record attempting to use `ntp_consensus` as a timestamp attestation method.


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
5. The payment beneficiary = DR beneficiary **OR** a valid `assigned_beneficiary` (see Section 10.6)
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

- The payment is **UNAUTHORIZED** within the Fides protocol
- A compliant Payment Executor **MUST** block the payment
- If executed anyway (e.g., via Break-Glass or malfeasance), it constitutes **undeniable proof of irregularity**

The system does not physically stop a determined attacker from moving funds, but it guarantees that such movement is permanently recorded as illegal.


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
| `payment_amount` | String (Fixed-Point) | Amount requested |
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
2. **Chained** — Each entry includes hash of previous entry (the first entry MUST have `previous_payment_hash` set to 64 zeros, same convention as Genesis Record)
3. **Complete** — Both authorized AND rejected payments MUST be recorded
4. **Public** — Ledger MUST be publicly accessible with NO exceptions
5. **Machine-readable** — JSON format required
6. **Timestamped** — All entries MUST have externally attested timestamps (same methods as DR)
7. **Externally Anchored** — The Payment Ledger MUST be periodically anchored to an external medium, following the same requirements as the DR Ledger (Section 8.3). This prevents the "silent deletion" of rejected payments by a corrupt Executor.

**Beneficiary Transparency (Critical):** Beneficiary identifiers MUST NOT be redacted, encrypted, or hidden under any circumstances. The entire purpose of Fides is to create accountability for public spending. Allowing hidden beneficiaries defeats this purpose entirely.

If a jurisdiction claims that certain beneficiaries require secrecy (e.g., witness protection, undercover operations), those expenditures fall outside the scope of Fides. Such payments:
- MUST NOT use the Fides protocol
- MUST be authorized through separate, jurisdiction-specific classified expenditure procedures
- MUST be subject to oversight by appropriate classified oversight bodies (e.g., intelligence committees)

**Rationale:** A protocol that allows hiding who receives public money is not an anti-corruption protocol—it is a corruption-enabling protocol. There are no exceptions.

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

#### 7.7.5 Payment Execution Atomicity (Critical)

The payment to the financial system MUST be executed **within the lock**, not after. This prevents the "Two Generals Problem" where Fides records success but the bank fails.

**Correct execution order:**

```
function executePayment(decision_id, payment):

    lock = acquireLock(decision_id)

    try:
        // Step 1: Verify authorization
        if not isPaymentAuthorized(decision_id, payment):
            recordPayment(decision_id, payment, REJECTED, reason)
            return false

        // Step 2: Execute at financial institution (INSIDE LOCK)
        bank_result = callBankAPI(payment)

        if bank_result.failed:
            recordPayment(decision_id, payment, BANK_FAILED, bank_result.error)
            return false

        // Step 3: Record success ONLY after bank confirms
        recordPayment(decision_id, payment, EXECUTED, bank_result.confirmation)
        return true

    finally:
        releaseLock(lock)
```

**Critical requirements:**

1. **Bank execution inside lock** — The call to the financial institution MUST occur while holding the exclusive lock
2. **Record after confirmation** — `recordPayment()` with `EXECUTED` status MUST only be called after receiving bank confirmation
3. **Bank failure handling** — If the bank fails, record as `BANK_FAILED`, not `EXECUTED`
4. **Idempotency key** — Pass `payment_id` to bank as idempotency key to prevent duplicate execution on retry

**Prohibited patterns:**

```
// WRONG: Record before bank execution
recordPayment(EXECUTED)  // ← records success
releaseLock()
callBankAPI()           // ← bank fails, but ledger says success
```

**New rejection reason code:**

| Code | Description |
|------|-------------|
| `BANK_EXECUTION_FAILED` | Authorization passed but bank rejected/failed the transfer |

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

To detect silent sabotage:

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

#### 8.4.1 Anchor Media Redundancy (Critical)

The Immutability Guardian MUST NOT be a single point of failure. External anchors MUST be published to multiple independent media simultaneously.

**Mandatory Requirements:**

1. **Minimum Anchor Media:** Anchors MUST be published to at least TWO (2) independent media types from different categories:
   - Category A: Public blockchains (Bitcoin, Ethereum)
   - Category B: Distributed timestamping services (RFC 3161 TSAs from different operators)
   - Category C: Academic/NGO archives (Internet Archive, university repositories)
   - Category D: International press (published hash in major newspapers)

2. **Media Independence:**
   - The two chosen media MUST NOT share common infrastructure
   - The two chosen media MUST NOT be controlled by entities in the same jurisdiction
   - Failure of one medium MUST NOT affect availability of the other

3. **Anchor Consistency:**
   - The same state hash and chain height MUST be published to all media
   - Discrepancy between media anchors indicates Guardian compromise
   - Technical Auditor MUST verify cross-media consistency

4. **Guardian Plurality (Mandatory):**
Implementations MUST have at least TWO Immutability Guardians from different jurisdictions or distinct institutional nature (e.g., one Governmental, one Academic/NGO).
- The system MUST NOT accept a chain state unless signed by a quorum (e.g., 2-of-3) of Guardians.
- **Fork Resolution:** In case of anchor discrepancy between Guardians, the Record Operator's local state is considered SUSPECT. The system MUST enter fail-closed mode until Guardians reach consensus or Oversight Authority intervenes.
- **Consensus Definition:** Consensus requires agreement of ALL active Guardians on a single state hash. If consensus is not reached within 72 hours, the Oversight Authority MUST designate one Guardian's state as authoritative and initiate replacement procedures for dissenting Guardians.

**Failure Mode:** If anchor cannot be published to minimum 2 media within 24 hours, system enters fail-closed state per Section 15.5.

**Rationale:** A single Immutability Guardian with a single anchor medium is a single point of failure. Government pressure on one entity/medium can compromise the entire immutability guarantee.

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

### 9.3 Permitted Exception Types (Implementation Defined)

Implementations MUST define a formal taxonomy of approved exception types. The following reference taxonomy SHOULD be used as a baseline, but jurisdictions MAY define additional types required by local law.

**Reference Taxonomy:**

| Type | Description | Maximum Term |
|------|-------------|--------------|
| `public_calamity` | Natural disaster, declared emergency | 90 days |
| `court_order` | Judicial mandate requiring immediate action | As specified by court |
| `health_emergency` | Imminent threat to public health | 30 days |
| `essential_service` | Prevent interruption of critical service | 15 days |
| `national_security` | Classified defense/security matter | 180 days |
| `late_registration` | Decision registered after 72h window | N/A (one-time) |

**Constraints on Extension:**
- **No Free Text:** Exception types MUST be pre-defined in the implementation's configuration. Use of arbitrary strings is PROHIBITED.
- **Generic Ban:** Generic types like "other", "special", or "misc" are PROHIBITED.
- **Structural Compliance:** Any custom exception type MUST still define a `maximum_term` and `reinforced_deciders` requirement.

### 9.4 Additional SDR Fields

In addition to the standard DR fields, every exception must contain:

| Field | Type | Description |
|-------|------|-------------|
| `exception_type` | Enum | One of the permitted types (9.3) |
| `formal_justification` | String | Public text justifying (minimum 100 characters) |
| `maximum_term` | DateTime | Expiration date (required, must respect type limits) |
| `reinforced_deciders` | Array | >= 2x minimum deciders of common DR (MUST be unique identifiers) |
| `oversight_authority` | String | ID of authority that must review this SDR (MUST NOT be in deciders_id or reinforced_deciders) |

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

### 9.8 Exception Abuse Prevention (Exponential Decay)

To discourage the normalization of exceptions (e.g., renewing a "State of Calamity" indefinitely), the protocol imposes an algorithmic cooldown cost.

**Exponential Cooldown Rule:**
If a new SDR is **recorded** with the same `exception_type` and `authority_id` as a previous SDR contained within the last 365 days, the implementation MUST enforce a waiting period or increased quorum requirement.

*Mandatory Formula: `Cooldown = BASE_COOLDOWN * (2 ^ consecutive_occurrences)`.*
*Constraint: `BASE_COOLDOWN` MUST be at least 7 days.*
*Cap: `Cooldown` MUST NOT exceed `MAX_COOLDOWN` (Implementation Parameter, recommended: 365 days).*


**Rationale:** This ensures that "permanent emergencies" become exponentially politically expensive to maintain, forcing a return to normal order.

---

## 10. Lifecycle Modifications

### 10.1 Principle

Decisions can be revoked or assigned. History is never erased.

### 10.2 Revocation Record (RR)

A decision is revoked exclusively by a Revocation Record (RR), which:

- References the original `decision_id` (NOTE: The Genesis Record CANNOT be revoked. Any RR targeting decision_id `00000000-0000-0000-0000-000000000000` is INVALID.)
- Is chained and immutable
- **Locking:** A Revocation Record MUST acquire the same exclusive lock as payments before being recorded. Revocations and payments are mutually exclusive operations (see Section 7.6).
- Is public
- Has valid deciders with revocation authority

### 10.3 Modification Authority

A Revocation Record (RR) or Assignment Record (AR) is valid only if signed by:

1. **Original deciders** — Any decider listed in the original DR's `deciders_id`
2. **High-Level Authority** — An entity with documented override authority (e.g., hierarchical superior, oversight board, DAO governance contract). See 10.3.1.
3. **Oversight authority** — The `oversight_authority` specified in an SDR
4. **Judicial authority** — A court with jurisdiction (requires `court_order` reference)

#### 10.3.1 Authority Verification (Critical)

Implementations MUST provide a verifiable mechanism to prove the override authority relationship.

**Requirement:** The modification record MUST reference a public, immutable source of truth (e.g., an externally timestamped organization chart, bylaws, or smart contract) that established the authority *before* the original decision was made.

**Rationale:** Without verifiable governance history, a compromised system could fabricate "superiors" retroactively.

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

### 10.6 Assignment Record (AR)

An Assignment Record redirects the payment right of a DR to a new beneficiary (e.g., factoring, court garnishment).

**Rules:**
1.  **Reference:** MUST reference the original `decision_id`.
2.  **Consent:** MUST be signed by the *current* beneficiary (voluntary assignment) OR by a Judicial Authority (garnishment).
3.  **Chain:** If multiple assignments exist, they form a sub-chain (A -> B -> C).

#### 10.6.1 Assignment Security (Critical)
To prevent unauthorized redirection of funds (theft of receivables), Assignments MUST be cryptographically authorized according to their type:

1.  **Voluntary Assignment (Factoring):**
    - **Signer:** The `previous_owner` (Original Creditor).
    - **Rationale:** Only the owner of the credit can authorize its sale.

2.  **Involuntary Assignment (Judicial/Administrative):**
    - **Signer:** The `authority_id` (The Payer/Government).
    - **Rationale:** The Authority acknowledges the Court Order and commits to redirecting the payment.
    - **Verification:** The AR MUST include `judicial_reference` containing:
        - `court_order_hash`: SHA-256 of the official PDF/XML of the order.
        - `publication_url`: URL to the Official Gazette/Diary where the order is published.
        - `court_signature`: (Optional but Recommended) Digital signature of the Court itself.


**Validation Rule:**
```
isAssignmentValid(ar) ->
  if ar.type == VOLUNTARY: verifySignature(ar, ar.previous_owner)
  else: verifySignature(ar, ar.authority_id)
```
4.  **No Value Change:** Assignment CANNOT change `maximum_value` or `currency`.

### 10.7 Effect of Assignment

After a valid AR:
- The `isPaymentAuthorized` function MUST validate `payment.beneficiary` against the **latest assigned beneficiary**.
- Payment to the original beneficiary becomes **UNAUTHORIZED**.

**Note on Assignment Chain Disputes:** Disputes involving intermediate beneficiaries in an assignment chain (e.g., invalidation of a previous owner) are legal matters outside the scope of this protocol. The protocol reports the current chain state; courts determine legal ownership.

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

#### 11.9.2 Reconciliation Requirements

The Technical Auditor MUST perform full reconciliation at least monthly. The reconciliation process MUST verify:

1. **Cryptographic Integrity:** All signatures and timestamp attestations are valid.
2. **Chain Continuity:** The hash chain is unbroken from genesis to current tip.
3. **Financial Consistency:** The sum of authorized payments for any `decision_id` does not exceed its `maximum_value`.
4. **Temporal Validity:** No payments were authorized after DR revocation or SDR expiration.
5. **Anchor Consistency:** Published external anchors match the internal ledger state.

Any violation of these conditions constitutes a critical failure of the protocol.

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
| `FRAGMENTATION_PATTERN` | MEDIUM | Multiple DRs to same beneficiary below thresholds |


#### 11.9.4 Audit Report Publication

The Technical Auditor MUST publish audit reports that include:
1. Audit period (start and end timestamps)
2. Total DRs, payments, and anchors processed
3. All alerts with full details
4. Cryptographic signature of the auditor
5. Hash of previous audit report (for audit chain integrity)

**11.9.4.1 Auditor Validation (Who Audits the Auditor?):**
The Technical Auditor is NOT immune to scrutiny.
- The Auditor's signature MUST be verifiable by the Oversight Authority.
- The Auditor's history MUST be immutable (Audit Reports are chained).
- If the Auditor fails to publish (See 11.9.1 Liveness), the Oversight Authority MUST trigger a "Technical Intervention" to replace the Auditor.
- Any entity (public) can re-run the `isPaymentAuthorized` logic on the public data. If a public check differs from the Auditor's report, the Auditor is proven corrupt.

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

## 15. Operational Integrity

The system must operate on a **Fail Closed** basis. If any security invariant is violated, the system MUST return negative authorization for all payments until the issue is resolved.

### 15.1 Critical Failure Modes

Implementations MUST handle the following failure modes by entering a safe (suspended) state:

1.  **Timestamp Authority Failure:** If the external time source (TSA) is compromised or offline.
2.  **Chain Integrity Break:** If the hash of a record does not match the `previous_record_hash` of the next.
3.  **Anchor Mismatch:** If the published external anchor contradicts the internal ledger.
4.  **Executor Unavailability:** If the Payment Executor cannot confirm payment status.

### 15.2 Recovery Principle

Recovery from critical failures MUST be auditable. Any manual intervention to restore the system (e.g., re-signing, forking after a hack) MUST be recorded in a specific **Recovery Record** that is cryptographically chained to the ledger.

**History is never erased, only appended.**

### 15.3 Emergency Break-Glass (Survival Mode)

In existential crises (e.g., war, total infrastructure collapse) where strict integrity checks would cause loss of life, implementations MAY provide a "Break-Glass" mechanism.

**Rules:**
1.  **Forced Execution:** Allows payment execution despite validation failures (e.g., unreachable TSA).
2.  **Tainted Record:** MUST invoke `recordPayment` with a special `INTEGRITY_VIOLATION` status.
3.  **Degraded Timestamping:** If TSA is unreachable, the system MUST use local time and append a specific flag `"timestamp_source": "local_degraded"` to the payment record. This timestamp is NOT authoritative but preserves sequential ordering.
4.  **Permanent Stickiness:** The DR and all subsequent events are permanently marked as "Tainted".
5.  **Time Limit:** Break-Glass mode MUST NOT exceed `MAX_BREAK_GLASS_DURATION` (Implementation Parameter, max 24 hours).
6.  **Exit Strategy:** The system MUST automatically halt if the crisis is not resolved within the limit.



**Rationale:** The goal is to prevent silent corruption. A declared, public, and permanent violation is not silent. It is a desperate act recorded for history.

---

## Appendix A: Decision Record Schema (JSON)

```json
{
  "decision_id": "uuid-v4",
  "authority_id": "string",
  "deciders_id": ["string"],
  "act_type": "genesis | commitment | contract | amendment | purchase_order | grant | payroll | reimbursement | transfer",
  "currency": "string (ISO 4217)",
  "maximum_value": "string (Fixed-Point, e.g. \"100.00\")",
  "beneficiary": "string",
  "legal_basis": "string",
  "decision_date": "ISO 8601 with Z suffix",
  "previous_record_hash": "string (SHA-256, hex encoded)",
  "record_timestamp": "ISO 8601 with Z suffix",
  "timestamp_attestation": {
    "method": "rfc3161 | blockchain",
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
      "confirmations_at_record": "integer"
    }
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
  "references": "uuid-v4 (optional, required for amendments)",
  "program_id": "string (MANDATORY, format: {authority_id}:{local_identifier})",
  "delay_justification": {
    "delay_reason": "string (coded reason: system_outage | connectivity_failure | disaster_recovery | other)",
    "explanation": "string (required if reason is 'other')",
    "approval": "string (optional, oversight approval reference)"
  },
  "system_parameters": {
    "accepted_cas": ["string (CA Root IDs)"],
    "max_registration_delay": "integer (seconds)",
    "external_anchors": ["string (URLs)"],
    "role_assignments": {
      "record_operator": "string (Entity ID)",
      "immutability_guardian": "string (Entity ID)",
      "technical_auditor": "string (Entity ID)",
      "payment_executor": "string (Entity ID)"
    },
    "initial_org_chart": {
      "url": "string (URL to published organization chart)",
      "hash": "string (SHA-256)",
      "content": "string (JSON representation of initial hierarchy)"
    },
    "governance_succession": {
      "modification_authority": "string (e.g., '2/3 Cabinet Vote')",
      "amendment_process": "string (reference to legal procedure)"
    }
  },
  "is_sdr": "boolean (false for regular DR, true for SDR - see Appendix B)"
}
```

**Note on system_parameters:** This field is REQUIRED for `act_type: genesis` and OPTIONAL (informational) for others. For Genesis, `initial_org_chart` serves as the bootstrap for hierarchy verification (Section 10.3.1).


**Note:** The `proof` object contains method-specific fields. Only include fields relevant to the chosen `method`. See Section 6.9.1 for detailed format specifications.

---

## Appendix B: Special Decision Record Schema (JSON)

An SDR MUST include ALL mandatory DR fields (including `is_sdr: false` replaced by `is_sdr: true`) plus the additional fields below. The `reinforced_deciders` array MAY overlap with `deciders_id`; the union of both MUST contain at least 2x the implementation's minimum required deciders.

```json
{
  "...all DR fields...": "...",
  "is_sdr": true,
  "exception_type": "string (from implementation-defined taxonomy)",
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
  "timestamp_attestation": { "... (REQUIRED, same format as DR) ..." },
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

**Implementation Notes:**
- `dr.valid` is a **computed property**, not a stored field. It represents the result of applying all validity rules from Section 6.4 to the record.
- `dr.revoked` is a **computed property** indicating whether a valid Revocation Record exists for this `decision_id`.
- `findDecisionRecord()` MUST return `null` if the record exists but the chain is broken (hash mismatch). An invalid chain = no valid records.

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

        // Check for Assignment
        current_beneficiary = resolveCurrentBeneficiary(dr)
        if payment.beneficiary != current_beneficiary:
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

function sumPreviousPayments(decision_id):
    // Sum ONLY payments that were actually executed (money left the account)
    // authorization_result = true means Fides authorized it
    // execution_timestamp != null means the bank confirmed execution
    //
    // IMPORTANT: Do NOT include BANK_FAILED payments (authorized but bank rejected)
    // Those did not move money, so they should not consume the DR limit

    return sum(
        payment.amount
        for payment in PaymentLedger
        where payment.decision_id = decision_id
          AND payment.authorization_result = true
          AND payment.execution_timestamp != null
    )

function resolveCurrentBeneficiary(dr):
    // Recursive function to find the final beneficiary after assignments
    current_beneficiary = dr.beneficiary
    depth = 0
    MAX_DEPTH = 50

    while true:
        // Find if there is an assignment where previous_owner == current_beneficiary
        // AND original_decision_id == dr.decision_id
        assignment = findAssignmentRecord(current_beneficiary, dr.decision_id)

        if assignment == null:
            return current_beneficiary // No further assignment, this is the final one

        if not isAssignmentValid(assignment):
            throw Error("Invalid Assignment detected in chain")

        current_beneficiary = assignment.new_beneficiary
        depth++

        if depth > MAX_DEPTH:
            throw Error("Assignment chain depth exceeded limit")

    return current_beneficiary
```

---

## Appendix F: Invariants (Compliance Checklist)


An implementation is compatible with Fides v0.5 if:

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
- [ ] **Maximum registration delay (as parameterized) is enforced**
- [ ] **External anchor interval <= 24 hours**
- [ ] **Compliance test suite passes**

---

## Appendix G: Changes from v0.1

See [CHANGELOG.md](../spec/CHANGELOG.md) for detailed changes.

Summary of breaking changes:
1. Cryptographic signatures now mandatory (was implementation-defined)
2. Maximum registration delay of 7 days (was unlimited)
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
  "payment_amount": "string (Fixed-Point, e.g. \"100.00\")",
  "payment_currency": "string (ISO 4217)",
  "payment_beneficiary": "string",
  "request_timestamp": "ISO 8601 with Z suffix",
  "authorization_result": "boolean",
  "rejection_reason": "string (nullable, required if authorization_result == false)",
  "execution_timestamp": "ISO 8601 with Z suffix (nullable, required if authorization_result == true)",
  "payment_executor_id": "string",
  "previous_payment_hash": "string (SHA-256, hex encoded)",
  "timestamp_attestation": {
    "method": "rfc3161 | blockchain",
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
  "failure_type": "tsa_compromise | blockchain_reorg | chain_break | executor_unavailable | anchor_failure | key_compromise | integrity_violation",
  "detected_at": "ISO 8601",
  "detection_method": "string (how failure was detected)",
  "affected_records": ["uuid-v4 (decision_ids)"],
  "affected_payments": ["uuid-v4 (payment_ids)"],
  "severity": "CRITICAL",
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
  "timestamp_attestation": { "... (REQUIRED, same format as DR) ..." },
  "signatures": [ {"...": "..."} ]
}
```

---

## Appendix K: Assignment Record Schema (JSON)

```json
{
  "assignment_id": "uuid-v4",
  "original_decision_id": "uuid-v4",
  "authority_id": "string (from original DR)",
  "previous_owner": "string (beneficiary ID)",

  "new_beneficiary": "string (beneficiary ID)",
  "assignment_type": "voluntary | judicial | administrative",
  "legal_basis": "string (court order ID or contract ID)",
  "judicial_reference": {
    "court_order_hash": "string (SHA-256, required for judicial/administrative, MUST be null for voluntary)",
    "publication_url": "string (URL, required for judicial/administrative, MUST be null for voluntary)",
    "court_signature": "string (base64, optional)"
  },
  "assignment_date": "ISO 8601",
  "signatures": [
    {
      "signer_id": "string (MUST match previous_owner OR authority_id)",
      "public_key": "string (base64)",
      "algorithm": "Ed25519 | ECDSA-P256 | ECDSA-P384 | RSA-PSS",
      "signature": "string (base64)",
      "signed_at": "ISO 8601"
    }
  ],
  "previous_record_hash": "string (SHA-256)",
  "timestamp_attestation": { "... (REQUIRED) ..." }
}
```

**Signature Rule:**
- If `assignment_type` is `voluntary`: `signer_id` MUST be `previous_owner`.
- If `assignment_type` is `judicial` or `administrative`: `signer_id` MUST be `authority_id` (Authority acknowledges and executes the order).

---

**Fides Protocol v0.5.7**

*Universal Core.*

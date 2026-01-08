# Fides Protocol — Data Model Reference

This document specifies the data structures used in Fides Protocol v0.1.

---

## 1. Overview

Fides uses three primary record types:

1. **Decision Record (DR)** — Authorizes potential expenditure
2. **Special Decision Record (SDR)** — Authorizes exception
3. **Revocation Record (RR)** — Revokes a previous decision

All records share common properties:
- Immutable after creation
- Cryptographically chained
- Publicly accessible
- Machine-readable (JSON)

---

## 2. Decision Record (DR)

### 2.1 Schema

```json
{
  "record_type": "DR",
  "decision_id": "uuid-v4",
  "authority_id": "string",
  "deciders_id": ["string"],
  "act_type": "commitment | contract | amendment",
  "maximum_value": "decimal",
  "beneficiary": "string",
  "legal_basis": "string",
  "decision_date": "ISO 8601",
  "previous_record_hash": "string (SHA-256)",
  "record_timestamp": "ISO 8601",
  "signatures": ["string"]
}
```

### 2.2 Field Specifications

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `record_type` | String | Yes | Must be "DR" |
| `decision_id` | UUID v4 | Yes | Globally unique |
| `authority_id` | String | Yes | Non-empty, identifies issuing authority |
| `deciders_id` | Array[String] | Yes | Non-empty array |
| `act_type` | Enum | Yes | One of: commitment, contract, amendment |
| `maximum_value` | Decimal | Yes | > 0 |
| `beneficiary` | String | Yes | Non-empty, tax ID or entity identifier |
| `legal_basis` | String | Yes | Non-empty, legal reference |
| `decision_date` | DateTime | Yes | ISO 8601, <= record_timestamp |
| `previous_record_hash` | String | Yes | SHA-256 hex, 64 characters |
| `record_timestamp` | DateTime | Yes | ISO 8601, server time at recording |
| `signatures` | Array[String] | Yes | Non-empty array of signer identifiers |

### 2.3 Example

```json
{
  "record_type": "DR",
  "decision_id": "550e8400-e29b-41d4-a716-446655440000",
  "authority_id": "GOV-HEALTH-001",
  "deciders_id": ["OFFICIAL-12345", "OFFICIAL-67890"],
  "act_type": "contract",
  "maximum_value": 150000.00,
  "beneficiary": "SUPPLIER-ABC-123",
  "legal_basis": "Public Procurement Act, Article 24",
  "decision_date": "2024-03-15T10:30:00Z",
  "previous_record_hash": "a1b2c3d4e5f6...",
  "record_timestamp": "2024-03-15T10:31:45Z",
  "signatures": ["SIG-OFFICIAL-12345", "SIG-OFFICIAL-67890"]
}
```

---

## 3. Special Decision Record (SDR)

### 3.1 Schema

Extends DR with additional fields for exceptions.

```json
{
  "record_type": "SDR",
  "decision_id": "uuid-v4",
  "authority_id": "string",
  "deciders_id": ["string"],
  "act_type": "commitment | contract | amendment",
  "maximum_value": "decimal",
  "beneficiary": "string",
  "legal_basis": "string",
  "decision_date": "ISO 8601",
  "previous_record_hash": "string (SHA-256)",
  "record_timestamp": "ISO 8601",
  "signatures": ["string"],
  "exception_type": "string",
  "formal_justification": "string",
  "maximum_term": "ISO 8601",
  "reinforced_deciders": ["string"]
}
```

### 3.2 Additional Fields

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `exception_type` | String | Yes | Must be from predefined list |
| `formal_justification` | String | Yes | Non-empty, public justification |
| `maximum_term` | DateTime | Yes | ISO 8601, future date |
| `reinforced_deciders` | Array[String] | Yes | Length >= deciders_id length |

### 3.3 Valid Exception Types

Implementations must define a closed list. Examples:

- `PUBLIC_CALAMITY`
- `COURT_ORDER`
- `HEALTH_EMERGENCY`
- `ESSENTIAL_SERVICE_CONTINUITY`

Generic types are prohibited:
- ~~`EXCEPTIONAL`~~
- ~~`URGENT`~~
- ~~`SPECIAL`~~

### 3.4 Example

```json
{
  "record_type": "SDR",
  "decision_id": "660e8400-e29b-41d4-a716-446655440001",
  "authority_id": "GOV-HEALTH-001",
  "deciders_id": ["OFFICIAL-12345", "OFFICIAL-67890"],
  "act_type": "contract",
  "maximum_value": 500000.00,
  "beneficiary": "EMERGENCY-SUPPLIER-999",
  "legal_basis": "Emergency Powers Act, Section 5",
  "decision_date": "2024-03-20T08:00:00Z",
  "previous_record_hash": "b2c3d4e5f6a1...",
  "record_timestamp": "2024-03-20T08:05:00Z",
  "signatures": ["SIG-OFFICIAL-12345", "SIG-OFFICIAL-67890", "SIG-MINISTER-001"],
  "exception_type": "HEALTH_EMERGENCY",
  "formal_justification": "Urgent acquisition of medical supplies due to disease outbreak in northern region. Regular procurement timeline incompatible with emergency response requirements.",
  "maximum_term": "2024-06-20T23:59:59Z",
  "reinforced_deciders": ["OFFICIAL-12345", "OFFICIAL-67890", "MINISTER-001"]
}
```

---

## 4. Revocation Record (RR)

### 4.1 Schema

```json
{
  "record_type": "RR",
  "revocation_id": "uuid-v4",
  "target_decision_id": "uuid-v4",
  "authority_id": "string",
  "deciders_id": ["string"],
  "revocation_reason": "string",
  "revocation_date": "ISO 8601",
  "previous_record_hash": "string (SHA-256)",
  "record_timestamp": "ISO 8601",
  "signatures": ["string"]
}
```

### 4.2 Field Specifications

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `record_type` | String | Yes | Must be "RR" |
| `revocation_id` | UUID v4 | Yes | Globally unique |
| `target_decision_id` | UUID v4 | Yes | Must reference existing DR or SDR |
| `authority_id` | String | Yes | Non-empty |
| `deciders_id` | Array[String] | Yes | Non-empty array |
| `revocation_reason` | String | Yes | Non-empty |
| `revocation_date` | DateTime | Yes | ISO 8601 |
| `previous_record_hash` | String | Yes | SHA-256 hex |
| `record_timestamp` | DateTime | Yes | ISO 8601 |
| `signatures` | Array[String] | Yes | Non-empty array |

### 4.3 Example

```json
{
  "record_type": "RR",
  "revocation_id": "770e8400-e29b-41d4-a716-446655440002",
  "target_decision_id": "550e8400-e29b-41d4-a716-446655440000",
  "authority_id": "GOV-HEALTH-001",
  "deciders_id": ["OFFICIAL-12345"],
  "revocation_reason": "Contract cancelled due to supplier non-compliance with delivery terms.",
  "revocation_date": "2024-04-01T14:00:00Z",
  "previous_record_hash": "c3d4e5f6a1b2...",
  "record_timestamp": "2024-04-01T14:05:00Z",
  "signatures": ["SIG-OFFICIAL-12345"]
}
```

---

## 5. Payment Record (informational)

Payments are **not** part of the Fides record chain. They exist in the payment system.

However, for verification, a payment must provide:

```json
{
  "payment_id": "string",
  "decision_id": "uuid-v4",
  "value": "decimal",
  "beneficiary": "string",
  "payment_date": "ISO 8601"
}
```

This structure is used as input to `isPaymentAuthorized()`.

---

## 6. Hash Chain

### 6.1 Genesis Record

The first record in a chain has a special `previous_record_hash`:

```
previous_record_hash = SHA256("FIDES-GENESIS-" + authority_id)
```

### 6.2 Subsequent Records

For all records after genesis:

```
previous_record_hash = SHA256(canonical_json(previous_record))
```

### 6.3 Canonical JSON

To ensure deterministic hashing:

1. Keys sorted alphabetically
2. No whitespace
3. UTF-8 encoding
4. Numbers without trailing zeros
5. Dates in ISO 8601 UTC (Z suffix)

### 6.4 Example Chain

```
Record 0 (Genesis):
  previous_record_hash = SHA256("FIDES-GENESIS-GOV-HEALTH-001")
  record_hash = SHA256(Record 0) = "aaa..."

Record 1:
  previous_record_hash = "aaa..."
  record_hash = SHA256(Record 1) = "bbb..."

Record 2:
  previous_record_hash = "bbb..."
  record_hash = SHA256(Record 2) = "ccc..."
```

---

## 7. State Hash

For external anchoring, a state hash represents all records:

```
state_hash = SHA256(record_0_hash + record_1_hash + ... + record_n_hash)
```

Or using Merkle tree for efficiency:

```
state_hash = MerkleRoot(all_record_hashes)
```

---

## 8. Validation Rules

### 8.1 DR Validation

```
valid_dr(dr) =
  dr.record_type == "DR" AND
  is_uuid_v4(dr.decision_id) AND
  non_empty(dr.authority_id) AND
  non_empty(dr.deciders_id) AND
  dr.act_type IN ["commitment", "contract", "amendment"] AND
  dr.maximum_value > 0 AND
  non_empty(dr.beneficiary) AND
  non_empty(dr.legal_basis) AND
  is_iso8601(dr.decision_date) AND
  is_iso8601(dr.record_timestamp) AND
  dr.decision_date <= dr.record_timestamp AND
  is_valid_hash(dr.previous_record_hash) AND
  hash_chain_valid(dr) AND
  non_empty(dr.signatures)
```

### 8.2 SDR Validation

```
valid_sdr(sdr) =
  valid_dr(sdr) AND  // All DR rules apply
  sdr.record_type == "SDR" AND
  sdr.exception_type IN allowed_exception_types AND
  non_empty(sdr.formal_justification) AND
  is_iso8601(sdr.maximum_term) AND
  sdr.maximum_term > sdr.record_timestamp AND
  length(sdr.reinforced_deciders) >= length(sdr.deciders_id)
```

### 8.3 RR Validation

```
valid_rr(rr) =
  rr.record_type == "RR" AND
  is_uuid_v4(rr.revocation_id) AND
  exists(rr.target_decision_id) AND
  not_already_revoked(rr.target_decision_id) AND
  non_empty(rr.authority_id) AND
  non_empty(rr.deciders_id) AND
  non_empty(rr.revocation_reason) AND
  is_iso8601(rr.revocation_date) AND
  is_iso8601(rr.record_timestamp) AND
  is_valid_hash(rr.previous_record_hash) AND
  hash_chain_valid(rr) AND
  non_empty(rr.signatures)
```

---

## 9. Indexes (Implementation Guidance)

For efficient queries, implementations should maintain:

| Index | Purpose |
|-------|---------|
| `decision_id` | O(1) lookup by ID |
| `beneficiary` | Find all decisions for a beneficiary |
| `authority_id` | Find all decisions by authority |
| `record_timestamp` | Chronological queries |
| `act_type` | Filter by type |
| `revoked_decisions` | Quick revocation check |

---

## 10. Data Retention

All records are permanent. The protocol does not define data deletion.

For legal compliance (e.g., privacy laws), implementations may:
- Encrypt personal data fields
- Store encryption keys separately
- Delete keys after retention period

**The record structure and hash chain must remain intact.**

---

*This document is part of Fides Protocol reference materials.*

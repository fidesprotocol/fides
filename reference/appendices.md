# Appendices — Fides Protocol v0.1

> **Status:** Informative
> **Purpose:** Technical clarifications that complement the core specification
> **Note:** These appendices do not modify the frozen specification. They provide implementation guidance for elements referenced but not fully detailed in the core spec.

---

## Appendix D: Revocation Record Schema

A Revocation Record (RR) invalidates a previously issued Decision Record.

### Schema

```json
{
  "revocation_id": "string (UUID v4)",
  "record_type": "RR",
  "revoked_decision_id": "string (UUID v4)",
  "authority_id": "string",
  "revocation_date": "string (ISO 8601)",
  "reason": "string (required, non-empty)",
  "previous_record_hash": "string",
  "record_hash": "string",
  "signatures": ["string"]
}
```

### Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `revocation_id` | UUID v4 | Unique identifier for this revocation |
| `record_type` | string | Always `"RR"` |
| `revoked_decision_id` | UUID v4 | The `decision_id` being revoked |
| `authority_id` | string | Must match the authority of the original DR |
| `revocation_date` | ISO 8601 | When the revocation takes effect |
| `reason` | string | Mandatory justification for revocation |
| `previous_record_hash` | string | Hash of the previous record in the chain |
| `record_hash` | string | Hash of this record (excluding this field) |
| `signatures` | array | Format is implementation-defined |

### Chain Integration

- RR entries are appended to the same hash chain as DRs
- `previous_record_hash` references the last record (DR or RR) in the chain
- Once an RR is recorded, the referenced DR is permanently invalidated

### Validation Rules

1. `revoked_decision_id` must reference an existing, non-revoked DR
2. `authority_id` must match the original DR's authority
3. `reason` must be non-empty
4. `revocation_date` must be >= original DR's `decision_date`

---

## Appendix E: Special Decision Record Schema

A Special Decision Record (SDR) handles exceptions that cannot follow normal DR flow.

### Schema

```json
{
  "decision_id": "string (UUID v4)",
  "record_type": "SDR",
  "authority_id": "string",
  "decision_date": "string (ISO 8601)",
  "justification": "string (required, non-empty)",
  "exception_type": "string",
  "original_decision_id": "string (UUID v4) | null",
  "beneficiary_id": "string",
  "maximum_value": "string (fixed-precision decimal)",
  "currency": "string (ISO 4217)",
  "validity_period": {
    "start": "string (ISO 8601)",
    "end": "string (ISO 8601)"
  },
  "previous_record_hash": "string",
  "record_hash": "string",
  "signatures": ["string"]
}
```

### Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `decision_id` | UUID v4 | Unique identifier |
| `record_type` | string | Always `"SDR"` |
| `justification` | string | Mandatory. SDR without justification = invalid |
| `exception_type` | string | Category of exception (implementation-defined) |
| `original_decision_id` | UUID v4 or null | If present, this SDR modifies an existing DR |

### Exception Types (Illustrative)

| Type | Use Case |
|------|----------|
| `EMERGENCY` | Urgent situation requiring immediate payment |
| `AMENDMENT` | Modification to existing DR |
| `RETROACTIVE` | Regularization of past irregular situation |

### Validation Rules

1. `justification` is mandatory and must be non-empty
2. If `original_decision_id` is present, the referenced DR must exist
3. SDRs follow all other DR validation rules
4. SDRs are subject to enhanced audit scrutiny

### Design Note

SDRs exist to handle reality, not to bypass controls. A system with excessive SDRs indicates process failure, not protocol flexibility.

---

## Appendix F: State Hash Calculation

For external anchoring, implementations must publish a verifiable state hash.

### Calculation

```
state_hash = HASH(canonical_json({
  "authority_id": authority_id,
  "record_count": total_records,
  "last_record_hash": hash_of_last_record,
  "timestamp": ISO_8601_timestamp
}))
```

### Publication Format

```json
{
  "state_hash": "string",
  "authority_id": "string",
  "record_count": integer,
  "last_record_id": "string (UUID v4)",
  "last_record_hash": "string",
  "timestamp": "string (ISO 8601)",
  "anchor_medium": "string (where published)"
}
```

### Verification Process

A third party can verify by:

1. Requesting all records from the authority
2. Recalculating the hash chain
3. Computing the state hash
4. Comparing against the published anchor

### Periodicity

Implementation-defined. Recommended: at least once every 24 hours, or after every N records (where N is implementation-defined).

### Anchor Medium Examples

- Public blockchain transaction
- Official government gazette
- Notarized document
- Multiple independent public repositories

---

## Appendix G: Payment Tracking

To enforce cumulative limits, payments must be tracked.

### Payment Record Schema

```json
{
  "payment_id": "string (UUID v4)",
  "decision_id": "string (UUID v4)",
  "value": "string (fixed-precision decimal)",
  "currency": "string (ISO 4217)",
  "payment_date": "string (ISO 8601)",
  "beneficiary_id": "string"
}
```

### Cumulative Calculation

```
accumulated = sum(value for all payments where decision_id matches)
authorized = (accumulated + new_payment.value) <= DR.maximum_value
```

### Implementation Notes

1. Payment records may be stored separately from the DR/RR chain
2. Payment records must be immutable once created
3. The payment tracking system must be queryable by `decision_id`
4. Currency must match exactly — no automatic conversion

### Verification Function

```python
def is_payment_authorized(ledger, payment) -> bool:
    dr = ledger.get_decision_record(payment.decision_id)
    if dr is None:
        return False
    if ledger.is_revoked(payment.decision_id):
        return False
    if payment.currency != dr.currency:
        return False
    if payment.beneficiary_id != dr.beneficiary_id:
        return False
    if payment.payment_date < dr.decision_date:
        return False

    accumulated = ledger.get_accumulated_payments(payment.decision_id)
    return (accumulated + payment.value) <= dr.maximum_value
```

---

## Notes

- These appendices are **informative**, not normative
- Implementations may extend these schemas with additional fields
- Field formats (signatures, identifiers) remain implementation-defined per the core spec
- Questions about interpretation should reference the core specification first

---

*Clarification without modification.*

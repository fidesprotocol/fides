# Fides Protocol — Algorithms Reference

This document specifies the algorithms used in Fides Protocol v0.1.

---

## 1. Overview

Fides requires these core algorithms:

1. **Hash Chain** — Links records immutably
2. **Payment Authorization** — Verifies if payment is allowed
3. **Chain Integrity Verification** — Detects tampering
4. **State Hash Calculation** — Produces anchor hash

All algorithms are deterministic and publicly verifiable.

---

## 2. Hash Chain Algorithm

### 2.1 Hash Function

**Required:** A collision-resistant cryptographic hash function (e.g., SHA-256 or equivalent).

```
HASH(data) → hex string
```

Example using SHA-256: Output is 64 hexadecimal characters (256 bits).

### 2.2 Canonical JSON

The following pseudocode is illustrative. Any implementation producing identical canonical output is valid.

Before hashing, convert record to canonical form:

```python
def canonical_json(record):
    # 1. Sort keys alphabetically (recursive)
    # 2. No whitespace
    # 3. UTF-8 encoding
    # 4. Numbers: no trailing zeros, no leading zeros (except 0.x)
    # 5. Dates: ISO 8601 with Z suffix

    return json.dumps(
        record,
        sort_keys=True,
        separators=(',', ':'),
        ensure_ascii=False
    )
```

### 2.3 Genesis Hash

For the first record in a chain:

```python
def genesis_hash(authority_id, genesis_timestamp):
    seed = f"FIDES-GENESIS-{authority_id}-{genesis_timestamp}"
    return HASH(seed.encode('utf-8'))
```

Where `genesis_timestamp` is the `record_timestamp` of the first record in the chain (ISO 8601 UTC).

### 2.4 Record Hash

```python
def record_hash(record):
    canonical = canonical_json(record)
    return HASH(canonical.encode('utf-8'))
```

### 2.5 Chain Construction

```python
def create_record(record_data, previous_record):
    record_data['record_timestamp'] = current_utc_time()

    if previous_record is None:
        # Genesis
        record_data['previous_record_hash'] = genesis_hash(
            record_data['authority_id'],
            record_data['record_timestamp']
        )
    else:
        record_data['previous_record_hash'] = record_hash(previous_record)

    return record_data
```

---

## 3. Payment Authorization Algorithm

### 3.1 Main Function

```python
def is_payment_authorized(decision_id, payment):
    """
    Returns True if payment is authorized, False otherwise.
    No exceptions. No partial results. Binary only.
    """

    # Step 1: Find the decision record
    dr = find_decision_record(decision_id)

    if dr is None:
        return False  # No record found

    # Step 2: Check if revoked
    if is_revoked(decision_id):
        return False  # Decision was revoked

    # Step 3: Validate record integrity
    if not is_valid_record(dr):
        return False  # Record is invalid

    # Step 4: Check temporal order
    if payment.date < dr.decision_date:
        return False  # Payment before decision

    # Step 5: Check beneficiary match
    if payment.beneficiary != dr.beneficiary:
        return False  # Wrong beneficiary

    # Step 6: Check value limit
    total_paid = sum_previous_payments(decision_id)
    if (total_paid + payment.value) > dr.maximum_value:
        return False  # Would exceed limit

    # Step 7: Check expiration (for SDR only)
    if dr.record_type == 'SDR':
        if payment.date > dr.maximum_term:
            return False  # Exception expired

    # All checks passed
    return True
```

### 3.2 Helper Functions

```python
def find_decision_record(decision_id):
    """
    Retrieve DR or SDR by decision_id.
    Returns None if not found.
    """
    # Implementation-specific database query
    pass

def is_revoked(decision_id):
    """
    Check if a revocation record exists for this decision.
    """
    rr = find_revocation_record(decision_id)
    return rr is not None

def is_valid_record(dr):
    """
    Validate record structure and hash chain.
    """
    # Check required fields
    if not all_required_fields_present(dr):
        return False

    # Check hash chain
    if not verify_hash_chain(dr):
        return False

    return True

def sum_previous_payments(decision_id):
    """
    Sum all payments already made against this decision.
    """
    payments = find_payments_by_decision(decision_id)
    return sum(p.value for p in payments)
```

### 3.3 Verification Response

The function returns only `True` or `False`.

**Security warning:** Implementations MUST NOT expose rejection reasons to payment operators or deciders, as this could enable adversarial probing of the validation logic.

For internal debugging/audit only, implementations may log:

```python
def is_payment_authorized_with_reason(decision_id, payment):
    """
    Same logic but returns reason for rejection.
    Use for debugging only. Production uses binary version.
    """
    dr = find_decision_record(decision_id)

    if dr is None:
        return (False, "RECORD_NOT_FOUND")

    if is_revoked(decision_id):
        return (False, "DECISION_REVOKED")

    if not is_valid_record(dr):
        return (False, "INVALID_RECORD")

    if payment.date < dr.decision_date:
        return (False, "PAYMENT_BEFORE_DECISION")

    if payment.beneficiary != dr.beneficiary:
        return (False, "BENEFICIARY_MISMATCH")

    total_paid = sum_previous_payments(decision_id)
    if (total_paid + payment.value) > dr.maximum_value:
        return (False, "EXCEEDS_MAXIMUM_VALUE")

    if dr.record_type == 'SDR' and payment.date > dr.maximum_term:
        return (False, "EXCEPTION_EXPIRED")

    return (True, "AUTHORIZED")
```

---

## 4. Chain Integrity Verification

### 4.1 Full Chain Verification

```python
def verify_chain_integrity(records):
    """
    Verify the entire hash chain is intact.
    Returns (valid, first_invalid_index).
    """

    if len(records) == 0:
        return (True, None)

    # Verify genesis record
    first = records[0]
    expected_genesis = genesis_hash(first['authority_id'], first['record_timestamp'])

    if first['previous_record_hash'] != expected_genesis:
        return (False, 0)

    # Verify chain
    for i in range(1, len(records)):
        current = records[i]
        previous = records[i - 1]

        expected_hash = record_hash(previous)

        if current['previous_record_hash'] != expected_hash:
            return (False, i)

    return (True, None)
```

### 4.2 Incremental Verification

For efficiency, verify only new records:

```python
def verify_new_records(last_verified_hash, new_records):
    """
    Verify new records chain correctly from last verified point.
    """

    if len(new_records) == 0:
        return True

    # First new record must chain from last verified
    if new_records[0]['previous_record_hash'] != last_verified_hash:
        return False

    # Verify internal chain
    for i in range(1, len(new_records)):
        current = new_records[i]
        previous = new_records[i - 1]

        if current['previous_record_hash'] != record_hash(previous):
            return False

    return True
```

### 4.3 Anchor Verification

```python
def verify_against_anchor(records, anchor_hash, anchor_index):
    """
    Verify records match a published anchor.
    """

    calculated_state = calculate_state_hash(records[:anchor_index + 1])

    return calculated_state == anchor_hash
```

---

## 5. State Hash Calculation

### 5.1 Simple Method

Concatenate all record hashes:

```python
def calculate_state_hash_simple(records):
    """
    Simple state hash: hash of concatenated record hashes.
    """

    all_hashes = ""
    for record in records:
        all_hashes += record_hash(record)

    return HASH(all_hashes.encode('utf-8'))
```

### 5.2 Merkle Tree Method

More efficient for large datasets:

```python
def calculate_state_hash_merkle(records):
    """
    Merkle tree state hash: root of hash tree.
    """

    if len(records) == 0:
        return HASH(b"EMPTY")

    # Leaf level: hash of each record
    leaves = [record_hash(r) for r in records]

    # Build tree
    while len(leaves) > 1:
        next_level = []

        for i in range(0, len(leaves), 2):
            if i + 1 < len(leaves):
                combined = leaves[i] + leaves[i + 1]
            else:
                combined = leaves[i] + leaves[i]  # Duplicate odd leaf

            next_level.append(HASH(combined.encode('utf-8')))

        leaves = next_level

    return leaves[0]
```

### 5.3 Anchor Publication

```python
def publish_anchor(records, anchor_service):
    """
    Calculate and publish state hash to external anchor.
    """

    state_hash = calculate_state_hash_merkle(records)

    anchor_data = {
        "state_hash": state_hash,
        "record_count": len(records),
        "timestamp": current_utc_time(),
        "last_record_id": records[-1]['decision_id'] if records else None
    }

    anchor_service.publish(anchor_data)

    return anchor_data
```

---

## 6. Record Validation

### 6.1 DR Validation

```python
def validate_dr(dr):
    """
    Validate Decision Record structure and content.
    """

    errors = []

    # Required fields
    required = [
        'decision_id', 'authority_id', 'deciders_id', 'act_type',
        'maximum_value', 'beneficiary', 'legal_basis', 'decision_date',
        'previous_record_hash', 'record_timestamp', 'signatures'
    ]

    for field in required:
        if field not in dr or dr[field] is None:
            errors.append(f"MISSING_{field.upper()}")

    if errors:
        return (False, errors)

    # Type checks
    if not is_uuid_v4(dr['decision_id']):
        errors.append("INVALID_DECISION_ID")

    if not isinstance(dr['deciders_id'], list) or len(dr['deciders_id']) == 0:
        errors.append("INVALID_DECIDERS_ID")

    if not isinstance(dr['act_type'], str) or len(dr['act_type']) == 0:
        errors.append("INVALID_ACT_TYPE")  # act_type is implementation-defined

    # maximum_value should be a fixed-precision decimal (not floating-point)
    if not is_positive_decimal(dr['maximum_value']):
        errors.append("INVALID_MAXIMUM_VALUE")

    if not is_iso8601(dr['decision_date']):
        errors.append("INVALID_DECISION_DATE")

    if not is_iso8601(dr['record_timestamp']):
        errors.append("INVALID_RECORD_TIMESTAMP")

    # Temporal constraint
    if parse_date(dr['decision_date']) > parse_date(dr['record_timestamp']):
        errors.append("DECISION_DATE_AFTER_RECORD_TIMESTAMP")

    # Hash format
    if not is_valid_hash_hex(dr['previous_record_hash']):
        errors.append("INVALID_PREVIOUS_RECORD_HASH")

    if not isinstance(dr['signatures'], list) or len(dr['signatures']) == 0:
        errors.append("INVALID_SIGNATURES")

    return (len(errors) == 0, errors)
```

### 6.2 SDR Validation

```python
def validate_sdr(sdr, allowed_exception_types):
    """
    Validate Special Decision Record.
    """

    # First validate as DR
    valid, errors = validate_dr(sdr)

    if sdr.get('record_type') != 'SDR':
        errors.append("RECORD_TYPE_NOT_SDR")

    # Additional SDR fields
    if 'exception_type' not in sdr:
        errors.append("MISSING_EXCEPTION_TYPE")
    elif sdr['exception_type'] not in allowed_exception_types:
        errors.append("INVALID_EXCEPTION_TYPE")

    if 'formal_justification' not in sdr or not sdr['formal_justification']:
        errors.append("MISSING_FORMAL_JUSTIFICATION")

    if 'maximum_term' not in sdr:
        errors.append("MISSING_MAXIMUM_TERM")
    elif not is_iso8601(sdr['maximum_term']):
        errors.append("INVALID_MAXIMUM_TERM")
    elif parse_date(sdr['maximum_term']) <= parse_date(sdr['record_timestamp']):
        errors.append("MAXIMUM_TERM_NOT_FUTURE")

    if 'reinforced_deciders' not in sdr:
        errors.append("MISSING_REINFORCED_DECIDERS")
    elif len(sdr.get('reinforced_deciders', [])) < len(sdr.get('deciders_id', [])):
        errors.append("INSUFFICIENT_REINFORCED_DECIDERS")

    return (len(errors) == 0, errors)
```

### 6.3 RR Validation

```python
def validate_rr(rr, existing_records):
    """
    Validate Revocation Record.
    """

    errors = []

    required = [
        'revocation_id', 'target_decision_id', 'authority_id',
        'deciders_id', 'revocation_reason', 'revocation_date',
        'previous_record_hash', 'record_timestamp', 'signatures'
    ]

    for field in required:
        if field not in rr or rr[field] is None:
            errors.append(f"MISSING_{field.upper()}")

    if errors:
        return (False, errors)

    # Check target exists
    target = find_record_by_id(rr['target_decision_id'], existing_records)
    if target is None:
        errors.append("TARGET_DECISION_NOT_FOUND")

    # Check not already revoked
    if is_already_revoked(rr['target_decision_id'], existing_records):
        errors.append("ALREADY_REVOKED")

    # Standard validations
    if not is_uuid_v4(rr['revocation_id']):
        errors.append("INVALID_REVOCATION_ID")

    if not is_valid_hash_hex(rr['previous_record_hash']):
        errors.append("INVALID_PREVIOUS_RECORD_HASH")

    return (len(errors) == 0, errors)
```

---

## 7. Utility Functions

### 7.1 UUID Validation

```python
import re

def is_uuid_v4(value):
    """
    Check if value is valid UUID v4.
    """
    pattern = r'^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$'
    return bool(re.match(pattern, str(value).lower()))
```

### 7.2 ISO 8601 Validation

```python
from datetime import datetime

def is_iso8601(value):
    """
    Check if value is valid ISO 8601 datetime.
    """
    try:
        datetime.fromisoformat(value.replace('Z', '+00:00'))
        return True
    except:
        return False

def parse_date(value):
    """
    Parse ISO 8601 string to datetime.
    """
    return datetime.fromisoformat(value.replace('Z', '+00:00'))
```

### 7.3 Hash Validation

```python
def is_valid_hash_hex(value):
    """
    Check if value is valid hexadecimal hash string.
    Length depends on the hash function used (e.g., 64 chars for SHA-256).
    """
    if not isinstance(value, str):
        return False
    if len(value) < 32:  # Minimum reasonable hash length
        return False
    try:
        int(value, 16)
        return True
    except ValueError:
        return False
```

### 7.4 Decimal Validation

```python
from decimal import Decimal, InvalidOperation

def is_positive_decimal(value):
    """
    Check if value is a positive fixed-precision decimal.
    Implementations should use decimal types, not floating-point.
    """
    try:
        d = Decimal(str(value))
        return d > 0
    except (InvalidOperation, TypeError):
        return False
```

---

## 8. Error Codes

Illustrative error codes for implementation reference. Implementations MAY use different codes or formats:

| Code | Meaning |
|------|---------|
| `RECORD_NOT_FOUND` | Decision record does not exist |
| `DECISION_REVOKED` | Decision was revoked |
| `INVALID_RECORD` | Record failed validation |
| `CHAIN_BROKEN` | Hash chain integrity violated |
| `PAYMENT_BEFORE_DECISION` | Payment date precedes decision |
| `BENEFICIARY_MISMATCH` | Payment beneficiary differs from DR |
| `EXCEEDS_MAXIMUM_VALUE` | Would exceed authorized amount |
| `EXCEPTION_EXPIRED` | SDR maximum_term has passed |
| `ANCHOR_MISMATCH` | State hash differs from anchor |
| `GENESIS_INVALID` | First record hash incorrect |

---

## 9. Performance Considerations

### 9.1 Caching

- Cache validated records
- Cache revocation status
- Cache cumulative payment sums
- Invalidate cache on new records

### 9.2 Indexing

- Index by decision_id (primary)
- Index by beneficiary
- Index by authority_id
- Index revocations separately

### 9.3 Batch Operations

```python
def verify_batch_payments(payments):
    """
    Verify multiple payments efficiently.
    """

    # Group by decision_id
    by_decision = group_by(payments, lambda p: p.decision_id)

    results = []

    for decision_id, payment_group in by_decision.items():
        # Load DR once
        dr = find_decision_record(decision_id)

        # Check each payment
        for payment in payment_group:
            result = is_payment_authorized(decision_id, payment)
            results.append((payment.payment_id, result))

    return results
```

---

*This document is part of Fides Protocol reference materials.*

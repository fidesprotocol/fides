# Fides Protocol - Changelog

All notable changes to the Fides Protocol specification.

Format based on [Keep a Changelog](https://keepachangelog.com/).

---

## [0.3] - 2025-01-08

### Summary

Version 0.3 addresses critical security vulnerabilities identified through adversarial security review. These changes close attack vectors that could allow governments or malicious actors to circumvent the protocol's anti-corruption guarantees.

**This is a breaking change from v0.2.** Implementations compliant with v0.2 will need updates to comply with v0.3.

### Added

#### Payment Execution Atomicity (Section 7.7.5)
- **What:** Bank payment execution MUST occur inside the lock, before recording success
- **Why:** Prevents "Two Generals Problem" where Fides records success but bank transaction fails
- **Attack mitigated:** Create DR → fake bank success → extract money → real payment rejected
- **New rejection code:** `BANK_EXECUTION_FAILED`

#### External PKI Requirements (Section 6.3.3)
- **What:** Certificate Authority for signer certificates MUST be external to implementing jurisdiction
- **Why:** Government-controlled CA can issue certificates for non-existent signers
- **Attack mitigated:** Create fake signers → forge DRs → execute fraudulent payments
- **Acceptable CAs:** International commercial CAs, foreign government CAs, consortium CAs, blockchain-based identity systems

#### CA Identity Validation (Section 6.3.3.1)
- **What:** CAs MUST verify signer identity against official government registries; Certificate Transparency logging mandatory
- **Why:** External CA alone doesn't prevent fake certificates for non-existent people
- **Attack mitigated:** Obtain certificate for ghost identity → sign fraudulent DRs
- **Requirements:** CT logs searchable by ID, annual CA audits, red flag detection

#### Anchor Media Redundancy (Section 8.4.1)
- **What:** External anchors MUST be published to at least 2 independent media from different categories
- **Why:** Single Immutability Guardian with single anchor medium is single point of failure
- **Attack mitigated:** Government pressure on one entity/medium to compromise entire immutability
- **Categories:** Public blockchains, RFC 3161 TSAs, academic archives, international press
- **Optional:** Guardian plurality (2-of-3 agreement)

#### Organization Chart Versioning (Section 10.3.2)
- **What:** All org chart updates MUST be externally anchored with version history
- **Why:** Prevents retroactive modification of org charts to enable/block revocations
- **Attack mitigated:** Publish fake org chart → claim false hierarchical authority → revoke legitimate DRs
- **Requirements:** 7-day notice for changes affecting reporting relationships, immutable version history

#### Registration Delay Audit Trail (Section 6.4.5)
- **What:** Registrations delayed > 1 hour require documented justification; > 24 hours require supervisor approval
- **Why:** 72-hour window can be systematically abused for backdating
- **Attack mitigated:** Consistently register at 71 hours → effectively unlimited backdating
- **Monitoring:** Technical Auditor MUST track delay patterns per authority
- **Red flags:** Consistent 70-72h delays, spikes before holidays, supervisor self-approval

#### Exception Abuse Prevention (Section 9.8)
- **What:** Exponential cooldown for repeated SDRs of same type by same authority
- **Why:** Prevents "permanent emergencies" that normalize exceptions
- **Formula:** `Cooldown = 30 days * (2 ^ previous_consecutive_SDRs)`
- **Bypass:** 80% super-majority + oversight authority approval

#### Graceful Degradation for Anchor Failure (Section 15.5)
- **What:** Degraded mode after 72h anchor failure instead of complete halt
- **Why:** DoS attack on anchor infrastructure shouldn't paralyze essential government functions
- **Limits:** 10% of average daily volume cap, essential services only, physical hash sheet backup

### Changed

#### NTP Timestamp Method (Section 6.9.3)
- **Before:** NTP consensus was acceptable attestation method
- **After:** NTP (`ntp_consensus`) is **FORMALLY DEPRECATED AND REMOVED**
- **Why:** Unauthenticated UDP is trivially spoofable; NTS not universally available
- **Requirement:** Implementations MUST reject records using `ntp_consensus`

#### Beneficiary Transparency (Section 7.7.2)
- **Before:** "Ledger MUST be publicly accessible (excluding sensitive beneficiary details if jurisdiction requires)"
- **After:** "Ledger MUST be publicly accessible with NO exceptions"
- **Why:** Hidden beneficiaries defeat the entire anti-corruption purpose
- **Guidance:** Expenditures requiring beneficiary secrecy (witness protection, undercover ops) MUST NOT use Fides

#### Appendix A Schema
- **Before:** `method: "rfc3161 | blockchain | ntp_consensus"`
- **After:** `method: "rfc3161 | blockchain"` (NTP removed)

#### Appendix F Invariants
- Updated reference to "Fides v0.3" compliance

### Removed

- NTP consensus as valid timestamp attestation method
- Jurisdiction exception for beneficiary redaction

### Security Model Changes

| Assumption | v0.2 | v0.3 |
|------------|------|------|
| PKI trust | Government-controlled acceptable | External CA required + identity validation |
| Anchor redundancy | Single medium sufficient | Minimum 2 independent media |
| NTP security | Assumed trustworthy | Deprecated/removed |
| Org chart integrity | Timestamped once | Version-controlled with anchoring |
| Registration delay | 72-hour window unmonitored | Tiered justification + monitoring |
| Beneficiary secrecy | Jurisdiction exception allowed | No exceptions; use alternative process |
| Exception abuse | 30-day wait between same-type SDRs | Exponential cooldown |
| Anchor failure | Complete halt | Graceful degradation with limits |

### Migration Guide

#### For Existing v0.2 Implementations

1. **PKI:** Migrate to external CA with identity validation; cannot use government-issued certificates
2. **Anchoring:** Add second anchor medium from different category
3. **NTP:** Switch to RFC 3161 or blockchain; NTP no longer accepted
4. **Org Charts:** Implement versioning system; anchor all chart updates
5. **Registration:** Add delay_justification fields; implement supervisor approval workflow
6. **Beneficiary:** Remove any redaction logic; route sensitive payments outside Fides
7. **SDR Cooldown:** Implement exponential cooldown tracking

#### For New Implementations

Start with v0.3. Do not implement v0.2.

### Rationale

The vulnerabilities addressed in v0.3 were identified through adversarial security review assuming a motivated attacker with government-level resources. While v0.2 was technically sound, it assumed benevolent implementation. v0.3 assumes adversarial implementation and closes the corresponding attack vectors.

The core promise remains unchanged: **Fides does not stop corruption. It stops corruption from being silent.**

---

## [0.2] - 2025-01-08

### Summary

Version 0.2 addresses security gaps identified in v0.1 through community review. The changes eliminate ambiguities that could be exploited and add mandatory technical requirements that were previously "implementation-defined."

**This is a breaking change from v0.1.** Implementations compliant with v0.1 will need updates to comply with v0.2.

### Post-Release Clarifications (2025-01-08)

The following clarifications were added after initial v0.2 release based on security review feedback:

#### Timestamp Attestation Proof Formats (Section 6.9.1)
- **What:** Detailed specification for `timestamp_attestation.proof` field for each method (RFC 3161, blockchain, NTP consensus)
- **Why:** Original v0.2 had the proof field but didn't specify exact validation format, creating ambiguity
- **Gap addressed:** FIDES_v0.2_ANALYSIS.md "New gap discovered"

#### Hierarchical Superior Definition (Section 10.3.1)
- **What:** Explicit requirements for proving hierarchical authority including mandatory organization chart reference
- **Why:** "Hierarchical superior" was vague and unverifiable without documented evidence
- **Gap addressed:** FIDES_v0.2_ANALYSIS.md recommendation on revocation authority

#### Test Suite Status Clarification (Section 11.6)
- **What:** Added transitional compliance rules until official test suite is published
- **Why:** Test suite requirement was normative but suite didn't exist yet
- **Gap addressed:** FIDES_v0.2_ANALYSIS.md compliance test concern

#### Organization Chart External Timestamping (Section 10.3.1)
- **What:** Org charts used for hierarchical superior claims MUST have external timestamp proof (RFC 3161 or blockchain)
- **Why:** Without external timestamp, org charts could be retroactively falsified to enable invalid revocations
- **Gap addressed:** FIDES_v0.2_Rev2_ANALYSIS.md "Org Chart Falsification" vulnerability

#### Payment Executor Role (Section 11.2)
- **What:** Added 4th mandatory role: Payment Executor, responsible for calling `isPaymentAuthorized()` and executing payments
- **Why:** Original 3-role model didn't specify who executes payments. If same entity as Record Operator, could create-pay-revoke attack
- **Gap addressed:** FIDES_v0.2_Rev2_ANALYSIS.md "Payment Operator role" concern

#### Payment Ledger Specification (Section 7.7)
- **What:** Complete specification for payment ledger including schema, chaining, and rejected payment recording
- **Why:** Protocol specified Payment Executor but not how payments are registered. Without payment ledger, cannot audit blocked payments
- **Gap addressed:** FIDES_WHAT_FALTARIA_PARA_10.md "Payment Ledger Specification"

#### Timestamp Validation Procedures (Section 6.9.2)
- **What:** Rigorous step-by-step validation procedures for RFC 3161, blockchain, and NTP consensus timestamps
- **Why:** Format was specified but validation depth was not. Implementations could do "weak" validation
- **Gap addressed:** FIDES_WHAT_FALTARIA_PARA_10.md "Timestamp Attestation Validation Procedures"

#### Technical Auditor Reconciliation Algorithm (Section 11.9)
- **What:** Complete pseudocode algorithm for reconciling DR ledger, payment ledger, and external anchors
- **Why:** Technical Auditor role was defined but not what exactly they verify and how
- **Gap addressed:** FIDES_WHAT_FALTARIA_PARA_10.md "Technical Auditor Reconciliation Algorithm"

#### Failure Modes & Recovery (Section 15)
- **What:** Comprehensive failure mode documentation including TSA compromise, blockchain reorg, chain break, executor unavailable, anchor failure, and key compromise
- **Why:** "Fail closed" was stated as principle but specific failure scenarios and recovery procedures were undefined
- **Gap addressed:** FIDES_WHAT_FALTARIA_PARA_10.md "Failure Scenarios & Recovery Procedures"

#### New Appendices (H, I, J)
- **What:** Payment Ledger Entry Schema (H), Audit Report Schema (I), Recovery Record Schema (J)
- **Why:** New sections required corresponding schemas for implementation

### Added

#### Cryptographic Signatures (Section 6.3.2)
- **What:** Signatures MUST now be cryptographically verifiable (Ed25519, ECDSA, RSA-PSS)
- **Why:** v0.1 allowed "implementation-defined" signatures, which could mean just a name string. This made non-repudiation impossible.
- **Gap addressed:** FIDES_VULNERABILITIES.md #6

#### Maximum Registration Delay (Section 6.4.4)
- **What:** `record_timestamp` - `decision_date` cannot exceed 72 hours
- **Why:** v0.1 had no limit, allowing decisions to be silently backdated months later
- **Gap addressed:** FIDES_VULNERABILITIES.md #1

#### Timestamp Attestation (Section 6.9)
- **What:** `record_timestamp` MUST be attested by independent time authority (RFC 3161, blockchain, or NTP consensus)
- **Why:** v0.1 let the Record Operator set timestamps unilaterally
- **Gap addressed:** FIDES_VULNERABILITIES.md #11

#### Canonical Serialization (Section 6.6.1)
- **What:** Exact specification for JSON serialization before hashing
- **Why:** Without canonical form, same record could produce different hashes
- **Gap addressed:** FIDES_VULNERABILITIES.md #12

#### Chain Integrity Rules (Section 6.6.2)
- **What:** Single authoritative chain requirement, chain height in anchors
- **Why:** v0.1 didn't prevent chain divergence/forking
- **Gap addressed:** FIDES_VULNERABILITIES.md #5

#### Payment Serialization (Section 7.6)
- **What:** Payments against same decision_id MUST be processed serially with locks
- **Why:** Race conditions could allow exceeding maximum_value
- **Gap addressed:** FIDES_VULNERABILITIES.md #14

#### SDR Expiration Enforcement (Section 9.7)
- **What:** `isPaymentAuthorized` MUST check `maximum_term` for SDRs
- **Why:** v0.1 had the field but didn't require verification
- **Gap addressed:** FIDES_VULNERABILITIES.md #9

#### Exhaustive Exception Types (Section 9.3)
- **What:** Closed enum of permitted exception types with maximum terms
- **Why:** v0.1 gave examples but allowed any "specific" type
- **Gap addressed:** FIDES_VULNERABILITIES.md #8

#### Revocation Authority (Section 10.3)
- **What:** Explicit rules for who can revoke a DR
- **Why:** v0.1 said RR needs "valid deciders" but didn't define authority
- **Gap addressed:** FIDES_VULNERABILITIES.md #7

#### Revocation Record Schema (Section 10.3, Appendix C)
- **What:** Required fields for RR including reason and authority basis
- **Why:** v0.1 RR was underspecified

#### Separation Verification Criteria (Section 11.3)
- **What:** Concrete criteria for verifying role separation
- **Why:** v0.1 principle was too vague to audit
- **Gap addressed:** FIDES_VULNERABILITIES.md #10

#### Compliance Test Suite (Section 11.6)
- **What:** Mandatory test suite for claiming Fides compatibility
- **Why:** AGPLv3 doesn't prevent non-compliant "Fides" implementations
- **Gap addressed:** FIDES_VULNERABILITIES.md #15

#### Mandatory Act Types (Section 6.3.1)
- **What:** Enumerated act_type values with amendment constraints
- **Why:** v0.1 was fully implementation-defined
- **Gap addressed:** FIDES_VULNERABILITIES.md #2

#### Anchor Frequency Limits (Section 8.4)
- **What:** Maximum 24-hour anchor interval
- **Why:** v0.1 said "frequent" without definition
- **Gap addressed:** Related to timing attacks

### Changed

#### Signatures Field
- **Before:** `signatures: Array (format implementation-defined)`
- **After:** `signatures: Array of {signer_id, public_key, algorithm, signature, signed_at}`

#### Act Type Field
- **Before:** `act_type: String (implementation-defined taxonomy)`
- **After:** `act_type: Enum (mandatory taxonomy with optional extensions)`

#### Validity Rules
- **Before:** 7 rules
- **After:** 10 rules (added signature verification, timestamp attestation, registration delay)

#### SDR Fields
- **Before:** 4 additional fields
- **After:** 5 additional fields (added `oversight_authority`)

#### Exception Types
- **Before:** Examples only ("Public calamity, Court order, Health emergency, Essential service continuity")
- **After:** Exhaustive enum with maximum terms per type

### Removed

- Implicit assumption that implementations would "do the right thing"
- Flexibility in signature formats that enabled weak authentication

### Migration Guide

#### For Existing v0.1 Implementations

1. **Signatures:** Replace string-based signatures with cryptographic format
2. **Timestamps:** Integrate with TSA or blockchain timestamping
3. **Serialization:** Implement canonical JSON serialization
4. **Locks:** Add payment serialization per decision_id
5. **Validation:** Update `isPaymentAuthorized` to check SDR expiration
6. **Anchoring:** Ensure anchor interval <= 24 hours
7. **Testing:** Pass compliance test suite

#### For New Implementations

Start with v0.2. Do not implement v0.1.

### Rationale

The gaps in v0.1 were identified through security review (documented in `docs/FIDES_VULNERABILITIES.md`). While v0.1's principles were sound, the specification left too much to implementation discretion, creating exploitable ambiguities.

v0.2 closes these gaps while preserving the core principles:
- No record, no payment
- Immutable, public, attributed
- Binary verification
- Fail closed

The protocol is now more prescriptive but also more secure.

---

## [0.1] - 2025-01-XX

### Initial Release

First public version of the Fides Protocol.

- Core principles established
- Decision Record structure defined
- Payment authorization logic specified
- Governance separation required
- External anchoring mandated

**Status:** Frozen (historical reference)

**Known Issues:** See `docs/v0.1-KNOWN-ISSUES.md`

---

## Version Policy

- **v0.1** remains frozen as historical reference
- **v0.2** superseded by v0.3
- **v0.3** is the current active specification
- Future versions will follow semantic versioning
- Breaking changes require major version increment

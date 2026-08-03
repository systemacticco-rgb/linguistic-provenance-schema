# Linguistic Provenance Schema (LPS)

## Public overview

LPS is a reference implementation for binding visible text to an authenticated
provenance manifest. It records a claimed contribution breakdown for text,
including human, AI-generated, and AI-modified-human segments, then carries a
signed envelope alongside the text for later verification.

**Current status:** the audited v0.1 reference pipeline is accepted for its
implemented scope. This is not a production deployment approval, working-group
ratification, or interoperability/conformance claim for C2PA, SynthID, or any
other AI-watermarking system.

LPS is independent of C2PA. A selector carrier and use of the `c2pa-text`
library do not make LPS a C2PA profile, C2PA-compatible workflow, Content
Credential, JUMBF artifact, or COSE claim-signature artifact.

## What the reference implementation does

```text
visible text → manifest → signed envelope → compressed CBOR → embedded carrier
visible text + carrier → extraction → validation → cryptographic verification → result
carrier unavailable → exact canonical-text hash → registry recovery result
```

The current envelope includes authenticated `ev: 1` version metadata. Its
inner `content_signed_at` describes the generating source's content-record
signing or commitment event; outer `signed_at` describes the LPS complete-
envelope signing event. The exact field and validation contract is maintained
in the [technical specification](../SPEC.md).

LPS binds text by stripping trailing CR, LF, and U+0020, encoding the result as
UTF-8, and deriving both a SHA-256 hash and byte length from those same bytes.
Confidence provenance is preserved as either `tool` or `fallback` so a reader
can distinguish a supplied value from a locally calculated one.

## Verification scope

For a valid carrier, the verifier follows the normal certificate, signature,
and text-binding path. A changed visible text can therefore fail even when a
carrier remains present. Duplicate top-level envelope keys and parseable
invalid envelopes are rejected rather than repaired or recovered.

Registry recovery is deliberately narrower: only an absent, corrupted, or
unparseable carrier may trigger an exact canonical-text-hash lookup. An exact
match yields a limited `registry_required` result, not `verified`. A match
cannot restore span-level evidence, explain carrier loss, prove authorship,
provider origin, or issuer authorization.

The audited implementation distinguishes a no-match result from registry
transport/HTTP failure and malformed/incomplete registry data. See the
[working-group submission](working-group-submission.md) for the reviewer
explanation and the [technical specification](../SPEC.md) for the exact
status/reason contract.

## Confirmed audited scope

- Assertion-backed envelope, registry-routing, malformed-carrier, confidence,
  signing, and verification coverage was completed.
- Read-only live registry exact-match and no-match behavior was confirmed.
- Verification through the configured allowed HTTPS certificate route and
  visible-text tamper detection were confirmed.
- The locked fallback regression object is
  `{ ai_generated: 82, ai_modified_human: 15, human: 1 }`.

These are bounded reference-implementation findings, not service-level or
production-operations evidence.

## Separate Proposal 007 testing-tool observations

Proposal 007 is a separate cooperative-marker testing-tool effort and does
not change the LPS envelope, registry, or verification contract described
above. Its local Firefox/Linux tests observed marker survival for drag-copy,
double-click copy, BiDi-language content, and trailing normalization; malformed,
duplicate-header, orphaned-marker, and internal-codepoint inputs were rejected
with their testing-tool-specific deterministic errors.

These are limited observations, not a portability guarantee. Glyph rendering
varied between LibreOffice, VS Code, browsers, and OneNote; a visual glyph is
not itself evidence of marker corruption. The
[working-group submission](working-group-submission.md) records the complete
test cases, exact error identifiers, tested contexts, and remaining questions.

## Explicit limitations

The following remain deferred:

- production certificate issuer trust, revocation, rotation, and lifecycle
  governance;
- production key management, credential isolation, and credential policy;
- a complete canonical-CBOR profile and decoder bounds;
- broader cryptographic-profile decisions, including P-256 and HMAC/HKDF;
- provider, issuer, and `generating_id` identity semantics; and
- registry SLOs, monitoring, retry policy, incident response, and rollback.

A valid signature or certificate fingerprint does not establish an authorized
issuer. An LPS record is not independently provider-attested without a
separate provider-controlled signature and trust boundary.

## Read next

- [Working-group submission](working-group-submission.md) — reviewer-facing
  architecture, trust model, validated scope, and limitations.
- [Technical specification](../SPEC.md) — normative envelope, timestamp, text
  binding, confidence, validation, and result contract.
- [Architecture](../ARCHITECTURE.md) — components, control flow, and failure
  boundaries.
- [Security model](../SECURITY_MODEL.md) — trust boundaries and production
  exclusions.
- [Implementation status](../IMPLEMENTATION_STATUS.md) — audit evidence and
  deferred operational work.
- [Changelog](../CHANGELOG.md) — dated factual changes.

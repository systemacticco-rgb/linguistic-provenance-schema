# Linguistic Provenance Schema (LPS)

## Working-group submission

**Status:** audited reference-implementation current state; not evidence of
working-group ratification, production approval, or standards conformance.

LPS is a text-provenance reference implementation that binds visible text to
an authenticated manifest. Its contribution is a structured record of claimed
text contribution at segment level, including human, AI-generated, and
AI-modified-human segments, confidence, and confidence provenance.

This submission explains the audited scope and the decisions reviewers would
need to evaluate. It is not the normative field-by-field specification; that
contract is in https://github.com/systemacticco-rgb/lps-reference-implementation/blob/main/SPEC.md

## 1. Scope and standards boundary

LPS is independent of C2PA. Its current selector carrier, use of the
`c2pa-text` library, native envelope handling, and registry recovery do not
establish C2PA conformance, a Content Credential, JUMBF output, a COSE claim
signature, SynthID interoperability, or conformance with another AI-watermarking 
system.

The audited reference implementation is limited to manifest creation, signing,
compression, carrier embedding, verification, and registry recovery. It is a
bounded implementation record, not a claim that an origin assertion is true,
that a signer is authorized, or that a provider originated the asserted
content.

## 2. Reference architecture

```text
visible text → manifest → signed envelope → compressed CBOR → embedded carrier
visible text + carrier → extraction → validation → cryptographic verification → result
carrier unavailable → exact canonical-text hash → registry recovery result
```

The main boundaries are:

| Element | Role in the audited reference implementation |
|---|---|
| Manifest | Holds text binding, segments, `content_signed_at`, and confidence provenance. |
| Authenticated envelope | Carries direct outer `ev: 1`, outer `signed_at`, signature metadata, and the manifest. |
| Compression and carrier | Encodes the envelope as CBOR and carries it without altering visible text. `ev` remains a direct outer field, not a `FIELD_MAP` mapping. |
| Verifier | Classifies carrier condition, validates the envelope, verifies certificate/signature material, checks text binding, and returns a deterministic result. |
| Registry | Provides exact canonical-text-hash recovery only when a carrier is unavailable or unusable. |

The manifest's `content_signed_at` is the generating source's content-record
signing or commitment time. Outer `signed_at` is the time LPS signed the
complete envelope. Both values are authenticated, and an inner `signed_at` is
not part of the current contract.

Text binding strips trailing U+000D, U+000A, and U+0020, encodes the resulting
visible text as UTF-8, then derives both SHA-256 `text_hash` and byte
`text_length` from that one byte sequence. The verifier treats the hash and
length as one binding contract.

Confidence provenance distinguishes `tool` from `fallback`. The latter is an
LPS-calculated fallback, not a provider or classifier attestation. The locked
regression object is:

```js
{ ai_generated: 82, ai_modified_human: 15, human: 1 }
```

## 3. Verification and recovery model

Envelope structure is a security boundary, not an input-preparation detail.
Duplicate top-level CBOR envelope keys reject before version routing,
cryptography, certificate retrieval, registry access, or fallback:

```text
invalid_envelope / noncanonical_encoding / present
```

Likewise, a parseable invalid envelope remains `invalid_envelope`; it is never
reclassified as recoverable registry evidence.

The carrier and registry routes are deliberately distinct:

```text
valid carrier                 → verified or failed
parseable invalid envelope    → invalid_envelope; no fallback
absent/corrupted/unparseable  → exact-hash registry recovery
exact registry match          → registry_required / registry_match
no registry match             → degraded / registry_no_match
transport or HTTP failure     → degraded / registry_unavailable
malformed/incomplete response → degraded / registry_response_invalid
```

An exact registry match corroborates that the canonical visible-text hash has a
generation-time record. It does not restore segment data, explain carrier
absence or damage, prove stripping, identify a provider, authenticate an
issuer, or convert a carrier-free artifact into `verified`.

## 4. Current validated scope

The audit confirms the following bounded results:

- assertion-backed coverage for the locked envelope, malformed-carrier,
  registry, confidence, signing, and verification contracts;
- live, read-only exact-match registry recovery returning
  `registry_required / registry_match / absent`;
- live, read-only no-match behavior returning
  `degraded / registry_no_match / absent`;
- verification through the configured allowed HTTPS certificate route; and
- visible-text tamper detection returning `text_hash_mismatch`.

The stale confidence, signing, and verification regression tests were corrected
to assertion-backed current contracts. This evidence confirms the audited
reference path; it does not establish registry availability, production key
management, certificate governance, or service operations.

## 5. Trust model and limitations

The carrier and visible text are untrusted until verification completes. The
allowed HTTPS certificate route, certificate fingerprint, and signature
verification are confirmed boundaries. They do not establish a trusted issuer,
revocation handling, rotation, lifecycle governance, or an authorized-issuer
status.

Test-only signing material and dotenv use were sufficient for the audited test
context only. They do not establish production key custody, credential
isolation, or a production credential policy.

LPS currently establishes neither provider-origin attestation nor formal
provider, issuer, or `generating_id` identity semantics. A source hash,
certificate fingerprint, hosted certificate, or valid signature is not
independent provider-origin evidence and does not authorize an issuer.

## 6. Production exclusions and review topics

No production deployment approval follows from the reference-implementation
audit. The following remain deferred:

- certificate issuer trust, revocation, rotation, and lifecycle governance;
- production key management and credential isolation;
- a complete canonical-CBOR profile and decoder resource bounds;
- broader cryptographic-profile decisions, including P-256 and HMAC/HKDF;
- formal provider, issuer, and `generating_id` identity semantics;
- registry SLOs, monitoring, retry policy, incident response, and rollback;
- provider-attestation architecture and authorized-issuer governance; and
- any C2PA, SynthID, or general watermarking interoperability/conformance
  claim.

These are suitable working-group questions because they require policy,
governance, and independently reproducible validation—not merely further
reference-implementation changes.

## 7. Documentation and review entry points

- https://github.com/systemacticco-rgb/linguistic-provenance-schema/blob/main/README.md
gives the concise public overview.
- https://github.com/systemacticco-rgb/lps-reference-implementation/blob/main/SPEC.md owns
the normative envelope, timestamp, text-binding, validation, carrier-routing, registry-response,
and confidence contract.
- https://github.com/systemacticco-rgb/lps-reference-implementation/blob/main/DIAGRAMS.md
explains component and failure
boundaries.
- https://github.com/systemacticco-rgb/lps-reference-implementation/blob/main/SECURITY_MODEL.md owns trust limits and production exclusions.
- https://github.com/systemacticco-rgb/lps-reference-implementation/blob/main/IMPLEMENTATION_STATUS.md records the audit evidence and follow-ups.

## 8. Separate Proposal 007 testing-tool evidence

Proposal 007 is a separate cooperative-marker testing-tool effort. Its
2026-07-31 local observations do not amend this submission's LPS `ev: 1`
envelope, selector-carrier, registry, cryptographic, or production-trust
contract. They are bounded evidence about the Proposal 007 marker parser and
tested clipboard/rendering paths.

### 8.1 Proposed testing-tool architecture — ADR 2

ADR 2 is a proposed, tool-only marker/header design record. It uses an AI-only
model, a document-scoped base-5 total-count header, and an approved
U+2060–U+2064 library. Human spans and per-span ordinal or total-count fields
are excluded. The resulting header can support count-level anomaly reporting,
but cannot locate an ordinal gap or preserve the former human-marker
selective-stripping signal. Lens 200 remains undefined for inputs above 200
visible characters and has no LPS v0.1 or production requirement.

The tool normalizes received codepoints before scanning. An approved-library
codepoint in a valid pair's content is a forensic internal-signal observation;
it does not authenticate the content, establish AI origin, or make a valid
pair an LPS provenance record, and does not remove the pair from the tool's
valid-pair count. Marker forgery and removal remain possible for any party able
to edit text, and a missing marker does not prove stripping, intent, authorship,
or human origin.

| Tested case | Observed result | Testing-tool verifier result |
|---|---|---|
| Firefox/Linux hold-and-drag copy/paste | 100% marker survival in the tested flow | Valid |
| Firefox/Linux double-click copy/paste | 100% marker survival; no trailing space observed | Valid |
| Tested BiDi-language content | 100% marker survival despite highlighting glitches | Valid |
| Malformed sequence | Correct rejection | `E-0-0-2: INVALID_TYPE` |
| Duplicate header | 100% survival; correct rejection at normalized index `5` | `E006: DUPLICATE_HEADER` |
| Orphaned open marker | Correct rejection at normalized index `5` | `E-007: ORPHANED_OPEN` |
| Orphaned close marker | Correct rejection at normalized index `34` | `E-008: ORPHANED_CLOSE` |
| Trailing normalization | Correct behavior; 100% marker survival | Valid |
| Internal codepoints | 100% survival; correctly detected | `E-009: INTERNAL_SIGNAL` |

The current testing-tool direction sets headers as document-scoped, requires
support for differing header sizes, leaves Lens 200 undefined and
testing-tool-only, and excludes human spans, per-span ordinals, and per-span
total-count fields. Internal codepoints inside valid marker context use the
internal-signal path.

### 8.2 Recorded cross-transport observations

“100% survival” below means only that the tool found valid
signals after the named exercised route; it is not an interoperability,
clipboard, provenance, authorship, or stripping conclusion.

| Target | Recorded result | Scoped route observation |
|---|---:|---|
| Facebook Messenger | 100% survival | Desktop send → logout/login → mobile copy → resend → verifier. Double-click selection added trailing U+0020; iPhone-to-macOS Universal Clipboard succeeded. |
| Telegram | 100% survival | Desktop send → mobile copy → resend → verifier. Double-click or precise highlighting was needed for reliable selection; Universal Clipboard succeeded. |
| WhatsApp | 100% survival | Desktop send → mobile copy → resend → verifier. Double-click or precise highlighting was needed for reliable selection; Universal Clipboard succeeded. |
| Universal Clipboard, iPhone → macOS | 100% survival | A payload passed from the Claude mobile app to macOS and validated in the tool. |
| Facebook Web, macOS | No post survival recorded | Composer retained signals; the observed post path did not. The record does not identify a cause. |
| Instagram Web, macOS | 100% survival | Composer and post retained valid signals; Universal Clipboard was observed. |
| ChatGPT Web and desktop app, macOS | 100% survival | Sent messages retained valid signals when checked after days. |
| Claude Web and desktop app, macOS | 100% survival | Sent messages retained valid signals when checked after days. |
| Gemini Web, macOS | 100% survival | Sent messages retained valid signals. |
| X | 100% survival | Signals survived posting; no conclusion follows for video or photo editors. |
| Photoshop | 100% survival | No loss was observed; this does not generalize to video or photo editors. |
| Android and iOS mobile browser/tool use | 100% survival | The testing tool opened and validated on both mobile platforms. |
| Reddit, macOS browser | 100% survival | Valid signals survived after posting. |
| Notion desktop app, macOS | 100% survival | Valid signals survived copy/paste. |
| Slack desktop app, macOS | 100% survival | Valid signals survived copy/paste and sending. |

Selection and highlighting are part of the received-input boundary. A trailing
character or no-valid-signal result describes the sequence received by the
tool; it does not identify a responsible service, establish mutation, or infer
provenance. “No visual glyphs” is a rendering observation only, not
accessibility, font, renderer, or cross-platform evidence. The earlier report
of a different, longer LPS carrier on Facebook is not comparative evidence for
this Proposal 007 marker tool.

The observations are deliberately narrow. Firefox/Linux results do not prove
universal browser, operating-system, editor, clipboard, rendering, or
provider compatibility. Rendering is application-specific: LibreOffice on
Linux showed no glyphs; Linux VS Code showed yellow outlined squares in code
files only; Windows VS Code showed rectangles; Windows browser testing showed
no visual glyphs; and Windows OneNote rendered `ƒ{}`. Rendering alone is a
usability/disclosure concern, not evidence of marker corruption.

Open review work remains to identify the source of trailing spaces, repeat
clipboard paths across browsers, systems, and applications, determine whether
BiDi highlighting can affect selection boundaries or codepoint order, and
version-test glyph rendering. ADR 3 does not establish Windows or Linux route
behavior, external API or LLM transit, sanitizer treatment, accessibility, or
editor generalization. The shown local-test output identifiers and normalized
indexes remain separate from ADR 2's proposed detailed error catalog until a
Proposal 007 catalog reconciliation is approved. All Proposal 007 material
remains separate from the LPS implementation ADR and current v0.1 contract.

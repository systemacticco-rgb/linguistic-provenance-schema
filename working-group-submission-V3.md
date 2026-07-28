
# LINGUISTIC PROVENANCE SCHEMA (LPS)
### A proposed text profile for C2PA: span-level AI-contribution provenance

**Status:** [IMPLEMENTED / PROPOSED] v0.1 draft for working-group review
**Author:** Brayan Daniel Rodriguez Lugo
**Contact:** rodriguezlugobrayandaniel@gmail.com
**Date:** July 2026

---

1. INTRODUCTION

  This document is a v0.1 working-group submission for the
  Linguistic Provenance Schema (LPS), a proposed C2PA text profile
  for span-level AI-contribution provenance. It is submitted to
  surface inconsistencies in architectural reasoning and invite
  technical review before the implementation continues. This is a
  solo contribution. The reference implementation is functional and
  locally tested. Known constraints, open questions, and unresolved
  architectural items are stated explicitly throughout.

  [PROPOSED] LPS v0.1 is the portfolio's dominant long-term
  signed-manifest system. PROPOSAL 005/A.8R is complementary
  redundant-manifest research for LPS v0.1, not an independent system
  or C2PA Text A.9. PROPOSALS 006 and 007 are optional mechanisms that
  do not replace LPS v0.1; PROPOSAL 007 is a parallel, provisional,
  cooperative, removable, forgeable, non-forensic AI-only marker proposal.
  Human span markers are removed from its main grammar. Variants 007-A and
  007-B remain documentation-only fallbacks.

  [OBSERVED — SCOPED] Appendix D records browser testing-tool observations
  for PROPOSAL 007's cooperative marker grammar. That evidence is separate
  from the signed A.8 manifest transport evidence in Appendix C: it does not
  authenticate an issuer, provider, author, or AI origin, and it does not
  alter the v0.1 signed-manifest verification state model.

  [LAW / COMPLIANCE] Article 50 is an external adoption and compliance
  context, not proof that any LPS carrier is compliant. The Code of
  Practice is voluntary. Provider cooperation, transport preservation,
  renderer behaviour, crawler interpretation, and regulator scope are
  external dependencies.

  [HOLD — LANGUAGE MIGRATION] JavaScript/Node.js remains the reference
  implementation environment. Runtime latency, typed integration
  contracts, binary-hash handling, CPU concurrency, and cross-language
  transport are production dependencies, not working-group submission
  blockers; they block production-deployment and commercial-SLA claims.

## ABSTRACT

C2PA records that AI was involved in producing an asset. It does not
record how much, which parts, or in what manner. LPS is a proposed
C2PA assertion schema that records, at the level of spans within a
text, what proportion was human-authored, AI-generated, or human text
subsequently modified by AI — together with a confidence and, for
modified spans, a degree of modification. LPS introduces no new signing, embedding, or storage machinery of
its own. It is architected to reuse C2PA's existing text embedding
methods, claim-signature format, manifest repository, soft-binding
resolution, time-stamp authority integration, ingredient model, and
trust list — the trust list and TSA integration are architectural
reuse targets, not yet wired into the v0.1 reference implementation;
see §8.9 and §8.2. Its only contribution is the
assertion schema, the confidence source contract, the mathematical
confidence fallback mechanism, and the small technical extension
required to make C2PA assertions address sub-asset spans of text
rather than whole assets. LPS is designed to be layered with C2PA
file metadata and with statistical watermarking (e.g. the SynthID
class); it is not a replacement for either.

---

## 2. PROBLEM STATEMENT

An origin claim about a text is today asserted but not testable,
and the cost of a false claim falls on a third party who had no
way to check. Three concrete harms follow.

**False authorship in official documents.** A filing, statement, or
affidavit presented as human-authored may be wholly or partly
machine-produced, with no persistent signal of which.

**Credential and identity fraud.** Written work samples, claimed skills,
and qualifications can be AI-generated and passed off as a person's
own capability.

**Institutional vetting failure.** A company hiring, an editor publishing,
or a regulator reviewing is forced to accept a document with no
provenance and no means to test the origin claim.

The technical root is the export gap, demonstrated by the HaLLMark
research (Hoque et al., CHI 2024): AI contribution can be tracked
while writing, but once accepted the AI-originated text is visually
indistinguishable from human text, and on export the record
disappears. No shipped standard fills this gap at sub-document
granularity.

---

## 3. RELATIONSHIP TO C2PA

LPS is a text profile, not a parallel system. Stated explicitly:

**Reused unchanged:**
- Text embedding methods (variation-selector and
  structured/distributed carriers).
- Claim-signature format and time-stamp authority integration.
- Manifest repository and soft-binding resolution (used here
  for text recovery).
- Ingredient model (used here for multi-round provenance).
- Trust list (specified for both signing and registry write
  authorization; not yet implemented in the v0.1 reference
  implementation — the registry stub currently uses a single
  service-role credential as its access boundary. See §8.9).

**Extended:**
- Assertion target moves from whole-asset to sub-asset spans
  of text (offset- or hash-anchored ranges).

**Added (the contribution):**
- A standardized assertion schema for granular AI-contribution
  provenance: per-span origin (human / ai_generated /
  ai_modified_human), confidence, modification degree, and the
  derived overall human/AI proportions.

- A confidence source contract defining the AI tool as the primary
  and authoritative source of confidence values, with a defined
  fallback hierarchy and a mathematical derivation method when no
  tool-supplied value is available.

- An explicit algorithm-naming convention: the manifest's algorithm
  field carries the internal value es256, identifying ECDSA P-256
  with SHA-256 and IEEE P1363 raw r‖s signature encoding. This is
  a naming convention internal to LPS — it is not a JOSE or COSE
  algorithm identifier and does not imply the manifest is a COSE
  or JOSE structure. See §8.8. 

- A redundant embedding architecture (PROPOSAL 005), specified
  but not yet implemented, that would provide anchor manifests
  at paragraph boundaries and overlapping full manifest copies
  with cross-copy reconstruction — intended to increase signal
  survival across partial copy, signal stripping, and adversarial
  removal scenarios. See §4.3, §8.

**Verification state model — four states built, four specified:**

Built and tested in the v0.1 reference implementation:
- **verified** — signal intact, signature valid, text hash matches.
- **failed** — signal found but signature invalid or text modified.
- **degraded** — signal absent or corrupted, no registry record.
- **registry_required** — signal absent, exact visible-text hash
  matches an existing generation-time registry record.

Specified under PROPOSAL 005 (redundant embedding architecture,
§4.3), not yet implemented, targeted for v0.2:
- **anchor_only** — no full manifest recoverable, anchor manifests
  present and valid. Would return document-level fields only.
- **partial_recovery** — manifest partially reconstructed from
  surviving chunks across redundant copies. Would return
  reconstruction completeness percentage and gap map.
- **injection_detected** — conflicting certificates found across
  chunk assemblies. Would return both certificate fingerprints
  as forensic evidence of adversarial injection attempt.
- **reconstruction_corrupted** — checksum mismatch on buffer
  assembly after cross-copy reconstruction. Return fields and
  failure subcategories are not yet defined. This state is
  planned under PROPOSAL 005, not specified. It is listed here
  for architectural completeness only.
---

## 4. ARCHITECTURE

### 4.1 Bindings

LPS uses two binding types, in C2PA's vocabulary. A hard binding:
SHA-256 of the visible text, computed at signing time and
re-computed at verification time; exact, breaks on any byte change.
A soft binding: the invisible carrier embedded in the text. Its
preservation across copy-paste is transport-dependent; it does not
survive lossy reproduction. The signed manifest travels in the soft
binding; the hard binding is what the recovery registry indexes.

A.9 structured extraction has been removed from the v0.1 verification 
path. A.8 is the only extraction path. Redundant copies per paragraph 
are part of PROPOSAL 005's future A.8R carrier, specified but not yet 
implemented.

[SECURITY / LIMITATION] A.8 and the proposed A.8R carrier retain the
Unicode-conformance concern identified for the C2PA unstructured
variation-selector scheme. This is a carrier, standards, and adoption
limitation; it is not a claim that the cryptographic signature layer is
invalid.

[LIMITATION] Registry recovery can corroborate only an exact
visible-text hash for which a generation-time record exists. It does
not restore span-level evidence.

The hard binding is computed from the raw visible text in v0.1.
A trailing whitespace normalization rule — /[\r\n ]+$/ stripping
trailing carriage returns, newlines, and spaces — has been
identified as necessary to handle deterministic editor behavior
observed during the July 2026 survival study: Google Docs appends
U+000A on copy-out; Word Browser appends U+0020. Without
normalization both produce a hash mismatch despite the manifest
surviving intact. This normalization rule is post-submission work
and is not implemented in the v0.1 reference implementation.

### 4.2 Confidence source contract

The generating AI tool is the authoritative source of confidence.
When an AI tool produces or modifies a span, its API response must
supply the confidence value for that span directly. This is the only
source that reflects first-person certainty about the origin
classification. No other source supersedes it.

When the generating tool does not supply a confidence value, the
following sources are used in priority order:
1. Output from an approved AI detection classifier, mapped 0–100.
2. Human reviewer manual assignment, integer 0–100.
3. Mathematical fallback derivation.

Mathematical fallback: when no confidence value is supplied,
the system calculates an approximation from the document's segment
character distribution. For each origin type, the fallback confidence
equals the percentage of total document characters belonging to that
origin type, floored to the nearest integer. Example: a document
where 81.66% of characters are AI-generated produces a fallback
confidence of 81 for AI-generated segments. This value is a
structural approximation, not a forensic measurement. Every segment
in the manifest carries a confidence_source field recording whether
the value came from the tool, a classifier, a human reviewer, or
the mathematical fallback. This field survives compression,
embedding, and verification intact.

### 4.3 Redundant embedding architecture

[PROPOSED] PROPOSAL 005/A.8R is complementary redundant-manifest
carriage and partial-recovery research for LPS v0.1, not an
independent system or C2PA Text A.9. To improve recovery opportunities
after partial copy, stripping, or platform normalization, it specifies
a two-layer architecture that is not part of the v0.1 reference
implementation. Redundancy does not cure the A.8 carrier's
Unicode-conformance concern or establish transport survival.

**Layer 1 — Anchor manifest:** a minimal manifest containing
document-level fields only (text_hash, overall_ai_proportion,
human_proportion, algorithm, signed_at) would be embedded at the
start of every paragraph using the A.8 carrier method. Anchors
would be HMAC-protected to limit forgery. [HOLD] Their survival in
short copies and their ability to provide a document-level picture
remain unimplemented recovery hypotheses, not demonstrated results.

**Layer 2 — Overlapping redundant full manifest copies:** one complete
signed manifest copy would be embedded per paragraph using A.8R, an
A.8-derived redundant invisible variation-selector chunk carrier.
A.8R is not C2PA Text A.9. The C2PA Text A.9 structured visible-text method remains outside both v0.1 and PROPOSAL 005 A.8R scope. Adjacent A.8R copies would overlap
by 25% of their chunk range. Each chunk would carry a positional
header (sequence number, total count, copy identifier, type flag) so
surviving chunks from damaged copies could be identified and
combined across copies to reconstruct the full manifest even when
no single copy survived intact.

Reconstruction logic (specified): the verifier would collect all
surviving chunks, group by sequence number across all copies, fill
gaps in one copy using matching sequence positions from other
copies, and validate the assembled buffer via a SHA-256 checksum
stored in the first chunk. The number of complete copies would
scale automatically with document length — longer documents
carrying more redundancy.

### 4.4 Defense-in-depth across four channels

**Channel 1 — copy-paste** (carrier and visible text both preserved).
Defended by the embedded signed manifest. Deterministic,
returns verified.

**Channel 2 — targeted strip** (visible text preserved byte-for-byte,
carrier absent). A registry lookup can corroborate a matching
generation-time record only when the exact visible-text hash is
available. It neither establishes that stripping occurred nor restores
span-level evidence. PROPOSAL 005's anchor manifests, once
implemented, may extend recovery opportunities after partial loss; see
§4.3, §8.

**Channel 3 — lossy reproduction** (OCR, photograph, retype; visible
bytes changed). Not covered by LPS. This is the domain of
statistical watermarking, which survives in word choice rather
than in characters.

**Channel 4 — heavy rewrite.** Irreducible residual; no provenance
layer reliably survives.

### 4.5 The four-cell verification matrix

- Carrier survived, bytes preserved → verified.
- Carrier survived, bytes edited → failed. Whether the original
  signed manifest is returned depends on a length-mismatch
  disclosure threshold locked at 10% of the signed text_length.
  If the received text length differs from the signed text_length
  by 10% or less, original_manifest is returned as proof of
  alteration alongside the original claim. If the difference
  exceeds 10%, original_manifest is withheld to limit what an
  adversary can learn from deliberate large-mismatch replay attempts.
- Carrier absent, bytes preserved exactly → registry recovery may be
  available by content hash only when a generation-time record exists.
- Carrier absent, bytes edited → unrecoverable. Carrier absence alone
  does not establish stripping, intent, or human authorship.

This matrix describes the baseline system. The redundant embedding
architecture (§4.3) extends Channel 2 defense and introduces
partial recovery paths not captured by the four-cell model.

---

## 5. THREAT MODEL

A provenance system exists only because origin will be lied about.
Named adversaries:

**Strip** (remove carrier, leave text intact) — [LIMITATION] registry
hard-binding recovery can corroborate only an exact visible-text hash
with an existing generation-time record. A carrier's absence alone
does not prove stripping. PROPOSAL 005's proposed anchors may improve
recovery opportunities at paragraph boundaries but are not
implemented; see §4.3, §8.

**Forge** (fabricate a record claiming false origin) — PARTIALLY
DEFENDED: a forged signature does not validate against the
certificate-fingerprint check in the v0.1 reference implementation.
The trust list mechanism that would formalize signer authorization
is architecturally specified but not yet implemented — see §8.9.
Anchor HMAC protection against anchor forgery is specified under
PROPOSAL 005, not yet implemented — see §4.3, §8.

**Transfer/replay** (lift a valid signal onto different text) —
DEFENDED: the text-hash binds the manifest to its specific visible
text; a mismatch returns `failed`. A length-based disclosure
threshold, locked at 10%, withholds `original_manifest` when the
received text's length differs from the signed `text_length` by
more than that margin — preventing an adversary from using
extreme-mismatch replay to study document structure via repeated
submissions. `text_length` is a plain manifest field requiring no
separate cryptographic protection; it is covered by the same
signature as every other manifest field. See SPEC.md §4.1 (dictionary
entry) and §9 (threshold lock).

**Impersonation** (sign as an issuer one is not) — SPECIFIED, not yet
  enforced: the trust list mechanism that would defend against
  this is architecturally defined but not implemented in v0.1.
  See §8.9.

**Truncation** (present only the latest round, hide that earlier
rounds existed) — OPEN: undetectable from the text alone;
requires the registry to hold the chain. See §8.3.

**Manifest injection via foreign certificate** (adversary generates
own keypair, injects chunks with a valid but untrusted
certificate into a legitimately embedded document) — SPECIFIED,
not yet implemented (PROPOSAL 005): the verifier would pin the
certificate from the first valid assembly as the session anchor,
flagging any subsequent assembly producing a different certificate
as injection_detected with both certificate fingerprints recorded
as forensic evidence. This defense applies to the redundant
multi-copy embedding architecture and has no equivalent in the
single-copy v0.1 embedding model.

**Anchor substitution** (adversary replaces anchor manifests with
falsified proportions) — SPECIFIED, not yet implemented
(PROPOSAL 005): anchor HMAC validation would discard anchors
whose HMAC does not verify against the signing key derivation.
Stripped anchors would be recorded as anchor_layer: absent in
the verified output. The anchor layer, and therefore this
defense, does not exist in v0.1 — there is no anchor manifest
concept in the current embedding model. HMAC key-derivation
architecture is under active design; see §8.

**Registry poisoning** (adversary floods registry with fake records)
— PARTIALLY DEFENDED: content_hash format validation (64
lowercase hex characters) and a minimal safety-only check on
generating_id (printable ASCII, 1-128 characters, no control
characters) are implemented in the v0.1 registryClient.mjs ahead
of the Supabase insert. The generating_id check is intentionally
not a structural format validation — the identity/version schema
question remains open (SPEC.md §9) and is expected to be resolved
after working group input rather than decided unilaterally in
advance. Rate limiting is specified (SPEC.md §9) but not yet
implemented. Unlike the anchor and redundant-embedding defenses
above, the remaining rate-limiting gap does not depend on
PROPOSAL 005 or the pending key-hierarchy decision and is
buildable independently before submission.

**Magic prefix collision** (adversary crafts buffers that match
the LPS magic prefix by chance or design) — SPECIFIED, not yet
implemented (PROPOSAL 005): secondary validation after magic
prefix match would confirm type field is 0 or 1, version field
is 1, and total field is greater than 0, with an injection volume
cap discarding excess buffers beyond the expected count. The
magic-prefix chunk header does not exist in v0.1 — this defense
belongs entirely to the chunkLayer.mjs component, not yet built.

**Carrier-as-injection-channel** — ADDRESSED, see 5.1.

### 5.1 The injection duality

[SECURITY] Invisible Unicode can be used in prompt-injection attempts,
but invisible codepoints alone establish neither malicious intent nor
AI origin. The proposed defense is an input policy that would extract
candidate invisible payloads and test for a valid LPS signature. A
valid signature supports an integrity check; without the proposed
trust list it does not independently establish an authorised issuer.
Unsigned payloads may be quarantined or removed under a recipient's
security policy, but their absence of signature is not proof of
malice. This filter and the trust list it depends on are not
implemented in v0.1; see §8.9. A planned HMAC-protected anchor layer
(PROPOSAL 005, not implemented) is a separate proposed signal, not
present v0.1 evidence of injection.

---

## 6. DEMONSTRATION

A reference implementation produces and verifies LPS manifests
end to end. The pipeline is four stages: manifest generation
(segments → JSON manifest with text-hash, derived proportions,
and confidence_source per segment), signing (canonical-CBOR bytes
signed with ECDSA P-256, certificate delivered by public URL plus
fingerprint), embedding (compressed CBOR manifest in the text
carrier; the v0.1 root plain-text path uses A.8 invisible variation
selectors), and verification (four states — see §3).

Verified in memory: a freshly embedded text returns verified,
with the recomputed clean-text hash matching the signed text-hash
exactly, and the signature validating over canonical CBOR
(encoding independent of key order, resolving the
implementation-dependent-verification weakness identified in
independent C2PA analysis). Signatures use the IEEE P1363 raw r‖s
encoding specified by the ES256 algorithm identifier — confirmed
via internal round-trip testing and an independent cross-check
against the panva/jose library, an unrelated, spec-validated
JOSE implementation with no shared code path. This confirms the 
underlying signature primitive — curve, hash,
and byte encoding — is spec-compliant at the primitive level, not
merely self-consistent within this codebase. It does not mean the
LPS manifest itself is JOSE/COSE-compatible as a structure: the
manifest is not currently wrapped in a COSE_Sign1 or compact JWS
envelope, and a party with off-the-shelf COSE/JOSE tooling cannot
today parse an LPS-signed manifest directly. Full envelope-level
interoperability is a post-v0.1 target — see §8.8.
All four built states reproduced live: an unedited round-trip
returns verified; adding a visible character returns failed with
the original manifest; deleting into the carrier region returns
degraded; carrier-absent exact text can return registry recovery only
when a generation-time record exists. The remaining four states (anchor_only, partial_recovery,
injection_detected, reconstruction_corrupted) belong to PROPOSAL 005
and will be demonstrated once that architecture is implemented —
see §3 and §8.

[PENDING CROSS-CHECK] The following July transport observations are
scoped evidence, not a universal preservation, renderer, or platform
interoperability claim. Provider and platform behaviour remain
external dependencies. A matrix of 37 verification runs across 13
editors and platforms was collected July 7 2026. Eleven of the
thirteen observed editors and platforms preserved the signed text
without visible-text modification. Two (Google Docs and Word Browser)
introduced deterministic trailing whitespace during copy-out,
producing hash mismatches while preserving the embedded manifest and
signature. Two editors produced hash mismatches caused by
trailing whitespace appended automatically on copy-out — Google Docs
appends U+000A, Word Browser appends U+0020. The identified trailing
whitespace normalization rule (§4.1) is not implemented in v0.1. No
editor in this matrix stripped the carrier. The signal survived WhatsApp (sent), Telegram (sent), Facebook
(posted), LinkedIn (posted), Instagram (posted caption), ChatGPT
(sent, Chrome and Safari), and Claude (sent, Chrome and Safari).
Messaging platforms including Telegram, WhatsApp, ChatGPT, and Claude
strip trailing whitespace automatically on send, making the
normalization rule redundant on their send paths but not in compose.
Facebook does not normalize on post. The carrier is visible to
platform character counters — X/Twitter counts variation selectors
against the character limit, which is indirect confirmation the
carrier is present on that platform. A full transport matrix is
included as Appendix C. These observations do not establish that a
carrier survives every transport, that an absent carrier was stripped,
or that any platform has a particular intent.

The separate PROPOSAL 007 browser-tool evidence in Appendix D concerns
U+2060–U+2064 cooperative markers, not the A.8 signed-manifest carrier.
It must not be read as an additional signed-manifest transport result.


---

## 7. PRIOR ART COMPARISON

| Capability | C2PA | SynthID | HaLLMark | LPS |
|---|---|---|---|---|
| Sub-document granularity | No | No | tracking only, no export | Yes |
| Cryptographic binding | Yes | No | No | Yes |
| Carrier survival copy-paste | Yes | Yes | No | Scoped evidence² |
| Registry recovery after strip | Yes | No | No | Exact-hash record only² |
| Confidence scoring per span | No | No | No | Yes |
| Modification degree per span | No | No | No | Yes |
| Redundant embedding with reconstruction | No | No | No | Specified¹ |
| Lossy reproduction survival | No | Yes | No | No |

¹ Specified under PROPOSAL 005, not yet implemented in the v0.1
reference implementation. Targeted for v0.2. See §4.3, §8.

² [PENDING CROSS-CHECK / LIMITATION] LPS transport observations are
scoped, not universal preservation claims. Registry corroboration
requires an exact visible-text hash and an existing generation-time
record; it does not restore span-level evidence or establish stripping.
---

## 8. OPEN QUESTIONS WITH PROPOSED DIRECTIONS

Sections 8.1–8.9 form a single family — temporal validity and
provenance chaining across key and edit lifecycle events — and
are presented together rather than as scattered gaps. Sections
8.10 and 8.11 are separate, later-identified open questions
(replay-disclosure threshold and canonicalization determinism,
respectively) and do not belong to that family.

### 8.1 Recovery registry in production

The registry is a recovery index, not the source of truth; the
signed embedded manifest is. [LIMITATION] It can corroborate an
existing generation-time record only for an exact visible-text hash;
it does not recover span-level evidence and cannot establish that an
absent carrier was stripped. Proposed
direction: structure it as a Certificate-Transparency-style
append-only log, serving the role of a C2PA manifest repository
for text, queried via a soft-binding-resolution-equivalent path
keyed on the visible-text hash. Access: public read (verification
is a public good), trust-list-gated write (one trust mechanism
for signing and registration), every query logged with consumer
notice per C2PA privacy guidance.

### 8.2 Revocation, certificate expiry, emergency rotation

A provenance record cannot be revoked; a signing key can.
Proposed direction: append-only, versioned, immutable certificates
(added, never replaced, never deleted), so any past document can
always fetch the exact certificate that signed it, confirmed by
fingerprint. Trusted timestamping (RFC 3161, adopting C2PA's
existing COSE time-stamp authority integration) then resolves
three things at once: legal-grade "when," before/after
determination for revocation so honest pre-compromise signatures
survive, and expiry semantics so a signature made while a
certificate was valid remains verifiable after expiry.

### 8.3 Multi-round provenance

Reuses C2PA's ingredient model: each editing round emits a
manifest asserting the prior manifest's hash as an ingredient.
There is no hard byte ceiling in the A.8 carrier implementation.
Production manifest size is governed by editor copy-paste latency,
platform reclassification behavior, and token overhead — not by a
protocol-enforced limit. See §4 and the production constraints
documentation in SPEC.md §4.2 for operational guidance.
The genuinely unsolved problem, named precisely as the primary
post-v0.1 research item: span
survival under editing. Offset-anchored spans break when a later
round inserts or deletes text. Two candidate directions, neither
clean — anchor spans to a hash of their own text (survives
reordering, breaks on in-span edits), or re-segment each round
and preserve provenance only at the manifest-chain level (robust,
loses per-span lineage). Truncated chains are undetectable from text alone and depend on the registry holding the full chain.
Chain depth limit and poisoned chain detection are open
architectural questions proposed for working group resolution.

A related and currently undefined contract covers overwrite and re-signing: when an AI tool receives text already carrying a valid LPS manifest and produces modified output, no contract exists for (a) whether the output must carry a new manifest referencing the prior as a provenance chain, (b) whether manifest bytes surviving in the output must be stripped before re-signing, and (c) whether a verifier encountering a physically present but semantically invalid manifest from a prior signing cycle must surface that condition explicitly as a distinct verification state. This must be defined before any AI tool integration guidance is published.

### 8.4 Segment boundary granularity

The system requires the generating AI tool to report segment
boundaries at generation time. The granularity at which boundaries
must be reported is not yet defined. Proposed direction: sentence
level as the minimum required granularity. Word level is too
granular for reliable tool reporting. Paragraph level loses
forensic precision. Sentence level balances practical
implementability with forensic utility. This definition is
required before working group submission and before third-party
tool integration can begin.

### 8.5 Confidence source and fallback

The AI tool is the authoritative source of confidence values.
The mathematical fallback defined in §4.2 is a structural
approximation. The forensic weight a court or institution should
assign to fallback-derived confidence values versus tool-supplied
values is an open legal and evidentiary question proposed for
working group input. The confidence_source field in the manifest
ensures the distinction is always preserved and visible to
verifiers.

### 8.6 Identity and privacy

The registry stores content hashes, not content, so it is likely
outside personal-data scope. The generating-identity field is the
exception: depending on what it encodes, it may be personal or
pseudonymous data. This is flagged as an open data-protection
question, not asserted clean.

### 8.7 Cross-registry governance
Jurisdiction, liability for false-positive matches, and retention
are open governance questions, proposed to follow Certificate
Transparency's governance precedent.

### 8.8 Envelope-level signature interoperability

The reference implementation's signature primitive (ECDSA P-256, SHA-256, IEEE P1363 raw r‖s encoding) has been confirmed interoperable with standard JOSE/COSE tooling at the primitive level via independent library cross-check. The manifest itself is not currently packaged inside a standard signing envelope — no COSE_Sign1 structure, no compact JWS. A party with off-the-shelf COSE or JOSE tooling cannot today parse and verify an LPS-signed manifest directly; they would need an LPS-aware adapter even though the underlying signature would validate. Proposed direction: adopt a COSE_Sign1 envelope for v0.2, since C2PA's own claim- signature format is already COSE-based and this would close the gap between "uses a C2PA-compatible primitive" and "is structurally a C2PA-compatible artifact." Working group input requested on priority and whether v0.1 submission should wait for this or proceed with the primitive-level interoperability as currently demonstrated.

### 8.9 Trust list implementation status
The trust list described in §3 as reused C2PA infrastructure is
architecturally specified but not implemented in the v0.1 reference
implementation. The registry stub currently uses a single Supabase
service-role credential as its write-access boundary — a placeholder,
not a trust list. Signature verification in v0.1 checks the
certificate fingerprint against the manifest, not against a list of
authorized signers. Any party who generates a keypair and hosts a
certificate can produce a manifest that v0.1 verification accepts.

Defining who is authorized to sign — and the governance mechanism
for granting and revoking that authorization — requires working
group input. This is not buildable unilaterally before submission
because the authorization model depends on decisions about registry
hosting, issuer identity, and trust chain governance that are
themselves open questions (§8.1, §8.7). This is a named gap in
v0.1, not deferred implementation work.

### 8.10 Replay disclosure threshold

The `failed` state discloses `original_manifest` only within a
length-mismatch threshold, locked at 10% of the signed
`text_length` (v0.1 reference implementation, verificationTool.mjs
STEP 4). Received text length differing from signed text length by
more than 10% withholds `original_manifest`, limiting what an
adversary can learn from extreme-mismatch replay attempts. A
manifest missing `text_length` fails closed with no disclosure
rather than falling through to a disclosure decision. Consistent
with §5 "Transfer/replay" (DEFENDED) and SPEC.md §9. Confirmed via
pipeline-level tests: small-edit (within threshold) discloses;
extreme-mismatch (beyond threshold) withholds.

### 8.11 Canonicalization determinism across dependency upgrades

The v0.1 reference implementation currently guarantees that a signed
manifest's canonical byte representation stays reproducible only
because every deployment is pinned, via lockfile, to one exact
version of the CBOR encoding library used to produce those bytes.
This is sufficient for a single, non-distributed reference
implementation, but not yet resolved for either of two future
requirements: independent verifiers reproducing byte-identical
canonicalization after the encoding library is upgraded, and
PROPOSAL 005's cross-copy reconstruction, which must re-derive
canonical bytes from recovered fragments. Tagging a manifest with
its producing encoder version was considered and set aside: it
introduces a circular trust problem for reconstruction specifically,
since the verifier would need to already trust a field it cannot yet
verify in order to know which encoder to reconstruct with. This is
an open architectural question, not yet solved in the current
reference implementation, and will need resolution before PROPOSAL
005 ships or before implementation-level interoperability is
required.

## 9. LIMITATIONS

LPS proves the integrity of an origin claim, not its truth.
A verified record means the claim is authentic, the signature
validates against the certificate advertised in the manifest,
and the visible text is unaltered — not that a human "really"
wrote the human-attributed spans. In v0.1, issuer authorization
is not verified against a trust list. Any party who generates a
keypair and hosts a certificate can produce a manifest that v0.1
verification accepts. Verification that a record is signed by an
authorized issuer requires a trust list, which is a named gap in
v0.1 and is subject to working group input. See §8.9.It records what the signing tool
asserted at signing time. It does not independently establish
individual authorship unless issuer identity was itself verified
through the trust list.

LPS requires generation-time cooperation from the producing tool
(the same requirement C2PA and SynthID carry) and, for the
registry and any statistical-watermark layer, from model
providers. It does not survive lossy reproduction or heavy
rewrite (Channels 3 and 4).

The confidence fallback mechanism (§4.2) produces an approximation
derived from document-level character distribution, not a
signal-strength measurement. Manifests carrying fallback-derived
confidence values should be weighted accordingly in forensic
contexts. The confidence_source field makes this distinction
explicit and machine-readable.

The partial_recovery and anchor_only states are specified under
PROPOSAL 005 (not yet implemented) to represent incomplete
provenance recovery. A partial_recovery result would mean the
full segment breakdown could not be reconstructed — only the
portions that survived across redundant copies would be returned.
An anchor_only result would mean only document-level fields
survived. Neither state would be equivalent to verified. The
reconstruction_completeness percentage in partial_recovery would
provide a quantitative measure of how much of the provenance
record is present. None of this is testable until PROPOSAL 005
is built.

[PENDING CROSS-CHECK / LIMITATION] The A.8 invisible carrier is treated by operating systems as a
single grapheme cluster attached to the first visible character.
On platforms that expose this behavior — observed on macOS in
iMessage, Telegram, and WhatsApp — pressing backspace at the end
of the embedded text deletes the entire invisible carrier in one
or two keystrokes, without any visible indication that a
provenance signal was present or has been removed. The result is
a degraded state on verification, which is the correct and honest
outcome. This is not a defect in the carrier — it is an expected
consequence of how Unicode grapheme clusters are handled by text
input systems.A user who deliberately or accidentally backspaces
through the end of an LPS-embedded text will silently remove the
provenance signal. The degraded state records carrier absence without
attributing a cause.

[PENDING CROSS-CHECK / LIMITATION] LPS manifests cannot be embedded inside inline or fenced code syntax blocks in v0.1. Code renderers may display invisible Unicode variation selectors as visible replacement icons or colored markers, disrupting the carrier. GitHub file-level preservation is scoped transport evidence, not a universal claim. No carrier mechanism for code block provenance is defined in v0.1. This is a named scope exclusion, not an open architectural question — code block embedding requires a separate carrier definition before it can be considered for any future version.

---

## APPENDIX A — VERIFICATION OUTPUT (FOUR BUILT, FOUR SPECIFIED UNDER PROPOSAL 005)

### State 1 — verified

```json
{
  "status": "verified",
  "signed_at": "2026-07-08T02:38:14.081Z",
  "algorithm": "es256",
  "embedding_method_used": "A.8",
  "clean_text": "This is human written. This part was AI generated.",
  "disclosure_threshold_outcome": "not_applicable",
  "signed_text_length": 50,
  "received_text_length": 50,
  "overall_ai_proportion": 0.56,
  "human_proportion": 0.44,
  "segments": [
    {
      "segment_id": 1,
      "origin": "human",
      "confidence": 95,
      "start_offset": 0,
      "end_offset": 21,
      "ai_tool": null,
      "modification_degree": null
    },
    {
      "segment_id": 2,
      "origin": "ai_generated",
      "confidence": 88,
      "start_offset": 22,
      "end_offset": 49,
      "ai_tool": "claude-sonnet",
      "modification_degree": null
    }
  ]
}
```

### State 2 — failed (large modification, original_manifest withheld)

Text appended after signing exceeds the 10% disclosure threshold. Signature was not touched — manifest integrity holds. Visible text hash does not match.

```json
{
  "status": "failed",
  "reason": "Visible text was modified after signing — content hash does not match. Original manifest withheld: received text length differs from signed text length beyond the disclosure threshold.",
  "signed_at": "2026-07-08T02:38:14.081Z",
  "algorithm": "es256",
  "embedding_method_used": "A.8",
  "clean_text": "This is human written. This part was AI generated. TAMPERED",
  "disclosure_threshold_outcome": "exceeds_threshold",
  "signed_text_length": 50,
  "received_text_length": 59
}
```

### State 3 — failed (small modification, original_manifest disclosed)

Single-character append. Delta is within the 10% disclosure threshold. original_manifest is returned so the reviewer can compare what was signed against what was received.

```json
{
  "status": "failed",
  "reason": "Visible text was modified after signing — content hash does not match",
  "signed_at": "2026-07-08T02:38:14.081Z",
  "algorithm": "es256",
  "embedding_method_used": "A.8",
  "clean_text": "This is human written. This part was AI generated.!",
  "disclosure_threshold_outcome": "within_threshold",
  "signed_text_length": 50,
  "received_text_length": 51,
  "original_manifest": {
    "signed_at": "2026-07-08T02:38:14.081Z",
    "overall_ai_proportion": 0.56,
    "human_proportion": 0.44,
    "segments": [
      {
        "segment_id": 1,
        "origin": "human",
        "confidence": 95,
        "start_offset": 0,
        "end_offset": 21,
        "ai_tool": null,
        "modification_degree": null
      },
      {
        "segment_id": 2,
        "origin": "ai_generated",
        "confidence": 88,
        "start_offset": 22,
        "end_offset": 49,
        "ai_tool": "claude-sonnet",
        "modification_degree": null
      }
    ]
  }
}
```

### State 4 — degraded

No embedded signal found in submitted text. Extraction returned null. No manifest, no signature, no segments. [LIMITATION] Absence of signal establishes neither stripping, intent, nor human authorship.

```json
{
  "status": "degraded",
  "reason": "No embedded signal found in input text",
  "limitation_note": "Carrier absence does not establish stripping, intent, or human authorship"
}
```

## APPENDIX B — PLAIN-LANGUAGE FORENSIC REPORT

*(worked example — illustrative, not an actual case)*

**What was examined.** A text document, identified by content hash
c09319f5…d187a, received on 25 June 2026.

**What was found.** A provenance record was present in the document
and cryptographically intact. The record was signed by [issuer,
per trust list] on [signing time, per trusted timestamp]. The
visible text matches the text recorded at signing; it has not
been altered since.

**What it means.** The record states the text is 100% human-authored
across one span, classified with 95% confidence from the
generating tool. This means the signed origin claim is authentic
and unaltered. It does not by itself prove a specific individual
wrote the text; it proves the issuing tool recorded this origin
at signing time and that the record has not been tampered with.

**Confidence and limitations.** This finding establishes the integrity
of the origin claim, not its truth. Absence of such a signal in
other documents would not be evidence of human authorship; signals
can be removed by retyping, reformatting, or character
normalization. A classification at 95% confidence should be
weighed as a probability contribution, in the manner of other
digital-forensic evidence, not as a standalone verdict. Where
confidence_source is fallback rather than tool, the confidence
value is a mathematical approximation and should be weighted
accordingly.

**Chain of custody.** The document was received as [source]; its hash
was computed on receipt and is recorded above.

**Variant — failed:** "The text examined differs from the text
recorded at signing time — it has been changed since the record
was made. The original record claimed the following: [original
manifest fields]. The difference between the received text length
and the signed text length determines whether the full original
record is disclosed or withheld for security reasons."

**Variant — degraded:** "No provenance signal could be recovered.
The text may never have carried one, or it was removed. Absence
does not establish stripping, intent, or human authorship."

**Variant — registry_required:** "No signal was found in the text,
but a matching record was located in the provenance registry by
an exact visible-text hash, indicating that identical text had an
existing generation-time record. This does not establish why the
carrier is absent or restore span-level evidence."

**The following three variants (anchor_only, partial_recovery,
injection_detected) illustrate PROPOSAL 005's intended output and
cannot be produced by the v0.1 reference implementation. They are
shown for architectural clarity, not as demonstrated results.**

**Variant — anchor_only:** "No full provenance record was recoverable.
Partial records were found at paragraph boundaries confirming the
document was signed on [date], with an overall AI proportion of
[value] and human proportion of [value]. The segment-level
breakdown is not available."

**Variant — partial_recovery:** "A partial provenance record was
recovered from [reconstruction_completeness]% of the embedded
signal. [N] of [expected_segment_count] segments were
reconstructed. The remaining segments were unrecoverable. The
recovered record should be treated as incomplete. Full
verification could not be performed."

**Variant — injection_detected:** "An adversarial injection attempt
was detected. A legitimate provenance record was found alongside
a second record carrying a different certificate. Both certificate
fingerprints are recorded as forensic evidence. The legitimate
record is [session_cert_fingerprint]. The injected record is
[injected_cert_fingerprint]."

## APPENDIX C — TRANSPORT MATRIX AND TRAILING-ARTIFACT CHARACTERIZATION
[PENDING CROSS-CHECK] This appendix records scoped July transport observations. It does not establish universal carrier preservation, renderer behaviour, platform intent, or interoperability.
This appendix is built directly from the July 7, 2026 transport logs. The baseline matrix contains 22 runs across 13 editors/platforms; the first local control run is listed separately and excluded from the 13-platform count.

### C.1 Baseline transport matrix
| Target | Environment | Result | Observed artifact / interpretation |
|---|---|---|---|
| Local control run | Local verifier | verified | No editor/platform involved; control run. |
| Apple Notes | macOS | verified | No added space after pasting. |
| Google Docs | macOS Chrome | failed | Appended U+000A on copy-out. |
| Notion | macOS Chrome | verified | No added space after pasting. |
| Slack | macOS | verified | No added space after pasting. |
| iMessage | macOS | verified | No added space after pasting. |
| Telegram | macOS | verified | Send path strips trailing whitespace automatically. |
| WhatsApp | macOS | verified | Send path strips trailing whitespace automatically. |
| ChatGPT | macOS Chrome / Safari | verified | Compose and send paths preserved the manifest. |
| Claude AI | macOS Chrome / Safari | verified | Compose and send paths preserved the manifest. |
| Word Browser | macOS Chrome | failed | Came back with U+0020 on copy-out. |
| LinkedIn | macOS Chrome | verified | Compose and post preserved the manifest. |
| Facebook | macOS Chrome | verified | Compose and post preserved the manifest. |
| IG | macOS Chrome | verified | Compose and post preserved the manifest. |

### C.2 Trailing-artifact characterization
| Target / path | Result | Observed artifact | Note |
|---|---|---|---|
| Telegram input | failed | U+0020 trailing space | Trailing space added on input. |
| Telegram sent | verified | space stripped | Sending removed the trailing space. |
| WhatsApp input | failed | U+0020 trailing space | Trailing space added on input. |
| WhatsApp sent | verified | space stripped | Sending removed the trailing space. |
| Apple Notes | failed | U+0020 trailing space | Trailing space added on input. |
| LinkedIn input | failed | U+0020 trailing space | Trailing space added on input. |
| LinkedIn post | failed | U+0020 + U+000A | Post kept the added whitespace. |
| IG input | failed | U+0020 + U+000A | Trailing artifact after paste. |
| IG after post | verified | none | Post copy preserved the manifest. |
| Facebook input | failed | U+0020 trailing space | Trailing space added on input. |
| Facebook after post | failed | U+0020 trailing space | Space remained after post. |
| ChatGPT input | failed | U+0020 trailing space | Trailing space added on input. |
| ChatGPT after sent | verified | none | Sending removed the trailing space. |
| Claude input | failed | U+0020 trailing space | Trailing space added on input. |
| Claude after sent | verified | none | Sending removed the trailing space. |
The baseline matrix and the trailing-artifact characterization answer different questions. The first measures whether the signed text survives ordinary copy-paste and send/post paths; the second measures what happens when platforms add or preserve whitespace at the boundary.

Note: the two failed results for Google Docs and Word Browser
reflect the raw platform output submitted directly to the
verifier without modification. A manual verification step
confirmed the embedded manifest survived intact in both cases —
the carrier was not stripped by either platform. The trailing
characters added on copy-out (U+000A by Google Docs, U+0020 by
Word Browser) caused the hash mismatch. This manual confirmation
was for survival measurement purposes only. Trailing whitespace
normalization is not implemented in the v0.1 pipeline. The table
records pre-normalization verification results.

## APPENDIX D — PROPOSAL 007 SCOPED BROWSER TESTING-TOOL EVIDENCE

[PENDING CROSS-CHECK] This appendix records scoped browser testing-tool
observations for PROPOSAL 007 only. It does not confirm round trips through
other browsers or renderers; messengers or social-media platforms; API or
LLM channels; Windows or Linux; sanitizers, crawlers, IDEs, mobile platforms,
or external transports. Those channels remain pending cross-check. It also
does not establish a universal browser clipboard, transport, or injection-path
rule. Result labels use the PROPOSAL 007 verifier's E001–E011 vocabulary.
PROPOSAL 007 remains a cooperative, forgeable marker mechanism, not
cryptographic authentication.

### D.1 Selection, copy, and verifier observations

[OBSERVED — SCOPED] For the first retained U+2060 row, ordinary double-click
selection and copy through JavaScript `textContent`, HTML entity, and
JavaScript `insertAdjacentText` injection paths preserved one valid
document-level header and one valid AI pair across Georgia, System UI, and
Menlo. The verifier recorded one surviving pair against a header total of one
and reported `100.0% (1 / 1)`. The pasted value in this browser test contained
two trailing U+000A codepoints. Under the Proposal 007 verifier's defined
trailing-normalization rule, those trailing U+000A codepoints are normalized
and the valid document-level header and AI pair survive. This is not an
application-generated marker content: the Proposal 007 verifier handles them
as trailing transport/clipboard whitespace. This does not alter the separate
v0.1 hard-binding normalization status described in §4.1.

[OBSERVED — SCOPED] Endpoint-sensitive drag selection produced different
pasted codepoint sequences: a complete document-level header and AI pair,
header-only input, orphaned open or close markers, or no valid
document-level header or AI pair. Corresponding results, including E007,
E008, E011, and `NO_VALID_MARKDOWN_FOUND` where applicable, describe the
pasted codepoint sequence only. They do not prove stripping, provenance, AI
origin, or application mutation. Command-modified drag selection was
inconsistent: both full survival and partial or no-valid-signal outcomes were
observed. This is a scoped selection-behaviour observation, not a deterministic
font or injection defect.

| Injection path | Scoped observed selection outcome |
|---|---|
| JavaScript `textContent` | Complete selections survived; modified drag selections could yield orphaned-close/no-valid-signal results. |
| HTML entity | Selection endpoint and drag direction could produce full survival, header-only/orphaned-open results, or no valid signal. |
| JavaScript `insertAdjacentText` | Selection endpoint and drag direction could produce full survival, header-only/orphaned-open results, orphaned-close/no-valid-signal results, or no valid signal. |

[LIMITATION] These observations do not show that one injection path is
generally safer than another; the observed outcome depends on the selected
range.

### D.2 Controlled pathological verifier inputs

[OBSERVED — SCOPED] The testing tool's pathological generator exercised
controlled verifier inputs with supplied `header total 1 / AI spans 1`.
It is not a production embedding path.

| Mode | Observed result |
|---|---|
| Malformed sequence | E001 `TRUNCATED_MARKER` |
| Reordered fields | E002 `INVALID_TYPE` |
| Duplicate header | E006 `DUPLICATE_HEADER`; the first valid document-level header remains authoritative |
| Orphaned open | E007 `ORPHANED_OPEN` |
| Orphaned close | E008 `ORPHANED_CLOSE` |
| Trailing normalization | Valid document-level header and AI pair survive after normalization |
| Internal codepoints | E009 `INTERNAL_SIGNAL`; the valid AI pair remains counted |

### D.3 Re-embedding limitation

[LIMITATION] In the testing tool, re-embedding text that already contains
PROPOSAL 007 signals does not remove the prior signals. Repeated embedding can
therefore create compound input and verifier errors. A production policy for
pre-embedded input remains undefined and requires separate design work. This
does not infer how an LLM, provider, or external system must treat
pre-embedded input.

### D.4 Retained Bidi cross-check findings

[OBSERVED — SCOPED] Arabic, Hebrew, and Persian cards use the existing
Georgia, System UI, and Menlo assignments. The PROPOSAL 007 marker grammar
uses U+2060–U+2064; it does not use LRM, RLM, bidi embeddings, overrides, or
isolates. RTL visual-selection behaviour and copied-range boundaries remain
browser-rendering/selection concerns, not evidence that language content
strips markers. The row labelled U+2060 contains a complete document-level
header and a full AI open/close pair; it is not an isolated-U+2060 survival
test. Bidi transport, renderer, accessibility, and cross-platform questions
remain pending cross-check.

## About the author

LPS was designed and implemented by a self-taught software engineer focused on AI system security, cryptographic integrity, and provenance infrastructure for modern text-based systems.

The motivation behind LPS is to address a growing gap in AI-generated content: the inability to reliably trace, verify, and reason about textual provenance at a granular level without relying on opaque or centralized trust assumptions.

This work is part of an ongoing exploration into secure, adversarially resilient systems for AI-assisted environments, with a focus on practical implementation rather than theoretical design alone.

The author welcomes technical review, adversarial feedback, and collaboration from working groups and security-focused engineering communities.

---


## About the author

LPS was designed and implemented by a self-taught software engineer focused on AI system security, cryptographic integrity, and provenance infrastructure for modern text-based systems.

The motivation behind LPS is to address a growing gap in AI-generated content: the inability to reliably trace, verify, and reason about textual provenance at a granular level without relying on opaque or centralized trust assumptions.

This work is part of an ongoing exploration into secure, adversarially resilient systems for AI-assisted environments, with a focus on practical implementation rather than theoretical design alone.

The author welcomes technical review, adversarial feedback, and collaboration from working groups and security-focused engineering communities.

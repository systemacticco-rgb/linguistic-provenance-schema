## Public README

# Linguistic Provenance Schema (LPS)
### An AI C2PA Contribution Standard

**Status:** [IMPLEMENTED / PROPOSED] v0.1 reference implementation built and locally tested; core signing, embedding, verification, and registry stub are implemented; PROPOSAL 005 is specified rather than implemented. This document distinguishes implemented behavior, specified architecture, and future work.
**Author:** Brayan Daniel Rodriguez Lugo  
**Date:** July 2026 

---

## Abstract

Current content provenance systems typically record whether AI was involved, but not the degree, nature, or proportion of that contribution. This proposal defines the Linguistic Provenance Schema (LPS), a manifest schema designed to travel through existing C2PA-style infrastructure and complementary watermarking channels while recording section-level provenance for human-originated text, AI-generated text, and AI-modified human text.

LPS does not replace C2PA or SynthID. It defines a contribution-tracking layer that can be embedded and verified alongside existing provenance signals.

**[PROPOSED] Portfolio position.** LPS v0.1 is the dominant long-term
signed-manifest system. PROPOSAL 005/A.8R is complementary LPS
redundancy research, not an independent system or C2PA Text A.9.
PROPOSALS 006 and 007 are optional mechanisms that do not replace LPS
v0.1. PROPOSAL 007 is a parallel, provisional cooperative AI-only
marker proposal: it is removable, forgeable, non-forensic, and human
span markers are removed from its main grammar; its variants remain
documentation-only fallbacks.

### Proposal 007 scoped browser testing-tool evidence

[OBSERVED — SCOPED] The observations in this section apply only to
PROPOSAL 007's cooperative U+2060–U+2064 marker grammar. They are
separate from this README's signed A.8 manifest transport evidence and
do not change the v0.1 signed-manifest verification states. PROPOSAL 007
remains cooperative and forgeable, not cryptographic authentication.

[PENDING CROSS-CHECK] The browser testing-tool results do not confirm
round trips through other browsers or renderers; messengers or social-media
platforms; API or LLM channels; Windows or Linux; sanitizers, crawlers,
IDEs, mobile platforms, or external transports. Those channels remain
pending cross-check. The observations do not establish a universal browser
clipboard, transport, or injection-path rule.

For the first retained U+2060 row, ordinary double-click selection and copy
through JavaScript `textContent`, HTML entity, and JavaScript
`insertAdjacentText` injection paths preserved one valid document-level
header and one valid AI pair across Georgia, System UI, and Menlo. The
verifier recorded one surviving pair against a header total of one and
reported `100.0% (1 / 1)`. In the observed browser test, the pasted value
contained two trailing U+000A codepoints. The Proposal 007 verifier's
defined trailing-normalization rule handles them as trailing
transport/clipboard whitespace, not application-generated marker content;
the valid document-level header and AI pair survive. This does not alter
the separate v0.1 text-hash normalization status.

Endpoint-sensitive drag selection produced different pasted codepoint
sequences: a complete document-level header and AI pair, header-only input,
orphaned open or close markers, or no valid document-level header or AI pair.
Results including E007, E008, E011, and `NO_VALID_MARKDOWN_FOUND` where
applicable describe the pasted codepoint sequence only; they do not prove
stripping, provenance, AI origin, or application mutation. Command-modified
drag selection was inconsistent: both full survival and partial or
no-valid-signal outcomes were observed. This is a scoped selection-behaviour
observation, not a deterministic font or injection defect.

| Injection path | Scoped observed selection outcome |
|---|---|
| JavaScript `textContent` | Complete selections survived; modified drag selections could yield orphaned-close/no-valid-signal results. |
| HTML entity | Selection endpoint and drag direction could produce full survival, header-only/orphaned-open results, or no valid signal. |
| JavaScript `insertAdjacentText` | Selection endpoint and drag direction could produce full survival, header-only/orphaned-open results, orphaned-close/no-valid-signal results, or no valid signal. |

[LIMITATION] These results do not show that one injection path is generally
safer than another; the observed outcome depends on the selected range. The
Proposal 007 verifier uses the E001–E011 result vocabulary. Its pathological
generator exercised controlled verifier inputs with supplied `header total 1
/ AI spans 1`; it is not a production embedding path.

| Mode | Observed result |
|---|---|
| Malformed sequence | E001 `TRUNCATED_MARKER` |
| Reordered fields | E002 `INVALID_TYPE` |
| Duplicate header | E006 `DUPLICATE_HEADER`; the first valid document-level header remains authoritative |
| Orphaned open | E007 `ORPHANED_OPEN` |
| Orphaned close | E008 `ORPHANED_CLOSE` |
| Trailing normalization | Valid document-level header and AI pair survive after normalization |
| Internal codepoints | E009 `INTERNAL_SIGNAL`; the valid AI pair remains counted |

[LIMITATION] In the testing tool, re-embedding text that already contains
PROPOSAL 007 signals does not remove the prior signals. Repeated embedding
can create compound input and verifier errors. A production policy for
pre-embedded input remains undefined and requires separate design work; this
does not infer how an LLM, provider, or external system must treat
pre-embedded input.

[OBSERVED — SCOPED] Arabic, Hebrew, and Persian cards use the existing
Georgia, System UI, and Menlo assignments. The PROPOSAL 007 marker grammar
does not use LRM, RLM, bidi embeddings, overrides, or isolates. RTL
visual-selection behaviour and copied-range boundaries remain
browser-rendering/selection concerns, not evidence that language content
strips markers. The row labelled U+2060 contains a complete document-level
header and full AI open/close pair; it is not an isolated-U+2060 survival
test. Bidi transport, renderer, accessibility, and cross-platform questions
remain pending cross-check.

---

## 1. The Problem

### 1.1 The HaLLMark Gap

The HaLLMark Effect (Hoque et al., CHI 2024) demonstrated that AI contribution to writing can be tracked during the writing process — distinguishing AI-written text from AI-influenced text in real time. The authors explicitly acknowledged the limitation of their system:

> *"Provenance is legible during writing but not persistent in the final document: once accepted, AI-originated text is visually indistinguishable from human-written text. Persistent provenance mechanisms are therefore a design implication for future tools rather than a feature of the HaLLMark system itself."*

This proposal addresses that future work directly.

### 1.2 The Binary Problem

Current provenance systems record whether AI was involved. They do not record:

- What portion of the content was AI-generated versus human-originated
- Which specific sections were generated, edited, or influenced by AI
- What AI tool contributed and at which stage of creation
- The degree of human modification applied after AI generation

A document that is 5% AI-edited and a document that is 95% AI-generated are treated identically by current systems. That distinction matters in legal proceedings, journalistic verification, academic integrity, and institutional trust contexts.

### 1.3 The Export Problem

Existing workflow tools that track AI contribution — including the HaLLMark system — record provenance during the writing session. When the document is exported, shared, or copy-pasted, that record disappears. The final artifact carries no persistent signal of how it was created.

This is insufficient for any context where the document's authenticity matters after it leaves the creator's environment.

---

## 2. The Existing Landscape

### 2.1 C2PA — What It Covers and Where It Breaks

The Coalition for Content Provenance and Authenticity (C2PA) standard embeds cryptographically signed metadata manifests into digital content recording creator identity, creation time, tools used, and edit history. C2PA is an open standard with broad adoption across Adobe, Microsoft, Google, and others.

C2PA breaks when:

- File metadata is stripped during platform processing
- Content is screenshotted or re-encoded
- The file container is discarded

C2PA records AI involvement but not degree or nature of contribution.

A comprehensive independent security analysis (Golaszewski et al., UMBC/NSA, April 2026) confirmed these failure modes in current C2PA implementations, finding that timestamps can be replaced without detection, revoked certificates are accepted as valid by conforming validators, and the same content can receive contradictory verdicts from different compliant tools. The authors conclude that C2PA should not yet be relied upon for high-stakes uses such as journalism or legal evidence. LPS does not resolve these implementation vulnerabilities — it operates as an additional layer that produces value precisely because no single layer is sufficient.

### 2.2 SynthID — What It Covers and Where It Breaks

Google DeepMind's SynthID embeds invisible watermarks at the pixel, audio sample, or token level. These signals survive screenshots, compression, and re-encoding that destroys C2PA metadata.

SynthID breaks when:

- Content is processed through analog conversion
- Heavy editing redistributes the statistical signal
- The content was not generated by a SynthID-enabled tool — SynthID only covers Google's own generators

SynthID can confirm content came from a SynthID-enabled generator. It cannot tell you who created it, when, what was changed, or the degree of AI contribution. Detection requires Google's proprietary detector — it is not an open standard.

### 2.3 C2PA Text Infrastructure — What Exists

The `encypherai/c2pa-text` library (MIT licensed) implements all three C2PA 2.4 text embedding methods:

- **Unstructured (A.8):** Invisible Unicode Variation Selectors appended to text — [PENDING CROSS-CHECK] preservation is transport-dependent
- **Structured (A.9):** ASCII armor blocks inside comments or front matter
- **HTML (A.7):** Script or link elements in document head

This library handles the delivery mechanism — embedding and extracting C2PA manifests from text. It does not define what the manifest should contain for contribution tracking purposes.

### 2.4 Linguistic Steganography — What Exists

SA-ANS (Self-Adaptive Asymmetric Numeral Systems) and related approaches can hide data payloads within text by modifying token probability distributions during generation — embedding information in word choices while preserving natural fluency.

### 2.5 The Gap None of Them Fill

Each system addresses one layer. None address the combination:

| Requirement | Status |
|---|---|
| Persistent in the final document after export | ✗ C2PA breaks on strip, HaLLMark doesn't persist |
| Records degree and nature of contribution | ✗ SynthID is binary, C2PA records involvement not proportion |
| Works across generators, not just proprietary tools | ✗ SynthID is closed |
| Survives copy-paste for text | [PENDING CROSS-CHECK] Unicode selectors have scoped transport evidence; they deliver only what the manifest contains |

The delivery mechanism exists. The standardized manifest schema for contribution tracking does not.

---

## 3. The Proposal

### 3.1 What LPS Is

The Linguistic Provenance Schema (LPS) is a standardized manifest schema for recording degree and nature of AI contribution in digital content. It is designed to be:

- Embedded using existing `c2pa-text` infrastructure (Unicode variation selectors; preservation depends on the transport path)
- Layered with C2PA metadata manifests and SynthID watermarks as complementary signals
- Readable by any C2PA-compliant verification tool that implements the LPS extension
- Applicable to text, audio, and document formats

For v0.1, LPS does not require new embedding infrastructure. It defines what goes inside the manifest that existing infrastructure delivers. [PROPOSED] PROPOSAL 005/A.8R is a complementary redundant-carriage direction for LPS v0.1, not C2PA Text A.9; it remains future work until specified and accepted.

### 3.2 What LPS Records

A minimal LPS manifest contains:

```json
{
  "lps_version": "0.1",
  "text_hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "text_length": 1204,
  "content_segments": [
    {
      "segment_id": "s001",
      "start_offset": 0,
      "end_offset": 245,
      "origin": "human",
      "confidence": 95,
      "confidence_source": "tool"
    },
    {
      "segment_id": "s002",
      "start_offset": 246,
      "end_offset": 891,
      "origin": "ai_generated",
      "ai_tool": "claude-sonnet-4",
      "confidence": 98,
      "confidence_source": "tool"
    },
    {
      "segment_id": "s003",
      "start_offset": 892,
      "end_offset": 1204,
      "origin": "ai_modified_human",
      "ai_tool": "claude-sonnet-4",
      "modification_degree": 0.4,
      "confidence": 87,
      "confidence_source": "tool"
    }
  ],
  "overall_ai_proportion": 0.62,
  "human_proportion": 0.38,
  "signing_tool": "lps-reference-implementation-v0.1",
  "signed_at": "2026-06-07T00:00:00Z"
}
```
*Note: this example illustrates the manifest structure only. It
does not include the `signature`, `cert_url`, or `cert_fingerprint`
fields added by the signing layer — see Section §3.2.1 and the working-group submission appendix for the full signed-manifest shape and current signature encoding status.*


Confidence values are probabilistic estimates, not legal determinations. They are implementation outputs used to support later interpretation, not standalone verdicts. Any evidentiary use depends on jurisdiction, expert review, and the broader context of the record.

Every segment carries a `confidence_source` field recording how its confidence value was produced. Three values are defined: `tool` (supplied directly by the generating AI tool), `derived` (supplied by an approved AI detection classifier or human reviewer), and `fallback` (calculated by mathematical derivation from document-level character distribution — see SPEC.md §1.2). Note: the v0.1 reference implementation currently only ever produces `tool` or `fallback` — there is no classifier or human-reviewer input path implemented yet, so `derived` is schema-defined but not currently emitted.

### Compression rules — v0.1

All manifests are compressed before embedding using the shortcode
dictionary defined in SPEC.md section 4.1. The verification tool
must expand shortcodes using the same dictionary before reading
any field.

#### Default field assumption
Fields `lv` (lps_version) and `st` (signing_tool) are omitted
at embed time in v0.1. The verification tool assumes lps-v0.1
defaults if these fields are absent. If non-default values are
present they override the assumption.

#### Confidence encoding
Confidence values are stored as integers 0-100, not floats 0.0-1.0.
Example: confidence of 0.95 is stored as 95. The verification
tool returns confidence as integers in v0.1 — division by 100
for display is specified but not yet implemented in
verificationTool.mjs. Consumers of the verification output
should expect integers in the range 0-100.

#### Algorithm field value convention
The `algorithm` field carries the string value `es256`, 
derived from the JOSE algorithm identifier ES256, represented 
internally as the lowercase string es256. This identifies the 
underlying signaturealgorithm used by the LPS implementation 
— ECDSA P-256, SHA-256, IEEE P1363 raw r‖s
signature encoding — and is an LPS-internal naming convention,
not a COSE integer algorithm identifier (COSE registers ES256 as
integer -7). The manifest is not currently packaged as a COSE or
JOSE structure (no COSE_Sign1 or compact JWS envelope). 
If LPS adopts a standard COSE_Sign1 envelope in a future version,
the algorithm field may be replaced by the standard COSE algorithm
identifier rather than the current internal convention

#### Immutability rule
The shortcode dictionary is versioned and immutable.
Existing codes never change after publication.
New codes may only be added in future versions.
A manifest produced under v0.1 must remain readable by any
future verification tool that supports v0.1.

### text_hash field
SHA-256 fingerprint of the visible text computed before embedding.
The verification tool hashes the clean extracted text and compares
it against this value. If they do not match, the visible text was
modified after signing and verification returns failed.
Format: 64-character lowercase hex string.
Computed by: manifestGenerator.mjs at manifest creation time.
Checked by: verificationTool.mjs at verification time.

### text_length field
Character count of the visible text at signing time, computed
after the trailing strip rule /[\r\n ]+$/ is applied. The value
reflects the stripped text length, not the raw input length.
The same strip is applied identically at verification time before
the received text length is compared against this field. Always present — no default omission, unlike
lv/st. Used only in the failed state (text_hash mismatch) to decide
whether original_manifest is disclosed: disclosure is withheld when
the received text's length differs from text_length by more than
10%. This limits what an adversary can learn from a deliberate
large-mismatch replay attempt (see working-group-submission.md §5,
"Transfer/replay"). This field is a plain manifest field with no
separate cryptographic protection of its own — it is protected the
same way every other manifest field is, by the signature over the
whole manifest object (see Section §3.2.1 for signature scope). It carries
no special forgery risk beyond what already applies to text_hash or
any other signed field.
Computed by: manifestGenerator.mjs at manifest creation time.
Checked by: verificationTool.mjs at verification time, failed-state
disclosure decision only.

### 3.2.1 Full signed manifest structure

The signing layer wraps the inner manifest in an envelope that adds
the cryptographic signature and certificate reference. The canonical
signed manifest shape — as produced by `signingLayer.mjs` and
reconstructed by `compression.mjs` `decompress()` — is:

```json
{
  "manifest": {
    "lps_version": "lps-v0.1",
    "text_hash": "<sha256-64-char-hex>",
    "text_length": 50,
    "content_segments": [
      {
        "segment_id": 1,
        "start_offset": 0,
        "end_offset": 21,
        "origin": "human",
        "confidence": 95,
        "confidence_source": "tool",
        "ai_tool": null,
        "modification_degree": null
      }
    ],
    "overall_ai_proportion": 0.56,
    "human_proportion": 0.44,
    "signing_tool": "lps-reference-implementation-v0.1",
    "signed_at": "<ISO-8601>"
  },
  "signature": "<base64-encoded-r‖s-bytes>",
  "cert_url": "https://raw.githubusercontent.com/systemacticco-rgb/lps-certificates/main/cert.pem",
  "cert_fingerprint": "<sha256-64-char-hex-of-DER-bytes>",
  "algorithm": "es256",
  "signed_at": "<ISO-8601>"
}
```

**Two `signed_at` fields — they are not duplicates.**

`manifest.signed_at` — the provenance claim timestamp. Produced by
`manifestGenerator.mjs` at the moment the content segments, hashes,
and proportions are recorded. This field is inside the signed
payload and is cryptographically protected by the signature. Any
alteration to it invalidates verification.

`signed_at` (outer) — the signing envelope timestamp. Produced by
`signingLayer.mjs` at the moment the manifest is signed and wrapped.
This field sits outside the signed payload. It is not written to
the registry — the registry record carries its own `created_at`
timestamp generated by the database at insert time. The outer
`signed_at` travels in the signed manifest envelope only. It is
not cryptographically protected in isolation — its integrity
depends on the envelope, not on the signature over the inner
manifest.

In practice both timestamps are nearly identical because manifest
generation and signing run sequentially in the same pipeline call.
They are architecturally distinct: the inner timestamp belongs to
the provenance record, the outer timestamp belongs to the signing
event and the registry entry.

**`cert_fingerprint`** is a SHA-256 hash of the certificate's
DER-encoded bytes (`X509Certificate.raw`), not of the PEM string.
Both the signing layer and the verification tool compute fingerprints
from DER bytes. Using PEM text would make the comparison sensitive
to line-ending and encoding differences across platforms.

**`signature`** is the IEEE P1363 raw r‖s encoding of the ECDSA
P-256 signature over the canonical CBOR bytes of the inner
`manifest` object, encoded as standard base64. It is not
DER-encoded and not base64url. See §3.5 and SPEC.md §3 for
the signing constraints.

**Note:** `original_manifest` returned by the verifier in a `failed`
state is not this structure. It is a disclosure-safe subset
containing only `signed_at`, `overall_ai_proportion`,
`human_proportion`, and `segments`. It intentionally excludes
`text_hash`, `text_length`, `signature`, `cert_url`,
`cert_fingerprint`, and `algorithm`. See Appendix A, State 3.

### 3.3 The Token Extension

In addition to local embedding, LPS supports a token-based architecture where the embedded manifest contains a pointer to a server-side record:

```json
{
  "lps_version": "lps-v0.1",
  "token": "lps_a7f3c9...",
  "server": "https://verify.lps-standard.org",
  "local_summary": {
    "overall_ai_proportion": 0.62,
    "segment_count": 3
  }
}
```
Note: the server domain shown above is a placeholder. The hosting architecture remains an open design question for the working group phase, including whether the registry is foundation-hosted, federated, or distributed across multiple compatible nodes.

The registry record contains the content hash, generating identity,
and timestamp. It does not store the full manifest. The full signed
manifest — including the complete segment breakdown, origin types,
confidence values, and proportions — travels in the local embedding.
The token points to the registry record, not to a stored manifest copy.

[LIMITATION] Registry recovery can corroborate only an exact
visible-text hash with an existing generation-time record. It does not
establish why a carrier is absent, restore span-level evidence, or
replace the embedded signed manifest. A matching record indicates that
identical text had been registered; it is not a substitute for the
manifest's contribution fields.

Server-side records also enable:

- Logging of every verification attempt — chain of custody for the verification process itself
- Detection of unusual query frequency — signal of laundering attempts
- Manifest updates as new information is discovered post-publication

### 3.4 The Layered Signal Architecture

LPS is most valuable as one layer in a multi-signal system:

| Layer | Signal | Survives Stripping | Carries Context | Open Standard |
|---|---|---|---|---|
| C2PA | Metadata manifest | No | Yes — full | Yes |
| SynthID | Pixel/token watermark | Yes | No — binary | No |
| LPS | Contribution manifest | [LIMITATION] Transport-dependent | Yes — granular | Proposed |
| Behavioral | Statistical patterns | Yes | Partial | Varies |

No single layer is sufficient. The value emerges from triangulation. Partial signals from multiple layers produce a confidence picture more useful and more honest than any binary verdict.

### 3.5 Compression and Delivery Architecture

Manifests are compressed before embedding using two layers:

**Layer 1 — Shortcode dictionary**
All field names and origin values are replaced with short codes
defined in SPEC.md section 4.1. `content_segments` becomes `cs`.
`ai_generated` becomes `aig`. The verification tool expands codes
on extraction. Dictionary is versioned and immutable.

**Layer 2 — CBOR binary encoding**
The shortcoded manifest is encoded as CBOR binary before embedding.
CBOR eliminates JSON structural overhead — no quotes, colons, or
brackets. Numbers stored as binary. Estimated saving: 50-70% over
raw JSON after shortcodes applied.

**Certificate delivery**
The signing certificate is not embedded in the manifest. It is
hosted at a stable public URL. The manifest carries `cert_url`
pointing to the certificate and `cert_fingerprint` — a SHA-256
hash of the certificate — so the verification tool can confirm
the fetched certificate is authentic before using it.

**Capacity and carrier behavior**
The v0.1 reference implementation uses Method A.8 for the root
plain-text copy/paste path. A.8 encodes the compressed manifest as
invisible Unicode variation selectors appended to the visible text.
Larger manifests produce longer invisible selector sequences; they
are not automatically converted into visible structured text blocks.

[SECURITY / LIMITATION] A.8 and the proposed A.8R carrier carry the
Unicode-conformance concern identified for the C2PA unstructured
variation-selector scheme. This is a carrier and adoption limitation,
not a claim that LPS cryptographic signatures are invalid.

Earlier drafts described a 220-byte fallback threshold from A.8 to
A.9. That is no longer the implemented behavior. The practical
constraint is empirical editor survival: longer invisible payloads
may be more likely to be normalized, stripped, or damaged by some
editors or transports and must be measured per copy/paste path.

#### Production constraints and safe operating ranges

[PENDING CROSS-CHECK] Safe manifest size: production manifests with realistic segment counts
(3–10 segments) land between 400 bytes and 1,500 bytes compressed.
Invisible character counts at this size remain below approximately
3,000 variation selectors. The July 2026 study observed clean
copy-paste behaviour at this size on its scoped test paths.

Editor latency threshold: invisible character counts above approximately
6,000 variation selectors — corresponding to manifests above
approximately 2,500–3,000 bytes — produce measurable copy-paste latency
in rich-text editors that process character-level clipboard payloads.
Apple Notes on macOS exhibits this behavior at 5kb manifest size and
above. Latency is not carrier corruption — verification succeeds at all
tested sizes.

AI compose input reclassification: platforms including Claude and the
OpenAI ecosystem may reclassify large invisible-character payloads as
file uploads rather than plain text when pasted into compose inputs. The
manifest survives reclassification but the workflow breaks. The
reclassification threshold varies by platform and is not under LPS
control. Measurement across all target platforms is an open item
(OPEN-4).

Token overhead: see the token overhead section above.

[PENDING CROSS-CHECK / LIMITATION] Code block constraint: LPS manifests
must not be embedded inside code syntax blocks. Code renderers may
display invisible Unicode characters as visible replacement icons or
colored markers. GitHub file-level preservation is scoped evidence,
not a universal transport claim. The constraint applies to inline and
fenced code blocks only.

**Embedding methods**
Method A.8 (Unstructured) — invisible Unicode variation selectors
appended to the text. This is the current v0.1 root plain-text
copy/paste carrier.

Method A.9 (Structured) — visible structured text, such as
ASCII-armour, comments, or front matter. A.9 is not part of the
v0.1 verification path and is not implemented in the current
verifier. A.8 is the only extraction path. A.9 is defined in C2PA
Text as a structured visible-text method; it is not an invisible
fallback for A.8 and remains outside v0.1 scope.

[PROPOSED] Future larger invisible manifests may use PROPOSAL 005's
A.8R carrier: an A.8-derived redundant invisible variation-selector
chunk direction. It is complementary LPS research, remains future
work, and is not C2PA Text A.9.

The verification tool reports which method was recovered when that
information is available.

### 3.6 The Anti-Forensic Boundary

[LIMITATION] If an embedded provenance signal is absent or corrupted,
LPS reports the degraded state and any independently available
evidence. Carrier absence establishes neither stripping, intent, nor
human authorship.

[SECURITY] Invisible Unicode alone establishes neither AI origin nor
malicious intent. LPS v0.1 can verify the integrity of a signed claim;
it does not convert a missing carrier into a forensic attribution.

### 3.7 Server-Side Notarization Registry

The embedded LPS manifest has scoped copy-paste evidence, not a
universal survival guarantee. Screenshots, OCR, retyping, and analog
conversion may remove or destroy the embedded signal. When that
happens, the verification tool can consult registry evidence only if
an exact visible-text hash matches an existing generation-time record.

The registry is an auxiliary evidence layer, not a substitute for the embedded manifest or cryptographic verification.

**How it works**

At the moment AI content is generated, the generating
tool computes a SHA-256 fingerprint of the content and
writes it to an append-only server-side log. The log
records two things only: the fingerprint and the timestamp.
The content itself is never stored. The registry knows
that this exact content existed at this exact moment.
Nothing more.

The registry issues a unique token for each record —
a cryptographically random identifier that gets embedded
in the document alongside the LPS manifest. The token is
the key to the registry record.

**How verification works**

If the embedded signal survived: the verification tool
extracts the token, queries the registry, and receives
the stored fingerprint and timestamp. It hashes the
received document and compares. If they match, the
document is confirmed as existing at that timestamp
unchanged.

If the embedded signal is absent: the verification tool hashes the
received document and queries the registry by content hash. If an
existing generation-time record matches the exact visible-text hash,
it can corroborate that registered text existed at that recorded time.
It does not independently confirm origin type, contribution
proportion, issuer, or the reason the carrier is absent. The embedded
manifest is the source of those fields. Registry recovery returns the
registry_required state, not verified.

**What it cannot confirm**

The registry can corroborate only that an exact visible-text hash has
an existing generation-time record. It does not by itself confirm
authorship, origin type, contribution proportion, or carrier removal.
The embedded LPS manifest is the source of the contribution fields.

**Why it requires AI provider cooperation**

The record must be written at generation time by the
generating system. A third party can hash a document and
submit it to a registry — but that only proves the third
party had the document at that moment, not when the AI
generated it. The forensic value depends on the record
existing before the content leaves the generating system.

[LAW / COMPLIANCE] Article 50 is an external adoption and compliance
context, not proof that an LPS carrier is compliant. The Code of
Practice is voluntary. Provider adoption remains an external
dependency; the reference implementation demonstrates a reference
architecture and the working-group submission proposes a standard.

**Architecture status**

Stub implemented — June 21 2026.
Two Supabase tables: registry_records and usage_events.
registerContent() writes content hash, generating identity,
and timestamp at generation time. queryRegistry() retrieves
by token or content hash fallback.
verificationTool.mjs registry_required state triggers
queryRegistry() when no embedded signal is found.
Full production architecture: see the working-group submission §8.1.
Open questions: registry hosting, record retention,
token revocation, cross-registry legal access.

### 3.8 Token Binding and Bypass Countermeasures

[LIMITATION] The embedded LPS manifest has scoped copy-paste evidence;
it does not survive every transformation. Screenshot, OCR, and retyping
can leave clean text with no invisible carrier. No cryptographic
mechanism prevents that at the content level.

This issue is handled as a separate future device-capture discussion and is not part of the registry architecture described here.

**Layer 1 — Screenshot blocking on native apps**

[HOLD] This is a future device-capture direction, not a v0.1 LPS
capability or an established carrier-preservation claim.

Native mobile implementations of LPS-compliant AI tools
render generated content in a hardware-backed protected
display surface. On iOS and Android, this surface cannot
be captured by the system screenshot mechanism. Screenshot
returns black. The only way to extract the generated text
is through the app's own export mechanism — which re-embeds
the LPS signal on every copy or export operation.

This applies to native apps only. Web browsers do not have
access to protected display surfaces. The casual screenshot
bypass is closed. A second physical device photographing
the screen remains an irreducible residual risk.

**Layer 2 — Server-side token binding**

[PROPOSED] Server-side binding is a registry direction, not a
replacement for the signed manifest or a way to infer why a carrier is
absent.

At generation time, the AI tool registers the content with
the LPS registry. The registry binds three things to a
unique cryptographically random token: the content hash,
the generating identity — which AI tool, which account —
and the creation timestamp.

The token is embedded in the document alongside the LPS
manifest. When the signal is absent after screenshot, OCR,
or manual removal, the token may also be absent from the document.
The server-side binding is not. The registry record exists
permanently and independently of what happens to the document.

When the content is later hashed by any party, a registry match can
corroborate only an exact visible-text hash with an existing
generation-time record. It does not establish stripping, intent,
human authorship, or recover span-level evidence.

This is the direct equivalent of the Chilean transit QR
system: the QR was screenshotted, the visual was bypassed,
but the server-side record still attributed the usage to
the correct account because the binding was server-side,
not in the visual representation.

**Layer 3 — Usage tracking**

Every verification query against the registry is logged
as a usage event alongside the generation record. The
complete forensic timeline becomes: when the content was
generated, by whom, and every time it was submitted for
verification — and when it was not.

The absence of voluntary verification events between
generation and a mandatory checkpoint — a court submission,
a regulatory filing — is itself a forensic signal. The
content was held and used without verification for a
documented period. That gap contributes to the forensic
picture without being conclusive alone.

**The law as an external adoption context**

[LAW / COMPLIANCE] Legal disclosure and regulator interpretation are
external to LPS. Article 50 does not make an LPS carrier compliant or
turn a registry record into proof of disclosure compliance.

**Architecture status**

Stub implemented — June 21 2026.
Usage event logging functional — every queryRegistry() call
writes a usage_events record with token, queried_by,
query_type, and timestamp. Token path and content hash
fallback path both logged independently.
Full production architecture including identity binding
and credentialed access: see the working-group submission §5.
---

## 4. Limitations

### 4.1 Requires AI Developer Cooperation

[LIMITATION] LPS manifests require generation-time cooperation from the
producing tool. Provider cooperation, transport preservation, renderer
behaviour, crawler interpretation, and regulator scope are external
dependencies.

### 4.2 Laundering Vulnerability

| Attack Vector | LPS Response |
|---|---|
| Copy-paste | [PENDING CROSS-CHECK] Preservation is transport-dependent |
| Re-encoding through different formats | Manifest may be stripped — LPS degrades |
| Analog conversion (photograph of screen, re-recording of audio) | Manifest destroyed — LPS fails |
| Heavy editing that preserves the carrier | Carrier intact, visible text hash mismatches — returns failed. original_manifest disclosed or withheld depending on the 10% length threshold. |
| Heavy editing that removes the carrier | Carrier absent or corrupted — returns degraded. Carrier absence does not establish stripping, intent, or human authorship. |

[LIMITATION] A degraded result reports the available evidence; it does
not attribute a cause to carrier absence.

### 4.3 Encoding Capacity Constraints

[PENDING CROSS-CHECK / LIMITATION] Unicode variation selectors have
limited capacity for embedded data. The v0.1 implementation uses A.8
exclusively. Transport observations from the survival matrix are scoped
evidence, not universal editor or platform claims. The A.8 carrier also
has the named Unicode-conformance concern; this is a carrier/adoption
limitation, not a cryptographic-signature invalidity claim.

### 4.4 Statistical Detectability

[SECURITY / LIMITATION] Sophisticated adversaries may be able to detect
and strip LPS signals. Layering can preserve independent evidence, but
carrier absence alone does not establish stripping or intent.

### 4.5 Trust List Not Yet Implemented

The trust list described as reused C2PA infrastructure is architecturally specified but not implemented in the v0.1 reference implementation. The registry stub currently uses a single Supabase service-role credential as its write-access boundary — a placeholder, not a trust list. Signature verification in v0.1 checks the certificate fingerprint against the manifest, not against a trust list of authorized signers.

### 4.6 Registry Access Model Undefined

The registry's intended access model (public read vs. credentialed-only) is not yet settled between this repository's design documents —see the working-group submission §8.1 for the open tiered-access question. Treat any statement about registry access as provisional until the working group resolves registry hosting and access architecture.

### 4.7 Certificate Revocation Checking Not Implemented

Certificate revocation checking is part of the intended production verification architecture but is not implemented in the current reference implementation. verificationTool.mjs fetches the certificate, confirms its fingerprint matches the manifest, and verifies the signature — it does not check whether the certificate has been revoked. This is a known gap relative to the C2PA weaknesses (Golaszewski et al., §2.1) that motivate LPS's design.

### 4.8 Canonicalization Version Pinning Not Yet Resolved

Signature validity depends on the manifest's canonical byte representation staying reproducible over time. In the current reference implementation this is guaranteed only by pinning every deployment, via lockfile, to one exact version of the CBOR encoding library used to produce those bytes — not by a version identifier recorded in the manifest itself. This is sufficient for a single reference implementation verifying its own output, but is not yet resolved for independent implementations verifying manifests across encoder upgrades, or for PROPOSAL 005's cross-copy reconstruction path, which must re-derive canonical bytes from recovered fragments. This is an open question, not a defect in current behavior — see the working-group submission §8.11 for the reasoning and the approaches considered.

### 4.9 Code Block Carrier Not Defined

[PENDING CROSS-CHECK / LIMITATION] LPS manifests cannot be embedded
inside inline or fenced code syntax blocks in v0.1. Code renderers may
display variation selectors as visible replacement icons or coloured
markers. GitHub file-level preservation is scoped evidence, not a
universal claim. No carrier mechanism for code-block provenance is
defined in v0.1.


---

## 5. Legal and Institutional Use Cases

### 5.1 Judicial Evidence

Courts are already encountering AI-generated evidence without standardized tools for evaluating provenance claims. LPS enables a structured forensic report that documents:

- What provenance signals were present at time of verification
- What signals were absent, without inferring the cause of absence
- The pattern of degradation across the file's history
- Confidence levels for each finding

The report is designed for two audiences simultaneously: plain language for judicial readers, technical detail for expert witness examination and cross-examination.

LPS output is forensic evidence, not legal proof. Admissibility depends on jurisdiction, expert qualification, and judicial precedent — the same conditions that govern digital forensics reports today.

### 5.2 Journalistic Verification

News organizations verifying content from external sources need to know not just whether content is authentic but whether it has been AI-modified and to what degree. LPS provides section-level granularity that file-level C2PA cannot.

### 5.3 Compliance

[LAW / COMPLIANCE] Article 50 is an external adoption and compliance
context. It does not prove that any LPS carrier is compliant. The Code
of Practice is voluntary, and provider marking and deployer disclosure
remain distinct obligations whose scope depends on the applicable legal
and regulatory interpretation. LPS may be evaluated as a
contribution-tracking layer; it is not a compliance certification.

---

## 6. Proposed Path Forward

### 6.1 Open Specification Working Group

LPS should be developed as an open specification through a working group that includes:

- AI tool developers — to implement manifest generation
- C2PA working group members — to ensure compatibility with existing infrastructure
- Legal and forensic professionals — to validate the judicial use case requirements
- Journalists and media organizations — to validate the verification workflow requirements

### 6.2 Reference Implementation

[IMPLEMENTED] A reference implementation of LPS manifest generation and
verification has been developed in JavaScript (Node.js ES modules),
using Node.js built-in `crypto` for signing/verification and
`c2pa-text` for text embedding. Dependency code in `node_modules` is
outside the repository implementation scope. The core pipeline is built
and locally tested.

[HOLD — LANGUAGE MIGRATION] JavaScript remains a reference
implementation environment. Runtime latency, typed integration
contracts, binary-hash handling, CPU concurrency, and cross-language
transport remain production dependencies, not submission blockers.
Signatures use the ES256 primitive (ECDSA P-256, SHA-256, IEEE P1363 raw r‖s encoding), cross-checked against the independent panva/jose library, confirming interoperability of the ES256 cryptographic primitive. This does not imply JOSE/JWS or COSE envelope interoperability. — not a claim that the manifest itself is JOSE/COSE-compatible as a structure. The manifest is not currently wrapped in a COSE_Sign1 or compact JWS envelope; full envelope-level interoperability is a future-version target, not current state.

[PENDING CROSS-CHECK] Text hash and text length are computed after
stripping trailing carriage returns (U+000D), newlines (U+000A), and
spaces (U+0020) from the visible text. The same strip is applied at
signing and verification time. This rule was derived from a scoped
editor survival matrix, not a universal transport claim. Two observed
editors append trailing characters automatically on copy-out: Google
Docs appends a newline and Word Browser appends a space. Without
normalization both observed paths produce a hash mismatch despite the
manifest surviving intact.

[PROPOSED] A redundant embedding architecture (PROPOSAL 005/A.8R) is
complementary LPS redundancy research, not an independent system or
C2PA Text A.9. Its anchors, overlapping copies, and reconstruction
remain specified rather than implemented, and retain the A.8
Unicode-conformance concern.

Text carrying LPS-embedded manifests incurs increased token
consumption when passed to any language model API. Unicode variation
selectors used by the A.8 carrier are not collapsed by tokenizers —
each invisible character consumes token budget independently.
[PENDING CROSS-CHECK] Candidate production manifests in the stated
3–10-segment, 400–1,500-byte compressed range produce invisible
character counts below approximately 3,000 variation selectors. Token
overhead and transport behaviour remain implementation- and
platform-dependent; larger profiles require separate measurement.

Two adoption scenarios both carry this constraint. If LPS is adopted
at generation time — the AI provider embeds a manifest at the moment
content is produced — the embedded text returned to the caller
already carries the invisible payload, and any downstream API call
passing that text to another model will consume additional tokens. If
LPS is adopted via API retrieval — a third party fetches or receives
LPS-embedded text and passes it to a model — the same overhead
applies at that call boundary. In either case, integrations must
account for this overhead in token budget planning.

Strip the manifest before passing text to contexts where token cost
is the primary constraint and provenance is not required at that step.

### 6.3 C2PA Working Group Submission

This proposal has been prepared as a formal working-group
submission. The reference implementation validates the technical
architecture described in this document.
---

## 7. References

1. Hoque, M.N. et al. "The HaLLMark Effect: Supporting Provenance and Transparent Use of Large Language Models in Writing with Interactive Visualization." CHI 2024. https://dl.acm.org/doi/fullHtml/10.1145/3613904.3641895
2. C2PA Specification v2.4. https://c2pa.org/specifications
3. encypherai/c2pa-text. https://github.com/encypherai/c2pa-text
4. Screenshot Gap Analysis. https://medium.com/@sevrusik/the-death-of-the-screenshot-why-c2pa-fails-against-ai-generated-evidence
5. OpenAI/Google Dual-Layer Announcement, May 2026. https://c2paviewer.com/articles/openai-google-c2pa-synthid-2026
6. 2026 US Treasury National Money Laundering Risk Assessment
7. FBI Voice Cloning Family Scam Guidance, 2024
8. EU AI Act Article 50 — Synthetic Content Disclosure Requirements
9. Golaszewski, E. et al. "Verifying Provenance of Digital Media: Why the C2PA Specifications Fall Short." UMBC/NSA, April 2026. https://arxiv.org/html/2604.24890v1

---
## About

LPS is a research-driven system built to explore provenance tracking, cryptographic integrity, and adversarial resilience in AI-assisted text systems.

The project is developed independently with a focus on security engineering, system design, and verifiable structure rather than rapid feature shipping or surface-level tooling.

The goal is to evolve LPS into a robust reference implementation that can be reviewed, tested, and extended by security and AI infrastructure communities.

*v0.1 — Work in Progress — July 2026*

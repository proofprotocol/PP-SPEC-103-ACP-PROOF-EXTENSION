# AI Disclosure

*How this specification was made, and by whom.*

This file states what in PP-SPEC-103 came from a human and what came from an AI assistant. It is published so that readers, implementers, standards bodies, and examiners can judge the provenance of the work for themselves.

## Summary

The method is Craig Ellrod's. The words were drafted with Claude.

Craig Ellrod, a human, directed this work and owns every Proof Protocol method it references. Claude, an AI assistant made by Anthropic, was used to analyze the ACP repository and to draft, structure and edit the text under his direction. Every section is subject to review and acceptance by Craig Ellrod before release.

## Ratio

| Dimension | Human | AI |
|---|---|---|
| Conception and every normative decision | 75% | 25% |
| Design details and naming | 40% | 60% (Claude proposed: delivery through the ACP Extensions Framework, the field names, session_state_hash and its exclusions, the approval binding, the error codes, and the no-self-verification rule) |
| Words, code, and schemas as written | 5% | 95% |
| Review, direction, acceptance, decision to publish | 100% | 0% |

Method: self-attested. Shares drafted by the AI from its itemized record of what it proposed and what the author directed, for the author to confirm or correct before release. MAY be re-measured with a tool such as ProofJack.com.

Machine-readable form: [ai_disclosure.json](ai_disclosure.json), conforming to PP-SPEC-077 Section 21.15. The two forms agree; if they ever differ, the JSON is wrong and must be corrected.

## This version (0.1.0, 2026-11-05)

**Human (Craig Ellrod):**
- The Proof Correlation ID, Time Anchor, exclusionary boundary and mandate concepts the extension carries
- Decision that the ACP interface is its own Proof Protocol specification, the interface to ACP's interface
- Decision to contribute to ACP rather than stay a silent witness, keeping methods out of anything submitted under the ACP CLA
- Decision that the extension leads with proof on the product (Part A) and treats purchase integrity as secondary (Part B)
- Canonical definition: proof that a product under pressure, does what the vendor says it does
- Designated ProofRegister (https://proofregister.com) as the reference Registry
- Review and acceptance (pending)

**AI (Claude Opus 5.5):**
- Use of the ACP Extensions Framework (third-party extension with schema and spec URLs) so normative text stays in this repository
- Field set, processing rules, error codes, conformance checklist, and JSON Schema
- The `session_state_hash` approval binding and the no-self-verification rule
- SEP draft text
- Text as written and the repository files
- Finding that ACP feeds are closed (additionalProperties false) with no extension mechanism, and the resulting two-part SEP (feed extensibility in core; proof as a third-party extension)
- EfficacyClaim, TestedSubject, RecordRef and ClaimResult fields; the custom_attributes interim pointer
- Rule that reproof status is looked up in the Registry, never stored in a claim, following PP-SPEC-077 21.8.3
- Text of Section 4.4 and the note that an unreachable Registry yields unproven claims, not false proof

**Pending acceptance by Craig Ellrod:**
- Field set and names
- session_state_hash exclusions (proof, messages)
- Error codes
- EfficacyClaim field set
- Two-part SEP framing

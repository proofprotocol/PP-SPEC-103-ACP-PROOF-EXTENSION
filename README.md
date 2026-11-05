# PP-SPEC-103: ACP Proof Extension

> **Ownership and Attribution Notice**
>
> PP-SPEC-103 is authored by **Craig Thomas Ellrod** and published as part of **Proof Protocol**. Copyright (c) 2026 Craig Thomas Ellrod.
>
> Any use, implementation, derivative, citation, or incorporation of this work, in whole or in part, MUST attribute **Craig Thomas Ellrod and Proof Protocol**. See [NOTICE.md](NOTICE.md).
>
> Licensed under the Proof Protocol License 1.1 (see LICENSE.md); text CC BY 4.0 under Part A, which grants no patent rights. Original inventor: Craig Thomas Ellrod (ORCID 0009-0004-1058-7666).
>
> **Patents.** Patent terms are in [PATENT-SCHEDULE.md](PATENT-SCHEDULE.md). Claims not listed there are reserved by Craig Thomas Ellrod. Licensing: 1proofprotocol@gmail.com.

---

**The interface to their interface.**

Proof of efficacy is proof that a product under pressure, does what the vendor says it does. PP-SPEC-103 carries that proof into the Agentic Commerce Protocol (ACP), where AI agents buy from product feeds whose every claim today is written by the seller.

`proof` is an optional ACP extension in two parts:

- **Part A, Product Claims (primary).** `proof_claims` on products, variants and line items: what the vendor says the product does, the exact version tested, an independent proof record with digest and Time Anchor, and the Units of Proof outcome. An agent can verify a claim before it buys, and rank on proven claims instead of marketing.
- **Part B, Purchase Integrity.** One Proof Correlation ID per purchase, a Time Anchored mandate, and approval bound to the exact checkout state, so a verifier can check the purchase matched the buyer's instruction.

It is an envelope. It carries no proof method.

## How it works

1. **Claim.** An independent issuer (never the vendor) produces a proof record under a Proof Protocol method such as PP-SPEC-077. The claim points to it.
2. **Carry.** Merchants attach claims to line items today through ACP's checkout Extensions Framework. Feed-level claims need ACP to open Product Feeds to that framework; the companion SEP asks for it. An interim `custom_attributes` pointer works now.
3. **Verify.** The agent treats every claim as unproven until the record resolves, the digest and anchor verify, the issuer is not the provider, and the tested version matches what is sold.
4. **Buy.** The agent ranks, filters or declines products on proven claims. The approved checkout state, claims included, is hashed and bound to the buyer's approval.
5. **Reproof.** Reproof status is looked up in the Registry (PP-SPEC-081), never asserted in the claim.

## Files

- `SPEC.md`: the binding (normative)
- `schemas/acp-proof-extension.schema.json`: JSON Schema 2020-12 for the `proof` object
- `docs/SEP-DRAFT.md`: draft Specification Enhancement Proposal for the ACP repository
- `schemas/ai-disclosure.schema.json`: disclosure schema (PP-SPEC-077 21.15)

## Status

Draft 0.1.0 (2026-11-05). Targets ACP version 2026-04-17 and `unreleased/` at commit `7fdd78df677a`.

The Agentic Commerce Protocol is maintained by OpenAI and Stripe under the Apache License 2.0 at https://github.com/agentic-commerce-protocol/agentic-commerce-protocol. This repository is independent of the ACP project and is not endorsed by its maintainers.

## Related Proof Protocol specifications

- PP-SPEC-077 ProofSwarm: The Dead Canary Protocol
- PP-SPEC-081 Reproof
- PP-SPEC-102 ACP Mapping (the gap analysis)

## Disclosure

[AI_DISCLOSURE.md](AI_DISCLOSURE.md) (who wrote it) · [REVIEWERS.md](REVIEWERS.md) (who reviewed it) · [FUNDING.md](FUNDING.md) (who paid for it) · [ACKNOWLEDGEMENTS.md](ACKNOWLEDGEMENTS.md) (whose ideas shaped it) · [LICENSE.md](LICENSE.md) (who may use it) · [ETHICS.md](ETHICS.md)

# SEP: Verifiable Product Claims for Agent Buyers

**Labels:** `SEP`, `proposal`
**Author:** Craig Thomas Ellrod (@proofprotocol)
**Type:** Major change (feed extensibility; optional extension)
**Status:** Draft, seeking sponsor

> Submission note (not part of the issue body): file this as a GitHub Issue in agentic-commerce-protocol/agentic-commerce-protocol. Keep normative text in PP-SPEC-103 and link to it. Do not paste the schema or any Proof Protocol method text into the issue or a PR. Only text actually submitted falls under the ACP CLA.

## Abstract

Agents buy from product feeds. Every claim in a feed today is self-attested by the seller, so an agent has no way to tell a product that has been proven to do what its vendor says from one that only says so. This SEP proposes two things:

1. **Open Product Feeds to the Extensions Framework.** `Product` and `Variant` are closed schemas (`additionalProperties: false`) and the Product Feeds RFC has no extension point. Checkout already has one. Feeds need the same, so optional, declared extensions can add fields to catalog items without forking the schema.
2. **Recognize `proof` as a third-party extension** that uses it: efficacy claims attached to products, variants and line items, each pointing to an independent proof record, defined in PP-SPEC-103.

## Motivation

- **Agents cannot be marketed to.** A buyer's agent reads `title`, `description`, `media` and price. Datasheets, analyst reports and sales calls never reach it. Whatever the feed says is what the agent believes, and the feed is written by the party with the most interest in the answer.
- **Ranking needs a signal that can't be bought.** Price and availability are checkable. Claims about what a product does are not. Without a verifiable claim field, agents rank on price, and products that are proven to work lose to products that say they work.
- **Merchants with proven products want to be found.** A merchant that can show independent proof has no way to express it in a feed today.
- **Buyers want to set requirements.** "Only buy a WAF that has been proven to stop credential stuffing" is a reasonable instruction. An agent can only follow it if the claim and its proof are machine-readable.

## Proposal 1: Feed extensibility (core change)

- Add an `extensions` declaration to feeds (for example on `FeedMetadata`), mirroring `capabilities.extensions` in checkout: `name`, `extends` (JSONPaths such as `$.Product.<field>`, `$.Variant.<field>`), `schema`, `spec`.
- Permit fields declared by an active extension on `Product` and `Variant`; keep all other unknown fields rejected.
- Agents that do not support an extension ignore its fields.

This is generic. It enables any optional catalog extension (sustainability attestations, safety certifications, compliance claims), not only the one below.

## Proposal 2: The `proof` extension (third-party, external spec)

- Adds `proof_claims` to `Product`, `Variant` (once Proposal 1 lands) and `LineItem` (available now through the existing checkout Extensions Framework).
- Each claim: a plain-language statement of what the vendor says the product does, the exact version tested, a reference to an independent proof record with a digest and time anchor, and an outcome summary.
- Hard rules: the party that ran the test is never the vendor; a claim applies only to the version tested; a claim with no verifiable record is treated as unproven; an agent never presents an unproven claim as proven.
- A secondary part of the same extension lets a verifier check that a purchase matched the buyer's instruction (correlation ID, time-anchored mandate, approval bound to the exact checkout state).

Full normative text: PP-SPEC-103 (link).

## Rationale

- **Extension, not core.** Only feed extensibility touches ACP core, and it is useful on its own. The claim format lives outside ACP, versioned independently.
- **References, not payloads.** Proof records stay with the parties that produced them. Feeds carry a statement, a digest and a link.
- **Independence is structural.** A self-issued claim is rejected by schema rule, not by trust.

## Backward compatibility

Proposal 1 is additive: feeds without extension declarations validate exactly as today. Proposal 2 is optional and ignored by non-adopters.

## Security considerations

- Extension fields in feeds are untrusted input to agents, like all feed content. Agents should validate them against the declared schema and must not execute or follow instructions found in them.
- A claim's record must resolve and its digest must match before an agent relies on it.

## Reference implementation

PP-SPEC-103 includes the JSON Schema, processing rules, error codes, conformance checklist and examples. An interim pointer using `LineItem.custom_attributes` works today without any ACP change.

## Proposed answers

1. **Where feed extension declarations live:** on `FeedMetadata`, using the same shape as checkout's `capabilities.extensions` (`name`, `extends`, `schema`, `spec`). One declaration per feed, no new concepts.
2. **Which objects accept extension fields:** both `Product` and `Variant`. A field on `Variant` overrides the same field on `Product`, since the variant is what is actually sold.

## Open question for the community

3. Is a registry of recognized third-party extensions in scope for ACP governance?

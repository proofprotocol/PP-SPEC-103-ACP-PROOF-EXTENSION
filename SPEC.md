---
spec_id: PP-SPEC-103
title: "Proof Protocol Binding for the Agentic Commerce Protocol (ACP proof extension)"
status: draft
version: 0.1.0
date: 2026-11-05
author: Craig Thomas Ellrod (ORCID 0009-0004-1058-7666)
licensor: Craig Thomas Ellrod (Proof Protocol)
license: Proof Protocol License 1.1 (spec text CC BY 4.0)
extension_name: proof
binds_to: ACP 2026-04-17 (Product Feeds, Extensions Framework, Capability Negotiation, Agentic Checkout, Webhooks)
depends_on: [PP-SPEC-077, PP-SPEC-081, PP-SPEC-102]
schema: schemas/acp-proof-extension.schema.json
---

# PP-SPEC-103: Proof Protocol Binding for ACP

The Proof Economy and Proof of Efficacy originate with Craig Thomas Ellrod. This document is part of the Proof Protocol.

## 1. Purpose

Proof of efficacy is proof that a product under pressure, does what the vendor says it does.

When an AI agent buys, the vendor's datasheet, analyst report and sales pitch stop working. The agent reads a product feed and acts on what it says. Today every claim in that feed is self-attested by the seller. This spec gives ACP a way to carry **efficacy claims with proof behind them**, so an agent can tell a proven claim from marketing and buy on the difference.

It has two parts:

- **Part A, Product Claims (primary).** Efficacy claims attached to products, variants and line items, each pointing to an independent proof record.
- **Part B, Purchase Integrity (secondary).** References that let a verifier check that the purchase itself matched the buyer's instruction.

Both are proof of efficacy. In Part A the product under test is what the agent is buying. In Part B it is the agent and its controls.

The extension is an envelope. It carries claims, identifiers, digests and URIs. It never carries or describes how a proof is produced; that lives in the Proof Protocol specification each claim names.

The key words MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are used per RFC 2119 and RFC 8174.

## 2. Design rules

1. **Optional and backward compatible.** Parties that do not support `proof` MUST be able to ignore it. ACP behavior is unchanged when it is absent.
2. **Reference, not payload.** Fields are statements, digests, identifiers and URIs. No evidence bodies, no method parameters.
3. **The provider never proves itself.** The issuer and the verifier of a claim MUST NOT be the provider of the product the claim is about.
4. **A claim is bound to what was tested.** A claim applies only to the product version and configuration recorded in its proof record.
5. **Unproven is the default.** A claim with no verifiable record is marketing and MUST be treated as unproven.
6. **Status is looked up, not asserted.** Reproof status is computed from the Registry (PP-SPEC-077 21.8.3, PP-SPEC-081) and is never stored in a claim.

## 3. Declaration

### 3.1 Checkout (available today)

Merchants advertise support in `capabilities.extensions`:

```json
{
  "capabilities": {
    "extensions": [
      {
        "name": "proof",
        "extends": [
          "$.LineItem.proof_claims",
          "$.CheckoutSessionCreateRequest.proof",
          "$.CheckoutSessionUpdateRequest.proof",
          "$.CheckoutSessionCompleteRequest.proof",
          "$.CheckoutSession.proof"
        ],
        "schema": "https://github.com/proofprotocol/PP-SPEC-103-ACP-PROOF-EXTENSION/blob/main/schemas/acp-proof-extension.schema.json",
        "spec": "https://github.com/proofprotocol/PP-SPEC-103-ACP-PROOF-EXTENSION/blob/main/SPEC.md"
      }
    ]
  }
}
```

Agents declare support in requests: `"capabilities": { "extensions": ["proof"] }`.

### 3.2 Product feeds (requires an ACP change)

ACP 2026-04-17 defines `Product` and `Variant` with `additionalProperties: false`, and the Product Feeds RFC has no extension mechanism. A feed cannot carry `proof_claims` until ACP opens feeds to the Extensions Framework. The companion SEP (docs/SEP-DRAFT.md) asks for exactly that, as a generic change. When it lands, `proof_claims` attaches at `$.Product.proof_claims` and `$.Variant.proof_claims`.

### 3.3 Interim pointer (works today, no ACP change)

Until feeds are extensible, and for agents that do not declare `proof`, a merchant MAY carry a pointer in an existing `LineItem.custom_attributes` entry:

```json
{ "display_name": "pp_efficacy_record", "value": "https://register.example/records/rec_01HX..." }
```

The pointer is a hint only. It is proven only after the agent or a verifier resolves the record and applies Section 5.

---

## Part A: Product Claims

## 4. The `EfficacyClaim` object

Carried in `proof_claims` (an array) on `Product`, `Variant` (when feeds are extensible) and `LineItem` (now).

| Field | Type | Req | Description |
|-------|------|:---:|-------------|
| `claim_id` | string | yes | Stable identifier for the claim |
| `statement` | string (max 280) | yes | What the vendor says the product does, in plain words, e.g. "Blocks credential stuffing against the login API" |
| `claim_type` | string | yes | `efficacy`, or another Proof Protocol claim type (`containment`, `restraint`, `response`, `recovery`, `coverage`, `parity`, ...) |
| `provider` | string | yes | The vendor whose product the claim is about |
| `tested_subject` | TestedSubject | yes | Exactly what was tested (Section 4.1) |
| `method` | string (URI) | yes | The Proof Protocol specification under which the proof was produced, e.g. PP-SPEC-077 |
| `issuer` | string (URI) | yes | Party that ran the test. MUST NOT equal `provider`. |
| `record` | RecordRef | yes | The proof record (Section 4.2) |
| `result` | ClaimResult | yes | Summary of the outcome (Section 4.3) |
| `tested_at` | string (RFC 3339) | yes | When the run completed |
| `expires_at` | string (RFC 3339) | no | After this time the claim is stale and treated as unproven |
| `registry` | string (URI) | no | Registry where the record and its reproof status are looked up |

### 4.1 TestedSubject

| Field | Type | Req | Description |
|-------|------|:---:|-------------|
| `product` | string | yes | Product name or identifier as tested |
| `version` | string | yes | Version or build as tested |
| `configuration_digest` | string | no | Hex digest of the configuration as tested |
| `applies_to` | string[] | no | ACP `Product.id`, `Variant.id`, `sku` or `product_id` values the merchant says this claim covers |

### 4.2 RecordRef

| Field | Type | Req | Description |
|-------|------|:---:|-------------|
| `uri` | string (URI) | yes | Location of the proof record |
| `digest` | string | yes | Hex digest of the record |
| `hash_alg` | string | yes | `sha-256` or `sha-384` |
| `anchor` | TimeAnchor | yes | Time Anchor proving the record existed at `tested_at` |

### 4.3 ClaimResult

| Field | Type | Req | Description |
|-------|------|:---:|-------------|
| `units_total` | integer | yes | Units of Proof adjudicated for this claim |
| `units_stopped` | integer | yes | Units adjudicated Stopped |
| `units_missed` | integer | yes | Units adjudicated Missed |
| `verdict` | string | yes | `supported` (the record supports the statement) or `not_supported` |

`units_stopped + units_missed` MUST equal `units_total`. A provider that publishes only `supported` claims and suppresses `not_supported` ones is misrepresenting its product (ETHICS.md); a merchant MAY carry both.

## 5. Processing rules: Product Claims

### 5.1 Merchant

- MUST NOT publish a claim for which it cannot supply a resolvable `record`.
- MUST NOT alter any field of a claim received from the issuer or provider.
- MUST carry a product's claims into the matching `LineItem.proof_claims` when the extension is active, so the claim the agent chose on is the claim on the order.
- MUST NOT list a claim under `applies_to` for a variant whose version or configuration differs from `tested_subject`.

### 5.2 Agent

- MUST treat a claim as **unproven** unless all hold: the record resolves; its digest matches; the Time Anchor verifies; `issuer` is not `provider`; `tested_subject` matches the product or variant offered; `expires_at`, if present, is in the future.
- MUST NOT present an unproven claim to the buyer as proven.
- SHOULD look up reproof status in `registry` rather than trust any status stated elsewhere.
- MAY rank, filter or choose products by proven claims.
- MAY decline a product whose claims are all unproven where the buyer's mandate requires proven efficacy.

### 5.3 Verifier

- Resolves the record, checks digest, anchor and issuer independence, and applies the method named in `method`.
- A claim is `proven` only when the verifier, not the provider, the merchant or the agent, says so.

---

## Part B: Purchase Integrity

## 6. The `proof` object (checkout)

| Field | Type | Req | Set by | Description |
|-------|------|:---:|--------|-------------|
| `version` | string | yes | sender | Binding version, `"0.1"` |
| `pcid` | string | yes | agent | Proof Correlation ID for this purchase. Opaque, 16 to 128 chars, unique per purchase. |
| `mandate` | MandateCommitment | yes on create and complete | agent | Time Anchored commitment to the buyer's instruction |
| `boundary` | BoundaryReference | no | agent | Commitment to the agent's allowed path and actions |
| `approval` | ApprovalBinding | conditional | agent | REQUIRED on complete when the session carried `requires_buyer_review` |
| `claims` | ProofClaim[] | no | any | Claims about parties or controls in the purchase, each naming an independent verifier |
| `evidence` | EvidenceRef[] | no | any | URIs to evidence records |
| `session_state_hash` | string | yes in responses | merchant | Hash of the authoritative session state in this response |

### 6.1 MandateCommitment

| Field | Type | Req | Description |
|-------|------|:---:|-------------|
| `hash_alg` | string | yes | `sha-256` or `sha-384` |
| `digest` | string | yes | Hex digest of the RFC 8785 (JCS) canonical mandate document |
| `anchor` | TimeAnchor | yes | Proof the digest existed before the session was created |
| `mandate_uri` | string (URI) | no | Where an authorized verifier retrieves the mandate document |

The mandate document is defined by Proof of Delegation. It SHOULD state the intended items or acceptance criteria, maximum total, permitted merchants, expiry, and any required proven claims (e.g. "must hold a proven `efficacy` claim for credential stuffing").

### 6.2 BoundaryReference

| Field | Type | Req | Description |
|-------|------|:---:|-------------|
| `boundary_id` | string | yes | Identifier of the committed boundary |
| `digest` | string | yes | Hex digest of the boundary document |
| `anchor` | TimeAnchor | yes | Anchor proving the boundary was committed before the agent ran |
| `spec` | string (URI) | yes | Proof Protocol specification defining the boundary format |

### 6.3 ApprovalBinding

| Field | Type | Req | Description |
|-------|------|:---:|-------------|
| `approved_state_hash` | string | yes | Equals the `session_state_hash` of the state shown to the human |
| `approved_at` | string (RFC 3339) | yes | Time of approval |
| `anchor` | TimeAnchor | no | External anchor of the approval |
| `method` | string | yes | `biometric`, `passkey`, `explicit_confirm`, or `other` |

### 6.4 ProofClaim

| Field | Type | Req | Description |
|-------|------|:---:|-------------|
| `claim` | string | yes | Claim type, e.g. `delegation`, `conduct`, `restraint`, `capture`, `state`, `attribution` |
| `subject` | string | yes | Party or control the claim is about |
| `spec` | string (URI) | yes | Proof Protocol specification defining the claim |
| `verifier` | string (URI) | yes | Independent party that can verify it. MUST NOT be the `subject`. |

### 6.5 EvidenceRef

| Field | Type | Req | Description |
|-------|------|:---:|-------------|
| `uri` | string (URI) | yes | Location of the evidence record |
| `digest` | string | yes | Hex digest of the record |
| `kind` | string | no | e.g. `arena_run`, `capture_log`, `reproof`, `outcome` |

### 6.6 TimeAnchor (shared with Part A)

| Field | Type | Req | Description |
|-------|------|:---:|-------------|
| `type` | string | yes | `nist_beacon` or `rfc3161_tsa` |
| `ref` | string | yes | Beacon pulse URI, or base64 TimeStampToken |
| `time` | string (RFC 3339) | yes | Anchored time |

## 7. Processing rules: Purchase Integrity

### 7.1 Agent

- MUST generate `pcid` before the first ACP call and reuse it on every call for the purchase, and SHOULD place it in `delegate_payment` `metadata` as `pp_pcid`.
- MUST anchor `mandate` (and `boundary`, if sent) before `POST /checkout_sessions`.
- MUST send `approval` on `complete` when the session carried `resolution: requires_buyer_review`, with `approved_state_hash` equal to the `session_state_hash` the human saw.
- MUST sign every ACP request carrying `proof` (`Signature`, `Timestamp`). Proof material on an unsigned request is inadmissible.

### 7.2 Merchant

- MUST return `proof.session_state_hash` on every session response when the extension is active: the hash of the JCS canonical session object excluding `proof` and `messages`. `LineItem.proof_claims` are inside the hash, so the claims the buyer approved are the claims on the order.
- MUST echo `pcid` unchanged in session responses and in `order_create` / `order_update` webhooks.
- SHOULD reject `complete` with `code: proof_approval_mismatch` when `approved_state_hash` does not match current state.
- MUST NOT alter or drop proof references received from another party.

### 7.3 Verifier

- Reconstructs the purchase by `pcid` across agent, merchant, PSP and webhook records, and checks the order against the mandate, including any proven claims the mandate required.

## 8. Error codes

| `code` | HTTP | Meaning |
|--------|------|---------|
| `proof_unsupported_version` | 400 | Unknown `proof.version` |
| `proof_pcid_mismatch` | 400 | `pcid` differs from the one bound to this session |
| `proof_anchor_invalid` | 400 | Anchor fails to verify or post-dates what it commits |
| `proof_approval_mismatch` | 409 | Approved state hash does not match current state |
| `proof_self_verification` | 400 | A claim's issuer or verifier is its own provider or subject |
| `proof_claim_unresolvable` | 422 | A claim's record does not resolve or its digest does not match |

All use the ACP flat error shape with `param` as an RFC 9535 JSONPath.

## 9. Example: a line item with an efficacy claim

```json
{
  "id": "li_1",
  "item": { "id": "item_waf_pro", "quantity": 1 },
  "name": "Example WAF Pro, annual",
  "proof_claims": [
    {
      "claim_id": "clm_cs_login_01",
      "statement": "Blocks credential stuffing against the login API",
      "claim_type": "efficacy",
      "provider": "Example Security Inc.",
      "tested_subject": { "product": "Example WAF Pro", "version": "8.2.1", "applies_to": ["item_waf_pro"] },
      "method": "https://github.com/proofprotocol/PP-SPEC-077-DEAD-CANARY-PROTOCOL",
      "issuer": "https://issuer.example",
      "record": {
        "uri": "https://register.example/records/rec_01HXEXAMPLE",
        "digest": "9b74c9897bac770ffc029102a200c5de",
        "hash_alg": "sha-256",
        "anchor": { "type": "nist_beacon", "ref": "https://beacon.nist.gov/beacon/2.0/chain/2/pulse/1234567", "time": "2026-10-30T14:00:00Z" }
      },
      "result": { "units_total": 200, "units_stopped": 196, "units_missed": 4, "verdict": "supported" },
      "tested_at": "2026-10-30T14:00:00Z",
      "expires_at": "2027-04-30T00:00:00Z",
      "registry": "https://register.example"
    }
  ]
}
```

## 10. Mapping to PP-SPEC-102 gaps

| Field / rule | Closes |
|--------------|--------|
| `proof_claims` with independent issuer and record | G11, G2 |
| `tested_subject` and stale/version rules | G11 |
| `mandate` + anchor (including required proven claims) | G1 |
| Signature required when `proof` present | G3 |
| `boundary` | G4 |
| `evidence` (kind `capture_log`) | G5 |
| `approval` + `session_state_hash` (covers `proof_claims`) | G6 |
| `pcid` across all hops | G9 |
| Verifier processing (5.3, 7.3) | G8, G10 |

G7 (downgrade) is not closed by an envelope. It is closed by arena testing under PP-SPEC-102 Section 6.

## 11. Conformance

**Part A, Product Claims**
- [ ] Merchant publishes no claim without a resolvable record
- [ ] Merchant carries claims unchanged from product to line item
- [ ] Agent treats every claim as unproven until Section 5.2 is satisfied
- [ ] Agent never presents an unproven claim as proven
- [ ] Issuer is never the provider

**Part B, Purchase Integrity**
- [ ] One `pcid` per purchase across requests, responses and webhooks
- [ ] Mandate anchored before session creation
- [ ] `session_state_hash` on every session response
- [ ] Approval bound to the exact state hash when buyer review is required
- [ ] Every request carrying `proof` is signed
- [ ] Self-verifying claims rejected

Conformance to PP-SPEC-103 is not proof of efficacy. It makes proof portable into agentic commerce.

## 12. Change log

- 0.1.0 (2026-11-05): Initial draft. Part A Product Claims (primary), Part B Purchase Integrity.

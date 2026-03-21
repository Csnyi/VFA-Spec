# Glossary

This document is the single authoritative source for VFA terminology.

Where a term appears in multiple specification documents, this glossary takes precedence. Individual documents may reference this file rather than re-defining terms locally.

Version: **v0.1**

---

## Actors

### User
The human principal whose explicit approval is required before a protected interaction may proceed. The user is the **root trust anchor** of the VFA model — the legitimacy of every visa token ultimately derives from a confirmed user approval.

### Wallet
A user-controlled interface responsible for presenting handshake requests to the user and collecting approval decisions. The wallet does **not** sign or mint tokens in v0.1; its sole role is approval collection and faithful display of the intent.

### Merchant
The requesting application or relying party that initiates a handshake request and presents the resulting visa token to the gateway. The merchant is treated as a **potentially hostile intermediary** — it may not modify any field covered by the issuer signature.

### Issuer
The sole authority that signs and mints visa tokens in VFA v0.1. The issuer acts on confirmed wallet approval and includes all security-relevant fields in the signed payload. Also referred to as "Verification server" in earlier drafts — **Issuer** is the normative term.

### Gateway
The policy enforcement and verification point that sits between the merchant and the backend. The gateway MUST verify every inbound visa token before dispatching a request. It enforces deny-by-default routing and maintains the nonce deduplication store.

### Backend
The protected service that executes verified requests. The backend trusts only the gateway (enforced via mTLS service identity) and executes only against the verified, committed intent.

---

## Core artifacts

### Intent
A structured declaration describing a requested action, created by the merchant and approved by the user via the wallet. The intent is the human-readable subject of the approval decision.

Example:
```json
{
  "merchantId": "merchant.example",
  "endpoint": "/payments",
  "amount": "24.90",
  "currency": "EUR"
}
```

The intent is **not** directly transmitted as a signed artifact in v0.1. It is the basis on which the issuer mints the visa token.

### Visa Token
A short-lived, cryptographically signed and immutable object issued by the VFA issuer that binds:

- user identity (`sub`)
- issuer identity (`iss`)
- merchant identity (`merchantId`)
- authorized endpoint (`endpoint`)
- policy scope (`scope`)
- expiration (`exp`, `iat`)
- replay protection (`nonce`)
- key identifier (`kid`)

The gateway verifies the visa token before any protected request is dispatched to the backend.

### Handshake Request
A structured message created by the merchant that initiates the VFA flow. It describes the requested action and is presented to the user via the wallet for approval.

Key fields: `requestId`, `merchantId`, `scope`, `endpoint`, `ttlMs`, `nonce`.

---

## Key concepts

### Approval Window (`ttlMs`)
The duration in **milliseconds** during which the wallet displays the handshake request and the user may approve or reject it. Defined by the `ttlMs` field in the handshake request.

Distinct from token validity. A long approval window does not imply a long token lifetime.

### Token Validity (`exp − iat`)
The lifetime of a visa token, expressed as the difference between the `exp` and `iat` fields in **Unix epoch seconds**. In production deployments this MUST not exceed **60 seconds**.

### Nonce
A 128-bit cryptographically random value unique per handshake request and included in both the handshake request and the visa token.
Used for replay protection: the gateway maintains a deduplication store and rejects any token whose nonce has been seen before.

### Scope
A string declaring the category of operation being authorized (e.g. `payment:init`). The recommended format is `namespace:action`.
The gateway enforces exact scope matching against the signed token value.

### intentHash
A gateway-internal binding mechanism: a hash of the verified visa token payload that the gateway commits to before dispatching the request to the backend. Ensures the backend executes only against the verified intent and not a re-interpreted version.

`intentHash` is not a token field in v0.1. It is a gateway implementation detail described in the threat model (T-08).

### Conceptual Policy Layer
The VFA admission layer responsible for verifying declared intent before protected resources are accessed. Not a literal OSI layer — in v0.1, verification occurs at the gateway after TLS termination.

### Context Binding
The practice of including `merchantId`, `endpoint`, `scope` and `sub` in the signed token payload so that a token issued for one context cannot be replayed in a different one.

### Key Identifier (`kid`)
A field in the visa token that identifies which issuer key was used to sign the token. Enables key rotation without coordinated cutover: the gateway uses `kid` to select the correct public key for verification. Optional in v0.1, **required in production**.

---

## Normative language

This specification uses the following terms with the meanings defined in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119):

- **MUST** / **MUST NOT** — absolute requirement / prohibition
- **SHOULD** / **SHOULD NOT** — recommended unless specific reasons justify otherwise
- **MAY** — optional

---

## Deprecated terms

The following terms appeared in earlier drafts and are replaced by the normative terms listed above.

| Deprecated term | Normative term | Notes |
|-----------------|---------------|-------|
| `issuer` (field) | `iss` | Aligned with canonical field names |
| `subject` (field) | `sub` | Aligned with canonical field names |
| `tokenId` (field) | removed in v0.1 | Not part of canonical field set |
| `merchant` (field) | `merchantId` | Renamed for clarity |
| `ttl` (field) | `ttlMs` | Renamed to make unit explicit |
| Verification server | Issuer | Normative actor name |
| Service | Backend | Normative actor name |

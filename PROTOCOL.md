# VFA Handshake Protocol

The VFA handshake ensures that a user or controlling entity explicitly approves an interaction before it is executed.

---

## Actors

- **Wallet** — user-controlled approval interface
- **Merchant** — requesting application or relying party
- **Issuer** — authority minting and signing visa tokens
- **Gateway** — policy enforcement and verification point
- **Backend** — protected service executing verified requests

> In some deployments the Issuer and Gateway may be co-located, but they represent distinct logical roles.

---

## Protocol Flow (v0.1)

### Step 1 — Merchant creates a handshake request

```json
{
  "requestId": "req-12345",
  "merchantId": "merchant.example",
  "scope": "payment:init",
  "endpoint": "/payments",
  "ttlMs": 300000,
  "nonce": "abcxyz128bit"
}
```

`ttlMs` defines the user-facing approval window in **milliseconds** (how long the wallet displays the request). It is distinct from the token's `exp` field.

### Step 2 — Wallet displays the request to the user

The wallet renders the intent summary independently from the merchant's UI, based on the structured schema.

### Step 3 — User approves or rejects

The wallet returns one of two decisions:

- **ACCEPT**
- **REJECT**

A rejected request must not produce a visa token.

### Step 4 — Issuer generates a visa token

On acceptance, the **issuer** mints a short-lived visa token (see [TOKEN_FORMAT.md](TOKEN_FORMAT.md)) binding the approved scope, identity, merchant, endpoint, and expiration.

Token lifetime in production: **≤ 60 seconds** (`exp - iat`).

### Step 5 — Merchant sends request with token

```http
POST /payments HTTP/1.1
Host: gateway.example
Authorization: Bearer <visa_token>
Content-Type: application/json
```

### Step 6 — Gateway verifies the token

The gateway checks:

- signature (issuer public key)
- expiration (`exp`)
- nonce uniqueness (deduplication store)
- merchant and endpoint binding (`merchantId`, `endpoint`)
- scope match

### Step 7 — If valid, request is forwarded to backend

The backend executes only against the verified, committed intent.

---

## Handshake request fields

| Field | Required | Meaning |
|-------|----------|---------|
| `requestId` | yes | unique request identifier |
| `merchantId` | yes | requesting party identity |
| `scope` | yes | requested operation category |
| `endpoint` | yes | target API endpoint |
| `ttlMs` | yes | approval window in milliseconds |
| `nonce` | yes | freshness value for replay protection |

---

## Possible policy outcomes

- allow request to production backend
- route request to sandbox backend
- deny request entirely
- require re-approval

---

## Error and Rejection Handling

VFA v0.1 defines both a successful authorization path and explicit failure paths. A protected interaction MUST proceed only if the visa token is valid and the gateway verification succeeds immediately before dispatch.

### User rejection

If the user rejects the handshake request in the wallet, the issuer MUST NOT mint a visa token.

The merchant MAY receive a rejection result from the issuer or wallet-facing approval channel, for example:

- `status: rejected`
- `reason: user_rejected`

In this case, the flow terminates and no protected request may be sent to the gateway for that interaction.

### Expired or abandoned approval

If the user does not approve the handshake request within the approval window defined by `ttlMs`, the request is considered expired.

After expiry:

- the wallet MUST treat the request as no longer actionable
- the issuer MUST NOT mint a visa token for that request
- the merchant MUST obtain a new handshake request before retrying

A long approval window does not extend token validity and does not authorize late issuance.

### Invalid or unacceptable token

If the merchant sends a request with a missing, invalid, expired, replayed, or context-mismatched visa token, the gateway MUST reject the request and MUST NOT forward it to the backend.

Examples of invalidity include:

- signature verification failure
- unknown or unacceptable `kid`
- `exp` already elapsed
- nonce replay detected
- `aud`, `merchantId`, `endpoint`, or `scope` mismatch
- malformed token encoding
- token lifetime exceeds policy
- actual HTTP request does not match the signed token fields

In these cases, the gateway returns an authorization failure response. A typical HTTP mapping is:

- `401 Unauthorized` for missing, malformed, expired, or unverifiable token
- `403 Forbidden` for validly formed but policy-disallowed or context-mismatched token

Implementations MAY choose different status codes, but they MUST preserve the security property that failed verification results in no backend dispatch.

### Error response guidance

Gateway error responses SHOULD be concise and safe for exposure to an untrusted intermediary. They SHOULD indicate the failure class without revealing sensitive verifier internals.

Example response body:

```json
{
  "error": "invalid_token",
  "reason": "signature_verification_failed"
}
```
Example rejection cases include:

- missing_token
- invalid_token
- expired_token
- replayed_token
- context_mismatch
- policy_denied
- user_rejected

### Fail-closed requirement

All verification failures are fail-closed.

If verification cannot be completed, the gateway MUST deny the request by default and MUST NOT dispatch it to the backend. The backend MUST accept requests only from the gateway and MUST treat direct merchant access as unauthorized.

---

## v0.1 note

This document describes the handshake at a normative conceptual level for v0.1. A stricter wire format and interoperability test suites can be defined in later versions.

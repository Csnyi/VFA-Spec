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

## v0.1 note

This document describes the handshake at a normative conceptual level for v0.1.
A stricter wire format and interoperability test suites can be defined in later versions.

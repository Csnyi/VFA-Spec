# Visa Token Format

A visa token represents cryptographic proof that an interaction has been approved.

---

## Terminology

**Intent**

A structured declaration describing a requested action, created by the merchant and approved by the user via the wallet.

Example:

```json
{
  "merchantId": "merchant.example",
  "endpoint": "/payments",
  "amount": "24.90",
  "currency": "EUR"
}
```

**Visa Token**

A cryptographically signed artifact issued by the VFA issuer that binds identity, intent, and policy scope. The visa token is the normative artifact in VFA v0.1.

The issuer signs the token — not the wallet. The wallet's role is to collect explicit user approval; the issuer then mints the token based on that approval.

---

## Canonical Fields

The following fields are normative for VFA v0.1 tokens.

| Field | Type | Required | Meaning |
|-------|------|----------|---------|
| `iss` | string | yes | token issuer identifier |
| `sub` | string | yes | user identity |
| `merchantId` | string | yes | initiating merchant identifier |
| `aud` | string | yes | relying gateway or service |
| `scope` | string | yes | operation category (e.g. `payment:init`) |
| `endpoint` | string | yes | specific API endpoint being authorized |
| `nonce` | string | yes | replay protection value; must be unique per token |
| `iat` | number | yes | issued-at timestamp (Unix epoch seconds) |
| `exp` | number | yes | expiration timestamp (Unix epoch seconds) |
| `kid` | string | optional in v0.1, **required in production** | key identifier; enables key rotation without token reissuance |

> `kid` MUST be included in production deployments. The gateway uses it to select the correct issuer public key for signature verification. Without `kid`, key rotation requires coordinated cutover across all gateway nodes.

Additional optional fields MAY be defined by implementations.

---

## Scope Semantics

The `scope` field declares the category of operation that the user explicitly authorizes. The gateway uses this value to enforce policy before dispatching a request to the backend.

In VFA v0.1, the scope value is defined by the service deployment and enforced by the gateway policy configuration.

### Scope format

The recommended format is:

`namespace:action`

Where:

- `namespace` identifies the functional domain or service
- `action` identifies the operation being authorized

Examples:

```
payment:init
payment:capture
payment:refund
account:update
document:sign
```

The `namespace:action` format is **RECOMMENDED** for interoperability and readability. Implementations MAY use alternative strings if they follow local policy conventions, but gateways MUST enforce exact string matching.

### Scope authority

The set of valid scopes is defined by the protected service deployment and configured in the gateway policy layer.

The issuer MUST include the scope in the signed visa token payload. The gateway MUST verify that the scope value is authorized for the target endpoint.

Example policy rule:

| Endpoint | Allowed scope |
|----------|---------------|
| `/payments/init` | `payment:init` |
| `/payments/refund` | `payment:refund` |

If the scope in the visa token does not match the expected scope for the endpoint, the gateway MUST reject the request.

### Scope binding

The scope is part of the signed token payload and therefore protected by the issuer signature.

A scope issued for one operation MUST NOT authorize a different operation, even if other token fields match.

Example:

```
scope: payment:init
endpoint: /payments/init
```

MUST NOT be accepted for:

```
endpoint: /payments/refund
```

### Extensibility

Future versions of VFA may introduce standardized scope namespaces or registry mechanisms. In v0.1, scope definitions remain deployment-local and are enforced by the gateway policy configuration.

---

## Example payload

```json
{
  "iss": "vfa.example",
  "sub": "user123",
  "merchantId": "merchant.example",
  "aud": "gateway.example",
  "scope": "payment:init",
  "endpoint": "/payments",
  "nonce": "abcxyz128bit",
  "iat": 1710000000,
  "exp": 1710000060,
  "kid": "vfa-key-2024-01"
}
```

> Note: `exp - iat` should not exceed 60 seconds in production deployments.
> See the [Time Formats](#time-formats) section below.

---

## Signature model

The payload must be signed by the **issuer** using an asymmetric algorithm.

Recommended algorithms: **Ed25519** or **ECDSA P-256**.

For MVP or laboratory implementations only, a shared-secret HMAC may be used as a temporary measure. This must not be used in production.

PKCS#1 v1.5 and `alg: none` are disallowed in all environments.

---

## Validation expectations

A verifier should check:

- token structure and required fields
- signature validity (asymmetric verification against issuer public key)
- expiration time (`exp`)
- intended audience (`aud`)
- merchant and endpoint binding (`merchantId`, `endpoint`)
- scope correctness
- nonce uniqueness (server-side deduplication store)

---

## Verification Models

VFA supports multiple verification models.

### Local verification (recommended)

Gateway verifies the visa token signature locally using the issuer's public key.

### Introspection verification

Gateway sends the token to the issuer for validation.

### Hybrid

Gateway verifies signature locally and optionally checks revocation or policy via issuer API.

---

## Encoding note

The exact token serialization format is not fixed in v0.1. Implementations may use a compact serialized form, JSON-based form, or another deterministic encoding.

---

## Time Formats

VFA uses two distinct time representations:

**Handshake requests:**

`ttlMs` — token lifetime expressed in **milliseconds**

Example: `"ttlMs": 300000` (= 300 seconds)

**Visa tokens:**

`iat` and `exp` — timestamps expressed in **seconds since Unix epoch**

Example: `"iat": 1710000000, "exp": 1710000060` (= 60-second validity window)

> Production deployments SHOULD enforce a maximum token lifetime of **60 seconds** (`exp - iat ≤ 60`).
> The 300-second window shown in handshake examples is the user-facing approval window,
> not the token validity period.
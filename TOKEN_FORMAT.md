# Visa Token Format

A visa token represents cryptographic proof that an interaction has been approved.

---

## Example payload

```json
{
  "tokenId": "tok-abc123",
  "issuer": "vfa.example",
  "subject": "user123",
  "aud": "merchant.example",
  "scope": "payment:init",
  "iat": 1710000000,
  "exp": 1710000300,
  "nonce": "abcxyz"
}
```

---

## Core fields

### `tokenId`
Unique token identifier.

### `issuer`
Authority or component issuing the token.

### `subject`
Approved user or entity.

### `aud`
Intended audience or relying party.

### `scope`
Approved action, permission, or policy scope.

### `iat`
Issued-at timestamp.

### `exp`
Expiration timestamp.

### `nonce`
Optional freshness value linked to the handshake request.

---

## Signature model

The payload must be signed by the issuer.

For MVP or laboratory implementations, this may use a shared-secret HMAC.

Production deployments should prefer asymmetric cryptography and stronger key management practices.

---

## Validation expectations

A verifier should check:

- token structure
- signature validity
- expiration time
- intended audience
- scope correctness
- replay protections where implemented

---

## Encoding note

The exact token serialization format is not fixed in v0.1.
Implementations may use a compact serialized form, JSON-based form, or another deterministic encoding.

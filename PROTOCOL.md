# VFA Handshake Protocol

The VFA handshake ensures that a user or controlling entity explicitly approves an interaction before it is executed.

---

## Actors

- **Wallet** — user-controlled approval interface
- **Merchant** — requesting application or relying party
- **Issuer / Verification server** — component issuing or validating visa tokens
- **Gateway** — policy enforcement point

---

## High-level flow

1. Merchant creates a handshake request.
2. Wallet receives or scans the request.
3. User accepts or rejects the request.
4. If accepted, an issuer creates a visa token.
5. Merchant presents the visa token with the protected request.
6. Gateway verifies the token and applies policy.

---

## Handshake request example

```json
{
  "requestId": "req-12345",
  "merchant": "merchant.example",
  "scope": "payment:init",
  "ttl": 300000,
  "nonce": "abcxyz"
}
```

### Suggested fields

- `requestId` — unique request identifier
- `merchant` — requesting party identity
- `scope` — requested action or permission set
- `ttl` — lifetime in milliseconds
- `nonce` — freshness value

---

## User decision

The wallet returns one of two decisions:

- **ACCEPT**
- **REJECT**

A rejected request must not produce a visa token.

---

## Result of acceptance

If the request is accepted, the issuer returns a short-lived visa token containing the approved scope and time limits.

---

## Gateway behavior

On receiving a request carrying a visa token, the gateway should:

1. parse the token
2. verify its signature
3. validate expiration and scope
4. check replay and revocation controls if implemented
5. apply routing or allow/deny policy

---

## Possible policy outcomes

- allow request to production backend
- route request to sandbox backend
- deny request entirely
- require re-approval

---

## v0.1 note

This document intentionally describes the handshake at a conceptual level.
A stricter wire format and interoperability rules can be defined in later versions.

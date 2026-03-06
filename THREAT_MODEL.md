# Threat Model

This document lists important threat categories relevant to the VFA architecture.

---

## Token replay

An attacker attempts to reuse a valid token after initial issuance.

### Mitigations

- short token lifetime
- request nonces
- token identifiers
- replay caches
- revocation support

---

## Token forgery

An attacker attempts to create a fake token.

### Mitigations

- cryptographic signatures
- protected signing keys
- deterministic verification rules

---

## Audience confusion

A valid token intended for one relying party is reused against another.

### Mitigations

- audience binding (`aud`)
- strict verifier checks

---

## Scope escalation

A token approved for one action is reused for a broader action.

### Mitigations

- explicit scope definition
- exact scope matching
- narrow token lifetime

---

## Unauthorized direct access

A request bypasses approval logic and attempts to access a protected service directly.

### Mitigations

- mandatory gateway enforcement
- deny-by-default routing
- backend verification where appropriate

---

## Compromised merchant behavior

A requesting application tries to obtain more approval than needed or misrepresents the requested action.

### Mitigations

- clear wallet UX
- explicit scope display
- auditing and policy controls

---

## Key compromise

Signing or verification material is exposed.

### Mitigations

- secure key storage
- key rotation
- key separation between environments
- incident response procedures

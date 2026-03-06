# Security Model

VFA assumes that protected services require proof of explicit consent or policy approval before certain interactions are allowed.

---

## Primary security objectives

- ensure that requests can be linked to explicit approval
- prevent trivial token tampering
- reduce replay risk using short-lived artifacts
- enable policy enforcement before backend access

---

## Trust boundaries

The architecture typically contains the following trust boundaries:

- **Wallet boundary** — user-controlled approval environment
- **Merchant boundary** — requesting application or service
- **Issuer boundary** — authority creating proof artifacts
- **Gateway boundary** — enforcement and routing point
- **Protected service boundary** — sensitive application backend

---

## Assumptions

- the wallet is trusted by the user to present the approval request correctly
- the issuer protects signing material appropriately for the chosen deployment model
- the gateway enforces verification before routing sensitive requests
- tokens are short-lived

---

## Production expectations

Production deployments should implement:

- secure key storage
- key rotation
- audience binding
- replay protection
- scoped authorization
- logging and audit controls
- rate limiting and abuse controls

---

## Demo / lab note

A shared-secret HMAC scheme may be acceptable in a demonstration environment, but it should not be treated as a production-grade security architecture.

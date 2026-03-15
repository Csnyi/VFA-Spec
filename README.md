# VFA-Spec

Virtual Flow Agreement — Protocol Specification

Method and System for Cryptographically Verified Multi-Entity Intent Handshake and Delegated Execution of Digital Transactions

This repository contains the **protocol specification** for the Virtual Flow Agreement (VFA) concept.

VFA defines a cryptographic intent verification layer for digital interactions, enabling applications and gateways to verify user consent and policy compliance before allowing access to protected services.

The specification describes:

- the handshake protocol
- the visa token format
- the verification and routing flow
- the security model
- the threat model

---

## Trust Model (v0.1)

VFA v0.1 uses an **issuer-signed visa token model**.

Roles:

- **Wallet** — collects explicit user approval for an action
- **Issuer** — mints a short-lived visa token based on the approved intent
- **Merchant** — presents the visa token when calling the protected service
- **Gateway** — verifies the visa token and enforces policy
- **Backend** — executes only verified requests

Flow summary:

1. Merchant creates a handshake request
2. User approves the request in the wallet
3. Issuer generates a **visa token**
4. Merchant sends request + visa token
5. Gateway verifies token and enforces policy
6. Backend executes the request

The **visa token** is the cryptographic artifact binding:

- identity
- declared intent
- policy scope
- expiration

> The wallet-signed intent model (where the user's wallet directly signs the
> artifact rather than an issuer) is considered a future extension and is not
> normative in v0.1.

---

## Related repositories

Implementation and demonstration projects:

- **VFA-MVP** — reference implementation of the handshake flow → https://github.com/Csnyi/VFA-MVP
- **VFA-Lab** — architecture sandbox and gateway routing demo → https://github.com/Csnyi/VFA-Lab
- **VFA-cloud-PoC** — cloud operation PoC (deployment scenario) → https://github.com/Csnyi/VFA-cloud-PoC
- **VFA-Spec** - this repository: protocol specification

### VFA-MVP protocol notes (v0.1)

VFA-MVP implements the handshake flow defined in this specification with the following
characteristics relevant to v0.1:

- **Issuer role** is fulfilled by the Flask backend (`backend/server.py`)
- **Token format**: `base64url(JSON payload) + "." + base64url(HMAC-SHA256(secret, payload_b64))`  
  This is a lab-only format; production deployments MUST use asymmetric signatures (Ed25519 / ECDSA P-256)
- **Approval window** (`ttlSec`) and token lifetime (`exp`) are tracked separately in SQLite
- **Nonce / replay protection** is not implemented in the MVP — listed as a known limitation
- **Scope** is represented as a JSON list (e.g. `["age_over_18", "loyalty_id"]`) rather than
  the recommended `namespace:action` string; a `scopeHash` (SHA-256) is stored in the visa record

### VFA-Lab protocol notes (v0.1)

VFA-Lab implements the gateway and policy enforcement layer defined in this specification:

- **Policy outcomes**: `prod` (valid token), `sandbox` (missing / invalid token), `deny` (by policy config)
- **Token verification** uses HMAC shared secret (lab only); asymmetric verification is the declared next step
- **Revocation** is implemented as a local JSON store at the gateway; distributed revocation is a known gap
- **DEFAULT_ROUTE** environment variable controls the unauthenticated routing policy
- The gateway demonstrates the **L3.5 conceptual overlay** — policy enforcement logically between IP routing
  and application processing; see [docs/FUTURE.md](docs/FUTURE.md) for the extended vision

### VFA-cloud-PoC protocol notes (v0.1)

VFA-cloud-PoC demonstrates the VFA handshake applied to cloud operations where critical actions must be verified before execution.

Key characteristics:

- **Use case**: verification of user intent before sensitive operations (e.g. deployment or production actions)
- **Actor mapping**:
  - Wallet → user approval interface
  - Issuer → intent verification service
  - Gateway → policy enforcement before the operation
  - Backend → cloud operation executor
- **Intent binding**: the operation request references the verified intent artifact before execution
- **Execution model**: backend must execute the operation corresponding to the committed intent
- **Threat focus**: prevention of *execution mismatch* where an executed action differs from the user-approved intent

This PoC illustrates how the VFA mechanism can be applied to operational workflows such as CI/CD pipelines and production deployment controls.

---

## Specification documents

| Document | Description |
|----------|-------------|
| [SPEC.md](SPEC.md) | High-level overview of the VFA concept |
| [PROTOCOL.md](PROTOCOL.md) | Handshake and interaction protocol |
| [TOKEN_FORMAT.md](TOKEN_FORMAT.md) | Visa token structure and fields |
| [SECURITY_MODEL.md](SECURITY_MODEL.md) | Security assumptions and design |
| [THREAT_MODEL.md](THREAT_MODEL.md) | Threat analysis |
| [GLOSSARY.md](GLOSSARY.md) | Authoritative terminology definitions |
| [CHANGELOG.md](CHANGELOG.md) | Version history |
| [docs/FUTURE.md](docs/FUTURE.md) | Post-v0.1 vision: L3.5, wallet-signed intent, delegation |

---

## Repository layout

```text
VFA-Spec
├─ README.md
├─ SPEC.md
├─ PROTOCOL.md
├─ TOKEN_FORMAT.md
├─ SECURITY_MODEL.md
├─ THREAT_MODEL.md
├─ GLOSSARY.md
├─ CHANGELOG.md
├─ PATENTS
├─ LICENSE
├─ diagrams/
├─ examples/
└─ docs/
   └─ FUTURE.md
```

---

## Specification License

The VFA specification is released under the Apache 2.0 license.

This allows independent implementations of the protocol.

---

## Intellectual Property

The VFA concept may be subject to patent applications.

The specification in this repository is published to support
open research and interoperable implementations.

See the [PATENTS](PATENTS.md) file for additional information.

---

## Status

Draft specification.

Version: **v0.1**

This repository is experimental and may evolve as the VFA architecture matures.

---

## Delegation

Delegated authorization is considered a future extension.

Potential fields include:

- `delegator`
- `delegate`
- `delegationScope`
- `delegationExp`

Delegation is not normative in VFA v0.1.

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

---

## Specification documents

| Document | Description |
|----------|-------------|
| [SPEC.md](SPEC.md) | High-level overview of the VFA concept |
| [PROTOCOL.md](PROTOCOL.md) | Handshake and interaction protocol |
| [TOKEN_FORMAT.md](TOKEN_FORMAT.md) | Visa token structure and fields |
| [SECURITY_MODEL.md](SECURITY_MODEL.md) | Security assumptions and design |
| [THREAT_MODEL.md](THREAT_MODEL.md) | Threat analysis |
| [CHANGELOG.md](CHANGELOG.md) | Version history |

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
├─ CHANGELOG.md
├─ PATENTS
├─ LICENSE
├─ diagrams/
└─ examples/
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

See the [PATENTS](PATENTS) file for additional information.

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

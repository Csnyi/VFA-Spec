# VFA Future Vision — L3.5 Policy Overlay

This document describes architectural directions beyond VFA v0.1.

The concepts here are exploratory and not normative for the current specification.

---

## Motivation

VFA v0.1 defines an application-layer verification protocol: a gateway
checks a visa token embedded in an HTTP request and makes a routing decision
before dispatching to the backend.

This is useful, but it places the enforcement point inside the application
stack — after TLS termination, after TCP connection establishment, after the
HTTP request is already formed.

A natural question arises: can policy enforcement move earlier in the
communication process, closer to connection establishment itself?

---

## The L3.5 Concept

VFA's policy layer is sometimes described as operating at **"L3.5"** — a
conceptual designation, not an IEEE standard.

The idea is that a VFA policy overlay sits logically between the routing
layer (L3 / IP) and the transport layer (L4 / TCP) of the communication
stack, influencing decisions before application data is exchanged.

```
L2:  Link (Ethernet / Wi-Fi)
L3:  IP routing
     ↕
L3.5: VFA Policy Overlay  ← decision plane (optional)
     ↕
L4:  TCP
L5:  TLS
L7:  Application (HTTP / API)
```

In practice, L3.5 is not a literal network layer. It describes a
**decision plane** that can:

- be evaluated at TLS handshake time (before any application data)
- influence connection routing before the HTTP request is processed
- be implemented at a network edge, load balancer, or sidecar level

The L3.5 label captures the intent: policy enforcement that is
**earlier than application logic** but **later than IP routing**.

---

## TLS-Layer Integration (Vision)

One direction for L3.5 is integrating visa token verification into the
TLS handshake itself.

Conceptual sketch:

```
Client                          VFA Gateway
  |                                  |
  |---  ClientHello                  |
  |     + vfa_token extension  ----> |
  |                                  |  verify token signature
  |                                  |  evaluate policy
  |                                  |
  | <-- ServerHello                  |
  |     + vfa_accepted / vfa_denied  |
  |                                  |
  |  (if accepted: route to PROD)    |
  |  (if denied: route to SANDBOX)   |
```

This would allow the gateway to make a routing decision before the HTTP
request is transmitted — reducing the attack surface and enabling
zero-trust enforcement at connection establishment time.

This extension concept is **not defined** in VFA v0.1 and would require
a dedicated TLS extension specification.

---

## System Architecture at L3.5 Scale

A full L3.5 deployment would operate across two planes:

### Policy Plane

The policy gateway makes routing decisions based on visa token validity.

```
User / Client
     |
     | HTTPS request (optional: Bearer visaToken)
     ↓
Policy Gateway  ←→  Revocation Store
     |
     ├─ valid token  →  Merchant API (PROD)
     └─ invalid / missing  →  Sandbox API
```

### Identity Plane

A separate identity plane handles wallet operations and token issuance.

```
Wallet UI / Wallet Service
     |
     | issue visaToken
     ↓
   User / Client
     |
     ↓
Policy Gateway  →  Token Verify Logic
```

These two planes are logically separate but coordinated: the identity
plane issues tokens, the policy plane enforces them.

---

## Wallet-Signed Intent Model

VFA v0.1 uses an **issuer-signed** model: the issuer (server-side) mints
the visa token after the wallet collects user approval.

A future extension under consideration is the **wallet-signed intent model**:
the user's wallet directly signs the intent artifact, and the gateway
verifies the wallet's signature rather than (or in addition to) an issuer
signature.

This would enable:

- offline or decentralized issuance
- hardware-backed signing keys on the user device
- cryptographic proof of intent that does not require trusting a central issuer

This model is not normative in VFA v0.1 and would require:

- a standardized wallet key format and registration mechanism
- gateway-side wallet key resolution (analogous to JWKS for issuers)
- replay protection without a central nonce store

---

## Delegation

Delegated authorization is another future extension: a token holder
delegating a subset of their authorization to another party.

Potential fields for a delegation extension:

| Field | Meaning |
|-------|---------|
| `delegator` | identity of the original authorizing party |
| `delegate` | identity of the authorized delegate |
| `delegationScope` | subset of the original scope being delegated |
| `delegationExp` | expiration of the delegation grant |

Delegation is not normative in VFA v0.1.

---

## Standardized Scope Registry

VFA v0.1 uses deployment-local scope definitions. A future version may
introduce a standardized scope namespace registry, enabling interoperability
across independent implementations.

---

## Formal Interoperability Test Suites

VFA v0.1 does not define wire-level interoperability tests. A future
version could introduce:

- canonical test vectors for token generation and verification
- a compliance test suite for gateway implementations
- a reference implementation contract

---

## Summary of Future Extensions

| Topic | Status |
|-------|--------|
| L3.5 TLS handshake integration | exploratory concept |
| Wallet-signed intent model | considered future extension |
| Delegation | considered future extension |
| Standardized scope registry | considered future extension |
| Formal interoperability test suites | deferred to post-v0.1 |
| Mandatory cryptographic algorithm suite | deferred to post-v0.1 |
| Production governance model | deferred to post-v0.1 |

---

*This document is non-normative. None of the concepts described here are
part of the VFA v0.1 specification.*

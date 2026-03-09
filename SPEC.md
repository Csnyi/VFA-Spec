# VFA Specification Overview

Virtual Flow Agreement (VFA) is a framework for verifying **intent and authorization** in digital interactions.

The goal of VFA is to ensure that a request reaching a protected service has been **explicitly approved by the user or controlling entity**.

Traditional authentication verifies **identity**.

VFA additionally verifies **intent**.

---

## Core idea

A request should carry cryptographic proof that the interaction was explicitly approved.

This proof is represented by a **visa token**.

The visa token can then be verified by a gateway or policy enforcement layer before a request is allowed to reach a sensitive backend.

---

## Architectural concept

Typical VFA-style interaction:

```text
Wallet → Merchant → Gateway → Protected Service
```

The gateway verifies the visa token and makes a routing or access decision based on policy.

---

## What VFA adds

VFA introduces a policy-oriented verification step between request creation and service access.

This layer may evaluate:

- explicit user consent
- token validity
- token scope
- route selection policy
- sandbox vs production destination

---

## Conceptual Policy Layer

The VFA policy layer is a **conceptual admission layer** responsible for verifying declared intent before protected resources are accessed.

This layer is not a literal OSI layer.

In current implementations (v0.1), verification occurs at the gateway after TLS termination, as part of the request handling pipeline — logically between transport and application processing.

---

## Design goals

- explicit intent verification
- cryptographic proof of approval
- short-lived authorization artifacts
- policy-driven routing
- separation between protocol definition and implementation

---

## Non-goals for v0.1

This draft does not attempt to define:

- a final wire-level transport standard
- a mandatory cryptographic algorithm for all deployments
- a production-ready governance model
- formal interoperability test suites

These can be added in later versions.

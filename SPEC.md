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

## v0.1 Implementation Summary

Two reference implementations exist for VFA v0.1.

### VFA-MVP — Handshake reference implementation

VFA-MVP demonstrates the full handshake flow: merchant request → wallet approval →
visa token issuance → merchant verification.

Key characteristics:

- **Wallet**: browser UI; scans merchant QR, sends ACCEPT/REJECT to server
- **Issuer (server)**: Flask + SQLite; mints HMAC-SHA256 tokens (lab only)
- **Token format**: `base64url(payload) + "." + base64url(HMAC-SHA256(secret, payload_b64))`
- **Scope**: JSON list per request (e.g. `["age_over_18", "loyalty_id"]`)
- **Known gaps**: no replay protection, no asymmetric signing, CORS open for local dev

### VFA-Lab — Gateway and policy routing demo

VFA-Lab demonstrates the gateway policy enforcement layer.

Key characteristics:

- **Gateway**: Node.js reverse proxy; verifies HMAC token, reads revocation store
- **Policy outcomes**: `prod` (valid token) / `sandbox` (missing or invalid) / `deny` (by config)
- **Revocation**: local JSON store; distributed revocation is a declared next step
- **L3.5 concept**: the gateway acts as a policy decision plane logically between
  IP routing and application processing

### VFA-cloud-PoC — Cloud operation intent verification

VFA-cloud-PoC demonstrates how the VFA model applies to real-world
operational workflows, particularly in cloud and deployment environments.

Key characteristics:

- **Use case**: verification of user intent before executing sensitive operations such as production deployments or infrastructure changes
- **Actor mapping**:
  - Wallet → user approval interface (human-in-the-loop confirmation)
  - Issuer → intent verification service
  - Gateway → policy enforcement layer before execution
  - Backend → cloud operation executor (e.g. CI/CD pipeline)
- **Intent binding**: the operation request is cryptographically bound to a verified intent artifact before execution
- **Execution guarantee model**: the backend must execute only the action corresponding to the verified intent, without reinterpretation
- **Threat focus**: mitigation of *execution mismatch* — where the executed action differs from the user-approved intent

This PoC extends the VFA concept beyond API access control into **operational security**, demonstrating how intent verification can be used to protect high-impact actions in distributed systems.

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

For exploratory directions including L3.5 TLS integration, wallet-signed
intent model, and delegation, see [docs/FUTURE.md](docs/FUTURE.md).

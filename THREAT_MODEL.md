# Threat Model — Cryptographic Intent Verification System

This document covers threat categories relevant to the VFA four-party intent verification architecture: **user wallet → merchant frontend → issuer → API gateway → backend services**.

In VFA v0.1, the **issuer** signs and mints visa tokens based on explicit user approval collected by the wallet. The merchant frontend is treated as a potentially hostile intermediary.

> **Terminology note:** This document uses "intent" to refer to the user-approved action declaration, and "visa token" for the issuer-signed cryptographic artifact. These are distinct concepts.

---

## T-01: Token Replay

Threat: An attacker reuses a previously issued, valid visa token.

### Variants
- **Network replay** — captured token re-sent from a MITM position
- **Merchant-driven replay** — merchant stores and re-submits a token later
- **Cross-context replay** — token reused against a different merchant or endpoint

### Mitigations
- Short token lifetime (**≤ 60 s**), enforced via `exp` field server-side
- 128-bit CSPRNG nonce per token; server-side nonce deduplication store
- Context binding: `merchantId` + `endpoint` + `sub` included in the signed payload
- Idempotency keys on the API

---

## T-02: Token Forgery

Threat: An attacker constructs a fake visa token or modifies a signed one.

### Mitigations
- Ed25519 or ECDSA P-256 signatures; PKCS#1 v1.5 and `alg: none` disallowed
- Canonical serialization (JCS / RFC 8785) before signing
- All security-relevant fields covered by the signature
- Algorithm pinning: `alg` field included in the signed payload

---

## Audience and Context Confusion

A valid intent is used in a context other than the one it was issued for.

### Variants
- **Audience confusion** — intent accepted by a different relying party (`aud` mismatch)
- **Context drift** — intent replayed under a different `merchantId` or endpoint
- **Session injection** — valid intent injected into a different user session

### Mitigations
- Strict `aud` / `merchantId` / `endpoint` binding in the signed payload
- Gateway enforces exact context match before execution
- Session anomaly detection: session-user-intent correlation checked at runtime

---

## Scope Escalation

A token or intent approved for one action is reused for a broader or different action.

### Mitigations
- Explicit, narrow scope per intent; exact scope matching at verification
- Short token lifetime; scope included in the signed payload
- Aggregated risk scoring: cumulative scope across a session, not only per-intent

---

## Downgrade Attack

An attacker forces a weaker cryptographic protocol or algorithm version.

### Variants
- TLS downgrade (TLS 1.3 → legacy)
- Signature algorithm downgrade (e.g. P-521 → P-256, or to PKCS#1 v1.5)
- API version downgrade (to a version lacking security controls)

### Mitigations
- TLS 1.3 only; HSTS with `includeSubDomains`; `TLS_FALLBACK_SCSV`
- Algorithm allowlist enforced by the gateway; no negotiation of weaker algorithms
- Old API versions deactivated; minimum supported version enforced

---

## Impersonation

An attacker poses as a legitimate party (merchant, gateway, or user).

### Variants
- **Fake merchant frontend** — phishing site tricks user into signing a malicious intent
- **Gateway impersonation** — DNS/BGP hijack redirects signed intents to a rogue endpoint
- **User impersonation** — stolen credentials used to authenticate as a legitimate user

### Mitigations
- PKI-based merchant certificates; signed `merchantId` verified by the wallet
- Mutual TLS (mTLS) on all service-to-service channels; endpoint pinning in the wallet
- Hardware MFA (FIDO2 / WebAuthn); wallet presence proof required per intent

---

## Compromised Merchant Behavior

The merchant frontend actively manipulates the intent to the user's detriment.

### Variants
- **UI spoofing** — user sees different intent than what is signed (visual mismatch)
- **Parameter injection** — extra or modified fields appended after signing
- **Visual fatigue manipulation** — critical fields hidden below the fold or in fine print
- **Multiple submission** — single approval used to trigger multiple API calls

### Mitigations
- Wallet renders intent summary independently from structured schema; no free-text from merchant in approval UI
- Signature covers all critical fields; gateway rejects any unsigned additions
- Nonce: one intent = one execution; server-side deduplication

---

## T-08: Execution Mismatch

Threat: The backend executes an action different from the signed and verified intent — the gap between gateway verification and actual backend dispatch.

### Mitigations
- Intent commitment: gateway commits to the `intentHash` (a hash of the verified visa token payload) before dispatching to the backend; backend executes only against the committed intent
- Execution receipt returned by the backend and validated by the gateway against the original `intentHash`; gateway rejects any response not bound to the committed intent
- Execution result may be independently verified by the wallet against the approved intent summary (future extension — not normative in v0.1)
- Gateway verification performed immediately before dispatch (not cached from an earlier step)

> Note: `intentHash` is a gateway-internal binding mechanism; it is not a token field in v0.1.

---

## Approval Fatigue

Users reflexively approve requests without reading them after repeated prompts.

### Mitigations
- Rate limit on approvals (e.g. max 5 per minute per session)
- High-risk intents require explicit re-confirmation with highlighted risk fields
- Aggregated risk threshold: daily/session cumulative limits, not only per-intent
- Wallet displays running count: *"3rd approval in the last 5 minutes"*

---

## Policy Bypass and Lateral Movement

A compromised or malicious backend calls internal services directly, bypassing the gateway's intent verification layer.

### Mitigations
- Zero-trust network segmentation: internal APIs reachable only via the gateway service account
- All internal calls authenticated with mTLS; unsigned or gateway-bypass requests rejected and alerted
- Policy-as-Code: gateway configuration version-controlled, peer-reviewed, with change alerting
- Immutable infrastructure principles for gateway deployments

---

## Intent Laundering

Individual intents appear legitimate in isolation but form part of a coordinated attack chain (e.g. `approve upload` → `approve permission change` → `approve export`).

### Mitigations
- Workflow-level audit: intent sequences analysed, not only individual intents
- Anomaly detection on unusual action combinations within a time window
- Intent scope restricted per merchant context (limits cross-context chaining)
- Context binding (see above) constrains which intent types are valid per merchant

---

## Unauthorized Direct Access

A request bypasses approval logic and reaches a protected service directly.

### Mitigations
- Mandatory gateway enforcement; deny-by-default routing
- Backend verifies every request originates from the gateway (mTLS service identity)
- No direct external exposure of internal service endpoints

---

## Key Compromise

Signing or verification key material is exposed.

### Mitigations
- Hardware-backed key storage (HSM / TEE) in the wallet; private key never exported
- Key rotation with bounded validity period (≤ 1 year)
- Real-time revocation propagation to all gateway nodes (≤ 30 s)
- Key separation across environments (dev / staging / prod)
- Incident response procedure defined and tested

---

## Revocation Failures

Revoked tokens, intents, or keys continue to be accepted.

### Mitigations
- OCSP stapling / real-time key-status API at every gateway node
- Short token lifetime as primary defence; revocation as secondary layer
- Intent revocation endpoint; nonce store used to invalidate outstanding intents
- Revocation events propagated immediately via pub/sub

---

## Delegation Abuse

Delegated execution rights are broader, longer-lived, or more transferable than intended.

### Mitigations
- Least-privilege delegation: explicit scope, action list, and TTL in the signed payload
- Maximum delegation depth: 1 level; re-delegation disallowed
- Sender-constrained delegated tokens (DPoP)
- Anomaly detection on delegated token usage patterns

---

## Audit Integrity Failures

Execution logs are tampered with or incomplete, preventing accountability.

### Mitigations
- Cryptographically chained (hash-chain), append-only audit log
- WORM storage for audit data; independent external audit server with real-time mirroring
- Sequence numbers on intents; gap detection for missing entries
- Structured mandatory logging on all API paths, including exception handlers

---

## AI Agent Interactions

Autonomous AI agents request intent approvals on behalf of users, introducing new delegation and attestation risks.

### Variants
- **Autonomous over-reach** — agent generates intents the user did not intend
- **Prompt injection** — malicious content in processed data (e.g. uploaded file) instructs the agent to generate unintended intents without user awareness
- **Intent spam / delegation chain** — agent generates high-volume or cascading intent requests

### Mitigations
- Agent identity attestation: cryptographic proof that agent acts under explicit user delegation
- Human-in-the-loop thresholds by risk level; high-risk intents always require manual approval
- Structured intent schema required; free-text → intent conversion via audited intermediate step
- Agent-generated intents visually distinguished in wallet UI (source labelled)
- Rate limiting on agent-initiated intents; sandboxed processing of external content
- Explicit delegation policy: agent scope, TTL, and permitted action list defined at delegation time

---

## Threat Summary

| ID   | Category                                          | Severity        |
|------|---------------------------------------------------|-----------------|
| T-01 | Token Replay (network / merchant / cross-context) | Critical        |
| T-02 | Token Forgery                                     | Critical        |
| T-03 | Audience and Context Confusion                    | High            |
| T-04 | Scope Escalation                                  | High            |
| T-05 | Downgrade Attack                                  | High            |
| T-06 | Impersonation (merchant / gateway / user)         | Critical        |
| T-07 | Compromised Merchant Behavior                     | Critical        |
| T-08 | Execution Mismatch                                | Critical        |
| T-09 | Approval Fatigue                                  | High            |
| T-10 | Policy Bypass and Lateral Movement                | Critical        |
| T-11 | Intent Laundering                                 | High            |
| T-12 | Unauthorized Direct Access                        | High            |
| T-13 | Key Compromise                                    | Critical        |
| T-14 | Revocation Failures                               | High            |
| T-15 | Delegation Abuse                                  | High            |
| T-16 | Audit Integrity Failures                          | High            |
| T-17 | AI Agent Interactions                             | High / Critical |
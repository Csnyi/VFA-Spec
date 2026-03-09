# Changelog

All notable changes to this project will be documented in this file.

The format is based on Keep a Changelog, and this project follows Semantic Versioning in spirit for draft releases.

## [Unreleased]

### Added
- `README.md` — Trust Model (v0.1) block: normative roles and 6-step flow summary
- `README.md` — Delegation section as explicit future-extension placeholder
- `TOKEN_FORMAT.md` — Terminology section: Intent vs Visa Token distinction
- `TOKEN_FORMAT.md` — Canonical Fields table (normative for v0.1)
- `TOKEN_FORMAT.md` — Verification Models section (local / introspection / hybrid)
- `TOKEN_FORMAT.md` — Time Formats section: `ttlMs` (ms) vs `iat`/`exp` (Unix seconds)
- `TOKEN_FORMAT.md` — `kid` field added: optional in v0.1, required in production
- `PROTOCOL.md` — Backend added as explicit actor
- `PROTOCOL.md` — Normative step-by-step Protocol Flow (v0.1) with HTTP example
- `PROTOCOL.md` — Handshake request fields table
- `SECURITY_MODEL.md` — Core artifact section (Visa Token definition)
- `SECURITY_MODEL.md` — Trust boundaries table with trust level and key assumption per actor
- `SECURITY_MODEL.md` — User added as root trust anchor
- `SECURITY_MODEL.md` — Assumptions section structured per actor
- `GLOSSARY.md` — new file: single authoritative terminology source covering all actors, core artifacts, key concepts, normative language (RFC 2119), and deprecated term mapping

### Changed
- `TOKEN_FORMAT.md` — `nonce` is now **required** (was optional); `exp` capped at 60 s in production
- `TOKEN_FORMAT.md` — Canonical field names aligned: `iss`, `sub`, `merchantId`, `endpoint` added; `tokenId`/`issuer`/`subject` removed
- `TOKEN_FORMAT.md` — Signature model updated: asymmetric algorithms required; HMAC limited to lab-only
- `TOKEN_FORMAT.md` — Example payload updated: `kid` field added
- `PROTOCOL.md` — `merchant` field renamed to `merchantId`; `ttl` renamed to `ttlMs`; `endpoint` added
- `SECURITY_MODEL.md` — Full rewrite: aligned with issuer-signed model; merchant hostile intermediary role explicit; production expectations split into Required / Recommended
- `THREAT_MODEL.md` — Header paragraph corrected: issuer (not wallet) signs tokens in v0.1
- `THREAT_MODEL.md` — "Token / Intent Replay" → T-01 Token Replay; "Token / Intent Forgery" → T-02 Token Forgery
- `THREAT_MODEL.md` — T-08 Execution Mismatch: removed wallet-signs-intentHash claim; `intentHash` scoped as gateway-internal
- `THREAT_MODEL.md` — Threat Summary table updated with T-0x IDs
- `SPEC.md` — Policy layer section renamed and reworded; OSI layer disclaimer clarified
- `diagrams/signing_flow.mmd` — Marked as future extension (not normative for v0.1)
- `examples/visa_token_example.json` — Updated to canonical field names and 60 s expiry
- `examples/handshake_request_example.json` — Updated to `merchantId`, `ttlMs`, `endpoint`

## [0.1.0] - Initial draft

### Added
- Initial VFA-Spec repository structure
- Overview specification document
- Handshake protocol draft
- Visa token format draft
- Security model draft
- Threat model draft
- Mermaid diagram scaffolds
- Example JSON artifacts

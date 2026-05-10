# Stoa RFCs

This directory holds the normative RFCs for the Stoa specification.

## Process

1. **Draft** — author submits a PR with the RFC + rationale + at least one reference implementation (or a credible plan to build one). File name format: `RFC-NNNN-short-title.md`.
2. **Review** — minimum **14 days** public comment period. Editor + community engage in PR comments and a paired GitHub Discussion.
3. **Implementation** — at least one Stoa Edge implementation passes a conformance test for the change.
4. **Editor sign-off** — Vext (editor of v0.1) merges if no unresolved blocking objections. After foundation handoff: TSC majority.

See [CONTRIBUTING.md](../CONTRIBUTING.md) for the full process and [GOVERNANCE.md](../GOVERNANCE.md) for editorial veto limits and anti-capture commitments.

## Index

| RFC | Title | Status | Implementations |
|---|---|---|---|
| [0001](RFC-0001-stoa-1-wire-envelope.md) | Stoa/1 Wire Envelope | Draft (2026-05-10) | [`stoa-edge`](https://github.com/Vext-Labs-Inc/stoa-edge), [`stoa-sdk`](https://github.com/Vext-Labs-Inc/stoa-sdk) |

Future RFCs (draft pipeline):

- RFC-0002: Capability Manifest format
- RFC-0003: Discovery (`/.well-known/stoa.json`)
- RFC-0004: Conformance levels (L0–L4)
- RFC-0005: Foundation Merkle anchoring
- RFC-0006: Hive issuer attestation chain
- RFC-0007: State-delta bus (SSE + WebPush + Bloom prefix)
- RFC-0008: Privacy-class taxonomy
- RFC-0009: Settlement rails (Stripe, x402, ACH, prepaid)
- RFC-0010: Offline-first bundle format

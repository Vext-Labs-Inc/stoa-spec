# RFC-0001: Stoa/1 Wire Envelope

| Field | Value |
|---|---|
| RFC | 0001 |
| Title | Stoa/1 Wire Envelope |
| Author | Vext Labs (editor of v0.1) |
| Status | **Draft** (open public review) |
| Created | 2026-05-10 |
| Updated | 2026-05-10 |
| Comment period | 14 days from publication |
| Implementations | [`stoa-edge`](https://github.com/Vext-Labs-Inc/stoa-edge), [`stoa-sdk`](https://github.com/Vext-Labs-Inc/stoa-sdk) |
| Supersedes | — |
| License | CC-BY-4.0 |

---

## Abstract

This RFC specifies **Stoa/1**, the on-the-wire request/response envelope used by Stoa-conformant SaaS endpoints and the agents that call them. Stoa/1 sits above MCP and OpenAPI 3.1; it adds eight primitives those layers do not provide: signed receipts, state deltas, cost reconciliation, side-effect manifests, idempotency declarations, hive-issued agent identity, privacy-class enforcement, and saga compensation references. Stoa/1 is the mandatory wire format for L1+ conformance.

This RFC is the first normative specification published under the Stoa governance process. The shapes here are the contract every other repo (`stoa-edge`, `stoa-graph`, `stoa-identity`, `stoa-bus`, `stoa-sdk`, `stoa-conformance`) implements.

---

## 1. Motivation

MCP gives an agent a tool name and a JSON Schema input. That is it. There is no protocol-level mechanism for:

- knowing whether a call is idempotent (so agents know whether retry is safe),
- proving the call happened (so audit trails exist),
- knowing what the call cost (so budgets can be enforced),
- knowing what side effects are now in flight (so compensation can run),
- knowing the agent identity is real (so vendors can trust the caller),
- knowing what privacy class data flows through (so PHI doesn't escape HIPAA-zone vendors),
- subscribing to subsequent state changes (so plans can recompose without polling).

Today every implementation reinvents these primitives ad-hoc, prose-encoded into tool descriptions or webhook contracts. Agents must read English documentation to make safe retry decisions. Compliance teams cannot ingest agent activity into existing SIEM. The market has been waiting since the November 2024 launch of MCP for the upper-stack Anthropic refused to ship.

Stoa/1 ships it.

---

## 2. Terminology

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174).

- **Capability** — a named, typed operation a vendor exposes. Identified by URN of the form `urn:stoa:cap:<vendor>.<domain>.<op>@<semver>`.
- **Resource** — a stable entity addressed by URN of the form `urn:stoa:res:<vendor>.<type>:<id>`.
- **Plan** — an ordered (possibly DAG-shaped) declaration of capability calls executed as a single saga.
- **Hive** — the issuing authority for an agent's JWT. A hive is identified by a `did:web` document.
- **Receipt** — a signed JWS produced by the vendor (and optionally co-signed by the agent) attesting to a capability execution.
- **Foundation root** — the daily Merkle root anchoring all receipts produced across the network in the prior 24h.

---

## 3. Request envelope

A Stoa/1 request envelope is a UTF-8 JSON object posted to the vendor's capability endpoint. The content type **MUST** be `application/json` or `application/stoa+json` (the latter is preferred and signals Stoa/1 explicitly).

### 3.1 Shape

```jsonc
{
  "stoa": "1",                                    // REQUIRED
  "cap": "urn:stoa:cap:<vendor>.<domain>.<op>@<semver>",  // REQUIRED
  "idem": "<idempotency-key>",                    // REQUIRED for create/update/action; OPTIONAL for read
  "agent": {                                      // REQUIRED
    "jwt": "<JWS compact>",                       // REQUIRED
    "issuer": "did:web:<hive>",                   // REQUIRED (mirrors JWT.iss for routing convenience)
    "reputation_hint": "<issuer>:<tier>"          // OPTIONAL
  },
  "trace": {                                      // OPTIONAL
    "parent": "<W3C trace context>",
    "plan": "<plan_id>",
    "step": <integer>
  },
  "budget": {                                     // OPTIONAL
    "ceiling_cents": <integer>,                   // hard kill at this amount
    "currency": "USD",                            // ISO 4217
    "settlement": "<rail>:<reference>"            // see §3.2
  },
  "privacy": {                                    // REQUIRED if any input field is PII/PHI/FINANCIAL
    "input_classes": ["<class>", ...],
    "output_classes": ["<class>", ...],
    "jurisdiction": "<ISO 3166-1 alpha-2 or region code>"
  },
  "resume": "<continuation-token>" | null,        // OPTIONAL; for long-running ops
  "input": { ... },                               // REQUIRED; validates against capability input schema
  "compensation": {                               // OPTIONAL; overrides graph-declared default
    "on_undo": "urn:stoa:cap:<...>",
    "key_path": "$.<jsonpath>"
  },
  "policy": {                                     // OPTIONAL
    "require_human_confirmation": <bool>,
    "max_retry": <integer>,
    "preferred_region": "<region code>"
  }
}
```

### 3.2 Settlement rails

The `budget.settlement` field's value is `<rail>:<reference>`. Vendors **MUST** support at least one of:

| Rail | Reference shape | Description |
|---|---|---|
| `stripe` | `cust_<id>` | Stripe customer-of-record monthly invoicing |
| `stripe-connect` | `acct_<id>` | Per-vendor Connect account |
| `x402-escrow` | `0x<hex>` | EVM agent-escrow contract (Base, Polygon, etc.) |
| `ach-net30` | `<vendor-defined>` | Traditional B2B invoice |
| `prepaid` | `stoa-credits:<account>` | Stoa Foundation float |
| `vendor-invoice` | `<vendor-defined>` | Vendor-managed invoice |

A vendor that accepts no rails **MUST** declare `auth.kinds: ["none"]` in its discovery doc (i.e., the capability is free).

### 3.3 Idempotency requirements (L2+)

For capabilities with `side_effects.kind` ∈ `{create, update, action}`:

- The vendor **MUST** honor `idem` as an idempotency key.
- The vendor **MUST** persist `(idem) → (receipt)` for at least 24 hours.
- If the same `idem` arrives with the same input hash, the vendor **MUST** return the previously-issued receipt.
- If the same `idem` arrives with a *different* input hash, the vendor **MUST** return `idempotency_conflict`.

For `read`/`query` capabilities, `idem` is **OPTIONAL** but **RECOMMENDED** for caching.

### 3.4 Privacy classes

The `privacy.input_classes` and `output_classes` fields list field-level privacy classes (see [STOA.md §13.1](../STOA.md#131-class-taxonomy) for the full taxonomy). The runtime **MUST** verify that every cap on the path can handle the declared classes before forwarding. Vendors that cannot honor the declared classes **MUST** return `privacy_zone_mismatch`.

### 3.5 Compensation override

The agent **MAY** override the graph-declared compensation by providing `compensation.on_undo` (a URN) and `compensation.key_path` (a JSON Path expression over the response output that yields the resource ID to compensate). This is rare; default compensation lives in the capability graph (see [STOA.md §9.6](../STOA.md#96-compensation-declaration-in-the-graph)).

---

## 4. Response envelope

A Stoa/1 response envelope is a UTF-8 JSON object returned by the vendor with HTTP status 200 (success), 4xx (client error), or 5xx (server error).

### 4.1 Success shape

```jsonc
{
  "stoa": "1",                                    // REQUIRED
  "status": "ok",                                 // REQUIRED for success
  "receipt": {                                    // REQUIRED at L4; OPTIONAL at L0–L3
    "alg": "ES256" | "EdDSA",                     // REQUIRED
    "sig": "<JWS compact serialization>",         // REQUIRED; detached signature over the receipt body
    "vendor_did": "did:web:<vendor>",             // REQUIRED
    "agent_co_sig": "<JWS compact>",              // OPTIONAL; agent counter-signs
    "merkle_root": "<hex>",                       // OPTIONAL until receipt is anchored (typically T+24h)
    "merkle_proof": ["<hex>", ...],               // OPTIONAL; proof of inclusion in the daily root
    "ts": <unix-seconds>,                         // REQUIRED
    "input_hash": "sha256:<hex>",                 // REQUIRED
    "output_hash": "sha256:<hex>",                // REQUIRED
    "state_delta_hash": "sha256:<hex>" | null     // REQUIRED if state_delta present
  },
  "state_delta": {                                // OPTIONAL; emit when a resource changed
    "resource": "urn:stoa:res:<...>",
    "version": <integer>,
    "etag": "<HTTP ETag>",
    "changeset": [
      { "op": "add" | "replace" | "remove" | "create" | "delete", "path": "<JSON Pointer>", "value_hash": "<hex>" | null, "old_hash": "<hex>" | null }
    ]
  },
  "continuation": "<opaque-token>" | null,        // OPTIONAL; present iff partial response
  "cost": {                                       // REQUIRED at L2+
    "actual_cents": <integer>,
    "breakdown": [ { "kind": "vendor.api" | "stoa.runtime" | "...", "amount_cents": <integer> }, ... ],
    "settlement_ref": "<rail>:<tx-or-invoice-ref>"
  },
  "side_effects": [                               // REQUIRED if any external side effect occurred
    {
      "kind": "<class>",                          // see §4.4
      "when": "T+<seconds>" | "now",              // when the effect actually fires (immediate vs scheduled)
      "undo": "urn:stoa:cap:<...>" | null,
      "undo_args": { ... } | null
    }
  ],
  "warnings": [                                   // OPTIONAL; non-fatal advisories
    { "code": "<warning-code>", "field": "<json-pointer>" | null, "removal": "<date>" | null }
  ],
  "lineage": {                                    // OPTIONAL; for L4 + lineage tracking
    "consumed_resources": ["urn:stoa:res:<...>", ...],
    "produced_resource": "urn:stoa:res:<...>" | null
  },
  "output": { ... }                               // REQUIRED; validates against capability output schema
}
```

### 4.2 Receipt signature

The `receipt.sig` is a [JWS Compact Serialization](https://www.rfc-editor.org/rfc/rfc7515) over the canonical JSON of the receipt body, with the `sig` field itself omitted from the body during signing (detached-signature pattern).

The signing key is the vendor's. Public keys **MUST** be published at `<vendor-host>/.well-known/did.json` (a `did:web` document), under a key with usage `assertionMethod`.

Receipts **SHOULD** be Merkle-anchored within 24 hours into a public log (foundation root). Anchoring is the responsibility of the runtime (see `stoa-edge`). Once anchored, `receipt.merkle_root` and `receipt.merkle_proof` are populated and become verifiable by any third party with the foundation's daily-root signature.

### 4.3 State delta semantics

The `state_delta.changeset` follows [JSON Patch](https://www.rfc-editor.org/rfc/rfc6902) operations except:

- Values are referenced by hash (`value_hash`), not inline. The runtime MAY include inline values for clients that opt in via an `Accept-Stoa-Inline-Values: 1` request header.
- The atomic operations `create` (entire resource created) and `delete` (entire resource removed) are added beyond standard JSON Patch.

State deltas published to a resource **MUST** be picked up by the bus (see `stoa-bus`) and fanned out to subscribers. The vendor **MUST** retain a 24h delta log per resource for `resync` operations.

### 4.4 Side-effect classes

The `side_effects[].kind` is a free-form vendor-defined string of the form `<scope>.<noun>.<verb>`. Recommended values:

- `external.email_will_send` — outbound email queued
- `external.crm.write` — CRM record created/updated
- `external.payments.charge` — charge initiated
- `external.message.send` — IM/Slack/SMS sent
- `external.file.write` — file written outside the resource graph
- `internal.cache_invalidated` — server-side cache flush (rarely needs reporting)

`undo` references the compensating capability URN; `undo_args` provides the required input. If the side effect is irrevocable, `undo` is `null` and the agent **SHOULD** be prompted before commit (see `policy.require_human_confirmation`).

### 4.5 Error shape

```jsonc
{
  "stoa": "1",
  "status": "error",
  "error": {
    "code": "<reserved-code>" | "<vendor-code>",  // REQUIRED
    "message": "<human-readable>",                 // REQUIRED
    "remediation": {                               // REQUIRED
      "hint": "<reserved-hint>",
      "next_capability": "urn:stoa:cap:<...>" | null,
      "retry_after_ms": <integer> | null,
      "compose_hint": "<free-form>" | null
    },
    "trace_id": "<trace-id>",
    "details": { ... }                             // free-form
  },
  "receipt": { ... } | null,                       // even errors get receipts when L4
  "envelope": { "id": null, "as_of": "<RFC 3339>" }
}
```

### 4.6 Reserved error codes

The following codes are **reserved** by Stoa/1 and **MUST** behave as documented:

| Code | Meaning | Default remediation |
|---|---|---|
| `unauthenticated` | No valid auth | `auth-refresh` |
| `forbidden` | Auth valid but scope insufficient | `escalate-to-user` |
| `not_found` | Resource does not exist | `permanent-failure` |
| `validation_failed` | Input failed schema validation | `fix-input-and-retry` |
| `rate_limited` | Quota exceeded | `backoff` |
| `idempotency_conflict` | Same key, different body | `permanent-failure` |
| `state_conflict` | ETag/version mismatch | `fix-input-and-retry` |
| `service_unavailable` | Vendor outage | `backoff` |
| `cost_limit_exceeded` | Vendor billing cap or per-call ceiling | `escalate-to-user` |
| `privacy_zone_mismatch` | Class/jurisdiction mismatch | `route-to-compliant-vendor` |
| `human_confirmation_required` | Cap requires user gate | `escalate-to-user` |
| `compensation_required` | Prior step needs to compensate first | `compose-different-capabilities` |

Vendors **MAY** define additional codes within their declared error namespace (see capability manifest). Agents that encounter an undeclared code **SHOULD** fall back to treating it as `permanent-failure`.

---

## 5. Identity (hive JWT)

### 5.1 JWT shape

The `agent.jwt` field carries a [JWS Compact Serialization](https://www.rfc-editor.org/rfc/rfc7515) signed by the issuing hive. The JWT payload **MUST** include:

```jsonc
{
  "iss": "did:web:<hive>",                          // REQUIRED
  "sub": "agent:<vendor>:<role>:<instance-id>",     // REQUIRED
  "aud": "did:web:<vendor>",                        // REQUIRED
  "scope": ["caps:<urn-pattern>", ...],             // REQUIRED
  "rep_class": "<tier>",                            // OPTIONAL
  "jit_policy": "<key=value;...>",                  // OPTIONAL (e.g. "ttl=300s,bind=ip+ja3")
  "exp": <unix-seconds>,                            // REQUIRED; SHOULD be ≤ 600s
  "iat": <unix-seconds>,                            // REQUIRED
  "cnf": { "jwk": { ... } },                        // OPTIONAL; proof-of-possession key (RFC 7800)
  "act_as": {                                       // OPTIONAL; when acting on behalf of a human
    "user_id": "<id>",
    "consent_token": "<opaque>"
  },
  "model_attestation": {                            // OPTIONAL; for caps requiring TEE-rooted attestation
    "model_id": "<canonical-id>",
    "model_hash": "sha256:<hex>",
    "system_prompt_hash": "sha256:<hex>"
  }
}
```

### 5.2 Verification

Vendors **MUST** verify:

1. The signature against the hive's `did:web` document under a key with `usage: assertionMethod`.
2. `aud` matches the vendor's own DID.
3. `exp` is in the future and `iat` is in the past (clock skew tolerance: 60 seconds).
4. `iss` is in the vendor's accepted-issuer policy (default: foundation-published allowlist).
5. The requested capability is within the granted `scope`.

Failure on any returns `unauthenticated` (signature/expiry) or `forbidden` (scope/issuer).

### 5.3 Model attestation

The optional `model_attestation` claim is the agent's pledge of which model and system prompt are operating. Vendors **MAY** require it for high-stakes capabilities (typically `finance.*`, `medical.*`, `legal.*`).

For TEE-rooted hives (Phala, Marlin, AWS Nitro, Intel TDX), the attestation is verifiable. For non-TEE hives, it is a signed *claim* whose accuracy is backed by reputation. Vendors decide what they require per cap.

---

## 6. Idempotency examples

### 6.1 Successful retry

Request 1 (T=0):
```json
{"stoa":"1","cap":"urn:stoa:cap:hubspot.contacts.create@2.3.1","idem":"plan_482:step_3","input":{"email":"a@b.com"}, ...}
```
Response 1 (T=0+150ms):
```json
{"stoa":"1","status":"ok","receipt":{"sig":"R1...","input_hash":"sha256:abc...","output_hash":"sha256:def..."},"output":{"id":"84021"}}
```

Request 2 (T=10s, same `idem`, same input):
Response 2: identical to Response 1 (same `R1` signature). No new contact created.

### 6.2 Conflict

Request 3 (T=20s, same `idem`, different input):
Response 3:
```json
{"stoa":"1","status":"error","error":{"code":"idempotency_conflict","message":"...","remediation":{"hint":"permanent-failure"}}}
```

---

## 7. HTTP transport

Stoa/1 is JSON-over-HTTP/1.1+. Recommended bindings:

- **Method**: `POST` for all capabilities (even `read`-class, for idempotency-key support).
- **Path**: `<vendor-host>/<openapi_path>`. The `cap` URN resolves to a path via the capability graph (see [STOA.md §7](../STOA.md#7-the-network--the-federated-capability-graph)).
- **Content-Type**: `application/stoa+json` preferred; `application/json` accepted.
- **Accept**: `application/stoa+json` preferred.
- **Idempotency-Key** header: vendors **MUST** also accept the legacy Stripe-style `Idempotency-Key` HTTP header that mirrors `idem` field; behavior is identical.
- **Trace headers**: W3C `traceparent` / `tracestate` mirror `trace.parent` and **SHOULD** be set.
- **Acting-As** header: when `agent.jwt.act_as` is present, `Acting-As: user_id=<id>; agent_id=<sub>; session_id=<id>; trace_id=<id>` **MUST** be emitted.

### 7.1 Streaming responses

For capabilities that may stream (typically `query` or long-running `action`), vendors **MAY** respond with `Content-Type: text/event-stream`. Each SSE event payload is itself a Stoa/1 envelope with `continuation` set on intermediate events. The terminal event has `continuation: null`.

### 7.2 Subscriptions (live state bus)

Per-resource subscriptions live at `<vendor-host>/v1/subscribe?res=<resource-urn>&since=<version>` returning SSE state-delta events (see [STOA.md §12](../STOA.md#12-live-state-bus)). This is normative for L3+ conformance.

---

## 8. Conformance

A vendor declaring `Stoa/1` conformance **MUST**:

- Implement the request envelope parsing per §3
- Implement the response envelope shape per §4 for at least one declared capability
- Honor reserved error codes per §4.6
- Honor `idem` per §3.3 for `create`/`update`/`action` capabilities
- Verify hive JWTs per §5.2

A vendor declaring **L2+** conformance **MUST** also:

- Issue `cost` blocks per §4.1 with accurate `actual_cents`
- Honor settlement rails declared in `auth.kinds`

A vendor declaring **L3+** conformance **MUST** also:

- Issue state envelopes (`state_delta`) per §4.3
- Implement live subscriptions per §7.2
- Use opaque continuation tokens for pagination

A vendor declaring **L4** conformance **MUST** also:

- Issue signed receipts per §4.2 on every response (success and error)
- Anchor receipts to the foundation daily Merkle root within 24h
- Declare and honor compensation per §3.5
- Enforce privacy classes per §3.4

The full conformance test suite is at [`Vext-Labs-Inc/stoa-conformance`](https://github.com/Vext-Labs-Inc/stoa-conformance).

---

## 9. Security considerations

### 9.1 JWT replay

Short-lived tokens (`exp ≤ 600s`) and proof-of-possession (`cnf.jwk` with mutual-TLS or DPoP-style binding) reduce replay surface. Vendors **SHOULD** require PoP for high-stakes capabilities.

### 9.2 Idempotency-key squatting

Idempotency keys **MUST** be scoped to the agent (`<agent_id>:...`) so one agent cannot squat another's keys. Vendors that ignore agent scoping fail conformance.

### 9.3 Side-effect leakage

Vendors **MUST NOT** include raw PII/PHI in receipt bodies. Only `input_hash` and `output_hash` are stored. The full input/output is logged only by the vendor, and only if its privacy policy allows.

### 9.4 Cost griefing

Without `budget.ceiling_cents`, a misbehaving cap can rack up unbounded cost. Agents **SHOULD** always set a ceiling. Runtimes **MUST** enforce the ceiling at the Budget DO before forwarding.

### 9.5 Capability impersonation

URNs are vendor-namespaced. The capability graph's attestation chain is the source of truth for whether a URN is genuine. Agents **SHOULD** prefer caps with foundation attestation > vendor self-attestation > third-party attestation.

---

## 10. Backwards compatibility

### 10.1 With MCP

A Stoa/1 endpoint **MAY** also expose itself as an MCP server. The MCP `tools/list` and `tools/call` operations are mapped to the capability manifest. MCP clients see prose descriptions; Stoa clients see the typed envelope.

This is the recommended migration path: ship MCP first, then add Stoa/1 atop the same handlers.

### 10.2 With OpenAPI

Existing OpenAPI 3.1 servers can become Stoa-conformant by:
1. Adding `x-stoa` extensions to operations (capability URN, side-effect annotations, idempotency declaration, error mapping)
2. Wrapping responses in the Stoa/1 envelope
3. Publishing `/.well-known/stoa.json` discovery

The `stoa-graph` repo ships an OpenAPI→Stoa generator.

---

## 11. Open questions

- **Q1: Vendor key rotation cadence.** Should we mandate rotation every 90 days? RECOMMENDED for now; may be tightened to MUST in a future minor.
- **Q2: Agent co-signature mandatory at L4?** Currently OPTIONAL. Pro: stronger non-repudiation. Con: not all hives mint co-signing keys. Open for review.
- **Q3: SSE vs WebSockets for subscriptions.** Currently SSE. Some clients prefer WS. Open for v0.2.
- **Q4: Receipt anchoring SLA.** "Within 24h" is loose. Should we require T+1h? Trade-off: cost of anchoring infra vs. audit latency.

Comments welcome on the [GitHub Discussion thread for this RFC](https://github.com/Vext-Labs-Inc/stoa-spec/discussions).

---

## 12. Reference

- [STOA.md](../STOA.md) — master architecture (this RFC is a normative excerpt of §5)
- [SPEC.md](../SPEC.md) — Stoa v0.1 specification
- [`stoa-edge`](https://github.com/Vext-Labs-Inc/stoa-edge) — reference runtime implementation
- [`stoa-sdk`](https://github.com/Vext-Labs-Inc/stoa-sdk) — reference client SDK
- [`stoa-conformance`](https://github.com/Vext-Labs-Inc/stoa-conformance) — automatable test suite
- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — keywords
- [RFC 7515](https://www.rfc-editor.org/rfc/rfc7515) — JWS
- [RFC 7800](https://www.rfc-editor.org/rfc/rfc7800) — proof-of-possession
- [RFC 6902](https://www.rfc-editor.org/rfc/rfc6902) — JSON Patch
- [W3C did:web](https://w3c-ccg.github.io/did-method-web/) — DID method

---

## 13. Changelog

- **2026-05-10** — RFC-0001 published as draft. 14-day comment period opens.

— *Stoa Editor Team, Vext Labs*

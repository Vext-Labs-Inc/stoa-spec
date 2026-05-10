# Stoa v0.1 — Specification

**Status:** Draft 1 (open standard, public review pending)
**Author:** Vext Labs — editor of v0.1
**Date:** 2026-05-10 (renamed from Stoa → Stoa; OSS commitment formalized)
**Scope:** Wire-level protocol that any SaaS vendor can implement to expose itself to autonomous agents as a first-class consumer. **Open standard, fully open source.** Consumed by Vext Pro Hive AND by every other agent platform that adopts it.
**License:** Spec under **CC-BY-4.0**. Conformance suite, reference adapters, runtime, and SDKs all under **Apache-2.0**. Vext Labs is the editor of v0.1; spec hands off to a neutral foundation (Linux Foundation candidate) at the **5-vendor trigger** — see §11 and [GOVERNANCE.md](GOVERNANCE.md).
**Companion:** [STOA.md](STOA.md) (master architecture + manifesto), [GOVERNANCE.md](GOVERNANCE.md), [ECONOMICS.md](ECONOMICS.md), [BRAND.md](BRAND.md)

---

## 0. Why this exists

Today's autonomous agents drive software built for humans. They click pixels, scrape DOMs, parse rendered HTML, and hallucinate when a button moves. That is the root cause of most agent failures in the wild. The industry's response has been "make the agent smarter." That is the wrong direction.

**Stoa is the answer:** SaaS products publish a typed, contract-first, self-describing surface designed for agents — the *substrate* through which agents act on the world. When the UI redesigns, the Stoa contract is unchanged; the agent never breaks.

Concretely, a Stoa-conformant product publishes a **capability manifest** (JSON Schema + OpenAPI 3.1 + MCP server) listing every action an agent can take, with typed inputs, typed outputs, side-effect annotations, idempotency keys, and rollback semantics. Every response uses a **deterministic state envelope** with a **signed receipt**. Every error is a **typed, remediable contract**. Permissions are **capability-scoped**, not endpoint-scoped. Identity is **hive-attested** with optional model attestation. Multi-step plans get **automatic compensation** declared in the capability graph itself.

This spec is what the manifest, envelope, errors, and scopes look like.

---

## 1. Stance

Stoa sits **above** OpenAPI 3.1, JSON Schema 2020-12, and MCP — it does not replace them. A vendor with a clean OpenAPI spec is 80% of the way there. Stoa adds the missing 20% that agents actually need:

- **Side-effect annotations** (idempotent? mutating? destructive? requires-confirmation?)
- **Deterministic state envelopes** (versioning, canonical IDs, continuation tokens)
- **Signed receipts** anchored to a public Merkle root (audit + compliance)
- **Typed error contracts** with remediation hints and retry policy enums
- **Capability-level permission scopes**
- **Hive-issued agent identity** with optional model attestation
- **Saga compensation** declared in the capability graph
- **Privacy classes** that route PII/PHI/Financial through compliant infrastructure
- **Live state bus** so agents subscribe to resource changes instead of polling
- **Cost oracle** with declared price + reliability + latency per capability
- **Conformance test suite** that's automatable

The minimum viable Stoa implementation is an existing OpenAPI 3.1 spec + an `x-stoa` extension block per operation + a `/.well-known/stoa.json` discovery file. **Stoa-conformant servers can speak MCP at the wire layer** for backwards compatibility with existing MCP clients.

---

## 2. Discovery

Every Stoa-conformant product MUST serve a discovery document at:

```
GET https://<host>/.well-known/stoa.json
```

```jsonc
{
  "spec_version": "stoa-0.1",
  "vendor": {
    "name": "Acme CRM",
    "homepage": "https://acme-crm.com",
    "support_email": "agents@acme-crm.com",
    "verified": false               // true once Vext (or any conformance authority) audits
  },
  "manifest_url": "https://acme-crm.com/api/.well-known/stoa-manifest.json",
  "openapi_url":  "https://acme-crm.com/api/openapi.json",
  "mcp_url":      "https://acme-crm.com/mcp",                      // optional; recommended
  "auth": {
    "kinds": ["oauth2", "api_key"],
    "oauth2_well_known": "https://acme-crm.com/.well-known/oauth-authorization-server"
  },
  "rate_limits": {
    "default_qps": 10,
    "burst": 30,
    "documented_at": "https://acme-crm.com/docs/rate-limits"
  },
  "conformance": {
    "level": "core",                  // core | typed-errors | typed-state | full
    "tested_at": "2026-04-15T00:00:00Z",
    "report_url": "https://acme-crm.com/conformance/stoa-0.1-report.json"
  }
}
```

`spec_version` MUST be a string of form `stoa-<major>.<minor>`. Agents discovering a vendor at a higher minor version they don't recognize SHOULD proceed assuming forward-compatibility (additive changes only); higher major version requires explicit support.

---

## 3. The capability manifest

### 3.1 Shape

```jsonc
{
  "spec_version": "stoa-0.1",
  "manifest_id": "acme-crm-prod",
  "manifest_version": "2026.04.15",
  "vendor": "Acme CRM",
  "capabilities": [
    {
      "id": "crm.contacts.create",
      "summary": "Create a contact record",
      "inputs": { "$ref": "#/components/schemas/CreateContactInput" },
      "outputs": { "$ref": "#/components/schemas/Contact" },
      "side_effects": {
        "kind": "create",                   // see §3.3
        "idempotency": "client-key",        // see §3.4
        "rollback": "delete-by-id",         // see §3.5
        "destructive": false,
        "requires_confirmation": false,
        "rate_class": "write"               // see §3.6
      },
      "scopes": ["crm:contacts:write"],     // §6
      "errors": [                           // §5
        { "code": "duplicate_email", "remediation": "search-then-update" },
        { "code": "validation_failed", "remediation": "fix-input-and-retry" },
        { "code": "rate_limited", "remediation": "backoff" }
      ],
      "openapi_path": "/v1/contacts",
      "openapi_method": "POST"
    }
    /* … more capabilities … */
  ],
  "components": {
    "schemas": { /* JSON Schema 2020-12 definitions */ }
  }
}
```

### 3.2 The `id` namespace

Capability IDs follow the pattern `<domain>.<resource>.<action>`. This is the key the planner queries against. Agents should never key off `openapi_path` directly — paths drift; IDs are stable. Recommended domains:

- `crm`, `email`, `calendar`, `chat`, `docs`, `tickets`, `code`, `finance`, `hr`, `legal`, `marketing`, `analytics`, `infra`, `auth`, `files`, `forms`, `payments`, `shipping`, `inventory`, `media`, `search`, `monitoring`

Vendors register custom domains by including them in the manifest's optional `domains` block. Reserved: `stoa` (used for spec metadata) and `vext` (Vext-only).

### 3.3 `side_effects.kind` enum

| Value | Meaning | Agent guidance |
|---|---|---|
| `read` | No side effects; safe to retry indiscriminately | Cache server-side; agents may parallelize |
| `create` | Creates a new resource | Always require idempotency key |
| `update` | Modifies existing | Use ETag / version field; reject blind writes |
| `delete` | Destroys a resource | `destructive: true` MUST be set |
| `action` | Triggers a side effect outside the resource graph (send email, charge card, run job) | `destructive` is true unless explicitly safe |
| `query` | Read with non-trivial computation (search, aggregate) | Caching at the gateway is fine; results may be paginated |

### 3.4 `idempotency` enum

| Value | Meaning |
|---|---|
| `none` | Not idempotent. Agent must commit only once; on uncertainty, treat as committed |
| `client-key` | Idempotent if the agent supplies an `Idempotency-Key` header |
| `natural-key` | Idempotent on a natural-key field (e.g. `email` for contact creation) |
| `server-dedupe` | Server deduplicates within a window; agent may retry freely within that window |

Conformance level `core` requires that any `create`/`update`/`action` capability declare *one of* these. `none` is allowed but flagged in the conformance report.

### 3.5 `rollback` enum

| Value | Meaning |
|---|---|
| `none` | No rollback path (e.g. an outbound email already sent) |
| `delete-by-id` | Inverse is a delete on the returned ID |
| `update-with-prior-state` | Inverse is an update reverting to the previous state envelope |
| `compensating-action:<id>` | Pointer to another capability ID that compensates this one |

### 3.6 `rate_class`

Free-form vendor-defined rate-limit class string. Agents respect the per-class QPS published in the discovery document.

---

## 4. State envelope

Every Stoa response MUST be wrapped in a state envelope. This is the part that kills "page 2 of 17" hallucinations.

### 4.1 Shape

```jsonc
{
  "data": { /* the actual resource(s) — typed per capability output schema */ },
  "envelope": {
    "id": "ctc_01HK...",                    // canonical ID (resource-stable, cross-version)
    "version": "v3",                         // resource version for optimistic concurrency
    "etag": "W/\"abc123\"",                  // mirrors HTTP ETag; SHOULD also be in HTTP header
    "as_of": "2026-04-15T12:34:56Z",         // server time when this snapshot was rendered
    "fresh_for_ms": 30000,                   // suggested cache TTL
    "continuation": {                        // present iff partial response
      "token": "opaque-cursor-string",
      "estimated_remaining": 47,
      "ordering": "created_at_desc"
    },
    "links": {                               // canonical resource graph traversal
      "self": "/v1/contacts/ctc_01HK",
      "owner_account": "/v1/accounts/acc_01HK"
    },
    "consistency": "strong"                  // strong | read-after-write | eventual
  },
  "warnings": [                              // optional; non-fatal advisories
    { "code": "deprecated_field", "field": "phone_legacy", "removal": "2027-01-01" }
  ]
}
```

`data` MUST be present on success. `envelope.id` and `envelope.as_of` are mandatory. Everything else is optional but strongly recommended.

### 4.2 Pagination contract

Continuation tokens are opaque strings. Servers MUST accept the token verbatim on the next call (`?continuation=<token>`). Servers MUST NOT change ordering across continuation calls (`envelope.continuation.ordering` is the contract). When `continuation` is absent, the response is complete.

This kills the entire class of "the agent re-asked for page 1 instead of page 2" failures.

### 4.3 Streaming responses

For capabilities with `side_effects.kind in ("query", "action")` that may stream, servers MAY use SSE. Each SSE event payload is itself an envelope-wrapped chunk. The terminal event has `envelope.continuation = null`.

---

## 5. Error contracts

Every Stoa error response MUST follow:

```jsonc
{
  "error": {
    "code": "duplicate_email",                   // enum from capability's declared errors
    "message": "A contact with this email already exists",
    "remediation": {
      "hint": "search-then-update",
      "next_capability": "crm.contacts.search",  // capability ID to call next
      "retry_after_ms": null
    },
    "trace_id": "trc_01HK...",                   // for vendor-side debugging
    "details": { /* free-form, vendor-specific */ }
  },
  "envelope": {
    "id": null,
    "as_of": "2026-04-15T12:34:56Z"
  }
}
```

### 5.1 The `remediation.hint` enum

| Hint | Agent action |
|---|---|
| `fix-input-and-retry` | The input failed validation — re-derive and retry |
| `backoff` | Rate-limited or transient — retry with exponential backoff per `retry_after_ms` |
| `auth-refresh` | Token expired — refresh and retry once |
| `escalate-to-user` | Requires human input the agent cannot supply |
| `permanent-failure` | Don't retry; record and move on |
| `search-then-update` | Resource exists; convert to update flow |
| `search-then-merge` | Resource exists; merge logic required |
| `wait-and-poll` | Async result; poll the returned `next_capability` |
| `compose-different-capabilities` | The action requires multiple capabilities; planner should re-decompose |

### 5.2 Error code namespace

Error codes are vendor-defined within their capability's declared `errors` array. Agents that encounter an undeclared code SHOULD fall back to treating it as `permanent-failure` — a vendor that emits undeclared codes fails conformance.

### 5.3 Reserved codes

These codes are reserved by the spec and MUST behave as documented:

| Code | Meaning | Default remediation |
|---|---|---|
| `unauthenticated` | No valid auth | `auth-refresh` |
| `forbidden` | Auth valid but scope insufficient | `escalate-to-user` |
| `not_found` | Resource does not exist | `permanent-failure` |
| `validation_failed` | Input failed schema validation | `fix-input-and-retry` |
| `rate_limited` | Quota exceeded | `backoff` |
| `idempotency_conflict` | Same key, different body | `permanent-failure` |
| `state_conflict` | ETag/version mismatch | `fix-input-and-retry` (re-fetch + reconcile) |
| `service_unavailable` | Vendor-side outage | `backoff` |
| `cost_limit_exceeded` | Vendor billing cap | `escalate-to-user` |

---

## 6. Permission scopes

Stoa scopes are **capability-level**, not endpoint-level. The pattern:

```
<domain>:<resource>:<action>
```

Examples:
- `crm:contacts:read` — read contact records
- `crm:contacts:write` — create + update (NOT delete)
- `crm:contacts:delete` — destroy
- `crm:contacts:*` — any contact action
- `crm:*:read` — read-only across all CRM resources
- `crm:*:*` — full CRM access

Vendors MAY declare custom non-standard scopes; those MUST be prefixed with `x-` (e.g. `x-acme:territory:write`).

### 6.1 The minimum-capability principle

The Hive's planner requests the smallest scope that covers the plan. If a step needs `crm.contacts.create` only, the planner asks for `crm:contacts:write`, not `crm:*:*`. The Stoa layer audits actual capability use against scope grant per-call; over-scoped grants generate a warning in the conformance report.

---

## 7. Authentication

### 7.1 Supported kinds

| Kind | Use case |
|---|---|
| `oauth2` | Standard for any tenant-bound API. RFC 8414 metadata at `/.well-known/oauth-authorization-server` |
| `api_key` | Simple service-to-service (header: `X-Stoa-Key`) |
| `mtls` | Enterprise integrations. Cert distributed via SPIFFE or vendor-specific channel |
| `oidc` | When the agent's identity itself matters (e.g. policy enforcement based on which user is acting through the agent) |
| `agent-bearer` | New: an agent-attested JWT signed by the calling Hive (Vext-issued); vendors may accept it as a delegated identity (§7.3) |

### 7.2 The `Acting-As` header

When an agent acts on behalf of a human, requests SHOULD include:

```
Acting-As: user_id=usr_01HK; agent_id=ag_01HK; session_id=sess_01HK; trace_id=trc_01HK
```

This lets vendors record audit trails that distinguish "user did X" from "user's agent did X" — important for regulated domains (healthcare, finance, legal).

### 7.3 `agent-bearer` (Vext-attested delegation)

A Vext-issued JWT carrying:
- `iss` = Hive Coordinator
- `sub` = agent_id
- `aud` = vendor host
- `act_as` = user_id
- `scope` = capability scope claim
- `cnf` = workspace public key (for mTLS-bound tokens)

Vendors who accept `agent-bearer` get the benefit of automatic audit and enforcement without implementing OAuth themselves. This is the on-ramp for small SaaS vendors.

---

## 8. Conformance levels

A vendor's `conformance.level` claim is one of:

| Level | Requires |
|---|---|
| `core` | Discovery doc + capability manifest with IDs + side-effect annotations |
| `typed-errors` | `core` + every error has a declared code and remediation hint |
| `typed-state` | `typed-errors` + every response uses the state envelope; pagination uses opaque continuation tokens; ordering is documented |
| `full` | `typed-state` + capability scopes + Acting-As header support + `/.well-known` discovery; passes the conformance test suite (§9) |

The Vext Stoa marketplace surfaces vendors by level. `full`-conformant vendors get the prominent shelf; `core` vendors get listed but flagged.

---

## 9. Conformance test suite (outline)

The suite is automatable and runs against a candidate Stoa host. Open-source under MIT.

### 9.1 What it tests

For each declared capability:

1. **Discovery roundtrip.** GET `/.well-known/stoa.json` resolves; manifest URL resolves; manifest validates against the v0.1 JSON Schema.
2. **Schema-level conformance.** Inputs accept all schema-valid examples; reject all schema-invalid ones with `validation_failed`.
3. **Idempotency.** Repeat a `create` with the same key; second call returns the same ID without creating a duplicate.
4. **State envelope.** All responses include `envelope.id` and `envelope.as_of`.
5. **Pagination determinism.** Continuation tokens preserve ordering and don't drift records.
6. **Error envelope.** Every declared error code is reachable by a synthetic input; the response matches §5.
7. **Scope enforcement.** Calls with insufficient scope return `forbidden` with the right scope-mismatch detail.
8. **Acting-As propagation.** When the header is set, the vendor's audit log records both user and agent (verified via a vendor-supplied audit-readback endpoint, optional).
9. **Rate-limit honesty.** Sustained QPS at the documented `default_qps` does not return `rate_limited`; QPS at `burst` for short bursts also does not.
10. **Webhook contract** (if vendor declares webhook delivery): retries, ordering, idempotency.

### 9.2 Output

The suite emits a JSON report with per-capability pass/fail/warn and a top-line `conformance.level`. Vendors include the report at `conformance.report_url` so consumers can re-verify.

### 9.3 Where it lives

`https://github.com/stoa-spec/stoa-conformance` — open-source, MIT.

---

## 10. The Stoa registry — federated, free, forkable

The capability registry is **federated by design**. No single registry is the canonical source. Multiple registries can exist; URNs resolve cross-registry by signed pointer.

- **Foundation registry** at `https://caps.stoa.foundation/` — the default, free, hosted by the Stoa Foundation post-handoff (by Vext Labs pre-handoff).
- **Vendor-run registries** — any vendor can run their own (HubSpot, Stripe, OpenAI for app store, etc.).
- **Self-hosted registries** — buyers can run private registries inside their VPC for compliance-strict deployments.
- **Search by capability URN** (`find me everything that does crm.contacts.create`) — semantic + structural + compositional.
- **Vext-attested adapters** — OpenAPI-to-Stoa shims for major SaaS vendors not yet self-conformant. Apache-2.0 code; Vext charges $2K/mo for hosted maintenance + SLA, but the code is open and forkable.
- **No marketplace fee on the standard.** Listing in the foundation registry is free, forever. There is no platform tax, no listing tax, no traffic tax on Stoa itself. Vext's revenue is in Theron + AE OS, not in the registry. See [ECONOMICS.md](ECONOMICS.md).

---

## 11. Versioning & foundation handoff

- Spec version is `stoa-<major>.<minor>`. v0.1 is initial draft.
- v1.0 ships at the **5-vendor trigger**: when 5 independent vendors hit L4 conformance, spec hands off to a neutral foundation (Linux Foundation candidate) and v1.0 freezes under foundation governance.
- Spec changes are RFC-driven; live on GitHub at `https://github.com/stoa-spec/stoa-spec`.
- Backward compatibility within a major version is mandatory. Major bumps are rare and require a deprecation window of 12 months.

**Public, dated, irreversible commitment:** Vext Labs commits to transfer editorial authority, trademark, repository ownership, and conformance certification to a neutral foundation when five independent vendors achieve L4 conformance. Failure to initiate transfer within 90 days of the trigger firing is grounds for any party to fork the spec under existing licenses; Vext explicitly waives any objection to such forks. See [GOVERNANCE.md](GOVERNANCE.md).

---

## 12. Why an agent should prefer Stoa

| Characteristic | Browser scraping | OpenAPI 3.1 | Stoa |
|---|---|---|---|
| Stability under UI redesign | broken | unaffected | unaffected |
| Idempotency contract | no | partial | yes |
| Pagination determinism | no | partial | yes |
| Typed errors w/ remediation | no | error code only | yes |
| Capability discovery | DOM-walk | path-based | ID-based, semantic |
| Cost predictability | no | partial | yes (rate_class + vendor pricing manifest) |
| Audit trail (Acting-As) | manual | manual | first-class |
| Conformance attestation | n/a | none | open test suite |

Higher-fidelity adapters cost less to run (fewer retries, no scraping retries on layout drift, predictable rate limits) and produce better outcomes (no "the bot booked the wrong meeting because the calendar UI changed"). For Vext Pro Hive, Stoa is the default ingest; browser scraping is the explicit fallback we report to the user as low-fidelity.

---

## 13. Open questions

- **Cost manifest.** Should vendors publish a per-capability price hint (USD per call, USD per GB)? Useful for the Hive cost governor; controversial because it commits the vendor publicly. Likely deferred to v0.2.
- **Action revocation.** Should the spec define a "revoke this in-flight action" endpoint? Useful for the cost governor's hard-kill. Not in v0.1.
- **Webhooks vs polling.** Async results today are ad-hoc. v0.2 should standardize.
- **Cross-vendor transactions.** Multi-step plans across two vendors (book + invoice) need a saga pattern. Out of scope for v0.1; flagged for v0.2.
- **Privacy classification.** Inputs/outputs containing PII should be flagged so the Hive's cost governor / privacy auditor can route to the right retention policy. Likely v0.2 (`x-privacy-class`).

---

## 14. License — fully open source

**Stoa is fully open source. Forever. Across all artifacts.**

| Artifact | License |
|---|---|
| Spec text (this document, RFCs, governance docs) | **CC-BY-4.0** |
| Conformance test suite (`stoa-conformance`) | **Apache-2.0** |
| Runtime code (`stoa-edge`) | **Apache-2.0** |
| SDKs (`stoa-sdk` — TS/Python/Rust) | **Apache-2.0** |
| Federated registry implementation (`stoa-graph`) | **Apache-2.0** |
| Reference adapters (`stoa-adapters/*`) | **Apache-2.0** |
| Trademark "Stoa" + logo | Held by Vext Labs pre-handoff; transferred to foundation at the 5-vendor trigger under a trademark policy that allows any conformant vendor to use the marks descriptively. |

These licenses are **irrevocable**. Vext Labs cannot relicense, restrict, or rescind. Anyone with a copy retains rights regardless of what Vext does later. The community can fork at any time without permission.

The spec is governed by an open RFC process; Vext Labs is the editor of v0.1 but **not the long-term steward** — at the 5-vendor trigger, editorial authority, trademark, and repository ownership transfer to a neutral foundation (Linux Foundation candidate). See §11 and [GOVERNANCE.md](GOVERNANCE.md).

**Why open source:** Stoa is substrate. Substrate layers — TCP/IP, HTTP, OAuth, OpenAPI, MCP — never get monetized at the protocol layer. They get monetized at the value layer. Vext's value is **Theron** (council of 30 specialist 100B+ models) and **AE OS** (the surface where intent becomes work). Stoa is what makes both reach the world; charging for Stoa would suppress adoption and shrink the cake. See [ECONOMICS.md](ECONOMICS.md).

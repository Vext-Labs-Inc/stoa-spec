# Stoa — The Open Substrate for Agent-Readable SaaS

**Status:** Master Architecture & Manifesto, v1 draft
**Date:** 2026-05-10
**Editor:** Vext Labs (v0.1 → v1.0; foundation handoff at 5 independently-conformant vendors)
**License:** Spec under CC-BY-4.0. Runtime, SDKs, conformance suite, and reference adapters under Apache-2.0. **Everything Stoa is open source. Forever.**
**Companion:** [SPEC.md](SPEC.md) (the wire spec), [GOVERNANCE.md](GOVERNANCE.md), [ECONOMICS.md](ECONOMICS.md), [FOUNDER_ESSAY.md](FOUNDER_ESSAY.md)

> *In the colonnaded walkway of ancient Athens, philosophers and merchants worked under one roof. Ideas became contracts. Contracts became commerce. The στοά was the substrate of how Athens did business with itself.*
>
> *Stoa is what we call the substrate of how the world does business with agents.*

---

## Table of contents

1. [What Stoa is](#1-what-stoa-is)
2. [Why open source — and why the commons wins](#2-why-open-source--and-why-the-commons-wins)
3. [What's actually broken in 2026](#3-whats-actually-broken-in-2026)
4. [The three pillars](#4-the-three-pillars)
5. [The wire — Stoa/1](#5-the-wire--stoa1)
6. [The runtime — Stoa Edge](#6-the-runtime--stoa-edge)
7. [The network — the federated capability graph](#7-the-network--the-federated-capability-graph)
8. [Identity — DIDs, hives, JWTs, reputation](#8-identity--dids-hives-jwts-reputation)
9. [Sagas & compensation](#9-sagas--compensation)
10. [Cost & settlement — multi-rail](#10-cost--settlement--multi-rail)
11. [Receipts & audit](#11-receipts--audit)
12. [Live state bus](#12-live-state-bus)
13. [Privacy classes](#13-privacy-classes)
14. [Offline-first bundles](#14-offline-first-bundles)
15. [Composition primitives — plans, bundles, dependencies](#15-composition-primitives--plans-bundles-dependencies)
16. [Sandbox, replay, lineage](#16-sandbox-replay-lineage)
17. [Adapter framework — for non-native vendors](#17-adapter-framework--for-non-native-vendors)
18. [Versioning & deprecation](#18-versioning--deprecation)
19. [Conformance levels](#19-conformance-levels)
20. [Governance & foundation handoff](#20-governance--foundation-handoff)
21. [The economics — why this works for Vext](#21-the-economics--why-this-works-for-vext)
22. [Five modules to build in parallel](#22-five-modules-to-build-in-parallel)
23. [Six-month shipping plan](#23-six-month-shipping-plan)
24. [Ten things that become possible](#24-ten-things-that-become-possible)
25. [FAQ](#25-faq)

---

## 1. What Stoa is

Stoa is **the open substrate for agent-readable SaaS**. Three artifacts, one product, all open source:

- **The standard** — a typed, contract-first, self-describing wire protocol that any SaaS vendor implements to expose itself to autonomous agents as a first-class consumer. Sits *above* OpenAPI 3.1, JSON Schema 2020-12, and MCP. Adds the missing 20% that agents actually need: side-effect annotations, idempotency, rollback, capability-scoped permissions, deterministic state envelopes, typed error contracts, signed receipts.
- **The runtime** — a stateless edge mesh that handles execution: semantic discovery, multi-vendor saga compensation, per-call cost governance, signed receipts, live state subscriptions, privacy-zone routing, agent identity verification. Runs on Cloudflare Workers + Durable Objects. Anyone can self-host. Anyone can fork.
- **The network** — a federated, signed, semantic capability graph. Every capability has a stable URN, a vendor signature, an attestation chain, an embedding for retrieval, a price oracle entry, a reliability score, a privacy zone. Daily signed bundles distributed as ~5MB tarballs. No central choke point.

Together they form **the layer agents reach into the world through**. The agent has intent; Stoa has execution.

**One-paragraph elevator:**

> *Stoa is the open standard, runtime, and federated registry that lets any agent — yours, ours, OpenAI's, Anthropic's, Cursor's, Devin's, the one you're about to ship — call any SaaS as a typed, signed, idempotent, cost-governed, audit-trailed capability instead of clicking through a UI it was never meant to use. Spec under CC-BY-4.0. Runtime and SDKs under Apache-2.0. Foundation hand-off at five independently-conformant vendors. Vext Labs is the editor of v0.1, the first reference implementation, and the first consumer through Theron and AE OS — and that is the entire extent of our claim.*

That paragraph goes on the home page.

---

## 2. Why open source — and why the commons wins

Stoa is **completely open source, all three pillars, forever.** Spec, runtime, SDKs, conformance suite, reference adapters, and the federated registry implementation. Apache-2.0 / MIT / CC-BY-4.0 across the artifacts. Anyone can fork the registry; anyone can run their own marketplace; anyone can write their own runtime.

This is a deliberate strategic choice and the right one. Five reasons:

### 2.1 Commons economics are the only economics that work for substrate

Substrate layers — TCP/IP, HTTP, OAuth, OpenAPI, MCP, WebRTC — never get monetized at the protocol layer. They get monetized at the value layer that runs on top. **Linux is free; Red Hat sells a $40B distribution. Kubernetes is free; AWS, GCP, and Azure sell $100B in managed clusters. PostgreSQL is free; Neon, Supabase, and Crunchy Data have raised $1B combined. MCP is free; Anthropic still owns the conversation.** The companies that try to rent the substrate (think: every "open core" plus license trap) get forked or routed around within 18 months.

Vext's value is **Theron** (the brain) and **AE OS** (the surface). Stoa is what makes them powerful. If Stoa is proprietary, every agent platform on the planet has an incentive to fork or build a competing standard — and they will, because they're well-funded and the substrate is too important to cede. If Stoa is genuinely open, every agent platform has an incentive to *adopt* it and contribute to it, because they can't be rent-extracted.

We win by making the cake bigger, not by owning the slice. The marketplace economics still work even if 50% of agent traffic eventually routes through OpenAI's or Anthropic's catalog instead of ours, because Vext earns on **Theron specialist calls** and **AE OS subscriptions** regardless of which Stoa registry brokers them.

### 2.2 The standard is only worth something if vendors trust it

A vendor publishing a Stoa contract is making a forward commitment to keep it stable. They will not make that commitment to a standard one company can revoke or rewrite. Open source removes the threat. Foundation hand-off (§20) makes it credible.

### 2.3 The runtime is only worth something if buyers trust it

A buyer routing their agent traffic through a runtime is putting receipts, billing, and audit trails in that runtime's hands. They will not do that with a black box. Open source means anyone can audit the runtime, fork it, self-host it, run their own. The fact that **most won't bother** (because the hosted Vext-run instance is faster and cheaper than self-hosting Cloudflare Workers + R2) is fine — what matters is the option exists.

### 2.4 The network is only worth something if it's federated

A central capability graph is a single point of compromise, a single point of censorship, a single point of failure. Stoa's graph is federated by design (§7): Vext runs one registry, HubSpot can run their own, OpenAI can run one for their app store, the Linux Foundation can run the canonical foundation registry. URNs are global; signatures resolve cross-registry; conflicts resolve by attestation strength. **No one company can be the sole gatekeeper.**

### 2.5 The economic moat is in the value layer, not the substrate

This is the deepest reason. Vext's competitive moat is:

- **Theron** — 30 specialist 100B+ models trained on primary sources, served on a stack we own end-to-end. Nobody else has the corpus, the recipes, or the council architecture.
- **AE OS** — the desktop surface where intent becomes work. Eyes, hands, planning, memory, trust. Nobody else has the OS shell.
- **Hive** — the Pro tier swarm. Nobody else has the autonomous orchestration loop.
- **Brain v3** — GNN + Multi-Head Q-Net + World Model + MCTS. Our reasoning fabric.

Stoa is the *connective tissue* that lets all four reach the world. Giving it away is what makes Theron + AE OS + Hive worth more, not less. **The OS is what people pay for. Stoa is the air the OS breathes.**

---

## 3. What's actually broken in 2026

Before designing the solution, name what's wrong. Six things, all confirmed in the May 2026 frontier scan:

**3.1 Agents are clicking pixels.** Browserbase, Stagehand, Skyvern, Bytebot, Anthropic Computer Use, OpenAI Operator — every one of them is a workaround for the absence of an agent-native surface. They simulate humans because the SaaS has no machine declaration of what it can do. Expensive (vision API every step), slow (round-trip per click), brittle (button moves → flow breaks).

**3.2 MCP won description but lost trust.** 97M monthly SDK downloads. 10K+ public servers. Universal endorsement. *And* an "RCE by design" classified by Anthropic as "expected behavior" (OX Security, April 2026), 492 zero-auth servers exposed, 9-of-11 registries poisoned in red-team, Pentagon supply-chain risk designation, Julien Simon's "MCP ignored 40 years of RPC best practices" critique going viral. **MCP gives you a tool name and a JSON Schema input. It does not give you idempotency, typed errors, state envelopes, identity attestation, signed receipts, compensation, cost ceilings, or privacy classes.** The market wants those things and is angry they don't exist.

**3.3 Stripe solved one slice.** Stripe's Machine Payments Protocol, Agentic Commerce Suite, Sessions 2026 launches — gold standard for *payments*. There is no analog for CRM, calendar, email, search, analytics, ticketing, infrastructure, files, forms, identity, or anything else.

**3.4 OpenAI Instant Checkout failed.** September 2025 launch with Etsy + Shopify + Stripe; killed March 2026. Lesson: agent commerce is not "slap a payment primitive onto a chatbot." Agent commerce requires re-thinking the entire intent-to-fulfillment funnel.

**3.5 Compensation is bespoke.** Temporal raised $300M at $5B in February 2026. Restate, Inngest, LangGraph, Mastra all ship durable execution. **None of them ship cross-vendor saga compensation natively.** If your 5-step plan touches Stripe + Salesforce + GitHub + Twilio + a custom API and step 4 fails, you write every rollback handler by hand. The other vendors don't declare what their compensation endpoints are because the protocol has no slot for them.

**3.6 No cost oracle exists.** There is no machine-queryable feed of "what does this capability cost across vendors right now, with what P50/P99 latency, with what reliability." Agents pick tools by prose. That is irrational. That is the gap.

Stoa fixes all six.

---

## 4. The three pillars

```
        ┌────────────────────────────────────────────┐
        │              THE STANDARD                  │
        │  Stoa/1 wire envelope, capability manifest │
        │   typed I/O, side-effects, idempotency,    │
        │   rollback, scopes, errors, conformance    │
        └────────────────────────────────────────────┘
                            │
        ┌───────────────────┴────────────────────────┐
        │              THE RUNTIME                   │
        │  Stoa Edge: stateless workers, Saga DO,    │
        │  Budget DO, signed receipts, identity      │
        │  verification, live state bus, sandbox     │
        └────────────────────────────────────────────┘
                            │
        ┌───────────────────┴────────────────────────┐
        │              THE NETWORK                   │
        │  federated capability graph: signed Merkle │
        │  tree, embeddings, attestations, price     │
        │  oracle, daily bundles, no choke point     │
        └────────────────────────────────────────────┘
```

The three pillars are independent. A vendor can implement the standard without ever touching the runtime (they self-host). An agent can use the runtime without joining a particular registry (it speaks Stoa/1 to any compliant endpoint). A registry can federate without running its own runtime.

But used together, they compose into something the world has never had: a substrate where **intent → execution** is a typed, signed, replayable, audit-trailed, cost-governed, privacy-aware operation.

---

## 5. The wire — Stoa/1

Stoa/1 is the on-the-wire envelope. It sits above MCP (and above plain OpenAPI) and adds the eight things MCP/OpenAPI miss. A vendor with a clean OpenAPI spec is 80% of the way there; a vendor speaking Stoa/1 is 100%.

### 5.1 Request envelope

```jsonc
{
  "stoa": "1",
  "cap": "urn:stoa:cap:hubspot.contacts.create@2.3.1",
  "idem": "ag_7f3b...c91:plan_482:step_3",
  "agent": {
    "jwt": "eyJ...",
    "issuer": "did:web:hive.vext.ai",
    "reputation_hint": "vext-hive:tier-3"
  },
  "trace": {
    "parent": "01HW8K...",
    "plan": "plan_482",
    "step": 3
  },
  "budget": {
    "ceiling_cents": 12,
    "currency": "USD",
    "settlement": "x402-escrow:0xab..."
  },
  "privacy": {
    "input_classes": ["PII.email", "PII.name"],
    "output_classes": ["PII.email", "INTERNAL.id"],
    "jurisdiction": "US"
  },
  "resume": null,
  "input": { "email": "...", "first_name": "..." },
  "compensation": {
    "on_undo": "urn:stoa:cap:hubspot.contacts.delete@2.3.1",
    "key_path": "$.id"
  },
  "policy": {
    "require_human_confirmation": false,
    "max_retry": 3,
    "preferred_region": "us-east"
  }
}
```

**Field by field:**

- `stoa`: protocol version. `"1"` for v1. Forward-compatible within major.
- `cap`: stable capability URN (§7). Agents key off URNs, not paths.
- `idem`: idempotency key. The runtime persists `(idem) → (receipt)` so retries are exactly-once. Pattern is `<agent_id>:<plan_id>:<step_id>` but any UUID works.
- `agent.jwt`: the hive-issued JWT (§8). Signed by an issuer in the foundation allowlist.
- `agent.issuer`: the issuing hive's DID. Vendors check against their accepted-issuer policy.
- `trace`: distributed tracing context. Same shape as W3C Trace Context.
- `budget.ceiling_cents`: hard kill at this amount. `402 Payment Required` if exceeded.
- `budget.settlement`: how to actually pay. Multi-rail (§10).
- `privacy.input_classes` / `output_classes`: declared field-level privacy. Routes through compliant infra (§13).
- `privacy.jurisdiction`: country/region of the data subject. PHI/PII routing depends on it.
- `resume`: continuation token from a prior partial response (long-running ops).
- `input`: typed payload, validates against the capability schema.
- `compensation`: agent-supplied override of the graph-declared rollback. Optional; default comes from the capability graph.
- `policy`: per-call agent policy. `require_human_confirmation` triggers an in-loop approval primitive (§15).

### 5.2 Response envelope

```jsonc
{
  "stoa": "1",
  "status": "ok",
  "receipt": {
    "alg": "ES256",
    "sig": "MEUCIQ...",
    "vendor_did": "did:web:hubspot.com",
    "merkle_root": "0x9f...",
    "merkle_proof": "...",
    "ts": 1715212345,
    "input_hash": "sha256:0xab...",
    "output_hash": "sha256:0xcd..."
  },
  "state_delta": {
    "resource": "urn:stoa:res:hubspot.contact:84021",
    "version": 17,
    "etag": "W/\"a4b...\"",
    "changeset": [{"op": "create", "path": "/", "value_hash": "0xef..."}]
  },
  "continuation": null,
  "cost": {
    "actual_cents": 8,
    "breakdown": [
      {"kind": "vendor.api", "amount_cents": 6},
      {"kind": "stoa.runtime", "amount_cents": 2}
    ],
    "settlement_ref": "stripe:pi_3OK..."
  },
  "side_effects": [
    {
      "kind": "external.email_will_send",
      "when": "T+30s",
      "undo": "urn:stoa:cap:hubspot.contacts.suppress@2.3.1",
      "undo_args": {"id": "84021"}
    }
  ],
  "warnings": [
    { "code": "deprecated_field", "field": "phone_legacy", "removal": "2027-01-01" }
  ],
  "lineage": {
    "consumed_resources": ["urn:stoa:res:posthog.event_query:f3b..."],
    "produced_resource": "urn:stoa:res:hubspot.contact:84021"
  },
  "output": { "id": "84021", "email": "..." }
}
```

**The eight things this adds beyond MCP/OpenAPI:**

1. **`receipt`** — a JWS-detached signature over `(cap, agent, ts, cost, input_hash, output_hash, state_delta_hash)`. Anchored daily to a public Merkle root (Sigstore Rekor / EAS-style). Verifiable by anyone with the vendor's DID document. **Third-party auditable. Replayable. Tamper-evident.**
2. **`state_delta`** — the resource that changed, its new version, the changeset. Consumed by the live state bus (§12) so other agents subscribed to that resource get the delta.
3. **`continuation`** — opaque token for paginated/streaming responses. Servers MUST accept it verbatim; ordering is preserved.
4. **`cost`** — actual amount spent vs. declared ceiling, broken down by component, with a settlement reference for reconciliation.
5. **`side_effects`** — manifest of what will happen outside the resource graph (emails sent, money charged, jobs queued) with undo capability URN where applicable. Saga compensation walks this.
6. **`warnings`** — non-fatal advisories. Deprecations, partial results, fallback notes.
7. **`lineage`** — what resources this call consumed and produced. Builds the data lineage graph (§16).
8. **`output`** — the actual result, typed per capability schema.

### 5.3 Error envelope

Every error response follows:

```jsonc
{
  "stoa": "1",
  "status": "error",
  "error": {
    "code": "duplicate_email",
    "message": "A contact with this email already exists",
    "remediation": {
      "hint": "search-then-update",
      "next_capability": "urn:stoa:cap:hubspot.contacts.search@2.3.1",
      "retry_after_ms": null,
      "compose_hint": null
    },
    "trace_id": "trc_01HK...",
    "details": { "existing_id": "84019" }
  },
  "receipt": { /* even errors get receipts, for audit */ },
  "envelope": { "id": null, "as_of": "2026-04-15T12:34:56Z" }
}
```

Reserved codes: `unauthenticated`, `forbidden`, `not_found`, `validation_failed`, `rate_limited`, `idempotency_conflict`, `state_conflict`, `service_unavailable`, `cost_limit_exceeded`, `privacy_zone_mismatch`, `human_confirmation_required`, `compensation_required`.

Remediation hints: `fix-input-and-retry`, `backoff`, `auth-refresh`, `escalate-to-user`, `permanent-failure`, `search-then-update`, `search-then-merge`, `wait-and-poll`, `compose-different-capabilities`, `request-budget-increase`, `route-to-compliant-vendor`.

---

## 6. The runtime — Stoa Edge

Stoa Edge is the layer between the agent and the SaaS. It is a stateless edge mesh that handles execution: routing, idempotency, sagas, budgets, identity verification, signed receipts, live state, privacy, and observability.

### 6.1 Why edge

An agent plan calls 8 vendors. You want the saga coordinator at <50ms RTT to all of them, not pinned to one region. Edge-deployed workers in 300+ POPs (Cloudflare Workers, Deno Deploy, Vercel Edge Functions, RunPod Serverless for heavier compute) put the runtime as close as possible to both the agent and the vendor.

### 6.2 Why mostly stateless

Runtime workers are stateless. The two things that must be stateful — saga checkpoints and per-agent budget meters — live in **Cloudflare Durable Objects** (or self-hosted Postgres + advisory locks for forks). DOs give you single-writer-per-key without standing up a database. Per `plan_id` there is exactly one Saga DO; per `agent_id` there is exactly one Budget DO.

```
   ┌──────────┐      ┌─────────────────────┐      ┌──────────┐
   │  Agent   │─────▶│  Stoa Edge Worker   │─────▶│  Vendor  │
   └──────────┘      │  (stateless)        │      └──────────┘
                     │                     │
                     │  ┌──────────────┐   │      ┌──────────┐
                     │  │  Saga DO     │───┼─────▶│  Receipt │
                     │  │  (plan_id)   │   │      │  Log (R2 │
                     │  └──────────────┘   │      │  + CT)   │
                     │  ┌──────────────┐   │      └──────────┘
                     │  │  Budget DO   │   │      ┌──────────┐
                     │  │  (agent_id)  │   │      │ Cap Graph│
                     │  └──────────────┘   │      │ (KV+R2)  │
                     │  ┌──────────────┐   │      └──────────┘
                     │  │  Subscript.  │   │      ┌──────────┐
                     │  │  DO (res_urn)│───┼─────▶│ State Bus│
                     │  └──────────────┘   │      │ (SSE/WP) │
                     └─────────────────────┘      └──────────┘
```

### 6.3 Idempotency without state

The worker is stateless; idempotency lives in two places:

- The vendor honors `Idempotency-Key` (Stripe-style — Stoa mandates it for L2+ conformance).
- The Saga DO records `(plan_id, step) → receipt` so a retry that lands on a different worker still finds the prior result.

Belt-and-suspenders: even if both the worker and the DO are amnesic, the receipt log (anchored daily to a public Merkle root) is the source of truth.

### 6.4 Routing — URN to endpoint

The `cap` URN is the address. The capability graph (§7) resolves URN → endpoint URL + public key + price + privacy zone + reliability score. The worker holds a 60s LRU of resolutions. Vendor failures bump the URN's reliability score, which feeds into planner re-routing on the next call.

For caps with multiple implementations (e.g., two vendors implement `urn:stoa:cap:email.send`), the worker selects by per-agent policy: cheapest / fastest / highest-rep / explicit-vendor.

### 6.5 Capability graph caching

Each region's edge node holds a signed snapshot in CF KV (~5MB compressed). Updates are diff-patches over R2 like a git pack — `caps-2026-05-10.zstd` references parents, so cold start is one R2 GET. Verification happens once at cache-load, not per request.

### 6.6 The five Stoa Edge module boundaries

The runtime is split into five logical modules, all wire-isolated, all forkable:

1. **`stoa-edge-router`** — request envelope parsing, JWT verification, URN resolution, vendor request fan-out, response envelope assembly.
2. **`stoa-edge-saga`** — Saga DO implementation, checkpoint persistence, compensation walking, recovery on partial failure.
3. **`stoa-edge-budget`** — Budget DO, cost reconciliation, hard-kill at ceiling, settlement adapter dispatch.
4. **`stoa-edge-receipts`** — receipt issuance, daily Merkle anchoring, cross-registry receipt log mirroring.
5. **`stoa-edge-bus`** — state-delta SSE/WebPush fanout, Bloom-filter prefix subscriptions, vendor SDK to publish deltas.

Each can be replaced independently behind the Stoa/1 wire. A vendor running on AWS Lambda or Fly.io can implement any subset and be conformant.

---

## 7. The network — the federated capability graph

The network is the **address book of every capability that exists**. Federated, signed, semantic, no choke point.

### 7.1 The capability node

Every capability is a node:

```jsonc
{
  "urn": "urn:stoa:cap:hubspot.contacts.create@2.3.1",
  "vendor_did": "did:web:hubspot.com",
  "schema": {
    "input_ref": "https://schemas.hubspot.com/contacts/create-input.json",
    "output_ref": "https://schemas.hubspot.com/contacts/create-output.json",
    "schema_hash": "sha256:0xab..."
  },
  "embedding": [0.012, -0.034, ...],   // 768-d, for NL → cap retrieval
  "attestations": [
    {"by": "did:web:stoa.foundation", "kind": "conformance:L3", "sig": "..."},
    {"by": "did:web:vext.ai", "kind": "compatibility:tested", "sig": "..."},
    {"by": "did:web:hubspot.com", "kind": "self", "sig": "..."}
  ],
  "price": {
    "oracle": "x402:price-feed-v1",
    "current_cents": 8,
    "stale_after": 300,
    "tiers": [...]
  },
  "reliability": {
    "window_24h": 0.997,
    "p50_latency_ms": 84,
    "p95_latency_ms": 142,
    "p99_latency_ms": 312,
    "samples": 18204
  },
  "privacy_zones": ["US", "EU"],
  "compensation": "urn:stoa:cap:hubspot.contacts.delete@2.3.1",
  "side_effect_class": "external.crm.write",
  "scopes_required": ["hubspot:contacts:write"],
  "human_confirmation_class": "none",
  "deprecation": null
}
```

### 7.2 Storage and federation

Content-addressed blobs in R2 (Iroh/IPFS-style); index in a federated registry. Each registry is a signed Merkle tree (Sigstore Rekor + AT Protocol PDS hybrid). Vext runs one. HubSpot can run their own. OpenAI can run one for their app store. The Linux Foundation runs the canonical foundation registry.

Cross-registry references are URNs + signed pointers. **No central choke point.**

### 7.3 Conflict resolution

Two registries claim the same URN? Resolved by attestation strength:

1. Foundation attestation > vendor self-attestation > third-party attestation
2. Newer attestation > older
3. Signed by issuer in agent's accepted-issuer policy > unsigned

Agents pick conflict-resolution policy per workspace.

### 7.4 Discovery — semantic + structural

**Semantic:** agent embeds the natural-language query, runs cosine search over the embedding column locally. Top-50 capability URNs returned in <10ms with no network round-trip (the bundle is local).

**Structural:** agent issues a graph query: *"find all caps with side_effect_class=external.email.send in US privacy zone with reliability > 0.99 and price < $0.01."* Returns ranked candidates.

**Compositional:** agent says *"build a plan that ends in `urn:stoa:cap:cal.events.create`"* — runtime walks the dependency graph backwards, generating possible plans (e.g., `posthog.events.query → hubspot.contacts.search → cal.events.find_slot → cal.events.create`).

### 7.5 Daily signed bundles

Each registry publishes daily:

```
https://caps.stoa.foundation/2026-05-10/full.tar.zst        (5.2MB)
https://caps.stoa.foundation/2026-05-10/diff-from-2026-05-09.tar.zst  (84KB)
https://caps.stoa.foundation/2026-05-10/full.tar.zst.sig    (foundation ES256)
```

Bundle layout:

```
caps/
  hubspot.contacts.create@2.3.1.json     (schema + embedding + price + DID)
  resend.email.send@1.4.0.json
  ...
manifest.json   (URN → file, with content hashes)
roots/
  foundation.sig
  vext-hive.sig
  ...
```

A Cursor on a plane validates the foundation sig, mounts as in-process index, plans entirely offline. Embeddings local, LLM local, the only network event is *executing* the plan. Diff updates use git-pack-style delta encoding, ~80KB/day typical.

---

## 8. Identity — DIDs, hives, JWTs, reputation

Agents are not anonymous. Each agent presents a JWT signed by its **hive** — Anthropic Skills, OpenAI Apps, Vext Hive, Cursor Agents, Devin, custom. The hive is a `did:web` issuer.

### 8.1 The hive JWT

```jsonc
{
  "iss": "did:web:hive.vext.ai",
  "sub": "agent:vext:cyber-7:8492",
  "aud": "did:web:hubspot.com",
  "scope": ["caps:hubspot.contacts.*", "caps:hubspot.deals.read"],
  "rep_class": "tier-3",
  "jit_policy": "ttl=300s,bind=ip+ja3",
  "exp": 1715215000,
  "iat": 1715214700,
  "cnf": { "jwk": {...} },                  // proof-of-possession key
  "act_as": { "user_id": "usr_01HK", "consent_token": "..." },
  "model_attestation": {
    "model_id": "theron-cyber-v71a",
    "model_hash": "sha256:0xab...",
    "system_prompt_hash": "sha256:0xcd..."
  }
}
```

### 8.2 Trust — the issuer allowlist

Vendors maintain an issuer allowlist. Out of the box, the Stoa Foundation publishes a default list (the major hives, signed). Vendors can add/remove and write policies (`accept:tier-2+`, `deny:hive.unknown`, `require:model_attestation`).

**The allowlist itself is open and forkable.** The foundation's list is one of N. Enterprise customers maintain their own. Conflicts: agent presents JWT from issuer not in vendor's list → `forbidden` with hint to use a different hive.

### 8.3 Reputation

Reputation accrues per-hive at the foundation registry. Every receipt the vendor returns is co-signed by the agent and posted to a public reputation log (Certificate Transparency-style append-only). The foundation aggregates:

```
hive: did:web:hive.vext.ai
  total_actions:  18,402,193
  disputed:           14,201
  refunded:           42,108
  rep_score:        99.4 (tier-3)
  per_class:
    crm.*:   99.7
    email.*: 98.9
    finance.*: 99.8
```

New hives bootstrap by paying a stake (x402 escrow) that gets slashed on adversarial behavior.

### 8.4 Revocation

Short-lived tokens (5 min default) make CRLs unnecessary at the vendor. Hives publish a *misbehavior log* — if an agent goes rogue mid-session, the hive posts to the log and vendors stop accepting that `sub` within seconds via webhook.

### 8.5 Rotation

Keys rotate on a schedule baked into `did:web` documents. Vendors cache for `Cache-Control: max-age` and revalidate on signature mismatch. Rotation failure = vendor falls back to last-known-good.

### 8.6 Audit

Every JWT use leaves a receipt. The receipt log is the audit trail.

### 8.7 The model attestation primitive

The hardest unsolved problem in agent identity is: **how does the vendor know the agent is actually running the model it claims, with the system prompt it claims, on behalf of the user it claims?** Auth0 CIBA solves human-in-the-loop approval. Stoa adds **model attestation**.

The hive includes in the JWT:

- `model_attestation.model_id` — which model is running.
- `model_attestation.model_hash` — content hash of the model weights (or its TEE attestation).
- `model_attestation.system_prompt_hash` — hash of the system prompt at session start.

For TEE-rooted hives (Phala, Marlin, AWS Nitro), this is a verifiable attestation. For non-TEE hives, it is a *claim* the hive signs and stakes reputation against. The vendor's policy decides what level of attestation it requires for each capability class. **`finance.*` caps require TEE attestation; `crm.read` caps accept unattested claims with sufficient reputation.**

This is the bridge between the crypto-native verifiable-compute world and the enterprise compliance world. Nobody else has shipped it.

---

## 9. Sagas & compensation

Multi-step plans across multiple vendors. Failure mid-plan rolls back automatically. **Compensations are declared in the capability graph itself**, so the agent doesn't write them.

### 9.1 The plan declaration

```jsonc
{
  "plan_id": "plan_482",
  "agent": "did:web:hive.vext.ai#cyber-7",
  "budget_ceiling_cents": 250,
  "max_wall_seconds": 600,
  "steps": [
    {
      "id": 1,
      "cap": "urn:stoa:cap:posthog.events.query@2.0.0",
      "input": { "event": "pricing_page_view", "since": "-7d" }
    },
    {
      "id": 2,
      "cap": "urn:stoa:cap:hubspot.contacts.search@2.3.1",
      "input_from": { "emails": "$.steps[1].output.distinct_emails" }
    },
    {
      "id": 3,
      "cap": "urn:stoa:cap:cal.events.create@1.5.2",
      "fan_out": "$.steps[2].output.contacts",
      "input_from": { "invitee": "$item.email", "duration_min": 30 }
    }
  ],
  "on_failure": "compensate",
  "on_partial_success": "report_and_continue"
}
```

### 9.2 Execution semantics

The Saga DO records each step's receipt in append-only fashion. State persists across worker deaths. On step-3 failure for prospect 17:

1. Saga DO walks backwards, looks up `compensation` on each completed step's cap (declared in graph).
2. Applies `cal.events.cancel` with prospect 17's `cal_event_id`.
3. Records the compensation receipt.
4. If the original step was within a fan-out, only that branch compensates; siblings continue.

### 9.3 Recovery

If the worker dies mid-saga, the next request to the same `plan_id` lands on the same DO (Cloudflare's request routing keys by name), reads the checkpoint, and resumes. If the agent process dies, the DO times out idle sagas and either auto-compensates or surfaces a "needs-decision" event to the agent's hive.

### 9.4 Live UI

The agent's UI subscribes to `plan_482` over SSE. Each step emits `step.started`, `step.ok`, `step.failed`, `compensation.started`, `compensation.ok`. The user sees a live timeline with a one-click "abort + rollback" button that calls `urn:stoa:plan:abort`.

### 9.5 Side-effect honesty

Emails sent are *almost* uncompensable. The saga records `kind: external.email_will_send, undo: suppress` and the runtime suppresses delivery if compensation runs within 30s. Beyond that window the receipt notes "non-revertible side effect persisted" and the user is notified explicitly.

For irrevocably destructive actions (a wire transfer that has cleared, a physical shipment that has dispatched), compensation cap can be `none` and the manifest must declare `requires_human_confirmation: true` so the agent gets a hard gate before commit.

### 9.6 Compensation declaration in the graph

Every cap in the graph declares its compensation, if any:

```jsonc
{
  "urn": "urn:stoa:cap:stripe.charges.create@2.0.0",
  "compensation": {
    "cap": "urn:stoa:cap:stripe.charges.refund@2.0.0",
    "key_path": "$.id",
    "constraints": [
      {"kind": "time", "max_seconds_after": 7776000},
      {"kind": "vendor_state", "must_be": "succeeded"}
    ]
  }
}
```

**This is the unique Stoa primitive nobody else has.** Temporal makes you write this code. Stoa lets the cap publisher declare it once, and every plan that uses the cap gets compensation for free.

---

## 10. Cost & settlement — multi-rail

Every cap declares a price, signed and stale-checked. The agent presents a budget ceiling per call and per plan. The Budget DO debits actual costs and hard-kills at the limit.

### 10.1 Settlement rails (all surfaced via `settlement` field)

| Rail | Use case | Format |
|---|---|---|
| `stripe:cust_xyz` | Customer-of-record monthly invoicing | Stripe Customer ID |
| `stripe-connect:acct_xyz` | Per-vendor payout via Connect | Stripe Connect account |
| `x402-escrow:0xab...` | Agent escrow contract; per-call micropayment; receipts settle on-chain hourly | EVM contract address |
| `ach-net30` | Traditional B2B invoice | ACH details negotiated out-of-band |
| `prepaid:stoa-credits` | Stoa Foundation float; vendors paid weekly | Foundation account |
| `sap-procurement` | Enterprise procurement integration (SAP Ariba etc.) | Procurement reference |

### 10.2 The price oracle

Per-cap pricing entries are signed by the vendor with a freshness timestamp. Agents reject prices older than `stale_after`. Agents can subscribe to price changes via the live state bus (§12) for plans that span minutes-to-hours.

For caps with surge pricing (think: rideshare API), the vendor publishes a price function: `current = base + surge_multiplier * activity_index`. The graph stores the function; the worker evaluates at call time and includes the resolved price in the receipt.

### 10.3 The Budget DO

```
Per agent_id:
  total_budget_cents:    10000
  spent_cents:            4203
  open_holds_cents:        450
  available_cents:        5347
```

Each step's worker calls `BudgetDO.charge(cost)` before forwarding. On exceed, returns `402 Payment Required` with `cap:` reference to `urn:stoa:cap:stoa.budget.increase` so the agent can request a top-up from the user (or a different funding source).

### 10.4 Cross-rail settlement

A plan that spans Stripe-customer, x402-escrow, and ACH-invoice settlements is fine. Each step's settlement is recorded independently on the receipt. The Saga DO surfaces a unified cost summary; the underlying rails settle on their own clocks.

### 10.5 No vendor lock-in to a payments provider

Stoa is **not Stripe-only**. Stripe is a default rail because it's the easiest to integrate, but x402, ACH, prepaid foundation credits, and procurement systems are first-class. The runtime can route the same logical capability through different settlement rails per agent policy.

---

## 11. Receipts & audit

Receipts are the audit trail and the bill, in one artifact.

### 11.1 What's in a receipt

```jsonc
{
  "alg": "ES256",
  "sig": "MEUCIQ...",                      // detached JWS over the body
  "vendor_did": "did:web:hubspot.com",
  "agent_co_sig": "MEUCIQ...",             // agent counter-signs
  "ts": 1715212345,
  "cap": "urn:stoa:cap:hubspot.contacts.create@2.3.1",
  "input_hash": "sha256:0xab...",           // not the input itself
  "output_hash": "sha256:0xcd...",
  "state_delta_hash": "sha256:0xef...",
  "cost_actual_cents": 8,
  "settlement_ref": "stripe:pi_3OK...",
  "trace_id": "trc_01HK...",
  "merkle_root": "0x9f...",                 // root of the daily log this receipt is anchored in
  "merkle_proof": ["0xab...", "0xcd...", ...]   // proof of inclusion
}
```

Receipts contain hashes, not raw data, by default. Privacy classes (§13) determine what's hashable vs. what's redactable.

### 11.2 Daily Merkle anchoring

Every 24 hours, all receipts from all participating vendors are merkle-rooted into a public log. The foundation publishes `daily_root_2026-05-13.sig`. Anyone with the daily root + a receipt + its merkle proof can verify inclusion **even if the vendor has gone bankrupt**.

This is Sigstore Rekor for the agent economy.

### 11.3 Verification

```bash
$ stoa verify receipts.jsonl --root daily_root_2026-05-13.sig
✓ 412 receipts verified
✗ 0 invalid signatures
✗ 0 missing merkle proofs
✗ 0 root mismatches
```

### 11.4 Enterprise SIEM bridge

Stoa receipts ingest natively into Splunk, Datadog, Snowflake, S3-via-Firehose, or any log destination via the **Stoa SIEM Connector** (Apache-2.0, in `stoa-sdk`). Receipts become structured events; security teams can query "all agent actions on `*.finance.*` capabilities by vendor X user Y in the last 30 days" with regular SIEM queries. **This is the first time enterprise compliance can ingest agent activity natively.**

### 11.5 Dispute resolution

Any party — buyer, vendor, or third party — can file a dispute against a receipt by posting to the foundation's dispute log. Disputes link to the receipt's merkle proof, the alleged issue, and supporting evidence (other receipts, vendor logs, etc.). Resolution is human-arbitrated for v1; future versions can plug in arbitration capabilities (`urn:stoa:cap:stoa.dispute.arbitrate`).

---

## 12. Live state bus

Polling is dead. Each capability that returns a `state_delta` opts the resource into the state bus.

### 12.1 Subscriptions

Agents subscribe by URN:

```http
GET /v1/subscribe?res=urn:stoa:res:hubspot.contact:84021&since=v17
Accept: text/event-stream
Authorization: Bearer <hive_jwt>
```

Vendor pushes (via Stoa Edge fanout) when the resource changes via *any* path — UI, API, another agent:

```
event: delta
data: {"urn":"...:84021","version":18,"changeset":[{"op":"replace","path":"/email","old_hash":"0xa","new_hash":"0xb"}],"by":"agent:openai:apps#42"}
```

The agent's planner sees the delta, marks `plan_482` as *invalidated* if the resource was a dependency, and triggers recompose. Recompose either replays from the affected step or asks the user for a decision.

### 12.2 Prefix subscriptions

For massive fanout — a contact list of 50K resources — clients subscribe to a *prefix* (`urn:stoa:res:hubspot.contact:*`) with a Bloom-filter intersection so the vendor only pushes when something the client cares about changes.

```http
GET /v1/subscribe?prefix=urn:stoa:res:hubspot.contact:*&filter=bloom-base64-...
```

### 12.3 WebPush for offline clients

Cursor closed the laptop. WebPush wakes the client with the bare delta; planner re-engages on resume.

### 12.4 Vendor SDK to publish deltas

Vendors emit deltas via:

```python
from stoa import bus

bus.publish(
    res="urn:stoa:res:hubspot.contact:84021",
    version=18,
    changeset=[{"op": "replace", "path": "/email", "old_hash": "0xa", "new_hash": "0xb"}],
    by="agent:openai:apps#42",
)
```

This is the missing primitive that turns webhooks-as-an-afterthought into a first-class capability of every Stoa-conformant SaaS.

### 12.5 Backpressure & reliability

Subscribers get sequence numbers per resource. Missed deltas trigger a `resync` that returns the current full resource state plus all deltas since `since`. Vendors retain a 24h delta log per resource by default; longer windows are a paid tier on the foundation registry.

---

## 13. Privacy classes

Every input/output field declares a class. Classes propagate through the saga. The runtime enforces routing, redaction, and refusal.

### 13.1 Class taxonomy

```
PUBLIC                  — no restrictions
INTERNAL                — vendor or buyer tenant boundary
PII.email
PII.name
PII.phone
PII.address
PII.dob
PII.ssn                 — US-specific, requires high-security handling
PII.eu_personal_data    — GDPR scope
PHI.diagnosis           — HIPAA-protected
PHI.medication
PHI.treatment_plan
FINANCIAL.account
FINANCIAL.card_pan      — PCI scope (use tokens)
FINANCIAL.transaction
SECRET.credential       — never log, never persist past use
SECRET.api_key
LEGAL.privileged        — attorney-client communications
```

Vendors and buyers can register custom classes (`x-vext.attack_pattern`, `x-acme.blueprint`) — must be prefixed with `x-`.

### 13.2 What the runtime enforces

- **Routing.** A cap with `PHI.diagnosis` is only resolved to vendors whose graph entry declares `privacy_zones: ["HIPAA-US"]`. If none in the agent's accepted-vendor policy are HIPAA-zone, the call is refused with `privacy_zone_mismatch` + remediation hint.
- **No-log.** Workers refuse to write inputs/outputs to logs when `SECRET.*` or `PHI.*` classes are present; only hashes go into receipts. Disk write pressure on edge workers is a non-issue (mostly stateless), but logs are the leak vector — Stoa elides at protocol level.
- **Auto-redact.** When a saga summary is shown to the user's agent UI, PII is tokenized (`<PII.email#a4b>`) and only un-redacted on explicit query through a dedicated cap (`urn:stoa:cap:stoa.pii.unredact`).
- **Cross-zone refusal.** EU-PII input flowing to a US-only cap returns `409 PrivacyZoneMismatch` with a hint URN to the EU equivalent cap if one exists.
- **Receipt hashing.** Receipts always store hashes, not values, when classes are sensitive. Audit can prove "this PII was processed" without exposing it.

### 13.3 Why this matters

Agents cross privacy boundaries in ways humans rarely do. A human filling out a form sees the form. An agent passing data through 8 vendors might inadvertently leak PHI from a HIPAA-compliant vendor to a non-compliant one. **Stoa prevents this at protocol level.**

This is also the path to selling Stoa into regulated industries: healthcare, finance, legal, government. The receipts + privacy classes + signed audit trail are exactly what compliance officers need.

---

## 14. Offline-first bundles

Each Stoa registry publishes daily signed bundles. Clients pull, verify, and use locally.

### 14.1 The bundle

```
caps-2026-05-10.tar.zst              5.2MB
caps-2026-05-10.tar.zst.sig          (foundation ES256)
diff-from-2026-05-09.tar.zst         84KB
```

Layout inside the bundle:

```
manifest.json                        (URN → file path + content hash)
caps/
  hubspot.contacts.create@2.3.1.json
  hubspot.contacts.delete@2.3.1.json
  resend.email.send@1.4.0.json
  cal.events.create@1.5.2.json
  ...
embeddings.bin                       (concatenated 768-d vectors, indexed by URN)
embeddings.idx                       (URN → offset)
attestations/
  foundation.sig
  vext-hive.sig
  hubspot.com.sig
  ...
```

### 14.2 Why local

Agents on planes. Agents in air-gapped enterprise networks. Agents running on phones. Agents at the edge with intermittent connectivity. **Local-first means the planner runs offline; only execution requires network.**

For a Cursor user on a flight, this means: type the plan, get a local plan synthesis, get a cost estimate, queue for execution on landing. No round-trip to a remote planner.

### 14.3 Update mechanism

Daily diffs are git-pack-style delta-encoded against the parent. ~80KB/day typical. Verify foundation signature on the diff, apply, re-index embeddings (rebuild the affected `embeddings.idx` slice).

### 14.4 Local capability search

```python
from stoa.local import index

idx = index.load("./caps-2026-05-10")
results = idx.search(
    query="schedule a demo with someone from the contact list",
    top_k=10,
    privacy_zone="US",
    max_price_cents=5
)
# returns ranked list of capability URNs with metadata
```

### 14.5 Fallback to remote

If the local bundle is older than `max_age` (configurable), the SDK transparently falls back to the foundation registry's HTTPS API. Offline-first, not offline-only.

---

## 15. Composition primitives — plans, bundles, dependencies

The wire envelope handles single calls. Real agent work is multi-call. Stoa composes via four primitives.

### 15.1 Declarative plans

(See §9.1 for shape.) A plan is a DAG of steps with conditional branching, fan-out, fan-in, and on-failure semantics. Plans are themselves resources (`urn:stoa:res:plan:plan_482`) and have receipts.

### 15.2 Capability bundles

A vendor or third party can publish a *bundle* — a named composite that exposes N caps as one logical unit. Example:

```jsonc
{
  "urn": "urn:stoa:bundle:vext.checkout-flow@1.0.0",
  "summary": "Standard ecommerce checkout: validate cart → charge card → confirm fulfillment → receipt",
  "exposed_cap": "urn:stoa:cap:vext.checkout.execute@1.0.0",
  "internal_plan": [
    {"id": 1, "cap": "urn:stoa:cap:cart.validate@2.0.0"},
    {"id": 2, "cap": "urn:stoa:cap:stripe.charges.create@2.0.0"},
    {"id": 3, "cap": "urn:stoa:cap:fulfillment.confirm@1.5.0"},
    {"id": 4, "cap": "urn:stoa:cap:stripe.receipts.send@1.0.0"}
  ],
  "compensation_built_in": true
}
```

Bundles let the agent call one URN and get a transactional, compensable, signed-receipt-bearing composite operation.

### 15.3 Dependency declarations

A cap can declare prerequisites:

```jsonc
{
  "urn": "urn:stoa:cap:stripe.invoices.send@2.0.0",
  "depends_on_resource": [
    {"urn_pattern": "urn:stoa:res:stripe.charge:*", "where": "$.status == 'succeeded'"}
  ]
}
```

The runtime checks the dependency graph before allowing the call. If unmet, returns `precondition_failed` with the missing resource type. Agents can then plan a satisfying sub-plan.

### 15.4 Human-in-the-loop primitive

```jsonc
{
  "urn": "urn:stoa:cap:stoa.confirm.user@1.0.0",
  "summary": "Block until the human-of-record confirms or rejects",
  "inputs": {
    "prompt": "string",
    "options": ["array", "of", "choices"],
    "expires_in_seconds": "integer"
  },
  "outputs": {
    "choice": "string",
    "confirmed_at": "datetime",
    "confirmer_did": "string"
  }
}
```

Caps can declare `human_confirmation_class` in their manifest:

- `none` — no confirmation needed
- `notify` — fire-and-forget user notification, proceed
- `gate` — block until user approves (the cap's response only returns after `stoa.confirm.user` resolves)
- `hard_gate` — gate AND log the confirmation receipt as a separate audit artifact (used for irreversible actions)

The runtime injects a `stoa.confirm.user` step transparently for `gate`/`hard_gate` caps. The human approves through whatever channel their hive provides (Slack DM, push notification, AE OS prompt, etc.).

### 15.5 Cross-cap data flow

Plans declare `input_from` references with JSON-Path expressions over the saga state. The runtime resolves them at step execution time. This is the only way data flows between steps — there is no global mutable state.

---

## 16. Sandbox, replay, lineage

Three primitives that fall out of the receipt model.

### 16.1 Sandbox

```python
plan = stoa.Plan(...)
result = stoa.sandbox.run(plan, vendors_recorded_at="2026-05-10")
```

The runtime replays the plan against a recorded snapshot of vendor responses. No real side effects. Useful for: pre-flight cost estimation, regression testing of agent behavior, training data generation.

Vendors opt in by publishing `urn:stoa:cap:<vendor>.sandbox.snapshot` capabilities the foundation runs daily.

### 16.2 Replay

Receipts are replayable. Given a list of receipts and the daily Merkle roots they're anchored to, anyone can reconstruct the exact sequence of agent actions, with input/output hashes, costs, and timestamps. **Reproducibility is a feature.**

```bash
$ stoa replay receipts-2026-05-10.jsonl --redact-classes "PII.*,PHI.*"
[plan_482]
  step 1: posthog.events.query  → 142 events  $0.01  240ms
  step 2: hubspot.contacts.search → 38 matches $0.02  180ms
  ...
```

### 16.3 Lineage

Every receipt includes `lineage.consumed_resources` and `lineage.produced_resource`. Walking the receipt log builds the data lineage graph: *"this email sent to prospect 17 was produced by step 4c of plan_482, which consumed the contact list from step 2, which consumed the events from step 1."* For data governance, this is the receipt of where data went.

---

## 17. Adapter framework — for non-native vendors

Most SaaS vendors don't ship Stoa contracts on day 1. The adapter framework lets a third party (Vext, the foundation, a community contributor, the buyer themselves) wrap an existing vendor's API into a Stoa surface.

### 17.1 The adapter

```typescript
import { StoaAdapter } from "@stoa/sdk";

export const slackAdapter = new StoaAdapter({
  vendor_did: "did:web:slack.com",
  attestation_kind: "third-party:vext.ai",
  capabilities: [
    {
      urn: "urn:stoa:cap:slack.message.post@1.0.0",
      input_schema: { ... },
      output_schema: { ... },
      side_effect: { kind: "external.message.send", undo: "slack.message.delete" },
      handler: async (input, ctx) => {
        const res = await fetch("https://slack.com/api/chat.postMessage", { ... });
        return ctx.envelope({ output: res, state_delta: {...} });
      }
    }
  ]
});
```

Adapters can run as:

- **Hosted by the foundation** — Vext-attested at launch; community-attested later.
- **Self-hosted by the buyer** — runs in the buyer's infrastructure for compliance-strict deployments.
- **Hosted by the vendor as a stepping-stone** — vendor adopts Stoa at adapter level first, then promotes to native conformance.

### 17.2 OpenAPI → Stoa generator

`stoa.from-openapi` takes any OpenAPI 3.x doc and emits a Stoa adapter skeleton: capability URNs, side-effect annotations (heuristically inferred from HTTP method), idempotency declarations, scope tokens, error mappings. **80% of work done automatically; vendor reviews the 20%.**

This is the primitive that turns "we should ship Stoa" into "we shipped Stoa by Friday."

### 17.3 Webhook → state bus adapter

Existing webhooks become Stoa state-delta streams via:

```typescript
import { webhookToBus } from "@stoa/sdk";

webhookToBus({
  vendor: "stripe",
  webhook_path: "/webhooks/stripe",
  resource_mapping: {
    "charge.succeeded": (event) => ({
      urn: `urn:stoa:res:stripe.charge:${event.data.object.id}`,
      version: event.data.object.metadata.version,
      changeset: [...]
    })
  }
});
```

Every existing webhook in every existing SaaS becomes a Stoa state-delta stream automatically.

---

## 18. Versioning & deprecation

### 18.1 Capability versioning

URNs include semver: `urn:stoa:cap:hubspot.contacts.create@2.3.1`.

- **Patch (2.3.1 → 2.3.2):** bug fix, no behavior change. Agents auto-upgrade.
- **Minor (2.3 → 2.4):** additive change (new optional input field, new output field). Agents auto-upgrade if their schema validates.
- **Major (2 → 3):** breaking change. Old version remains live for 12 months minimum (deprecation window). New version is a separate URN.

### 18.2 Deprecation

The capability graph entry includes:

```jsonc
{
  "deprecation": {
    "deprecated_at": "2026-04-01T00:00:00Z",
    "removal_at": "2027-04-01T00:00:00Z",
    "successor": "urn:stoa:cap:hubspot.contacts.create@3.0.0",
    "migration_guide": "https://hubspot.com/docs/stoa-migration-2-to-3.md"
  }
}
```

Agents calling deprecated caps get a `warning` in the response envelope. The successor is in the warning. Auto-migration tools in the SDK convert old plans to use new caps where possible.

### 18.3 Spec versioning

Stoa/1 is the wire envelope version. Stoa/2 would be a major spec change. Vendor manifests declare which wire versions they speak; agents pick the highest mutually supported.

Spec versions hand off to the foundation via RFC. Vext is editor of v0.1 → v1.0; foundation governs v2.0+.

---

## 19. Conformance levels

Five levels. Vendors self-declare or are foundation-certified.

| Level | Name | Requires |
|---|---|---|
| **L0** | Discoverable | Discovery doc at `/.well-known/stoa.json`; openapi or mcp endpoint declared |
| **L1** | Typed | L0 + capability manifest with URNs, typed I/O, side-effect annotations |
| **L2** | Idempotent | L1 + idempotency declarations + Idempotency-Key support + typed errors with remediation hints |
| **L3** | Stateful | L2 + state envelope on all responses + opaque continuation tokens + ordering guarantees + state-delta bus |
| **L4** | Composable | L3 + capability scopes + Acting-As headers + signed receipts + compensation declarations + privacy classes + price oracle |

A vendor at L0 is a vendor anyone *can* call from an agent. A vendor at L4 is a vendor agents *should* call — full saga support, full audit trail, full compensation, full privacy.

The Stoa Foundation marketplace surfaces vendors by level. L4-conformant vendors get the prominent shelf; L1 vendors are listed but flagged.

### 19.1 Conformance test suite

`stoa-conformance` (Apache-2.0 on GitHub) is automatable and runs against a candidate Stoa host. For each declared capability:

1. Discovery roundtrip
2. Schema-level conformance (valid inputs accepted, invalid rejected with `validation_failed`)
3. Idempotency (repeat with same key returns same ID, no duplicate)
4. State envelope shape
5. Pagination determinism
6. Error envelope shape
7. Scope enforcement
8. Acting-As propagation
9. Rate-limit honesty
10. Receipt signature validity
11. State-delta emission
12. Compensation roundtrip (call cap, then call its compensation, verify rollback)
13. Privacy class enforcement (refuse cross-zone, redact in receipts)
14. Cost reconciliation (declared vs. actual)

The suite emits a JSON report. Vendors host the report at `conformance.report_url`.

---

## 20. Governance & foundation handoff

### 20.1 Editor → foundation

Vext Labs is editor of v0.1 because someone has to ship the first version. The spec moves under a foundation — most likely Linux Foundation, where MCP, A2A, and ACP already live — at the **5-vendor trigger**: when 5 independent vendors hit L4 conformance.

**Public commitment:** the foundation hand-off is real, dated, and irreversible. The 5-vendor trigger is in the v0.1 spec text. Failure to hand off after the trigger fires is grounds for any party to fork the spec under the existing CC-BY-4.0 license. Vext does not retain steward authority after hand-off.

### 20.2 RFC process

All spec changes flow through public RFCs at `github.com/stoa-spec/stoa-spec`. RFCs require:

- Implementation in at least one runtime
- Compliance test in the conformance suite
- 2-week public comment period
- Editor sign-off (pre-handoff) or foundation TSC majority (post-handoff)

### 20.3 Trademarks

The "Stoa" name and logo are held by Vext Labs pre-handoff. At handoff, both are transferred to the foundation under a trademark license that allows any conformant vendor to use them. Vext retains use of "Stoa-attested" for adapters Vext maintains.

### 20.4 Conformance authority

The foundation runs the canonical conformance test suite and issues conformance certificates. Vendors can self-declare any level; foundation certification is an additional attestation that surfaces in the marketplace.

Multiple conformance authorities can exist. The foundation is the default; enterprise customers can run their own; security firms can offer paid attestation services.

### 20.5 Anti-capture commitments

To prevent any single party (including Vext) from capturing the standard:

- **License is irrevocable.** CC-BY-4.0 + Apache-2.0 cannot be unwound.
- **Conformance suite is open.** Anyone can run their own attestation.
- **Reference adapters are forkable.** Apache-2.0.
- **Federation is mandatory.** Spec requires URN namespace can be served by any compliant registry.
- **Trademark hand-off.** At foundation transition, name and logo move with the spec.

---

## 21. The economics — why this works for Vext

Detailed breakdown lives in [ECONOMICS.md](ECONOMICS.md). Summary:

**Vext makes money on three things, none of which is Stoa itself:**

1. **Theron specialist calls** — every time an agent (anyone's agent, anywhere in the world) calls a Theron capability through Stoa, Vext earns the per-call rate. `urn:stoa:cap:vext.cyber.scan` is $0.50/req + $5/finding regardless of which runtime brokered the call. Theron is the brain; Stoa is just the wire.
2. **AE OS subscriptions** — buyers pay for the OS surface where intent becomes work. Stoa is the substrate AE OS is built on; the value is in the OS, not the substrate.
3. **Hive Pro tier** — autonomous swarm orchestration on top of Theron + AE OS + Stoa. Burn-rate billing per inference second.

**Vext does NOT make money on:**

- The Stoa spec itself (CC-BY-4.0)
- The Stoa runtime code (Apache-2.0)
- The Stoa SDK (Apache-2.0)
- The Stoa conformance suite (Apache-2.0)
- The Stoa registry implementation (Apache-2.0)
- Stoa-attested adapters Vext maintains for third-party vendors (Apache-2.0; Vext can charge for *hosting* the adapter as a managed service, but the code is open)
- Marketplace listing fees (zero)
- Conformance certification fees (foundation runs this; foundation may charge enterprise tiers)

**Why this works:**

- Free Stoa = adoption = traffic to Vext-built adapters and Theron caps
- Open runtime = trust = enterprise buys Vext-managed runtime hosting
- Open registry = federation = no anti-trust risk, no platform-tax accusations
- Vext's moat is **Theron's training stack and AE OS's integration depth**, neither of which is in Stoa

**The math:** if Stoa hits 1B agent calls/month within 18 months (plausible at MCP's 97M monthly SDK growth rate), and Theron caps capture 0.1% of those calls at $0.50 average, that's $500K/month from one cap class alone, growing exponentially with adoption. AE OS is a separate revenue stream entirely. Stoa being free **multiplies** these revenue streams; charging for Stoa would suppress them.

---

## 22. Five modules to build in parallel

Each ships independently behind the Stoa/1 wire. Each is its own GitHub repo under `github.com/stoa-spec/`. All Apache-2.0.

### 22.1 `stoa-edge`

The runtime workers. Cloudflare Workers + Durable Objects framework. Saga DO, Budget DO, Subscription DO, request envelope handling, signed-receipt issuance, settlement adapter dispatch.

**Lead deliverable:** ~2K LOC TypeScript Worker that handles end-to-end Stoa/1 calls against a real vendor with signed receipts.

**Owner:** Distributed-systems lead.

### 22.2 `stoa-graph`

The federated capability registry. R2-backed Merkle tree, embedding index, attestation chain, daily diff-bundle publisher, foundation signing, federation protocol.

**Lead deliverable:** Daily signed bundle at `caps.stoa.foundation/2026-XX-XX/full.tar.zst`.

**Owner:** Data + crypto lead.

### 22.3 `stoa-identity`

`did:web` hive issuance, JWT minting with proof-of-possession, reputation log, revocation feed, foundation issuer-allowlist, model attestation primitive.

**Lead deliverable:** Vext Hive as first issuer; foundation publishes default issuer-allowlist; vendor SDK validates JWTs against allowlist.

**Owner:** Identity/security lead.

### 22.4 `stoa-bus`

State-delta SSE/WebPush fanout. Bloom-filter prefix subscriptions. Vendor SDK to publish deltas. Webhook-to-bus adapter for legacy vendors.

**Lead deliverable:** Live demo of a single vendor (HubSpot or Cal.com) emitting state deltas to a subscribed agent in real time.

**Owner:** Realtime lead.

### 22.5 `stoa-sdk`

Client libraries (TS, Python, Rust) for agents: cap retrieval, plan execution, receipt verification, offline bundle management, sandbox replay, SIEM connector.

**Lead deliverable:** `npm install @stoa/sdk` with end-to-end example: `stoa.plan(...).execute().verify()`.

**Owner:** DX lead.

### 22.6 Plus: `stoa-conformance`

The test suite. Apache-2.0. Runs against any Stoa-conformant host.

### 22.7 Plus: `stoa-spec`

The spec itself, RFC archive, governance docs. CC-BY-4.0.

---

## 23. Six-month shipping plan

Milestone-by-milestone. All public. All open from day 1.

| Month | Spec | Runtime | Network | Demo |
|---|---|---|---|---|
| **1** | Stoa/1 v1 RFC public; rename Janus → Stoa across codebase; foundation handoff plan published | `stoa-edge` skeleton on CF Workers; one Saga DO + Budget DO; signed receipts for one cap | First registry running at `caps.stoa.foundation`; 9 Theron caps + 1 HubSpot cap | 1-step demo: HubSpot via runtime, signed receipt verifiable with `stoa verify` CLI |
| **2** | `stoa-conformance` v1 published (the missing repo) | Receipt log → daily R2 Merkle root; SIEM connector alpha | Embedding index live; cosine search in SDK | Cursor demo: NL → cap retrieval offline from local bundle |
| **3** | Conformance level certs published; first vendor gets L3 | Saga compensation walking; budget hard-kill; x402 + Stripe + agent-escrow settlement adapters | Price oracle entries; 25 caps across 10 vendors | 5-step workflow with mid-saga failure + full rollback; receipts verifiable |
| **4** | Foundation hand-off plan with named foundation candidate; first vendor RFCs land | `stoa-identity`: `did:web` hives, JWT minting, foundation issuer-allowlist, model attestation alpha | Reputation log v1; first 3 hives onboarded (Vext + 2 of OpenAI Apps / Anthropic Skills / Cursor / Devin) | Vendor accepts hive JWT in prod |
| **5** | Privacy-class spec finalized | `stoa-bus`: SSE + WebPush; Bloom-filter prefix subs; webhook-to-bus adapter | Privacy zones in graph; HIPAA-zone routing live | Live-state demo: external mutation invalidates plan; agent recomposes |
| **6** | Spec v1.0 freeze (12-month deprecation clock for breaking changes) | Full SDK (TS/Python/Rust); offline bundles GA; sandbox/replay/lineage GA | 25 production-grade adapters published; foundation handoff begun if 5-vendor trigger fires | **Public launch + founder essay + 90s film + 5-platform integration story** |

### 23.1 Week 1 actions to start

1. Decide the name. ✅ **Stoa.**
2. Rename the codebase: `janus.tryvext.com → stoa.tryvext.com`; all `janus_*` files → `stoa_*`; all "Janus" content → "Stoa".
3. Write the v1 RFC for Stoa/1 wire envelope (the JSON shapes in §5). 1,500 words. Public on day 1.
4. Stand up `stoa-edge` Worker that handles ONE capability end-to-end (`hubspot.contacts.create`) with: signed receipt, Saga DO, Budget DO, `Idempotency-Key`, x402-or-Stripe settlement, side-effect manifest. Ship the CF Worker + DO code. ~5 days.
5. Publish `caps.stoa.foundation/2026-05-13/full.tar.zst.sig` with 9 Theron caps + the 1 HubSpot cap. Daily cron. ~2 days.
6. Verifier CLI: `stoa verify receipts.jsonl --root foundation_root_2026-05-13.sig`. Anyone can run it. ~1 day.

Once those six exist, Stoa stops being a spec and starts being **the substrate**.

---

## 24. Ten things that become possible

1. **A one-person company runs a 200-person SaaS.** AE OS + Stoa + Theron means a solo founder's agent swarm can operate every SaaS the company depends on, in parallel, with full audit trail. The economics flip: headcount becomes capability budget.

2. **An agent ships its own integrations.** The first time an agent encounters a new SaaS, it pulls the SaaS's Stoa manifest, validates against the conformance suite, generates an adapter if needed, and publishes the adapter back to the foundation. The integration library grows from agent activity.

3. **Cross-vendor sagas with provable rollback.** "Charge the customer's card AND ship the order AND email the receipt — or none of the above, with receipts of the rollback." Today this is bespoke saga code; with Stoa it's a 10-line plan declaration.

4. **Compliance audit becomes one query.** "Show me every action this agent took on PHI-classed data in the last 90 days, with proof of zone-correctness." Receipts + privacy classes + Merkle anchoring make this one SIEM query.

5. **Agent identity verifiable at inference time.** A vendor receiving a finance.* call can require model attestation (TEE-rooted hash of the model + system prompt + delegation chain). Insurance companies will require this within 18 months.

6. **Capability cost competition.** Five vendors implement `urn:stoa:cap:email.send` at different prices, latencies, reliabilities. Agents pick rationally, vendors compete on quality, not on UI lock-in.

7. **The end of the integration tax.** Today every B2B SaaS has a "Connect to X / Connect to Y / Connect to Z" backlog of months of engineering. With Stoa, the integration is a 20-line adapter that auto-generates from the OpenAPI spec.

8. **Live-state-driven autonomous workflows.** An agent's plan declared 3 days ago re-engages when a calendar invite is declined, a payment fails, a shipment delays — without polling, without webhooks-to-nowhere, without losing context.

9. **A federated Internet of Capabilities.** No central marketplace. Multiple registries. Cross-resolution by URN. The same capability can be served by multiple vendors, attested by multiple authorities, settled in multiple currencies. The pattern that worked for HTTP, DNS, and email, applied to agent action.

10. **The end of the app era.** Users stop downloading apps. Stop opening apps. Stop learning UIs. The agent has the intent; Stoa has the runtime; the SaaS has the data. The UI is for humans only — and humans will increasingly skip the UI.

---

## 25. FAQ

**"Doesn't this make Vext irrelevant?"**

No. Vext owns Theron (the brain) and AE OS (the surface). Stoa is the connective tissue. Free Stoa increases Theron's reach (agents on every platform call Theron caps) and AE OS's value (the OS that consumes Stoa natively is the one that wins). Charging for Stoa would slow adoption and shrink the cake.

**"What stops OpenAI / Anthropic / Google from forking Stoa and running their own?"**

Nothing — and that's fine. The standard is genuinely open. They can publish their own marketplaces, run their own runtimes, federate with the foundation registry. Vext wins as long as Stoa wins. Vext's moat is Theron + AE OS, not the wire.

**"How is this different from MCP + Smithery?"**

MCP is a tool-calling wire format with prose descriptions. Smithery is a directory of MCP servers. Stoa adds: idempotency, typed errors, signed receipts, multi-vendor sagas with declared compensation, cost oracles, privacy classes, agent identity attestation, live state bus, conformance levels, federated registry. **MCP is the bottom layer; Stoa is everything that sits above it that the market needs and MCP refused to ship.** Stoa-conformant servers can speak MCP at the wire layer for backwards compat.

**"Why a foundation handoff? Why not just keep editing it?"**

Standards owned by single companies don't get adopted. Vendors won't commit to a contract one company can revoke. Foundation handoff is what makes Stoa real beyond Vext.

**"What about the security disasters MCP has had?"**

Stoa addresses them at protocol level. (1) Mandatory signed receipts make malicious behavior auditable. (2) Hive-issued JWTs with model attestation make agent identity verifiable. (3) Privacy classes prevent unintended data flow. (4) Conformance levels make security posture transparent. (5) Federated registry with attestation chains makes registry poisoning much harder. We will not repeat MCP's "by design" mistakes.

**"What's the status of the v0.1 codebase?"**

Janus v0.1 ships an MCP gateway, 9 Theron caps, marketplace skeleton, OpenAPI converter, Stripe Connect remittance, and a published spec. We're renaming everything to Stoa, opening every artifact, and shipping the runtime that turns the contract into a substrate over the next 6 months. Detailed plan in §23.

**"How much will this cost Vext?"**

Engineering: 5 leads × 6 months. Foundation handoff: lawyers + governance setup. Hosting: CF Workers and R2 are dirt cheap at our scale; sub-$10K/month for the first year. Compared to the upside (Theron + AE OS revenue compounding off Stoa adoption), this is the highest-leverage spend Vext can make.

**"What if no other vendors adopt it?"**

Theron itself is the first MR-SaaS. 9 specialist capabilities live on day 1. Even with zero external adopters, Stoa is the agent-callable surface for Vext's own products, and that alone justifies the spec + runtime + registry. External adoption multiplies the value; lack of it doesn't kill it.

**"Why now?"**

MCP shipped 18 months ago and hit 97M monthly downloads. Anthropic just shipped Computer Use (clicking pixels), confirming the human-UI workaround era. Stripe shipped Machine Payments Protocol (one slice). OpenAI shipped Apps SDK (one platform). Temporal raised at $5B (durable execution). The market wants the substrate; nobody has shipped the substrate. **The window is now and it closes within 12 months.**

---

## Closing

Stoa is what the world needs. Open source is the only way it gets built. Vext is the right team to ship v0.1 because we have Theron and AE OS as the first consumers, the spec experience from Janus v0.1, and the conviction that the value is in the brain and the surface, not the wire.

Six months. Five modules. Three pillars. One substrate.

The colonnaded walkway is open.

— *Stoa, by Vext Labs*

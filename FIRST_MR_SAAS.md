# The first machine-readable SaaS product

## What "the first MR-SaaS" actually means

Stoa is the standard. A "Stoa product" is any service exposed through
the standard. To be the FIRST machine-readable SaaS product means:

1. The product must do real work that produces real value (not a demo)
2. The product must be designed for agents from day one, not a wrapper
   around a human flow
3. The product must charge real money through the standard's payment
   primitive, not a side channel
4. The product must be on `/.well-known/stoa.json` as an attested
   capability with structured I/O, scopes, errors, idempotency
5. Other agents must be able to call it without reading docs — the
   manifest IS the docs

Today's `/.well-known/stoa-vext.json` has 5 capabilities, all
read-only (spec / marketplace / specialists / contact form). None do
real work. We are the publisher of the standard but not a vendor
**on** the standard yet. That changes here.

## The strategic move: Theron-as-Stoa-capabilities

We have something nobody else has: 30 specialist models trained on
primary sources, currently sold via human chat at tryvext.com. The
first MR-SaaS is **not a new product — it's a new front door for
the products we already have.**

Every Theron specialist becomes a Stoa capability with:

- Structured input schema (what the agent must provide)
- Structured output schema (what the agent gets back, with citation
  attribution per claim)
- Pricing in $/call or $/token
- Side-effect declaration (read / query / mutate / external)
- Capability scope tokens
- Stripe-backed payment via Stoa Pay
- Full conformance level (typed-state, not just core)

The marketplace listing flips from "humans pricing pages" to
"agents discovering capability manifests." Cursor / Cline / Devin /
Operator can find Theron in their MCP catalog or via Stoa
discovery and call our specialists directly.

## Why this is the right first MR-SaaS

**Defensibility.** No other Stoa vendor will have:
- 99% SecQA on cyber
- 85% IFEval on language
- 30-specialist roster trained from scratch
- The Capability Injection Protocol that ships them for $30/spec

**Dollar volume per call.** Pentest engagements run $5K-50K. Code
reviews $200-2,000. Legal review $500-5,000. These aren't $0.001
calls — each Stoa capability invocation is a real transaction.

**Network effect.** Every agent platform (Cursor, Devin, Cline,
Operator) is desperate for high-quality MCP/Stoa-compatible
domain experts to call. The first roster of 30 specialists with
real benchmarks becomes the default. The marketplace fills in
behind us.

**Self-demonstrating.** The first thing a Stoa skeptic asks is
"show me a real product on this thing." Theron specialists on
the v0.1 manifest is the answer.

## The 9 capabilities that ship as the first MR-SaaS

Each capability wraps a real Theron specialist and includes pricing,
scope, structured I/O, and the full Stoa typed-error contract.

| Capability ID | Specialist | Side-effect | Price | What an agent gets |
|---|---|---|---|---|
| `vext.cyber.scan` | Theron-Cyber | external (writes to scan target) | $0.50/req + $5/finding | Structured vuln list with CVE/CWE mapping, exploit chain, severity, attribution to specialist |
| `vext.cyber.report` | Theron-Cyber | mutate (writes to customer scope) | $50 / report | Pentest-grade report with executive summary + technical detail + reproduction + remediation |
| `vext.code.review` | Theron-Code | read | $0.10/file | Per-file findings with bug/perf/style severity + suggested patch |
| `vext.code.refactor` | Theron-Code | mutate (writes patch suggestion) | $1/refactor | Patch in unified diff format + test verification |
| `vext.legal.review` | Theron-Legal (corpus) | read | $5 / contract | Per-clause analysis with risk score + redline suggestions + jurisdiction notes |
| `vext.medical.review` | Theron-Medical (corpus) | read | $10 / chart | Differential diagnosis + ICD-10 coding + drug-interaction check |
| `vext.council.synthesize` | Full council | read (multi-spec fan-out) | $0.10/1K input + $0.40/1K output | Cross-domain answer with per-claim attribution to which specialist contributed |
| `vext.research.search` | Theron-Research (corpus) | external (web search) | $0.20/query | Citation-grounded synthesis from primary sources only — no hallucinated refs |
| `vext.classify.intent` | Theron-Language | read | $0.001/req | Routes a customer query to the right Theron specialist + confidence score |

## The cleanest demo flow

A Cursor user types "fix the auth bug" in their IDE. Cursor's agent:

1. Calls `vext.code.review` on the auth files (`$0.50` for 5 files)
2. Theron-Code returns 3 findings, one tagged "potential CSRF"
3. Cursor's agent escalates to `vext.cyber.scan` on the endpoint
4. Theron-Cyber confirms exploitable CSRF + provides PoC payload
5. Cursor's agent calls `vext.code.refactor` with the finding
6. Theron-Code returns a patch in unified diff format
7. Cursor applies the patch, runs tests, ships the fix
8. Total cost: ~$3.50, total time: ~12 seconds, zero human review

Today this is impossible because no security service is agent-
callable. Stoa + Theron-as-MR-SaaS is the first place it works.

## What we ship in week 1 to make this real

### Manifest expansion
`marketing/public/.well-known/stoa-vext.json` grows from 5 to 14
capabilities (the 5 existing read-only ones + the 9 above). Every new
entry includes `pricing`, `inputs`, `outputs`, full `errors`,
`scopes`, `idempotency`. The conformance level on the file flips from
`core` to `typed-state` because the new capabilities have full
structured I/O contracts.

### Capability handlers
`marketing/api/stoa/[...path].ts` grows handlers for the 9 new
capabilities. Each is a thin shim that:
1. Validates the input against the schema in the manifest
2. Looks up the user's Stoa key + scope token + Stripe payment intent
3. Calls the appropriate Theron specialist endpoint
4. Wraps the response in a Stoa state envelope
5. Returns it

### Pricing + payments
The capability handlers use the existing Stripe integration to
pre-authorize the call's cost, run the work, then capture or release
based on success. Refunds on failure are part of the spec's
typed-error contract.

### Marketplace listing
We list ourselves as the first attested vendor on the marketplace
with all 14 capabilities visible. The status flips from "○ FIRST"
to "★ ATTESTED" because we're now actually attesting our own work.

## What we ship in week 2-3

### Stoa Concierge
A meta-capability `vext.concierge.fulfill` that takes a natural-language
intent and orchestrates the right Theron capabilities + 3rd-party Stoa
vendors to fulfill it. This is the consumer face — agents (or humans
via tryvext.com chat) describe what they want; concierge figures out
which Stoa calls to make.

Example: `{"intent": "make sure my codebase passes SOC 2 controls"}` →
Concierge calls `vext.cyber.scan` + `vext.code.review` + (eventually)
`vext.legal.review` for policy docs + writes a SOC 2 evidence package.

### Stoa Pay (escrow + receipts)
The first capability where money flows through Stoa, not just calls.
Agents A and B can transact via `stoa.pay.escrow.create` →
`stoa.pay.escrow.release` (on confirmed delivery) or
`stoa.pay.escrow.refund` (on failure). VEXT holds escrow as the
neutral party. Theron-Legal arbitrates disputes.

### Vendor onboarding kit
`vext.stoa.from-openapi` — feed any OpenAPI 3 spec, get back a
Stoa manifest skeleton + the conformance test suite. The thing that
turns "we should be on Stoa" into "we're on Stoa by Friday" for
every SaaS vendor.

## What we ship in month 2+

- 10 partner vendors onboarded onto Stoa through the kit
- Stoa marketplace v2 with real attested ratings
- Theron-Legal arbitration for inter-vendor disputes
- Cross-vendor capability composition (`vext.concierge` orchestrates
  3rd-party Stoa calls, not just Theron)

## The pitch (for press / investors / customers)

> **Vext built the standard for machine-readable SaaS — and we're the first product on it.** Theron's 30 specialists are now agent-callable through Stoa capabilities. Cursor, Cline, Devin, and Operator can hire Theron-Cyber for pentests, Theron-Code for reviews, Theron-Legal for contracts — all through one open standard, with structured I/O, capability-scoped auth, and cryptographic proof of execution. Pricing per call, paid through Stoa Pay. The first time agents have hired agents to do regulated-domain work.

## Why this beats every other "first MR-SaaS" candidate

| Alternative | Why not this | Why Theron-as-Stoa wins |
|---|---|---|
| New booking API (Calendly-killer) | Crowded category; humans-first competitors | Theron is unique; no competitors |
| Stripe-flavored Stoa Pay | Stripe will build it themselves; we lose | Pay is layer 2; we own layer 1 first |
| "OpenAPI → Stoa" generator | Tooling, not a product. Nobody pays for it directly. | Tool ships in week 2 to feed our marketplace |
| Inter-agent escrow | No buyer side yet — too early | Comes after Theron-as-Stoa has volume |
| Concierge that orchestrates other vendors | Requires those vendors to exist | Comes after we list 10 partner vendors |
| VEXT internal dogfood (bounty pipeline) | Doesn't have customers besides us | Same architecture, but external customers from day one |

The single most defensible first move is the one that uses what we
already have (Theron) to define what only we can offer (specialist
roster on the standard we wrote). Everyone else has to build both
the model and the marketplace; we have both already.

## The honest single-line answer

**The first MR-SaaS is Theron itself, listed on Stoa with
capability-priced specialist calls — Pentest as a Capability,
Code Review as a Capability, Legal Review as a Capability —
each agent-callable, structured, paid through the standard.**

Everything else is downstream of that.

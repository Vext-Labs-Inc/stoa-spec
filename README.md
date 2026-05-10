# Stoa

**The open substrate for agent-readable SaaS.**

Stoa is the open standard, runtime, and federated registry that lets any agent — yours, ours, OpenAI's, Anthropic's, Cursor's, Devin's, the one you're about to ship — call any SaaS as a typed, signed, idempotent, cost-governed, audit-trailed capability instead of clicking through a UI it was never meant to use.

> *In the colonnaded walkway of ancient Athens, philosophers and merchants worked under one roof. Ideas became contracts. Contracts became commerce. The στοά was the substrate of how Athens did business with itself.*
>
> *Stoa is what we call the substrate of how the world does business with agents.*

---

## License

| Artifact | License |
|---|---|
| **Spec** (this document, [SPEC.md](SPEC.md), [STOA.md](STOA.md), all `.md` files) | [CC-BY-4.0](LICENSE) |
| **Reference runtime** (`stoa-edge`, coming Q3 2026) | [Apache-2.0](LICENSE-CODE) |
| **SDKs** (`stoa-sdk` — TS/Python/Rust, coming Q3 2026) | [Apache-2.0](LICENSE-CODE) |
| **Conformance suite** (`stoa-conformance`, coming Q3 2026) | [Apache-2.0](LICENSE-CODE) |
| **Federated registry** (`stoa-graph`, coming Q3 2026) | [Apache-2.0](LICENSE-CODE) |
| **Reference adapters** (`stoa-adapters`, coming Q3 2026) | [Apache-2.0](LICENSE-CODE) |

These licenses are **irrevocable**. Anyone with a copy retains rights regardless of what Vext does later. Fork freely.

---

## Read me first

If you have **30 seconds**: read the four bullets in [§1 of STOA.md](STOA.md#1-what-stoa-is).

If you have **5 minutes**: read [STOA.md](STOA.md) §1 (what Stoa is) + §2 (why open source) + §10 ("no UI to learn" loop) + §22 (five modules to build).

If you have **30 minutes**: read [STOA.md](STOA.md) end to end. ~8K words. 25 sections. Everything.

If you're a SaaS vendor: jump to [SPEC.md](SPEC.md).

If you're an agent platform (OpenAI Apps, Anthropic Skills, Cursor, Devin, Cline, Continue, Codeium, Zed, Aider, custom): jump to [STOA.md §10](STOA.md#10-the-no-ui-to-learn-loop--end-to-end) for the demo flow + [STOA.md §22](STOA.md#22-five-modules-to-build-in-parallel) for the modules to integrate.

If you're a buyer: jump to [STOA.md §11](STOA.md#11-receipts--audit) (the audit trail) + [STOA.md §13](STOA.md#13-privacy-classes) (privacy classes) + [GOVERNANCE.md](GOVERNANCE.md) (why you can trust this).

If you're an investor or industry analyst: jump to [ECONOMICS.md](ECONOMICS.md) and [FOUNDER_ESSAY.md](FOUNDER_ESSAY.md).

---

## What's in this repo

| File | What |
|---|---|
| [STOA.md](STOA.md) | **The master architecture and manifesto.** ~8K words. Read this. |
| [SPEC.md](SPEC.md) | Stoa v0.1 wire specification — capability manifest, state envelope, typed errors, scopes, conformance levels |
| [GOVERNANCE.md](GOVERNANCE.md) | OSS governance, foundation handoff plan, anti-capture commitments, public dated commitments |
| [ECONOMICS.md](ECONOMICS.md) | Why open source works for the editor (Linux/Red Hat parallel), where revenue lives, market sizing |
| [POSITIONING.md](POSITIONING.md) | Brand positioning, three audiences, the GTM motion |
| [FOUNDER_ESSAY.md](FOUNDER_ESSAY.md) | "Your software needs a surface agents can read. We built it." Founder essay; press launch piece |
| [FIRST_MR_SAAS.md](FIRST_MR_SAAS.md) | Theron specialists as the first Stoa-native product |
| [BRAND.md](BRAND.md) | Naming rationale, brand stance, marketing site IA |

---

## Quick read: what Stoa adds beyond MCP and OpenAPI

| Characteristic | Browser scraping | OpenAPI 3.1 | MCP | **Stoa** |
|---|---|---|---|---|
| Stability under UI redesign | broken | unaffected | unaffected | unaffected |
| Idempotency contract | no | partial | no | **yes (declared, mandatory at L2+)** |
| Pagination determinism | no | partial | no | **yes (opaque continuations)** |
| Typed errors w/ remediation | no | code only | prose only | **yes (enum + hints + next_capability URN)** |
| Capability discovery | DOM-walk | path-based | keyword over prose | **URN + embedding + structural query** |
| Cost predictability | no | no | no | **yes (price oracle, per-capability)** |
| Multi-vendor sagas | manual | manual | manual | **declared in graph; auto-compensation** |
| Signed receipts | n/a | n/a | n/a | **JWS + daily Merkle root anchoring** |
| Live state subscriptions | manual webhook | manual webhook | manual | **state-delta bus, prefix subs** |
| Privacy classes (PII/PHI) | manual | manual | manual | **declared at field level; routing enforced** |
| Agent identity attestation | n/a | n/a | n/a | **hive-issued JWT + model attestation** |
| Audit trail (Acting-As) | manual | manual | manual | **first-class** |
| Conformance attestation | n/a | none | none | **open test suite, 5 levels** |

Stoa-conformant servers can **speak MCP at the wire layer** for backwards compatibility with MCP clients. Stoa is the upper-stack that MCP refused to ship.

---

## The three pillars

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

---

## Editor + foundation handoff

Vext Labs is editor of v0.1. The spec hands off to a neutral foundation (Linux Foundation candidate, alongside MCP/A2A/ACP at the Agentic AI Foundation) at the **5-vendor trigger**: when 5 independent vendors achieve L4 conformance.

> *Vext Labs commits to transfer editorial authority, trademark, repository ownership, and conformance certification to a neutral foundation when five independent vendors achieve L4 conformance. Failure to initiate transfer within 90 days of the trigger firing is grounds for any party to fork the spec under existing licenses; Vext explicitly waives any objection to such forks.*

That commitment is in [SPEC.md §11](SPEC.md#11-versioning--foundation-handoff) and [GOVERNANCE.md](GOVERNANCE.md). It is irreversible.

---

## How to contribute

See [CONTRIBUTING.md](CONTRIBUTING.md). Public RFC process. All discussions in [GitHub Discussions](https://github.com/Vext-Labs-Inc/stoa-spec/discussions). All changes via PR with public 14-day comment period.

---

## Roadmap (six months from 2026-05-10)

| Month | Spec | Runtime | Network |
|---|---|---|---|
| **1** | Stoa/1 v1 RFC public | `stoa-edge` skeleton | First registry running at `caps.stoa.foundation` |
| **2** | `stoa-conformance` v1 published | Receipt log → daily Merkle root; SIEM connector | Embedding index live |
| **3** | First vendor at L3 | Saga compensation + budget hard-kill + multi-rail settlement | Price oracle entries; 25 caps across 10 vendors |
| **4** | First vendor RFCs land | `stoa-identity` (DIDs + JWT + model attestation) | Reputation log v1; 3 hives onboarded |
| **5** | Privacy-class spec finalized | `stoa-bus` (SSE + WebPush) | Privacy zones in graph; HIPAA routing live |
| **6** | Spec v1.0 freeze | Full SDK (TS/Python/Rust); offline bundles GA | 25 production-grade adapters; foundation handoff begun if 5-vendor trigger fires |

Detailed plan in [STOA.md §23](STOA.md#23-six-month-shipping-plan).

---

## Five modules to build in parallel

Each module ships independently behind the Stoa/1 wire. Each will live as its own repo under `Vext-Labs-Inc/` (and migrate to a foundation org at handoff). All Apache-2.0.

1. **`stoa-edge`** — runtime workers (Cloudflare Workers + Durable Objects)
2. **`stoa-graph`** — federated capability registry
3. **`stoa-identity`** — DID-based hive identity + JWT + reputation
4. **`stoa-bus`** — live state-delta SSE/WebPush fanout
5. **`stoa-sdk`** — client libraries (TS/Python/Rust)

Plus: **`stoa-conformance`** (test suite, Apache-2.0).

---

## Who's behind this

[Vext Labs](https://tryvext.com/) is the editor of Stoa v0.1.

Vext sells [Theron](https://theron.tryvext.com/) (a council of 30 specialist 100B+ models), AE OS (the OS surface), and Hive Pro (autonomous agent swarm) — on top of the open Stoa substrate.

**Vext does not sell Stoa.** That is the entire point. See [ECONOMICS.md](ECONOMICS.md) for why this works.

---

## Contact

- **Spec / vendor adoption:** [stoa@tryvext.com](mailto:stoa@tryvext.com)
- **Press / partnerships:** [press@tryvext.com](mailto:press@tryvext.com)
- **Founder:** [annalea@tryvext.com](mailto:annalea@tryvext.com)
- **GitHub Issues + Discussions:** [github.com/Vext-Labs-Inc/stoa-spec](https://github.com/Vext-Labs-Inc/stoa-spec)
- **Subdomain:** [stoa.tryvext.com](https://stoa.tryvext.com/)

— *Stoa, by Vext Labs*

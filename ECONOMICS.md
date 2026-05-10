# Stoa — The Economics of Giving It Away

**Status:** v1
**Date:** 2026-05-10
**Author:** Vext Labs
**Companion:** [STOA.md](STOA.md), [GOVERNANCE.md](GOVERNANCE.md)

---

## 1. The thesis

**Stoa is fully open source. Vext makes money on Theron and AE OS. The math works because Stoa adoption multiplies Theron and AE OS revenue, and any attempt to monetize Stoa would suppress its adoption.**

This document shows the math.

---

## 2. The four product layers and where revenue lives

```
   ┌─────────────────────────────────────────────────────┐
   │  AE OS                                  PAID         │  Subscriptions, seats, enterprise
   │  the surface where intent becomes work               │
   └─────────────────────────────────────────────────────┘
                          │
   ┌─────────────────────────────────────────────────────┐
   │  Hive (Pro tier)                        PAID         │  Per-second burn, autonomous swarm
   │  the swarm orchestration on top of Theron           │
   └─────────────────────────────────────────────────────┘
                          │
   ┌─────────────────────────────────────────────────────┐
   │  Theron                                 PAID         │  Per-call rates on specialist caps
   │  the council of 30 specialist 100B+ models          │
   └─────────────────────────────────────────────────────┘
                          │
   ┌─────────────────────────────────────────────────────┐
   │  Stoa                                   FREE         │  Spec, runtime, registry, SDKs, conformance
   │  the open substrate                                  │
   └─────────────────────────────────────────────────────┘
                          │
   ┌─────────────────────────────────────────────────────┐
   │  MCP / OpenAPI / JSON Schema            FREE         │  Industry primitives Stoa builds on
   └─────────────────────────────────────────────────────┘
```

Vext's revenue lives at the top three layers. Stoa is infrastructure.

---

## 3. The Linux/Red Hat parallel

The clearest analog is Linux:

| Layer | Open or paid? | Who profits |
|---|---|---|
| Linux kernel | Free | Nobody directly |
| Linux distros (RHEL, Ubuntu, etc.) | Paid for support, free for code | Red Hat ($40B IBM acq), Canonical |
| Cloud-managed Linux (EC2, GCE, etc.) | Paid | AWS ($90B/year), GCP, Azure |
| Apps running on Linux | Paid | Every SaaS company on Earth |

**Linus did not get rich from Linux.** Red Hat got rich. AWS got richer. SaaS got richest. The kernel being free was the precondition for the entire upper-stack value capture.

Same pattern with Stoa:

| Layer | Open or paid? | Who profits |
|---|---|---|
| Stoa spec + runtime + registry | Free | Nobody directly |
| Stoa-managed runtime hosting | Paid | Vext (via AE OS), Cloudflare, anyone offering managed hosting |
| Theron specialist caps on Stoa | Paid (per-call) | Vext |
| AE OS subscriptions | Paid | Vext |
| Enterprise agent platforms running on Stoa | Paid | Vext, OpenAI, Anthropic, Cursor, Devin, every other agent platform |

Vext's job is **not** to be Linus. Vext's job is **to be Red Hat + AWS + the SaaS company**, all at the value layers above the substrate.

---

## 4. Where each dollar comes from

### 4.1 Theron specialist caps — primary revenue

Every time *any* agent in the world calls a Theron capability via Stoa, Vext earns the per-call rate. Today there are 9 Theron caps with these rates (from the existing manifest):

| Capability | Rate |
|---|---|
| `vext.cyber.scan` | $0.50/req + $5/finding |
| `vext.cyber.report` | $50/report |
| `vext.code.review` | $0.10/file |
| `vext.code.refactor` | $1/refactor |
| `vext.legal.review` | $5/contract |
| `vext.medical.review` | $10/chart |
| `vext.council.synthesize` | $0.10/1K input + $0.40/1K output |
| `vext.research.search` | $0.20/query |
| `vext.classify.intent` | $0.001/req |

These caps run on infrastructure Vext owns end-to-end. The cost of serving them is fixed (RunPod Serverless inference time + Cloudflare egress). The price is set by Vext. Stoa is the wire that delivers the call; **Vext keeps 100% of the per-call revenue from its own caps regardless of which runtime brokers the call.**

### 4.2 AE OS subscriptions — primary revenue

AE OS is the surface where humans give intent and watch the swarm execute. Stoa is the substrate AE OS uses to reach the world. AE OS is paid:

- **Personal tier**: $40/month for individual creators
- **Pro tier (Hive)**: burn-rate billing per inference second
- **Enterprise tier**: $50K+/year per organization, custom

Stoa being free is what makes AE OS valuable: the OS that consumes Stoa natively (with full saga support, full audit trail, full privacy classes) is the OS people pay for. **An AE OS that requires its users to also subscribe to a separate "agent runtime layer" is dead on arrival.**

### 4.3 Hive Pro burn-rate — primary revenue

Hive is the autonomous swarm tier. Stoa is the layer Hive's planner reasons over. Stoa caps become Hive's tools; Stoa receipts become Hive's audit trail. Hive billing is per-inference-second, all flowing to Vext.

### 4.4 Stoa-managed hosting — small but recurring

Vext can run **managed Stoa Edge** for vendors and buyers who don't want to self-host. Apache-2.0 means anyone *can* self-host; convenience pricing means most won't bother:

- **Vendor managed runtime**: $500/month for vendors who want Vext to host their Stoa endpoint (with SLA, monitoring, scaling). This is hosting fees, not protocol fees.
- **Buyer managed runtime**: $1K-10K/month tiered by call volume for enterprise buyers who want Vext-managed Stoa Edge in their VPC peer.

Estimated revenue: $50K-200K/month at scale. Real but not the main thing.

### 4.5 Vext-attested adapters — small but strategic

For SaaS vendors who don't want to ship Stoa themselves but want to be *on* Stoa, Vext can build and maintain an attested adapter. Apache-2.0 code, but Vext charges for the maintenance commitment:

- **Vext-attested adapter**: $2K/month per vendor for adapter maintenance, conformance updates, compatibility testing.
- Estimated: 25 adapters at launch × $2K = $50K/month recurring.

The adapter code is open; what's paid for is the SLA and maintenance.

### 4.6 What Vext does **not** charge for

To be explicit, Vext does **not** charge for:

- The Stoa specification (CC-BY-4.0)
- The Stoa Edge runtime code (Apache-2.0)
- The Stoa SDKs in any language (Apache-2.0)
- The Stoa conformance suite (Apache-2.0)
- The Stoa registry implementation (Apache-2.0)
- The federated graph hosted at `caps.stoa.foundation` (free tier always)
- Listing in the Stoa marketplace (free)
- Self-hosting Stoa Edge in your own infrastructure
- Running your own Stoa registry
- Forking any Stoa repo
- Using Stoa-conformant infrastructure built by competitors (OpenAI, Anthropic, Cursor)

These are all free. Forever.

---

## 5. The flywheel — why free Stoa multiplies Vext revenue

```
           Stoa is free
                │
                ▼
  Every agent platform consumes the catalog
                │
                ▼
  Every SaaS vendor publishes a Stoa contract
                │
                ▼
  Theron caps on Stoa become reachable from every agent
                │
                ▼
  Every Cursor / Claude Desktop / OpenAI Apps / Devin / etc.
  agent that wants to do real work calls Theron caps
                │
                ▼
  Vext earns per-call revenue at scale, with zero
  customer-acquisition cost on the agent platform side
                │
                ▼
  Theron's reach justifies AE OS as the OS that consumes
  Stoa natively
                │
                ▼
  AE OS subscription revenue compounds
```

Compare with the alternative — proprietary Stoa:

```
        Stoa requires a license fee
                │
                ▼
  Agent platforms refuse to adopt (they'd be paying
  a competitor for substrate)
                │
                ▼
  SaaS vendors don't ship Stoa contracts (no demand)
                │
                ▼
  Theron caps reach only Vext's own AE OS users
                │
                ▼
  Theron is a niche product on a niche surface
                │
                ▼
  AE OS plateaus at the boundary of Vext's direct sales
```

The free path leads to Theron caps being called billions of times per month from every agent platform on Earth. The proprietary path leads to Theron being a Vext-only feature with maybe 10K daily active calls.

**Free Stoa is the highest-leverage decision Vext can make.**

---

## 6. The market sizing

### 6.1 Agent traffic projections

The frontier scan confirmed:

- MCP: 97M monthly SDK downloads as of March 2026 (4,750% growth in 16 months)
- Stripe Sessions 2026 projects agent commerce at $400B GMV by 2028
- A2A: 150+ orgs, production deployments
- Composio: 250+ platforms, agent-call volume in tens of millions/month already

Conservative agent traffic estimate by mid-2027: **100B agent tool calls/month** across all platforms combined.

### 6.2 Theron's addressable share

Theron caps are domain-specialist (cyber, code, legal, medical, research). They're not commodity. They compete on quality (99% SecQA, 85% IFEval, primary-source training, no distillation). When agents need *expert* output, Theron is the call.

If Stoa hits 100B calls/month and Theron caps capture 0.05% of those calls (5M/month) at $0.20 weighted average price, that's **$1M/month from Theron caps alone**. At 0.1% capture, $2M/month. At 0.5% capture, $10M/month.

These are bottom-up estimates. Top-down: Theron is targeted at the segment of agent traffic that needs verified, citation-bearing, regulated-domain output. That segment is small in count but high in dollar value (pentest engagements run $5K-50K, code reviews $200-2,000, legal review $500-5,000). **Each Stoa-brokered Theron call is worth $0.50-$50 to Vext.**

### 6.3 AE OS addressable market

AE OS is a desktop OS shell for autonomous agent work. The TAM is:

- 30M individual creators / power users (Cursor, Claude users, Devin users) at $40/month tier = $14B TAM
- 100K enterprise teams at $50K/year average = $5B TAM
- Hive Pro burn-rate: variable, scales with usage

AE OS is the surface where Stoa value materializes for end users. **Stoa adoption is the precondition for AE OS becoming the default surface.** A free Stoa accelerates AE OS into every market segment; a paid Stoa caps AE OS at Vext's direct-sales bandwidth.

---

## 7. The cost side

What does Stoa cost Vext to run?

### 7.1 Engineering

- 5 module leads × 6 months to ship v1 (per the §22 plan in STOA.md)
- After ship: 2-3 maintainers ongoing through pre-handoff
- Post-handoff: 1 maintainer (Vext's representative on the foundation TSC)

Estimated: $500K through ship; $300K/year ongoing.

### 7.2 Infrastructure

- Cloudflare Workers + Durable Objects: scales linearly with calls; ~$0.50 per million requests at scale
- R2 storage for capability bundles + receipt log: ~$15/TB/month
- Daily Merkle anchoring: trivial compute

Estimated: under $10K/month for the first year. Under $50K/month at 1B calls/month.

### 7.3 Foundation handoff costs

- Legal/IP transfer
- Trademark transfer
- Foundation membership dues
- Initial TSC seed funding

Estimated: $200K one-time + $100K/year membership at top tier.

### 7.4 Total

**Roughly $1M one-time + $500K/year ongoing.** Against revenue projections of $1M/month at minimum capture rate, the ROI is positive within the first 6 months of v1 launch and grows from there.

---

## 8. The competitive math

### 8.1 What if OpenAI builds their own Stoa?

They might. They probably will. **And it doesn't hurt Vext.**

- The standard is genuinely open. OpenAI's marketplace consuming Stoa contracts is fine — vendors win twice (their contract reaches OpenAI's catalog and the foundation's catalog).
- Theron caps remain Theron caps regardless of which marketplace lists them. OpenAI cannot compete on Theron's domain expertise without acquiring Vext.
- AE OS competes with OpenAI's "Operator" / "ChatGPT Agent" surface; Stoa being open means AE OS can use the same primitives. The question becomes which OS provides better intent-to-execution UX, not which OS controls the substrate.
- Federation guarantees interop. An OpenAI registry and the foundation registry resolve URNs identically.

### 8.2 What if Anthropic doubles down on MCP and refuses Stoa?

They might. **And it doesn't hurt Vext, because Stoa-conformant servers can speak MCP at the wire layer.** Backwards compat with MCP is a v0.1 design goal.

If Anthropic refuses to adopt Stoa primitives (idempotency, signed receipts, sagas), MCP servers will still work — they just stay at L1 conformance and miss the upper-stack capabilities. The market segment that needs L4 (regulated industries, enterprise compliance, high-value transactions) will buy from L4-conformant vendors and route through L4-aware runtimes. That's where Theron and AE OS live.

### 8.3 What if every agent platform builds their own competing standard?

They might try. They will fail to coordinate, and Stoa's federated registry will become the only place vendors can publish a contract that reaches every platform. The consolidation pattern follows TCP/IP: many competing networking standards in the 1980s; one universal substrate by 1995.

The first-mover advantage is the catalog of integrations Vext ships at launch (25 vendors at L4 conformance via Vext-attested adapters). Competitors have to write their own from scratch.

---

## 9. The exit math

If Stoa becomes the agent substrate (TCP/IP analog):

- **Vext IPO scenario**: Theron + AE OS + Hive revenue at scale. Comparable: Snowflake at IPO ($120B market cap). Anthropic at private market ($200B+). Vext at scale: $50-100B.
- **Vext acquisition scenario**: by a hyperscaler that wants to internalize the value layer. Comparable: Red Hat (IBM, $40B). Datadog (acquisition rumors at $50B+). Vext acquisition: $20-50B.
- **Foundation hand-off pre-acquisition**: Stoa lives on; Vext's value is purely Theron + AE OS, valued on those merits.

If Stoa adoption stalls:

- Vext still has Theron + AE OS + Hive as standalone products. Theron's quality is unique; AE OS's surface is unique. Stoa being free doesn't reduce these.
- Worst case is Stoa is a free open-source asset Vext maintains for its own products. Cost: $500K/year. Manageable.

The downside is bounded. The upside is multi-tens-of-billions.

---

## 10. The decision

Stoa is fully open source. Spec, runtime, SDKs, conformance suite, registry, reference adapters. CC-BY-4.0 + Apache-2.0. Foundation handoff at the 5-vendor trigger.

**Vext makes money on Theron, AE OS, and Hive — on top of the open substrate.** Stoa being free is what makes those three products worth what they're worth.

This is not a sacrifice. It's the highest-leverage move on the board.

— *Vext Labs, May 2026*

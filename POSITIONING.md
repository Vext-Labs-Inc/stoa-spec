# Stoa — Positioning

**Status:** Draft 1
**Date:** 2026-05-01
**Companion:** [SPEC.md](SPEC.md), [BRAND.md](BRAND.md)
**Anchor:** Annalea, on confirming the name —
> *"The first Marketplace for Machine-Readable SaaS — and our very own integrations for our own agents and everyone else's."*

That sentence is the product. This doc explains what it means and what we have to do to live it.

---

## 1. What Stoa actually is — three things in one product

Most product positioning fails because it confuses one thing for three. Stoa is three distinct things, deliberately, and that is the moat.

### 1.1 The standard
A typed, contract-first, self-describing wire protocol that any SaaS vendor can ship — so agents stop scraping their UI and start hitting a typed contract. Lives at `stoa.tryvext.com/spec`. CC-BY-4.0. Vext is the editor today; spec hands off to a foundation once 5 independently-conformant vendors ship.

### 1.2 The marketplace
The first marketplace for Machine-Readable SaaS. A directory of every vendor that has shipped a Stoa contract, searchable by capability ID, filterable by conformance level. Vendors get distribution into agent platforms; agents get a typed catalog of everywhere they can act. Vext takes a marketplace fee on traffic that's billed through us.

### 1.3 The integrations library
A library of Vext-built and Vext-attested adapters covering every SaaS that hasn't yet shipped its own Stoa contract. Every adapter is **dual-purpose:**

- **For Vext's own agents** (Theron / Hive / AE OS) — the adapter is the highest-fidelity tool surface, the one the planner picks first.
- **For everyone else's agents** (OpenAI Responses, Claude Desktop, Cursor, Codeium, Devin, Gemini CLI, custom agents) — the same adapter is reachable as an MCP server. Anyone running an agent can `mcp connect stoa.tryvext.com` and get the same typed catalog Vext consumes.

This is the load-bearing positioning move: **we don't build the integration layer for ourselves. We build it for the entire agent industry, and we happen to be the first consumer.**

---

## 2. Why three-in-one is the moat

The brief asked for three things separately. Bundling them is what makes the bet defensible.

| Without the standard | Without the marketplace | Without the integrations |
|---|---|---|
| Each adapter is a one-off — no portability, no audit, no procurement story | Vendors who ship Stoa have nowhere for buyers to find them; agents have no canonical catalog | The standard is a PDF, the marketplace is empty, and nobody believes it for 18 months |

With all three:

- **Vendors** see a marketplace they can ship into today, with a clear standard to conform to and a clear path to take their adapter from "Vext-attested" to "vendor-published `full` conformance." Their incentive: agent-traffic revenue.
- **Buyers** (engineering leaders deploying agents) see a catalog of typed integrations on day 0 — they don't have to wait for vendors to catch up. Their incentive: their agents stop hallucinating against UIs.
- **Other agent platforms** (OpenAI / Claude / Cursor / etc.) see a free MCP server hosting hundreds of typed integrations they don't have to build themselves. Their incentive: ship better agents without building 200 integrations from scratch.

The first two arrows already pointed at Vext. The third arrow is the one Annalea named: **everyone else's agents reach into the same catalog**. That's how Stoa becomes the default, not just the Vext-flavored version of MR-SaaS.

---

## 3. Who pays, and for what

**Stoa itself is free.** No marketplace listing fee. No traffic share on Stoa. No vendor tax. All Stoa code is Apache-2.0; the spec is CC-BY-4.0. This is not a freemium play with a paid tier hidden behind conformance levels — L1 through L4 conformance is earned by meeting the spec, not by paying Vext.

Vext monetizes three things that are adjacent to Stoa — not Stoa itself:

| Audience | Free | Paid |
|---|---|---|
| **SaaS vendor** | Publish your own Stoa contract; get listed at any conformance level; appear in the marketplace; ship your own adapter using the Apache-2.0 runtime | $2K/mo for **Vext to host and maintain a Vext-attested adapter on your behalf** (convenience, not required); **Stoa-managed runtime hosting** at cost + margin if you want Vext to run the gateway instead of self-hosting |
| **Buyer** (engineering leader) | Browse the marketplace; consume any vendor with `core+` Stoa contract via MCP for free; connect agents directly | If they're a Pro Hive customer, the burn-rate billing pays for Theron/Hive usage — not for Stoa access itself |
| **Other agent platform** | Consume the catalog via MCP free | If they want Vext to bill agent traffic through our rails (white-label marketplace), revenue share on transactions Vext processes |
| **Vext** | Operate the reference registry, editor of v0.1 spec, publish the conformance suite | Earns on Vext-attested adapter maintenance subscriptions (~$2K/mo each), Stoa-managed runtime hosting (convenience pricing), and agent-traffic billing rails (12% on transactions Vext processes) — never on Stoa adoption itself |

The economics: ~$50K/mo recurring from 25 attested-adapter subscriptions at launch + billing-rail revenue on agent traffic that opts into Vext payment processing. Stoa adoption itself compounds the value of Theron and AE OS — that's the real return on the open-source bet.

---

## 4. The "everyone else's agents" mechanic — concretely

Here's how an OpenAI Responses developer or a Cursor user actually consumes Stoa:

```bash
# In Cursor settings → MCP servers
mcp connect https://stoa.tryvext.com/mcp
```

That single command gives them:

- The full Stoa catalog as MCP tools (every chartered vendor's capabilities)
- Auth flow per vendor (OAuth / API key / agent-bearer JWT)
- Stoa's typed error contracts surfaced as MCP error envelopes
- Per-call cost reporting (so the Cursor user's burn meter is honest)

**They don't need a Vext account.** The standard is open; the MCP gateway is free; only revenue-sharing vendors pay us. This is the moat: every agent platform's developers find their way to `stoa.tryvext.com` because there's no friction and no competing catalog.

When a vendor at the marketplace asks *"why should I ship Stoa?"* the answer is:
> *Because OpenAI Responses developers, Claude Desktop users, Cursor users, Devin instances, and Vext Hive sessions all consume Stoa. You ship one contract; you reach every agent platform.*

---

## 5. The category sentence

If a procurement person asks *"what is Stoa?"* the answer in one breath:

> *Stoa is the open standard for agent-readable SaaS — and the first marketplace where vendors who ship Stoa contracts get distributed into every agent platform. Vext Labs is the editor, the integrations library, and the first marketplace operator. The standard is open and CC-BY-4.0; the integrations are Vext-built but consumable by anyone's agents via MCP; the marketplace is where agent-traffic revenue compounds.*

That paragraph goes on the home page. Short version on the hero:

> *Stoa. The substrate of agent-readable SaaS. The first marketplace for Machine-Readable SaaS — built for our agents. Consumable by everyone else's.*

---

## 6. What changes from the v1 marketing site IA

The marketing site IA in [BRAND.md](BRAND.md) §6.1 is mostly right, but the navigation needs to surface the three things equally. Adjusted top-nav:

```
[ Spec ]   [ Marketplace ]   [ For vendors ]   [ For agent platforms ]   [ Docs ]
```

The new tab is **"For agent platforms"** — the page that explains how OpenAI Responses / Claude Desktop / Cursor / Codeium / Devin developers connect their agents to the Stoa catalog via MCP. With code samples, a one-line copy-paste connect command, and a list of which platforms have already integrated.

This page is what makes the "everyone else's agents" claim real. Without it, Stoa reads as a Vext-only thing dressed up as open.

---

## 7. No partner motion — the catalog is the product

We deliberately do *not* run partnership outreach to agent platforms (Cursor, Claude Desktop, Codeium, OpenAI Responses, Devin, Continue, Cline, Zed, Aider, etc.). Three reasons:

1. **The standard is open and the gateway is free.** Any MCP-capable client connects with one URL paste. No partnership is *needed* to make the integration work.
2. **Asking for a partnership weakens the position.** It signals we need them. We don't. The catalog exists; their users will find it.
3. **Annalea's stated stance:** *"I don't need partners from them — ew."* This is correct read. The brand stance is "we built the agent-readable surface for the entire agent industry, including yours, whether you opt in or not."

What we do instead:

- **Document the connect line** for every major MCP-capable client on `tryvext.com/stoa/agents`. Users find it themselves.
- **Ship an OpenAI-format adapter** at `stoa.tryvext.com/openai/manifest` so OpenAI Responses developers (who don't have first-class MCP) can pull function-calling tools directly. Same standard, different shape; no partnership required.
- **Let the catalog speak for itself.** When OpenAI Responses developers / Cursor users / Claude Desktop users start using the catalog, the agent platforms notice. If they want to deepen integration, they come to us. We do not chase.

The category-defining brands — Stripe, Plaid, Twilio — did not become defaults by partnering with the platforms that consumed them. They became defaults by being the easiest typed surface to consume. Same play here.

---

## 8. The press hook — sharpened

> *"Vext Labs launches Stoa, the open standard for agent-readable SaaS, with 25 vendors at launch and integration with 5 major agent platforms — including Cursor, Claude Desktop, Codeium, OpenAI Responses, and Devin. The marketplace is the first place a SaaS vendor can earn agent-traffic revenue across every major agent platform with a single contract."*

That sentence is the TechCrunch / Information lede. The "everyone else's agents" framing is what makes it *news* and not just *another agent vendor announcement*.

---

## 9. The risk

The honest risk: **at least one agent platform will see Stoa as competitive infrastructure they want to control themselves.** OpenAI is the most likely — they have a stated interest in being the catalog of catalogs. Mitigation:

- **The standard is genuinely open.** OpenAI can publish their own marketplace on top of the same Stoa contracts — we don't lose, vendors win twice.
- **Foundation hand-off is real.** When 5 independently-conformant vendors ship, the spec moves to a foundation. We give up control of the standard to keep the marketplace.
- **First-mover wins the integrations library.** The 25 day-1 adapters are written by us; a competitor would have to write their own. We have a 6-month head start and the working catalog before any competitor ships v1.

The position: *we win by making the cake bigger, not by owning the slice.* The marketplace economics work even if 50% of agent-traffic eventually routes through OpenAI's catalog instead of ours — we still earn on the vendor-attestation subscriptions and the integrations we maintain.

---

## 10. The decision (already made — confirming)

STOA, three things in one product, marketed as "the first marketplace for Machine-Readable SaaS — built for our agents, consumable by everyone else's." Marketing site IA updated to surface "For agent platforms" as a top-nav peer to "For vendors." Press launch lede sharpened around the 5-platform integration story.

Next: the marketing site scaffold under `marketing/src/pages/stoa/` + production-ready home page + the "For agent platforms" page that makes the open-to-everyone claim real.

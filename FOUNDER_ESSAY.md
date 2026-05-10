# Your Software Needs a Surface Agents Can Read. We Built It.

**By Annalea Layton**, founder, Vext Labs
**Embargoed until launch (week 4 of the FULL SEND ship plan)**
**Target outlets:** TechCrunch, The Information, Stratechery cross-post, Vext blog
**Target length:** ~1,400 words
**Target reading time:** 6 minutes

---

I've spent the last year watching agents fail.

Not the dramatic failures — the polite ones. An agent books the wrong meeting because the calendar UI shipped a new modal last Thursday. An agent re-asks for "page 1 of search results" because it lost track of which page it was on. An agent emails seventeen duplicate contacts into a CRM because there was no idempotency key. An agent gives up on a workflow because a button moved by 8 pixels.

The industry's response has been: *make the agent smarter.*

I think the industry is wrong. Not about needing smart agents — we obviously need those. About the failure mode. Most of these agents aren't failing because they're dumb. They're failing because we're forcing them through a door that wasn't built for them.

Today's SaaS has one front door. The UI. It was designed for humans pointing a mouse, scanning with eyes, holding two seconds of working memory across a click. We built a $300 billion industry around that door. Then we sent agents through it.

The result is the entire industry's current frustration. Look closely at any "the agent broke" thread on X and you'll find the same root cause: pixels, scraped DOMs, parsed HTML, hallucinated buttons.

We can keep training agents to navigate the human UI better. Or we can build a machine-readable surface alongside it.

---

Today, Vext Labs is releasing **Stoa** — an open standard for agent-readable SaaS.

The Stoa was the colonnaded walkway in ancient Athens where philosophers and merchants worked under one roof. Ideas became contracts. Contracts became commerce. It was the substrate of the agora — the infrastructure layer that made everything else possible. We picked the name because the metaphor is the product. Stoa is the substrate your software stands on when it becomes readable to agents. The human UI keeps shipping. The Stoa contract is unchanged through every redesign.

Concretely, an MR-SaaS — a Machine-Readable SaaS — publishes a typed contract at `/.well-known/stoa.json`. Every action an agent can take is in there: typed inputs, typed outputs, side-effect annotations (idempotent? mutating? destructive?), idempotency keys, rollback semantics, capability-level permission scopes, typed error contracts with remediation hints. Every response is wrapped in a deterministic state envelope that kills the entire class of "the agent re-asked for page 1 instead of page 2" failures.

It is built on top of OpenAPI 3.1 and JSON Schema 2020-12 and MCP. A vendor with a clean OpenAPI spec is 80% of the way there. Stoa adds the missing 20% that agents actually need to act safely against your API. The spec is CC-BY-4.0. The conformance test suite is open-source under MIT. The reference adapter implementations are Apache 2.0. We are the editor today; once five independent vendors hit `full` conformance, the spec hands off to a foundation.

*Stoa is fully open source. Spec under CC-BY-4.0. Runtime, SDKs, conformance suite, and registry implementation under Apache-2.0. Forever. Vext Labs is the editor of v0.1; spec hands off to a neutral foundation at the 5-vendor trigger — when 5 independent vendors achieve L4 conformance. We don't sell Stoa. We sell Theron (the brain) and AE OS (the surface). Stoa is the substrate that makes them matter.*

You can read the whole specification at `tryvext.com/stoa/spec`. It is 14 sections, ~5,000 words. The shape that matters is one paragraph:

> *Vendors publish a capability manifest listing every action agents can take, with side-effect annotations, deterministic state envelopes, and typed error contracts. Agents stop scraping the UI; vendors stop hallucinating against UIs that were never meant for them.*

That's the surface. That's the entire pitch.

---

The harder part is what we're shipping alongside the spec.

A spec, on its own, is a PDF. We're shipping three things together because shipping any one of them alone wouldn't matter.

**The standard** — the contract a vendor publishes.

**The marketplace** — at `tryvext.com/stoa/marketplace`, the first directory of every vendor that has shipped a Stoa contract. Searchable by capability ID. Filterable by conformance level. We're launching with 25 Vext-attested vendors that we've chartered on their behalf — Slack, Gmail, Notion, Linear, GitHub, Stripe, Plaid, HubSpot, Salesforce, Twilio, Sentry, Datadog, Cloudflare, and 12 others. Every one of them is at `typed-state` conformance level on day 1. Vendors take over the contract from us when they're ready.

**The integrations library** — and this is the part nobody else is doing — the same library that powers our own agents at Vext Hive is consumable by *every other agent platform* through one MCP server URL.

That last part is the load-bearing positioning move. We are not building the catalog for ourselves. We are building it for the entire agent industry — and we happen to be the first consumer.

Here is what that means in practice. Any agent client that speaks MCP — and they all do now — connects to the catalog with one line:

```
mcp connect https://stoa.tryvext.com/mcp
```

That's the entire integration. No partnership negotiated. No agent platform notified. No Vext account required. The user pastes the URL into their client's settings and inherits the entire Stoa catalog as typed tools. We are not in the business of partner agreements with agent platforms. We are in the business of the catalog. The standard is open. The gateway is free. The only people who pay us are revenue-sharing vendors who route their agent traffic through Vext's billing rails (12% revenue share, like Plaid for finance).

When a vendor at the marketplace asks me *"why should I ship Stoa?"*, the answer is short:

> *Because every MCP-capable agent client on the planet — and that's now all of them — can consume your contract with one URL paste. You ship one contract; you reach every agent client.*

---

I want to address the two questions I expect from this announcement.

**"Won't OpenAI just build their own version?"**

Probably. They have an obvious incentive to be the catalog of catalogs.

That's fine. The standard is genuinely open — Apache-2.0 runtime, CC-BY-4.0 spec. OpenAI can publish their own marketplace on top of the same Stoa contracts — vendors win twice. We win by making the cake bigger. Even if 50% of agent traffic eventually routes through OpenAI's catalog instead of ours, the marketplace economics still work for us — we earn on the Vext-attested adapter subscriptions and the integrations we maintain.

The first-mover advantage isn't the standard. It's the integrations library. We have written 25 production-grade adapters; a competitor would have to write their own. We have a six-month head start and a working catalog before any competitor ships v1. That is the moat that compounds.

**"Why should anyone trust Vext to be the steward?"**

We don't think we should be the long-term steward. We are the editor of v0.1 because somebody has to ship the first version. The spec moves under a foundation — most likely Linux Foundation, where Cisco's AGNTCY and Google's A2A spec already live — once five independent vendors hit `full` conformance. That's a real off-ramp, not marketing language.

In the meantime, we are publicly committing to:
- Spec under CC-BY-4.0. Anyone can fork, run their own marketplace, build competing infrastructure.
- Conformance suite under MIT. Anyone can run their own attestation program.
- Reference adapters under Apache 2.0. Anyone can fork the adapter library and run their own competing one.

If a year from now there are five Stoa marketplaces and five adapter libraries, we will have done what we set out to do. The category will exist. Vendors will earn from agent traffic. Agents will stop hallucinating against UIs that weren't built for them.

---

I started Vext Labs because I believe the entire industry is building agents on top of an API surface that was never designed for them, and I think we can do better.

Theron is our council of 30 specialist 100B-parameter models. Hive is our Pro tier swarm built on Theron. Sentry is the training stack that makes the brain. AE OS is the surface where it all comes together.

Stoa is what makes them all consequential. It is integrated end-to-end with Theron, Council, and Hive. Every chartered vendor is reachable from inside AE OS as a typed tool the swarm can call. **You stop downloading apps. You stop opening them. You ask the agent, and the agent does the work, on every chartered surface, in parallel.** The OS is the integration layer. The agents are the interface. The app era is ending.

Stoa is the surface your software opens to every agent on the planet, including ones that aren't ours.

If you run a SaaS company, your customers' agent traffic is already 5–15% of your total volume and growing fast. By 2027 it crosses 50% for most B2B SaaS. The vendors who ship Stoa contracts first will have the audit trail, the typed cost meter, the procurement story, and the agent-traffic revenue. The vendors who don't will have brittle integrations breaking on every UI redesign and customers wondering why your product alone is making their agents fail.

You can charter your product in 30 minutes at `tryvext.com/stoa/vendors`. You can browse the marketplace at `tryvext.com/stoa/marketplace`. You can read the spec at `tryvext.com/stoa/spec`.

Or you can wait. The category exists either way.

— Annalea
*Founder, Vext Labs*
*[date of launch]*

---

## Press contact
Press inquiries: **press@tryvext.com**
Stoa standard / vendor inquiries: **stoa@tryvext.com**
Founder: **annalea@tryvext.com**

## Asset kit (for outlet inclusion)
- Colonnade hero glyph (SVG): `https://tryvext.com/stoa/assets/stoa-faces.svg`
- 90-second product film (MP4): `https://tryvext.com/stoa/assets/stoa-launch.mp4`
- 25-vendor day-1 logo sheet: `https://tryvext.com/stoa/assets/stoa-vendors.png`
- Founder photo + bio sheet: `https://tryvext.com/press/annalea-layton-kit.zip`

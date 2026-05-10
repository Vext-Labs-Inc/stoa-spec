# Stoa — Governance & Foundation Handoff

**Status:** Public commitment, v1
**Date:** 2026-05-10
**Author:** Vext Labs (editor of v0.1)
**Companion:** [STOA.md](STOA.md), [SPEC.md](SPEC.md), [ECONOMICS.md](ECONOMICS.md)

---

## TL;DR

Stoa is open source forever. Spec under CC-BY-4.0. Code under Apache-2.0. Editor today is Vext Labs. **At the 5-vendor trigger** (5 independent vendors achieve L4 conformance), the spec, trademark, and editorial authority hand off to a Linux Foundation project — irreversibly, by public commitment baked into v0.1.

This document explains the governance model, the handoff trigger, the licenses, and the anti-capture commitments that make those commitments credible.

---

## 1. Why governance matters before launch

Open standards owned by single companies don't get adopted. Vendors won't commit to a contract one company can revoke. Buyers won't route audit trails through infrastructure one company can pull. Agent platforms won't endorse a registry one company can censor.

History is unambiguous: every protocol that captured a category had a credible **non-corporate** governance path — TCP/IP under IETF, HTTP under W3C/IETF, OpenAPI under OAI/Linux Foundation, Kubernetes under CNCF, Bitcoin/Ethereum under their respective foundations, MCP under the Agentic AI Foundation (LF).

Stoa needs the same path, declared up front, with a real trigger.

---

## 2. The licenses (irrevocable)

| Artifact | License | Why |
|---|---|---|
| Spec text (`stoa-spec/`) | **CC-BY-4.0** | Free to use, distribute, fork. Attribution required. Same license MCP, OpenAPI, OAuth all use. |
| Runtime code (`stoa-edge/`) | **Apache-2.0** | Patent grant + commercial use. Industry default. |
| SDKs (`stoa-sdk/`) | **Apache-2.0** | Same. |
| Conformance suite (`stoa-conformance/`) | **Apache-2.0** | Same. |
| Reference adapters (`stoa-adapters/`) | **Apache-2.0** | Same. |
| Foundation registry implementation (`stoa-graph/`) | **Apache-2.0** | Same. |
| Logo and "Stoa" wordmark | Trademarks held by Vext Labs pre-handoff; transferred at handoff under a **trademark policy** that allows any conformant vendor to use the marks descriptively. | |

These licenses are **irrevocable** by their terms. Vext cannot relicense, restrict, or rescind. The community can fork at any time without permission.

---

## 3. The 5-vendor handoff trigger

When **5 independent vendors achieve L4 conformance** (full conformance, foundation-certified), spec stewardship transfers to the foundation. Specifically:

### 3.1 What counts as an "independent vendor"

- A legal entity unaffiliated with Vext Labs (no equity, no board seats, no exclusive commercial agreements).
- Operating in production (real customer traffic, not staging).
- Self-hosting or vendor-published manifest at `/.well-known/stoa.json`.
- Passing the L4 conformance test suite with a public report.
- Distinct corporate parent — five subsidiaries of one parent count as one vendor.

### 3.2 What "handoff" means

- **Editorial authority** — Vext relinquishes the right to merge spec changes unilaterally. Future RFCs require foundation TSC sign-off.
- **Trademark transfer** — "Stoa" wordmark and logo move to the foundation under a policy that allows any conformant vendor to use them.
- **Repository transfer** — `github.com/stoa-spec/*` moves from `vext-labs` org to the foundation org.
- **Domain transfer** — `stoa.foundation` and any other governance domains transfer to the foundation. Vext retains `stoa.tryvext.com` for its own implementation marketing.
- **Conformance authority** — foundation runs the canonical certification program. Vext can continue to offer Vext-attested adapters, but cannot certify L4 conformance independently.

### 3.3 The candidate foundations

In order of preference:

1. **Linux Foundation** — already hosts MCP (Agentic AI Foundation), A2A, ACP, and OpenAPI. Most natural home. LF projects have proven governance, neutral license review, paid TSC support, and a working trademark policy template.
2. **Cloud Native Computing Foundation (CNCF)** — appropriate if Stoa is positioned as cloud-native infrastructure (capability registry as cluster service). Less obvious fit but possible.
3. **OpenInfra Foundation** — backup option.
4. **Apache Software Foundation** — possible for the code repos but doesn't typically host wire specs.

Lead candidate: **Linux Foundation, Agentic AI Foundation sub-project** (alongside MCP). Approach AAF leadership at the 4-vendor mark with formal proposal.

### 3.4 Public dated commitment

Section 14 of [SPEC.md](SPEC.md) carries the binding language:

> *Vext Labs commits to transfer editorial authority, trademark, repository ownership, and conformance certification to a neutral foundation when five independent vendors achieve L4 conformance. Failure to initiate transfer within 90 days of the trigger firing is grounds for any party to fork the spec under existing licenses; Vext explicitly waives any objection to such forks.*

That language is irreversible by the same logic as the licenses.

---

## 4. Pre-handoff governance (Vext as editor)

Until handoff, Vext operates as editor with the following constraints:

### 4.1 RFC process

All spec changes go through public RFC at `github.com/stoa-spec/stoa-spec`:

1. **Draft** — author submits PR with spec change + rationale + reference implementation
2. **Review** — minimum 14 days public comment
3. **Implementation** — at least one Stoa Edge implementation passes the new conformance test
4. **Editor sign-off** — Vext (editor) merges if no unresolved blocking objections
5. **Versioning** — change lands in next minor or major release per semver rules

### 4.2 Editorial veto limits

Vext can veto an RFC only on these grounds, with public written reasoning:

- The change breaks backward compatibility within a major version
- The change introduces a security regression that has no mitigation
- The change is technically infeasible (no working implementation)

Vext cannot veto on:

- Competitive grounds ("this favors a competitor")
- Strategic grounds ("we'd rather ship this differently")
- Aesthetic grounds ("we don't like the API shape")

Vetoes are appealable: any 3 independent vendors can co-sign an appeal that triggers re-review with binding arbitration by a neutral third party.

### 4.3 No reference implementation lock-in

Vext's runtime (`stoa-edge`) is one of N reference implementations. Other parties are explicitly invited to ship competing runtimes. The spec describes the wire; runtimes are independent.

### 4.4 Anti-capture commitments

Beyond the licenses and the 5-vendor trigger, Vext makes these binding commitments:

1. **No exclusive commercial agreements.** Vext will not sign deals with vendors, customers, or agent platforms that grant exclusive rights or preferential treatment in Stoa registries.
2. **No spec changes that disadvantage non-Vext implementations.** RFC review explicitly evaluates whether a change advantages Vext's runtime over forks. Reviewers can block on this ground.
3. **Public roadmap.** All spec roadmap items public at `github.com/stoa-spec/roadmap`. No private prioritization.
4. **Public meetings.** Editor meetings (weekly through the pre-handoff period) are recorded and public.
5. **Conflict-of-interest disclosure.** Vext employees serving as editors disclose all conflicts; conflicted RFCs require third-party reviewer sign-off.

---

## 5. Post-handoff governance (foundation as steward)

Standard Linux Foundation project governance:

- **Technical Steering Committee (TSC)** — typically 5-9 seats, elected from contributors per project bylaws
- **Vext Labs** — entitled to one TSC seat as v0.1 editor and reference implementation maintainer; one of N
- **Vendor seats** — vendors at L4 conformance can stand for election
- **Buyer seats** — large buyers (e.g., enterprise customers running self-hosted Stoa) can stand for election
- **Term limits** — typically 2-year staggered terms
- **Public meetings, public minutes, public votes**

Vext gets no special privileges post-handoff beyond the maintenance of `stoa-edge` (which is one runtime among many) and `stoa.tryvext.com` (Vext's own implementation marketing).

---

## 6. Conformance authority

Pre-handoff, Vext runs the canonical conformance test suite and issues conformance certificates. Specifically:

- **Vendors self-declare** any level by hosting a `conformance.report_url` with the test suite output.
- **Foundation certification** is an additional attestation by Vext (acting as foundation editor) that surfaces in the registry.
- **Multiple authorities allowed.** Anyone can run the conformance suite (it's Apache-2.0) and issue attestations. The foundation's attestation is the default surface; alternatives exist.

Post-handoff, foundation runs the canonical authority. Multiple authorities continue to be allowed.

---

## 7. The federation guarantee

Stoa is federated by design. No single registry is the canonical source. The spec mandates:

- **URN namespace is global.** Any compliant registry can serve any URN.
- **Cross-registry resolution.** Registries publish signed pointers to URNs they don't host.
- **Conflict resolution by attestation strength** — foundation attestation > vendor self-attestation > third-party attestation; agents pick policy per workspace.
- **No single registry can be the gatekeeper.** Even the foundation's registry is one of N.

This guarantee is in the spec text and cannot be removed without a major version bump.

---

## 8. Forking rights

Anyone can fork Stoa at any time, for any reason. Specifically:

- Fork the spec — the licenses permit it.
- Run a competing registry — federation is mandatory; the foundation registry has no privileged status beyond convention.
- Run a competing runtime — `stoa-edge` is Apache-2.0; alternative runtimes are explicitly invited.
- Run a competing conformance authority — `stoa-conformance` is Apache-2.0; alternative authorities can issue their own certificates.
- Run a competing marketplace — there is no canonical marketplace; the foundation may publish a default marketplace, alternatives are first-class.

Vext explicitly waives any objection to forks. Forks that maintain conformance with the spec are fully interoperable with the foundation registry.

---

## 9. Funding

Stoa Foundation operations (post-handoff) are funded by:

- **Member dues** — typical LF membership tiers ($5K–$500K/year depending on size). Vendors at L4 conformance may join as voting members; buyers as non-voting members.
- **Conformance certification fees** — foundation may charge for certification (typical: free for self-declared, paid for foundation-attested with priority review).
- **Registry hosting tier** — foundation registry has free tier (basic listing) and paid tier (priority hosting, longer state-delta retention, faster diffs). Self-hosted and federated registries are free.

Vext Labs commits to founding membership at the highest tier for the first 3 years post-handoff, regardless of foundation choice. This is a public commitment.

---

## 10. Key governance dates

| Date | Milestone |
|---|---|
| **2026-05-10** | This document published; v0.1 spec public; governance commitments binding |
| **2026-06-01** | Foundation candidate proposals submitted (LF/CNCF) |
| **2026-09-01** | First independent vendor reaches L4 (target: HubSpot or Stripe) |
| **2027-01-01** | 5-vendor trigger target. Handoff initiated. |
| **2027-04-01** | Handoff complete. Foundation governs. |
| **2027-06-01** | Spec v1.0 freeze under foundation. |

These are targets, not promises. The 5-vendor trigger is the binding commitment; the dates above are the planning targets to hit it.

---

## 11. The trust math

Why should vendors and buyers believe these commitments?

1. **Licenses are legal contracts.** CC-BY-4.0 and Apache-2.0 are not retractable. Anyone with a copy of the code or spec retains rights regardless of what Vext does later.
2. **The 5-vendor trigger is in the spec.** Removing it requires a spec change, which requires editor sign-off, which Vext explicitly cannot use to block its own removal (anti-capture commitment §4.4).
3. **Trademark transfer is contractually binding** at the trigger. Failure to transfer is grounds for fork.
4. **Vext's revenue is in Theron + AE OS.** Capturing Stoa would slow adoption of Stoa, which would slow Theron's reach. The economic incentive is aligned with openness.
5. **Public dated commitments accumulate reputational cost.** Failing to honor them publicly destroys Vext's standing in the agent industry. The cost of breaking these promises exceeds any benefit.

If those four mechanisms are insufficient: the licenses still hold. Anyone forks. Stoa wins regardless.

---

## 12. What this means for vendors today

- **Adopt without fear of vendor lock-in.** The spec is open; the runtime is open; the registry is federated; the trademark hands off.
- **Your contract is permanent.** Once published, your manifest at `/.well-known/stoa.json` is yours. No platform tax can extract it.
- **You can self-host.** Vext-managed runtime is a convenience, not a requirement. Self-hosting on Cloudflare Workers (or any Worker-compatible runtime: Deno Deploy, Vercel Edge, Fly.io) is supported.
- **Your conformance certification persists.** Foundation-certified L4 conformance is recognized regardless of which registry brokers your traffic.
- **You are not locked into a payment processor.** Stripe is the default; x402, ACH, prepaid foundation credits, and procurement systems are first-class.

## 13. What this means for buyers today

- **Audit trails are yours.** Receipts you collect are signed and Merkle-anchored; verifiable independent of any registry's continued existence.
- **You can ingest into any SIEM.** Apache-2.0 SIEM connector means no vendor-specific tooling required.
- **You control privacy zone routing.** Privacy classes are enforced by the runtime; you set the policy.
- **You can self-host.** For compliance-strict deployments, run your own Stoa Edge inside your VPC. Apache-2.0 means no licensing fees.
- **You retain your data.** Capability schemas and receipts contain hashes, not PII. The runtime's no-log policy on `SECRET.*` and `PHI.*` classes is enforced at protocol level.

## 14. What this means for agent platforms

- **Free integration into a typed catalog.** Connect via MCP, OpenAI Apps, or Stoa native — same caps either way.
- **No partnership negotiation needed.** The catalog is open; the gateway is free; the standard is forkable.
- **Your users get a typed, auditable, privacy-aware agent surface across every Stoa-conformant SaaS.** Without you having to build 200 integrations.
- **You can fork.** Run your own registry, your own runtime, your own marketplace. Federation guarantees interop.

## 15. Closing

Stoa is open because that's what makes it work. Vext Labs is the v0.1 editor because someone has to ship the first version. Foundation handoff is the path that makes Stoa real beyond Vext.

If a year from now there are five Stoa marketplaces and five reference runtimes and the foundation governs the spec, we will have done what we set out to do. The category will exist. Vendors will earn from agent traffic. Agents will stop hallucinating against UIs that weren't built for them.

And Vext will sell Theron and AE OS, on top of the substrate everyone uses.

That's the whole plan.

— *Vext Labs, editor of Stoa v0.1*

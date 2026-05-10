# Contributing to Stoa

Stoa is an open standard. Anyone can contribute. This document explains how.

## Quick links

- **Discussions:** [github.com/Vext-Labs-Inc/stoa-spec/discussions](https://github.com/Vext-Labs-Inc/stoa-spec/discussions) — start here for ideas, questions, proposals
- **Issues:** [github.com/Vext-Labs-Inc/stoa-spec/issues](https://github.com/Vext-Labs-Inc/stoa-spec/issues) — bugs, errata, concrete change requests
- **Pull requests:** [github.com/Vext-Labs-Inc/stoa-spec/pulls](https://github.com/Vext-Labs-Inc/stoa-spec/pulls) — RFCs and implementations
- **Code of Conduct:** [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
- **Governance:** [GOVERNANCE.md](GOVERNANCE.md)

## How to contribute

### Tier 1 — discussion (no commitment required)

Open a [Discussion](https://github.com/Vext-Labs-Inc/stoa-spec/discussions). Categories:

- **Ideas** — *"What if we added X to the wire envelope?"* — preliminary brainstorming
- **Q&A** — *"Why does the spec require Y?"* — questions answered by editor + community
- **Show and tell** — *"I shipped a Stoa-conformant adapter for Z"* — community implementations
- **Polls** — for design choices where editor wants community signal

### Tier 2 — issue (concrete request)

Open an [Issue](https://github.com/Vext-Labs-Inc/stoa-spec/issues) when you have:

- A specific bug in the spec text (typo, contradiction, ambiguity)
- A specific gap (the spec doesn't say what should happen in case X)
- A specific change request (use one of the issue templates)

### Tier 3 — RFC + pull request (proposal with implementation)

For substantive spec changes, follow the RFC process:

1. **Draft** — open a PR with the spec change + rationale + at least one reference implementation (or a credible plan to build one). Title format: `RFC: <short title>`.
2. **Review** — minimum **14 days** public comment period. Editor + community engage.
3. **Implementation** — at least one Stoa Edge implementation passes a new conformance test for the change.
4. **Editor sign-off** — Vext (editor of v0.1) merges if no unresolved blocking objections. Editorial veto is bounded — see [GOVERNANCE.md §4.2](GOVERNANCE.md#42-editorial-veto-limits).
5. **Versioning** — change lands in next minor or major release per semver rules.

After foundation handoff (5-vendor trigger), editor sign-off becomes foundation TSC majority.

## What can be changed

| Change kind | Path | Notes |
|---|---|---|
| Patch (typo, clarification) | Issue or PR | Direct merge if uncontroversial |
| Minor spec change (additive, backward-compatible) | RFC | 14-day review |
| Major spec change (breaking) | RFC | 14-day review + 12-month deprecation window |
| Reference implementation | PR | Must include conformance tests |
| Conformance suite addition | PR | Must come with worked example |

## What can NOT be changed

- The licenses (CC-BY-4.0 + Apache-2.0). These are irrevocable.
- The 5-vendor handoff trigger. Removing it would require a spec change that the editor explicitly cannot use to block its own removal — see [GOVERNANCE.md §4.4](GOVERNANCE.md#44-anti-capture-commitments).
- Federation requirements. The spec mandates that any compliant registry can serve any URN. This is structural and cannot be removed without a major version bump.

## Code contributions

The spec lives here. Code lives in sibling repos (coming Q3 2026):

- `Vext-Labs-Inc/stoa-edge` — runtime workers
- `Vext-Labs-Inc/stoa-graph` — federated registry
- `Vext-Labs-Inc/stoa-identity` — DID + JWT + reputation
- `Vext-Labs-Inc/stoa-bus` — live state SSE/WebPush
- `Vext-Labs-Inc/stoa-sdk` — TS/Python/Rust client libraries
- `Vext-Labs-Inc/stoa-conformance` — test suite

Each will have its own CONTRIBUTING.md once shipped.

## Conformance contributions

If you've built a Stoa-conformant SaaS, vendor adapter, or runtime:

1. Run the conformance suite (`npx stoa-conformance` once shipped).
2. Open a PR adding your conformance report URL to a community registry.
3. Foundation will surface it in the marketplace.

Multi-authority conformance is supported — anyone can run the conformance suite and issue attestations. Foundation attestation is the default; alternatives are first-class.

## Style

- Spec text: terse, normative, no marketing prose. RFC 2119 keywords (MUST/SHOULD/MAY) used precisely.
- Code: language-idiomatic. Conformance tests over docs.
- Commits: imperative present (`add saga compensation declaration`, not `added` or `adds`).
- PR titles: clear, scoped, no emoji.
- Issues: minimum reproducible spec excerpt + observed behavior + expected behavior.

## Conflict of interest

Editors and TSC members disclose conflicts in PR/RFC reviews. A conflicted reviewer can comment but not approve a controversial change without third-party reviewer co-sign.

## Trademark

The "Stoa" name and logo are held by Vext Labs pre-handoff and transferred to the foundation at the 5-vendor trigger. Use of the name to describe your conformant implementation ("Stoa-conformant") is fine. Use of the name as the brand for a competing standard or runtime is not.

## Recognition

Contributors are recognized in:

- Git history (preserved across foundation handoff)
- `CONTRIBUTORS.md` (auto-generated, updated quarterly)
- Annual contributor report at `stoa.foundation` post-handoff

## Contact

- Spec questions: `stoa@tryvext.com`
- Code of Conduct violations: `conduct@tryvext.com`
- Editor (until handoff): `annalea@tryvext.com`

— *Stoa Editor Team*

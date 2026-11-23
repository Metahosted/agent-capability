# Agent Capability

Two drafts for agents that answer for a business, built on [A2A](https://github.com/a2aproject/A2A) and [Agentic Resource Discovery (ARD)](https://github.com/ards-project/ard-spec).

| Draft | What it does | Identifier |
|---|---|---|
| **On Request**, an A2A extension | Lets an agent say that it gives some information to the person who asks, but does not publish it, and where the person can ask. Fixes how the agent replies when it refers rather than answers. | [`https://agent-capability.org/ext/on-request/v1`](ext/on-request/v1/) · [schema](ext/on-request/v1/schema.json) |
| **Agent Capability Trust v1** | Ties an agent to the business it names, at two levels: the business controls the domain, and the domain belongs to the registered business. Usable as an ARD trust manifest and as a trusted key source for A2A card signatures. | [`https://agent-capability.org/trust/v1`](trust/v1/) · [schema](trust/v1/schema.json) |

Each draft is served at its identifier. The same texts are also at the top of this repository ([`on-request-extension.md`](on-request-extension.md), [`trust-framework.md`](trust-framework.md)), where the A2A proposal links to them; the two copies are kept identical.

**Status:** drafts for discussion. Not final. Breaking changes are expected.
- On Request: draft 0.2, 23 November 2026. Adds optional request metadata (`actingFor`, `skillId`) and an optional `reason` on referrals. Changes from 0.1 are in its Annex D.
- Agent Capability Trust v1: draft 0.2, 23 November 2026. Renames the verifying role Verifier & Validator (V&V), allows a V&V operator to run ARD services without self-preference, and adds supplementary services. Changes from 0.1 are in its Annex D.

Draft 0.1 of each (1 November 2026) is in this repository's history.

**Discussion:** the On Request extension is proposed to the A2A project in [a2aproject/A2A#2421](https://github.com/a2aproject/A2A/issues/2421). Comments are welcome there or as issues on this repository.

**Licence:** [Apache 2.0](LICENSE).

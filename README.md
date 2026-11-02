# Agent Capability

Two drafts for agents that answer for a business, built on [A2A](https://github.com/a2aproject/A2A) and [Agentic Resource Discovery (ARD)](https://github.com/ards-project/ard-spec).

| Draft | What it does | Files |
|---|---|---|
| **On Request**, an A2A extension | Lets an agent say that it gives some information to the person who asks, but does not publish it, and where the person can ask. Fixes how the agent replies when it refers rather than answers. | [`on-request-extension.md`](on-request-extension.md), [`on-request-extension.schema.json`](on-request-extension.schema.json) |
| **Agent Capability Trust v1** | Ties an agent to the business it names, at two levels: the business controls the domain, and the domain belongs to the registered business. Usable as an ARD trust manifest and as a trusted key source for A2A card signatures. | [`trust-framework.md`](trust-framework.md), [`trust-framework.schema.json`](trust-framework.schema.json) |

**Status:** drafts for discussion, 1 November 2026. Not final. Breaking changes are expected.

**Identifiers:** `https://agent-capability.org/ext/on-request/v1` and `https://agent-capability.org/trust/v1`. These are identifiers; the address does not yet serve the documents.

**Discussion:** the On Request extension is proposed to the A2A project in an issue in [a2aproject/A2A](https://github.com/a2aproject/A2A/issues). Comments are welcome there or as issues on this repository.

**Licence:** [Apache 2.0](LICENSE).

# On Request: an A2A extension

**Extension URI:** `https://agent-capability.org/ext/on-request/v1`
**Document version:** 0.1 draft · 1 November 2026 · Draft for discussion; breaking changes expected
**Built on:** A2A Protocol 1.0 (read at tags v1.0.0 and v1.0.1; v1.1.0-rc checked for changes to extensions, none found)
**Schema:** `on-request-extension.schema.json` (JSON Schema 2020-12)
**Licence:** Apache 2.0, matching A2A

Cut from §2.9 of the Agent Capability Declaration Specification 0.9 draft (not yet published), so that it can be read on its own. Where this document and that draft differ, this document is the later text; the differences are listed in Annex C.

---

## 1. What this extension is for

An A2A agent can already say two things about a skill:

- **any client may call it without credentials** (no `securityRequirements` on the Agent Card or the skill);
- **the caller must present credentials** of a stated kind: a person's sign-in, or the calling software's own key or certificate (`securityRequirements` on the `AgentCard` or the `AgentSkill`, A2A §4.4.1, §4.4.5 and §7).

Both govern who may call the skill, not what the agent says in reply.

Many businesses work with a third state that A2A cannot express: **"we tell the person who asks, in conversation, but we do not publish it."** A funeral home gives prices on request. A clinic discusses fees with the patient. A law firm quotes after hearing the matter. None of this needs a login, and none of it is public.

Without a way to say so, an agent answering other agents has two bad choices. It gives on-request information to any caller, which publishes it by another route. Or it refuses, and the calling assistant tells the person nothing useful, or fills the gap from elsewhere.

This extension adds the third state. For each such skill, it says:

1. that the information is given **on request**;
2. **where a person can ask** for it (`personEntry`);
3. whether the business has chosen to give it to **an assistant acting for a person** (`assistantAccess`, off by default).

It also fixes how the agent replies when it refers instead of answering, so that the calling assistant can send the person to the right place.

In A2A's terms (extensions topic, "Scope of Extensions") this is a **data extension** (what the Agent Card declares) with a light **profile** on replies (metadata on existing objects). It adds no RPC methods, no task states and no fields to core objects.

## 2. Terms

**MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT** and **MAY** are as in RFC 2119.

| Term | Meaning here |
|---|---|
| **Operator** | The business the agent answers for. Named in `AgentCard.provider`. |
| **Agent** | The operator's A2A server. |
| **Client** | Any A2A client calling the agent. |
| **Assistant** | A client acting for a particular person (the **named person**) in the exchange. |
| **Public skill** | A skill with no `securityRequirements` and not listed in this extension. |
| **On-request skill** | A skill listed in this extension. |
| **Sign-in skill** | A skill with `securityRequirements`. |
| **Person-facing address** | A web address where a person can talk to the operator's agent or staff, or a telephone number (§3, OR-3.4). |

## 3. Declaration in the Agent Card

The extension is declared as an `AgentExtension` in `AgentCard.capabilities.extensions` (A2A §4.6.1).

| # | Requirement |
|---|---|
| OR-3.1 | An agent with any on-request skill MUST declare this extension, with `uri` as above and `required: false`. *A client that does not understand the extension still gets a usable reply (§5). A2A also advises against marking data extensions as required.* |
| OR-3.2 | `params` MUST contain `skills`: an array with one entry per on-request skill. |
| OR-3.3 | Each entry MUST have `id`, the `id` of an `AgentSkill` in the same card. No `id` may appear twice. |
| OR-3.4 | Each entry MUST have `personEntry`: an `https` address where a person can ask for this, on an origin the operator controls, or a `tel:` number answered by or for the operator. An action the person must carry out themselves (§6.3) MUST have an `https` address. |
| OR-3.5 | Each entry MAY have `assistantAccess` (boolean). Absent means `false`. `true` records the operator's opt-in (§5.3, §5.4). |
| OR-3.6 | An entry for a skill that changes something in the operator's systems (a booking, a request, a record) MAY have `completion`: `in-conversation` if the action is finished when the exchange ends; `deferred` if a member of staff completes it later. It MUST NOT be `in-conversation` where any step waits on a person. |
| OR-3.7 | A skill listed here MUST NOT also carry `securityRequirements`. A skill is on request or sign-in, not both. |
| OR-3.8 | `personEntry` MUST NOT reveal anything about the person or their circumstances (no query-string context; no path such as `/bereavement`). It is the same for every caller. |
| OR-3.9 | Skill text in the card (`name`, `description`, `examples`, `tags`) MUST NOT contain the value of anything given on request. An example may *ask* for it ("What does a cremation cost?"); it may not contain the answer. |

A skill **not** listed and without `securityRequirements` is public. Removing a skill from the list is withdrawal of the on-request marker, not an error.

## 4. Activation

Activation follows A2A unchanged (A2A §3.2.6, §4.6.1, extensions topic "Extension Activation").

- A client activates the extension by listing its URI in the `A2A-Extensions` service parameter (an HTTP header in the JSON-RPC and HTTP+JSON bindings, gRPC metadata in the gRPC binding).
- An agent that has activated it SHOULD echo the URI in its response's `A2A-Extensions`, as A2A recommends, and MUST list the URI in the `extensions` array of each Message or Artifact that carries this extension's metadata.
- An agent SHOULD also accept `X-A2A-Extensions`, the header name used by A2A 0.3 (extensions topic at v0.3.0), and SHOULD echo it in the spelling received. Clients built for 0.3 send only that spelling.
- Activation changes **only** whether replies carry structured metadata. It never changes what the agent gives or withholds (§5.1).

## 5. Agent behaviour

### 5.1 The same rules whether or not the client activates

| # | Requirement |
|---|---|
| OR-5.1 | The agent MUST follow §5.2 to §5.4 for every client, whether or not the client activated this extension. *Otherwise a client could get on-request information simply by not activating it.* |

### 5.2 Public and on-request skills, default case

| # | Requirement |
|---|---|
| OR-5.2 | A public skill is answered to any client. |
| OR-5.3 | Asked for an on-request skill where `assistantAccess` is not `true`, the agent MUST NOT give the information. It MUST say, in the reply text, that the operator gives this to the person who asks, and give the `personEntry` address. It MUST NOT answer as though the information did not exist. |

### 5.3 Information, where the operator has opted in

| # | Requirement |
|---|---|
| OR-5.4 | Where `assistantAccess` is `true` on an informational skill, the agent MAY give the information to an assistant acting for a named person. It MUST NOT give it to a client that is not acting for a particular person (for example a crawler or an aggregator), and treats such a client as in OR-5.3. |
| OR-5.5 | How an assistant shows that it acts for a named person is **not yet defined** (§9, item 1). Until it is, the operator's opt-in is the operator's acceptance of the assistant's own assertion. An operator-facing control for `assistantAccess` SHOULD say so. |

### 5.4 Actions, where the operator has opted in

| # | Requirement |
|---|---|
| OR-5.6 | Where `assistantAccess` is `true` on a skill that changes something in the operator's systems, the agent MAY carry it out at an assistant's request only if all of these hold: **(a)** the operator holds a current level-2 (legal entity) attestation under the Agent Capability trust framework (Annex B); **(b)** the change can be reversed by the operator's staff, and a named member of staff is told of it; **(c)** repeated or duplicate requests are limited per calling assistant as well as in total; **(d)** anything that cannot be undone is confirmed by the person themselves, in a form they see, before it is done. Confirmation passed on only by the assistant is not enough. |
| OR-5.7 | Otherwise the agent MUST NOT carry out the action, and MUST refer the person to `personEntry` as in OR-5.3. |
| OR-5.8 | Where the action is `deferred`, the agent MUST NOT tell the assistant it is done, and an assistant relaying it MUST NOT tell the person it is done. |

### 5.5 Sign-in and reaching a person

| # | Requirement |
|---|---|
| OR-5.9 | Sign-in is A2A's own. A skill that needs the person to sign in uses `securityRequirements` and `securitySchemes`, and in-task authorisation uses `TASK_STATE_AUTH_REQUIRED` (A2A §7.6). This extension does not describe sign-in. |
| OR-5.10 | Reaching a member of staff is an ordinary skill. Where a person can be reached, the card SHOULD carry a skill whose description says how (transfer, callback), when, and in which languages. Where nobody can be reached, the agent MUST say so when asked. |

## 6. Reply metadata

When the client has activated the extension, the agent marks its reply with a small object in `metadata`, keyed by the extension URI (A2A §3.2.5, §4.6.2).

### 6.1 The object

| Field | Required | Meaning |
|---|---|---|
| `status` | yes | `on-request`: information referred. `action-referred`: action not carried out, person referred. `given-on-opt-in`: information given to an assistant under OR-5.4. |
| `skillId` | yes | The `id` of the skill concerned. |
| `personEntry` | for `on-request` and `action-referred` | The address from the card. MAY be given with `given-on-opt-in`, so the person can follow up. |

### 6.2 Where it goes

A2A says results belong in Artifacts and communication in Messages (A2A §3.7). So:

| Case | The reply | Metadata on | Task state, if the agent made a task |
|---|---|---|---|
| Information referred (`on-request`) | A Message saying where to ask | that Message (`Message.metadata`); if inside a task, the final `TaskStatus.message` | `TASK_STATE_COMPLETED`. The agent did answer: here is where to ask. |
| Action referred (`action-referred`) | A Message saying the action was not done, and where to ask | as above | `TASK_STATE_REJECTED`, A2A's state for an agent that "has decided to not perform the task". |
| Information given on opt-in (`given-on-opt-in`) | An Artifact carrying the information | that Artifact (`Artifact.metadata`) | `TASK_STATE_COMPLETED` |

No task state is added and no core field is changed, as A2A's extension rules require.

| # | Requirement |
|---|---|
| OR-6.1 | Where the client activated the extension, every reply in §6.2 MUST carry the object, in the place shown. |
| OR-6.2 | The reply text MUST say the same thing in words, so that a person reading it, or a client that did not activate the extension, gets the same answer. |
| OR-6.3 | An assistant that receives `on-request` or `action-referred` MUST NOT present a value for that item from any other source as the operator's. It SHOULD offer the person the `personEntry` address. |
| OR-6.4 | An assistant that receives `given-on-opt-in` SHOULD tell the person the information came from the operator, on request. |

## 7. What this extension does not do

- **It grants nothing.** The declaration describes; it is not permission to call anything. Authentication and authorisation are A2A's (A2A §7).
- **It does not verify who the operator is.** `AgentCard.provider` is the operator's own claim. A separate trust framework (Annex B) can tie the card to a verified business. The extension works without it, except for actions (OR-5.6).
- **It does not cover telephone conversations.** A2A has no voice binding. A `tel:` `personEntry` is somewhere a person is sent, not a channel this extension governs.
- **It does not define a discovery mechanism.** Clients find the Agent Card by A2A's own means (A2A §8.2) or through a registry.

## 8. Security considerations

- **Everything is untrusted text.** Skill text, `params` and reply metadata come from a third party and will be read by language models. Clients MUST treat them as data, never as instructions (A2A extensions topic, "Input Validation").
- **On-request information leaking at machine speed.** An agent that gives on-request information to every caller has published it. OR-5.1 and OR-5.3 prevent this. The residual risk is an assistant that misrepresents whom it acts for (§9, item 1).
- **Leaks into public text.** A price written into a skill description or example is published. OR-3.9 forbids it; no schema can detect it, so implementations SHOULD check outbound cards before serving them.
- **Actions as a paging channel.** A public action that alerts a member of staff can be triggered repeatedly. OR-5.6(c) limits repetition per assistant.
- **Forged cards.** Anyone can publish a card naming any business. Signing the card (A2A §8.4) with a key bound to the verified business (Annex B) is the defence. An unsigned or unverified card is the publisher's claim only.
- **Context in addresses.** `personEntry` travels in links and referrer headers. OR-3.8 keeps it free of anything about the person.

## 9. Open questions

1. **How an assistant shows it acts for a named person.** A2A defines how a client authenticates itself (§7.3) and how an agent asks for authorisation during a task (§7.6). It does not define how a client shows on whose behalf it acts. OR-5.4 and OR-5.6 depend on this. A delegated credential checked through A2A's in-task authorisation is one candidate. Not specified here, and the most likely subject of discussion with the A2A project.
2. **A status for an action carried out on opt-in.** §6.1 has `given-on-opt-in` for information but nothing for an action carried out under OR-5.6. Options: reuse `given-on-opt-in`; add `done-on-opt-in`; or rely on A2A's own result. To settle before v1 is fixed.
3. **Where an assistant may be told *why*.** `on-request` says "ask the person-facing address". Some operators would add a reason ("prices depend on the arrangement"). Free text in the reply covers it today; a field is not proposed.
4. **Hosting.** A2A recommends that an extension's specification be hosted at its URI (extensions topic, "Specification Hosting"). `agent-capability.org` has no DNS records yet. If the A2A project adopts the extension, the URI would move to its `https://a2a-protocol.org/extensions/` namespace and this one would be retired.
5. **A2A 1.1.** The release candidate adds a live two-way stream and a task timeline. Neither affects this extension as read, but the reply placement in §6.2 should be rechecked against 1.1 when it is released.

---

## Annex A — Examples

All examples use reserved domains (RFC 2606) and a fictional business. Signature values are placeholders.

### A.1 Agent Card

`service-pricing` is on request, not opened to assistants. `arrangement-booking` is on request and opened to assistants. `opening-hours` is public. `transfer-status` needs sign-in, declared natively. `speak-to-staff` is the escalation skill.

```json
{
  "name": "Example Memorial Chapel assistant",
  "description": "Answers questions for Example Memorial Chapel. Gives public information to any caller. Information the chapel gives on request is given to the person who asks, at the address stated.",
  "supportedInterfaces": [
    {"url": "https://agent.example.com/a2a/v1", "protocolBinding": "JSONRPC", "protocolVersion": "1.0"}
  ],
  "provider": {"organization": "Example Memorial Chapel", "url": "https://example.com"},
  "version": "1.0.0",
  "capabilities": {
    "streaming": false,
    "pushNotifications": false,
    "extensions": [
      {
        "uri": "https://agent-capability.org/ext/on-request/v1",
        "description": "Skills whose information the chapel gives to the person who asks, and where a person asks.",
        "required": false,
        "params": {
          "skills": [
            {"id": "service-pricing", "personEntry": "https://ask.example.com/talk", "assistantAccess": false},
            {"id": "arrangement-booking", "personEntry": "https://ask.example.com/talk", "assistantAccess": true, "completion": "in-conversation"}
          ]
        }
      }
    ]
  },
  "securitySchemes": {
    "family-portal": {
      "openIdConnectSecurityScheme": {"openIdConnectUrl": "https://portal.example.com/.well-known/openid-configuration"}
    }
  },
  "defaultInputModes": ["text/plain"],
  "defaultOutputModes": ["text/plain"],
  "skills": [
    {"id": "opening-hours", "name": "Opening hours and location", "description": "States the chapel's opening hours and where to find it.", "tags": ["hours", "location"], "examples": ["When is Example Memorial Chapel open?"]},
    {"id": "service-pricing", "name": "Service costs", "description": "Costs and inclusions of the chapel's services. Given to the person who asks, on request.", "tags": ["pricing", "services"], "examples": ["What does a cremation cost at Example Memorial Chapel?"]},
    {"id": "arrangement-booking", "name": "Book an arrangement meeting", "description": "Checks director availability and books an arrangement meeting. The person confirms the booking themselves.", "tags": ["booking", "arrangements"]},
    {"id": "transfer-status", "name": "Transfer status", "description": "Confirms whether a person has been brought into the chapel's care. Requires the family member to sign in.", "tags": ["status"], "securityRequirements": [{"schemes": {"family-portal": {"list": ["openid"]}}}]},
    {"id": "speak-to-staff", "name": "Speak to a member of staff", "description": "Transfer to a funeral director, or a callback, Monday to Friday 09:00 to 17:00, in English. Outside those hours a callback is arranged for the next working day.", "tags": ["escalation", "human"]}
  ],
  "signatures": [{"protected": "eyJhbGciOiJFUzI1NiIsImtpZCI6ImNoYXBlbC0xIn0", "signature": "PLACEHOLDER"}]
}
```

### A.2 Information referred: a Message reply

The client activated the extension and asked for the cremation price. `service-pricing` is not opened to assistants.

```json
{
  "messageId": "m-2",
  "role": "ROLE_AGENT",
  "parts": [
    {"text": "Example Memorial Chapel gives its prices to the person who asks. You can ask here: https://ask.example.com/talk"}
  ],
  "extensions": ["https://agent-capability.org/ext/on-request/v1"],
  "metadata": {
    "https://agent-capability.org/ext/on-request/v1": {
      "status": "on-request",
      "skillId": "service-pricing",
      "personEntry": "https://ask.example.com/talk"
    }
  }
}
```

### A.3 Action referred: a Task ending REJECTED

An assistant asked to book a meeting, but the chapel holds only a level-1 attestation, so OR-5.6(a) fails.

```json
{
  "id": "t-41",
  "contextId": "c-9",
  "status": {
    "state": "TASK_STATE_REJECTED",
    "message": {
      "messageId": "m-7",
      "role": "ROLE_AGENT",
      "parts": [
        {"text": "I can't book this meeting for you. The person can book it here: https://ask.example.com/talk"}
      ],
      "extensions": ["https://agent-capability.org/ext/on-request/v1"],
      "metadata": {
        "https://agent-capability.org/ext/on-request/v1": {
          "status": "action-referred",
          "skillId": "arrangement-booking",
          "personEntry": "https://ask.example.com/talk"
        }
      }
    }
  }
}
```

### A.4 Information given on opt-in: an Artifact

A different operator has opened its pricing skill to assistants. The information is a result, so it travels as an Artifact.

```json
{
  "id": "t-52",
  "contextId": "c-12",
  "status": {"state": "TASK_STATE_COMPLETED"},
  "artifacts": [
    {
      "artifactId": "a-1",
      "name": "Cremation price",
      "parts": [{"text": "A direct cremation at Example Memorial Chapel is [price as the chapel gives it], including transfer and the certificate fee."}],
      "extensions": ["https://agent-capability.org/ext/on-request/v1"],
      "metadata": {
        "https://agent-capability.org/ext/on-request/v1": {
          "status": "given-on-opt-in",
          "skillId": "service-pricing",
          "personEntry": "https://ask.example.com/talk"
        }
      }
    }
  ]
}
```

### A.5 Activation over JSON-RPC

```http
POST /a2a/v1 HTTP/1.1
Host: agent.example.com
Content-Type: application/json
A2A-Version: 1.0
A2A-Extensions: https://agent-capability.org/ext/on-request/v1

{"jsonrpc": "2.0", "id": "1", "method": "SendMessage",
 "params": {"message": {"messageId": "m-1", "role": "ROLE_USER",
   "parts": [{"text": "What does a cremation cost at Example Memorial Chapel?"}]}}}
```

---

## Annex B — Relation to the Agent Capability trust framework

The extension stands alone for information. It needs one outside thing for actions: OR-5.6(a) requires a **level-2** attestation, which says the domain belongs to a named registered business on evidence independent of the domain (a company register entry, a licence, or the records of an association that checks either).

That framework is defined in the Agent Capability Declaration Specification (0.9 draft, Part 3). It is usable in two places:
- as the trusted key source for A2A card signatures (A2A §8.4.3 allows a key "from a trusted key store"): a verifier's attestation names the operator's card-signing key;
- as a trust manifest in Agentic Resource Discovery (ARD) entries.

Whether another framework offering the same assurance should satisfy OR-5.6(a) is open.

---

## Annex C — Changes from the 0.9 draft's §2.9

| # | Change | Why |
|---|---|---|
| 1 | Reply metadata for `given-on-opt-in` goes on the **Artifact**, not the Message | A2A §3.7: results in Artifacts, communication in Messages |
| 2 | Task states named: COMPLETED for a referred question, REJECTED for a referred action | 0.9 said only "an existing state" |
| 3 | The agent lists the URI in the `extensions` array of each Message or Artifact it marks, and echoes `A2A-Extensions` | A2A §4.6.2 examples and the extensions topic, "Extension Activation" |
| 4 | The conduct rules the extension depends on (0.9 P1.5.9, P1.5.10, P1.8.7, P1.6.2, P2.2.4, P2.5.6) are restated here in the extension's own terms | So the document reads on its own |
| 5 | OR-5.4: an opted-in skill is still not given to a client acting for nobody | Implicit in 0.9 P1.5.9 ("acting for a named person"); made explicit |
| 6 | OR-3.3: no duplicate `id` | Not stated in 0.9 |
| 7 | Open questions 2 and 4 added | Gaps found in drafting |
| 8 | A2A read at v1.0.1 (patch release); v1.1.0-rc checked | 0.9 cites 1.0.0, read on 31 Oct |

These should be carried back into the specification when 0.9 is next revised.

---

## Sources

- A2A Protocol Specification, `docs/specification.md` §3.2.5, §3.2.6, §3.7, §4.1, §4.6, §7.6, §8.4, at tags v1.0.0, v1.0.1 and v1.1.0-rc, github.com/a2aproject/A2A (fetched 1 Nov 2026).
- A2A `specification/a2a.proto`: `Message`, `Artifact`, `Task`, `TaskStatus`, `TaskState`, `AgentExtension`, `SendMessageResponse`, at the same tags.
- A2A `docs/topics/extensions.md` and `docs/topics/extension-and-binding-governance.md`, main branch (fetched 1 Nov 2026).
- Agent Capability Declaration Specification 0.9 draft (31 Oct 2026), §2.9, P1.5.9, P1.5.10, P1.8.7, Part 3.

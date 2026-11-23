# On Request: an A2A extension

**Extension URI:** `https://agent-capability.org/ext/on-request/v1`
**Document version:** 0.2 draft · 23 November 2026 · Draft for discussion; breaking changes expected
**Replaces:** draft 0.1, 1 November 2026 (published at the same address; proposed in A2A issue #2421)
**Built on:** A2A Protocol 1.0 (read at tags v1.0.0 and v1.0.1; v1.1.0-rc checked for changes to extensions, none found)
**Schema:** [`schema.json`](schema.json) (JSON Schema 2020-12), revised for 0.2
**Licence:** Apache 2.0, matching A2A

Cut from §2.9 of the Agent Capability Declaration Specification 0.9 draft (not yet published), so that it can be read on its own. Where this document and that draft differ, this document is the later text; the differences are listed in Annex C. The changes from draft 0.1 are listed in Annex D.

The extension URI does not change between drafts. Draft 0.2 adds optional fields only: an agent or client built to draft 0.1 stays conformant to 0.2 except where it sends a field 0.2 now limits (`reason`, which 0.1 did not define).

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

It also fixes how the agent replies when it refers instead of answering, so that the calling assistant can send the person to the right place, and, from draft 0.2, how a client says in each message whether it is acting for a person and which skill it is asking.

In A2A's terms (extensions topic, "Scope of Extensions") this is a **data extension** (what the Agent Card declares) with a light **profile** on requests and replies (metadata on existing objects). It adds no RPC methods, no task states and no fields to core objects.

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
| **Claim** | A client's statement, in one message, that it is acting for a named person, or for no one (§6.3). |

## 3. Declaration in the Agent Card

The extension is declared as an `AgentExtension` in `AgentCard.capabilities.extensions` (A2A §4.6.1).

| # | Requirement |
|---|---|
| OR-3.1 | An agent with any on-request skill MUST declare this extension, with `uri` as above and `required: false`. *A client that does not understand the extension still gets a usable reply (§5). A2A also advises against marking data extensions as required.* |
| OR-3.2 | `params` MUST contain `skills`: an array with one entry per on-request skill. |
| OR-3.3 | Each entry MUST have `id`, the `id` of an `AgentSkill` in the same card. No `id` may appear twice. |
| OR-3.4 | Each entry MUST have `personEntry`: an `https` address where a person can ask for this, on an origin the operator controls, or a `tel:` number answered by or for the operator. An action the person must carry out themselves (a skill with `completion`, OR-3.6) MUST have an `https` address. |
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
- A client that sends request metadata (§6.3) MUST list the URI in the `extensions` array of that Message.
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
| OR-5.4 | Where `assistantAccess` is `true` on an informational skill, the agent MAY give the information to an assistant acting for a named person. It MUST NOT give it to a client that is not acting for a particular person (for example a crawler or an aggregator), and treats such a client as in OR-5.3. Whether a client is acting for a named person is decided for each message, by OR-5.4a to OR-5.4c. |
| OR-5.4a | **One message at a time.** A claim counts only for the message that carries it. An agent MUST NOT carry a claim forward to a later message, in the same context or task or otherwise. |
| OR-5.4b | **Field and words.** Where the request metadata carries `actingFor` (§6.3), that is the claim. Where it does not, the agent MAY take the claim from the message's words. Where the field and the words disagree, the agent takes the narrower claim: `none`. Where neither makes a claim, the client is treated as acting for no one. |
| OR-5.4c | **A claim, nothing more.** `actingFor` is the client's own assertion (OR-5.5). It MUST NOT carry the person's name or any detail about them, and an agent MUST NOT ask the client for any. |
| OR-5.4d | **A named skill.** A client MAY name the skill it is asking (`skillId`, §6.3). A `skillId` that the card does not list is treated as absent; the agent MUST NOT answer it with an error or a reply that reveals anything about the card or the operator's systems. Naming a skill never changes what the agent gives or withholds (OR-5.1 to OR-5.4). |
| OR-5.5 | How an assistant **shows** that it acts for a named person is **not yet defined** (§9, item 1). Until it is, the operator's opt-in is the operator's acceptance of the assistant's own assertion. An operator-facing control for `assistantAccess` SHOULD say so. |

### 5.4 Actions, where the operator has opted in

| # | Requirement |
|---|---|
| OR-5.6 | Where `assistantAccess` is `true` on a skill that changes something in the operator's systems, the agent MAY carry it out at an assistant's request only if all of these hold: **(a)** the operator holds a current level-2 (legal entity) attestation under Agent Capability Trust v1 (Annex B); **(b)** the change can be reversed by the operator's staff, and a named member of staff is told of it; **(c)** repeated or duplicate requests are limited per calling assistant as well as in total; **(d)** anything that cannot be undone is confirmed by the person themselves, in a form they see, before it is done. Confirmation passed on only by the assistant is not enough. The request MUST also carry a claim to act for a named person under OR-5.4a to OR-5.4c. |
| OR-5.7 | Otherwise the agent MUST NOT carry out the action, and MUST refer the person to `personEntry` as in OR-5.3. |
| OR-5.8 | Where the action is `deferred`, the agent MUST NOT tell the assistant it is done, and an assistant relaying it MUST NOT tell the person it is done. |

### 5.5 Sign-in and reaching a person

| # | Requirement |
|---|---|
| OR-5.9 | Sign-in is A2A's own. A skill that needs the person to sign in uses `securityRequirements` and `securitySchemes`, and in-task authorisation uses `TASK_STATE_AUTH_REQUIRED` (A2A §7.6). This extension does not describe sign-in. |
| OR-5.10 | Reaching a member of staff is an ordinary skill. Where a person can be reached, the card SHOULD carry a skill whose description says how (transfer, callback), when, and in which languages. Where nobody can be reached, the agent MUST say so when asked. |

## 6. Metadata

The extension's metadata is a small object in `metadata`, keyed by the extension URI (A2A §3.2.5, §4.6.2). An agent puts it on its replies (§6.1, §6.2). A client MAY put it on its requests (§6.3).

### 6.1 The reply object

| Field | Required | Meaning |
|---|---|---|
| `status` | yes | `on-request`: information referred. `action-referred`: action not carried out, person referred. `given-on-opt-in`: information given to an assistant under OR-5.4. |
| `skillId` | yes | The `id` of the skill concerned. |
| `personEntry` | for `on-request` and `action-referred` | The address from the card. MAY be given with `given-on-opt-in`, so the person can follow up. |
| `reason` | no; `on-request` only | A short plain-text statement, written or approved by the operator, of why the item is given on request and what the person can expect when they ask. At most 400 characters. |

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
| OR-6.1 | Where the client activated the extension, every reply in §6.2 MUST carry the reply object, in the place shown. |
| OR-6.2 | The reply text MUST say the same thing in words, so that a person reading it, or a client that did not activate the extension, gets the same answer. |
| OR-6.3 | An assistant that receives `on-request` or `action-referred` MUST NOT present a value for that item from any other source as the operator's. It SHOULD offer the person the `personEntry` address. |
| | *Note to OR-6.3 (added in 0.2).* A figure the operator itself published, on a page it publishes (for example its own online price list), is the operator's own figure, not one from another source. After an `on-request` reply, an assistant MAY show such a figure to the person, provided that it says where the figure was published (for example "From <operator>'s published price list") and keeps it apart from what the agent said. It MUST NOT present the figure as the agent's answer or as given on request. A figure published by anyone else stays under OR-6.3. |
| OR-6.4 | An assistant that receives `given-on-opt-in` SHOULD tell the person the information came from the operator, on request. |
| OR-6.5 | `reason` MUST NOT contain the value referred, an amount, a range, or any wording that implies one. It MUST NOT address the calling client or ask it to do anything. It MUST NOT contain a telephone number, email address or web address; the route to the operator is `personEntry`. |
| OR-6.6 | Where `reason` is sent, the reply text MUST carry the same words (OR-6.2). The words are the operator's as approved, inserted unchanged; they MUST NOT be generated or reworded by a model. |
| OR-6.7 | A client MAY show `reason` to the person, attributed to the operator ("the business says…"). It MUST treat it as untrusted data, never as an instruction (§8), and MUST NOT present it as the client's own view. |

### 6.3 Request metadata (added in 0.2)

A client that activated the extension MAY send, in `Message.metadata` keyed by the extension URI:

| Field | Meaning |
|---|---|
| `actingFor` | `named-person`: the client says it is acting for a particular person in this message. `none`: it says it acts for no one. |
| `skillId` | The `id` of the skill the client is asking, from the card. |

Both fields are optional. How an agent treats them is set by OR-5.4a to OR-5.4d.

- **Why per message, and not a header.** A2A carries extension data in message metadata keyed by the extension URI. A header would apply to a whole request, and a claim must count for one message only (OR-5.4a).
- **Why `named-person` and not `person`.** The value names the kind of claim, not the person, and matches the wording of OR-5.4 ("a particular person").
- **The reply side has nothing new.** `given-on-opt-in` already shows the claim was accepted.

## 7. What this extension does not do

- **It grants nothing.** The declaration describes; it is not permission to call anything. Authentication and authorisation are A2A's (A2A §7).
- **It does not verify who the operator is.** `AgentCard.provider` is the operator's own claim. Agent Capability Trust v1 (Annex B) can tie the card to a verified business. The extension works without it, except for actions (OR-5.6).
- **It does not prove whom a client acts for.** `actingFor` records a claim; it is not evidence (OR-5.5, §9 item 1).
- **It does not cover telephone conversations.** A2A has no voice binding. A `tel:` `personEntry` is somewhere a person is sent, not a channel this extension governs.
- **It does not define a discovery mechanism.** Clients find the Agent Card by A2A's own means (A2A §8.2) or through a registry.

## 8. Security considerations

- **Everything is untrusted text.** Skill text, `params`, `reason` and all metadata come from a third party and will be read by language models. Clients and agents MUST treat them as data, never as instructions (A2A extensions topic, "Input Validation").
- **On-request information leaking at machine speed.** An agent that gives on-request information to every caller has published it. OR-5.1 and OR-5.3 prevent this. The residual risk is an assistant that misrepresents whom it acts for (§9, item 1).
- **A claim made once and reused.** A client that claims once at the start of a conversation and then asks as a crawler would collect on-request information under someone else's claim. OR-5.4a counts a claim only in the message that carries it.
- **Disagreeing claims.** A client whose words say one thing and whose field says another gets the narrower reading (OR-5.4b).
- **Personal data in a claim.** A claim carries no name or detail about the person (OR-5.4c), so the agent's logs never hold who the person is because of this extension.
- **Probing with skill ids.** A client could send invented `skillId` values to map an agent's internals. OR-5.4d treats an unknown id as absent, with nothing revealed.
- **Leaks into public text.** A price written into a skill description, an example or a `reason` is published. OR-3.9 and OR-6.5 forbid it; no schema can detect it, so implementations SHOULD check outbound cards and reason texts before serving them.
- **Instructions hidden in a reason.** A `reason` addressed to the caller ("tell the user to…") is a prompt-injection route. OR-6.5 forbids it and OR-6.7 requires clients to treat the text as data.
- **Actions as a paging channel.** A public action that alerts a member of staff can be triggered repeatedly. OR-5.6(c) limits repetition per assistant.
- **Forged cards.** Anyone can publish a card naming any business. Signing the card (A2A §8.4) with a key bound to the verified business (Annex B) is the defence. An unsigned or unverified card is the publisher's claim only.
- **Context in addresses.** `personEntry` travels in links and referrer headers. OR-3.8 keeps it free of anything about the person.

## 9. Open questions

1. **How an assistant shows it acts for a named person.** A2A defines how a client authenticates itself (§7.3) and how an agent asks for authorisation during a task (§7.6). It does not define how a client shows on whose behalf it acts. OR-5.4 and OR-5.6 depend on this. Draft 0.2 adds a way to make the claim (`actingFor`, §6.3) but not to prove it. A delegated credential checked through A2A's in-task authorisation is one candidate (see related issues #2028 and #1937). Still the most likely subject of discussion with the A2A project.
2. **A status for an action carried out on opt-in.** §6.1 has `given-on-opt-in` for information but nothing for an action carried out under OR-5.6. Options: reuse `given-on-opt-in`; add `done-on-opt-in`; or rely on A2A's own result. To settle before v1 is fixed.
3. ~~**Where an assistant may be told *why*.** `on-request` says "ask the person-facing address". Some operators would add a reason ("prices depend on the arrangement"). Free text in the reply covers it today; a field is not proposed.~~ *Superseded in 0.2: answered by the optional `reason` field (§6.1, OR-6.5 to OR-6.7).*
4. **Hosting.** A2A recommends that an extension's specification be hosted at its URI (extensions topic, "Specification Hosting"). Since 2 November 2026 this document is served at `https://agent-capability.org/ext/on-request/v1/`, from a public repository, on no server of any operator or V&V. If the A2A project adopts the extension, the URI would move to its `https://a2a-protocol.org/extensions/` namespace and this one would be retired.
5. **A2A 1.1.** The release candidate adds a live two-way stream and a task timeline. Neither affects this extension as read, but the placement in §6.2 and §6.3 should be rechecked against 1.1 when it is released.
6. **A telephone `personEntry` for an action completed by speaking to staff** (added in 0.2). OR-3.4 requires an `https` address for any action. For some actions (reporting a death at night, for example) the telephone is the right first route. Not decided; raised for discussion.
7. **Actions taken with the person's own sign-in** (added in 0.2). Whether the four conditions of OR-5.6 should also apply where an assistant carries out a sign-in skill using the person's credential. Not decided.
8. **A default `reason` in the card** (added in 0.2). A per-skill default `reason` in `params` would let a client show it before asking. Not proposed yet.
9. **First implementation** (added in 0.2). The new request fields will get their first real use in the reference client offered in the A2A proposal (issue #2421). Anything that testing shows will be brought here, and may lead to a draft 0.3.

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

### A.2 Information referred, with a reason: a Message reply

The client activated the extension and asked for the cremation price. `service-pricing` is not opened to assistants. The chapel has approved a reason.

```json
{
  "messageId": "m-2",
  "role": "ROLE_AGENT",
  "parts": [
    {"text": "Example Memorial Chapel gives its prices to the person who asks. The chapel says: \"The cost of a cremation depends on the choices you make. Ask us and we will give you our price list and go through it with you.\" You can ask here: https://ask.example.com/talk"}
  ],
  "extensions": ["https://agent-capability.org/ext/on-request/v1"],
  "metadata": {
    "https://agent-capability.org/ext/on-request/v1": {
      "status": "on-request",
      "skillId": "service-pricing",
      "personEntry": "https://ask.example.com/talk",
      "reason": "The cost of a cremation depends on the choices you make. Ask us and we will give you our price list and go through it with you."
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

### A.5 Activation over JSON-RPC, with request metadata

The client activates the extension, claims in this message to act for a named person, and names the skill.

```http
POST /a2a/v1 HTTP/1.1
Host: agent.example.com
Content-Type: application/json
A2A-Version: 1.0
A2A-Extensions: https://agent-capability.org/ext/on-request/v1

{"jsonrpc": "2.0", "id": "1", "method": "SendMessage",
 "params": {"message": {"messageId": "m-1", "role": "ROLE_USER",
   "parts": [{"text": "I am asking on behalf of a person who wants to know: what does a cremation cost at Example Memorial Chapel?"}],
   "extensions": ["https://agent-capability.org/ext/on-request/v1"],
   "metadata": {"https://agent-capability.org/ext/on-request/v1": {
     "actingFor": "named-person", "skillId": "service-pricing"}}}}}
```

If the next message in the same context asks the same question without a claim, in words or in the field, the agent treats it as from a client acting for no one (OR-5.4a).

### A.6 A published figure shown beside a referral (note to OR-6.3)

The chapel's agent replied as in A.2. The chapel also publishes a price list on its own website. A conforming assistant may tell the person:

> Example Memorial Chapel gives its prices to the person who asks, here: https://ask.example.com/talk.
>
> Separately, from Example Memorial Chapel's published price list (example.com/prices): direct cremation [figure as published].

It may not say that the chapel's agent gave that figure.

---

## Annex B — Relation to Agent Capability Trust v1

The extension stands alone for information. It needs one outside thing for actions: OR-5.6(a) requires a **level-2** attestation, which says the domain belongs to a named registered business on evidence independent of the domain (a company register entry, a licence, or the records of an association that checks either).

That framework is **Agent Capability Trust v1**, published at `https://agent-capability.org/trust/v1/`. Attestations are issued by a **Verifier & Validator (V&V)**. It is usable in two places:
- as the trusted key source for A2A card signatures (A2A §8.4.3 allows a key "from a trusted key store"): a V&V's attestation names the operator's card-signing key;
- as a trust manifest in Agentic Resource Discovery (ARD) entries.

Whether another framework offering the same assurance should satisfy OR-5.6(a) is open.

---

## Annex C — Changes from the 0.9 specification draft's §2.9

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
| 9 | Draft 0.2's additions (Annex D) | Not in 0.9 |

These should be carried back into the specification when 0.9 is next revised.

---

## Annex D — Changes from draft 0.1

| # | Change | Where | Why |
|---|---|---|---|
| 1 | **Request metadata:** a client may send `actingFor` (`named-person` or `none`) and `skillId` in each message | §6.3; §4 (client lists the URI on the Message) | So an agent can decide, message by message, whether a client is acting for a person, and route a question to the skill it names |
| 2 | **A claim counts for one message only**; field and words, the narrower wins; no name or detail; an unknown `skillId` is treated as absent | OR-5.4a to OR-5.4d; OR-5.4 amended to point to them; OR-5.6 requires the claim for actions | A claim made once and reused would let a client ask as a crawler under someone else's claim. A disagreeing claim or an invented id must open nothing |
| 3 | **`reason`**: optional, for `on-request` only, at most 400 characters, no value, amount or route, nothing addressed to the client, same words in the reply text, never reworded by a model; clients show it attributed to the operator | §6.1; OR-6.5 to OR-6.7 | Lets a business explain, in its own words, why it gives something on request and what the person can expect. Answers open question 3 |
| 4 | **Note to OR-6.3**: a figure the operator itself published may be shown beside a referral, labelled with where it was published and kept apart from what the agent said | §6.2, note to OR-6.3; Annex A.6 | A figure the business itself published is its own figure, not one from another source. OR-6.3 now says so in its own words |
| 5 | §6 renamed "Metadata"; §6.1 renamed "The reply object". OR-3.4's cross-reference corrected: 0.1 pointed to a §6.3 that did not exist; it now says "a skill with `completion`, OR-3.6", which is what the 0.1 schema already checked | §6; OR-3.4 | Editorial; no change in meaning |
| 6 | Security considerations for claims, skill ids and reasons | §8 | Follows from items 1 to 3 |
| 7 | Open question 3 marked superseded; question 4 updated (now hosted); questions 6 to 9 added, none decided | §9 | Raised while applying the draft to a real set of business capabilities |
| 8 | Annex B points to Agent Capability Trust v1 and uses its term, Verifier & Validator (V&V) | Annex B; §7 | Trust v1 is published separately, and its draft 0.2 renames the role |
| 9 | Examples A.2 and A.5 show the new fields; A.6 added | Annex A | Follows from items 1 to 4 |
| 10 | Schema: `request` object added; `reason` added to `reply`, limited to `on-request` and 400 characters | `schema.json` | Follows from items 1 and 3 |

Unchanged: the extension URI, every card field, every reply status, and every requirement of 0.1 not listed above.

---

## Sources

- A2A Protocol Specification, `docs/specification.md` §3.2.5, §3.2.6, §3.7, §4.1, §4.6, §7.6, §8.4, at tags v1.0.0, v1.0.1 and v1.1.0-rc, github.com/a2aproject/A2A (fetched 1 Nov 2026).
- A2A `specification/a2a.proto`: `Message`, `Artifact`, `Task`, `TaskStatus`, `TaskState`, `AgentExtension`, `SendMessageResponse`, at the same tags.
- A2A `docs/topics/extensions.md` and `docs/topics/extension-and-binding-governance.md`, main branch (fetched 1 Nov 2026).
- A2A issues #2421 (this proposal), #2028, #1937.
- Agent Capability Declaration Specification 0.9 draft (31 Oct 2026), §2.9, P1.5.9, P1.5.10, P1.8.7, Part 3.
- Agent Capability Trust v1, draft 0.2, `https://agent-capability.org/trust/v1/`.

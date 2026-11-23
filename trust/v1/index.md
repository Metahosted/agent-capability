# Agent Capability Trust v1

**Framework identifier:** `https://agent-capability.org/trust/v1`
**Document version:** 0.2 draft · 23 November 2026 · Draft for discussion; breaking changes expected
**Replaces:** draft 0.1, 1 November 2026 (published at the same address)
**Schema:** [`schema.json`](schema.json) (JSON Schema 2020-12); 0.2 changes its description only
**Licence:** Apache 2.0

A trust framework that ties an agent, and the domain it answers from, to a verified business. It is usable in two places:
- as a **trust manifest in Agentic Resource Discovery (ARD)** catalogue entries (ARD v0.91 §4.5);
- as the **trusted key source for A2A Agent Card signatures** (A2A v1.0 §8.4.3).

Cut from Part 3 of the Agent Capability Declaration Specification 0.9 draft and the Verification Mechanism Specification 0.5.7 (neither yet published), so it can be read on its own. It adds one principle: only a certified V&V can make a business "verified", and nothing the business publishes about itself can claim it. Differences from those documents are listed in Annex C. The changes from draft 0.1 are listed in Annex D.

**What changed in 0.2, in brief.** The role that verifies is now called a **Verifier & Validator (V&V)**. An organisation that runs a V&V may also run ARD registry services, provided it gives no preference to its own attestations. A V&V may offer supplementary services, such as helping a business list its agent with ARD registries, on conditions that keep them separate from verification. Field names in documents and schemas are unchanged.

---

## 1. What this framework proves, and what it doesn't

Anyone can publish an Agent Card naming any business. A2A's `provider` field is the publisher's own claim. A card signature proves only that whoever holds the key signed it. ARD binds an entry to the domain in its identifier, but leaves "how do we know it's really them" to whatever trust framework the entry names.

This framework answers that question at two levels:

| Level | It proves | It does not prove |
|---|---|---|
| **1: domain control** | The business itself, not its website vendor, controls the domain the agent answers for. | Who the business is. Anyone controls their own domain perfectly, including an impersonator with a plausible one. |
| **2: legal entity** | The domain belongs to a named, registered business, on evidence that does not come from the domain: a company register entry, a licence, or the records of an association that checks either. | That anything the business says is true, or that its agent works. |

**No attestation, at either level, asserts quality, accuracy, fitness, or that any capability exists or works.** Verification says who is speaking, never whether they are right.

## 2. Terms

**MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT** and **MAY** are as in RFC 2119.

| Term | Meaning |
|---|---|
| **Operator** | The business the agent answers for. |
| **Implementer** | Whoever builds or hosts the agent. May be the operator or a vendor. |
| **Verifier & Validator (V&V)** | A service that checks an operator against §4 and issues attestations (§5). It may also validate (§9). *Called "verifier" in draft 0.1.* Certification (§7) applies to its verification. |
| **V&V operator** | The organisation that runs a V&V. It may run other services as well (§3). |
| **Certified V&V** | A V&V on the current trust list (§7.2). |
| **Attestation** | A document, signed by a V&V, stating what it checked about an operator and at what level. |
| **Attestation log** | A public, append-only record of every attestation a V&V issues and revokes (§7.3). |
| **Trust list** | The public list of certified V&Vs (§7.2). |
| **List keeper** | The neutral body that maintains the trust list (§7.2). |
| **Consumer** | Anyone who reads an attestation and acts on it: an assistant, an ARD registry, a validator. |
| **Validator** | A service that checks cards, catalogues and sites against this framework and the Agent Capability specification, and reports. Advisory only (§9). A service that only validates is a validator, not a V&V, and needs no certification. |
| **Supplementary service** | Anything a V&V offers operators besides verification (§3.3). |

**Names on the wire are unchanged.** The attestation payload's `verifier` field (§5.2) and the trust list's `verifiers` array (schema `trustList`) keep their 0.1 names. They hold the V&V's domain and the list of certified V&Vs.

## 3. Who may be a V&V, and what else it may do

### 3.1 Anyone, on identical terms

| # | Requirement |
|---|---|
| TF-3.1 | Any organisation MAY operate a V&V. Its attestations count as verified only under §7. |
| TF-3.2 | A V&V MUST offer verification on identical terms to every operator, whoever implemented the operator's agent and whoever operates the V&V. |
| TF-3.3 | A V&V MUST NOT require that it be the only V&V for an operator, and MUST NOT fail a check because another V&V's artefacts are present. Several V&Vs per operator is the expected case. |
| TF-3.4 | A level-2 attestation MAY be issued by the holder of the evidence itself, such as a licensing board or a trade association, acting as a V&V under this framework. Level-1 and level-2 attestations MAY come from different V&Vs. |
| ~~TF-3.5 (0.1)~~ | ~~A verifier MUST NOT keep or publish an index of operators for searching, MUST NOT aggregate authority across operators, and MUST NOT ingest, store or publish any agent's conversation content. Searching is ARD's job; a verifier that also searched would become a gatekeeper.~~ *Superseded in 0.2 by TF-3.5 and TF-3.6 below.* |
| TF-3.5 | **The verification service itself** MUST NOT keep or publish an index of operators for searching, MUST NOT aggregate authority across operators, and MUST NOT ingest, store or publish any agent's conversation content. *Verification stays a narrow function: it says who is speaking, and nothing more.* |

### 3.2 A V&V operator that also runs ARD services

ARD registries (ARD v0.91) are run by many organisations. Barring all of them from also running a V&V would remove likely V&Vs without making verification fairer. What matters is that a registry run by a V&V operator does not prefer its own attestations.

| # | Requirement |
|---|---|
| TF-3.6 | A V&V operator MAY also operate an ARD registry or another service that lets people or software find agents, provided all of these hold: **(a) no discrimination in weightings:** the registry weighs a current attestation from any certified V&V the same in filtering and ranking, whoever issued it, and gives no weight to whether the operator is the V&V operator's own customer; **(b) separate services:** the registry and the V&V run as separate services, so that the V&V's decisions do not depend on the registry and the registry has no access to the V&V's records beyond what any consumer can read (§5.4, §7.3); **(c) disclosed:** the V&V discloses that its operator runs the registry, under TF-7.4; **(d) audited:** the audit (Annex A, item 10) tests the registry for self-preference; **(e)** the V&V still holds no conversation content (TF-3.5). |

### 3.3 Supplementary services

| # | Requirement |
|---|---|
| TF-3.7 | A V&V MAY offer supplementary services of its choice, provided they do not contravene this framework. For example, it may tell operators that they can submit their agent's address to ARD registries, or submit it on their behalf. |
| TF-3.8 | A supplementary service MUST be offered on identical terms to every operator (TF-3.2), MUST NOT be a condition of verification, and declining it MUST NOT affect verification. |
| TF-3.9 | Where a V&V submits an operator's agent to a registry, it MUST do so only at the operator's request, and only to the registries the operator chooses. Who submitted an agent carries no weight under this framework; only the attestation does. A registry relying on this framework MUST NOT give weight to which V&V submitted an entry. |

## 4. Methods

### 4.1 Level 1: domain control

Two methods are defined, identified in ARD's `verificationMethods` as below.

| # | Requirement |
|---|---|
| TF-4.1 | **`agent-capability-dns-01`.** The V&V issues a token: at least 256 bits of random data, base64url without padding. The operator publishes `agent-capability-verification=<token>` as a TXT record at `_agent-capability.<host>`, where `<host>` is the host the agent or card is served from, or at the same label under any ancestor up to the operator's registrable domain. A V&V MUST support this method. |
| TF-4.2 | The V&V MUST NOT accept a record at `<host>` itself. Where `<host>` is a CNAME to the implementer, a TXT query there is answered from the implementer's zone, so the implementer could pass the check alone. |
| TF-4.3 | **`agent-capability-http-01`.** The operator's origin serves the token in a JSON document at `/.well-known/agent-capability-challenge`. A V&V MAY support this method. Where `<host>` resolves through a CNAME to an origin outside its own registrable domain, this method alone MUST NOT be relied on, because only the implementer can publish at that origin. |
| TF-4.4 | DNS checks MUST: query every authoritative nameserver for the zone; validate DNSSEC where the zone is signed and record whether the answer was authenticated; not use a resolver the operator nominates; treat a mixed answer as present; and retry a negative answer after at least 60 seconds before reporting failure. |
| TF-4.5 | The challenge artefact MUST stay published for the life of the attestation, and the V&V MUST re-check it at least every 30 days. It MUST revoke only after the artefact is absent on at least three consecutive checks spanning at least 72 hours. A new owner of the domain will not reproduce a token they never received, so a change of hands revokes without anyone having to notice the sale. |
| TF-4.6 | Deleting the TXT record is how an operator asks for revocation. A V&V MAY offer a faster route, but MUST NOT revoke on a request it cannot tie to control of the zone. |

The full procedures, with the tests that established them, are in the Verification Mechanism Specification 0.5.7, Parts 2 to 5. Where that document and this one differ, this one is the later text.

### 4.2 The card-signing key

| # | Requirement |
|---|---|
| TF-4.7 | **New.** Where the operator signs its Agent Card (A2A §8.4), it MAY have the signing key named in its level-1 attestation. It does so by publishing the key's JWK thumbprint (RFC 7638, SHA-256, base64url) as a TXT record `agent-capability-key=<thumbprint>` at the same `_agent-capability` label as TF-4.1. |
| TF-4.8 | The V&V MUST fetch the public key (from the `jku` in the card's signature header, or as supplied by the operator), compute its thumbprint, and name the key in the attestation only if it matches a published record. |
| TF-4.9 | Several key records MAY be present, for key rotation. Removing a key record has the same effect on that key as TF-4.5 has on the attestation. |

*Why DNS:* only whoever controls the operator's zone can publish there. A key named this way was chosen by the business, not by its vendor, which limits what a compromised implementer can forge.

### 4.3 Level 2: the legal entity

| # | Requirement |
|---|---|
| TF-4.10 | **`agent-capability-legal-entity`.** A level-2 attestation requires a current level-1 attestation for the same domain, plus at least one of: **(a)** an entry in a government register of companies or business names in the operator's jurisdiction; **(b)** a current licence issued by the public body that licenses the operator's activity (a funeral establishment licence, a state bar record, a contractor or veterinary premises licence); **(c)** the records of a trade or professional association that itself checks (a) or (b) for its members and says that it does. |
| TF-4.11 | The V&V MUST check the evidence at its source. A document the operator uploads, not checked with its issuer, is not evidence. |
| TF-4.12 | The V&V MUST establish that the entity in the evidence holds the level-1 domain, by at least one of: **(a)** the evidence itself naming the domain or website; **(b)** a challenge sent to a contact point taken from the evidence holder's record, not from the operator's website; **(c)** the evidence holder confirming the domain to the V&V. A name match alone is not enough. |

## 5. The attestation

### 5.1 Form

| # | Requirement |
|---|---|
| TF-5.1 | An attestation is a JWS (RFC 7515, compact serialisation, media type `application/jose`) whose payload is the JSON object of §5.2, signed by the V&V. |
| TF-5.2 | The protected header MUST carry `alg`, `kid` and `jku`, and `jku` MUST be on the **V&V's** own domain, never the operator's or the implementer's. A key on the card's or catalogue's own origin protects nothing: whoever can rewrite the card can rewrite the key. |
| TF-5.3 | The V&V's signing keys MUST be those named for it on the trust list (§7.2), once a list exists. |

### 5.2 Payload

| Field | Required | Meaning |
|---|---|---|
| `framework` | yes | `https://agent-capability.org/trust/v1` |
| `id` | yes | Stable identifier, unique at the V&V |
| `verifier` | yes | The V&V's registrable domain (field name kept from 0.1) |
| `level` | yes | `1` or `2` |
| `domain` | yes | The operator's registrable domain |
| `agentCard` | where there is one | URL of the Agent Card covered |
| `declaration` | where one is retained | URL of the capability declaration covered |
| `cardKeys` | optional | Card-signing keys bound under §4.2: `kid`, `thumbprint` |
| `method` | yes | The methods relied on, from §4 |
| `checked` | yes | The exact label or URL checked, and whether DNSSEC authenticated it, so a third party can repeat the check |
| `entity` | level 2 only | §5.3 |
| `issued`, `expires` | yes | RFC 3339 |
| `log` | yes | URL of the V&V's attestation log (§7.3) |

| # | Requirement |
|---|---|
| TF-5.4 | The payload MUST NOT state anything about capability, quality, accuracy, fitness or business category. |

### 5.3 Level 2 fields

| # | Requirement |
|---|---|
| TF-5.5 | `entity` MUST state: the legal name; any trading name relied on; the jurisdiction; the register or licensing body; the record identifier there; how the domain link was established (TF-4.12 a, b or c); and the date checked. It MUST NOT state anything further about the operator. |
| TF-5.6 | The Agent Card's `provider.organization`, and the declaration's `operator.name` where one is retained, MUST equal the legal name or a trading name in `entity`. Otherwise a consumer MUST treat the level-2 attestation as not applying. |

### 5.4 Expiry, status and revocation

| # | Requirement |
|---|---|
| TF-5.7 | A level-1 attestation expires at most 90 days after issue. A level-2 attestation expires at the earliest of: 12 months after checking; the expiry of the evidence, such as a licence renewal date; and the lapse of the level-1 attestation it rests on. The V&V MUST re-check at the source before renewing. |
| TF-5.8 | Each attestation MUST have a status (`valid`, `expired` or `revoked`) that anyone can retrieve from the V&V without registering and without contacting the operator. It MAY be cached for at most one hour. |
| TF-5.9 | A V&V MUST revoke when: the domain changes hands (TF-4.5); level-2 evidence lapses; it finds that the operator's card or declaration misrepresents capability; or the operator asks (TF-4.6). Every revocation MUST be logged (§7.3). |

## 6. Where an attestation is used

### 6.1 ARD

ARD defines no verification procedure of its own. An entry's `trustManifest` names a framework in `trustSchema`, and a registry verifies the way that framework says (ARD §4.5.2).

| # | Requirement |
|---|---|
| TF-6.1 | An ARD entry relying on this framework MUST carry a `trustManifest` with: `identity` = `https://<operator registrable domain>`, which matches the publisher in the entry's identifier as ARD §4.5.1 requires; `identityType` = `https`; and `trustSchema` with `identifier` = `https://agent-capability.org/trust/v1`, `version` = `1.0`, `governanceUri` = the published framework, and `verificationMethods` listing the methods relied on. |
| TF-6.2 | `attestations` MUST list each current attestation, with `type` = `agent-capability:domain-control` (level 1) or `agent-capability:legal-entity` (level 2), `uri` = the attestation at the V&V, `mediaType` = `application/jose`, and `digest`. |
| TF-6.3 | A registry or consumer MUST check that the attestation names the same Agent Card URL as the entry it appears on. ARD's manifest signature covers the trust manifest, not the entry, so it does not bind the two. |
| TF-6.3a | **Added in 0.2.** A registry that relies on this framework MUST weigh a current attestation from any certified V&V the same in filtering and ranking, whoever issued it (compare TF-3.6(a), which binds a registry run by a V&V operator). |

### 6.2 A2A card signatures

A2A lets a client take a card-signing key "from a trusted key store" (A2A §8.4.3) but does not say which stores to trust.

| # | Requirement |
|---|---|
| TF-6.4 | A consumer MAY treat a card-signing key named in a current attestation from a certified V&V as from a trusted key store. A card signed with that key, whose `provider.url` is on the attested domain, is then the attested operator's card. |
| TF-6.5 | The consumer finds the attestation through the ARD entry (§6.1), or by asking the V&V's attestation log for the domain (§7.3). The card itself does not point to it (§8). |

## 7. Certified V&Vs only

A business is "verified" only if a certified V&V says so.

### 7.1 Certification

| # | Requirement |
|---|---|
| TF-7.1 | A V&V is certified when it has passed an independent audit against the criteria in Annex A, covering the preceding period of operation, and is on the trust list. Certification applies to verification. Validation on its own needs none (TF-9.5). |
| TF-7.2 | The audit MUST be carried out by an auditor independent of the V&V (no ownership, and no commercial relationship beyond the audit itself) who is qualified to conduct assurance engagements under a recognised standard, such as ISAE 3000 or the AICPA attestation standards. |
| TF-7.3 | The audit MUST be repeated at least every 12 months. The report MUST be published, in full or as a summary that states the criteria tested, the period, any exceptions found, and the auditor's conclusion. |
| TF-7.4 | A V&V MUST disclose any relationship with the implementers whose customers it verifies, and any registry or search service its operator runs (TF-3.6(c)). *(For example, ConvoGate is operated by Metahosted, which also implements agents, and ConvoGate's disclosure must say so.)* |

### 7.2 The trust list

| # | Requirement |
|---|---|
| TF-7.5 | The trust list MUST be kept by a list keeper that is not itself a V&V and does not compete with operators or implementers. It MUST NOT be ConvoGate or Metahosted. |
| TF-7.6 | The list MUST be published at a stable `https` address, signed by the list keeper, and readable by anyone without registration. |
| TF-7.7 | Each entry MUST give: the V&V's name and domain; the levels it may attest; the URL of its signing keys; the URL of its attestation log; the date and URL of its latest audit report; and the date by which the next audit is due. |
| TF-7.8 | The list keeper MUST remove a V&V whose audit is overdue, whose audit finds it does not meet Annex A, or which fails to log as §7.3 requires. Removal MUST be published with the date and reason. Attestations issued after removal are not verified. |
| TF-7.9 | **Consumers MUST accept an attestation as verification only if its signer was on the trust list when it was issued and is on it now.** Any other attestation is treated as absent, and the card or entry as unverified, not as invalid. |

### 7.3 The attestation log

| # | Requirement |
|---|---|
| TF-7.10 | Every V&V MUST record every attestation it issues, and every revocation, in a public attestation log, before or at the moment the attestation takes effect. |
| TF-7.11 | The log MUST be append-only, with each entry and its position provable. Logs built on the design of Certificate Transparency (RFC 9162), or a transparency service under the IETF SCITT architecture, meet this requirement. |
| TF-7.12 | Anyone MUST be able to read the log without registering, look up every attestation issued for a given domain, and obtain proof that a given attestation is in the log. |
| TF-7.13 | A consumer MUST NOT accept an attestation that the issuing V&V's log does not contain. |
| TF-7.14 | Operators SHOULD watch the logs for their own domain. An attestation the operator did not ask for is the first sign of a mistaken or dishonest V&V, and is grounds for complaint to the list keeper. |

### 7.4 Getting started

No list keeper exists yet. Until one does, certification cannot be required, but the log can.

| # | Requirement |
|---|---|
| TF-7.15 | **Stage 1, until a list keeper publishes a trust list:** the log (§7.3) is required of every V&V from the start. Each consumer decides for itself which V&Vs to accept, and SHOULD accept only V&Vs that keep a log and have published an audit report against Annex A. |
| TF-7.16 | **Stage 2, from the date a list keeper publishes the first trust list:** TF-7.9 applies in full. |
| TF-7.17 | ConvoGate, as the reference V&V, will submit to audit against Annex A during stage 1 and publish the report. *(Non-normative: a commitment by the authors, recorded here so it can be checked.)* |

## 8. No self-claims

Verification is stated by a V&V, at the V&V, or not at all.

| # | Requirement |
|---|---|
| TF-8.1 | An Agent Card has no field for verification status, and none may be added. The same applies to a capability declaration and to an ARD entry outside its `trustManifest`. |
| TF-8.2 | The free text of an Agent Card (`name`, `description`, skill `name`, `description`, `tags` and `examples`), of a capability declaration, and of an ARD entry (`displayName`, `description`, `representativeQueries`) MUST NOT claim that the operator is verified, certified or accredited, or endorsed by any V&V, whether or not it is. |
| TF-8.3 | A consumer MUST disregard any such claim, and MUST NOT repeat it to a person as fact. |
| TF-8.4 | A consumer MAY tell a person that a business is verified only on the basis of an attestation it has checked under §7, and SHOULD say at which level and by whom. |

*Why even true claims are barred:* a true "verified" in a card is textually identical to a false one. Allowing it would teach assistants to read it, and impersonators would write it.

## 9. Validators

| # | Requirement |
|---|---|
| TF-9.1 | A validator MUST report any claim barred by TF-8.2. |
| TF-9.2 | A validator MUST be advisory: failing validation has no consequence beyond a report. It MUST work against any domain, not only verified ones. |
| TF-9.3 | A validator's report is not verification and MUST NOT be presented as such. Where it reports that an operator is verified, it MUST name the attestation and the V&V, and MUST have checked the attestation under §7. |
| TF-9.4 | A validator SHOULD check: that a named card-signing key matches a published thumbprint (TF-4.7); that `provider.organization` matches a level-2 `entity` (TF-5.6); and that the attestation names the card it is found with (TF-6.3). |
| TF-9.5 | **Added in 0.2.** Anyone MAY operate a validator, with no certification. A service that validates but does not verify is a validator, not a V&V, and MUST NOT describe itself as a V&V. |

## 10. Governance

| # | Requirement |
|---|---|
| TF-10.1 | This framework MUST be published at its identifier URI under a permissive licence, and MUST NOT be held by any party that competes with adopters. |
| TF-10.2 | Nothing in it may make an operator's agent depend on a V&V to work. If every V&V and the trust list disappeared, every agent MUST still be reachable and usable; only the verified status is lost. |
| TF-10.3 | Being verified MUST NOT be a condition of conforming to the Agent Capability specification. It earns authority with consumers, and is required only for actions an assistant asks the agent to carry out, and for sign-in and transactional capability presented by a consumer. |
| TF-10.4 | The framework SHOULD be offered to ARD's maintainers for recognition, and the audit criteria and trust list to a neutral body able to act as list keeper. |

## 11. Threats this addresses

| Threat | How |
|---|---|
| An impersonator publishes a card for a real business | Without control of the business's zone it cannot pass TF-4.1, so no attestation names its card. |
| An impersonator with a plausible domain of its own | Passes level 1, fails level 2: the evidence does not come from the domain (TF-4.10, TF-4.11). |
| The vendor verifies itself as the business | DNS in the operator's zone is required where the host is delegated (TF-4.2, TF-4.3), and the card key is chosen in the same zone (TF-4.7). |
| A card claims "verified" | It can't in a field (TF-8.1), mustn't in text (TF-8.2), and consumers ignore it (TF-8.3). |
| A careless or dishonest V&V | Must pass audit to be listed (§7.1); everything it issues is public (§7.3); operators can see attestations they never asked for (TF-7.14); the list keeper removes it (TF-7.8). |
| A V&V becomes a gatekeeper | Identical terms (TF-3.2), several V&Vs (TF-3.3), no index in the verification service (TF-3.5), agents work without it (TF-10.2), list kept by a neutral body (TF-7.5). |
| **A V&V operator's registry ranks its own customers higher** (added in 0.2) | No discrimination in weightings (TF-3.6(a), TF-6.3a); separate services (TF-3.6(b)); disclosed (TF-7.4); audited for self-preference (Annex A, item 10). |
| **A V&V ties verification to its other services** (added in 0.2) | Supplementary services on identical terms, never a condition, and declining has no effect (TF-3.8); audited (Annex A, item 13). |
| **Registry submission used to gain weight** (added in 0.2) | Only at the operator's request (TF-3.9); who submitted carries no weight, only the attestation does (TF-3.9). |
| A domain is sold | The new owner does not republish the token, so the attestation revokes (TF-4.5). |

## 12. Open questions

1. **Who will keep the list.** Candidates are a standards body hosting the framework, or a consortium of the trade associations whose members are verified. No approach has been made.
2. **The audit standard in detail.** Annex A lists the criteria; the depth of testing, sample sizes and the auditor's qualifications need settling with a list keeper.
3. **Log operation.** Whether each V&V runs its own log, or V&Vs share logs run by others as browsers' certificate logs are, and how many logs a consumer should require.
4. **Self-claims on web pages.** §8 covers cards, declarations and ARD entries. Whether a business's own pages may say "verified" is not covered.
5. **Recognition by ARD.** Whether ARD registries will verify this framework, or ARD later pins a default scheme (ARD §4.5.2), is open.
6. **Ranking in registries generally** (added in 0.2). TF-3.6 and TF-6.3a deal with how registries weigh attestations. ARD v0.91 leaves ranking to each registry and says nothing on fairness, paid placement or self-preference. Whether registries should publish the signals they rank on and label paid placement is a question for ARD, to be raised there.
7. **A disclosure field on the trust list** (added in 0.2). Whether TF-7.7 should add a field recording the registry or search services a V&V's operator runs, so that consumers can read the TF-7.4 disclosure from the list. Not proposed yet.

---

## Annex A — Audit criteria

The audit tests, over the period covered, that the V&V:

1. Issues level-1 attestations only after TF-4.1 to TF-4.4 were met, and keeps evidence that a third party can use to repeat each check.
2. Never accepted a host-level record (TF-4.2), or HTTP alone for a delegated host (TF-4.3).
3. Re-checked every live attestation at least every 30 days, and revoked only under TF-4.5.
4. Named card keys only where TF-4.8 was met.
5. Issued level-2 attestations only on evidence checked at source (TF-4.11), with the domain link established under TF-4.12, and recorded the fields of TF-5.5.
6. Kept to the expiry limits of TF-5.7 and revoked on every ground of TF-5.9.
7. Logged every attestation and revocation (TF-7.10), with the log meeting TF-7.11 and TF-7.12, and checked that the log contains everything issued.
8. Kept its signing keys on its own domain (TF-5.2), protected them, and has a documented procedure for key compromise.
9. Offered identical terms to all operators (TF-3.2), and disclosed its relationships with implementers and any registry or search service its operator runs (TF-7.4).
10. ~~Kept no index, aggregated no authority, and held no conversation content (TF-3.5).~~ *Superseded in 0.2:* Kept no index in its verification service, aggregated no authority, and held no conversation content (TF-3.5). Where its operator runs a registry or search service: that service ran separately from the V&V (TF-3.6(b)), and weighed attestations from every certified V&V the same in filtering and ranking, with no preference for the V&V's own attestations or customers (TF-3.6(a)), tested on a sample of the registry's results.
11. Asserted nothing beyond its level (TF-5.4).
12. Investigated complaints, including attestations an operator says it never requested, and recorded the outcome.
13. **Added in 0.2.** Offered any supplementary services on identical terms, never as a condition of verification, and with no effect on verification for operators who declined (TF-3.8); submitted agents to registries only at the operator's request (TF-3.9).

---

## Annex B — Examples

All examples use reserved domains (RFC 2606) and a fictional business. Signature and digest values are placeholders. They are unchanged from 0.1: the field names did not change.

### B.1 DNS records the operator publishes

```
_agent-capability.example.com.  IN TXT "agent-capability-verification=kT3vQ9xY2mW8bR4sL1nF6hJ0cA5dE7gU3pZ9oI2uY8w"
_agent-capability.example.com.  IN TXT "agent-capability-key=NzbLsXh8uDCcd-6MNwXF4W_7noWXFZAfHkxZsRGC9Xs"
```

### B.2 A level-2 attestation payload

```json
{
  "framework": "https://agent-capability.org/trust/v1",
  "id": "att-7f3c2b",
  "verifier": "verifier.example.org",
  "level": 2,
  "domain": "example.com",
  "agentCard": "https://agent.example.com/.well-known/agent-card.json",
  "cardKeys": [
    {"kid": "chapel-1", "thumbprint": "NzbLsXh8uDCcd-6MNwXF4W_7noWXFZAfHkxZsRGC9Xs"}
  ],
  "method": ["agent-capability-dns-01", "agent-capability-legal-entity"],
  "checked": {
    "label": "_agent-capability.example.com",
    "dnssec": false
  },
  "entity": {
    "legalName": "Example Memorial Chapel LLC",
    "tradingNames": ["Example Memorial Chapel"],
    "jurisdiction": "US-OH",
    "source": "State Board of Embalmers and Funeral Directors",
    "recordId": "FH-000000",
    "domainLink": "evidence-holder-confirmed",
    "checked": "2026-11-01"
  },
  "issued": "2026-11-01T15:00:00Z",
  "expires": "2027-01-30T15:00:00Z",
  "log": "https://log.verifier.example.org/"
}
```

The level-2 attestation expires with the level-1 attestation it rests on (90 days), not at 12 months (TF-5.7).

### B.3 The ARD entry

```json
{
  "identifier": "urn:air:example.com:assistant:main",
  "displayName": "Example Memorial Chapel assistant",
  "type": "application/a2a-agent-card+json",
  "url": "https://agent.example.com/.well-known/agent-card.json",
  "description": "Public information about Example Memorial Chapel; refers people for anything given on request.",
  "trustManifest": {
    "identity": "https://example.com",
    "identityType": "https",
    "trustSchema": {
      "identifier": "https://agent-capability.org/trust/v1",
      "version": "1.0",
      "governanceUri": "https://agent-capability.org/trust/v1",
      "verificationMethods": ["agent-capability-dns-01", "agent-capability-legal-entity"]
    },
    "attestations": [
      {"type": "agent-capability:domain-control", "uri": "https://verifier.example.org/attestations/att-7f3c2a", "mediaType": "application/jose", "digest": "sha256:0000000000000000000000000000000000000000000000000000000000000000"},
      {"type": "agent-capability:legal-entity", "uri": "https://verifier.example.org/attestations/att-7f3c2b", "mediaType": "application/jose", "digest": "sha256:1111111111111111111111111111111111111111111111111111111111111111"}
    ]
  }
}
```

### B.4 A trust list entry

```json
{
  "name": "Example Verifier",
  "domain": "verifier.example.org",
  "levels": [1, 2],
  "keys": "https://verifier.example.org/.well-known/jwks.json",
  "log": "https://log.verifier.example.org/",
  "audit": {"date": "2026-10-15", "report": "https://verifier.example.org/audit/2026.pdf", "nextDue": "2027-10-15"},
  "status": "listed"
}
```

### B.5 What a consumer does, in order

1. Find the Agent Card (A2A's well-known path, or ARD).
2. Find the attestation: from the ARD entry, or by asking the V&V's log for the domain.
3. Check the V&V is on the trust list (stage 2), or is one the consumer accepts (stage 1).
4. Check the attestation's signature with the V&V's key, its status, its expiry, and that the log contains it.
5. Check the attestation names this card, and that the card's `provider.url` is on the attested domain.
6. Where the card is signed, check the signing key's thumbprint is in the attestation, then check the card's signature.
7. For level 2, check `provider.organization` matches a name in `entity`.
8. Ignore anything the card's own text says about being verified.

---

## Annex C — Changes from 0.9 Part 3 and the Verification Mechanism Specification 0.5.7

| # | Change | Why |
|---|---|---|
| 1 | §7: only certified V&Vs count; trust list kept by a neutral list keeper; public attestation log; two-stage start | Only a certified V&V can make a business "verified" |
| 2 | §8: no verification claims in card, declaration or ARD free text; consumers disregard them; validators report them | A self-claim is textually identical to a true one |
| 3 | TF-4.7 to TF-4.9: the card-signing key is bound by its thumbprint in the operator's DNS | 0.9 P3.1b.4 named a key in the attestation but did not say how the verifier knows the key is the operator's |
| 4 | §5.1: the attestation's form is fixed as a JWS with a defined payload | 0.9 listed the contents; the mechanism spec defined an unsigned JSON record at a status endpoint |
| 5 | Terms: "tier standard/elevated" and "registry" replaced by "level 1/2" and "verifier" throughout (and "verifier" by "V&V" in 0.2, Annex D) | 0.9 renamed them; the mechanism spec predates it |
| 6 | TF-5.7: level-1 expiry of 90 days carried from the mechanism spec §5.3; level-2 expiry from 0.9 P3.1c.5 | Brought together in one place |
| 7 | TF-7.4: V&Vs disclose relationships with implementers | Annex B 5 of 0.9: a verifier with a commercial stake |
| 8 | Draft 0.2's changes (Annex D) | Not in 0.9 or the mechanism spec |

To be carried back into specification 0.9 Part 3 and into the Verifier & Validator specification revision (0.6).

---

## Annex D — Changes from draft 0.1

| # | Change | Where | Why |
|---|---|---|---|
| 1 | **"Verifier & Validator" (V&V)** replaces "verifier" for the role; "certified V&V" replaces "certified verifier". Field names on the wire (`verifier`, `verifiers`) are unchanged | Throughout; §2 | A name that says what the role does: it verifies, and it may validate |
| 2 | **What V&V means:** a provider that verifies; certification applies to verification; validation on its own needs no certification, and a validation-only service is a validator, not a V&V | §2; TF-7.1; TF-9.5 | Validation stays open to anyone, with no certification |
| 3 | **TF-3.5 amended:** the bar on an index now applies to the verification service itself; the old text kept, marked superseded | §3.1 | ARD registries are run by many organisations. Barring all of them from also running a V&V removed likely V&Vs without making verification fairer |
| 4 | **TF-3.6 added:** a V&V operator may also run ARD registry or search services, with no discrimination in weightings, separate services, disclosure, an audit for self-preference, and no conversation content | §3.2; TF-7.4 | What matters is that a registry run by a V&V operator does not prefer its own attestations or customers |
| 5 | **Supplementary services (TF-3.7 to TF-3.9):** the V&V's choice, provided they do not contravene the framework; identical terms; never a condition of verification, declining has no effect; registry submission only at the operator's request; who submitted carries no weight | §3.3 | A V&V may help its customers, for example with listing in ARD registries, without that help becoming part of verification |
| 6 | **TF-6.3a added:** any registry relying on the framework weighs attestations from every certified V&V the same | §6.1 | So that a registry run by a V&V operator is not held to a stricter rule than any other registry |
| 7 | **Annex A:** item 10 amended (old text kept, marked superseded); item 9 extended to the registry disclosure; item 13 added for supplementary services | Annex A | Follows from items 3 to 5 |
| 8 | **Threats table:** three rows added; the gatekeeper row updated | §11 | Follows from items 3 to 5 |
| 9 | Open questions 6 and 7 added, neither decided | §12 | Raised for discussion |
| 10 | Schema: description updated to 0.2 and to the V&V term; no change to any definition | `schema.json` | Follows from item 1 |

Unchanged: the framework identifier, the two levels, every method, the attestation's form and fields, the trust list's fields, and every requirement of 0.1 not listed above.

---

## Sources

- A2A Protocol Specification v1.0.1, §8.4 (Agent Card signing), github.com/a2aproject/A2A (fetched 1 Nov 2026).
- Agentic Resource Discovery v0.91, `spec/ard.md` §4.5, §5.1 to §5.4 and the entry schema's `TrustManifest`, github.com/ards-project/ard-spec (fetched 1 Nov 2026; ard.md md5 5e1bb1f0b68d2ebcb34c37b880c41b13, unchanged since 30 Oct).
- Agent Capability Declaration Specification 0.9 draft, Part 3, §4, §5, Annex B 5 and 19.
- Verification Mechanism Specification 0.5.7, Parts 2 to 5.
- RFC 7515 (JWS), RFC 7638 (JWK thumbprint), RFC 9162 (Certificate Transparency 2.0).

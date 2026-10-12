---
title: Conformance checklist
description: Every requirement of draft 0.1 by conformance class and assurance level, with the section that states it, for implementers to assess themselves against.
order: 19
status: informative
---

# Conformance checklist

This checklist restates the requirements of draft 0.1 so that an implementer can assess a claim ([section 13](conformance.md)). The sections it cites are normative; where this list and a section differ, the section wins. Requirements marked SHOULD are recommendations; every other item is a MUST.

Use the items for the classes you claim, at the level you claim and every level below it. Record the evidence for each item in your conformance statement.

## Producer

| # | Requirement | Section |
| --- | --- | --- |
| P-1 | Every object it emits is valid on its own: schema, profile and object-level rules. | [8.2](envelope-and-errors.md) |
| P-2 | Identifiers carry their kind's prefix, are not reused for another object of the same kind, or for another approval, within the organization, and are unique within their list. | [2.3](conventions.md) |
| P-3 | An issued revision never changes; a change issues the next revision, with `revision_author` and `change_summary` from revision 2. | [2.5](conventions.md) |
| P-4 | Timestamps are UTC with `Z`; durations are integer seconds. | [2.7](conventions.md) |
| P-5 | Amounts are decimal strings with current ISO 4217 codes, and are compared and added exactly. | [2.8](conventions.md) |
| P-6 | No member is `null`; nothing whose result is unknown is reported as `failed` or `fail`. | [2.9](conventions.md) |
| P-7 | Action names, extension names and profile identifiers it defines are prefixed with a domain it controls, and so are the artifact and evidence kinds it defines for objects that follow the software-delivery profile. | [2.2](conventions.md), [14.3](software-delivery-profile.md) |
| P-8 | Extension data is only under `extensions`, and no extension relaxes a requirement of the draft. | [2.12](conventions.md) |
| P-9 | No token, key, password or other secret appears in any member; credentials are referenced in `credential_refs`. | [5.7](authority-binding.md), [16.3](security.md) |
| P-10 | Context is referenced, never copied, and `source.ref` is never a `data:` URI. | [4.2](context-manifest.md) |
| P-11 | Each criterion can be decided by its method. SHOULD: at least one criterion is required. | [3.2](work-contract.md) |
| P-12 | Receipt digests are computed as specified, and the receipts of each contract revision form one chain. | [6.6](execution-receipt.md) |
| P-13 | A receipt's summary describes every change, partial ones included, its outcome is `unknown` until known, and its times run forward: requested, decided, started, finished. | [6.3](execution-receipt.md) |
| P-14 | A receipt with attestation `none` is never presented as signed. SHOULD: it is labeled unsigned wherever it is shown. | [6.6](execution-receipt.md) |
| P-15 | SHOULD: every object of a contract that follows a profile names that profile. | [14.1](software-delivery-profile.md) |
| P-16 | SHOULD: identifiers and summaries carry no personal data the reader does not need. | [2.3](conventions.md), [16.4](security.md) |

## Consumer

| # | Requirement | Section |
| --- | --- | --- |
| C-1 | Decoding is strict: malformed UTF-8 JSON, a byte order mark, and a lone surrogate in a string or a member name are `invalid_json`, and the first repeated member name in text order is `duplicate_member`. | [2.1](conventions.md) |
| C-2 | An unsupported `protocol_version` is refused before anything else. | [2.14](conventions.md) |
| C-3 | Unknown members are refused, at any depth, outside extension values and PAP `terms`. | [2.13](conventions.md) |
| C-4 | Unknown extensions that are not critical are ignored; unsupported critical extensions are refused, on objects, envelopes and discovery documents alike; every extension of an authority binding counts as critical. | [2.12](conventions.md) |
| C-5 | A known profile's rules are applied on top of the core; an unknown profile leaves the core rules in force. | [2.14](conventions.md) |
| C-6 | Documents are validated in the order of section 8.2; envelopes are open-world. | [8.1](envelope-and-errors.md), [8.2](envelope-and-errors.md) |
| C-7 | `unknown` never satisfies a requirement. | [2.9](conventions.md) |
| C-8 | No digest, attestation or conformance claim is treated as proof that an action was authorized. | [6.6](execution-receipt.md), [13.4](conformance.md) |
| C-9 | Error responses use catalog codes and RFC 6901 string paths, and messages are never parsed. SHOULD: details are reported once each and sorted by path, then code. | [8.3](envelope-and-errors.md) |
| C-10 | A record is verified with every rule of section 9. | [9.1](invariants.md) |
| C-11 | Every valid conformance fixture is accepted, and every invalid one refused with at least its expected (code, path) pairs. SHOULD: exactly those pairs. | [13.1](conformance.md) |
| C-12 | Schemas are never fetched over the network to validate a document. | [2.14](conventions.md) |
| C-13 | A held revision is never replaced by different content under the same `id` and `revision`. | [2.5](conventions.md) |
| C-14 | A `traceparent` whose trace or parent identifier is all zeros is ignored. | [6.5](execution-receipt.md) |
| C-15 | A discovery document is checked for the requested host, and, when relied on, against its issuer's `enna_domains` and its `poppy.json` link. SHOULD: fetches are bounded and never reach loopback, private or link-local addresses. | [12.2](discovery.md) to [12.5](discovery.md) |
| C-16 | Formats are assertions, checked with the draft's rules (sections 2.7 and 2.14), never with a validator's own, and patterns match as ECMA-262 matches them, `$` only at the very end of a value. | [2.14](conventions.md) |

## Enforcement point

| # | Requirement | Section |
| --- | --- | --- |
| E-1 | Before every action, the gate looks up the idempotency key first (a replay skips the checks; a conflict is refused with `idempotency_conflict` alone), refuses every action while the contract is neither `authorized` nor `executing` when it knows the state, refuses every action over a record that fails its shape, chain, time, level or profile checks, wherever the failure lies, then checks that the agent asking is the binding's grantee, the window, revocation (of the binding and of every binding it was delegated from), grant, approvals and their quorum, tenant boundary (the context included), context freshness and limits (the contract's counting what every revision spent), and refuses before any side effect, with a `deny` receipt that gives every failing check's code. | [11.1](execution-semantics.md) |
| E-2 | Decisions rest on the checks of section 11.1 alone; no content grants authority. | [16.1](security.md), [11.1](execution-semantics.md) |
| E-3 | It computes payload digests itself; replays the same key for the same action, resource and payload; refuses the same key with any of the three different; and applies this across all revisions of a contract. | [11.2](execution-semantics.md) |
| E-4 | An outcome it cannot establish is recorded as `unknown`, the action is not repeated until the outcome is known, and the known outcome is recorded once. | [11.3](execution-semantics.md) |
| E-5 | The lifecycle follows the transition table: nothing is cancelled, revised or completed while an outcome is unknown, `resume` is never used while a block's cause holds, and approval blocks are left only by reapproval or revision. | [10.3](lifecycle.md) |
| E-6 | Contracts are blocked only for the business reasons of section 10.4; transport failures never block. | [10.4](lifecycle.md) |
| E-7 | Authority derived from a PAP session is never broader than its scopes; authority from a confirmed operation survives sign-out only for that operation. | [15.1](relationships-and-mappings.md) |
| E-8 | MCP's audience validation and its ban on token passthrough are kept, and PAP's DPoP and MCP's Bearer schemes are never interchanged. | [16.5](security.md) |
| E-9 | A missing approval is never reported as `TASK_STATE_AUTH_REQUIRED`. | [10.6](lifecycle.md) |
| E-10 | After an HTTP 429, it waits at least `Retry-After`. | [11.4](execution-semantics.md) |
| E-11 | SHOULD: idempotency keys are passed to downstream systems that deduplicate, and outcomes are reconciled before any retry. | [11.12](execution-semantics.md) |
| E-12 | Where it reports work to PAP, it sets `trace.transport.pap_operation_id` on the receipts of the operation's actions and on no others, and the operation's state and result come from those receipts' final receipts: a denial that a later outcome under the same key superseded is not a failure, and of a chain of retries linked by `retry_of` only the last attempt decides. | [10.7](lifecycle.md), [15.1](relationships-and-mappings.md) |

## Evaluator

| # | Requirement | Section |
| --- | --- | --- |
| V-1 | `technical_result` is derived from the required criteria, never asserted. | [7.4](evaluation-result.md) |
| V-2 | Every `pass` and `fail` carries evidence. | [7.2](evaluation-result.md) |
| V-3 | Every required criterion is evaluated, and no criterion outside the contract is named. | [9.12](invariants.md) |
| V-4 | Execution is not reported `complete` or `partial` while an outcome it relies on is unknown. | [7.3](evaluation-result.md) |
| V-5 | Acceptance is recorded only in a later revision, after every effect of the record has ended and no action's final outcome is unknown, cited or not, from the owner, an acceptor the contract names, or the owner's delegate. | [7.5](evaluation-result.md), [9.13](invariants.md) |
| V-6 | Business impact is kept apart and stays `unknown` until measured. | [7.6](evaluation-result.md) |
| V-7 | Each criterion's method is recorded. SHOULD: it is the method the contract states. | [7.2](evaluation-result.md) |
| V-8 | SHOULD: an evaluation cites each action's final receipt, the one that settled an earlier unknown outcome where there is one. | [7.1](evaluation-result.md) |

## Publisher of a discovery document

| # | Requirement | Section |
| --- | --- | --- |
| D-1 | The document is served over HTTPS at `/.well-known/enna.json`, and `organization.domain` matches the host. | [12.2](discovery.md) |
| D-2 | It names no private system and carries no personal data or secrets. | [12.2](discovery.md) |
| D-3 | `auth` is present when agents are listed, and the issuer's metadata lists the domain in `enna_domains`. | [12.4](discovery.md) |
| D-4 | When `poppy.json` is also published, the two documents link to each other. | [12.5](discovery.md) |
| D-5 | Conformance is self-declared, with a statement, and never described as certified. | [12.6](discovery.md), [13.4](conformance.md) |

## Assurance levels

### Essential

Every class requirement above applies. Essential allows the owner to approve their own contract within its limits, enforcement against the runtime's own state, and an idempotency store that lives in memory, which then demonstrates the semantics without guaranteeing them across restarts ([13.2](conformance.md)).

### Controlled

| # | Requirement | Section |
| --- | --- | --- |
| LC-1 | Separation of duties applies above the threshold the conformance statement states (or to every grant), lowered where a binding's `policy.sod_threshold` is lower, and holds when each effect starts. | [5.7](authority-binding.md), [9.6](invariants.md), [13.2](conformance.md) |
| LC-2 | Every approval carries `expires_at`; a record check reports one without it. | [9.16](invariants.md), [13.2](conformance.md) |
| LC-3 | The live revocation status at `revocation.status_ref` is consulted before every effect. SHOULD: any cache of it is short-lived, and its lifetime is stated. | [11.1](execution-semantics.md), [16.2](security.md) |
| LC-4 | Tenant and workspace boundaries are enforced before every effect. | [9.4](invariants.md) |
| LC-5 | Cost limits are enforced with exact decimal arithmetic. | [2.8](conventions.md) |
| LC-6 | Idempotency keys are stored durably and reserved atomically, and crash recovery is documented. | [11.12](execution-semantics.md) |
| LC-7 | SHOULD: required automated criteria are evaluated by a principal other than the executor. When they are not, the criterion says so in its `limitations` (a MUST, which a record check verifies). | [7.7](evaluation-result.md), [9.16](invariants.md) |

### Assured

| # | Requirement | Section |
| --- | --- | --- |
| LA-1 | Policy decisions come from an external decision point through an AuthZEN-shaped interface, and receipts identify them. Not testable until section 15.6 is normative, except that every receipt names its decision. | [13.2](conformance.md), [15.6](relationships-and-mappings.md) |
| LA-2 | Each delegation hop is backed by an RFC 8693 token exchange, recorded in `credential_refs`, and every binding in a chain is checked, revocation included. | [5.6](authority-binding.md), [9.16](invariants.md), [13.2](conformance.md) |
| LA-3 | Where policy requires several approvers, the grants state the quorum in `conditions.min_approvals`, and every effect is held to it with `verified` approvals from that many distinct people. | [5.4](authority-binding.md), [13.2](conformance.md) |
| LA-4 | Approvals relied on are `verified`; a PAP confirmation alone authorizes nothing. | [13.2](conformance.md) |
| LA-5 | Receipts are attested, with `jws` or `in_toto`. | [6.6](execution-receipt.md), [9.16](invariants.md) |
| LA-6 | No agent gets context whose classification and residency the serving systems do not enforce. | [4.5](context-manifest.md), [13.2](conformance.md) |
| LA-7 | Records are exported for audit, to OpenTelemetry and in W3C PROV terms, with retention and legal hold. Not testable until section 15.8 is normative. | [13.2](conformance.md), [15.8](relationships-and-mappings.md) |

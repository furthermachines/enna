---
title: Cross-object invariants
description: The rules a complete record of one contract revision must satisfy beyond each object's shape, with the exact checks, error codes and paths, and what they leave unchecked.
order: 9
status: normative
---

# Cross-object invariants

Each object can be valid on its own and the work can still be wrong: an approval for another revision, an action in another client's workspace, a cost over the limit, an acceptance before the work finished. This section defines thirteen invariants over a complete record, plus one rule about time. They are what turns five valid objects into evidence that can be checked.

## 9.1 The record and who checks it

A record is the complete set of objects for one revision of a work contract: exactly one revision of the contract, and any number of context manifests, authority bindings, execution receipts and evaluation results that serve it. A record may hold several revisions of one manifest, binding or evaluation. It never holds the same object twice, where the same object means the same `type`, `id` and `revision`.

A record is checked as a closed world: it is assumed complete, so a reference that does not resolve inside it is an error. An envelope, by contrast, assumes an open world ([section 8.1](envelope-and-errors.md)).

Checking a record takes these inputs besides the objects:

| Input | Meaning |
| --- | --- |
| The current time | Used only to reject records that describe the future ([9.15](invariants.md)). |
| The assurance level | `essential`, `controlled` or `assured` ([section 13](conformance.md)); `essential` when not given. Separation of duties ([9.6](invariants.md)) and the rules of [9.16](invariants.md) apply at `controlled` and `assured`. |
| The separation-of-duties threshold | Money, optional: the checking party's own. A binding's `policy.sod_threshold` applies instead only where it is lower ([section 5.7](authority-binding.md)). Without a threshold, every grant needs an approver other than the requester. |
| The live revocation status | The revoked binding and approval identifiers, each with the time it was revoked, fetched where each binding's `revocation.status_ref` points. |
| The supported extensions | Extensions the checking party supports, beyond the core `pap` extension. |

A check proceeds in this order:

1. The record, one JSON document, passes decoding ([section 2.1](conventions.md)) and is an array; otherwise the decoding error, or `schema_violation` at `""`.
2. Every object is valid on its own ([section 8.2](envelope-and-errors.md)), with its errors reported under `/<i>`, its index in the record. If any object is invalid, the check stops there.
3. The record holds exactly one work contract: none is a `schema_violation` at `""`, and every contract after the first is a `schema_violation` at `/<i>/type`. If not exactly one, the check stops there.
4. No object appears twice; a repetition is a `schema_violation` at `/<i>/id`, and the check stops there.
5. The invariants of [9.2](invariants.md) to [9.15](invariants.md), and the rules of the level ([9.16](invariants.md)), are checked, and all of their errors are reported together. Each invariant applies on its own: a stop inside an invariant, such as "nothing else is checked for this effect" or "the approvals are not examined", ends that invariant's checks for that object or effect only, and the other invariants still apply.

A check that stopped at steps 1 to 4 has not checked any invariant, so its errors say nothing about the invariants; the reference implementation marks such a result (`stopped: "shape"`). An Enforcement point MUST refuse every action over a record that fails steps 1 to 4, whichever object fails them.

The reference implementation treats a current time that is not a UTC timestamp, or a threshold that is not valid money, as a mistake by the caller: it throws instead of reporting an error about the record.

Time-dependent invariants are checked at each action's own start time, never at the current time. A record of past work stays valid when it is checked again later.

**Who checks.** A Consumer that verifies a record MUST apply every rule of this section. An Enforcement point MUST NOT start an effect that would make its record break [9.4](invariants.md) to [9.11](invariants.md), or the rules [9.16](invariants.md) adds at its level: it applies those rules before each action, as [section 11.1](execution-semantics.md) describes, rather than discovering the breach afterwards. It MUST refuse every action over a record that already breaks [9.15](invariants.md), or a rule [9.16](invariants.md) adds at its level, wherever in the record the break lies, as it does over a record that fails steps 1 to 4; the one exception is the rule that only `verified` approvals authorize an effect, which bears on the action it would authorize. An Evaluator MUST NOT issue an evaluation that breaks [9.12](invariants.md) or [9.13](invariants.md), and a Producer of receipts MUST NOT issue a chain that breaks [9.14](invariants.md).

An effect, in this section, is a receipt whose decision is `permit` and whose outcome is not `replayed`.

## 9.2 Invariant 1: references resolve

- The `contract` of every manifest, binding, receipt and evaluation names the record's contract. Another `id` is `reference_unresolved` at `/<i>/contract`.
- A receipt's `authority` and `context`, and a binding's `delegation.parent`, resolve to an object in the record with that `id` and `revision`. If the `id` is present only at another revision, it is `revision_mismatch`; if it is absent, it is `reference_unresolved`. Either is reported at the reference.
- Every receipt an evaluation cites is in the record (`reference_unresolved` at `/<i>/receipts/<j>`) and has the cited digest (`digest_mismatch` at the same path).

## 9.3 Invariant 2: revisions match

- The `contract` of every manifest, binding, receipt and evaluation names the record's contract revision. Another revision is `revision_mismatch` at `/<i>/contract`.
- The binding and the manifest a receipt cites serve the same contract revision as the receipt. Otherwise it is `revision_mismatch` at `/<i>/authority` or `/<i>/context`.

## 9.4 Invariant 3: tenant boundaries hold

The tenant boundary is the organization plus the workspace ([section 2.10](conventions.md)). The record's allowed scopes are the contract's scope and the resource scope of every grant, in any binding of the record, that names the contract's organization and has approved, unrevoked authority behind it: its binding holds approvals that are current, approve, are in scope and count for that grant, by approver and by role ([section 5.4](authority-binding.md)), that are not PAP confirmations, and that were not revoked, nor was the binding or any binding it was delegated from, when they were decided; and those approvals come from at least as many distinct people as the grant's `conditions.min_approvals`, or from one when it states none. A later revocation stops new effects, but does not shrink the boundary of work already recorded. Each allowed scope is reduced to its organization and workspace; projects do not count. A scope is inside the boundary when it lies within at least one allowed scope.

- The `scope` of every manifest, binding, receipt and evaluation is inside the boundary; otherwise `tenant_mismatch` at `/<i>/scope`.
- Every grant's resource scope names the contract's organization; otherwise `tenant_mismatch` at `/<i>/grants/<k>/resource/scope`. A grant can never reach another organization.
- An effect uses only context its own authority reaches: every item of the manifest an effect cites lies within the contract's scope or the resource scope of a grant of the effect's own binding that has approved authority behind it, as above, each reduced to its organization and workspace; otherwise `context_out_of_scope` at the effect's `/<i>/context`. Another agent's grant does not let this effect read a workspace, and neither does a grant of its own binding that lacks the approvals it needs. An effect whose binding is not in the record reaches the contract's scope alone.
- Every item of a manifest that no effect cites is inside the boundary; otherwise `context_out_of_scope` at `/<i>/items/<k>/scope`.
- The resource scope of every receipt whose decision is `permit` is inside the boundary; otherwise `tenant_mismatch` at `/<i>/action/resource/scope`.

A receipt whose decision is `deny` may name any scope. An attempt on another tenant that was refused is a valid record, and a useful one.

A grant that names another workspace of the same organization extends the boundary to that workspace, and a grant that names no workspace extends it to the whole organization, once approved authority stands behind it. Every such extension is therefore visible in a binding, next to the approvals that make it count, and a binding nobody approved extends nothing.

## 9.5 Invariant 4: approvals bind the revision and the binding

For every approval in every binding, whether it approves or denies:

- `covers.contract` names the record's contract `id` and revision; otherwise `approval_stale` at `/<i>/approvals/<k>/covers/contract`.
- `covers.binding_digest` equals that binding's digest ([section 5.3](authority-binding.md)); otherwise `approval_stale` at `/<i>/approvals/<k>/covers/binding_digest`.

An approval that passes both checks is current.

## 9.6 Invariant 5: approvers are in scope

- Every approval's approver is in scope for the contract: its owner, a stakeholder whose role is `approver`, or, for an approval whose `method` is `pap_confirmation`, its requester. Otherwise `approver_out_of_scope` at `/<i>/approvals/<k>/approver`. A PAP confirmation counts only for grants whose resource scope lies within the contract's own scope ([section 5.4](authority-binding.md)).
- **Separation of duties**, at the `controlled` and `assured` levels only. For each binding, take its approvals that are `approved`, current and in scope. If there are none, or if any of them comes from someone other than the contract's requester, the binding passes. Otherwise every one of them is the requester's own, and each grant of the binding that is over the threshold is `separation_of_duties` at `/<i>/grants/<k>`. The threshold is the one given to the check, or the binding's `policy.sod_threshold` where that is lower and in the same currency (another currency is `currency_mismatch` at `/<i>/policy/sod_threshold`, and the given threshold applies). A grant is over the threshold when its cost limit is greater than the threshold, or it has no cost limit, or there is no threshold. A cost limit equal to the threshold is not over. A cost limit in another currency than the threshold is `currency_mismatch` at `/<i>/grants/<k>/limits/cost` and is not counted as over.
- Separation of duties also holds when each effect starts ([9.7](invariants.md), step 4.2): once a binding holds a current approval in scope from someone other than the requester, the requester's own approvals no longer authorize effects under its grants over the threshold, so they cannot stand in when that approval expires or is revoked.
- At the `essential` level the owner may approve their own contract within its limits, even when the owner is also the requester.


## 9.7 Invariant 6: no effect without authority

Each effect is checked at its `started_at`, written `t` below, against the binding its `authority` names. (If the binding is not in the record, [9.2](invariants.md) has already reported it.)

0. The receipt's `actor.principal` is the binding's `grantee`, compared as in [section 2.11](conventions.md); otherwise `authority_insufficient` at `/<i>/actor/principal`, and nothing else is checked for this effect. Authority belongs to the agent it was granted to, and a person who operates that agent acts as it (`performed_by` `human`).
1. `t` is inside the binding's window, `valid_from <= t < expires_at`; otherwise `authority_expired` at `/<i>/started_at`.
2. The live status shows neither the binding nor any binding it was delegated from revoked at or before `t`, following `delegation.parent` through the record; otherwise `authority_revoked` at `/<i>/authority`. Revoking a binding revokes every binding delegated from it ([section 5.6](authority-binding.md)).
3. A grant of the binding covers the action ([section 5.2](authority-binding.md)); otherwise `authority_insufficient` at `/<i>/action`, and the approvals are not examined.
4. Approvals, with every result reported at `/<i>/authority`:
   1. If a denial that is current, in scope, decided at or before `t`, unexpired at `t` and not revoked at `t` exists, the result is `approval_denied`.
   2. Otherwise, take the approvals that approve and are in scope for this grant, by approver, by role (a role the grant requires counts only when the contract gives it to the approver, [section 5.4](authority-binding.md)) and, for a PAP confirmation, by scope (the grant's resource scope lies within the contract's scope) (excluding `standing` ones for `production.deploy` when the contract, the binding or the receipt follows the software-delivery profile). Where separation of duties covers the grant ([9.6](invariants.md)) and the binding holds a current approval in scope from someone other than the requester, leave out the requester's own: separation of duties holds when each effect starts. Of the approvals left, the valid ones are current, decided at or before `t`, unexpired at `t` and not revoked at `t`. If the valid approvals come from at least the grant's `conditions.min_approvals` distinct people, or from at least one when the grant states none, the effect passes ([section 5.4](authority-binding.md)).
   3. Otherwise, if there are valid approvals, from fewer people, the result is `quorum_not_met`.
   4. Otherwise, if the binding holds approvals that approve but none of them is in scope for this grant, because of who gave it or the role it was given under, the result is `approver_out_of_scope`. An approval that step 2 leaves out by its kind, a PAP confirmation for a grant whose resource scope lies outside the contract's scope or a standing approval for a production deploy, is not out of scope by who gave it, so step 5 applies.
   5. Otherwise, take the most recently decided of the approvals left in step 2. If there is none, or it was decided after `t`, the result is `approval_missing`. If it is not current, `approval_stale`. If it had expired at `t`, `approval_expired`. Otherwise it was revoked, and the result is `approval_revoked`.

A replay is not an effect: it needs no live authority, because nothing happens again ([section 11.2](execution-semantics.md)).

## 9.8 Invariant 7: context is fresh when used

For each effect that cites a context manifest, every item of that manifest is fresh at `t` ([section 4.3](context-manifest.md)). An item that is stale, or of unknown age, makes the effect `context_stale` at `/<i>/context`. An effect that cites no manifest is not checked for freshness.

## 9.9 Invariant 8: money is consistent

All amounts are compared exactly ([section 2.8](conventions.md)). Where two amounts are in different currencies, the result is `currency_mismatch` at the path given and the amount is left out of that comparison or sum.

- If the contract states `limits.cost`, no grant's cost limit is greater than it; otherwise `limit_exceeded` at `/<i>/grants/<k>/limits/cost`.
- If the contract states `limits.cost`, the costs of the record's receipts, replays excluded, are added in creation order (by `created_at`, ties in record order). At the first receipt where the total passes the limit, the result is `limit_exceeded` at `/<i>/cost`, reported once.
- For each grant, among the effects it covers, in creation order:
  - when the number of distinct idempotency keys passes the grant's `count`, the result is `limit_exceeded` at `/<i>/action`, at each receipt from the one that passes it;
  - when the total cost passes the grant's cost limit, the result is `limit_exceeded` at `/<i>/cost`, at each receipt from the one that passes it.

A denied attempt uses none of a grant's count. Replays cost nothing new.

A record holds one revision, so this check adds up that revision's costs only. A new revision does not reset the contract's budget: the Enforcement point counts what earlier revisions spent against the limit before each action ([section 11.1](execution-semantics.md)), which the record check cannot.

## 9.10 Invariant 9: delegation attenuates

For each binding whose `delegation` names a parent present in the record:

- If the parent has no `delegation` member, or its `subdelegation` is `forbidden`: `subdelegation_forbidden` at `/<i>/delegation/parent`, and nothing else is checked for this binding.
- If the binding's `depth` is not the parent's `depth` plus one: `attenuation_violated` at `/<i>/delegation/depth`.
- If the binding's `depth` is greater than the parent's `max_depth`: `subdelegation_forbidden` at `/<i>/delegation/depth`.
- If the binding's `max_depth` is greater than the parent's: `attenuation_violated` at `/<i>/delegation/max_depth`.
- If the binding's `expires_at` is later than the parent's: `attenuation_violated` at `/<i>/expires_at`. If its `valid_from` is earlier than the parent's: `attenuation_violated` at `/<i>/valid_from`.
- If the parent declares `policy.sod_threshold` and the binding declares none, a higher one, or one in another currency: `attenuation_violated` at `/<i>/policy/sod_threshold`.
- Each grant that narrows no grant of the parent ([section 5.6](authority-binding.md)) is `attenuation_violated` at `/<i>/grants/<k>`.

## 9.11 Invariant 10: idempotency holds

Take the receipts whose decision is `permit`, in creation order, grouped by `action.idempotency_key`:

- Every receipt with a key carries the action name, resource and payload digest of the first receipt with that key. Otherwise the receipt is one `idempotency_conflict`, reported at the first member that differs, in the order name, resource, payload digest: at `/<i>/action/name` when the name differs, else at `/<i>/action/resource` when the resource differs, else at `/<i>/action/payload_digest`.
- While a key's last outcome is `unknown`, the only receipt that may follow under it is the one that records the known outcome (`succeeded`, `failed` or `cancelled`); that reconciles it, and is not a second effect ([section 11.3](execution-semantics.md)). Any other receipt under the key, another `unknown` attempt or a replay, is `idempotency_conflict` at `/<i>/action/idempotency_key`: the action was used again before anyone knew what it did.
- Once a key's outcome is known, only replays may follow under it. Another known or `unknown` outcome means the action took effect, or may have, a second time: `idempotency_conflict` at `/<i>/action/idempotency_key`.
- Denied receipts are not part of this check: an attempt the Enforcement point refused, for reusing a key or for any other reason, is a valid record, and reserves nothing ([section 11.2](execution-semantics.md)).

## 9.12 Invariant 11: evidence matches criteria

For each evaluation in the record:

- Every criterion it names exists in the contract; otherwise `criterion_unknown` at `/<i>/criteria/<k>/criterion_id`.
- Every required criterion of the contract appears in it; otherwise `evidence_missing` at `/<i>/criteria`.
- Every `pass` or `fail` carries evidence; otherwise `evidence_missing` at `/<i>/criteria/<k>/evidence` (an object-level rule, [section 7.2](evaluation-result.md)).
- `technical_result` equals the value derived from the contract's required criteria ([section 7.4](evaluation-result.md)); otherwise `technical_result_mismatch` at `/<i>/technical_result`.

## 9.13 Invariant 12: acceptance comes last and from the owner

For each evaluation whose acceptance is decided (`accepted` or `rejected`):

- It is not in revision 1; otherwise `acceptance_premature` at `/<i>/acceptance/decision` (an object-level rule, [section 7.5](evaluation-result.md)).
- Its `execution.status` is not `incomplete`; no idempotency key among the record's receipts, cited or not, has a final outcome of `unknown`; `acceptance.at` is no earlier than the end of every effect of the record (its `finished_at`, or its `started_at` when it has none), nor than the `created_at` of every earlier revision of the same evaluation in the record; and an `accepted` decision needs at least one effect in the record, while a rejection needs none. Otherwise `acceptance_premature` at `/<i>/acceptance`.
- A key's final receipt is, in creation order, its last receipt that records the outcome of an effect (`succeeded`, `failed`, `cancelled` or `unknown`); for a key with none, its last replay; otherwise its last denial. A later known outcome therefore settles an earlier `unknown` under the same key, and a later `unknown` reopens it. Refusals and replays change nothing, so neither makes acceptance premature, however late they come. The PAP rollup uses the same final receipts ([section 15.1](relationships-and-mappings.md)).
- `acceptance.by` is the contract's owner, a stakeholder of the contract whose role is `acceptor`, or a principal the owner delegated acceptance to at `acceptance.at`: the grantee of a binding in the record whose grantor's `person` is the owner, which grants `acceptance.decide` on a resource of type `work_contract` with the contract's `id` or no `id`, which holds an approval by the owner that is current, `approved` and live at that time ([section 5.4](authority-binding.md), conditions 1 to 4) and no such denial by an approver in scope, and which is inside its window at that time, with neither it nor any binding it was delegated from revoked. Otherwise `acceptance_not_owner` at `/<i>/acceptance/by`.

## 9.14 Invariant 13: receipts chain

- Every receipt's `integrity.digest` equals the digest recomputed from the receipt ([section 6.6](execution-receipt.md)); otherwise `digest_mismatch` at `/<i>/integrity/digest`.
- The receipts form one chain, using the digests the receipts state. A record holds one contract revision, so the chain it checks is that revision's, and it restarts at every revision ([section 6.6](execution-receipt.md)). The receipts are taken in creation order, by `created_at` with ties in record order, as in [9.9](invariants.md), whatever order the record lists them in, so "a second receipt", "already names" and "the first receipt" below follow that order:
  - a second receipt with no `previous_digest` is `chain_broken` at `/<i>/integrity`;
  - a `previous_digest` that is no receipt's digest, one that another receipt already names (a fork), or one naming a receipt created later than this one (a link backwards in time) is `chain_broken` at `/<i>/integrity/previous_digest`;
  - receipts with no first receipt among them are `chain_broken` at the first receipt's `/<i>/integrity`.

A receipt that was changed and given a recomputed digest passes the first check, but the next receipt's `previous_digest` no longer names it, so the chain shows the change. A record edited and resealed from the change onwards, every later digest and link recomputed, is consistent again: the change then shows only against a digest the verifier already trusts, and so does a change to the last receipt ([section 6.6](execution-receipt.md)).

## 9.15 Records describe the past

Each of these timestamps is no later than the current time; otherwise `timestamp_in_future` at its path:

- every object's `created_at`;
- each context item's `provenance.retrieved_at` and `freshness.as_of`;
- each approval's `decided_at` and `external.confirmed_at`;
- each receipt's `action.requested_at`, `decision.evaluated_at`, `started_at` and `finished_at`;
- each evaluation's `acceptance.at`.

Expiries, deadlines and window starts (`expires_at`, `valid_from`, `deadline_at`, a business impact window's `starts_at`) may lie in the future.

## 9.16 What the assurance levels add

At the `controlled` and `assured` levels a record check also applies the rules below, which [section 13.2](conformance.md) states for those levels; at `essential` it applies none of them. They are the parts of a level that a record can show. The rest of a level, such as durable idempotency, a live revocation status, an external policy decision point or enforced residency, is a property of the systems that produced the record, and no record check can verify it.

At `controlled` and `assured`:

- Every approval carries an `expires_at`; otherwise `schema_violation` at `/<i>/approvals/<k>/expires_at`.
- When an evaluation's `evaluator.principal` is the `actor.principal` of an effect in the record, every required `automated_check` criterion it judges `pass` or `fail` states a limitation in its own `limitations` ([section 7.7](evaluation-result.md)); otherwise `schema_violation` at `/<i>/criteria/<k>/limitations`. The check sees that a limitation is stated, not what it says.

At `assured` only:

- Every receipt is attested: its `integrity.attestation.kind` is not `none`; otherwise `schema_violation` at `/<i>/integrity/attestation/kind`. The check sees the attestation's kind and reference; it does not verify a signature ([section 6.6](execution-receipt.md)).
- Every receipt names the policy decision it rests on in `decision.policy_ref`; otherwise `schema_violation` at `/<i>/decision/policy_ref`.
- Only `verified` approvals authorize an effect ([9.7](invariants.md), step 4.2). An effect that only `asserted` approvals cover is `approval_missing` at `/<i>/authority`. A PAP confirmation is always asserted, so at this level it never authorizes an effect alone.
- Every binding with a `delegation.parent` records the token exchange behind its hop: a `credential_refs` entry whose `kind` is `token_exchange`; otherwise `schema_violation` at `/<i>/credential_refs`.

A record that passes these rules carries what the level asks of a record. It does not show that the systems behind it meet the level, and a Consumer MUST NOT describe a passing record check as a conformance claim ([section 13.4](conformance.md)).

## 9.17 What the invariants do not check

This section is informative. The invariants check consistency and authority on record; they do not check that the record is true or complete. In particular, they do not check:

- that the actions a receipt describes happened as described, or that every action was recorded;
- the contract's `constraints`, `limits.risk` or `deadline_at` ([section 3.4](work-contract.md));
- disclosure, redactions, classification, residency or retention ([section 4.5](context-manifest.md));
- that a binding's `grantor.organization` is the contract's organization;
- that an evaluation cites each action's final receipt ([section 7.1](evaluation-result.md)): acceptance is checked against every receipt of the record, whatever the evaluation cites;
- that an effect cites a context manifest at all;
- that spending across the contract's revisions stays within its cost limit, or that idempotency keys stay closed across revisions: the record holds one revision, and the Enforcement point checks both ([section 11.1](execution-semantics.md), [section 11.2](execution-semantics.md)).

[Section 17](non-goals-and-open-questions.md) lists these among the limitations and open questions.

## 9.18 Examples

At the Controlled level with a threshold of USD 120.00, a record in which the requester, Dana, approved her own grants fails separation of duties. Five grants are over the threshold: four have no cost limit, and CI may cost up to USD 150.00. The preview grant, capped at USD 100.00, is not over.

```json
{
  "error": "invalid_bundle",
  "error_description": "The bundle breaks a cross-object invariant.",
  "details": [
    {
      "code": "separation_of_duties",
      "path": "/2/grants/0",
      "message": "needs an approver other than the requester"
    },
    {
      "code": "separation_of_duties",
      "path": "/2/grants/1",
      "message": "needs an approver other than the requester"
    },
    {
      "code": "separation_of_duties",
      "path": "/2/grants/2",
      "message": "needs an approver other than the requester"
    },
    {
      "code": "separation_of_duties",
      "path": "/2/grants/3",
      "message": "needs an approver other than the requester"
    },
    {
      "code": "separation_of_duties",
      "path": "/2/grants/4",
      "message": "needs an approver other than the requester"
    }
  ]
}
```

A record in which a grant changed after Priya approved the binding: the approval covers the old binding digest, so it is stale, and every effect under it is reported.

```json
{
  "error": "invalid_bundle",
  "error_description": "The bundle breaks a cross-object invariant.",
  "details": [
    {
      "code": "approval_stale",
      "path": "/2/approvals/0/covers/binding_digest",
      "message": "covers another binding"
    },
    {
      "code": "approval_stale",
      "path": "/3/authority",
      "message": "no valid approval covered this action when it started"
    },
    {
      "code": "approval_stale",
      "path": "/5/authority",
      "message": "no valid approval covered this action when it started"
    },
    {
      "code": "approval_stale",
      "path": "/6/authority",
      "message": "no valid approval covered this action when it started"
    },
    {
      "code": "approval_stale",
      "path": "/7/authority",
      "message": "no valid approval covered this action when it started"
    },
    {
      "code": "approval_stale",
      "path": "/8/authority",
      "message": "no valid approval covered this action when it started"
    }
  ]
}
```

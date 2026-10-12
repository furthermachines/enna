---
title: Execution semantics
description: How an Enforcement point gates each action, and how retries, partial failure, cancellation, stale context, scope changes, approval invalidation, idempotent replay, conflicting reuse and recovery work, with what is recorded versus enforced and no exactly-once promise.
order: 11
status: normative
---

# Execution semantics

This section says what happens around each action: the checks before it, what is recorded after it, and how work recovers when something goes wrong. Its rules bind Enforcement points and the Producers of receipts. They adopt PAP 0.1's operation semantics where PAP has one: an approval binds exactly one revision, effects happen at most once, nothing is reported as failed while its outcome is unknown, and a final result describes every change, partial ones included.

## 11.1 The gate before each action

Before an action starts, at the current time `t`, an Enforcement point first looks up the action's idempotency key ([11.2](execution-semantics.md)):

- A key whose earlier action had the same name, resource and payload, with a known outcome, is replayed, without the checks below: a replay is not an effect.
- A key whose earlier action had another name, resource or payload is refused with `idempotency_conflict` alone.
- A key whose earlier outcome is unknown is refused until that outcome is established ([11.3](execution-semantics.md)).
- A key that was never used, or was used only by refused attempts, goes on to the checks. A refused attempt reserves nothing.

An Enforcement point that knows the contract's lifecycle state refuses every action, replays included, unless the contract is `authorized` or `executing` ([section 10.1](lifecycle.md)), with `illegal_transition` among the reasons. A planned action or a retry is no exception. A used key is still decided by the lookup above, without the checks below: a replay refused this way has `illegal_transition` as its only reason, and a conflict has `idempotency_conflict` and `illegal_transition`.

Then it MUST check, against the authority binding the action would run under and the record so far:

1. that the record so far passes steps 1 to 4 of [section 9.1](invariants.md), that its receipt chain holds ([section 9.14](invariants.md)), that it describes no future ([section 9.15](invariants.md)), that it keeps every rule [section 9.16](invariants.md) adds at the Enforcement point's level other than the one on verified approvals, which check 6 applies, and that it knows the profile of the contract and of the binding ([section 2.14](conventions.md)). A failure here refuses every action, whichever object it is found in, another agent's binding included: the other checks cannot be trusted over such a record;
2. that the agent asking is the binding's grantee ([section 9.7](invariants.md)), since authority belongs to the agent it was granted to;
3. that `t` is inside the binding's window ([section 5.5](authority-binding.md));
4. that neither the binding, nor any binding it was delegated from, nor the approval it relies on is revoked ([section 5.6](authority-binding.md)). At the Essential level the runtime's own revocation registry is enough; at the Controlled level and above, the Enforcement point MUST consult the live status that `revocation.status_ref` points to, before every effect;
5. that a grant covers the action ([section 5.2](authority-binding.md));
6. that approvals authorize it, from as many distinct people as its grant's `min_approvals` requires, and no denial blocks it ([section 5.4](authority-binding.md)), including separation of duties at the Controlled level and above ([section 9.6](invariants.md)), and only `verified` approvals at the Assured level ([section 9.16](invariants.md));
7. that the action's resource, and every item of the context manifest it cites, lie inside the tenant boundary that the contract and the action's own binding allow ([section 9.4](invariants.md));
8. that every item of the context manifest the action cites is fresh at `t` ([section 4.3](context-manifest.md));
9. that the action stays within the cost and count limits of its grant and of the contract, as far as its cost can be known before it starts. The contract's limit covers what every revision of the contract has spent, since a new revision does not reset the budget. An Enforcement point MUST refuse every further action under a limit that recorded costs have already passed, whatever the action quotes.

If any check fails, the Enforcement point MUST refuse the action before any side effect, and issue a receipt whose decision is `deny`, with every failing check's code among its `reasons` ([section 6.3](execution-receipt.md)). When the reason is a business reason ([section 10.4](lifecycle.md)) that stops the work, the contract is blocked with it.

The cheapest refusals come first. An action with the wrong version or shape, an unknown action, or one outside every grant is refused before anything expensive or visible is done for it.

## 11.2 Idempotency: replay and conflicting reuse

Every action carries an idempotency key, chosen by the party that requests the action, and the digest of its payload ([section 6.2](execution-receipt.md)). The key identifies one logical action across retries, as PAP 0.1's client-chosen message IDs do: one action name, one resource and one payload. The same key for the same action, resource and payload replays; the same key with any of the three different conflicts.

- An Enforcement point MUST compute the payload digest itself, from the payload it is asked to act on, rather than trust a digest the caller supplies.
- **A key is reserved by the first permitted attempt under it.** A refused attempt reserves nothing: the same action MAY be attempted again under the same key once the cause of the refusal is removed.
- **Same key, same action, resource and payload.** The Enforcement point MUST NOT perform the action again. It returns the result of the earlier action, and records a receipt whose outcome is `replayed`, with no effects and no new cost, whose summary names the earlier receipt. A replay is not an effect: it needs no live authority, and it uses none of a grant's count. Its decision is `permit`, made when the replay was answered; the authorization it rests on is the earlier receipt's. A replay cites the authority and context of the earlier receipt when that receipt belongs to the same contract revision; a replay of an action recorded under an earlier revision cites neither, because the record of its own revision holds neither ([section 9.3](invariants.md)).
- **Same key, another action, resource or payload.** The Enforcement point MUST refuse the action with no effect, and record a receipt whose decision is `deny` with the reason `idempotency_conflict`. A key reused for another action is never answered as a replay: the caller would be told that an action happened that never ran.
- **Same key, earlier outcome unknown.** The Enforcement point MUST first establish what happened to the earlier action ([11.3](execution-semantics.md)).
- An Enforcement point MUST apply these rules to every key it has seen for a contract, across all of the contract's revisions, so that a new revision does not reopen a key. The record check of [section 9.11](invariants.md) sees one revision at a time and cannot verify this.
- At the Controlled level and above, the keys and their payload digests MUST be stored durably, and checking a key and reserving it MUST be one atomic step, so that the rules hold across concurrent requests, restarts and crashes.

Continuing the record of [section 9](invariants.md): the coding agent's runtime retried the CI request after a lost response, with the same key and payload. The Enforcement point replayed the earlier result.

```json
{
  "protocol_version": "0.1",
  "type": "execution_receipt",
  "id": "rcpt_ci_retry",
  "created_at": "2026-10-13T11:40:02Z",
  "scope": {
    "organization": "harborpine.example",
    "workspace": "internal",
    "project": "brand-approvals"
  },
  "profile": "software-delivery/0.1",
  "contract": {
    "id": "wc_brand_approvals",
    "revision": 1
  },
  "context": {
    "id": "ctx_brand_approvals",
    "revision": 1
  },
  "authority": {
    "id": "auth_coding_agent",
    "revision": 1
  },
  "action": {
    "name": "ci.run",
    "resource": {
      "type": "pipeline",
      "id": "collab-app-ci",
      "scope": {
        "organization": "harborpine.example",
        "workspace": "internal",
        "project": "brand-approvals"
      }
    },
    "requested_at": "2026-10-13T11:40:00Z",
    "idempotency_key": "brand-approvals-ci-1",
    "payload_digest": "sha256:f877baea3633b43df2d10f966e172cf53d9e38dc1a1f7396a1d4bf41c0f2371d"
  },
  "actor": {
    "principal": {
      "kind": "agent",
      "id": "https://agents.harborpine.example/coding",
      "display": "Coding agent"
    },
    "performed_by": "agent",
    "runtime": {
      "name": "fixture-runtime",
      "version": "0.1.0"
    }
  },
  "decision": {
    "result": "permit",
    "reasons": [],
    "evaluated_at": "2026-10-13T11:40:01Z",
    "policy_ref": "harborpine-agent-work@1"
  },
  "outcome": {
    "status": "replayed",
    "summary": "Same key and payload as rcpt_ci: returned that run's result. Nothing ran again."
  },
  "started_at": "2026-10-13T11:40:02Z",
  "finished_at": "2026-10-13T11:40:02Z",
  "effects": [],
  "trace": {
    "correlation_id": "brand-approvals"
  },
  "integrity": {
    "canonicalization": "rfc8785",
    "previous_digest": "sha256:652127f2acc181b8739046ff9a5432071cdc215ae685eec61530a41a4fa19c4a",
    "attestation": {
      "kind": "none"
    },
    "digest": "sha256:0a7414ea94b133327bfbbaf3cc17bd4648536617b89778cb841c73601d60f21f"
  }
}
```

A few minutes later, a request reused the same key with a different payload. The Enforcement point refused it before running anything.

```json
{
  "protocol_version": "0.1",
  "type": "execution_receipt",
  "id": "rcpt_ci_conflict",
  "created_at": "2026-10-13T11:45:01Z",
  "scope": {
    "organization": "harborpine.example",
    "workspace": "internal",
    "project": "brand-approvals"
  },
  "profile": "software-delivery/0.1",
  "contract": {
    "id": "wc_brand_approvals",
    "revision": 1
  },
  "context": {
    "id": "ctx_brand_approvals",
    "revision": 1
  },
  "authority": {
    "id": "auth_coding_agent",
    "revision": 1
  },
  "action": {
    "name": "ci.run",
    "resource": {
      "type": "pipeline",
      "id": "collab-app-ci",
      "scope": {
        "organization": "harborpine.example",
        "workspace": "internal",
        "project": "brand-approvals"
      }
    },
    "requested_at": "2026-10-13T11:45:00Z",
    "idempotency_key": "brand-approvals-ci-1",
    "payload_digest": "sha256:e083a2f343893c9396a82d8c852115a6d8027f80474783fddb1b138a4b735486"
  },
  "actor": {
    "principal": {
      "kind": "agent",
      "id": "https://agents.harborpine.example/coding",
      "display": "Coding agent"
    },
    "performed_by": "agent",
    "runtime": {
      "name": "fixture-runtime",
      "version": "0.1.0"
    }
  },
  "decision": {
    "result": "deny",
    "reasons": [
      "idempotency_conflict"
    ],
    "evaluated_at": "2026-10-13T11:45:01Z",
    "policy_ref": "harborpine-agent-work@1"
  },
  "outcome": {
    "status": "denied",
    "summary": "Refused before running: the key brand-approvals-ci-1 was already used with another payload."
  },
  "effects": [],
  "trace": {
    "correlation_id": "brand-approvals"
  },
  "integrity": {
    "canonicalization": "rfc8785",
    "previous_digest": "sha256:0a7414ea94b133327bfbbaf3cc17bd4648536617b89778cb841c73601d60f21f",
    "attestation": {
      "kind": "none"
    },
    "digest": "sha256:31e2a6d2ecc9e9f5a9ed9674b5b4594623489ccbe5b065e4bce47ae1af057385"
  }
}
```

With both receipts appended, the record of [section 9](invariants.md) is still valid at the Controlled level: the replay is not a second effect, and the conflicting request never ran.

## 11.3 Unknown outcomes and reconciliation

When an Enforcement point cannot tell whether an action took effect, because a request timed out, a downstream system gave no answer, or the runtime crashed between starting the action and recording it, then:

- it MUST record the outcome `unknown`, and MUST NOT record `failed`;
- it MUST NOT perform the action again, under the same key or a new one, until it knows the outcome, unless the downstream system deduplicates by that key;
- it SHOULD ask the downstream system what happened, using the idempotency key or the business reference it passed on;
- once it knows, it issues one more receipt under the same key and payload with the known outcome, whose summary describes every change that was made. That receipt reconciles the unknown one and is not a second effect ([section 9.11](invariants.md)).

```jsonc
{
  // The receipt issued when the CI service timed out. A later receipt under the
  // same idempotency key and payload records the known outcome.
  "outcome": {
    "status": "unknown",
    "summary": "The CI service timed out; whether run 813 started is not known."
  }
}
```

While an outcome is unknown, the contract stays `executing` ([section 10.3](lifecycle.md)), the PAP operation stays `in_progress` ([section 10.7](lifecycle.md)), and an evaluation that cites the receipt cannot be accepted ([section 9.13](invariants.md)). An evaluation that relies on a reconciled action SHOULD cite the receipt that records the known outcome.

## 11.4 Retries

- A retry after a transport failure is the same logical action. It carries the same idempotency key and payload, and the Enforcement point replays or reconciles it. It does not change the business state ([section 10.4](lifecycle.md)).
- A retry after a known failure is a new attempt. It carries a new idempotency key, it counts against the grant's `count`, and its cost counts against the limits. It SHOULD name the failed attempt's key in `action.retry_of`, so that whoever rolls the work up, such as a PAP operation's result ([section 15.1](relationships-and-mappings.md)), sees one action and its last attempt rather than a failure that was later fixed.
- A party that answers with HTTP 429 sends `Retry-After`, and an Enforcement point, or any Enna party acting as a client, MUST wait at least that long before it calls again, as in PAP 0.1.

## 11.5 Partial failure

- An action that made some changes and then failed has the outcome `failed`. Its `effects` list each change that was made, and its `summary` describes every change, partial ones included.
- When some actions of a contract succeed and others fail, the contract still reaches `execution_complete` once every outcome is known. The evaluation reports `execution.status` `partial`, and the owner decides at acceptance what the partial result is worth ([section 7.3](evaluation-result.md)).
- This draft defines no automatic compensation. Undoing a change is a new action, with its own grant, approval and receipt.

## 11.6 Cancellation

- **An action.** If an Enforcement point stops an action, the receipt's outcome is `cancelled` and lists any changes made before it stopped. An action asked to stop stays `unknown` until the Enforcement point knows whether it stopped, as a PAP operation stays `in_progress` after a cancel until the business knows.
- **A contract.** A contract cannot be cancelled while it is `executing` ([section 10.3](lifecycle.md)). To cancel work in progress, stop new effects by revoking the authority or blocking the contract, wait until every outcome is known, then apply `complete_execution` and `cancel`. In the PAP rollup, a cancelled receipt makes the operation `cancelled` ([section 10.7](lifecycle.md)).

## 11.7 Stale context

When an item of the cited manifest is stale or of unknown age, the Enforcement point refuses the action with `context_stale`, and the contract is blocked with that reason. To recover, the Producer refreshes the items, issues a new revision of the manifest with the new `as_of` and `retrieved_at`, and the contract resumes; later actions cite the new manifest revision. No new approval is needed: approvals cover the contract revision and the grants, not the context ([section 5.3](authority-binding.md)).

## 11.8 Scope changes

- **The work changes.** Any change to the contract issues a new revision, which returns to `awaiting_approval` ([section 10.3](lifecycle.md)) and needs its own authority bindings and approvals. The record of the earlier revision stays as it was.
- **An agent's authority changes.** A new revision of a binding that changes its grants, window, policy, delegation or anything else but its approvals has a different binding digest, so every approval of that binding becomes stale ([section 9.5](invariants.md)). Until the binding is approved again, the Enforcement point refuses effects under it with `approval_stale`.
- **The policy changes.** A policy change that adds a condition to a grant, such as a required approver role, changes the grants and so invalidates the approvals. A policy change that changes no grant does not invalidate approvals by itself; an organization whose policy change must invalidate them revokes them.

## 11.9 Approval invalidation

| What happened | The Enforcement point refuses with | The way back |
| --- | --- | --- |
| The approval expired | `approval_expired` | A new approval, after `request_approval` |
| The approval was revoked | `approval_revoked` | A new approval, after `request_approval` |
| An approver in scope denied | `approval_denied` | A new revision that answers the objection, or a decision from the approver that withdraws the denial by revoking it |
| The contract or the grants changed | `approval_stale` | Approval of the current revision and binding |
| The approver is not in scope, or not in a role the grant requires | `approver_out_of_scope` | An approval from someone in scope, in a required role |
| The requester approved alone above the threshold (Controlled and above) | `separation_of_duties` | An approval from someone other than the requester |

In each case the contract is blocked with that reason, and it leaves `blocked` only through `request_approval` and a new approval, or through `revise`; never through `resume` ([section 10.3](lifecycle.md)).

## 11.10 Recovery

| After | Recovery |
| --- | --- |
| An approval problem | Explicit reapproval: `request_approval`, then a new approval recorded in the binding, then `approve`. |
| Stale context | A new manifest revision with fresh items, then `resume`. |
| A revoked binding | A new binding with a new `id`, since a revocation covers every revision of the binding it names, and approval of it. |
| An expired binding | A new binding revision with the new window, and a new approval of it, since the window is part of what an approval covers ([section 5.3](authority-binding.md)). |
| A passed limit | A new contract revision with a higher limit, approved like any revision, or `cancel`. |
| A crash | For every key that was reserved without a recorded outcome: a receipt with the outcome `unknown`, then reconciliation ([11.3](execution-semantics.md)). |
| A failed action | A new attempt under a new key, with the failed attempt's key in `action.retry_of`, if the grant's limits allow it. |

## 11.11 Recorded versus enforced

JSON cannot enforce anything. An Enforcement point enforces before an action; a receipt records what it decided and what happened; a record check verifies afterwards that the record is consistent. A record that shows a violation is evidence that enforcement failed, not a prevention.

| Rule | Enforced before the action by | Recorded in | Checked afterwards by |
| --- | --- | --- | --- |
| Authority window and revocation | The Enforcement point, with the live status | Receipt `decision` and `reasons` | The record check ([9.7](invariants.md)) |
| Grants, approvals, separation of duties | The Enforcement point | Receipt `decision`; binding `approvals` | The record check ([9.5](invariants.md) to [9.7](invariants.md)) |
| Tenant boundary | The Enforcement point | Receipt `action.resource.scope` | The record check ([9.4](invariants.md)) |
| Context freshness | The Enforcement point | Receipt `context`; manifest `freshness` | The record check ([9.8](invariants.md)) |
| Cost and count limits | The Enforcement point, as far as cost is known in advance | Receipt `cost` | The record check ([9.9](invariants.md)) |
| Idempotency | The Enforcement point's key store | Receipt `action` and `outcome` | The record check ([9.11](invariants.md)), within one revision |
| Receipt integrity | Nobody: it is computed, not enforced | Receipt `integrity` | The record check ([9.14](invariants.md)), against a trusted anchor |
| Constraints, risk level, deadline | Nobody, unless expressed as grants and limits | Contract | Nobody |
| Disclosure, classification, residency, retention | The systems that hold the context, at the Assured level | Manifest | Nobody |
| That an action happened as described | Nobody | Receipt | Attestation and audit, outside this draft |

## 11.12 Durability and the limits of the promise

Enna does not promise exactly-once execution. What a conforming Enforcement point promises is at most one effect per idempotency key, as far as its own state reaches:

- At the Essential level its key store may live in memory. Deduplication then lasts only as long as the process: it demonstrates the semantics and is not a safety guarantee for real effects.
- At the Controlled level and above its key store MUST be durable and its check-and-reserve atomic, and the conformance statement of an Enforcement point that claims the level MUST document how it recovers after a crash.
- Even then, an effect in a downstream system can happen twice if that system does not deduplicate and the Enforcement point crashes after the effect but before recording it. An Enforcement point SHOULD pass the idempotency key, or a key derived from it, to downstream systems that deduplicate, as PAP's guides suggest deriving downstream keys from the operation ID, and SHOULD reconcile before any retry.

The reference runtime (`@enna/reference-runtime`), in the project's private build repository, is a demonstration. Its effects are simulated, and its idempotency and revocation stores live in memory, so it claims the Essential level only ([section 13.6](conformance.md)). By default its gate and its record checks apply the Controlled level's record rules: separation of duties, declared thresholds and quorums, and expiring approvals ([section 9.16](invariants.md)). Its key store and the spending it counts against the contract's limit both span every revision of a contract. It does not meet the Controlled level: its stores are in memory, and it consults its own revocation registry rather than a live status at `revocation.status_ref`.

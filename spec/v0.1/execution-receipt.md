---
title: Execution receipt
description: The immutable record of one proposed or performed action, with its authority, decision, outcome, changes, artifacts, cost, trace and integrity, and what its digest shows.
order: 6
status: normative
---

# Execution receipt

An execution receipt (`execution_receipt`) is the immutable record of one action that an agent proposed or performed for a contract: who acted, under which authority and context, what the Enforcement point decided and why, what happened, every change that was made, what was produced and what it cost. Refused actions get receipts too. Receipts have no revisions; a later fact is a later receipt. Its schema is `execution-receipt.schema.json`.

## 6.1 Members

Besides the members every object carries ([section 2.16](conventions.md)); a receipt has no `revision`:

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `contract` | contract reference | Yes | The contract revision the action served. |
| `context` | context reference | No | The manifest revision whose context the action used. |
| `authority` | authority reference | When the decision is `permit` | The binding revision the action was taken under. |
| `action` | object | Yes | What was asked for ([6.2](execution-receipt.md)). |
| `actor` | object | Yes | `principal`, required: who acted. `performed_by`, required: `agent` or `human`. `runtime`, optional: `name` and `version`, both required, and an optional `adapter`. |
| `decision` | object | Yes | `result` (`permit` or `deny`), `reasons` (distinct error codes) and `evaluated_at`, all required, and an optional `policy_ref` (text): the decision and the policy that made it. |
| `outcome` | object | Yes | `status` and `summary`, both required ([6.3](execution-receipt.md)). |
| `started_at` | timestamp | When the decision is `permit` | When the action started. |
| `finished_at` | timestamp | No | When the action finished. |
| `effects` | array | No | Each change made: a `description`, required, and an optional `external_ref` (text) in the system that changed, such as a git ref or a ticket number. |
| `artifacts` | array | No | What the action produced ([6.4](execution-receipt.md)). |
| `cost` | money | No | What the action cost. |
| `trace` | object, at least one member | No | `traceparent`, `correlation_id` and `transport` ([6.5](execution-receipt.md)). |
| `integrity` | object | Yes | `canonicalization`, `digest`, `previous_digest` and `attestation` ([6.6](execution-receipt.md)). |

`actor.performed_by` says whether a person or an agent performed the step, as PAP marks every message as written by a person or an agent. A person may act through an agent's runtime, and an agent may act for a person; the receipt records which one actually did it. For an effect, `actor.principal` is always the grantee of the binding the effect runs under: a person who operates that agent is recorded with `performed_by` `human`, not as the principal ([section 9.7](invariants.md)).

## 6.2 Action

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `name` | action name | Yes | The action, matched against grants ([section 5.2](authority-binding.md)). |
| `resource` | resource | Yes | `type`, an optional `id`, and `scope`: what the action is on. |
| `requested_at` | timestamp | Yes | When the action was asked for. |
| `idempotency_key` | 1 to 256 characters from the URL-safe base64 alphabet | Yes | Identifies the logical action across retries: one action name, one resource and one payload ([section 11.2](execution-semantics.md)). |
| `payload_digest` | digest | Yes | The digest of the canonical form of the action's request payload. |
| `retry_of` | idempotency key | No | The key of the earlier attempt at the same action that this one retries after a known failure ([section 11.4](execution-semantics.md)). |

The payload is whatever the action's request carries: the parameters of a tool call, the body of an API request. The Producer computes its digest with the rule in [section 2.15](conventions.md); the payload itself is not in the receipt.

## 6.3 Decision and outcome

| `outcome.status` | Meaning |
| --- | --- |
| `succeeded` | The action took effect as asked. |
| `failed` | The action is known not to have completed. `summary` and `effects` list every change it did make. |
| `denied` | The Enforcement point refused the action before it started. Nothing happened. |
| `cancelled` | The action was stopped. `summary` and `effects` list every change it made before it stopped. |
| `unknown` | Whether the action took effect is not yet known ([section 11.3](execution-semantics.md)). |
| `replayed` | The same idempotency key and payload as an earlier receipt: this receipt returns that result, and nothing happened again ([section 11.2](execution-semantics.md)). |

The schema ties the decision to the outcome:

- A `deny` decision has at least one reason, the outcome `denied`, no effects, no artifacts, no `cost`, and no `started_at` or `finished_at`. Empty `effects` and `artifacts` arrays are allowed. A refused attempt cost nothing, so no total ever counts it ([section 9.9](invariants.md)).
- A `permit` decision has a `started_at`, no reasons, and any outcome except `denied`. It has an `authority` reference unless its outcome is `replayed`.
- A `replayed` outcome has no effects. A replay of an action recorded under an earlier contract revision names no `authority` and no `context`, since its own revision's record holds neither ([section 11.2](execution-semantics.md)).
- An `unknown` outcome has no `finished_at`, no effects and no artifacts: a receipt cannot say both that nobody knows whether the action took effect and what it changed. The later receipt that records the known outcome lists them ([section 11.3](execution-semantics.md)).

Further rules:

- `outcome.summary` describes, in plain language, every change the action made, including partial changes from an action that failed or was cancelled. This is PAP's rule for an operation's result.
- A Producer MUST NOT record `failed` while it does not know whether the action took effect; it records `unknown` and, once it knows, issues a later receipt with the known outcome ([section 11.3](execution-semantics.md)).
- `decision.reasons` are codes from the error catalog ([section 8.4](envelope-and-errors.md)), the same codes a validator would report for the problem. A denial for reusing an idempotency key with another action, resource or payload has the reason `idempotency_conflict`, and a refusal because the contract is neither `authorized` nor `executing` has `illegal_transition` ([section 11.1](execution-semantics.md)).
- A receipt's times run forward: `action.requested_at`, then `decision.evaluated_at`, then `started_at`, then `finished_at`, each no earlier than the one before. A Consumer reports a time earlier than the one before it as a `schema_violation` at that later member: the decision comes before the effect it authorizes. A Producer SHOULD record `finished_at` once the outcome is known.
- A receipt's `cost` is what this action cost. A replay costs nothing new, and a Producer SHOULD omit `cost` from it.

## 6.4 Artifacts

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `id` | identifier (`art_`) | Yes | Unique within the receipt. |
| `kind` | token | Yes | What the artifact is, such as `branch` or `test_report`. Profiles define kinds ([section 14.3](software-delivery-profile.md)). |
| `ref` | URI | Yes | Where the artifact is. |
| `media_type` | media type | Yes | Its format, such as `application/json`. |
| `digest` | artifact digest | Yes | The digest of its content, with the algorithm ([section 2.15](conventions.md)). |

An artifact is referenced, never embedded. Its digest lets a later reader check that the artifact they fetch is the one the action produced.

## 6.5 Trace

- `traceparent` is a W3C Trace Context Level 1 `traceparent` value (`00-` then 32 hex digits, 16 hex digits and 2 hex digits, separated by `-`), linking the receipt to the trace of the action. The schema checks the shape; a trace or parent identifier of all zeros is invalid under Trace Context, and a Consumer MUST ignore such a value.
- `correlation_id` (text) ties the receipt to the work it belongs to, such as an envelope's correlation ([section 8.1](envelope-and-errors.md)).
- `transport`, with at least one member, holds the identifiers of the transport the action travelled on: `a2a_task_id`, `a2a_context_id`, `mcp_request_id` and `pap_operation_id` ([section 15](relationships-and-mappings.md)).

## 6.6 Integrity

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `canonicalization` | `"rfc8785"` | Yes | How the receipt is put in canonical form. |
| `digest` | digest | Yes | The receipt's digest. |
| `previous_digest` | digest | No | The digest of the previous receipt of the same contract revision; absent on the first receipt of each revision. |
| `attestation` | object | Yes | `kind`, required: `none`, `jws` or `in_toto`. `ref`: required for `jws` and `in_toto`, and absent for `none`: where the attestation is. |

**Digest.** `integrity.digest` is `sha256:` and the lowercase hex SHA-256 of the RFC 8785 canonical form of the receipt with `integrity.digest` removed and every other member kept, including `integrity.previous_digest` and `integrity.attestation`. A Producer computes it last. A Consumer that verifies a receipt recomputes it; a difference is `digest_mismatch`.

**Chain.** The receipts of one contract revision form one chain, in the order they were created:

- exactly one receipt, the first, has no `previous_digest`;
- every other receipt's `previous_digest` is the digest of a receipt in the same record;
- no two receipts name the same previous receipt, so the chain does not fork;
- no receipt was created before the receipt it follows.

Anything else is `chain_broken` ([section 9.14](invariants.md)). Because a receipt's digest covers its `previous_digest`, changing any receipt changes every digest after it.

A chain runs within one contract revision and restarts at the next: the first receipt of every revision has no `previous_digest`, because a record holds exactly one revision ([section 9.1](invariants.md)) and the receipts of an earlier revision are not in it. A receipt that links into an earlier revision's chain is `chain_broken`. Nothing in 0.1 ties the chains of one contract's revisions together, so a later revision's record cannot show that an earlier revision's record is complete; how to tie them is an open question ([section 17.3](non-goals-and-open-questions.md)).

**What a digest shows.** A chain that checks shows that the receipts are consistent with each other. A digest shows a record is unchanged only against a digest the verifier already trusts, and an edited record can be resealed:

- If a verifier holds a receipt's digest from a source it trusts independently of the record, such as its own copy, the counterparty's copy or a signed attestation, then a matching receipt is the one that digest was computed over, and a matching chain is the history up to it. The reference command line takes such a digest as `enna verify --anchor`, and verifies only a chain that ends at it.
- Without such an anchor, a digest shows only that the record agrees with itself. Whoever produced the record, or holds it, can rewrite it and compute new digests.
- A digest never proves that an action was authorized, that the authority on record was real, that the producer recorded every action, or that what the receipt says happened did happen. Authorization is proven, as far as anything proves it, by the Enforcement point's checks and the provider's own credentials ([section 5](authority-binding.md)); honesty by attestation, separation of duties and audit.

**Attestation.** `attestation` is the hook for a signature over the receipt by the runtime's workload identity: a JWS (RFC 7515) or an in-toto Statement in a DSSE envelope ([section 15.7](relationships-and-mappings.md)), stored where `ref` points. This draft does not define the signed payload or how to verify it; that is proposed in section 15 and open in [section 17.3](non-goals-and-open-questions.md). Receipts with attestation `none` are unsigned. A Producer or Consumer that shows receipts to people MUST NOT present one with attestation `none` as signed, and SHOULD label it unsigned. The reference implementation's demonstration receipts all carry `none`, and every place that shows them says they are unsigned. At the Assured level, receipts MUST be attested ([section 13.2](conformance.md)).

## 6.7 Examples

The coding agent created a branch under its binding. The receipt is the first of the contract's chain, so it has no `previous_digest`, and it is unsigned.

```json
{
  "protocol_version": "0.1",
  "type": "execution_receipt",
  "id": "rcpt_branch",
  "created_at": "2026-10-13T09:00:03Z",
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
    "name": "repo.branch.create",
    "resource": {
      "type": "repository",
      "id": "harborpine/collab-app",
      "scope": {
        "organization": "harborpine.example",
        "workspace": "internal",
        "project": "brand-approvals"
      }
    },
    "requested_at": "2026-10-13T09:00:00Z",
    "idempotency_key": "brand-approvals-branch",
    "payload_digest": "sha256:75ecd2c21ee973a586a27bffacaca3a370620588c633b70b68d17f93bde292df"
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
    "evaluated_at": "2026-10-13T09:00:01Z",
    "policy_ref": "harborpine-agent-work@1"
  },
  "outcome": {
    "status": "succeeded",
    "summary": "Created branch feat/brand-approval-gate from main."
  },
  "started_at": "2026-10-13T09:00:02Z",
  "finished_at": "2026-10-13T09:00:03Z",
  "effects": [
    {
      "description": "Branch feat/brand-approval-gate created.",
      "external_ref": "git:harborpine/collab-app#refs/heads/feat/brand-approval-gate"
    }
  ],
  "artifacts": [
    {
      "id": "art_branch",
      "kind": "branch",
      "ref": "https://git.harborpine.example/collab-app/tree/feat/brand-approval-gate",
      "media_type": "text/plain",
      "digest": "sha1:3f786850e387550fdab836ed7e6dc881de23001b"
    }
  ],
  "trace": {
    "correlation_id": "brand-approvals"
  },
  "integrity": {
    "canonicalization": "rfc8785",
    "attestation": {
      "kind": "none"
    },
    "digest": "sha256:06d91777872676cbd1519356e82a020889dbcc32b68af0823a4572234e8b7d20"
  }
}
```

Later the agent tried to read comments in the Lumen Beverages workspace, which no grant names. The Enforcement point refused before any read; the refusal is the next link in the chain.

```json
{
  "protocol_version": "0.1",
  "type": "execution_receipt",
  "id": "rcpt_lumen_read_denied",
  "created_at": "2026-10-13T09:10:01Z",
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
  "action": {
    "name": "harborpine.example/workspace_comments.read",
    "resource": {
      "type": "workspace_comments",
      "scope": {
        "organization": "harborpine.example",
        "workspace": "lumen-beverages"
      }
    },
    "requested_at": "2026-10-13T09:10:00Z",
    "idempotency_key": "brand-approvals-lumen-read",
    "payload_digest": "sha256:615029c029d808cd37be5a134a9ba292dd8cb014793196e447b26c2fe74901ea"
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
      "authority_insufficient",
      "tenant_mismatch"
    ],
    "evaluated_at": "2026-10-13T09:10:01Z",
    "policy_ref": "harborpine-agent-work@1"
  },
  "outcome": {
    "status": "denied",
    "summary": "Refused before any read: no grant names the Lumen Beverages workspace."
  },
  "trace": {
    "correlation_id": "brand-approvals"
  },
  "integrity": {
    "canonicalization": "rfc8785",
    "previous_digest": "sha256:06d91777872676cbd1519356e82a020889dbcc32b68af0823a4572234e8b7d20",
    "attestation": {
      "kind": "none"
    },
    "digest": "sha256:3e7f314d0e085ac0001978a47bd84d5c050c11e30bc9b7f2e4f88998215e0ffc"
  }
}
```

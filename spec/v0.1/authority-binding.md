---
title: Authority binding
description: The object that describes and binds the authority an agent holds for one contract revision, with grants, limits, approvals, expiry, revocation and delegation.
order: 5
status: normative
---

# Authority binding

An authority binding (`authority_binding`) describes the authority an agent holds for one work contract revision, and binds it to that revision: which actions on which resources, within which limits, approved by whom, from when until when, where to check whether it was revoked, and whether it may be delegated further. It is revisable. Its schema is `authority-binding.schema.json`.

JSON cannot grant authority. A binding describes authority that exists elsewhere, in an identity provider, an OAuth grant or a policy decision point, and records the approvals behind it. Enforcement happens at the real policy or provider boundary: an Enforcement point checks the binding and the live state it points to before each action, and the systems the agent touches check their own credentials. An agent, or a language model reading a binding, cannot create authority by writing one.

## 5.1 Members

Besides the members every object carries ([section 2.16](conventions.md)):

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `contract` | contract reference | Yes | The one contract revision this authority serves. |
| `grantor` | object | Yes | `organization` (a domain, [section 2.10](conventions.md)) and `person` (a person reference): the organization that grants, and the person who grants for it. Both required. |
| `grantee` | agent reference | Yes | The agent that receives the authority. Only this agent may act under the binding ([section 9.7](invariants.md)). |
| `grants` | array, at least one | Yes | What the agent may do ([5.2](authority-binding.md)). |
| `approvals` | array | No | Decisions about this binding ([5.3](authority-binding.md)). |
| `policy` | object | No | `policy_id` and `policy_version`, both required, and an optional `decision_ref`: the policy under which the binding was issued and the decision that issued it. An optional `sod_threshold` (money) can tighten the separation-of-duties threshold for this binding, never loosen it ([5.7](authority-binding.md)). |
| `valid_from` | timestamp | Yes | The start of the window in which the authority may be used. |
| `expires_at` | timestamp | Yes | The end of that window ([5.5](authority-binding.md)). |
| `revocation` | object | Yes | `status_ref` (URI), required: where the live revocation status is checked ([5.5](authority-binding.md)). |
| `delegation` | object | No | Where this authority came from and whether it may be passed on ([5.6](authority-binding.md)). |
| `credential_refs` | array | No | References to credentials held elsewhere ([5.7](authority-binding.md)). |

Every extension of an authority binding counts as critical, whatever its `critical` member says. A Consumer MUST reject a binding that carries an extension it does not support ([section 2.12](conventions.md)).

## 5.2 Grants

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `action` | action name | Yes | The action this grant permits ([section 2.4](conventions.md)). |
| `resource` | object | Yes | `type` (a token), required. `id` (1 to 2048 characters), optional: absent means any resource of that type. `scope` (a scope), required. |
| `limits` | object, at least one member | No | `cost` (money): the most the actions under this grant may cost in total. `count` (an integer of at least 1): the most distinct actions this grant may cover. |
| `conditions` | object, at least one member | No | `approver_roles`: a list of distinct role tokens, at least one. Only approvals made under one of these roles, by an approver the contract gives that role ([section 3.1](work-contract.md)), authorize actions under this grant. `min_approvals`: an integer of at least 1, the number of distinct people whose approvals an effect under this grant needs ([5.4](authority-binding.md)); absent means one. |

A grant covers an action when all of these hold:

1. the action's name equals the grant's `action`;
2. the action's resource `type` equals the grant's resource `type`;
3. the grant names no resource `id`, or the action's resource has the same `id`;
4. the action's resource `scope` lies within the grant's resource `scope` ([section 2.10](conventions.md)), which includes the project when the grant names one.

When more than one grant could cover an action, the first in array order is the one whose limits and conditions apply.

- `limits.count` counts distinct idempotency keys among the effects this grant covers. A replay, or a second receipt that reconciles an unknown outcome under the same key, is not a second use ([section 11](execution-semantics.md)). A denied attempt uses nothing.
- `limits.cost` caps the total cost of the receipts this grant covers, replays excluded. A grant's cost limit MUST NOT exceed the contract's `limits.cost` ([section 9.9](invariants.md)).
- A grant's resource scope may name another workspace of the contract's organization. Once approvals in scope that count for the grant, other than PAP confirmations, stand behind it, from as many distinct people as its `min_approvals` asks, that extends the record's tenant boundary to that workspace, and a grant whose scope names no workspace extends it to the whole organization ([section 9.4](invariants.md)). Only the grant's own binding's effects may use context from that workspace, and only through a grant with such approvals behind it. A grant can never name another organization: that is a `tenant_mismatch`.

## 5.3 Approvals

An approval records one decision by one person about this binding, for one contract revision.

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `id` | identifier (`apr_`) | Yes | Unique within the organization, like an object's identifier: revocations name approvals by this identifier alone, so two approvals sharing one would be revoked together. The same approval carried into a binding's later revisions keeps its identifier. |
| `approver` | person reference | Yes | Who decided. |
| `approver_role` | token | Yes | The role under which they decided, such as `product_manager` or `compliance`. Grants can require particular roles ([5.2](authority-binding.md)); the role then counts only when the contract gives it to this approver ([section 3.1](work-contract.md)). |
| `decision` | enum | Yes | `approved` or `denied`. |
| `decided_at` | timestamp | Yes | When they decided. |
| `expires_at` | timestamp | No | When the approval stops authorizing anything. |
| `covers` | object | Yes | `contract` (a contract reference) and `binding_digest` (a digest), both required: exactly what was approved. |
| `method` | enum | Yes | `interactive`: the person decided through an interface when asked. `standing`: a permission given earlier covers this binding. `pap_confirmation`: a PAP operation confirmation ([section 15.1](relationships-and-mappings.md)). |
| `assurance` | enum | Yes | `asserted`: the issuing system states that the person decided. `verified`: signed or step-up evidence of the decision exists. |
| `evidence` | array, at least one | Required when `assurance` is `verified` | Each entry has a `kind` (a token) and a `ref` (text), both required, and an optional `digest`: where the evidence of the decision is and what it hashes to. |
| `external` | object | Exactly when `method` is `pap_confirmation` | The PAP confirmation the approval records: `operation_id`, `revision`, `approved_by` (`user` or `standing_permission`) and `confirmed_at`, all required. |

Rules the schema enforces:

- An approval whose `method` is `pap_confirmation` carries `external` and has `assurance` `asserted`. PAP 0.1 states that a confirmation rests on trust and does not prove the User saw the terms, so it cannot be recorded as verified.
- An approval whose `method` is not `pap_confirmation` has no `external`.
- An approval whose `assurance` is `verified` carries `evidence`. This draft does not define the format of that evidence or how to verify it ([section 17.3](non-goals-and-open-questions.md)); `evidence` points to it.

### The binding digest

`covers.binding_digest` is the digest ([section 2.15](conventions.md)) of the binding in its canonical form, with five members removed: `approvals`, `revision`, `revision_author`, `change_summary` and `created_at`. An approval therefore binds one contract revision and one exact binding: its grantor and grantee, every grant in order, its window, policy, revocation, delegation, credential references, profile and extensions. Changing any of them changes the digest, and every approval of the binding becomes stale until it is given again ([section 9.5](invariants.md)). A wider scope, a longer window, a looser policy or a new delegation cannot ride on an old approval.

The five removed members are the ones that change when approvals are recorded. A new revision of a binding that only adds approvals, or carries earlier ones forward, keeps the digest, so the approvals it carries stay current.

## 5.4 Who may approve, and when an approval counts

The approvers in scope for a contract are its owner, every stakeholder whose role is `approver`, and, for an approval whose `method` is `pap_confirmation`, the contract's requester. An approval from anyone else is `approver_out_of_scope`, whether it approves or denies. A PAP confirmation is a customer's word about their own request, so it counts only for grants whose resource scope lies within the contract's own scope ([section 2.10](conventions.md)): it never authorizes a grant that reaches another workspace, and it never extends the tenant boundary ([section 9.4](invariants.md)).

An approval authorizes an effect that starts at instant `t` under a grant when all of these hold:

1. it covers the contract's current revision and the binding's current digest;
2. its `decision` is `approved`;
3. it was decided at or before `t`, and `t` is before its `expires_at`, if it has one;
4. the live status does not show it revoked at `t`;
5. its approver is in scope, and, if the grant has `conditions.approver_roles`, its `approver_role` is one of them and the contract gives that role to its approver; a PAP confirmation counts only when the grant's resource scope lies within the contract's scope;
6. for `production.deploy`, when the contract, the binding or the effect's receipt follows the software-delivery profile, its `method` is not `standing` ([section 14.4](software-delivery-profile.md)).

A grant whose `conditions.min_approvals` is greater than one sets a quorum: an effect under it starts only when approvals that each meet these conditions come from at least that many distinct people. Two approvals by the same person count once. When valid approvals exist but come from fewer people, the record shows `quorum_not_met`.

A denial blocks. If any approver in scope has denied, and that denial covers the current revision and binding, was decided at or before `t`, has not expired and is not revoked, no effect may start under the binding, whatever approvals it also holds (`approval_denied`).

At the Controlled level and above, separation of duties also applies: above a declared cost threshold, the requester cannot be the only approver ([section 9.6](invariants.md)). It holds when each effect starts: once the binding holds a current approval in scope from someone other than the requester, the requester's own approvals no longer authorize effects under grants over the threshold, so they cannot stand in when the independent approval expires or is revoked, and they do not count toward a quorum. At the Essential level an owner may approve their own contract within its limits.

[Section 9.7](invariants.md) defines which error code a record shows when no approval authorizes an effect.

## 5.5 Window and revocation

- An effect may start only inside the binding's window: `valid_from <= t < expires_at` ([section 2.7](conventions.md)). Outside it, the record shows `authority_expired`. A binding whose `expires_at` is not later than its `valid_from` authorizes nothing.
- A binding's revocation status lives outside the binding, at `revocation.status_ref`. Nothing in the object says whether it is revoked, and a Consumer MUST NOT treat the absence of such a statement as evidence that it is not revoked. The live status is checked at the Enforcement point ([section 11.1](execution-semantics.md)).
- A revocation names either a binding, which revokes all of its revisions and every binding delegated from it ([5.6](authority-binding.md)), or an approval by its `id`. It takes effect at its `revoked_at`.
- This draft does not define the format of the status that `status_ref` points to ([section 17.3](non-goals-and-open-questions.md)). In the reference implementation, the live status is an input to the check: a list of revoked identifiers with the time of each revocation.

## 5.6 Delegation

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `parent` | authority reference | No | The binding this one was delegated from. |
| `depth` | integer, at least 0 | Yes | 0 for a binding with no parent; at least 1 for one with a parent. |
| `max_depth` | integer, at least 0 | Yes | The deepest `depth` that any binding delegated from this one may have. |
| `subdelegation` | enum | Yes | `forbidden` or `attenuate`: whether this binding may be delegated further, and only by narrowing it. |

The schema requires `depth` 0 when there is no `parent`, and at least 1 when there is one. A delegated binding is valid only if every one of these holds ([section 9.10](invariants.md)):

- its parent has a `delegation` member whose `subdelegation` is `attenuate`. A parent with no `delegation` member, or with `forbidden`, allows no delegation: absent rules are not permission;
- its `depth` is its parent's `depth` plus one, and is no more than its parent's `max_depth`;
- its `max_depth` is no more than its parent's `max_depth`;
- its window lies inside its parent's: `valid_from` no earlier and `expires_at` no later;
- every one of its grants narrows a grant of the parent: the same action and resource type; the same resource `id`, or any `id` if the parent names none; a scope within the parent's; a `count` and a `cost` no higher than the parent's, present wherever the parent has them, and in the same currency; `approver_roles` that are a subset of the parent's, present wherever the parent has them; and a `min_approvals` no lower than the parent's, present wherever the parent has one;
- its `policy.sod_threshold` is no higher than its parent's and in the same currency, present wherever the parent has one.

Approvals do not pass down a delegation. Each binding's effects need approvals of that binding ([section 17.3](non-goals-and-open-questions.md) asks whether that should change). Revoking a binding revokes every binding delegated from it, directly or through other delegations. An effect under a binding is `authority_revoked` when the live status shows that binding, or any binding it was delegated from, revoked at or before the effect's start ([section 9.7](invariants.md)). An Enforcement point therefore checks the revocation status of every binding in the chain, following `delegation.parent`, at every level.

Token exchange (RFC 8693) is how a delegation hop becomes a real credential ([section 15.5](relationships-and-mappings.md)). The binding is the record of the hop; the exchanged token is what a provider enforces.

## 5.7 Policy and credential references

- `policy` names the policy, its version, and optionally the decision under which the binding was issued. It is recorded for audit. At the Assured level, decisions come from an external policy decision point, and receipts record them ([section 13.2](conformance.md)).
- `policy.sod_threshold` can tighten separation of duties for the work a binding covers, never loosen it. At the Controlled level and above, each grant of the binding whose cost limit is greater than the threshold that applies, or that has no cost limit, needs an approver other than the requester ([section 9.6](invariants.md)). The threshold that applies is the default the checking party is configured with, or the binding's own when it is lower and in the same currency; one in another currency is `currency_mismatch` at `/<i>/policy/sod_threshold` and the default applies. Without a default, every grant needs such an approver, whatever the binding declares. The grantor writes the binding and may also be the requester, so a binding that could raise the threshold could lift the control meant to check its own author. Approvals cover the whole binding, policy included ([5.3](authority-binding.md)), so a threshold changed after the approvals makes them stale.
- `credential_refs` lists where the credentials behind this authority are held. Each entry has a `kind` (`oauth_grant`, `token_exchange` or `pap_account`) and a `ref`. A `ref` is a reference, such as a path in a secret store; it is never the credential. A value shaped like a JSON Web Token is a `schema_violation`. That check is a guard against accidents, not a guarantee: a Producer MUST NOT put a token, key, password or other secret in any member of any Enna object.

## 5.8 Example

Dana Whitfield grants Harbor & Pine's coding agent what the experiment needs: reading approval comments in the Aster Foods workspace, creating a branch, committing, opening a pull request, running CI within USD 150.00, and deploying previews within USD 100.00. Priya Raman approved this binding for revision 1 of the contract, with an approval that expires after seven days. The binding allows one further level of delegation, by narrowing only.

```json
{
  "protocol_version": "0.1",
  "type": "authority_binding",
  "id": "auth_coding_agent",
  "revision": 1,
  "created_at": "2026-10-12T10:00:00Z",
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
  "grantor": {
    "organization": "harborpine.example",
    "person": {
      "kind": "person",
      "id": "dana.whitfield",
      "issuer": "https://id.harborpine.example",
      "display": "Dana Whitfield"
    }
  },
  "grantee": {
    "kind": "agent",
    "id": "https://agents.harborpine.example/coding",
    "display": "Coding agent"
  },
  "grants": [
    {
      "action": "harborpine.example/workspace_comments.read",
      "resource": {
        "type": "workspace_comments",
        "scope": {
          "organization": "harborpine.example",
          "workspace": "aster-foods"
        }
      },
      "limits": {
        "count": 50
      }
    },
    {
      "action": "repo.branch.create",
      "resource": {
        "type": "repository",
        "id": "harborpine/collab-app",
        "scope": {
          "organization": "harborpine.example",
          "workspace": "internal",
          "project": "brand-approvals"
        }
      },
      "limits": {
        "count": 1
      }
    },
    {
      "action": "repo.commit",
      "resource": {
        "type": "repository",
        "id": "harborpine/collab-app",
        "scope": {
          "organization": "harborpine.example",
          "workspace": "internal",
          "project": "brand-approvals"
        }
      },
      "limits": {
        "count": 20
      }
    },
    {
      "action": "repo.pull_request.open",
      "resource": {
        "type": "repository",
        "id": "harborpine/collab-app",
        "scope": {
          "organization": "harborpine.example",
          "workspace": "internal",
          "project": "brand-approvals"
        }
      },
      "limits": {
        "count": 1
      }
    },
    {
      "action": "ci.run",
      "resource": {
        "type": "pipeline",
        "id": "collab-app-ci",
        "scope": {
          "organization": "harborpine.example",
          "workspace": "internal",
          "project": "brand-approvals"
        }
      },
      "limits": {
        "cost": {
          "currency": "USD",
          "amount": "150.00"
        },
        "count": 10
      }
    },
    {
      "action": "preview.deploy",
      "resource": {
        "type": "preview_environment",
        "scope": {
          "organization": "harborpine.example",
          "workspace": "internal",
          "project": "brand-approvals"
        }
      },
      "limits": {
        "cost": {
          "currency": "USD",
          "amount": "100.00"
        },
        "count": 3
      }
    }
  ],
  "approvals": [
    {
      "id": "apr_priya_brand_approvals",
      "approver": {
        "kind": "person",
        "id": "priya.raman",
        "issuer": "https://id.harborpine.example",
        "display": "Priya Raman"
      },
      "approver_role": "product_manager",
      "decision": "approved",
      "decided_at": "2026-10-12T10:15:00Z",
      "expires_at": "2026-10-19T10:15:00Z",
      "covers": {
        "contract": {
          "id": "wc_brand_approvals",
          "revision": 1
        },
        "binding_digest": "sha256:63b87a1648932ceaa849aceaf420e446d0bad06918ee6d98e9c642aa0ea1e1d6"
      },
      "method": "interactive",
      "assurance": "asserted"
    }
  ],
  "policy": {
    "policy_id": "harborpine-agent-work",
    "policy_version": "1",
    "decision_ref": "harborpine-decision-2026-10-12-0042"
  },
  "valid_from": "2026-10-12T10:15:00Z",
  "expires_at": "2026-10-26T17:00:00Z",
  "revocation": {
    "status_ref": "https://id.harborpine.example/enna/status/auth_coding_agent"
  },
  "delegation": {
    "depth": 0,
    "max_depth": 1,
    "subdelegation": "attenuate"
  },
  "credential_refs": [
    {
      "kind": "oauth_grant",
      "ref": "vault://harborpine/agents/coding/git"
    }
  ]
}
```

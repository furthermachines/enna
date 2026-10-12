---
title: Work contract
description: The object that states what is to be done for an organization, the outcome it serves, who is accountable, how done is judged and within which limits.
order: 3
status: normative
---

# Work contract

A work contract (`work_contract`) states what is to be done for an organization, the business outcome it serves, who is accountable for it, how the result will be judged, and the limits it must stay within. It is revisable: every change issues a new revision, which needs approval again. Its schema is `work-contract.schema.json`.

## 3.1 Members

Besides the members every object carries ([section 2.16](conventions.md)):

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `summary` | text | Yes | The contract in plain language, complete enough to decide from on its own: what will happen, to what, at what cost, by when and under which conditions. PAP 0.1 requires the same of an operation's summary. |
| `objective` | object | Yes | `outcome`, required: the business result wanted. `request`, optional: the implementation that was asked for ([3.3](work-contract.md)). |
| `owner` | person | Yes | The person accountable for the contract. |
| `requester` | principal | Yes | Who asked for the work. |
| `stakeholders` | array | No | Each entry has a `principal` and a `role`: `approver`, `acceptor`, `consulted` or `informed`. An `acceptor` is a person. An `approver` may also have `approver_roles`, a list of distinct role tokens, at least one: the roles the contract gives that approver ([5.4](authority-binding.md)). No other stakeholder has roles. |
| `criteria` | array, at least one | Yes | The acceptance criteria ([3.2](work-contract.md)). |
| `constraints` | array | No | Each entry has an `id` (a local identifier), a `kind` (`must` or `must_not`) and a `statement`. |
| `deadline_at` | timestamp | No | When the work is due. |
| `limits` | object, at least one member | No | `cost`: money, the ceiling on what the work may cost. `risk`: `max_level` (`low`, `medium` or `high`), required, and optional `notes`. |
| `outputs` | array | No | The intended outputs. Each has an `id` (a local identifier), a `kind` (a token) and a `description`. |

Text members are non-empty strings. The object is closed ([section 2.13](conventions.md)).

## 3.2 Acceptance criteria

Each criterion says what must be true for the work to count as done, and how that is decided:

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `id` | `AC-` and a number | Yes | Stable within the contract ([section 2.3](conventions.md)). |
| `statement` | text | Yes | What must be true. |
| `required` | boolean | Yes | Whether the technical result depends on this criterion ([section 7.4](evaluation-result.md)). |
| `method` | enum | Yes | `automated_check`, `human_judgment` or `measurement_window`. |
| `measurement` | object | Exactly when `method` is `measurement_window` | `window_seconds`, an integer of at least 1, and `starts`: `on_acceptance` or `on_pilot_approval`. |

- A Producer MUST state each criterion so that its method can decide it. An automated check names something a machine can check. Human judgment names what a person is to judge; the Evaluator records that judgment as evidence. A measurement window names what is measured, over how long, and from which event.
- `measurement` is required for `measurement_window` and forbidden otherwise; either mistake is a `schema_violation`.
- A window that starts `on_acceptance` begins when the owner accepts the work. One that starts `on_pilot_approval` begins when a pilot is approved, which happens outside the contract. The evaluation records when a window actually began ([section 7.6](evaluation-result.md)).
- Only required criteria decide the technical result. A contract with no required criterion can never reach a technical `pass` ([section 7.4](evaluation-result.md)), so a Producer SHOULD mark at least one criterion as required.

## 3.3 Outcome and request

`objective.outcome` is the business result the organization wants, such as "agency clients can require their own approval before anything branded goes out". `objective.request` is the implementation someone asked for, such as "add an approval gate to external publishing, behind a feature flag". They are kept apart on purpose: the request can be the wrong path to the outcome, and a reviewer can only see that when both are written down.

Criteria and their evaluation judge whether the work was done well. Whether it achieved the outcome is business impact, which the evaluation records separately and which often stays `unknown` for a long time ([section 7.6](evaluation-result.md)).

## 3.4 What the contract enforces, and what it only records

- **Constraints** are statements, recorded for every party to read. This draft does not enforce them. An Enforcement point enforces a constraint only as far as it is expressed in an authority binding: an action no grant covers cannot happen, a workspace no grant names cannot be reached, and a limit caps use. A Producer SHOULD express every constraint that can be enforced in the authority binding as well, for example by granting nothing in a workspace the work must not touch.
- **`limits.cost`** is enforced across the whole contract: a new revision does not reset the budget. The cumulative cost of the contract's receipts, in every revision, may not pass the current revision's limit, and no grant may allow more than it. The record check sees one revision and adds up that revision's receipts ([section 9.9](invariants.md)); the Enforcement point, which sees every revision, counts the earlier ones too before each action ([section 11.1](execution-semantics.md)).
- **`limits.risk.max_level`** is recorded for approvers to weigh. This draft assigns no risk level to individual actions, so nothing checks it.
- **`deadline_at`** is recorded. A contract does not expire on its own; authority and approvals expire instead ([section 5](authority-binding.md)).
- **`outputs`** list what the work is meant to produce. A profile may define their kinds, such as `pull_request` and `preview_url` ([section 14](software-delivery-profile.md)).

## 3.5 Decision rights

- The approvers in scope for a contract are its owner and every stakeholder whose role is `approver`. For a confirmation that arrives through PAP, the requester is also in scope ([section 5.4](authority-binding.md)).
- The contract also says who holds which approver role: a grant that requires a role is met only by an approval made under that role by an approver whose `approver_roles` include it. An approval's `approver_role` is a claim; the contract's assignment is the decision. To give the owner a role, the contract lists the owner as an `approver` stakeholder with it.
- Acceptance is decided by the owner, by a person the owner names as a stakeholder whose role is `acceptor`, or by an agent the owner delegates it to through an authority binding ([section 7.5](evaluation-result.md)). A stakeholder whose role is `acceptor` and whose principal is not a person is a `schema_violation`.
- The roles are separate: an `approver` does not decide acceptance, and an `acceptor` does not approve authority. A contract that gives one person both rights lists that person once in each role.
- Stakeholders whose role is `consulted` or `informed` have no decision rights. The roles are recorded so that systems can involve the right people.

## 3.6 Revisions

Any change to a contract, however small, issues a new revision ([section 2.5](conventions.md)). The lifecycle returns to `awaiting_approval` ([section 10](lifecycle.md)), and an approval bound to an earlier revision becomes stale ([section 9.5](invariants.md)). Each revision has its own record: its authority bindings, receipts and evaluations refer to exactly that revision ([section 9.3](invariants.md)).

## 3.7 The `pap` extension

The core extension `pap` links a contract revision to an operation of the PAP 0.1 operations extension, version 1. It is defined for work contracts only, and it is never critical, so a Consumer that does not use PAP ignores it.

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `version` | `"1"` | Yes | The operations extension version this mirrors. |
| `critical` | `false` | No | If present, it is `false`. |
| `operation_id` | string, 1 to 256 characters | Yes | The PAP operation this contract revision mirrors. PAP sets no length limit; an operation whose identifier is longer than 256 characters cannot be mapped to a contract. |
| `revision` | integer, at least 1 | Yes | The PAP operation revision this contract revision mirrors. |
| `terms` | object | No | The operation's terms as PAP carries them. This draft does not define their members. |
| `expires_at` | timestamp | No | The last moment the PAP operation's revision can be confirmed, converted to UTC. |
| `user_approval_required` | boolean | No | Whether PAP requires the User's own approval of this revision, rather than a standing permission. |
| `url` | URI | No | A page at the business that shows the operation. |
| `account` | string | No | The account at the business that the PAP session was signed in to. |

- A contract revision that the person must confirm again is proposed as the next operation revision, and its `pap.revision` says which. A contract revision that changes only the business's own work keeps the operation revision the person confirmed, and the business does not propose it again; the two revision numbers then differ ([section 15.1](relationships-and-mappings.md)).
- The extension is closed: any other member is an `unknown_member`.
- [Section 15.1](relationships-and-mappings.md) describes how the members map to and from a PAP operation.

## 3.8 Example

Harbor & Pine's head of product, Dana Whitfield, owns a scoped experiment: block and record explicit client approval before a branded external project is published. She is also its requester; Priya Raman, the product manager, is the approver. The contract follows the software-delivery profile and caps the cost at USD 400.00.

```json
{
  "protocol_version": "0.1",
  "type": "work_contract",
  "id": "wc_brand_approvals",
  "revision": 1,
  "created_at": "2026-10-12T09:00:00Z",
  "scope": {
    "organization": "harborpine.example",
    "workspace": "internal",
    "project": "brand-approvals"
  },
  "profile": "software-delivery/0.1",
  "summary": "Run a scoped experiment: block and record explicit client approval before a branded external project is published, keeping client workspaces isolated. Budget USD 400.00, medium risk, due 2026-10-26.",
  "objective": {
    "outcome": "Agency clients can require their own approval before anything branded goes out, so they keep trusting shared projects.",
    "request": "Add an approval gate to external publishing for branded projects, behind a feature flag, with a preview for the owner."
  },
  "owner": {
    "kind": "person",
    "id": "dana.whitfield",
    "issuer": "https://id.harborpine.example",
    "display": "Dana Whitfield"
  },
  "requester": {
    "kind": "person",
    "id": "dana.whitfield",
    "issuer": "https://id.harborpine.example",
    "display": "Dana Whitfield"
  },
  "stakeholders": [
    {
      "principal": {
        "kind": "person",
        "id": "priya.raman",
        "issuer": "https://id.harborpine.example",
        "display": "Priya Raman"
      },
      "role": "approver",
      "approver_roles": [
        "product_manager"
      ]
    },
    {
      "principal": {
        "kind": "person",
        "id": "theo.okafor",
        "issuer": "https://id.harborpine.example",
        "display": "Theo Okafor"
      },
      "role": "consulted"
    }
  ],
  "criteria": [
    {
      "id": "AC-1",
      "statement": "Publishing a branded external project is blocked until a client approver has approved it.",
      "required": true,
      "method": "automated_check"
    },
    {
      "id": "AC-2",
      "statement": "Every approval is recorded with the approver, the time and the version approved.",
      "required": true,
      "method": "automated_check"
    },
    {
      "id": "AC-3",
      "statement": "Nothing from one client workspace is visible in another.",
      "required": true,
      "method": "automated_check"
    },
    {
      "id": "AC-4",
      "statement": "The owner reviews the preview and judges the flow usable.",
      "required": true,
      "method": "human_judgment"
    },
    {
      "id": "AC-5",
      "statement": "Pilot clients use the approval gate on at least half of their branded projects.",
      "required": false,
      "method": "measurement_window",
      "measurement": {
        "window_seconds": 2592000,
        "starts": "on_pilot_approval"
      }
    }
  ],
  "constraints": [
    {
      "id": "C-1",
      "kind": "must_not",
      "statement": "Read or write anything in the Lumen Beverages workspace."
    },
    {
      "id": "C-2",
      "kind": "must",
      "statement": "Keep the gate behind a feature flag that is off for clients outside the pilot."
    },
    {
      "id": "C-3",
      "kind": "must_not",
      "statement": "Deploy to production."
    }
  ],
  "deadline_at": "2026-10-26T17:00:00Z",
  "limits": {
    "cost": {
      "currency": "USD",
      "amount": "400.00"
    },
    "risk": {
      "max_level": "medium",
      "notes": "Preview only; nothing reaches clients without the flag."
    }
  },
  "outputs": [
    {
      "id": "OUT-1",
      "kind": "pull_request",
      "description": "A reviewed pull request with the gate."
    },
    {
      "id": "OUT-2",
      "kind": "preview_url",
      "description": "A preview the owner can try."
    }
  ]
}
```

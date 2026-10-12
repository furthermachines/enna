---
title: Relationships and mappings
description: How Enna builds on PAP 0.1 and bridges to it step by step, how it relates to MCP, A2A, OAuth, AuthZEN, in-toto, W3C PROV and OpenTelemetry, with a boundary table, a sequence in text, and each mapping marked implemented and tested or proposed.
order: 15
status: normative
---

# Relationships and mappings

Enna proposes portable business semantics and mappings to existing standards. It does not replace transports, discovery mechanisms, authentication, authorization systems or telemetry; it describes the business work that travels through them. This section says, for each standard, what Enna takes from it, where the boundary lies, and how the two map.

Every mapping carries one of two labels:

- **Implemented and tested**: the reference implementation, in the project's private build repository, implements the mapping as code in `@enna/protocol` and tests it against the version named. Only two mappings carry this label: the data mappings to and from PAP 0.1 operations, and the mapping of business states to A2A 1.0 task states.
- **Proposed**: a design for discussion. The reference implementation does not implement it, and no interoperability is claimed.

| Standard | Version (checked 2026-10-10) | Mapping | Status |
| --- | --- | --- | --- |
| Personal Agent Protocol | Draft 0.1; operations extension version 1 | Operation to and from a contract revision; confirmation to an approval; receipts to the operation's result; operation states; the discovery cross-links | Implemented and tested |
| A2A | Protocol 1.0 (releases v1.0.0 and v1.0.1) | Business states to and from task states | Implemented and tested |
| A2A | Protocol 1.0 | Enna objects as data parts under an extension URI | Proposed |
| MCP | 2026-07-28 | Enna objects in tool results; acting tools gated by authority | Proposed |
| OAuth token exchange | RFC 8693 | Delegation hops and actor chains | Proposed |
| OAuth rich authorization requests | RFC 9396 | Grants as `authorization_details` | Proposed |
| OAuth resource indicators | RFC 8707 | Audiences of the tokens behind a binding | Proposed |
| AuthZEN Authorization API | 1.0, OpenID Final Specification | The gate's decision as an evaluation request; fit to be evaluated | Proposed |
| in-toto, DSSE, SLSA; JWS | Statement v1 (framework release v1.2.0), DSSE v1.0.2, SLSA v1.2; RFC 7515 | Receipt attestations | Proposed |
| W3C PROV-O | Recommendation of 2013-04-30 | Records as provenance | Proposed |
| W3C Trace Context; OpenTelemetry GenAI conventions | Level 1 (Recommendation of 2021-11-23); GenAI conventions in Development status | Span attributes for actions, beside the `traceparent` member of receipts | Proposed |

## 15.1 The Personal Agent Protocol: the base and the bridge

Sources: the PAP 0.1 specification (personalagentprotocol.org/docs/spec) and its operations extension (personalagentprotocol.org/docs/extensions/operations), draft 0.1, pages stamped "Updated Oct 9, 2026", read on 2026-10-10. PAP publishes no schemas or code; the field shapes below come from its field tables.

### What Enna reuses unchanged

Enna references these PAP 0.1 rules normatively and does not restate them differently:

- BCP 14, with the five keywords PAP uses ([section 1.5](overview.md));
- the publication rules for well-known documents: HTTPS, HTTPS-only redirects, a document speaks only for the host requested, the `www.` rule, caching and size bounds ([section 12.2](discovery.md));
- extension naming, with plain names for core extensions and domain-prefixed names otherwise, and per-extension major versions; and PAP's `extension_required` error, which a PAP channel returns when it needs an extension that the agent did not list. Nothing Enna-specific is asked of a PAP agent ([12.5](discovery.md)), so a business does not require Enna's;
- OAuth-style error codes, and `WWW-Authenticate` challenges at the transport;
- DPoP sender-constrained tokens (RFC 9449), with Bearer tokens only at MCP ([15.2](relationships-and-mappings.md));
- client-chosen identifiers as idempotency keys: the same identifier and content replays, a different content conflicts ([section 11.2](execution-semantics.md));
- the operation semantics: revisions are immutable, an approval binds exactly one revision, effects happen at most once, nothing is reported as failed while the outcome is unknown, and a final result describes every change, partial ones included;
- HTTP 429 with `Retry-After`.

### What Enna extends

| PAP | Enna |
| --- | --- |
| The `organization` of `poppy.json` | The organization of `enna.json`, with legal name, jurisdiction and registration identifiers ([section 12.3](discovery.md)) |
| Scopes such as `poppy:read` and `poppy:write` | Structured grants with resources, scopes and limits ([section 5.2](authority-binding.md)) |
| Revocation and disconnect, learned from token errors | A revocation status that is checked before every effect ([section 5.5](authority-binding.md)) |
| An operation's `confirmation` | Approval records with approver, role, method, assurance and evidence ([section 5.3](authority-binding.md)) |
| An operation's `result` | The execution receipt ([section 6](execution-receipt.md)) |
| A message's `context` | The scoped context manifest ([section 4](context-manifest.md)) |
| Standing permissions kept privately by the agent | A visible, limited, revocable authority binding |
| Revisions issued by the company | Revisions that name their author, since business contracts are drafted by either side ([section 2.5](conventions.md)) |

### What Enna adds

Organizations as principals; accountable owners; roles and separation of duties; delegation that only narrows, with depth limits; the evaluation result; conformance classes and assurance levels; the `enna.json` discovery document; receipt integrity and attestation hooks.

### The bridge, step by step

When a person's PAP agent asks a business for something that the business carries out with Enna:

1. **Discovery.** The business publishes `poppy.json` and `enna.json`, linked both ways ([section 12.5](discovery.md)). The PAP agent may ignore the Enna entry; nothing Enna-specific is asked of it.
2. **Session and sign-in.** Unchanged PAP: DPoP-bound Session Tokens, Account Tokens, and the sign-in types PAP defines.
3. **Intake.** The request arrives on any PAP channel: a conversation, an OpenAPI call, an MCP tool or the website.
4. **Contract.** The business issues a work contract revision. Its requester is a person reference whose `id` is the pairwise User ID the agent assigned and whose `issuer` is the agent's `client_id`; the account the session is signed in to goes in the `pap` extension's `account`. The owner is on the business side. The PAP token's scopes are the outer limit: no authority derived from the request may exceed what the session's scopes allow.
5. **Proposal.** The business proposes the contract revision to the agent as the PAP operation revision that its `extensions.pap.revision` names, whose `summary` is the contract's summary. PAP requires that summary to let the User decide from it alone. A later contract revision that the person must confirm again is proposed as the next operation revision; one that changes only the business's own work keeps the operation revision the person confirmed, and is not proposed again.
6. **Confirmation.** The agent confirms that revision. The business stores the confirmation as an approval in the authority binding, with the method `pap_confirmation` and the assurance `asserted`, and the PAP record in `external`. PAP gives a confirmation no expiry, so the business sets the approval's `expires_at`, no later than the binding's, for example the operation's own `expires_at`; at the Controlled level every approval carries one ([section 9.16](invariants.md)). The confirmation counts only for grants inside the contract's own scope: a customer's word never reaches another customer's workspace ([section 5.4](authority-binding.md)). At the Controlled level and above, separation of duties applies ([section 9.6](invariants.md)): the confirmation is the requester's own approval, so every grant over the threshold, or without a cost limit, also needs a current approval in scope from a business approver, someone other than the requester. The example below carries both.
7. **Authority.** The business's agents receive authority bindings derived at the edge from the verified PAP context. A derived binding is never broader than the Session Token's scopes, and it never carries the token: a `credential_refs` entry of the kind `pap_account` names the account. PAP's guides give the same rule for internal services: verify the token at the edge, and never forward it between services.
8. **Execution.** An Enforcement point that performs an action for a PAP operation MUST set `trace.transport.pap_operation_id` to the operation's `operation_id` on every receipt of that action, and MUST NOT set it on receipts of other work. The rollup of step 10 selects exactly those receipts, by that ID, from whatever receipts it is given.
9. **Sign-out.** A confirmed PAP operation keeps running after the User signs out. Authority derived from a confirmed operation therefore survives sign-out, but an Enforcement point MUST limit it to that operation. Any other authority derived from the session ends at sign-out or disconnect.
10. **Result.** The operation's receipts, those whose `trace.transport.pap_operation_id` names its `operation_id`, roll up into its result and state. Whoever rolls them up selects them by that ID itself, so another operation's or other work's receipts never count. The operation reaches a final state only when every outcome is known.

The bridge's data mappings are implemented in `@enna/protocol` as `contractFromPapOperation`, `papOperationFromContract`, `approvalFromPapConfirmation`, `papResultFromReceipts` and `toPapOperationState`. They are tested for round trips (contract to operation to contract), against a schema of PAP's operation object written from PAP's field tables, and for the refusals below. The reference implementation has no PAP client or server, and makes no PAP requests.

### Field mappings

| PAP operation | Work contract revision |
| --- | --- |
| `operation_id` | `extensions.pap.operation_id` |
| `revision` | `extensions.pap.revision`; the contract's own `revision` may run ahead of it ([section 3.7](work-contract.md)) |
| `summary` | `summary` |
| `terms` | `extensions.pap.terms` |
| `expires_at` | `extensions.pap.expires_at`, converted to UTC with `Z` |
| `user_approval_required` | `extensions.pap.user_approval_required` |
| `url` | `extensions.pap.url` |
| `state` | The lifecycle state, by the table of [section 10.7](lifecycle.md) |
| (The session's User ID and the agent's `client_id`) | `requester`: `kind` `person`, `id` the User ID, `issuer` the `client_id` |
| (The signed-in account) | `extensions.pap.account` |

Everything else in the contract, such as the owner, objective, criteria and limits, comes from the business, since a PAP operation does not carry it.

| PAP confirmation | Approval |
| --- | --- |
| `revision` | `external.revision`; `covers.contract.revision` is the revision of the contract whose `extensions.pap.revision` it is |
| `approved_by` | `external.approved_by` |
| `confirmed_at` | `decided_at` and `external.confirmed_at`, converted to UTC |
| (The operation's `operation_id`) | `external.operation_id` |
| (The contract's requester) | `approver`, with `approver_role` `requester` unless the business names another role |
| (The binding) | `covers.binding_digest` |
| | `method` `pap_confirmation`, `assurance` `asserted`, `decision` `approved` |

The mapping from a confirmation refuses an operation that was not confirmed (`approval_missing`), a standing permission where the operation requires the User's own approval (`approval_missing`), a confirmation of another operation (`reference_unresolved`), a confirmation of another operation revision than the contract's `extensions.pap.revision` (`approval_stale`), and a contract whose requester is not a person (`approver_out_of_scope`). It sets `expires_at` when the business gives one.

The rollup's input is the operation's receipts: the receipts of the linked contract, in every revision, whose `trace.transport.pap_operation_id` is the operation's `operation_id` (step 8). It works on each action's final receipt. It groups those receipts by idempotency key, and takes each key's final receipt as the record check does ([section 9.13](invariants.md)): in creation order, the last receipt that records an effect's outcome (`succeeded`, `failed`, `cancelled` or `unknown`); for a key with none, its last replay; otherwise its last denial. A retry after a known failure carries a new key and names the failed attempt's key in `action.retry_of` ([section 11.4](execution-semantics.md)); the rollup treats such a chain as one action, decided by its last attempt. A `retry_of` that names another action or resource, or an attempt whose final outcome is not `failed`, settles nothing.

| Final receipts of the operation | PAP operation |
| --- | --- |
| None, or any final outcome `unknown` | `state` `in_progress`, `result` `null` |
| Otherwise, any final outcome `cancelled` | `state` `cancelled` |
| Otherwise, any final outcome `failed` or `denied` | `state` `failed` |
| Otherwise | `state` `succeeded` |
| Every final receipt, in the order its action was first attempted | `result.summary`: each final receipt's `outcome.summary`, one per line; `result.data`: `{ "receipts": [ { "id", "digest" } ] }` |

The rollup refuses receipts of more than one contract. A denial that a later permitted outcome under the same idempotency key supersedes is not a failure: the action was refused, then done, since a refused attempt reserves nothing and the action may be attempted again under its key ([section 11.2](execution-semantics.md)). A denial that nothing supersedes is a failure, because that action never happened. A later known outcome settles an earlier `unknown`, and a replay adds nothing.

### Example: a paid customer who cannot get in

An admin at Kestrel Media tells their personal agent that they paid but cannot get into their workspace. This example is illustrative: it shows the shapes of the mapping, not an exchange with a real PAP implementation. Harbor & Pine proposes an operation:

```jsonc
{
  // Business to agent: the proposal, as PAP's operations extension shapes it.
  "operation_id": "op_kestrel_restore",
  "revision": 1,
  "state": "proposed",
  "summary": "Restore access to the Kestrel Media workspace on the paid plan by re-running provisioning. No charge.",
  "terms": { "workspace": "kestrel-media", "charge": { "currency": "USD", "amount": "0.00" } },
  "expires_at": "2026-10-20T17:00:00Z",
  "user_approval_required": true,
  "url": "https://harborpine.example/account/operations/op_kestrel_restore",
  "confirmation": null,
  "result": null
}
```

The contract revision behind it, with the requester as PAP identifies them and the PAP-only members in the `pap` extension:

```json
{
  "protocol_version": "0.1",
  "type": "work_contract",
  "id": "wc_kestrel_access",
  "revision": 1,
  "created_at": "2026-10-18T09:00:00Z",
  "scope": {
    "organization": "harborpine.example",
    "workspace": "kestrel-media"
  },
  "summary": "Restore access to the Kestrel Media workspace on the paid plan by re-running provisioning. No charge.",
  "requester": {
    "kind": "person",
    "id": "u_8f3a2c91",
    "issuer": "https://pa.example/agent"
  },
  "owner": {
    "kind": "person",
    "id": "theo.okafor",
    "issuer": "https://id.harborpine.example",
    "display": "Theo Okafor"
  },
  "objective": {
    "outcome": "The Kestrel Media admin can use the workspace they paid for."
  },
  "criteria": [
    {
      "id": "AC-1",
      "statement": "The admin can sign in to the sandbox workspace.",
      "required": true,
      "method": "automated_check"
    }
  ],
  "limits": {
    "cost": {
      "currency": "USD",
      "amount": "0.00"
    },
    "risk": {
      "max_level": "low"
    }
  },
  "extensions": {
    "pap": {
      "version": "1",
      "operation_id": "op_kestrel_restore",
      "revision": 1,
      "terms": {
        "workspace": "kestrel-media",
        "charge": {
          "currency": "USD",
          "amount": "0.00"
        }
      },
      "expires_at": "2026-10-20T17:00:00Z",
      "user_approval_required": true,
      "url": "https://harborpine.example/account/operations/op_kestrel_restore",
      "account": "acct_kestrel_admin"
    }
  }
}
```

The agent shows the User the summary, the User approves, and the agent confirms revision 1. PAP's timestamps may carry an offset:

```jsonc
{
  // Agent to business: the confirmation of revision 1, as recorded on the operation.
  "confirmation": {
    "revision": 1,
    "approved_by": "user",
    "confirmed_at": "2026-10-18T09:30:00-05:00"
  }
}
```

The business records the confirmation as an asserted approval in the support agent's binding, with its time in UTC and an expiry, beside the owner's own approval, which the Controlled level requires. With the receipt below, the three objects are a record that holds at the Controlled level:

```json
{
  "protocol_version": "0.1",
  "type": "authority_binding",
  "id": "auth_support_kestrel",
  "revision": 1,
  "created_at": "2026-10-18T14:31:00Z",
  "scope": {
    "organization": "harborpine.example",
    "workspace": "kestrel-media"
  },
  "contract": {
    "id": "wc_kestrel_access",
    "revision": 1
  },
  "grantor": {
    "organization": "harborpine.example",
    "person": {
      "kind": "person",
      "id": "theo.okafor",
      "issuer": "https://id.harborpine.example",
      "display": "Theo Okafor"
    }
  },
  "grantee": {
    "kind": "agent",
    "id": "https://agents.harborpine.example/support",
    "display": "Support agent"
  },
  "grants": [
    {
      "action": "harborpine.example/provisioning.retry",
      "resource": {
        "type": "workspace",
        "id": "kestrel-media",
        "scope": {
          "organization": "harborpine.example",
          "workspace": "kestrel-media"
        }
      },
      "limits": {
        "count": 1
      }
    }
  ],
  "approvals": [
    {
      "id": "apr_kestrel_user",
      "approver": {
        "kind": "person",
        "id": "u_8f3a2c91",
        "issuer": "https://pa.example/agent"
      },
      "approver_role": "requester",
      "decision": "approved",
      "decided_at": "2026-10-18T14:30:00Z",
      "expires_at": "2026-10-20T17:00:00Z",
      "covers": {
        "contract": {
          "id": "wc_kestrel_access",
          "revision": 1
        },
        "binding_digest": "sha256:ebd962757c5981b5d39498cb67d4ee83d952b86b2730af8daee2605318acc68e"
      },
      "method": "pap_confirmation",
      "assurance": "asserted",
      "external": {
        "operation_id": "op_kestrel_restore",
        "revision": 1,
        "approved_by": "user",
        "confirmed_at": "2026-10-18T14:30:00Z"
      }
    },
    {
      "id": "apr_theo_kestrel",
      "approver": {
        "kind": "person",
        "id": "theo.okafor",
        "issuer": "https://id.harborpine.example",
        "display": "Theo Okafor"
      },
      "approver_role": "owner",
      "decision": "approved",
      "decided_at": "2026-10-18T14:31:00Z",
      "expires_at": "2026-10-20T17:00:00Z",
      "covers": {
        "contract": {
          "id": "wc_kestrel_access",
          "revision": 1
        },
        "binding_digest": "sha256:ebd962757c5981b5d39498cb67d4ee83d952b86b2730af8daee2605318acc68e"
      },
      "method": "interactive",
      "assurance": "asserted"
    }
  ],
  "valid_from": "2026-10-18T14:30:00Z",
  "expires_at": "2026-10-20T17:00:00Z",
  "revocation": {
    "status_ref": "https://id.harborpine.example/enna/status/auth_support_kestrel"
  },
  "credential_refs": [
    {
      "kind": "pap_account",
      "ref": "acct_kestrel_admin"
    }
  ]
}
```

The support agent re-runs provisioning, and the receipt names the PAP operation:

```json
{
  "protocol_version": "0.1",
  "type": "execution_receipt",
  "id": "rcpt_kestrel_provisioning",
  "created_at": "2026-10-18T14:32:05Z",
  "scope": {
    "organization": "harborpine.example",
    "workspace": "kestrel-media"
  },
  "contract": {
    "id": "wc_kestrel_access",
    "revision": 1
  },
  "authority": {
    "id": "auth_support_kestrel",
    "revision": 1
  },
  "action": {
    "name": "harborpine.example/provisioning.retry",
    "resource": {
      "type": "workspace",
      "id": "kestrel-media",
      "scope": {
        "organization": "harborpine.example",
        "workspace": "kestrel-media"
      }
    },
    "requested_at": "2026-10-18T14:32:00Z",
    "idempotency_key": "op_kestrel_restore-provisioning-1",
    "payload_digest": "sha256:4d1424400489128068897b38c5b09081481d259f515ca78725485fb7f201933c"
  },
  "actor": {
    "principal": {
      "kind": "agent",
      "id": "https://agents.harborpine.example/support",
      "display": "Support agent"
    },
    "performed_by": "agent",
    "runtime": {
      "name": "example-runtime",
      "version": "0.1.0"
    }
  },
  "decision": {
    "result": "permit",
    "reasons": [],
    "evaluated_at": "2026-10-18T14:32:01Z",
    "policy_ref": "harborpine-agent-work@2"
  },
  "outcome": {
    "status": "succeeded",
    "summary": "Re-ran provisioning for the Kestrel Media workspace; the sandbox is active and the admin's seat is assigned."
  },
  "started_at": "2026-10-18T14:32:02Z",
  "finished_at": "2026-10-18T14:32:05Z",
  "effects": [
    {
      "description": "Provisioning job completed for kestrel-media.",
      "external_ref": "provisioning-job:kestrel-media:2"
    }
  ],
  "trace": {
    "correlation_id": "kestrel-access",
    "transport": {
      "pap_operation_id": "op_kestrel_restore"
    }
  },
  "integrity": {
    "canonicalization": "rfc8785",
    "attestation": {
      "kind": "none"
    },
    "digest": "sha256:c4183c0fce36331897bcb53ceb3746253f3cfe5635ed3b777887918863c0add2"
  }
}
```

The receipt rolls up into the operation's final state and result:

```jsonc
{
  // Business to agent: the operation once every outcome is known (other members as before).
  "operation_id": "op_kestrel_restore",
  "revision": 1,
  "state": "succeeded",
  "result": {
    "summary": "Re-ran provisioning for the Kestrel Media workspace; the sandbox is active and the admin's seat is assigned.",
    "data": {
      "receipts": [
        {
          "id": "rcpt_kestrel_provisioning",
          "digest": "sha256:c4183c0fce36331897bcb53ceb3746253f3cfe5635ed3b777887918863c0add2"
        }
      ]
    }
  }
}
```

### Limits of the PAP mapping

- PAP is itself a draft. The mapping is pinned to PAP 0.1 and its operations extension version 1 as read on 2026-10-10, and will need review when PAP changes. PAP's pages are not versioned; the text at their URLs may change without notice. The digests in [section 18.1](references.md) identify the text this draft relies on.
- A PAP confirmation states that the agent says the User approved; PAP 0.1 says it does not prove that the User saw or approved anything. Enna therefore records it as `asserted`, and at the Assured level a confirmation alone does not authorize an effect ([section 13.2](conformance.md)).
- Enna's text is original. It adapts no PAP text, so neither of the project's repositories carries a NOTICE for PAP. It names PAP only to describe what it builds on, uses none of PAP's logo or site design, and implies no affiliation.

## 15.2 MCP

Source: the Model Context Protocol specification, revision 2026-07-28, "Current" on 2026-10-10. Status: **proposed**.

MCP supplies tool and context access; Enna describes the business contract, authority and evidence around their use.

- An MCP tool that acts SHOULD report the Enna object that records the action, such as its receipt or an envelope of objects, in the tool result's `structuredContent` under the member `enna`, with its serialized JSON also given as text content, as MCP recommends. PAP's operations extension uses the same place for its operations.
- A server whose tools act is an Enforcement point, or sits in front of one: it gates every acting `tools/call` with the checks of [section 11.1](execution-semantics.md). Tool annotations such as `readOnlyHint` and `destructiveHint` are untrusted hints, and never stand in for a grant.
- A receipt records the JSON-RPC request identifier in `trace.transport.mcp_request_id`, and its `traceparent` matches the `traceparent` key that MCP reserves in `_meta`.
- MCP's task statuses (`working`, `input_required`, `completed`, `failed`, `cancelled`, in the official tasks extension) are transport states, like A2A's. A business block is not a failed task.

Whatever this mapping's status, an Enforcement point that uses MCP or PAP keeps the authorization boundaries of [section 16.5](security.md): audience validation, no token passthrough, and DPoP and Bearer kept apart.

```jsonc
{
  // An MCP tool result from a server that gates actions (proposed shape).
  "resultType": "complete",
  "content": [
    { "type": "text", "text": "Refused before any read: no grant names the Lumen Beverages workspace." }
  ],
  "structuredContent": {
    "enna": {
      // The receipt that records the refusal; its other members are as in section 6.7.
      "protocol_version": "0.1",
      "type": "execution_receipt",
      "id": "rcpt_lumen_read_denied"
    }
  },
  "isError": true
}
```

## 15.3 A2A

Source: the Agent2Agent protocol, version 1.0 (specification release v1.0.0 of 2026-03-12 and v1.0.1 of 2026-05-28), checked 2026-10-10.

A2A supplies agent discovery and task, message and artifact exchange; Enna proposes the business meaning carried through them.

- **Task states: implemented and tested.** [Section 10.6](lifecycle.md) maps business states to A2A 1.0 task states and back, and says where they differ. `TASK_STATE_AUTH_REQUIRED` is about credentials and never stands for approval.
- **Objects in messages and artifacts: proposed.** An Enna object or envelope travels as a `Part` whose `data` is the document and whose `mediaType` is `application/json`. Contracts, manifests and bindings travel in messages; receipts and evaluations, which are the task's outputs, travel as artifacts, since A2A says outputs belong in artifacts. The parts are marked with an extension URI, proposed here as `https://businessagentprotocol.org/spec/v0.1/a2a`, which an agent declares in its card's `capabilities.extensions` and a client activates with the `A2A-Extensions` header. Following A2A's extension rules, an incompatible change would take a new URI. The URI is a proposal; nothing is published at it.
- Receipts record `trace.transport.a2a_task_id` and `a2a_context_id`.
- An agent's card is published where `enna.json` points (`agents[].transports.a2a.agent_card`). Enna does not change agent cards, and an A2A card's own signature, a JWS over its RFC 8785 canonical form, is unaffected.

```jsonc
{
  // An A2A 1.0 message carrying an Enna envelope as a data part (proposed).
  "messageId": "msg-7c1e",
  "role": "ROLE_AGENT",
  "contextId": "a2a-context-7f2",
  "extensions": ["https://businessagentprotocol.org/spec/v0.1/a2a"],
  "parts": [
    {
      // The envelope of section 8.1, shortened here.
      "data": { "protocol_version": "0.1", "type": "envelope", "id": "env_brand_approvals_handoff" },
      "mediaType": "application/json"
    }
  ]
}
```

## 15.4 OAuth resource indicators and rich authorization requests

Sources: RFC 8707 (Resource Indicators for OAuth 2.0, Proposed Standard, February 2020) and RFC 9396 (OAuth 2.0 Rich Authorization Requests, Proposed Standard, May 2023), checked 2026-10-10. Status: **proposed**. Referring to these building blocks is not a complete authorization design; it shows where Enna's records meet them.

- **Audience (RFC 8707).** A grant's resource is a business resource, not a token audience. When an Enforcement point obtains a token to reach a service, it SHOULD request it with `resource` naming that service, so the token works there and nowhere else. MCP requires its clients to send `resource`, naming the MCP server, in every authorization and token request.
- **Grants as authorization details (RFC 9396).** A grant can be requested as one `authorization_details` entry of a type this draft proposes, `https://businessagentprotocol.org/spec/v0.1/grant`: `actions` holds the action name, `identifier` the resource `id`, and the type-specific members `resource_type`, `scope`, `limits` and `conditions` hold the rest. RFC 9396 requires an authorization server to reject unknown types and unknown members, which fits Enna's closed objects, and its payment example writes amounts as strings, as Enna does.

```jsonc
{
  // One grant as an RFC 9396 authorization_details entry (proposed type).
  "authorization_details": [
    {
      "type": "https://businessagentprotocol.org/spec/v0.1/grant",
      "actions": ["ci.run"],
      "identifier": "collab-app-ci",
      "resource_type": "pipeline",
      "scope": { "organization": "harborpine.example", "workspace": "internal", "project": "brand-approvals" },
      "limits": { "cost": { "currency": "USD", "amount": "150.00" }, "count": 10 }
    }
  ]
}
```

## 15.5 OAuth token exchange

Source: RFC 8693 (OAuth 2.0 Token Exchange, Proposed Standard, January 2020), checked 2026-10-10. Status: **proposed**; required at the Assured level ([section 13.2](conformance.md)).

- Enna delegation is delegation, not impersonation: the delegate keeps its own identity and acts for the delegator. In a token exchange, the `subject_token` stands for the party on whose behalf the delegate acts and the `actor_token` for the delegate; the issued token's `act` claim names the current actor, with earlier actors nested inside it.
- Each delegated binding corresponds to one exchange: the exchange narrows the token's scope or authorization details as the child binding narrows the grants, and names the target service in `resource` or `audience`. The child binding records the issued token by reference, as a `credential_refs` entry of the kind `token_exchange`; the token itself is never in the binding.
- The token is what a provider enforces; the binding is the record. RFC 8693 says access control considers only the top-level claims and the current actor, with earlier actors for information. Where the token and the binding disagree, the narrower authority applies: a binding never widens a token.

## 15.6 AuthZEN

Source: the OpenID AuthZEN Authorization API 1.0, an OpenID Final Specification approved in January 2026, checked 2026-10-10. Status: **proposed**. Its fit is to be evaluated, not assumed.

AuthZEN defines how a policy enforcement point asks a policy decision point for a decision. At the Assured level, decisions come from an external decision point ([section 13.2](conformance.md)), and this is the interface proposed for it:

- The gate's question for one action becomes an evaluation request: `subject` is the agent (`type` `agent`, `id` its `client_id`), `action` is the action (`name`), `resource` is the action's resource (`type`, `id`, and the scope in `properties`), and `context` carries the contract and authority references, the idempotency key and the time.
- The decision, `true` or `false`, becomes the receipt's `decision.result`, and the receipt's `decision.policy_ref` identifies the decision.

Questions to settle before relying on it:

- AuthZEN requires a resource `id`; an Enna grant can cover every resource of a type, with no `id`.
- AuthZEN decisions are booleans with a free-form `context`; Enna needs catalog codes as reasons, and AuthZEN does not define where they go.
- AuthZEN's examples write times without seconds, which RFC 3339 `date-time` does not allow, so times need care at the boundary.
- Its Evaluations API, with `deny_on_first_deny`, could evaluate several checks in one call. Its working-group drafts for access requests and approvals, and for obligations, might fit approvals, but they are not final.
- Its metadata lives at `/.well-known/authzen-configuration`, which was not in IANA's registry on 2026-10-07.

```jsonc
{
  // An AuthZEN 1.0 evaluation request for one action (proposed mapping).
  "subject": { "type": "agent", "id": "https://agents.harborpine.example/coding" },
  "action": { "name": "ci.run" },
  "resource": {
    "type": "pipeline",
    "id": "collab-app-ci",
    "properties": {
      "scope": { "organization": "harborpine.example", "workspace": "internal", "project": "brand-approvals" }
    }
  },
  "context": {
    "contract": { "id": "wc_brand_approvals", "revision": 1 },
    "authority": { "id": "auth_coding_agent", "revision": 1 },
    "idempotency_key": "brand-approvals-ci-1",
    "time": "2026-10-13T11:05:01Z"
  }
}
```

## 15.7 Attestations: in-toto and JWS

Sources: the in-toto Attestation Framework v1 (repository release v1.2.0 of 2026-03-18, Statement type `https://in-toto.io/Statement/v1`), DSSE v1.0.2, SLSA v1.2 (provenance predicate `https://slsa.dev/provenance/v1`), and RFC 7515 (JWS), checked 2026-10-10. Status: **proposed**.

A receipt's `integrity.attestation` is the hook for a signature by the runtime's workload identity ([section 6.6](execution-receipt.md)):

- **in-toto (`in_toto`).** The receipt is attested by an in-toto Statement whose `subject` lists the receipt's artifacts by digest, whose `predicateType` is a type this draft proposes, `https://businessagentprotocol.org/spec/v0.1/receipt`, and whose `predicate` names the receipt and its `integrity.digest`. The Statement is signed in a DSSE envelope, and `attestation.ref` says where the envelope is. DSSE signs the payload bytes and deliberately avoids depending on canonicalization, while the receipt digest uses RFC 8785; the predicate's copy of the receipt digest ties the two together.
- **JWS (`jws`).** A JWS over the receipt's canonical form, signed with a key whose `kid` is published at the `receipts.jwks_uri` of the organization's `enna.json` ([section 12.3](discovery.md)); `attestation.ref` says where the JWS is.
- In the software-delivery profile, a Producer MAY attach SLSA provenance for the build to a `build` artifact.

This draft defines neither the exact signed payload nor a verification procedure; both are open ([section 17.3](non-goals-and-open-questions.md)). An attestation proves who signed a receipt, relative to a trusted key. Like a digest, it does not prove that the action was authorized or that the receipt is true.

```jsonc
{
  // An in-toto Statement for one receipt (proposed predicate type).
  "_type": "https://in-toto.io/Statement/v1",
  "subject": [
    { "name": "art_branch", "digest": { "sha1": "3f786850e387550fdab836ed7e6dc881de23001b" } }
  ],
  "predicateType": "https://businessagentprotocol.org/spec/v0.1/receipt",
  "predicate": {
    "receipt": {
      "id": "rcpt_branch",
      "digest": "sha256:06d91777872676cbd1519356e82a020889dbcc32b68af0823a4572234e8b7d20"
    }
  }
}
```

## 15.8 Provenance and telemetry: W3C PROV and OpenTelemetry

Sources: W3C PROV-O, a Recommendation of 2013-04-30 (PROV-JSON is only a Member Submission); W3C Trace Context Level 1, a Recommendation of 2021-11-23; and the OpenTelemetry GenAI semantic conventions, in Development status and moved to the `semantic-conventions-genai` repository, which had no release on 2026-10-10. Checked 2026-10-10.

**W3C PROV (proposed).** A record maps onto provenance terms, for audit export at the Assured level:

| Enna | PROV-O |
| --- | --- |
| Work contract | `prov:Plan` |
| Execution receipt | `prov:Activity`, with `prov:startedAtTime` and `prov:endedAtTime` |
| Artifact | `prov:Entity` that `prov:wasGeneratedBy` the receipt's activity |
| Context item | `prov:Entity` that the activity `prov:used` |
| Actor, owner, approver | `prov:Agent` (`prov:Person`, `prov:SoftwareAgent` or `prov:Organization`); the activity `prov:wasAssociatedWith` the actor, with `prov:hadPlan` the contract |
| Grantee and grantor | The grantee `prov:actedOnBehalfOf` the grantor |
| Evaluation result | `prov:Entity` generated by an evaluation activity that `prov:used` the receipts |

**OpenTelemetry (proposed).** A receipt's `trace.traceparent` member carries the W3C Trace Context of the action ([section 6.5](execution-receipt.md)); that member is part of the schema, not a mapping. The proposal: an Enforcement point emits a span for each action, beside the GenAI `execute_tool` and `invoke_agent` spans, with the attributes `enna.contract.id`, `enna.contract.revision`, `enna.receipt.id`, `enna.action.name`, `enna.decision.result` and `enna.outcome.status`, and the receipt's `traceparent` names that span. These attribute names are not registered with OpenTelemetry. Since the GenAI conventions have no release, a mapping to them would have to pin a commit.

## 15.9 Boundaries

| Concern | What Enna defines | What it leaves to others |
| --- | --- | --- |
| Discovering a business | `enna.json`, a proposed convention, linked to `poppy.json` | PAP's `poppy.json`; A2A agent cards at the registered `/.well-known/agent-card.json` |
| Carrying messages and tasks | The envelope, as data | A2A, MCP and PAP conversations and operations |
| Authenticating parties | Principal references that identify, never authenticate | OAuth 2.0 and its profiles, PAP's sessions, MCP authorization, A2A security schemes |
| Sender-constrained tokens | Nothing | DPoP (RFC 9449) for PAP; Bearer only at MCP |
| Deciding whether an action is allowed | Grants, approvals and the gate's checks, recorded in receipts | Policy decision points (AuthZEN-shaped at the Assured level) and providers' own authorization |
| Token audiences | Nothing | RFC 8707 resource indicators; MCP audience validation |
| Delegating authority | The binding chain, with attenuation and depth | RFC 8693 token exchange |
| Proving who signed a record | The attestation hook | JWS, in-toto with DSSE |
| Provenance | References between records | W3C PROV-O for export |
| Telemetry | `traceparent` in receipts | W3C Trace Context, OpenTelemetry |
| Policy languages, payments, model behavior | Nothing | The organization's chosen systems |

## 15.10 A sequence in text

The Kestrel Media request of [15.1](relationships-and-mappings.md), end to end. Each step names the standard that carries it.

| Step | From | To | What happens | Carried by |
| --- | --- | --- | --- | --- |
| 1 | Personal agent | Business | Fetches `poppy.json`; may note the link to `enna.json`. | PAP 0.1 discovery |
| 2 | Personal agent | Business | Starts a session with a DPoP-bound Session Token and sends the request in a conversation. | PAP 0.1, RFC 9449 |
| 3 | Business | Business | Issues work contract `wc_kestrel_access`, revision 1: requester the pairwise User ID at the agent's `client_id`; owner Theo Okafor. | Enna |
| 4 | Business | Personal agent | Proposes operation `op_kestrel_restore`, revision 1, with the contract's summary. | PAP operations v1 |
| 5 | Personal agent | Business | After showing the summary to the User, who approves, confirms revision 1 as approved by the user. | PAP operations v1 |
| 6 | Business | Business | Records the confirmation as an asserted approval in binding `auth_support_kestrel`, which grants the support agent one provisioning retry in the Kestrel Media workspace. | Enna |
| 7 | Business | Support agent | Hands over the contract and binding as an A2A task, which is now `TASK_STATE_WORKING`. | A2A 1.0 (data parts proposed) |
| 8 | Support agent | Enforcement point | Asks to perform `harborpine.example/provisioning.retry` with an idempotency key. The gate checks window, revocation, grant, approval, tenant, freshness, limits and key. | Enna; an AuthZEN request at the Assured level (proposed) |
| 9 | Enforcement point | Provisioning service | Calls the provisioning tool with its own credential, issued for that service only; no token is passed through. | MCP 2026-07-28, RFC 8707 |
| 10 | Enforcement point | Business | Issues receipt `rcpt_kestrel_provisioning` with the outcome `succeeded`, naming the PAP operation and the trace. | Enna; W3C Trace Context |
| 11 | Evaluator | Owner | Issues an evaluation of criterion `AC-1`, the admin can sign in, with evidence and acceptance pending. | Enna |
| 12 | Owner | Business | Accepts in a second revision of the evaluation. | Enna |
| 13 | Business | Personal agent | Reports the operation `succeeded`, with a result that describes every change and lists the receipts. | PAP operations v1 |

Read as a list: the personal agent discovers the business and signs in through PAP; the business turns the request into a contract and proposes it as a PAP operation; the User approves and the agent confirms; the business records that confirmation as an approval and hands the work to its support agent over A2A; the Enforcement point checks the support agent's request against the binding, calls the provisioning service with its own narrowly issued credential over MCP, and records a receipt; an evaluator judges the result and the owner accepts it; and the business reports the outcome back to the personal agent as the operation's final result.

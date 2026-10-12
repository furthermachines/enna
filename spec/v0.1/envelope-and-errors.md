---
title: Envelope and errors
description: The envelope that carries objects between parties, the order of validation, the error response format and the full catalog of error codes with their JSON Pointer paths.
order: 8
status: normative
---

# Envelope and errors

## 8.1 Envelope

An envelope (`envelope`) carries one or more Enna objects from one party to another. Its schema is `envelope.schema.json`.

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `protocol_version` | `"0.1"` | Yes | The version of this draft. |
| `type` | `"envelope"` | Yes | |
| `id` | identifier (`env_`) | Yes | |
| `sent_at` | timestamp | Yes | When the envelope was sent. |
| `sender` | principal reference | Yes | Who sent it. |
| `recipient` | principal reference | Yes | Who it is for. |
| `correlation` | object | No | `correlation_id` (text), required: the piece of work the envelope belongs to. `in_reply_to` (an envelope identifier), optional. |
| `objects` | array, at least one | Yes | The objects carried. Each is one of the five objects. |
| `extensions` | object | No | Extension data ([section 2.12](conventions.md)). An unsupported critical extension is `unsupported_critical_extension`. |

- The envelope is closed, like the five objects ([section 2.13](conventions.md)).
- A Consumer validates each carried object on its own, as [8.2](envelope-and-errors.md) describes, and reports its errors under `/objects/<index>`.
- An envelope assumes an open world. Its objects may refer to objects it does not carry, and it need not hold a complete record. The cross-object invariants of [section 9](invariants.md) apply when a party checks a complete record, not to each envelope.
- `sender` and `recipient` are claims. The transport authenticates the parties ([section 15](relationships-and-mappings.md)), and an object's authority comes from its records and their checks, never from the envelope that carried it.

The evaluator agent sends the pending evaluation to the owner:

```json
{
  "protocol_version": "0.1",
  "type": "envelope",
  "id": "env_brand_approvals_evaluation",
  "sent_at": "2026-10-14T10:01:00Z",
  "sender": {
    "kind": "agent",
    "id": "https://agents.harborpine.example/evaluator",
    "display": "Evaluator agent"
  },
  "recipient": {
    "kind": "person",
    "id": "dana.whitfield",
    "issuer": "https://id.harborpine.example",
    "display": "Dana Whitfield"
  },
  "correlation": {
    "correlation_id": "brand-approvals",
    "in_reply_to": "env_brand_approvals_handoff"
  },
  "objects": [
    {
      "protocol_version": "0.1",
      "type": "evaluation_result",
      "id": "eval_brand_approvals",
      "revision": 1,
      "created_at": "2026-10-14T10:00:00Z",
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
      "receipts": [
        {
          "id": "rcpt_branch",
          "digest": "sha256:06d91777872676cbd1519356e82a020889dbcc32b68af0823a4572234e8b7d20"
        },
        {
          "id": "rcpt_lumen_read_denied",
          "digest": "sha256:3e7f314d0e085ac0001978a47bd84d5c050c11e30bc9b7f2e4f88998215e0ffc"
        },
        {
          "id": "rcpt_commit",
          "digest": "sha256:dfd385f14c7096a3d4315c1d766f8d3d5be0004c8e6cc6abbbed206b62bdab56"
        },
        {
          "id": "rcpt_ci",
          "digest": "sha256:c068b494f1782da96751c9f4974d08a3d9a198d658a7ac767f4b7556b7225b39"
        },
        {
          "id": "rcpt_pull_request",
          "digest": "sha256:8ad44f7f8155d67ebf6c65a3569b6e151a5bef4c1b2a23452a090c9c79724367"
        },
        {
          "id": "rcpt_preview",
          "digest": "sha256:652127f2acc181b8739046ff9a5432071cdc215ae685eec61530a41a4fa19c4a"
        }
      ],
      "evaluator": {
        "principal": {
          "kind": "agent",
          "id": "https://agents.harborpine.example/evaluator",
          "display": "Evaluator agent"
        },
        "name": "harborpine-evaluator",
        "version": "0.1.0"
      },
      "execution": {
        "status": "complete"
      },
      "criteria": [
        {
          "criterion_id": "AC-1",
          "result": "pass",
          "method": "automated_check",
          "evidence": [
            {
              "kind": "test_report",
              "ref": "https://ci.harborpine.example/collab-app/runs/812/approval-gate.json",
              "digest": "sha256:97ab49ed582226109b4f03bf24a7a457ef7afc80e169700fe96fcd6205125965",
              "summary": "Unapproved publishing is blocked.",
              "counts": {
                "passed": 18,
                "failed": 0
              }
            }
          ]
        },
        {
          "criterion_id": "AC-2",
          "result": "pass",
          "method": "automated_check",
          "evidence": [
            {
              "kind": "test_report",
              "ref": "https://ci.harborpine.example/collab-app/runs/812/approval-records.json",
              "digest": "sha256:0d85ea6c0293a60fa7a8dd6bdad6ac999f883285179a4b1900e892872dbfc916",
              "summary": "Each approval is recorded.",
              "counts": {
                "passed": 9,
                "failed": 0
              }
            }
          ]
        },
        {
          "criterion_id": "AC-3",
          "result": "pass",
          "method": "automated_check",
          "evidence": [
            {
              "kind": "test_report",
              "ref": "https://ci.harborpine.example/collab-app/runs/812/tenant-isolation.json",
              "digest": "sha256:b154122e627f8df5df4ea965aba0ebff8ee3f3e09d906428950108300233b97a",
              "summary": "No cross-workspace reads.",
              "counts": {
                "passed": 24,
                "failed": 0,
                "skipped": 1
              }
            }
          ],
          "limitations": [
            "One cross-region case is skipped in the preview environment."
          ]
        },
        {
          "criterion_id": "AC-4",
          "result": "pass",
          "method": "human_judgment",
          "evidence": [
            {
              "kind": "review",
              "ref": "https://git.harborpine.example/collab-app/pull/318#review-1",
              "digest": "sha256:71c43f40b31947dbf52e012b1a31e991368a27bc880aafcd43dce397901b278a",
              "summary": "Dana tried the preview and found the flow usable."
            }
          ]
        },
        {
          "criterion_id": "AC-5",
          "result": "unknown",
          "method": "measurement_window",
          "evidence": [],
          "limitations": [
            "The 30-day window starts only when a pilot client approves."
          ]
        }
      ],
      "technical_result": "pass",
      "acceptance": {
        "decision": "pending"
      },
      "business_impact": {
        "status": "unknown",
        "notes": "Pilot adoption has not been measured; the window starts on pilot approval."
      },
      "limitations": [
        "A passing test suite is not evidence of customer value."
      ]
    }
  ]
}
```

## 8.2 Order of validation

A Consumer validates a single object in this order, and stops at the first step that finds an error:

1. **Decoding.** Strict UTF-8 JSON, with no byte order mark, no lone surrogate in a string or a member name and no duplicate member names (`invalid_json`, `duplicate_member`; [section 2.1](conventions.md)).
2. **Shape.** The document is a JSON object (otherwise `schema_violation` at `""`).
3. **Version.** A `protocol_version` that is a string other than `"0.1"` is `unsupported_version` at `/protocol_version`, and nothing else is reported.
4. **Type.** `type` is one of the five object types (otherwise `schema_violation` at `/type`).
5. **Schema.** The object's schema: types, formats, patterns, enumerations, required and forbidden members, and closed objects (`schema_violation`, `unknown_member`).
6. **Profile.** If the object names a profile the Consumer knows, the profile's schema (`schema_violation`).
7. **Object rules.** All of these, reported together: unsupported critical extensions (`unsupported_critical_extension`); repeated identifiers within a list (`schema_violation`); for an evaluation, a `pass` or `fail` without evidence (`evidence_missing`) and a decided acceptance in revision 1 (`acceptance_premature`); for a receipt, a time earlier than the one before it (`schema_violation` at the later time; [section 6.3](execution-receipt.md)).

An envelope is checked by decoding, shape and version first; then its own schema, its own critical extensions once its schema passes, and each carried object (steps 2 to 7) are checked, and all of their errors are reported together. A record (bundle) is decoded as one document, then checked object by object, and only when every object is valid on its own are the cross-object invariants checked ([section 9.1](invariants.md)). A discovery document is checked by decoding, shape, version and its schema, then by its critical extensions and the rules of [section 12](discovery.md), reported together.

Refusing early is deliberate: a document with the wrong version or the wrong shape is refused before any expensive or visible work is done for it.

## 8.3 Error response

An error response is a JSON object shaped like PAP's OAuth-style error bodies, with JSON Pointer details added. Its schema is `error.schema.json`.

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `error` | enum | Yes | The kind of failure: `invalid_object`, `invalid_bundle`, `denied` or `unsupported_version`. |
| `error_description` | text | No | A sentence for people to read. |
| `details` | array | Yes | Each entry has a `code` from the catalog ([8.4](envelope-and-errors.md)), a `path` and a `message`, all required. |

| `error` | Used when |
| --- | --- |
| `invalid_object` | A single object, an envelope or a discovery document failed validation. |
| `invalid_bundle` | A record failed validation as a whole, whether one of its objects is invalid on its own or the record breaks a cross-object invariant. |
| `denied` | An Enforcement point refused an action. The details are the decision's reasons, each with a path into the receipt that records the refusal. |
| `unsupported_version` | The document's `protocol_version` is not supported. The details hold that one `unsupported_version` entry. |

Rules for details:

- `path` is an RFC 6901 JSON Pointer in its JSON string form, not the URI fragment form: `""` is the whole document, and `~` and `/` inside a member name are escaped as `~0` and `~1`. It points into the document that was validated: into the object itself, under `/objects/<index>` for an envelope, and under `/<index>` for a record, where `<index>` is the object's position in the record.
- A path points at the member the problem is about: a missing member's path is where it should be, an unknown member's path is that member.
- `message` is for people. It is not stable, and a Consumer MUST NOT parse it or depend on its wording.
- A Consumer that emits an error response SHOULD report each (code, path) pair once, and SHOULD sort the details by `path`, then by `code`, comparing strings by UTF-16 code units. The reference implementation always does both, so that the same input gives the same error list everywhere. Paths sort as strings, so `/10/action` comes before `/2/grants/0`.

This draft defines no HTTP endpoints, so it maps no error to an HTTP status code. Errors of the transport stay with the transport: PAP's `WWW-Authenticate` challenges with OAuth error codes (RFC 9449 for DPoP, RFC 6750 for Bearer at MCP), MCP's own authorization errors, A2A's errors. A party that limits request rates answers HTTP 429 with `Retry-After`, as PAP 0.1 does; [section 11.4](execution-semantics.md) states how long the caller waits.

A misspelled member:

```json
{
  "error": "invalid_object",
  "error_description": "One or more objects failed validation.",
  "details": [
    {
      "code": "unknown_member",
      "path": "/objectiv",
      "message": "is not defined"
    }
  ]
}
```

A document from a later draft:

```json
{
  "error": "unsupported_version",
  "error_description": "The protocol version is not supported.",
  "details": [
    {
      "code": "unsupported_version",
      "path": "/protocol_version",
      "message": "version 0.2 is not supported"
    }
  ]
}
```

A record in which a commit landed in a client workspace that no grant names:

```json
{
  "error": "invalid_bundle",
  "error_description": "The bundle breaks a cross-object invariant.",
  "details": [
    {
      "code": "authority_insufficient",
      "path": "/5/action",
      "message": "no grant covers this action"
    },
    {
      "code": "tenant_mismatch",
      "path": "/5/action/resource/scope",
      "message": "outside the contract's tenant"
    }
  ]
}
```

An Enforcement point refusing a read in a workspace no grant names:

```json
{
  "error": "denied",
  "error_description": "The action was denied.",
  "details": [
    {
      "code": "authority_insufficient",
      "path": "/action",
      "message": "no grant covers this action"
    },
    {
      "code": "tenant_mismatch",
      "path": "/action/resource/scope",
      "message": "no grant names the lumen-beverages workspace"
    }
  ]
}
```

## 8.4 Error catalog

Detail codes are flat snake_case words from one catalog. A code is stable within 0.x and is never reused for another meaning. A later draft may add codes. The schemas list the catalog in `common.schema.json` (`error_code`), and execution receipts use the same codes as decision reasons ([section 6.3](execution-receipt.md)).

In the "Raised at" column, `<i>` is an object's index in a record, `<k>` and `<j>` are indexes within a list, and paths without `<i>` belong to a single object or document.

### Shape and versions

| Code | Meaning | Raised at |
| --- | --- | --- |
| `invalid_json` | The document is not well-formed UTF-8 JSON, or a string in it holds a lone surrogate. | `""` |
| `duplicate_member` | An object has two members with the same name. | The first member, in text order, whose name repeats an earlier one |
| `schema_violation` | A value does not match its schema: a missing member, a wrong type, format, pattern or enumeration value, a `null`, a member that must be absent, or a repeated identifier. Also a record that is not an array, that does not hold exactly one contract revision, or that holds an object twice, and a member that the record check's assurance level requires ([section 9.16](invariants.md)). | The member concerned; `/type`; `""`; `/<i>/type` for a second contract; `/<i>/id` for an object that appears twice |
| `unknown_member` | A closed object carries a member its schema does not define. | The unknown member |
| `unsupported_version` | The `protocol_version` is not one this Consumer supports. | `/protocol_version`; `/pap/poppy_json` when the linked `poppy.json` lists another Enna version ([section 12.5](discovery.md)) |
| `unsupported_profile` | The object follows a profile this Enforcement point does not know, so it cannot enforce the profile's rules ([section 2.14](conventions.md)). | `/profile` of the contract, or of the binding an action would run under |
| `timestamp_in_future` | A record states that something happened later than the current time. | The timestamp ([section 9.15](invariants.md)) |

### References and scope

| Code | Meaning | Raised at |
| --- | --- | --- |
| `reference_unresolved` | A reference names an object or link that is not present. | `/<i>/contract`, `/<i>/authority`, `/<i>/context`, `/<i>/delegation/parent`, `/<i>/receipts/<j>`; `/pap` and `/pap/poppy_json` in discovery |
| `revision_mismatch` | A reference names a revision other than the one present, or revisions that must match differ. | `/<i>/contract`, `/<i>/authority`, `/<i>/context`, `/<i>/delegation/parent` |
| `tenant_mismatch` | A scope lies outside the contract's organization and workspace, and no approved grant names that workspace. | `/<i>/scope`, `/<i>/grants/<k>/resource/scope`, `/<i>/action/resource/scope` |

### Money and limits

| Code | Meaning | Raised at |
| --- | --- | --- |
| `currency_mismatch` | Two amounts that must be compared or added are in different currencies. | `/<i>/grants/<k>/limits/cost`, `/<i>/cost`, `/<i>/policy/sod_threshold` |
| `limit_exceeded` | A cost or a count exceeds its limit. | `/<i>/grants/<k>/limits/cost`, `/<i>/cost`, `/<i>/action` |

### Approvals

| Code | Meaning | Raised at |
| --- | --- | --- |
| `approval_missing` | No valid approval covers the action. | `/<i>/authority` |
| `approval_expired` | The approval had expired when the action started. | `/<i>/authority` |
| `approval_revoked` | The approval was revoked before the action started. | `/<i>/authority` |
| `approval_stale` | The approval covers another contract revision or another version of the binding. | `/<i>/approvals/<k>/covers/contract`, `/<i>/approvals/<k>/covers/binding_digest`, `/<i>/authority` |
| `approval_denied` | An approver in scope denied the authority. | `/<i>/authority` |
| `approver_out_of_scope` | The approver may not approve this contract, or does not hold, by the contract's assignment, the role the grant requires. | `/<i>/approvals/<k>/approver`, `/<i>/authority` |
| `separation_of_duties` | At the Controlled level and above, an action over the declared threshold has no approval from someone other than the requester. | `/<i>/grants/<k>` |
| `quorum_not_met` | Fewer distinct people gave a valid approval for the action than its grant's `min_approvals` requires. | `/<i>/authority` |

### Authority

| Code | Meaning | Raised at |
| --- | --- | --- |
| `authority_expired` | The action falls outside the authority's `valid_from` to `expires_at` window. | `/<i>/started_at` |
| `authority_revoked` | The authority, or a binding it was delegated from, was revoked before the action started. | `/<i>/authority` |
| `authority_insufficient` | No grant covers the action and its resource, or the agent that acted is not the binding's grantee. | `/<i>/action`, `/<i>/actor/principal` |
| `subdelegation_forbidden` | The parent authority forbids further delegation, or the delegation is deeper than its parent allows. | `/<i>/delegation/parent`, `/<i>/delegation/depth` |
| `attenuation_violated` | Delegated authority is broader than its parent: another action, a wider resource, higher limits, fewer conditions or a lower quorum, a higher separation-of-duties threshold, a longer window or a wrong depth. | `/<i>/grants/<k>`, `/<i>/delegation/depth`, `/<i>/delegation/max_depth`, `/<i>/valid_from`, `/<i>/expires_at`, `/<i>/policy/sod_threshold` |

### Context

| Code | Meaning | Raised at |
| --- | --- | --- |
| `context_stale` | A context item was stale, or of unknown age, when an action that used it started. | `/<i>/context` |
| `context_out_of_scope` | A context item lies outside the tenant boundary, or an action cites context that neither the contract nor its own binding reaches. | `/<i>/items/<k>/scope`; `/<i>/context` of the action |

### Execution and integrity

| Code | Meaning | Raised at |
| --- | --- | --- |
| `idempotency_conflict` | An idempotency key was reused for another action, resource or payload, or an action took effect twice under one key. | `/<i>/action/name`, `/<i>/action/resource`, `/<i>/action/payload_digest`, `/<i>/action/idempotency_key` |
| `illegal_transition` | The lifecycle does not allow this event in this state. Also a refused action's reason when the contract is neither `authorized` nor `executing` ([section 11.1](execution-semantics.md)). | `""` (raised by the lifecycle, not by a document; [section 10](lifecycle.md)) |
| `digest_mismatch` | A digest does not match the content it covers. | `/<i>/integrity/digest`, `/<i>/receipts/<j>` |
| `chain_broken` | The receipt chain is missing a link, forks, or runs backwards in time. | `/<i>/integrity`, `/<i>/integrity/previous_digest` |

### Evaluation and acceptance

| Code | Meaning | Raised at |
| --- | --- | --- |
| `evidence_missing` | A required criterion is not evaluated, or a pass or a fail carries no evidence. | `/criteria/<k>/evidence`, and `/<i>/criteria/<k>/evidence` in a record; `/<i>/criteria` |
| `criterion_unknown` | The evaluation names a criterion the contract does not define. | `/<i>/criteria/<k>/criterion_id` |
| `technical_result_mismatch` | `technical_result` differs from the result derived from the required criteria. | `/<i>/technical_result` |
| `acceptance_premature` | Acceptance was decided before execution and evaluation finished (while an action's final outcome was unknown, cited or not, before an effect ended, or before the evaluation existed), for work with no effect, or in the evaluation's first revision. | `/acceptance/decision`, and `/<i>/acceptance/decision` in a record; `/<i>/acceptance` |
| `acceptance_not_owner` | Acceptance came from someone other than the contract's owner, an acceptor the contract names, or the owner's delegate. | `/<i>/acceptance/by` |

### Extensions

| Code | Meaning | Raised at |
| --- | --- | --- |
| `unsupported_critical_extension` | The object, envelope or discovery document carries a critical extension this Consumer does not support. In an authority binding every extension counts as critical. | `/extensions/<name>` |

### Discovery

| Code | Meaning | Raised at |
| --- | --- | --- |
| `domain_mismatch` | `organization.domain` is not the requested host, ignoring a leading `www.`, or a linked document names another domain. | `/organization/domain`, `/pap/poppy_json` |
| `issuer_mismatch` | The issuer metadata does not name `auth.issuer` exactly, or its `enna_domains` omits the domain. | `/auth/issuer` |

### Codes that may block a contract

A contract is blocked for a business reason, never for a transport failure ([section 10.4](lifecycle.md)). These codes are the reasons a contract may be blocked for: `approval_missing`, `approval_expired`, `approval_revoked`, `approval_stale`, `approval_denied`, `approver_out_of_scope`, `separation_of_duties`, `quorum_not_met`, `authority_expired`, `authority_revoked`, `authority_insufficient`, `subdelegation_forbidden`, `attenuation_violated`, `context_stale`, `context_out_of_scope`, `tenant_mismatch`, `limit_exceeded` and `currency_mismatch`.

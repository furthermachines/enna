---
title: Scoped context manifest
description: The object that lists, by reference, the context a contract revision may use, with purpose, provenance, freshness, classification and disclosure.
order: 4
status: normative
---

# Scoped context manifest

A scoped context manifest (`context_manifest`) lists the context that a work contract revision may use: what each item is, where it lives, who produced it, how fresh it must be, how sensitive it is, who may see it, and which parts of it the work actually needs. It refers to context; it never copies it. It is revisable. Its schema is `context-manifest.schema.json`.

A manifest makes data minimization and confidentiality visible. It does not make the context true: provenance records where something came from, which is not proof that it is correct.

## 4.1 Members

Besides the members every object carries ([section 2.16](conventions.md)):

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `contract` | contract reference | Yes | The contract revision this context serves. |
| `purpose` | text | Yes | Why the work needs this context. |
| `items` | array | Yes | The context items ([4.2](context-manifest.md)). The array may be empty. |
| `known_unknowns` | array of text | No | What the work needs to know and does not. |
| `excluded` | array | No | Context deliberately left out. Each entry has a `description` and a `reason`. |
| `residency` | text | No | Where the context may be stored and processed, for example a jurisdiction. |
| `retention_seconds` | duration | No | How long the context may be kept for this work. |

## 4.2 Items

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `id` | identifier (`item_`) | Yes | Unique within the manifest. |
| `kind` | token | Yes | What sort of context this is, such as `workspace_comments`. |
| `source` | object | Yes | `system` (text): the system that holds it. `ref` (URI): where it is. `version` (text): which version was used. All three are required. |
| `scope` | scope | Yes | Where the item belongs. |
| `classification` | enum | Yes | `public`, `internal`, `confidential` or `restricted`. |
| `provenance` | object | Yes | `produced_by` (principal): who retrieved or produced it. `retrieved_at` (timestamp). `method` (token): how, such as `api_read`. All three are required. |
| `freshness` | object | Yes | `as_of` (timestamp): when the item was true. `max_age_seconds` (duration): how old it may be when used. Both are optional; the object may be empty. |
| `disclosure` | object | Yes | `allowed_to`, required, at least one entry: who may see the item, each entry either a principal reference or a role token. `redactions`, optional: what was removed, as text. |
| `minimal` | array of text, at least one | Yes | The fields or facts of the source that the work uses. |
| `digest` | artifact digest | No | The digest of the item's content as retrieved ([section 2.15](conventions.md)). |

- `source.ref` is a reference, never a copy. A `data:` URI, in any mix of upper and lower case since URI schemes ignore case, is a `schema_violation`. A Producer MUST NOT embed the content itself anywhere in the manifest.
- `source.version` records exactly which version of the context the work used, such as a snapshot name or an entity tag, so that a later reader can ask for the same version.
- `minimal` makes data minimization explicit. An Enforcement point that hands context to agents SHOULD give the work only what `minimal` lists, and the system that serves the context should do the same.
- `digest` lets a later reader check that the content they fetch is the content the work used, as far as they trust the manifest that states it: a digest shows content is unchanged only against a digest the reader already trusts.

## 4.3 Freshness

The freshness of an item at an instant `t` is computed, not stored:

| Condition | Freshness |
| --- | --- |
| `max_age_seconds` is absent | `fresh`: no freshness requirement was stated. |
| `max_age_seconds` is present and `as_of` is absent | `unknown` |
| `as_of` is later than `t` | `unknown` |
| `t` is later than `as_of` plus `max_age_seconds` | `stale` |
| Otherwise | `fresh` |

Unknown is never fresh. An effect that cites a manifest needs every item of that manifest to be fresh when the effect starts ([section 9.8](invariants.md)); otherwise the record shows `context_stale`, and an Enforcement point refuses the action.

Because every item of the cited manifest is checked, a Producer SHOULD cite a manifest that lists only the context the action uses. When it refreshes an item, it issues a new revision of the manifest with the new `as_of`, and later receipts cite that revision ([section 11.7](execution-semantics.md)).

## 4.4 Tenant boundary

Every item's `scope` lies within the contract's tenant boundary: the contract's organization and workspace, or a scope of the same organization that an approved grant in the record names ([section 9.4](invariants.md)). An action may cite a manifest only when the contract's scope, or the grants of its own binding that have approved authority behind them, reach every item in it. Anything else is `context_out_of_scope`. A manifest therefore cannot list context from another tenant unless approved authority to reach that tenant is also on record, and an agent cannot use context that only another agent's authority reaches: context for different reaches goes in different manifests.

## 4.5 What is enforced, and where

The manifest records decisions about context; the systems that serve the context enforce them:

- **Scope and freshness** are checked against the record ([section 9](invariants.md)) and by an Enforcement point before each effect.
- **Disclosure, redactions and classification** are enforced by the system that serves the context and by the runtime that hands it to agents. At the Assured level, an Enforcement point MUST NOT give an agent context whose classification and residency the systems that serve it do not enforce ([section 13.2](conformance.md)); at the other levels they are recorded so that people and tools can see them.
- **Residency and retention** describe obligations of the systems that store the context. A JSON object cannot delete data or keep it in a region.

## 4.6 Example

The manifest for Harbor & Pine's experiment lists three items: approval-related comments from the Aster Foods client workspace, the internal feedback board, and the Aster Foods approver roles. The Lumen Beverages workspace is excluded, and the manifest says why. The two Aster Foods items lie outside the contract's own `internal` workspace; they are in scope because the coding agent's approved authority binding names the `aster-foods` workspace, and only the coding agent's actions cite this manifest ([section 5.8](authority-binding.md)).

```json
{
  "protocol_version": "0.1",
  "type": "context_manifest",
  "id": "ctx_brand_approvals",
  "revision": 1,
  "created_at": "2026-10-12T09:05:00Z",
  "scope": {
    "organization": "harborpine.example",
    "workspace": "internal",
    "project": "brand-approvals"
  },
  "contract": {
    "id": "wc_brand_approvals",
    "revision": 1
  },
  "purpose": "Ground the experiment in what clients asked for, reading no more than it needs.",
  "items": [
    {
      "id": "item_aster_comments",
      "kind": "workspace_comments",
      "source": {
        "system": "Harbor & Pine workspace comments",
        "ref": "https://app.harborpine.example/api/workspaces/aster-foods/comments?topic=approvals",
        "version": "snapshot-2026-10-12"
      },
      "scope": {
        "organization": "harborpine.example",
        "workspace": "aster-foods"
      },
      "classification": "confidential",
      "provenance": {
        "produced_by": {
          "kind": "agent",
          "id": "https://agents.harborpine.example/research",
          "display": "Research agent"
        },
        "retrieved_at": "2026-10-12T08:40:00Z",
        "method": "api_read"
      },
      "freshness": {
        "as_of": "2026-10-12T08:40:00Z",
        "max_age_seconds": 604800
      },
      "disclosure": {
        "allowed_to": [
          {
            "kind": "agent",
            "id": "https://agents.harborpine.example/planning",
            "display": "Planning agent"
          },
          {
            "kind": "agent",
            "id": "https://agents.harborpine.example/coding",
            "display": "Coding agent"
          },
          "product_team"
        ],
        "redactions": [
          "commenter names",
          "email addresses"
        ]
      },
      "minimal": [
        "comment text about approvals",
        "comment date"
      ],
      "digest": "sha256:4bc110296d2320f94ffed6f5520bcb8d3407e7fe72f57c3653f182b1e2bf4ea8"
    },
    {
      "id": "item_feedback_board",
      "kind": "feedback_items",
      "source": {
        "system": "Harbor & Pine feedback board",
        "ref": "https://app.harborpine.example/api/feedback?tag=approvals",
        "version": "2026-10-12"
      },
      "scope": {
        "organization": "harborpine.example",
        "workspace": "internal"
      },
      "classification": "internal",
      "provenance": {
        "produced_by": {
          "kind": "agent",
          "id": "https://agents.harborpine.example/research",
          "display": "Research agent"
        },
        "retrieved_at": "2026-10-12T08:45:00Z",
        "method": "api_read"
      },
      "freshness": {
        "as_of": "2026-10-12T08:45:00Z",
        "max_age_seconds": 604800
      },
      "disclosure": {
        "allowed_to": [
          "product_team"
        ]
      },
      "minimal": [
        "request titles",
        "vote counts"
      ]
    },
    {
      "id": "item_aster_approvers",
      "kind": "identity_records",
      "source": {
        "system": "Harbor & Pine identity",
        "ref": "https://id.harborpine.example/api/workspaces/aster-foods/roles/approver",
        "version": "etag-7f3a"
      },
      "scope": {
        "organization": "harborpine.example",
        "workspace": "aster-foods"
      },
      "classification": "restricted",
      "provenance": {
        "produced_by": {
          "kind": "agent",
          "id": "https://agents.harborpine.example/research",
          "display": "Research agent"
        },
        "retrieved_at": "2026-10-12T08:50:00Z",
        "method": "api_read"
      },
      "freshness": {
        "as_of": "2026-10-12T08:50:00Z",
        "max_age_seconds": 604800
      },
      "disclosure": {
        "allowed_to": [
          {
            "kind": "agent",
            "id": "https://agents.harborpine.example/coding",
            "display": "Coding agent"
          }
        ],
        "redactions": [
          "names"
        ]
      },
      "minimal": [
        "which client roles may approve"
      ]
    }
  ],
  "known_unknowns": [
    "Whether clients other than Aster Foods want the same gate.",
    "How Quillstone's brand check performs in practice."
  ],
  "excluded": [
    {
      "description": "The Lumen Beverages workspace.",
      "reason": "Not part of the pilot, and no grant covers it."
    }
  ],
  "retention_seconds": 7776000
}
```

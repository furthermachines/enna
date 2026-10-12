---
title: Discovery document
description: The proposed /.well-known/enna.json convention, its publication rules taken from PAP, its members, the issuer cross-check with enna_domains, and the two-way link with poppy.json.
order: 12
status: normative
---

# Discovery document

An organization that does agent work under this draft can say so in one public document: who it is, which of its agents accept work and through which transports, which issuer vouches for it, which approvals and delegation it supports, what it claims about its conformance, and where its PAP discovery document is. Its schema is `discovery.schema.json`.

## 12.1 Status of the path

`/.well-known/enna.json` is a convention proposed by this draft. It is **not registered with IANA**. RFC 8615 requires that a new well-known URI be registered, under the Specification Required policy with expert review; registration is a later step, to be attempted once this draft is mature enough for wide review. The registry's published guidance (its repository's README, last updated 2026-04-01, checked 2026-10-10) says that specifications hosted on single-purpose websites or repositories are not eligible unless they show significant deployment or community support, so registration is not assured.

Using this path claims no recognition, no registered namespace and no interoperability. It is distinct from the paths of other protocols: A2A 1.0 publishes agent cards at `/.well-known/agent-card.json`, which is registered; PAP 0.1 publishes `/.well-known/poppy.json`, which was not in the registry on 2026-10-10 (registry last updated 2026-10-07).

## 12.2 Publication

These rules adopt PAP 0.1's rules and guides for publishing `poppy.json`, comparing hosts without regard to ASCII case and ignoring one leading `www.` on either side, and add one about what the document may contain:

- A publisher MUST serve the document over HTTPS at `https://{domain}/.well-known/enna.json`, where the domain is its own or a subdomain it controls.
- The URL MAY redirect, for example to a hosting provider. A Consumer follows redirects only while every hop is HTTPS, and the document speaks only for the host originally requested.
- `organization.domain` MUST equal the host originally requested, compared without regard to ASCII case and ignoring one leading `www.` on either side. The host after a redirect does not count. A Consumer MUST reject a document that fails this rule, with `domain_mismatch` at `/organization/domain`.
- A Consumer SHOULD cache the document as its HTTP caching headers allow. Following PAP's guides, it SHOULD also bound the cache's lifetime, the number of redirects, the response size (PAP's guides suggest about 100 KB) and the time it waits; send no credentials when fetching; and refuse to fetch from loopback, private and link-local addresses.
- The document is public. A publisher MUST NOT put in it anything that should not be public: internal policy decision points, private systems, credentials or personal data.

## 12.3 Members

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `protocol_version` | `"0.1"` | Yes | The version of this draft. |
| `description` | text, 1 to 500 characters | No | What this document covers, in plain words: for example, which part of the organization it speaks for, or that it publishes a specification and runs no agents. It is for people and tools to read; nothing is decided from it. |
| `organization` | object | Yes | `name` and `domain` (a domain, [section 2.10](conventions.md)), required; `legal_name`, `jurisdiction` (a two-letter country code, optionally followed by a hyphen and a subdivision code of up to three characters, such as `GB` or `US-DE`) and `registration_ids` (each with a `scheme` token and a `value`), optional. |
| `auth` | object | When `agents` is present | `issuer`: the HTTPS URL of the OAuth authorization server that vouches for the organization ([12.4](discovery.md)). |
| `agents` | array, at least one | No | The organization's agents that accept work. Each has a `client_id` (HTTPS URL), a `role` (token), `accepts` (at least one profile or action namespace, as text) and `transports`, described below. |
| `conformance` | object | No | The organization's self-declared conformance ([12.6](discovery.md)). |
| `approvals` | object | No | `modes`, required: the distinct approval methods it uses (`interactive`, `standing`, `pap_confirmation`). `verifiable`, optional: whether it can produce verified approvals. |
| `delegation` | object | No | `max_depth`, required: the deepest delegation it allows. |
| `receipts` | object, at least one member | No | `jwks_uri` (HTTPS URL): where the keys that attest its receipts are published. `retention_seconds`: how long it keeps receipts. |
| `contacts` | object, at least one member | No | `security`, `privacy` and `agent_operations`, each an `https:` or `mailto:` URI. |
| `policies` | object, at least one member | No | `terms`, `privacy` and `data_processing`, each an HTTPS URL. |
| `pap` | object | No | `poppy_json`, required: the URL of the organization's PAP discovery document ([12.5](discovery.md)). |
| `extensions` | object | No | Extension data ([section 2.12](conventions.md)). A Consumer refuses a document that carries a critical extension it does not support (`unsupported_critical_extension`). |

An agent's `transports` has at least one of:

| Transport | Member | Meaning |
| --- | --- | --- |
| `a2a` | `agent_card` (HTTPS URL) | The agent's A2A agent card. |
| `mcp` | `url` (HTTPS URL) | The MCP server through which the agent accepts work. |
| `pap` | `endpoint` (HTTPS URL) | The PAP endpoint, such as a conversation endpoint, through which a person's agent reaches it. |

The document is closed: a Consumer MUST reject a member the schema does not define, at any depth, with `unknown_member`. A Consumer MUST reject an unsupported `protocol_version` before anything except decoding.

## 12.4 The issuer cross-check

A domain must not be able to claim another organization's issuer and collect what is meant for it. PAP 0.1 prevents this with a `poppy_domains` member in the issuer's metadata; this draft mirrors it with `enna_domains`:

- When `auth` is present and a Consumer relies on it, the Consumer MUST fetch the issuer's OAuth authorization server metadata (RFC 8414) and check that its `issuer` equals `auth.issuer` exactly and that its `enna_domains`, an array of domains, contains `organization.domain` exactly. Otherwise it reports `issuer_mismatch` at `/auth/issuer`, and MUST NOT trust that issuer for this domain.
- An issuer lists in `enna_domains` every domain whose `enna.json` names it. One issuer can serve both protocols, listing the same domains in `poppy_domains` and `enna_domains`.
- `enna_domains` is a metadata member proposed by this draft. It is not registered in IANA's OAuth Authorization Server Metadata registry.

```jsonc
{
  // Harbor & Pine's issuer metadata, served by the issuer under RFC 8414.
  "issuer": "https://id.harborpine.example",
  "token_endpoint": "https://id.harborpine.example/token",
  "revocation_endpoint": "https://id.harborpine.example/revoke",
  // PAP 0.1 requires this member for poppy.json; Enna adds the next one.
  "poppy_domains": ["harborpine.example"],
  "enna_domains": ["harborpine.example"]
}
```

## 12.5 The two-way link with poppy.json

An organization that serves both people's PAP agents and Enna work links the two documents both ways:

- In `poppy.json`, it lists Enna under `extensions` with the domain-named key `businessagentprotocol.org/enna`, a `version` of `"0.1"`, and a `url` pointing to `https://{domain}/.well-known/enna.json`. PAP agents that do not know this extension ignore it, as PAP's extension rules require, so nothing Enna-specific is asked of them.
- In `enna.json`, `pap.poppy_json` is the URL of its `poppy.json`.

A Consumer that has fetched both MUST check the link:

| Check | Otherwise |
| --- | --- |
| `enna.json` has a `pap` member. | `reference_unresolved` at `/pap` |
| `pap.poppy_json` equals the URL `poppy.json` was fetched from. | `reference_unresolved` at `/pap/poppy_json` |
| `poppy.json`'s `organization.domain` matches `enna.json`'s, without regard to ASCII case and ignoring one leading `www.`. | `domain_mismatch` at `/pap/poppy_json` |
| `poppy.json`'s `businessagentprotocol.org/enna` entry has a `url` that equals `https://{host}/.well-known/enna.json`, where `{host}` is the requested host without one leading `www.`, alone or after `www.`. The scheme and the host compare without regard to ASCII case, and everything else exactly, with no URL parser involved, so a port (`:443` included), userinfo, a trailing dot on the host, another path, a query or a fragment fails. | `reference_unresolved` at `/pap/poppy_json` |
| That entry's `version` is `"0.1"`. | `unsupported_version` at `/pap/poppy_json` |

PAP requires extensions it does not define to be named after a domain their definer controls. This draft uses its canonical domain, `businessagentprotocol.org`, for that purpose, and the project controls that domain (decision 5 of the repository's `GOVERNANCE.md`).

Harbor & Pine's `poppy.json`, with the link to its `enna.json`:

```jsonc
{
  "protocol_version": "0.1",
  "organization": { "name": "Harbor & Pine", "domain": "harborpine.example" },
  "auth": { "issuer": "https://id.harborpine.example" },
  "agent": {
    "protocols": [
      { "type": "poppy", "endpoint": "https://harborpine.example/poppy/conversations" }
    ]
  },
  "extensions": {
    "operations": { "version": "1", "endpoint": "https://harborpine.example/poppy/operations" },
    "businessagentprotocol.org/enna": {
      "version": "0.1",
      "url": "https://harborpine.example/.well-known/enna.json"
    }
  }
}
```

## 12.6 Conformance claims

`conformance` states what the organization claims under [section 13](conformance.md):

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `classes` | array, at least one, distinct | Yes | `producer`, `consumer`, `enforcement_point` and `evaluator`. |
| `level` | enum | Yes | `essential`, `controlled` or `assured`. |
| `profiles` | array of distinct profile identifiers | No | The profiles it implements. |
| `self_declared` | `true` | Yes | Always `true`: every claim is the organization's own. |
| `statement_url` | HTTPS URL | Yes | A statement that explains the claim, its scope and its evidence. |

Enna has no certification, no certifying body and no registry of conforming implementations. A claim in `enna.json` is the publisher's statement and nothing more. A Consumer SHOULD read the statement before relying on a claim, and MUST NOT treat a claim as proof that any particular action was authorized.

## 12.7 Order of checks

A Consumer checks a discovery document in this order: decoding ([section 2.1](conventions.md)), the version, the schema, then its critical extensions, the domain rule, then, when it has them, the issuer cross-check and the link with `poppy.json`. It stops after a failing schema check; the later checks are reported together.

## 12.8 Examples

Harbor & Pine's document, for a request to `harborpine.example` or `www.harborpine.example`. It lists a coding agent reachable over A2A and a support agent reachable through PAP, claims all four classes at the Essential level with the software-delivery profile, and links to its `poppy.json`.

```json
{
  "protocol_version": "0.1",
  "organization": {
    "name": "Harbor & Pine",
    "domain": "harborpine.example",
    "legal_name": "Harbor & Pine Software Ltd. (fictional)",
    "jurisdiction": "GB"
  },
  "auth": {
    "issuer": "https://id.harborpine.example"
  },
  "agents": [
    {
      "client_id": "https://agents.harborpine.example/coding",
      "role": "coding",
      "accepts": [
        "software-delivery/0.1"
      ],
      "transports": {
        "a2a": {
          "agent_card": "https://agents.harborpine.example/coding/agent-card.json"
        }
      }
    },
    {
      "client_id": "https://agents.harborpine.example/support",
      "role": "support",
      "accepts": [
        "harborpine.example/provisioning"
      ],
      "transports": {
        "pap": {
          "endpoint": "https://harborpine.example/poppy/conversations"
        }
      }
    }
  ],
  "conformance": {
    "classes": [
      "producer",
      "consumer",
      "enforcement_point",
      "evaluator"
    ],
    "level": "essential",
    "profiles": [
      "software-delivery/0.1"
    ],
    "self_declared": true,
    "statement_url": "https://harborpine.example/enna/conformance"
  },
  "approvals": {
    "modes": [
      "interactive",
      "pap_confirmation"
    ],
    "verifiable": false
  },
  "delegation": {
    "max_depth": 1
  },
  "receipts": {
    "retention_seconds": 31536000
  },
  "contacts": {
    "security": "mailto:security@harborpine.example"
  },
  "policies": {
    "privacy": "https://harborpine.example/privacy"
  },
  "pap": {
    "poppy_json": "https://harborpine.example/.well-known/poppy.json"
  }
}
```

The specification's own site serves a discovery document about itself. It publishes the specification, runs no agents and claims no conformance, so its document says who it is, in `description` what it covers, and in `contacts` where to report a security problem: GitHub's private vulnerability reporting on the specification's public repository.

```json
{
  "protocol_version": "0.1",
  "description": "Publishes the Enna specification, draft 0.1. Runs no agents and claims no conformance.",
  "organization": {
    "name": "Enna: the Business Agent Protocol",
    "domain": "businessagentprotocol.org"
  },
  "contacts": {
    "security": "https://github.com/furthermachines/enna/security/advisories/new"
  }
}
```

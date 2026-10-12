---
title: Security and privacy
description: The security principles of the draft, its threat model for business agent work with what Enna requires and what it leaves to others, and its rules on secrets and privacy.
order: 16
status: normative
---

# Security and privacy

## 16.1 Principles

- **JSON cannot grant authority.** An authority binding describes authority and records its approvals. Authority exists in identity providers, OAuth grants and policy decision points, and is enforced at Enforcement points and at the providers the agent touches ([section 5](authority-binding.md)).
- **Content never grants authority.** Text in a context item, a summary, a receipt, an extension or a message can inform an agent; it can never authorize an action or stand for an approval. An Enforcement point MUST decide from the checks of [section 11.1](execution-semantics.md) alone, and MUST NOT let text produced or retrieved during the work change that decision.
- **A digest shows a record is unchanged only against a digest you already trust** ([section 6.6](execution-receipt.md)). An edited record can be resealed, so a chain that checks shows only that its receipts are consistent with each other. A digest does not show authorization or an honest history.
- **Unknown never satisfies anything** ([section 2.9](conventions.md)): not a pass, not freshness, not an approval.
- **Refuse first.** Checks run before effects, and the cheapest refusals come first ([section 11.1](execution-semantics.md)).
- **Records are evidence, not prevention.** A record that shows a violation means enforcement failed ([section 11.11](execution-semantics.md)).
- **Claims are self-declared** ([section 13.4](conformance.md)).

## 16.2 Threat model

The parties are organizations, the people who own, request, approve and accept work, the agents that do it, the Enforcement points that gate it, the systems that hold context and perform effects, and, through PAP, people's personal agents. The assets are the organization's authority, its clients' data, the integrity of its records, the people's privacy, and the decisions people are accountable for.

The attacker may be an outsider, a misbehaving or manipulated agent, a compromised tool or data source, a dishonest insider, or a counterparty. Enna assumes that transports authenticate their parties, that identity providers and policy decision points work as specified, and that the Enforcement point itself is trusted by the organization that runs it. It does not defend against a compromised Enforcement point, except by making its records checkable afterwards and, at the Assured level, attributable through attestation.

| Threat | What this draft requires | What remains, and who handles it |
| --- | --- | --- |
| An agent or a model writes itself a binding or an approval | Authority comes from the Enforcement point's own state, the live status and provider credentials, never from a document an agent presents. At the Assured level, approvals are verified. | At the Essential and Controlled levels, approvals are asserted: the record is only as trustworthy as the system that issued it. |
| An old approval is reused for a changed contract or a changed binding | Approvals bind the contract revision and the binding digest, which covers the whole binding but its approvals and revision bookkeeping ([section 5.3](authority-binding.md)); a change makes them stale. Approvals can expire and be revoked by `id`. | A binding revision that only records approvals keeps earlier approvals current, by design. |
| One person asks for and approves the same work | Separation of duties at the Controlled level and above ([section 9.6](invariants.md)). | Collusion between two people; where the threshold is declared. |
| Delegated authority grows along a chain | Each hop must narrow every grant, its window and its depth ([section 5.6](authority-binding.md)); unknown delegation rules forbid delegation. Revoking a binding revokes every binding delegated from it. | The revocation of a chain is only as current as the live status the Enforcement point consults for each of its bindings. |
| An agent reaches another client's data | The tenant boundary is the organization plus the workspace; the gate refuses, and the refusal is recorded ([section 9.4](invariants.md)). | The systems that serve the data must enforce the same boundary; a record cannot. |
| A token is used where it was not meant to be, or passed through | Objects never carry tokens, only references; MCP's audience validation and its ban on token passthrough are kept; PAP-derived authority is never broader than the session's scopes ([section 15](relationships-and-mappings.md)). | Token theft is countered by DPoP and audience restriction in the transport and identity layers, not by Enna. |
| A retry, a replay or a race performs an effect twice | Idempotency keys with payload digests that the Enforcement point computes itself; durable, atomic keys at the Controlled level ([section 11.2](execution-semantics.md)). | Downstream systems that do not deduplicate; no exactly-once promise ([section 11.12](execution-semantics.md)). |
| A failure is hidden, or reported before it is known | `unknown` outcomes, reconciliation, and no `failed` while unknown ([section 11.3](execution-semantics.md)). | A producer that lies about outcomes. |
| A receipt is edited, removed or reordered | Digests over the canonical form, chained per contract revision ([section 9.14](invariants.md)). | Without a trusted anchor, whoever holds the record can rewrite all of it and the end of a chain can be cut. Attestation at the Assured level. |
| A receipt describes something that did not happen, or omits something that did | Nothing in a digest can show this. | Attestation by the runtime's workload identity, evaluation by a principal other than the executor, and audit. |
| Context misleads an agent, or carries instructions | Provenance, freshness, minimal fields and classification are recorded; provenance is not proof of truth; content never grants authority ([16.1](security.md)). | An agent can still be manipulated into proposing a harmful action; the gate limits what it can do. |
| Context is stale | Freshness is checked before each effect; unknown age is never fresh ([section 4.3](context-manifest.md)). | Sources that misstate their own `as_of`. |
| Too much data is collected or disclosed | References instead of copies, `minimal`, disclosure, retention and residency ([section 4](context-manifest.md)). | Enforcement by the systems that hold the data; required at the Assured level. |
| A domain impersonates a business in discovery | HTTPS, the domain rule, the issuer cross-check with `enna_domains`, and the two-way link with `poppy.json` ([section 12](discovery.md)). | Control of a domain is weak evidence of identity; stronger organization identity is an open question. |
| A discovery fetch is turned against the fetcher | Fetch only HTTPS, bound redirects, size and time, send no credentials, and refuse loopback, private and link-local addresses ([section 12.2](discovery.md)). | |
| An old or unknown version, or an unknown extension, slips through | Unsupported versions are refused first; critical extensions are refused when unsupported, and every extension of a binding counts as critical; objects are closed ([section 2](conventions.md)). | |
| Timestamps are manipulated | Time is an explicit input; records that describe the future are refused; windows exclude their end ([section 2.7](conventions.md)). | A producer controls its own timestamps; attestation adds a signer's statement, not a trusted clock. |
| Revocation arrives late | The live status is checked before every effect at the Controlled level and above ([section 11.1](execution-semantics.md)). | An Enforcement point that caches the status widens the window in which revoked authority still works; it SHOULD keep that window short and say how long it is in its conformance statement. |
| A huge or deeply nested document exhausts a validator | Strict decoding and early refusal; PAP's guides suggest bounding discovery documents to about 100 KB. | This draft sets no size limit for objects or arrays; a Consumer SHOULD set its own and refuse larger documents ([section 17.3](non-goals-and-open-questions.md)). |
| A conformance claim is overstated | Claims are self-declared, with a statement; no certification exists ([section 13.4](conformance.md)). | Readers must judge the statement. |

## 16.3 Secrets

- A Producer MUST NOT put a token, key, password, credential or other secret in any member of any Enna object, envelope or discovery document. `credential_refs` holds references to where credentials are kept ([section 5.7](authority-binding.md)); the schema refuses a reference shaped like a JSON Web Token, as a guard against accidents, not as a guarantee.
- Following PAP's guides, implementations keep tokens, keys, assertions and proofs out of logs, telemetry, error messages, model context and URLs.
- Examples, fixtures and demonstrations use obvious, inert placeholders and fictional organizations.

## 16.4 Privacy

- A person who arrives through PAP is identified by the pairwise User ID their agent assigned to this business, which PAP forbids deriving from personal data; Enna keeps that identifier and adds no other ([section 15.1](relationships-and-mappings.md)).
- Identifiers appear in references, logs and traces, so a Producer SHOULD NOT put personal data in them ([section 2.3](conventions.md)).
- Context is referenced, never copied, and `minimal` names what the work needs ([section 4.2](context-manifest.md)). Summaries in receipts and evaluations are read by many parties, so a Producer SHOULD write them without personal data the reader does not need.
- Retention and residency are recorded in manifests and, for receipts, in the discovery document. Keeping data no longer than stated is the duty of the systems that store it.
- A discovery document is public, so it names no private systems and carries no personal data, as [section 12.2](discovery.md) requires of its publisher.

## 16.5 Authorization boundaries at MCP and PAP

These requirements apply to every Enforcement point that uses MCP or PAP, whether or not the mappings of [section 15](relationships-and-mappings.md) are normative:

- **Audience validation.** MCP requires an MCP server to validate that a token was issued for it and to reject any other. An Enforcement point that serves MCP MUST do the same. An Enna object never carries a token, and an authority binding never stands in for this check.
- **No token passthrough.** MCP forbids an MCP server to accept or pass on tokens issued for other resources. An Enforcement point MUST NOT do either: when it calls another service, it uses its own credential for that service, recorded by reference in `credential_refs`.
- **DPoP and Bearer stay separate.** PAP binds its Session Tokens to a key with DPoP everywhere except at MCP, where it issues Bearer tokens restricted to the MCP server's URL. As read on 2026-10-10, MCP 2026-07-28's core authorization pages describe only the Bearer scheme. An Enforcement point MUST NOT accept a Bearer token where PAP requires DPoP, MUST NOT present a DPoP-bound token as a Bearer token, and MUST NOT treat the two schemes as interchangeable.

## 16.6 The reference implementation

The project's [threat model](https://businessagentprotocol.org/security/threat-model/) covers the reference implementation, its deterministic demonstration and the specification site: what they protect, what they assume, and which of the owner's decisions are still open.

---
title: Non-goals, limitations, open questions and alternatives
description: What Enna does not attempt, the known limitations of draft 0.1 and its reference implementation, the questions it leaves open, and the alternatives it considered.
order: 17
status: informative
---

# Non-goals, limitations, open questions and alternatives

## 17.1 Non-goals

Enna does not attempt to be, and does not replace:

- a transport, a message bus or an agent framework (A2A, MCP and PAP carry the work);
- an identity system or a token format (OAuth, OpenID and the transports' own mechanisms authenticate);
- a policy engine or a policy language (organizations keep theirs; [section 15.6](relationships-and-mappings.md) proposes an interface);
- a telemetry system (OpenTelemetry is one);
- a payment protocol (receipts record costs; they move no money);
- a legal contract format: a work contract describes work for agents and people, and has no legal force of its own;
- a benchmark for models or agents.

Some things matter as much as the five objects and are outside what a portable JSON object can solve: business judgment about what is worth doing, organizational accountability for decisions, reliable source data, production enforcement of identity and authority, and the measurement of real outcomes. Enna can record each of them and make it visible; it cannot supply them.

Enna does not promise exactly-once execution ([section 11.12](execution-semantics.md)), does not certify anyone ([section 13.4](conformance.md)), and in 0.1 registers nothing with IANA ([section 12.1](discovery.md)).

## 17.2 Limitations

Of the draft:

- Records describe and bind; only implementations enforce. Everything an Enforcement point does not check is a claim.
- A digest shows a record is unchanged only against a digest the reader already trusts, an edited record can be resealed, and the reference implementation's receipts are unsigned.
- The format of the evidence behind a `verified` approval, and of an attestation's signed payload, is not defined.
- Money cannot be negative, so refunds and credits cannot be recorded as amounts. Currency codes are checked for shape, not against ISO 4217 List One.
- A contract's `constraints`, `limits.risk` and `deadline_at` are recorded, not enforced ([section 3.4](work-contract.md)).
- No object holds the lifecycle state ([section 10.5](lifecycle.md)).
- Approvals do not pass down a delegation ([section 5.6](authority-binding.md)).
- Each contract revision has its own receipt chain; nothing links the chains of successive revisions.
- The record check sees one contract revision at a time, so it cannot verify that idempotency keys stay closed, or that spending stays within the contract's cost limit, across revisions ([section 11.1](execution-semantics.md), [11.2](execution-semantics.md)).
- Timestamps compare exactly, with no allowance for clock skew between parties.
- The schemas bound the length of identifiers and many strings, but not the number of items in arrays.
- There is one profile, for software delivery.
- No endpoints are defined: no way to submit an object, ask for a contract's state, or be notified when work completes.

Of the reference implementation:

- Its effects are simulated, its idempotency and revocation stores live in memory, it consults its own revocation registry rather than a live status, and it claims the Essential level only, though it applies the Controlled level's record rules by default ([section 13.6](conformance.md)).
- Its PAP mapping is data only: no PAP client, no PAP server. Its A2A mapping covers states only.
- Its scenarios are fictional, labeled as simulations, and their business impact stays `unknown`.

## 17.3 Open questions

Each question is open for discussion; none is decided by this draft.

| # | Question | Where it arises |
| --- | --- | --- |
| 1 | What compatibility rule applies after 1.0? For example, may a minor version only add members that a Consumer can ignore? | [2.14](conventions.md) |
| 2 | Should objects carry the lifecycle state, or should transitions be recorded as their own records? | [10.5](lifecycle.md) |
| 3 | Which endpoints, if any, should Enna define for submitting objects and reading state, and which code reports a conflicting re-submission of an object revision? | [2.5](conventions.md) |
| 4 | What evidence makes an approval `verified`: a JWS over the approval, an HTTP Message Signature (RFC 9421), a step-up authentication record? | [5.3](authority-binding.md) |
| 5 | What exactly does an attestation sign, and how is it verified? | [6.6](execution-receipt.md), [15.7](relationships-and-mappings.md) |
| 6 | What format does the revocation status at `status_ref` have? | [5.5](authority-binding.md) |
| 7 | When several of an agent's bindings, or several revisions of one binding, cover an action, which one does an Enforcement point use, and how does a reader of the record know the choice was right? | [5.2](authority-binding.md), [11.1](execution-semantics.md) |
| 8 | Should an approval of a parent binding authorize its narrowed children without a new approval? | [5.6](authority-binding.md) |
| 9 | How should the records of one contract's revisions be tied together, so that a later revision's record shows that every earlier one is complete? Each revision's receipts form a chain of their own. | [6.6](execution-receipt.md), [9.1](invariants.md) |
| 10 | Should there be a profile for customer operations, starting from the support scenario? | [14.6](software-delivery-profile.md) |
| 11 | How should credits and refunds be expressed, given that amounts are non-negative? | [2.8](conventions.md) |
| 12 | When, and how, should `/.well-known/enna.json` be registered with IANA, and `enna_domains` registered as authorization server metadata? | [12.1](discovery.md), [12.4](discovery.md) |
| 14 | Does AuthZEN 1.0 fit the gate's decision, given its required resource `id` and its boolean decisions? | [15.6](relationships-and-mappings.md) |
| 15 | What size limits should objects, arrays and records have? | [16.2](security.md) |
| 16 | Should time comparisons allow for clock skew between parties, and how much? | [2.7](conventions.md) |
| 17 | Who may cancel a contract: the owner, the requester, an approver? | [10.2](lifecycle.md) |
| 18 | How should an organization's identity be established more strongly than by control of a domain? | [16.2](security.md) |
| 19 | Should separation of duties apply to acceptance, so that an owner who is also the requester cannot accept the work alone? | [7.5](evaluation-result.md), [9.6](invariants.md) |
| 20 | How is a delegation from one agent to another recorded? A binding's `grantor.person` must be a person, so the delegating agent appears only through `delegation.parent`, and nothing records which person authorized the child binding beyond its approvals. | [5.6](authority-binding.md) |
| 21 | What happens to the PAP bridge when a later PAP version changes its operation semantics, given that PAP's pages are not versioned? | [15.1](relationships-and-mappings.md), [18.1](references.md) |
| 23 | Should a record check verify an Evaluator's independence beyond seeing that a limitation is stated, for example by naming the actor whose work it judged? | [7.7](evaluation-result.md), [9.16](invariants.md) |

Questions 13, on the names this draft mints under `businessagentprotocol.org`, and 22, on publishing the conformance fixtures, were answered on 2026-10-11, by decision 5 of the repository's `GOVERNANCE.md`, which confirms the project's control of that domain, and by the [conformance kit](conformance-kit.md); the other questions keep their numbers.

## 17.4 Alternatives considered

| Alternative | Why this draft chose otherwise |
| --- | --- |
| Add Enna's members to `poppy.json` instead of publishing `enna.json` | `poppy.json` describes a company to personal agents and follows PAP's versioning. A separate document, linked both ways, lets each evolve on its own while one fetcher can read both ([section 12](discovery.md)). |
| Use A2A's task states as the business lifecycle | Task states describe a transport exchange; approval, acceptance and business blocks have no task state of their own. Enna keeps its own lifecycle and maps it, saying where the match is approximate ([section 10.6](lifecycle.md)). |
| Open objects that ignore unknown members, as PAP does | A misspelled member of an authority binding must fail, not be ignored. Enna closes its objects and gives new data one place to go: extensions ([section 2.13](conventions.md)). |
| Money as JSON numbers | Binary floating point cannot hold every decimal amount, and RFC 8785 serializes numbers as doubles. Amounts are decimal strings, compared exactly ([section 2.8](conventions.md)). |
| ISO 8601 durations | Integer seconds are unambiguous and compare directly. |
| camelCase members, as A2A writes them | PAP uses snake_case, and Enna builds on PAP so that one client can speak both. |
| Approvals as separate objects | Approvals live in the binding they cover and bind a digest of it. A changed approval is a new binding revision. |
| Require signed receipts in 0.1 | Key management and verification formats need more design. 0.1 defines the attestation hook and requires it at the Assured level ([section 6.6](execution-receipt.md)). |
| Assurance levels named after company sizes | Risk decides the controls, not headcount: a small regulated company needs strong controls, and an enterprise's low-risk work does not ([section 13.3](conformance.md)). |
| Promise exactly-once execution | No one can promise it across systems that fail independently. Enna promises at most one effect per key at a conforming Enforcement point, and reconciliation for the rest ([section 11.12](execution-semantics.md)). |
| RFC 9457 problem details for errors | Enna follows PAP's OAuth-style error bodies, so a client that speaks PAP reads Enna errors the same way, and adds JSON Pointer details ([section 8.3](envelope-and-errors.md)). |
| The organization as the only tenant boundary | Multi-tenant products host many clients inside one organization; the workspace is the boundary that keeps them apart ([section 2.10](conventions.md)). |
| A W3C PROV or JSON-LD serialization as the native format | Plain JSON with closed schemas is simpler to validate and implement; PROV is a mapping for export ([section 15.8](relationships-and-mappings.md)). |

The project's decision records, in the project's private build repository, record these and the other design decisions, with their context and consequences.

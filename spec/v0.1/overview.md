---
title: Overview, status and terms
description: What Enna is and is not, the status of this draft, how to read its requirements, and the terms it uses.
order: 1
status: normative
---

# Overview, status and terms

## 1.1 What Enna is

Enna, the Business Agent Protocol (BAP), is a draft, vendor-neutral standard for agent work done for organizations. It defines five portable objects that travel with a piece of business work:

| Object | What it records | Section |
| --- | --- | --- |
| Work contract | What is to be done and why, who is accountable, how done is judged, and within which limits | [3](work-contract.md) |
| Scoped context manifest | Which context the work may use, by reference, with its purpose, provenance, freshness and who may see it | [4](context-manifest.md) |
| Authority binding | Which actions an agent may take on which resources, within which limits, approved by whom and until when | [5](authority-binding.md) |
| Execution receipt | What was proposed or done, under which authority and context, the decision, the outcome and every change made | [6](execution-receipt.md) |
| Evaluation result | Whether each acceptance criterion passed, on what evidence, and whether the accountable owner accepted the outcome | [7](evaluation-result.md) |

Around these objects the draft defines shared conventions ([section 2](conventions.md)), an envelope and an error format ([section 8](envelope-and-errors.md)), cross-object invariants ([section 9](invariants.md)), a business lifecycle ([section 10](lifecycle.md)), execution semantics for retries, failures and recovery ([section 11](execution-semantics.md)), a discovery document ([section 12](discovery.md)), conformance classes and assurance levels ([section 13](conformance.md)), one domain profile for software delivery ([section 14](software-delivery-profile.md)), and mappings to the standards it builds on ([section 15](relationships-and-mappings.md)).

Enna is not a transport, an identity system, a policy engine, a telemetry system or a token format. It describes business meaning that travels through those systems; [section 17](non-goals-and-open-questions.md) lists what it does not attempt.

## 1.2 The hypothesis

When agents work for an organization, the work is easier to hand off, inspect, evaluate and accept if five things travel with it in a portable form: the intent, the context, the authority, the evidence of what was done, and the accountable outcome. This draft makes that hypothesis testable. It does not claim that the five objects already make agent work portable across tools and runtimes; that is what implementations of this draft are meant to find out.

A two-person company and a global enterprise need the same five things. What differs is how strongly each is controlled, and that depends on the risk of the work more than on the size of the organization. [Section 13](conformance.md) defines three assurance levels chosen by risk.

## 1.3 Relationship to the Personal Agent Protocol

The Personal Agent Protocol (PAP) draft 0.1 (personalagentprotocol.org, checked 2026-10-10) covers one person's agent working with one company. Enna covers the work that organizations' agents do, inside a business and between businesses, and the point where a person's PAP agent hands work to a business.

Enna builds on PAP 0.1. Where PAP has a convention, Enna adopts it by normative reference instead of inventing another, so that one client can speak both. Where businesses need more, Enna extends PAP's ideas: organizations as principals, accountable owners, roles and separation of duties, delegation that can only narrow, evaluation and acceptance, and records that can be checked. [Section 15.1](relationships-and-mappings.md) describes the bridge step by step.

This draft is written independently of PAP's authors and implies no affiliation with them or endorsement by them. It names PAP only to say what it builds on. "PAP" is this draft's own abbreviation: PAP's site does not use it, and calls the protocol by its short name, Poppy. The abbreviation appears in identifiers this draft defines (the `pap` extension, `pap_confirmation`, `pap_account`, `pap_operation_id` and the discovery member `pap`); whether to keep it is part of a counsel review that the repository's `GOVERNANCE.md` lists as an open decision.

## 1.4 Status of this document

- This is draft 0.1, dated 2026-10-10, revised 2026-10-11: a proposal for discussion and implementation experiments. Any part of it may change before version 1.0, including in incompatible ways.
- The draft is founder-led. Further Machines started it and intends to implement it. The protocol is vendor-neutral: implementing it needs nothing from Further Machines, and no product is required to use it. The repository's `GOVERNANCE.md` describes how decisions are made.
- No standards body has reviewed or ratified it. The discovery document's path is not registered with IANA. Nothing is certified against it, and no adoption is claimed. The only implementation this draft knows of is the reference implementation, in the project's private build repository.
- Since 2026-10-11 this text has been licensed under CC BY 4.0, and the schemas, the fixtures and the code under the Apache License 2.0; see `LICENSING.md` in the repository. The names Enna, Business Agent Protocol and BAP are not licensed.

### Document history

| Version | Date | Change |
| --- | --- | --- |
| 0.1 | 2026-10-10 | First draft. The repository's `CHANGELOG.md` lists its contents. |
| 0.1 | 2026-10-11 | Revised under the same version and addresses. Normative changes, by section: 2.1, a byte order mark and a lone surrogate are `invalid_json`, and decoding reports only the first repeated member; 2.2, 2.4, 2.12 and 2.14, a domain prefix follows the label rule of 2.10; 2.7, a leap second is the same instant as the next day's midnight; 2.10, the syntax of a domain and where it applies; 2.13, each object's own `properties` names every member it can carry; 2.14, `format` is an assertion with the rules stated, `uri` is RFC 3986's `URI` production, and patterns keep to a portable subset in which `$` matches only at the end; 2.15, every integer is at most 2^53 - 1; 8.2, decoding refuses a byte order mark and a lone surrogate, a receipt's times run forward among the object rules, and a record decodes as one document; 8.4, `invalid_json` covers a lone surrogate, `duplicate_member` is raised at the first repeated member, and `currency_mismatch` also at `/<i>/policy/sod_threshold`; 9.1, a record decodes as one document, and each invariant applies on its own; 9.4, an effect whose binding is not in the record reaches the contract's scope alone; 9.7, a PAP confirmation outside the contract's scope leaves `approval_missing`; 9.14, the chain follows creation order; 12.2 and 12.5, hosts compare without regard to ASCII case, `poppy.json`'s link is a string rule, and the project controls `businessagentprotocol.org`; 13.1, the fixtures are published with the conformance kit; 17.3, questions 13 and 22 are answered; 18, new references; and checklist item C-16. The repository's `CHANGELOG.md` lists every change. |

The table lists normative changes. On 2026-10-12, with no normative change, the example of the specification site's own discovery document in [section 12.8](discovery.md) gained `contacts.security`, the address for security reports that the owner chose that day, and so did the conformance kit's valid fixture `/spec/v0.1/fixtures/valid/discovery-spec-site.json`, which is that document, so its SHA-256 in the manifest changed. The same day, also with no normative change, sections 1.4, 1.5, 2, 11.12, 13.6, 15, 16.6, 17.4 and 18.2 stopped pointing to files of the project's private build repository: they name the schemas by their address on the site, the threat model by its page, and the reference implementation, its tests, the decision records and the research notes without paths.

## 1.5 How to read the requirements

The words MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are used as BCP 14 (RFC 2119 and RFC 8174) describes, and only when they appear in capitals. These are the five keywords PAP 0.1 uses; this draft uses no others.

Every requirement has an explicit scope. It names the party it binds, which is one of the conformance classes defined in [section 13](conformance.md) (a Producer, a Consumer, an Enforcement point or an Evaluator), a publisher of a discovery document, or a definer: anyone who defines an action name, an extension or a profile. Requirements on the reference implementation are stated as such in [section 13.6](conformance.md).

A file whose front matter says `status: informative` contains no requirements. Examples, notes and the rationale paragraphs in normative sections are informative.

JSON examples use Harbor & Pine, a fictional software company at `harborpine.example`, and other fictional names under the reserved `.example` top-level domain. A fenced `json` example is one complete document (an object, an envelope, a discovery document or an error response), and the reference implementation's test suite, in the project's private build repository, validates every one against the schemas. A fenced `jsonc` example is a partial document, or a document of another protocol, and may carry `//` comments; the tests check only that it parses.

## 1.6 Terms

| Term | Meaning |
| --- | --- |
| Organization | A business or other body that owns work and grants authority. It is identified by a domain it controls. |
| Workspace | A tenant area inside an organization, for example one client's area in a shared multi-tenant product. The organization plus the workspace is the tenant boundary. |
| Project | A grouping of work inside an organization or workspace. A project is not a tenant boundary. |
| Principal | A person, agent, service or organization, named by a principal reference ([section 2.11](conventions.md)). |
| Agent | Software that acts for an organization or a person. An agent is identified by its `client_id`, an HTTPS URL, as PAP identifies personal agents. |
| Owner | The person accountable for a work contract. The owner may approve the contract within its limits and decides acceptance, unless an acceptor or a delegate decides it. |
| Requester | The principal that asked for the work. |
| Stakeholder | A principal with a role in a contract: `approver`, `acceptor`, `consulted` or `informed`. |
| Approver in scope | A person who may approve or deny authority for a contract: its owner, a stakeholder whose role is `approver`, or, for a confirmation that arrives through PAP, its requester ([section 5.4](authority-binding.md)). |
| Acceptor | A person the owner names in a contract, as a stakeholder whose role is `acceptor`, to decide acceptance ([section 7.5](evaluation-result.md)). |
| Grantor | The organization, and the person acting for it, that grants authority. |
| Grantee | The agent that receives authority. |
| Grant | One permitted action on one kind of resource, with optional limits and conditions. |
| Approval | A recorded decision, `approved` or `denied`, by an approver about the grants of one authority binding for one contract revision. |
| Action | A named operation on a resource, such as `repo.commit`, that an agent proposes and an Enforcement point permits or denies. |
| Effect | A permitted action that is not a replay. An effect may change the world. |
| Producer | An implementation that creates Enna objects. |
| Consumer | An implementation that reads and validates Enna documents and acts on their meaning. |
| Enforcement point | A runtime that decides, before each action, whether it may proceed under the authority it holds, performs or forwards permitted actions, and issues receipts. |
| Evaluator | An implementation that judges a contract's criteria and issues evaluation results. |
| Record | The complete set of objects for one revision of a work contract: the contract, its context manifests, authority bindings, receipts and evaluation results. A record checked as a whole is also called a bundle. |
| Idempotency key | A value that identifies one logical action across retries ([section 11](execution-semantics.md)). |
| Payload digest | The digest of the canonical form of an action's request payload. |
| Trusted anchor | A digest or signature the verifier obtained from a source it trusts independently of the record being checked, such as a counterparty's stored copy or a signed attestation. |
| Profile | A named, versioned set of additional rules and vocabulary for one domain of work ([section 14](software-delivery-profile.md)). |
| Extension | Data under an `extensions` member, defined by this draft or by a party that controls a domain ([section 2.12](conventions.md)). |
| User, Personal Agent, Company, operation, confirmation | As PAP 0.1 defines them. |

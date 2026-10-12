---
title: Conformance
description: The four conformance classes, the Essential, Controlled and Assured assurance levels chosen by risk rather than headcount, self-declared claims, and what the reference implementation claims.
order: 13
status: normative
---

# Conformance

Conformance has two dimensions. A **class** says what an implementation is: it produces objects, consumes them, enforces authority, or evaluates outcomes. An **assurance level** says how strong its controls are. A claim names one or more classes, one level, and the profiles implemented.

## 13.1 Classes

| Class | What it does | Its requirements |
| --- | --- | --- |
| Producer | Creates Enna objects. | Every object it emits is valid on its own ([sections 2 to 8](conventions.md)). Receipts carry correct digests and chain ([section 6.6](execution-receipt.md)). It reports `unknown`, never `failed`, while an outcome is not known. |
| Consumer | Reads and validates Enna documents and acts on their meaning. | It validates in the order of [section 8.2](envelope-and-errors.md); rejects unsupported versions, unknown members and unsupported critical extensions; never treats `unknown` as satisfying anything ([section 2.9](conventions.md)); never treats a digest or a claim as proof of authorization; and, when it verifies a record, applies every rule of [section 9](invariants.md). |
| Enforcement point | Gates and performs actions under authority, and issues receipts. | It is a Producer of receipts. It applies the gate of [section 11.1](execution-semantics.md) before every action and refuses before any side effect, keeps the idempotency rules of [section 11.2](execution-semantics.md) and the unknown-outcome rules of [section 11.3](execution-semantics.md), and follows the lifecycle of [section 10](lifecycle.md). |
| Evaluator | Judges criteria and issues evaluation results. | It is a Producer of evaluation results. It derives the technical result ([section 7.4](evaluation-result.md)), backs every `pass` and `fail` with evidence, keeps execution, technical result, acceptance and business impact apart, and records acceptance only as a later revision, after execution, from the owner, an acceptor the contract names, or the owner's delegate ([section 7.5](evaluation-result.md)). |

**Conformance fixtures.** The conformance fixtures define expected outcomes, and are published with this draft under `/spec/v0.1/fixtures/`: `valid/` holds documents that pass, with the check and the inputs to validate each under (`valid/index.json`), and `invalid/` holds cases, each built from a valid document or given as exact text or bytes, with the exact list of (code, path) pairs expected. The [conformance kit](conformance-kit.md) defines the case format and how to run the cases, and lists them with the test vectors, the catalogs and the manifest published beside them. A Consumer that claims conformance MUST accept every valid fixture, and reject every invalid one, of the kinds it validates (objects, envelopes, records and discovery documents), reporting at least every expected (code, path) pair; it SHOULD report exactly the expected list.

## 13.2 Assurance levels

The levels are cumulative: Controlled includes everything Essential requires, and Assured everything Controlled requires. Every class requirement of [13.1](conformance.md) applies at every level.

### Essential

Typical for startups and for low-risk work anywhere.

- A claim at this level covers a deployment that acts for a single organization.
- An Enforcement point MAY let the owner approve their own contract within its limits ([section 9.6](invariants.md)).
- An Enforcement point MAY check authority, approvals and revocations against its own state.
- A Producer MUST give its receipts digests and chain them ([section 6.6](execution-receipt.md)).
- An Evaluator MUST record the method of each criterion ([section 7.2](evaluation-result.md)).

### Controlled

Typical for growing businesses, and for any work where one person should not both ask and approve. A record check at this level verifies the parts of it that a record can show ([section 9.16](invariants.md)); the rest is a property of the systems behind the record.

- An Enforcement point MUST apply separation of duties above a declared cost threshold ([section 9.6](invariants.md)). Its conformance statement MUST state the threshold it applies, or that it applies separation of duties to every grant; a binding's `policy.sod_threshold` can only lower it ([section 5.7](authority-binding.md)).
- A Producer MUST give every approval an `expires_at`, and every approval binds the contract revision and the binding digest ([section 5.3](authority-binding.md)). A record check reports an approval without one ([section 9.16](invariants.md)).
- An Enforcement point MUST consult the live revocation status at `revocation.status_ref` before every effect ([section 11.1](execution-semantics.md)).
- An Enforcement point MUST enforce tenant and workspace boundaries before every effect ([section 9.4](invariants.md)).
- An Enforcement point MUST enforce cost limits with exact decimal arithmetic ([section 2.8](conventions.md)).
- An Enforcement point MUST store idempotency keys durably and check and reserve them atomically, and its conformance statement MUST document how it recovers after a crash ([section 11.12](execution-semantics.md)).
- For a required `automated_check` criterion, the Evaluator SHOULD be a principal other than the one that performed the actions; when it is the same, the evaluation MUST say so in that criterion's `limitations` ([section 7.7](evaluation-result.md)). A record check verifies that a limitation is stated ([section 9.16](invariants.md)).

### Assured

Typical for enterprises and for regulated work, whatever the size of the organization.

- An Enforcement point MUST take its policy decisions from an external policy decision point, through an interface shaped like an AuthZEN 1.0 evaluation request (subject, action, resource, context), and a Producer MUST identify the decision in each receipt's `decision.policy_ref` ([section 15.6](relationships-and-mappings.md)). A record check verifies that every receipt names a decision; whether the interface has the proposed shape cannot be tested until section 15.6 is normative.
- Each delegation hop MUST be backed by an OAuth token exchange (RFC 8693) that narrows the token as the binding narrows the grants, and the Producer of the delegated binding MUST record it in `credential_refs` with the kind `token_exchange` ([section 15.5](relationships-and-mappings.md)). An Enforcement point MUST check every binding in a delegation chain; at every level, the revocation of any of them revokes the bindings delegated from it ([section 5.6](authority-binding.md)).
- Where policy requires several approvers, the Producer of a binding MUST state in `conditions.min_approvals` how many each grant it covers needs, so that the Enforcement point and the record check both hold every effect to the quorum ([section 5.4](authority-binding.md)). Because the next requirement applies to every approval relied on, each approval that counts toward a quorum is `verified`.
- An Enforcement point MUST rely only on approvals whose assurance is `verified`, with evidence. A PAP confirmation is always `asserted` ([section 5.3](authority-binding.md)), so at this level a PAP confirmation alone does not authorize an effect; the business also needs a verified approval, for example one the person gives on the business's own site while signed in, which PAP 0.1 itself suggests where proof is needed.
- A Producer MUST attest its receipts with the runtime's workload identity: `integrity.attestation.kind` is `jws` or `in_toto`, never `none` ([section 6.6](execution-receipt.md)).
- An Enforcement point MUST NOT give an agent context whose classification and residency the systems that serve it do not enforce ([section 4.5](context-manifest.md)).
- An Enforcement point MUST export its records for audit, to OpenTelemetry and in W3C PROV terms ([section 15.8](relationships-and-mappings.md)), and keep them with defined retention and legal hold. The export's shape cannot be tested until section 15.8 is normative.

## 13.3 Choosing a level by risk, not headcount

The level follows the risk of the work, not the size of the organization:

| Typical fit | Level | Why |
| --- | --- | --- |
| A startup, or low-risk internal work in a company of any size | Essential | One accountable owner, local enforcement and receipts with digests are proportionate. An enterprise's low-risk internal drafting can run here. |
| A growing business, or work where one person should not both ask and approve | Controlled | Separation of duties, expiring approvals, live revocation, tenant boundaries and durable idempotency become necessary once several people, clients or tenants share the work. |
| An enterprise, or regulated work at any size | Assured | External policy decisions, verified approvals, attested receipts and enforced residency are what auditors and regulators need. A two-person company handling regulated health data needs this level for that work. |

An organization may run different work at different levels. The `level` in its discovery document is the level it meets for all the work its listed agents accept ([section 12.6](discovery.md)); work held to a higher level is described in its conformance statement.

## 13.4 Claims are self-declared

- Every claim is the claimant's own statement, published in `enna.json` with `self_declared: true` and a `statement_url` ([section 12.6](discovery.md)). The statement SHOULD say which classes, level and profiles are claimed, for which systems, on what evidence (for example, the conformance fixtures passed), and with which known gaps.
- There is no certification, no certifying body, no registry of conforming implementations and no badge. A claimant MUST NOT describe a claim as certified, endorsed or approved by this project.
- A claim states what the claimant built. It is not evidence that any particular action was authorized, and a Consumer MUST NOT treat it as such.

## 13.5 Conformance checklist

The [conformance checklist](conformance-checklist.md) lists every requirement of this draft by class and level, with the section that states it, for implementers to assess themselves against.

## 13.6 What the reference implementation claims

The reference implementation, in the project's private build repository, claims the **Producer, Consumer, Enforcement point and Evaluator** classes at the **Essential** level, with the profile `software-delivery/0.1`, for its own draft contract. To make that claim, it MUST:

1. emit only objects that pass `validateObject`, and records that pass `validateBundle` at the level it runs at, `controlled` by default. The one exception is a record that the `tampered_receipt` fault corrupts on purpose, after sealing, to show that verification catches it;
2. accept every valid conformance fixture and reject every invalid one with exactly the expected (code, path) list (its conformance tests);
3. validate every `json` example in this specification (its tests of the specification's examples);
4. implement the lifecycle as data and test every legal and illegal transition;
5. test the PAP 0.1 data mappings and the A2A 1.0 state mapping ([section 15](relationships-and-mappings.md));
6. refuse unauthorized actions before any simulated side effect, and show in its tests that a refused action leaves no effect;
7. issue receipts with attestation `none` and label them as unsigned wherever it shows them, as its command line does;
8. keep time and randomness injected, so that the same inputs produce byte-identical outputs.

By default its gate and its record checks apply the Controlled level's record rules ([section 9.16](invariants.md)): separation of duties, declared thresholds and quorums, and expiring approvals. It consults its own revocation registry before every effect, and it counts spending across a contract's revisions. It does **not** meet the Controlled level: its idempotency and revocation stores live in memory, and it consults its own registry rather than a live status at `revocation.status_ref`. It does not claim the Assured level, and cannot: its receipts are unsigned, so its gate refuses every action at that level, and its command line says what a record lacks for a level instead of claiming one. It implements no PAP client or server, and no adapter for A2A, MCP, OAuth, AuthZEN, W3C PROV, in-toto or OpenTelemetry; [section 15](relationships-and-mappings.md) marks those mappings as proposed.

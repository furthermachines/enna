---
title: Conventions
description: Encoding, names, identifiers, revisions, references, time, money, scope, principals, extensions, versions and digests shared by every Enna document.
order: 2
status: normative
---

# Conventions

These conventions apply to the five objects (sections 3 to 7), the envelope ([section 8](envelope-and-errors.md)) and the discovery document ([section 12](discovery.md)) unless a section says otherwise. Where PAP 0.1 has a convention, Enna adopts it, so that one client can read both: snake_case members, kind-prefixed identifiers, integer revisions, domain-named extensions and the publication rules for well-known documents.

The JSON Schemas, published with this draft under `/spec/v0.1/schemas/`, express the parts of these rules that a schema can check. Where this text and a schema disagree, that is a defect in the draft, to be reported and fixed.

## 2.1 Encoding

An Enna document is JSON encoded in UTF-8.

- A Consumer MUST reject a document that is not well-formed JSON in UTF-8, with `invalid_json` at the path `""`. A byte order mark before the JSON text is not well-formed: RFC 8259, section 8.1, forbids a producer to add one, and decoding is strict, so a Consumer rejects it whether it checks the document as bytes or as text.
- A Consumer MUST reject a document in which a string or a member name holds a lone surrogate (a `\uD800` to `\uDFFF` escape that is not half of a surrogate pair), with `invalid_json` at `""`: I-JSON (RFC 7493) forbids it, and JCS cannot canonicalize it ([2.15](conventions.md)). A JSON parser may replace a lone surrogate silently, or drop it with the earlier of two repeated members, so a Consumer may need to look at the text itself.
- A Consumer MUST reject a document in which any object has two members with the same name, with `duplicate_member` at the pointer of the repeated member. Decoding reports one `duplicate_member`, at the first member, in text order, whose name repeats an earlier member of the same object, and nothing else. A JSON parser that silently keeps one of the two values is not enough on its own.
- These three checks come first, and run in the order listed. When one fails, no other check runs.

## 2.2 Names and vocabulary

- Member names and enumeration values defined by this draft are lowercase snake_case.
- Some values have their own syntax, defined where they appear: identifiers ([2.3](conventions.md)), action names ([2.4](conventions.md)), currency codes ([2.8](conventions.md)), domains ([2.10](conventions.md)), media types, extension names ([2.12](conventions.md)) and profile identifiers ([2.14](conventions.md)).
- A token is a vocabulary word chosen by whoever issues the object: a resource `type`, an item `kind`, an `approver_role`, a provenance `method` and the like. A token is a snake_case word of at most 128 characters, optionally prefixed by a domain ([2.10](conventions.md)) and a slash:

  ```text
  ^([a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?(\.[a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?)+/)?[a-z][a-z0-9_]*$
  ```

  Tokens are meaningful within the organization that issues the object, unless this draft or a profile defines them. A Producer MAY prefix a token with a domain it controls to make the token's origin explicit.
- Unprefixed action names, extension names and profile identifiers are reserved for this draft and its profiles. Any other definer MUST prefix it with a domain they control. A validator cannot tell who defined a name, so this rule is not machine-checked.
- A profile may reserve the unprefixed tokens of a vocabulary it defines. The software-delivery profile reserves the unprefixed artifact and evidence kinds: in an object that follows it, any unprefixed kind it does not define is a `schema_violation` ([section 14.3](software-delivery-profile.md)).

## 2.3 Identifiers

Every object, and every approval, artifact and context item, has an identifier with a mandatory prefix that names its kind:

| Prefix | Kind |
| --- | --- |
| `wc_` | work contract |
| `ctx_` | context manifest |
| `auth_` | authority binding |
| `rcpt_` | execution receipt |
| `eval_` | evaluation result |
| `env_` | envelope |
| `apr_` | approval |
| `art_` | artifact |
| `item_` | context item |

- An identifier is 1 to 256 characters in total from the URL-safe base64 alphabet (`A` to `Z`, `a` to `z`, `0` to `9`, `-` and `_`). It starts with its kind's prefix and has at least one character after the prefix. A value without the right prefix is a `schema_violation`.
- The Producer of an object chooses its identifiers. A Producer MUST NOT give the same identifier to two different objects of the same kind, or to two different approvals, within one organization; the revisions of one object share its identifier, and an approval carried into a binding's later revisions keeps its own. Identifiers appear in references, logs and traces, so a Producer SHOULD NOT put personal data or secrets in them.
- Within one list, identifiers are unique. A Consumer MUST reject a repeated identifier within the criteria, constraints or outputs of a contract, the items of a manifest, the approvals of a binding, the artifacts of a receipt, or the criteria or receipts of an evaluation, with `schema_violation` at the later repetition.
- Acceptance criteria use stable identifiers of the form `AC-1`: `AC-` and a number from 1 to 999999999 without leading zeros. A Producer MUST NOT reuse a criterion identifier within a contract for a criterion with a different meaning, so that evaluations and later revisions can refer to it.
- Constraints, outputs, workspaces and projects use local identifiers: 1 to 64 characters from the same alphabet.
- Idempotency keys are 1 to 256 characters from the same alphabet ([section 11.2](execution-semantics.md)).

## 2.4 Action names

An action name is at least two snake_case segments joined by dots, optionally prefixed by a domain ([2.10](conventions.md)) and a slash, at most 256 characters:

```text
^([a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?(\.[a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?)+/)?[a-z][a-z0-9_]*(\.[a-z][a-z0-9_]*)+$
```

- This draft defines one action, `acceptance.decide`, which delegates the acceptance decision ([section 7.5](evaluation-result.md)).
- Profiles define the actions of their domain; `software-delivery/0.1` defines seven ([section 14](software-delivery-profile.md)).
- An organization names its own actions under its domain, for example `harborpine.example/provisioning.retry`.

Actions match by exact string equality. Prefixing keeps an organization's action from colliding with one a later profile defines.

## 2.5 Revisions

Work contracts, context manifests, authority bindings and evaluation results are revisable. Execution receipts and envelopes are immutable and have no revision.

- A revisable object carries `revision`, an integer from 1.
- A revision is immutable once issued. A Producer MUST NOT issue different content under an `id` and `revision` it has issued. A change issues the next revision, numbered one higher than the latest, under the same `id`.
- From revision 2 on, `revision_author` (a principal reference) and `change_summary` (text) are required; a missing one is a `schema_violation`. They MAY appear on revision 1. Business contracts can be drafted and countered by either side, so every revision names who made it.
- A Consumer that holds a revision MUST NOT replace it with different content that arrives under the same `id` and `revision`. Receiving identical content again changes nothing. This draft defines no endpoint, so how such a conflict is reported is left open ([section 17.3](non-goals-and-open-questions.md)).

## 2.6 References

Objects refer to each other by reference, never by embedding:

| Reference | Members | Refers to |
| --- | --- | --- |
| Contract reference | `id` (`wc_`), `revision` | One revision of a work contract |
| Context reference | `id` (`ctx_`), `revision` | One revision of a context manifest |
| Authority reference | `id` (`auth_`), `revision` | One revision of an authority binding |
| Receipt reference | `id` (`rcpt_`), `digest` | One receipt, and the digest it must have |

A reference to a revisable object names exactly one revision. A receipt reference carries the receipt's digest, so a reader can tell whether the receipt it holds is the one that was cited. References are closed objects.

```jsonc
{
  "contract": { "id": "wc_brand_approvals", "revision": 1 },
  "receipts": [
    {
      "id": "rcpt_branch",
      "digest": "sha256:06d91777872676cbd1519356e82a020889dbcc32b68af0823a4572234e8b7d20"
    }
  ]
}
```

## 2.7 Time

- A timestamp is an RFC 3339 `date-time` in UTC: `YYYY-MM-DDThh:mm:ssZ`, optionally with a fraction of a second after the seconds, with an uppercase `T` and `Z`, and with every field in range. A leap second is accepted only as `23:59:60`. Any other offset, a lowercase `t` or `z`, or missing seconds is a `schema_violation`.
- Members that hold a timestamp end in `_at`. Durations are non-negative integer seconds, in members that end in `_seconds`, within the bound of every integer ([2.15](conventions.md)). A Consumer compares a time plus a duration without forming a new timestamp, so a large duration or a time near the year 9999 is never an error.
- Timestamps compare by instant. Fractions of a second compare exactly, digit by digit, never as floating-point numbers. A leap second, `23:59:60`, is the same instant as `00:00:00` of the next day, fraction included, as in POSIX time: `2026-06-30T23:59:60.5Z` and `2026-07-01T00:00:00.5Z` are equal.
- A window from `a` to `b` includes `a` and excludes `b`: an instant `t` is inside it when `a <= t < b`. This applies to an authority binding's `valid_from` and `expires_at`, and to an approval's `decided_at` and `expires_at`. A revocation takes effect at its `revoked_at`: an action that starts at that instant or later is revoked. A context item is fresh at `t` while `t <= as_of + max_age_seconds` ([section 4.3](context-manifest.md)).
- Every time-dependent rule in this draft is stated relative to an explicit current time. The reference implementation takes the current time as an input and never reads a clock inside a check, so its results are reproducible; other implementations SHOULD do the same.
- A record describes the past. A Consumer that verifies a record MUST reject an event timestamp later than the current time with `timestamp_in_future` ([section 9.15](invariants.md)). Expiries, deadlines and window starts are not events and are exempt.
- Timestamps that arrive from PAP with an offset are converted to UTC with `Z` when they are mapped ([section 15.1](relationships-and-mappings.md)).

## 2.8 Money

```jsonc
{ "currency": "USD", "amount": "1250.00" }
```

- Money is a closed object with two members, both required.
- `currency` is three uppercase letters, an ISO 4217 alphabetic code. A Producer MUST use a code from the current ISO 4217 List One (the edition published 2026-09-17 was the latest on 2026-10-10). Validators in this draft check the shape only, not membership in the list, so that they do not carry a dated table.
- `amount` is a non-negative decimal string: digits with no leading zero unless the integer part is `0`, then optionally a point and 1 to 6 fraction digits. It has no sign, no exponent and no spaces:

  ```text
  ^(0|[1-9][0-9]*)(\.[0-9]{1,6})?$
  ```

  Negative amounts cannot be expressed in 0.1 ([section 17.2](non-goals-and-open-questions.md)).
- A Consumer, Enforcement point or Evaluator that compares or adds amounts MUST do so exactly, using decimal or scaled-integer arithmetic, never binary floating point. Trailing zeros carry no meaning: `"400"` equals `"400.00"`. The number of fraction digits need not match the currency's minor unit, which ISO 4217 sets at 0, 2, 3 or 4 digits depending on the currency.
- Comparing or adding amounts in different currencies is an error, `currency_mismatch`. There is no implicit conversion.

Amounts are strings because RFC 8785 canonicalization and I-JSON treat numbers as IEEE 754 doubles, which cannot hold every decimal amount exactly. RFC 9396's payment example also writes its amount as a string.

## 2.9 Absent, null and unknown

- An absent member is not stated. A Consumer MUST NOT infer a value for it, except where this draft defines what absence means; the section that defines the member says so. For example, an authority binding with no `delegation` member allows no further delegation ([section 5.6](authority-binding.md)).
- `null` is never a valid value of a member this draft defines; it is a `schema_violation`. Inside extension data and inside PAP `terms`, which this draft does not define, the defining party's rules apply.
- `unknown` is an explicit value wherever a third state matters: a criterion result and the technical result (`pass`, `fail` or `unknown`), a receipt outcome (`unknown`), business impact (`unknown`), and the freshness of a context item, which is computed as `fresh`, `stale` or `unknown`.
- `unknown` never satisfies a requirement: no Consumer, Enforcement point or Evaluator may treat it as a pass, as fresh or as an approval. Nor is it a failure: a Producer MUST NOT report `failed` or `fail` for something whose result is not known.

## 2.10 Scope

```jsonc
{ "organization": "harborpine.example", "workspace": "aster-foods", "project": "brand-approvals" }
```

- `organization` is a domain the organization controls. `workspace` and `project` are optional local identifiers.
- A domain is lowercase: two or more labels separated by dots, each label 1 to 63 characters from `a` to `z`, `0` to `9` and `-`, neither starting nor ending with `-`, and at most 253 characters in all. It is the syntax of a scope's `organization`, a grantor's `organization` ([section 5.1](authority-binding.md)), an `organization` principal's `id` ([2.11](conventions.md)) and a discovery document's `organization.domain` ([section 12.3](discovery.md)); any other value there is a `schema_violation`:

  ```text
  ^[a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?(\.[a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?)+$
  ```
- The domain that may prefix a token ([2.2](conventions.md)), an action name ([2.4](conventions.md)), an extension name ([2.12](conventions.md)) or a profile identifier ([2.14](conventions.md)) is a domain as defined here, except that the 253-character limit is not applied to it: the length limit of the whole value, where it has one, bounds it.
- The organization plus the workspace is the tenant boundary. A scope with no `workspace` spans the whole organization.
- A project groups work and is not a tenant boundary. Grants can still be narrowed to a project ([section 5.2](authority-binding.md)).
- Scope A lies within scope B when both name the same organization, A names the same workspace wherever B names one, and A names the same project wherever B names one.

## 2.11 Principals

A principal reference names a person, agent, service or organization:

| `kind` | `id` | `issuer` |
| --- | --- | --- |
| `person` | The person's subject identifier at the issuer | Required: an HTTPS URL |
| `agent` | The agent's `client_id`, an HTTPS URL that serves its client metadata, as PAP 0.1 identifies personal agents | Optional |
| `service` | Any identifier the organization uses for the service, 1 to 2048 characters | Optional |
| `organization` | The organization's domain ([2.10](conventions.md)) | Optional |

- `display` is an optional name for people to read. It is never used to match principals.
- Two references name the same principal when `kind`, `id` and `issuer` are all equal; an `issuer` absent from both counts as equal. The comparison is exact.
- A person who arrives through PAP is named by the pairwise User ID their personal agent assigned, with the agent's `client_id` as the issuer ([section 15.1](relationships-and-mappings.md)).
- A principal reference identifies; it does not authenticate. Authentication belongs to the transport and the identity provider.

## 2.12 Extensions

```jsonc
{
  "extensions": {
    "example.com/notes": { "version": "1", "text": "Defined by example.com, ignorable." },
    "example.com/limits": { "version": "2", "critical": true }
  }
}
```

- Extension data lives only under `extensions`, an object keyed by extension name. A name is either plain, for an extension this draft defines, or a domain ([2.10](conventions.md)), a slash and a name: `^([a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?(\.[a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?)+/)?[a-z][a-z0-9_.-]*$`.
- Each value is an object with a required `version` (a major version, optionally with a minor: `1` or `1.2`) and an optional boolean `critical`. The extension defines its other members. An incompatible change to an extension takes a new major version, as in PAP.
- This draft defines one extension, `pap`, which links a work contract revision to a PAP operation ([section 3.7](work-contract.md)). It is defined for work contracts only.
- The definer of an extension MAY add requirements with it, and MUST NOT relax a requirement of this draft.
- A Consumer MUST ignore an extension it does not support unless the extension is critical. A Consumer MUST reject an object, an envelope or a discovery document that carries a critical extension it does not support, with `unsupported_critical_extension` at `/extensions/<name>` (with `/` in the name escaped as `~1`). An envelope's own extensions are checked as the envelope's, and each carried object's as that object's ([section 8.2](envelope-and-errors.md)).
- In an authority binding, every extension counts as critical whatever its `critical` member says: a Consumer MUST reject an authority binding that carries any extension it does not support. Authority the Consumer does not understand never grants anything.

## 2.13 Closed objects

Every object this draft defines is closed. A Consumer MUST reject a member the schema does not define, at any depth, with `unknown_member` at that member's pointer, except inside an extension's value and inside PAP `terms`. The schemas close every object with `additionalProperties: false`, and each object's own `properties` names every member it can carry, the shared members of [2.16](conventions.md) included; `extensions` names its members by the pattern of [2.12](conventions.md) instead. A member is unknown exactly when that list, or that pattern, does not name it, whatever a validator does with annotations.

A misspelled member therefore fails validation instead of being ignored, and new data has one explicit place to go: an extension.

## 2.14 Versions, profiles and schemas

- Every object, envelope and discovery document carries `protocol_version`, which is `"0.1"` for this draft.
- A Consumer MUST reject a document whose `protocol_version` it does not support, with `unsupported_version` at `/protocol_version`. It checks this before anything except decoding ([2.1](conventions.md)) and reports nothing else for that document. A missing or non-string `protocol_version` is a `schema_violation`.
- Before version 1.0, any two minor versions may be incompatible, so a 0.1 Consumer supports exactly `"0.1"`. The compatibility rule after 1.0, for example that a minor version only adds members a Consumer may ignore, is an open question ([section 17.3](non-goals-and-open-questions.md)).
- A profile identifier is a name, a slash and a `major.minor` version, optionally prefixed by a domain ([2.10](conventions.md)) and a slash: `^([a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?(\.[a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?)+/)?[a-z][a-z0-9-]*/[0-9]+\.[0-9]+$`. An object names the profile it follows in `profile`, for example `"software-delivery/0.1"`.
- A Consumer that knows an object's profile MUST apply the profile's rules on top of the core rules. A Consumer that does not know it MUST still apply the core rules, and MAY otherwise ignore the profile. An Enforcement point, by contrast, MUST refuse every action under a contract, or a binding, whose profile it does not know, with `unsupported_profile` at that object's `/profile`: it cannot enforce rules it does not know.
- The schemas are JSON Schema 2020-12. Their `$id` values have the form `https://businessagentprotocol.org/spec/v0.1/schemas/<name>.schema.json`. An `$id` is an identifier, not a location to fetch: a Consumer MUST NOT fetch schemas over the network to validate a document, and SHOULD bundle them, as the reference implementation does by compiling them ahead of time. The schema files are `common`, `work-contract`, `context-manifest`, `authority-binding`, `execution-receipt`, `evaluation-result`, `envelope`, `error`, `discovery` and `profiles/software-delivery-0.1`.
- In this draft `format` is an assertion, not an annotation: a Consumer MUST apply each format's rule as this draft states it, not whatever its validator's format check does. Two formats are used. `date-time` is the timestamp rule of [2.7](conventions.md). `uri` is the `URI` production of RFC 3986, Appendix A, without the `path-empty` alternative of its `hier-part`: a scheme, a colon and a hierarchical part that is not empty, then an optional query and fragment. The production allows ASCII characters only, each where it names it, so any other character is percent-encoded; a port is digits only, and a bracket appears only around an IP literal in the authority.
- Patterns use only the regular-expression subset that JSON Schema recommends for interoperability (no class escapes such as `\s` or `\d`, no lookaround, no backreferences), so ECMA-262, RE2 and PCRE engines agree on them, provided `$` matches only at the very end of the value, as in ECMA-262. PCRE's `$` also matches before a final line break unless it is told not to.

## 2.15 Canonical form and digests

- A digest in this draft is `sha256:` followed by 64 lowercase hexadecimal digits: the SHA-256 hash of the UTF-8 bytes of the value's canonical form under the JSON Canonicalization Scheme (JCS, RFC 8785).
- JCS requires I-JSON input. Every number in a member this draft defines is an integer (money is a string), and every such integer is at most 9007199254740991 (2^53 - 1), and at least -9007199254740991 where it may be negative: the range an IEEE 754 double holds exactly, so every platform reads it exactly and canonical forms are the same everywhere. An integer outside that range is a `schema_violation`. A canonicalizer MUST fail on a lone surrogate, as RFC 8785 requires.
- Two digests are defined over Enna data: the receipt digest ([section 6.6](execution-receipt.md)) and the binding digest that an approval covers ([section 5.3](authority-binding.md)).
- Artifacts and evidence carry digests of their own content as `<algorithm>:<lowercase hex>`, where the algorithm is `sha1`, `sha256`, `sha384` or `sha512`. What the digest is computed over depends on the artifact's kind. SHA-1 appears only where an external system names its objects by SHA-1, such as a git commit ([section 14](software-delivery-profile.md)).
- A digest shows that data is unchanged only against a digest the reader already trusts: whoever holds the data can change it and compute a new digest. It does not show that an action was authorized or that a history is honest ([section 6.6](execution-receipt.md)).

## 2.16 Members every object carries

Each of the five objects has these members, besides its own:

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `protocol_version` | `"0.1"` | Yes | The version of this draft ([2.14](conventions.md)). |
| `type` | string | Yes | `work_contract`, `context_manifest`, `authority_binding`, `execution_receipt` or `evaluation_result`. Any other value is a `schema_violation` at `/type`. |
| `id` | identifier | Yes | With the prefix of the object's kind ([2.3](conventions.md)). |
| `created_at` | timestamp | Yes | When this object, or this revision of it, was issued. |
| `scope` | scope | Yes | Where the object belongs. It lies within the contract's tenant boundary ([section 9.4](invariants.md)). |
| `profile` | profile identifier | No | The profile whose rules the object follows ([2.14](conventions.md)). |
| `extensions` | object | No | Extension data ([2.12](conventions.md)). |

The four revisable objects add `revision` (required), and `revision_author` and `change_summary`, which are required from revision 2 ([2.5](conventions.md)).

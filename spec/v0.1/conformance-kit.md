---
title: Conformance kit
description: What the fixtures, test vectors and catalogs published with draft 0.1 hold, how to run them against an implementation in any language, and how to check a download against the manifest.
order: 20
status: informative
---

# Conformance kit

The conformance kit is the set of files that tests an implementation of this draft, in any language: the conformance fixtures of [section 13.1](conformance.md), test vectors for canonical forms and digests, and catalogs of the draft's tables in JSON. The site publishes each file byte for byte as the protocol package of the reference implementation holds it, with a manifest that gives the SHA-256 of every file.

The kit's files are licensed under the Apache License 2.0. This page, like the rest of the specification, is licensed under CC BY 4.0. The kit belongs to draft 0.1 and changes with it, at the same addresses: a correction to the draft changes the files it touches, and the manifest's hashes show which files changed.

## The files

Every address is on `https://businessagentprotocol.org`.

| Address | What it holds |
| --- | --- |
| `/spec/v0.1/fixtures/valid/` | Documents that pass, one per file, and `index.json`, which says how to check each. |
| `/spec/v0.1/fixtures/invalid/` | Cases that fail, one per file, each with the (code, path) pairs its check reports. |
| `/spec/v0.1/fixtures/vectors/canonicalization.json` | JSON texts with their canonical forms and digests ([section 2.15](conventions.md)). |
| `/spec/v0.1/fixtures/vectors/digests.json` | The canonical form and digest of each distinct execution receipt and authority binding in the valid fixtures ([sections 5.3](authority-binding.md) and [6.6](execution-receipt.md)). |
| `/spec/v0.1/catalogs/error-codes.json` | The error catalog ([section 8.4](envelope-and-errors.md)). |
| `/spec/v0.1/catalogs/lifecycle.json` | The business lifecycle and its mappings to A2A and PAP states ([section 10](lifecycle.md)). |
| `/spec/v0.1/catalogs/identifiers.json` | The prefix of each kind of identifier ([section 2.3](conventions.md)). |
| `/spec/v0.1/catalogs/profiles.json` | The profiles this draft defines ([section 2.14](conventions.md)). |
| `/spec/v0.1/catalogs/extensions.json` | The extensions this draft defines ([section 2.12](conventions.md)). |
| `/spec/v0.1/schemas/` | The JSON Schemas ([section 2.14](conventions.md)). |
| `/spec/v0.1/manifest.json` | Every file under `/spec/v0.1/` but the pages and the manifest itself, with its media type, SHA-256 and license. |

A folder has no page of its own; the manifest lists every file in it. The fixtures, the vectors, the catalogs and the manifest are served as `application/json`, and the schemas as `application/schema+json`.

## Running the fixtures

1. Read `fixtures/valid/index.json`. It names every valid file, with the check to run on it (`validate`) and the inputs of that check (`options`).
2. Run each valid file's check on its document, with its inputs. The check reports no (code, path) pair.
3. For each file in `fixtures/invalid/`, build the input. A case with `input_text` gives the exact text to check, and a case with `input_base64` the exact bytes; both are checked by decoding alone ([section 2.1](conventions.md)), so hand them to the decoder as they are. Any other case starts from a copy of the valid document its `base` names, applies its `patch`, and, when it has `reseal`, recomputes the digests the patch made stale (below).
4. Run the check the case's `validate` names on the input, with the case's `options`.
5. Compare the (code, path) pairs the check reports with the case's `expected`. [Section 13.1](conformance.md) states the requirement: a Consumer that claims conformance MUST report at least every expected pair, and SHOULD report exactly the expected list (checklist item C-11).

A claim covers the kinds of document the implementation validates: objects, envelopes, records and discovery documents. Every Consumer decodes, so the `json` cases apply to all of them. `validate` names one of these checks:

| `validate` | The check |
| --- | --- |
| `json` | Decoding alone: strict UTF-8 JSON, with no byte order mark, no lone surrogate and no repeated member name ([section 2.1](conventions.md)). Only invalid cases use it. |
| `object` | One of the five objects, in the order of [section 8.2](envelope-and-errors.md). |
| `envelope` | An envelope and the objects it carries ([section 8.2](envelope-and-errors.md)). |
| `bundle` | A record of one contract revision: its objects, then the invariants ([section 9.1](invariants.md)). |
| `discovery` | An `enna.json` document, in the order of [section 12.7](discovery.md). |

Each `path` in `expected` points into the input as [section 8.3](envelope-and-errors.md) says: into the object itself, under `/objects/<index>` for an envelope, and under `/<index>` for a record.

## The case format

### Valid documents

Each file in `valid/` holds one document, or, for a record, a JSON array of objects. `valid/index.json` maps each file name to how it is checked:

| Member | Meaning |
| --- | --- |
| `validate` | The check: `object`, `envelope`, `bundle` or `discovery`. |
| `options` | The inputs the check takes besides the document (below). Absent means none. |

For example, the record in `bundle-brand-approvals.json` is checked at the Controlled level:

```jsonc
{
  "bundle-brand-approvals.json": {
    "validate": "bundle",
    "options": {
      "now": "2026-10-20T00:00:00Z",
      "level": "controlled"
    }
  }
}
```

### Invalid cases

Each file in `invalid/` is one case:

| Member | Meaning |
| --- | --- |
| `description` | What is wrong, in a sentence. |
| `section` | The section of this draft whose rule the case exercises. |
| `validate` | The check, as above, or `json` for text or bytes that decoding alone refuses. |
| `base` | The name of a valid fixture the case starts from. |
| `input_text` | Instead of `base`, the exact text to check, for `json` cases. |
| `input_base64` | Instead of `base` or `input_text`, the exact bytes to check, in base64 (RFC 4648, the standard alphabet, padded), for `json` cases whose bytes are not text. A runner decodes it to bytes and decodes those as [section 2.1](conventions.md) says. |
| `patch` | Edits applied to a copy of `base`, in order (below). |
| `reseal` | When present, the digests to recompute after the patch (below): `receipts` for the receipt chain and the evaluations' citations, `all` for the approvals' binding digests as well. |
| `options` | The inputs of the check (below). |
| `expected` | The exact list of `{ "code", "path" }` pairs the check reports, in order: sorted by path, then by code. |

A case with a `base` and no `patch` checks the valid document as it is, under other inputs or as another kind of document. A case, in full:

```jsonc
{
  "description": "A cost in another currency than the limits.",
  "section": "9.9",
  "validate": "bundle",
  "base": "bundle-brand-approvals.json",
  "patch": [
    {
      "op": "replace",
      "path": "/6/cost/currency",
      "value": "EUR"
    }
  ],
  "reseal": "receipts",
  "options": {
    "now": "2026-10-20T00:00:00Z"
  },
  "expected": [
    {
      "code": "currency_mismatch",
      "path": "/6/cost"
    }
  ]
}
```

### Patches

A patch is a list of operations from RFC 6902, limited to `add`, `replace` and `remove`. Each has an `op`, a `path` (an RFC 6901 JSON Pointer into the document) and, for `add` and `replace`, a `value`. In an array, `add` inserts before the index it names, and the index `-` appends.

### Resealing

A case resealed with `all` first sets, for every authority binding in the record, every approval's `covers.binding_digest` to the binding's digest ([section 5.3](authority-binding.md)). Then, for both `receipts` and `all`:

1. Take the execution receipts in array order.
2. For each, remove `integrity.previous_digest`; if another receipt came before it in the array, set `integrity.previous_digest` to the `integrity.digest` of the receipt just before it in the array.
3. Set its `integrity.digest` to its digest ([section 6.6](execution-receipt.md)).
4. Finally, for every evaluation result, set the `digest` of each entry of its `receipts` to the digest of the receipt with that `id`, where the record holds one.

A case without `reseal` is checked as patched: its digests are left as the base had them, on purpose.

### Options

The options name the inputs of [section 9.1](invariants.md) and [section 12](discovery.md):

| Option | Input |
| --- | --- |
| `now` | The current time, a UTC timestamp. |
| `level` | The assurance level: `essential`, `controlled` or `assured`. Absent means `essential`. |
| `sodThreshold` | The checking party's separation-of-duties threshold, as money. |
| `revocations` | The live revocation status: a list of `{ "id", "revoked_at" }`, each a binding or approval identifier and the time it was revoked. |
| `requestedHost` | For a discovery document, the host it was requested from. |
| `issuerMetadata` | For a discovery document, the authorization server metadata its `auth.issuer` serves. |
| `poppy` | For a discovery document, the linked `poppy.json`: its `url` and its `document`. |

No case names the supported extensions of [section 9.1](invariants.md): every case is checked with no extension supported but the core one, `pap`, which work contracts carry ([section 2.12](conventions.md)).

## From a schema validator to (code, path) pairs

The schemas check the shape of a document ([section 8.2](envelope-and-errors.md), step 5). They refer to each other by `$id`, which is an identifier, not a location to fetch, so bundle all of them, from `/spec/v0.1/schemas/`, with the validator ([section 2.14](conventions.md)). A JSON Schema validator reports failing assertions, not (code, path) pairs. Have it report every failing assertion of a step, since a step reports all of its errors together, where most validators stop at the first unless they are told otherwise. Apply a profile's schema (step 6) only once the object's own schema passes: [section 8.2](envelope-and-errors.md) stops at the first step that finds an error, so a profile's errors are never reported beside the object schema's, as `object-schema-before-profile.json` shows. From its output:

- A failing `required` is `schema_violation` at the pointer of the missing member.
- A member that `additionalProperties` refuses is `unknown_member` at that member's pointer.
- A `false` subschema, which forbids a member, is `schema_violation` at that member.
- Every other failing assertion is `schema_violation` at its instance location: the value that fails it.
- The `if` of a conditional reports nothing itself; the assertions of its `then` or `else` report instead. An `allOf` reports nothing of its own either.
- Each (code, path) pair is reported once, sorted by path, then by code, comparing strings by UTF-16 code units ([section 8.3](envelope-and-errors.md)).

Every object's schema lists all of the object's members and closes it with `additionalProperties: false`, so a member is unknown exactly when that list does not name it, whatever a validator does with annotations ([section 2.13](conventions.md)). Unknown members therefore come out the same in every validator.

## Formats and text

- `format` is an assertion in this draft, with the rules of [section 2.14](conventions.md): `date-time` is the timestamp rule of [section 2.7](conventions.md), and `uri` is the `URI` of RFC 3986 with a hierarchical part that is not empty. A library's own checks may differ. Its `date-time` check may refuse `23:59:60`, or accept a sixtieth second at another time of day, and its time arithmetic may not treat `23:59:60` as the same instant as `00:00:00` of the next day, as section 2.7 does. Its `uri` check may accept a value with nothing after its scheme, a port that is not digits, or a bracket in a path; fixtures hold each. Apply the draft's rules.
- Patterns keep to the subset of regular expressions that JSON Schema recommends ([section 2.14](conventions.md)), and `$` matches only at the very end of a value, as in ECMA-262. In PCRE, Python, Java and .NET, `$` also matches before a final line break unless the engine is told otherwise, for example with PCRE's `PCRE2_DOLLAR_ENDONLY` option, or by ending the pattern with `\z` instead (`\Z` in Python). A fixture holds a domain that ends with a line break.
- A lone surrogate in a string or a member name is `invalid_json` at `""`, found at decoding ([section 2.1](conventions.md)). Many JSON parsers replace a lone surrogate or drop it without a word, so check the text itself. Decoding is strict UTF-8 too: an overlong form, a truncated sequence, a continuation byte with no lead byte and an encoded surrogate are all refused, where a lenient decoder would replace them.
- A record is decoded as one document before its objects are checked ([section 9.1](invariants.md)), so a lone surrogate anywhere in it is `invalid_json` at `""`, not an error at one object's path.

## The test vectors

### Canonicalization

`fixtures/vectors/canonicalization.json` tests the canonical form of [section 2.15](conventions.md), the JSON Canonicalization Scheme of RFC 8785. Besides `draft` and `defined_in`, it holds `cases`, each with:

| Member | Meaning |
| --- | --- |
| `description` | What the case shows, in a sentence. |
| `input_text` | A JSON text to parse. |
| `canonical` | Its canonical form, exactly. |
| `digest` | `sha256:` and the lowercase hexadecimal SHA-256 of the canonical form's UTF-8 bytes: a digest as section 2.15 defines it. |
| `error` | Instead of `canonical` and `digest`, why the text has no canonical form. `lone_surrogate`: a string or a member name holds a lone surrogate, on which a canonicalizer fails. |

Parse `input_text`, canonicalize the value, and compare the result with `canonical` and its digest with `digest`. The cases are the examples of RFC 8785 and others that follow its rules: member names sorted by UTF-16 code units at every depth, the escapes in strings, integers, numbers from its Appendix B written in another form, and the literals. Strict decoding refuses the text of an `error` case before a canonicalizer sees it ([section 2.1](conventions.md)), so to test a canonicalizer on it, parse the text with a parser that keeps a lone surrogate. A case:

```jsonc
{
  "description": "Zero is written 0, and so is minus zero.",
  "input_text": "[0, -0]",
  "canonical": "[0,0]",
  "digest": "sha256:3d5812abc84c11768aa73a732c85d75dbed439188f5bb3239e9b762ea31d9862"
}
```

### Digests

`fixtures/vectors/digests.json` gives the two digests this draft defines over Enna data for the execution receipts and authority bindings in the valid fixtures, once for each distinct digest, with the canonical form each is computed over. Besides `draft`, it holds `receipts` and `bindings`, each with its own `defined_in` and its `cases`. Each case is the first receipt or binding found with its digest, taking the fixtures in the order of their file names and the objects of each in record order.

| Member | Meaning |
| --- | --- |
| `fixture` | The fixture's path under `/spec/v0.1/fixtures/`, such as `valid/bundle-brand-approvals.json`. |
| `pointer` | The JSON Pointer of the object in that fixture: `""` for a file that holds it alone, `/3` for the fourth object of a record, `/objects/2` for the third object an envelope carries. |
| `id` | The object's identifier. |
| `canonical` | The canonical form the digest covers: the receipt without `integrity.digest` ([section 6.6](execution-receipt.md)), or the binding without `approvals`, `revision`, `revision_author`, `change_summary` and `created_at` ([section 5.3](authority-binding.md)). |
| `digest` | For a receipt, its digest, which its `integrity.digest` states. |
| `previous_digest` | For a receipt that states one, its `integrity.previous_digest`: the digest of the receipt before it in its chain. |
| `binding_digest` | For a binding, its digest, which the `covers.binding_digest` of each of its approvals states. |

## The catalogs

Each file in `/spec/v0.1/catalogs/` starts with `draft`, which is `"0.1"`, and `defined_in`: the `section` that defines what the file lists, and the `url` of that section's page. Entries keep the order of the protocol package's own data.

- `error-codes.json`: `codes`, each with its `code`, the `area` of [section 8.4](envelope-and-errors.md) it is listed under, and its `description`.
- `lifecycle.json`: `states`, `terminal_states`, `events`, `transitions` (each a `from` state, an `event` and a `to` state) and `block_reasons`, from [sections 10.1 to 10.4](lifecycle.md). Then `a2a`, the mapping of [section 10.6](lifecycle.md): `task_states`, the A2A task states; `states`, giving each business state its task state `a2a`, a `match` of `exact` or `approximate`, and a `note`; and `unmapped`, the task states that stand for no business state, each with the reason. Then `pap`, the mapping of [section 10.7](lifecycle.md): `states`, giving each business state its operation state `pap`, or `from_receipts` where the receipts decide it, a `match` of `exact`, `approximate` or `derived`, and a `note`; and `unmapped`. A `note` is empty where an exact match needs none.
- `identifiers.json`: `prefixes`, each kind of identifier with its prefix.
- `profiles.json`: `profiles`, each with its `id`, the `$id` of its schema as `schema`, which is also the address the schema is published at, and the `defined_in` of the section that defines it.
- `extensions.json`: `core`, each core extension with the type of `object` it is defined for, its `name`, and the `defined_in` of the section that defines it.

## The manifest

`/spec/v0.1/manifest.json` lists every file under `/spec/v0.1/` but the pages and the manifest itself: the Markdown of each page of the specification, the schemas, the fixtures, the vectors and the catalogs. Besides `draft`, it holds `files`, sorted by `path`, each with:

| Member | Meaning |
| --- | --- |
| `path` | Where the file is published, on `https://businessagentprotocol.org`. |
| `media_type` | The media type the file is served with: `text/markdown; charset=utf-8` for the Markdown, `application/schema+json` for the schemas, and `application/json` for the rest. |
| `sha256` | The SHA-256 of the file's bytes, as 64 lowercase hexadecimal digits. |
| `license` | The file's license, by its SPDX identifier: `CC-BY-4.0` for the Markdown, and `Apache-2.0` for the rest. |

The manifest holds no time, so the same files always give the same manifest. To check a download, compute the SHA-256 of each file's bytes exactly as they were served, before parsing them or changing their line ends, and compare it with the file's `sha256`. To see what changed between two copies of the kit, compare their manifests. Like any digest, a hash shows that a file is unchanged only against a manifest you already trust ([section 2.15](conventions.md)).

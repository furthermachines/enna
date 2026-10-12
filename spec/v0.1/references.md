---
title: References
description: The normative and informative references of draft 0.1, each with the version cited and the date it was checked.
order: 18
status: normative
---

# References

Each reference gives the version this draft cites. A date in the Checked column is the day the source was read for this draft. "Not rechecked" marks long-stable references cited by number without being read again for this draft.

## 18.1 Normative references

| Reference | Version cited | Checked |
| --- | --- | --- |
| Personal Agent Protocol, specification. personalagentprotocol.org/docs/spec | Draft 0.1, pages stamped "Updated Oct 9, 2026" | 2026-10-10 |
| Personal Agent Protocol, operations extension. personalagentprotocol.org/docs/extensions/operations | Version 1, in draft 0.1 | 2026-10-10 |
| BCP 14: RFC 2119 and RFC 8174, key words for requirement levels | As PAP 0.1 cites them | Not rechecked |
| RFC 8259, the JSON data interchange format | | Not rechecked |
| RFC 7493, the I-JSON message format | | Not rechecked |
| RFC 3339, date and time on the internet | Proposed Standard, July 2002, as updated by RFC 9557 (April 2024) | 2026-10-10 |
| RFC 3986, URI generic syntax | | Not rechecked |
| RFC 6901, JSON Pointer | Proposed Standard, April 2013 | 2026-10-10 |
| RFC 8785, JSON Canonicalization Scheme | Informational (Independent Submission), June 2020 | 2026-10-10 |
| FIPS 180-4, Secure Hash Standard (SHA-256) | | Not rechecked |
| JSON Schema 2020-12, Core and Validation | draft-bhutton-json-schema-01 and draft-bhutton-json-schema-validation-01, 2022-06-10 | 2026-10-10 |
| ECMA-262, ECMAScript language specification, for the regular expressions of section 2.14, in which `$` matches only at the end of the input when the `m` flag is not set. 262.ecma-international.org/17.0 | 17th edition, June 2026, approved by the Ecma General Assembly on 2026-06-30 | 2026-10-11 |
| ISO 4217, currency codes | List One as published by SIX on 2026-09-17; the standard's edition could not be confirmed on iso.org | 2026-10-10 |
| ISO 3166, country and subdivision codes (for `jurisdiction`) | | Not rechecked |
| RFC 8615, well-known URIs | Proposed Standard, May 2019 | 2026-10-10 |
| RFC 8414, OAuth 2.0 authorization server metadata | | Not rechecked |
| W3C Trace Context Level 1 | Recommendation, 2021-11-23 | 2026-10-10 |
| A2A (Agent2Agent) protocol. a2a-protocol.org/latest/specification | Protocol version 1.0: release v1.0.0 (2026-03-12) and v1.0.1 (2026-05-28); task states from `specification/a2a.proto` at v1.0.1 | 2026-10-10 |

PAP publishes no versioned URLs, changelog or archive, so its pages are identified by what they said. These are the SHA-256 digests of the Markdown alternates of PAP's pages, as served on 2026-10-10:

| Page | Bytes | SHA-256 |
| --- | --- | --- |
| personalagentprotocol.org/docs/spec.md | 75056 | `207db936b1819fa3669d7e2079be6c8887f30a1028ab10520cb8e51e3fcadeb5` |
| personalagentprotocol.org/docs/extensions/operations.md | 17121 | `d7ecd37076e02409b15b57a28814a2cb526333151027e8be7eb9695b671c1539` |
| personalagentprotocol.org/docs/extensions.md | 1638 | `41247520f3e66f01881f1e347fb082a1f70af6103bf782fdefc83df2cf9f076b` |
| personalagentprotocol.org/docs/guides.md | 22462 | `303076649b8c301b802d0031e4007d598e66f4a3d1c15355d12d9f6ba715cb3e` |
| personalagentprotocol.org/docs/overview.md | 3383 | `2467c0efefae4f3e5d23be207a1ffa81813a2d4c28098c23c793d9b9bb5d9242` |

This draft restates the PAP rules it applies outside the bridge, such as the publication rules of [section 12.2](discovery.md) and the operation semantics of [section 11](execution-semantics.md). The bridge of [section 15.1](relationships-and-mappings.md) depends on PAP's own text, and PAP is a normative reference for it alone.

## 18.2 Informative references

| Reference | Version cited | Checked |
| --- | --- | --- |
| Model Context Protocol. modelcontextprotocol.io/specification | Revision 2026-07-28 ("Current"), including its authorization pages, security best practices and the tasks extension (ext-tasks v0.2.2) | 2026-10-10 |
| OAuth Client ID Metadata Document | draft-ietf-oauth-client-id-metadata-document-00, as MCP 2026-07-28 cites it | 2026-10-10 |
| RFC 8693, OAuth 2.0 Token Exchange | Proposed Standard, January 2020 | 2026-10-10 |
| RFC 9396, OAuth 2.0 Rich Authorization Requests | Proposed Standard, May 2023 | 2026-10-10 |
| RFC 8707, Resource Indicators for OAuth 2.0 | Proposed Standard, February 2020 | 2026-10-10 |
| RFC 9449, OAuth 2.0 Demonstrating Proof of Possession (DPoP) | Proposed Standard, September 2023 | 2026-10-10 |
| RFC 6750, OAuth 2.0 Bearer Token Usage | | Not rechecked |
| RFC 7009, OAuth 2.0 Token Revocation | Title confirmed | 2026-10-10 |
| RFC 7515, JSON Web Signature | Title confirmed | 2026-10-10 |
| RFC 9421, HTTP Message Signatures | Title confirmed | 2026-10-10 |
| RFC 9457, Problem Details for HTTP APIs | Proposed Standard, July 2023 | 2026-10-10 |
| RFC 4648, the Base16, Base32 and Base64 data encodings, for the conformance kit's `input_base64` | Proposed Standard, October 2006 | Not rechecked |
| RFC 6902, JSON Patch, for the conformance kit's patches | Proposed Standard, April 2013 | Not rechecked |
| POSIX, base definitions, section 4.19, "Seconds Since the Epoch", in which every day is exactly 86400 seconds, for the leap second of section 2.7. pubs.opengroup.org/onlinepubs/9799919799/basedefs/V1_chap04.html | IEEE Std 1003.1-2024, The Open Group Base Specifications Issue 8 | 2026-10-11 |
| RE2 regular expression syntax, for section 2.14: `$` matches at the end of the text, like `\z`, and `\Z` is not supported. github.com/google/re2/wiki/Syntax | The wiki page as last edited on 2023-08-11, its 25th revision | 2026-10-11 |
| PCRE2, the pcre2pattern documentation, for section 2.14: `$` also matches before a newline at the end of the subject unless `PCRE2_DOLLAR_ENDONLY` is set. pcre.org/current/doc/html/pcre2pattern.html | Release 10.48, its page stamped "Last updated: 25 October 2025" | 2026-10-11 |
| OpenID AuthZEN Authorization API | 1.0, OpenID Final Specification, approved January 2026 | 2026-10-10 |
| W3C PROV-O | Recommendation, 2013-04-30 | 2026-10-10 |
| PROV-JSON | W3C Member Submission, 2013-04-24 | 2026-10-10 |
| in-toto Attestation Framework | Spec v1, repository release v1.2.0 (2026-03-18); Statement `https://in-toto.io/Statement/v1` | 2026-10-10 |
| DSSE, Dead Simple Signing Envelope | v1.0.2 (2024-05-10) | 2026-10-10 |
| SLSA | Specification v1.2; provenance predicate `https://slsa.dev/provenance/v1` | 2026-10-10 |
| OpenTelemetry semantic conventions | v1.44.0 (2026-08-04); GenAI conventions in Development status, moved to `open-telemetry/semantic-conventions-genai` with no release | 2026-10-10 |
| IANA Well-Known URIs registry | Last updated 2026-10-07 | 2026-10-10 |
| Well-known URI registry guidance, github.com/protocol-registries/well-known-uris | README last updated 2026-04-01 | 2026-10-10 |

The project's research notes, in the project's private build repository, record what each source said on the date it was checked, with the URLs read.

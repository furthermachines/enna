# Changelog

Every change to the specification, its schemas, the reference implementation and the site is listed here. Before 1.0, any draft may change incompatibly with the one before.

## 0.1: the reporting addresses, 2026-10-12

- **Decision 3 is decided.** Security reports go through GitHub's private vulnerability reporting on the public repository, which is preferred because it keeps the report and its fix together, or by email to `security@businessagentprotocol.org`. Conduct reports go by email to `conduct@businessagentprotocol.org`. The project's owner reads both addresses. `SECURITY.md`, `CODE_OF_CONDUCT.md`, `GOVERNANCE.md`, both `CONTRIBUTING.md` files, both `README.md` files and the threat model say so. The site's discovery document still names the GitHub form in `contacts.security`.
- **Site.** The security page, its description and the FAQ name both ways to report, and the FAQ the conduct address. The home page and the FAQ no longer list an email channel among the open decisions.
- **Cloudflare's Email Address Obfuscation**, on for the zone, would rewrite an address into a link that only a Cloudflare script decodes, which the site's security policy blocks. Every address the site renders sits between `<!--email_off-->` and `<!--/email_off-->` markers, which Cloudflare leaves alone: the pages write them, and the Markdown pipeline adds them around every `mailto:` link of a rendered document. The threat model lists the threat.
- **Tests.** The site's tests check that every address in every built page sits inside the markers, and fail without them; that the security page renders `SECURITY.md`'s email address and the preference for GitHub; that the governance page shows decision 3 decided and renders the conduct address; and that no page says there is no email channel or that decision 3 is partly decided.
- **Runbook.** It records decision 3, why the markers exist, the fourth deploy (the public repository, at commit `1c45ada`) with its `504 Gateway Timeout` and what was checked live, the first publication of `furthermachines/enna`, and the Email Routing set up for the two addresses.

## 0.1: the specification in a public repository, 2026-10-12

- **Public repository** (decision 2, decided 2026-10-12). The specification is public at `https://github.com/furthermachines/enna`: the 20 Markdown alternates of its text, the 10 JSON Schemas, the catalogs, the fixtures and test vectors and `manifest.json`, as the site publishes them under `/spec/v0.1/`, with `LICENSE`, `LICENSE-CC-BY-4.0`, `LICENSING.md`, `GOVERNANCE.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md` and this changelog, its own `README.md` and `CONTRIBUTING.md`, kept in the build repository for it, and a `.gitattributes` holding `* -text`, so that clones keep the bytes the manifest hashes. The TypeScript packages, the site's code and the documentation (the decision records, the runbook and the research notes) stay private. Questions and proposals go in its issues; it does not merge pull requests, because it is regenerated from this repository's production build at each release, and whether contributors to the specification text make a patent commitment is undecided.
- **`npm run export:public -- <folder>`** writes it from the production build. Before it writes or deletes anything, it refuses a manifest path that is not a plain file name under `/spec/v0.1/` (a backslash, an empty, `.` or `..` segment, or an encoded separator), a file whose SHA-256 is not the manifest's, any input that is a symbolic link or lies behind one, a folder that holds files but is neither a checkout holding only `.git` nor an earlier export, and this repository or any folder that contains it, compared by real path so that a symbolic link or another case of the path is refused too. It says to run `npm run build:production` first when the manifest is missing. It runs as a command through a symbolic link, and importing it runs nothing. In an earlier export it deletes everything but `.git` first. Its tests hold each refusal; each was shown failing first. The runbook gives the order of the first publication: the repository and its reporting form, then the deploy, then the export.
- **The Markdown alternates link to each other by their published names.** Each is its section's file as written, frontmatter included, with each link to a numbered section, inline or as a reference definition, renamed to the name the site publishes it under, such as `03-work-contract.md#...` to `work-contract.md#...`. The file is parsed as Markdown with `satteri`, the parser Astro uses, which the site and the release tests now declare as a dependency, and only those destinations change, at their offsets in the source, so code of every kind, list continuations and every other byte stay as written. The links named the source files, which neither the site nor the public repository publishes, so they did not resolve. The manifest's hashes of the 20 Markdown files change with them.
- **Security reports** (decision 3, partly decided 2026-10-12) go through GitHub's private vulnerability reporting on the public repository. `SECURITY.md` links it and says what to include in a report, and the site's `/.well-known/enna.json` names it in `contacts.security`. The conformance kit's valid fixture `/spec/v0.1/fixtures/valid/discovery-spec-site.json` is that document, so it gained the member too, and its bytes and its SHA-256 in the manifest changed. Section 12.8 says so and shows the document with it, and a dated note under the document history of section 1.4 records the change, which is not normative. The email channel, for security and conduct reports, is still open, so `CODE_OF_CONDUCT.md` still names no channel for conduct reports.
- **Documents.** `GOVERNANCE.md`, `CONTRIBUTING.md`, `LICENSING.md`, `README.md`, the runbook and the threat model say where the specification is public, that the build repository stays private, and how security problems are reported. The documents the public repository holds name a section by its published name, link the site for the threat model and the third-party notices, and name the private build's files without paths. So does the specification, with no normative change (sections 1.4, 1.5, 2, 11.12, 13.6, 15, 16.6, 17.4 and 18.2): it names the schemas by their address on the site, the threat model by its page, and the reference implementation, its tests, the decision records and the research notes without paths into the private build repository, and the dated note in section 1.4 records it. The threat model adds GitHub as a trust boundary, and a threat: the public repository differing from what the site publishes, with what the hash check does not catch.
- **Site.** The FAQ, the Implement page, the home page and the security page link the public repository and the reporting form, and say that the packages and the playground's code stay private; pages that name the private build repository say so. A rendered document's link to a section by its published name, or to this site's address, becomes a link to the page.
- **Tests.** The site's tests check that the security page links the reporting form from `SECURITY.md` itself, that the site's discovery document names it, that the governance page shows decision 2 decided and decision 3 partly decided, and that no page's text or description says the repository is not public or that no private reporting channel exists. They check that each Markdown alternate differs from its source only in the numbers of the sections it links, that every link in every alternate names a Markdown file the site publishes (639 broken before the rename), and, literally, how the fixture build renames each link form, a link in a list item's continuation among them, and keeps code blocks and inline code as written; both link checks parse the Markdown as the rewrite does. The release suite checks that every reference in the public repository's Markdown names a file it holds or a public URL, and that its link check allows the public repository and still finds any other repository named for the project.

## 0.1: licensed and published, 2026-10-11

- **License.** The owner licensed the project on 2026-10-11, as proposed: Apache-2.0 for code (the packages, the site, the scripts, the tests and the examples), the JSON Schemas and the fixtures, and CC BY 4.0 for the specification text, the documentation, the repository's Markdown files and the site's prose.
  - `LICENSE` and `LICENSE-CC-BY-4.0` hold the two texts, copied unchanged from apache.org and creativecommons.org.
  - `LICENSING.md` replaces the license proposal. It says which license covers what, and how the names, patents, contributions, the specification's examples and third-party material are handled.
  - Decision record 0029 is accepted, and keeps the proposal's reasoning, alternatives and dependency check.
  - Every `package.json` declares `"license": "Apache-2.0"`, and every package stays private.
- **Publication.** The site has been live at `businessagentprotocol.org` since 2026-10-11 (the runbook's deployment record).
- **Site and documents.**
  - The site publishes the license texts, unchanged, as plain text at `/licenses/apache-2.0.txt` and `/licenses/cc-by-4.0.txt`, and its governance page shows `LICENSING.md` as its License section.
  - The footer, the home page, the FAQ, the governance, contributing, security and conduct documents, the notices, the threat model, the runbook and the specification's status no longer say that no license has been granted, or that publication waits on a decision.
- **Owner decisions.** `GOVERNANCE.md`'s publication blockers are now owner decisions, each with its state. The license is decided, and the Cloudflare account and zone are confirmed. The repository's home, a private reporting channel, counsel's reviews of how PAP is named and of the Source Serif 4 font notice, and the owner's review of all public copy are open, and none of them holds back the site. Whether the site sends HSTS is still open too (the runbook).
- **The project controls `businessagentprotocol.org`** (decision 5): section 12.5 now says so, open question 13, which waited on it, is answered and its row is gone from section 17.3, and decision record 0022 records the confirmation. A note under the table of 17.3 says that questions 13 and 22 were answered on 2026-10-11, and that the other questions keep their numbers.
- **The specification says what changed.** Section 1.4 dates draft 0.1 2026-10-10, revised 2026-10-11, and its document history lists this revision's normative changes by section.
- **Portable validation.** An implementation in any language reaches the same (code, path) pairs as the reference implementation:
  - A lone surrogate in a string or a member name is `invalid_json` at `""`, at decoding, before a repeated member name is looked for (spec 2.1, 8.2 and 8.4, checklist C-1). `parseJson` scans the text, so it also finds one that a repeated member name drops. `validateObject`, `validateEnvelope`, `validateBundle` and `validateDiscovery` apply the same check to a value given to them already decoded, so `validateBundle` no longer throws when it digests one. Two fixtures, `json-lone-surrogate.json` and `json-lone-surrogate-name.json`, hold the rule.
  - No validator throws on input of any depth, or on any kind of value. A value given already decoded that holds anything else JSON text cannot produce (a value that contains itself, an object of a class such as a `Date` or a `Map`, a number that is not finite, `undefined`, a hole in an array, a function, a symbol or a bigint) is `invalid_json` at `""` before any check, as decoding would refuse it; an object that two parents share is fine. `canonicalize` keeps its own stack, so any depth `JSON.parse` accepts canonicalizes, to the same output, and it refuses a value that contains itself. The command line looks for a record's latest time the same way, and the generated validators compare `uniqueItems` entries with an equality of their own instead of Ajv's fast-deep-equal, which recursed once per level and is no longer inlined (the third-party notices, and a dated note in decision record 0003). `validateBundle`, `enna validate` and `enna verify` now check a record whose receipt holds an extension nested 100,000 deep, where they threw. A hole in an array is refused where it is found, by the walk and by `canonicalize` alike, so an array's length alone costs nothing: `new Array(2 ** 32 - 1)` is refused at once instead of exhausting the heap. Size stays the caller's to bound, as spec 16.2 leaves it and open question 15 asks: a canonical form longer than the longest string the engine holds, as an object shared at many places can give, still makes `canonicalize`, and so `validateBundle`, throw.
  - A byte order mark before the JSON text is `invalid_json` at `""`, whether a document is checked as bytes or as text (spec 2.1, 8.2 step 1, checklist C-1): `parseJson` no longer lets its decoder drop one. Decoding reports one `duplicate_member`, at the first member in text order whose name repeats an earlier one (spec 2.1 and 8.4). The fixture format gains `input_base64`, the exact bytes of a `json` case, and fixtures hold bytes that are not UTF-8 four ways, the byte order mark as bytes and as text, and the first-repetition rule twice.
  - Every object closes with `additionalProperties: false`, and each object's own `properties` names every member it can carry, the shared members as `true` where the object does not narrow them, so a member is unknown exactly when that list does not name it, whatever a validator does with annotations (spec 2.13). No schema uses `unevaluatedProperties`, and no fixture's expected list changed.
  - `format` is an assertion, and the specification states each rule it uses (spec 2.14): `date-time` is the timestamp rule of 2.7, which now says how a leap second compares, and `uri` is RFC 3986's `URI` with a hierarchical part that is not empty. The domain rule is stated in 2.10 and carried by the pattern instead of `format: hostname`, and the URL patterns allow printable ASCII instead of `[^\s]`, so ECMA-262, RE2 and PCRE engines agree on every pattern once `$` matches only at the end. New fixtures hold a date that does not exist, a misplaced leap second, a domain label that starts with a hyphen, a domain that ends with a line break, a URL with a character outside ASCII, and a contract created during a leap second.
  - The reference implementation checks `uri` as section 2.14 states it, with the `URI` production of RFC 3986, Appendix A, without `path-empty`, which the validator generator compiles into the generated validators; 2.14 now names the production. ajv-formats' `uri`, used before, accepts text the RFC refuses, such as `https://a:b:c/` or `x:/[::1]`, by reading it as a path. Four fixtures hold a port that is not digits and a bracket in a path, each in a `uri` and in an HTTPS URL, and ajv-formats now supplies `date-time` alone.
  - The domain that may prefix a token, an action name, an extension name or a profile identifier follows the label rule of 2.10 (spec 2.2, 2.4, 2.10, 2.12 and 2.14), so a prefix such as `-a.example/` is refused; the value's own length limit, not a domain's 253 characters, bounds it. Spec 2.13 says that `extensions` names its members by the pattern of 2.12. Four fixtures hold the rule, one for each kind of prefixed value.
  - A new test file, `schemas.test.ts`, checks that every closed object names its members, that the patterns alone decide domains and URLs when formats are ignored, and that no pattern uses a class escape, a lookaround or a backreference.
  - Decision record 0043 records these choices; 0002 notes that its first reason no longer applies, and 0012 points to 0043. The third-party notices name the format the validators take from ajv-formats.
  - Step 7 of section 8.2 names the receipt time order of 6.3, which `validateObject` already applied, and checklist item P-13 includes it.
  - A record is decoded as one document before its objects are checked (spec 8.2 and step 1 of 9.1), so a lone surrogate anywhere in it is `invalid_json` at `""`, as `validateBundle` reports it.
- **Every integer stays within 2^53 - 1** (spec 2.15): every integer member of every schema now has a maximum of 9007199254740991, as the durations already had, so a `revision`, a delegation `depth`, a grant's `count`, a test report's counts or a measurement window that some platforms would read inexactly is a `schema_violation`. Section 2.15 states the bound once, and 2.7 refers to it. A test holds every integer in every schema to it, and three fixtures hold a measurement window, a business impact window and a revision of 9007199254740992.
- **Each invariant applies on its own** (spec 9.1, step 5): a stop inside an invariant, such as "nothing else is checked for this effect", ends that invariant's checks for that object or effect only. `validateBundle` now checks the context an effect cites for freshness (9.8) even when its actor is not the binding's grantee, and for reach (9.4) even when its binding is not in the record, where the contract's scope alone reaches. Six fixtures hold reach, freshness and counting for both kinds of effect, and the unit test that pinned the old reading follows the ruling.
- **An approval left out by its kind is missing, not out of scope** (spec 9.7, step 4.4): a PAP confirmation for a grant that reaches beyond the contract's scope now leaves `approval_missing` at `/<i>/authority`, as a standing approval for a production deploy already did; `approver_out_of_scope` stays for who gave an approval and the role it was given under. A fixture holds it, and the bundle test that pinned `approver_out_of_scope` follows.
- **The receipt chain follows creation order** (spec 9.14): "a second receipt", "already names" and "the first receipt" take receipts by `created_at`, ties in record order, as 9.9 does, whatever order a record lists them in. `validateBundle` walked them in record order, and now takes creation order. `enna verify` checked the chain in the order the file listed it; it now follows the links from the first receipt, taking creation order only for receipts the links leave unordered, so it agrees with `validateBundle` on receipts created in the same instant. It finds each link through a map from a digest to the receipts that name it, so a chain of 5,000 receipts orders in milliseconds. Three fixtures list a late receipt first, each with a chain fault whose path the two orders would place differently.
- **`poppy.json`'s link is a string rule** (spec 12.5): its `url` equals `https://{host}/.well-known/enna.json`, where `{host}` is the requested host without one leading `www.`, alone or after `www.`, with the scheme and host compared without regard to ASCII case and everything else exactly. `validateDiscovery` parsed the URL, which dropped an explicit `:443` and skipped userinfo; it now compares strings. Fixtures hold `:443`, userinfo and a trailing dot on the host.
- **Hosts compare without regard to ASCII case only** (spec 12.2 and 12.5): `validateDiscovery` folded the domain rule's hosts and `poppy.json`'s domain as Unicode does, so a host that starts with U+212A KELVIN SIGN matched `k`; it now folds `A` to `Z` alone, as the link already did. Three fixtures hold a KELVIN SIGN: in the requested host, in `poppy.json`'s domain, and in its link.
- **Conformance fixtures.** Every invalid fixture names, in a new member `section`, the section of the specification whose rule it exercises, and the case format defines the member. A test holds each to a numbered heading of the specification.
  - 236 new invalid cases, those the other bullets name among them, 296 in all, give the conditions that the rules of sections 2 to 9, 12 and 14 name a case each: every list in which an identifier may not repeat, every decision, outcome and time rule of a receipt, every kind of unresolved reference, the conditions of the tenant boundary, separation of duties, delegation, idempotency, the receipt chain and premature or unauthorized acceptance, every rule the Controlled and Assured levels add to a record, the check order of objects, envelopes, records and discovery documents, where a check stops and where it reports again, and the link between `enna.json` and `poppy.json`.
  - The README and the runbook count what a production build now holds: 321 fixture and vector files, and 418 files in all. The README also gives the current counts of the check (1,634 package tests and 146 site tests, Biome over 487 files), the 12 header rules `wrangler dev` parses, and section 17's 21 open questions, and says which of its summaries date from 2026-10-10.
  - Eight new valid fixtures hold rules that an implementation too strict would break: an acceptance by an agent the owner delegated it to; a refusal and a replay, both after acceptance, that cost and count nothing, the replay needing no live authority and no fresh context; an unknown outcome settled under its key; exact decimal sums that land on a limit of 0.3; an effect, an approval and fresh context on the inclusive edge of their instants; the variations a check tolerates (an unknown profile, a non-critical extension, another project, a principal's display, a person who operates the agent, second revisions of a binding and an evaluation); the owner approving their own contract at the Essential level; and a `poppy.json` link written in capitals with `www.`. The 9.13 delegate cases now start from the delegated-acceptance record.
  - `bundle-chain-broken.json` linked to a digest that regenerating the valid fixtures had left behind, so it tested a link to no receipt rather than the fork its description names; it now forks the chain.
  - Section 8.4 now lists `/<i>/policy/sod_threshold` among the paths of `currency_mismatch`, as 5.7 and 9.6 state.
- **Conformance kit.** The protocol package holds test vectors and catalogs beside its fixtures, and its tests check each file:
  - `fixtures/vectors/canonicalization.json`: 21 cases of RFC 8785, their canonical forms written by hand from the RFC, with the digest of each, or the error a lone surrogate gives instead. The package's `canonicalize` agrees with every one.
  - `fixtures/vectors/digests.json`: the canonical form and digest of each distinct execution receipt and authority binding in the valid fixtures, one case per digest: the first found, taking the fixtures by file name and each in record order, so 18 receipts and 6 bindings stand for the 54 and 13 the fixtures hold. A test rebuilds it from the fixtures, with `vitest -u` to rewrite it.
  - `catalogs/`: the error codes, the lifecycle with its A2A and PAP mappings, the identifier prefixes, the profiles and the core extensions, each with the section that defines it. A test rebuilds each from the package's constants.
  - Biome leaves the files the tests write as they write them.
  - The site publishes the fixtures, the vectors and the catalogs byte for byte under `/spec/v0.1/fixtures/` and `/spec/v0.1/catalogs/`, never the fixtures' README, and `/spec/v0.1/manifest.json`, which lists every file served under `/spec/v0.1/` but the pages, with its media type, SHA-256 and license. The manifest holds no time, so the same files give the same manifest. `public/_headers` serves the three as `application/json`, and the reference page and the specification's index list them beside the schemas.
  - The site's tests check that the published files are the package's, byte for byte, and every one of them; that the manifest's hashes, media types and licenses match the build, and that it misses no file; and that every `defined_in` names a page the site builds. The release suite checks the new media types as `wrangler dev` serves them.
  - A new informative page, the conformance kit (`spec/v0.1/conformance-kit.md`, at `/spec/v0.1/conformance-kit/` with its Markdown alternate), follows the checklist. It says what each file holds, how to run the fixtures and check a result against them, how a JSON Schema validator's output maps to (code, path) pairs, and where formats, patterns, lone surrogates and decoding need care, and it describes the vectors, the catalogs and the manifest. The case format moved there in full from the fixtures README, which now points to the page and to the published fixtures.
  - The kit page's mapping from a schema validator's output says to collect every failing assertion of a step, where most validators stop at the first, and to apply a profile's schema only once the object's own schema passes; `object-schema-before-profile.json` holds that stop (spec 8.2, steps 5 and 6).
  - Section 13.1 says the fixtures are published with the draft under `/spec/v0.1/fixtures/`, names the kit page, and no longer says that no license has been granted. Open question 22, how the fixtures are versioned and published and under which license, is answered: with the draft, under `/spec/v0.1/`, under Apache-2.0. Its row is gone from section 17.3, and the other questions keep their numbers. `GOVERNANCE.md` says a claim by Further Machines would be checkable against the published fixtures, and `CONTRIBUTING.md` says how to rewrite the derived files.
  - The kit page's resealing step 2 sets each receipt's `integrity.previous_digest` to the digest of the receipt just before it in the array.
- **Checklist item C-16** (spec 2.14): formats are assertions, checked with the draft's rules (2.7 and 2.14), never with a validator's own, and patterns match as ECMA-262 matches them, `$` only at the very end of a value.
- **References** (spec 18): ECMA-262, 17th edition, a normative reference for the regular expressions of 2.14; and, informative, RFC 4648 and RFC 6902, for the conformance kit's `input_base64` and patches, POSIX (IEEE Std 1003.1-2024, with the address of its base definitions), for the leap second of 2.7, and RE2 and PCRE2, for the `$` of 2.14, identified by what their pages say: RE2's wiki page as last edited on 2023-08-11, and PCRE2 release 10.48, whose page is stamped "Last updated: 25 October 2025". The research notes on standards and tooling record what ECMA-262, POSIX, RE2 and PCRE2 said when they were read on 2026-10-11, and section 18 no longer dates every check 2026-10-10.
- **Names break before a hyphen, never after it, on every page.** A line that ends on a hyphen reads as hyphenation, so a name in code, an address or a path in running text or a table cell, and an Internet-Draft name may break before a hyphen, and each hyphen stays on one line with the character after it, as the Chicago Manual of Style breaks URLs. So do the names the worked examples and the playground take from the runtime: actions, references, identifiers, idempotency keys, workspaces, resources and the evaluator. Short codes such as `SHA-256` and `C-11`, dates such as 2026-10-11 and other numbers joined by a hyphen, such as FIPS 180-4, and the license identifiers `Apache-2.0`, `CC-BY-4.0`, CC BY 4.0 and CC BY, which the License section of the governance page split, stay whole in running text, in code and in tables. One function, `textParts` in the site's names module, breaks the text of the pages made from Markdown and of the views the examples and the playground share; the pages written by hand use the same markup, and a test holds every built page to the rule. The breaks are markup, so a copied name reads as written; ordinary hyphenated words break as text does, and code blocks wrap as they are.
- **The Implement page is for any language.** `/implement/`, now "Implement Enna", works from what the site publishes: what the draft publishes and under which license, a Consumer built in the order of section 8.2, digests, the conformance fixtures, an Enforcement point or an Evaluator, and the first profile, each linked to its section and to fixtures that test it. The TypeScript reference comes last, marked optional, with its examples and the command line's own output. The calls to action that lead there say "Implement Enna". Its schema step, as the kit page says, has a validator report every failing assertion and applies a profile's schema only once the object's own schema passes, and a Producer's objects must pass its first four steps.
- **Tests.** The site's tests check that both license texts are served byte for byte as plain text, that every page's footer names both licenses and links to the License section, that no page says publication waits on a decision, and that none says no license has been granted.

## 0.1: first draft, 2026-10-10

When this draft was written, nothing was published: publication waited on the seven owner decisions then listed in `GOVERNANCE.md`, and no license had been granted (the license proposal). Both changed on 2026-10-11 (above).

### Specification (`spec/v0.1/`)

Eighteen sections, a conformance checklist and a README that states the folder's file and front matter contract.

- **Overview and conventions.** Status and terms; encoding, names, kind-prefixed identifiers, revisions, references, UTC timestamps, exact decimal money, absent, null and unknown, scope, principals, extensions (`critical` is honoured on objects, envelopes and discovery documents), versions, and digests with one statement of what they show: a verified chain shows the receipts are consistent with each other; a digest shows a record is unchanged only against a digest you already trust, and an edited record can be resealed.
- **The five objects.** Work contract, scoped context manifest, authority binding, execution receipt and evaluation result.
  - Stakeholders may be approvers, acceptors, consulted or informed. The contract gives each approver the roles they may approve under (`stakeholders[].approver_roles`), and a role a grant requires counts only for the approver the contract gives it to.
  - An approval covers the contract revision and `binding_digest`, the digest of the whole binding except its approvals and revision bookkeeping, so a change to any grant, the window, the policy, the revocation, the delegation or the credentials makes it stale.
  - A grant may set a quorum (`conditions.min_approvals`); a binding may lower, never raise, the separation-of-duties threshold (`policy.sod_threshold`); revoking a binding revokes every binding delegated from it.
  - A receipt may name the failed attempt it retries (`action.retry_of`) and the PAP operation its action was done for (`trace.transport.pap_operation_id`); a replayed permit may omit `authority`.
  - Evidence may carry `measurements`, each with a name, an integer value, a unit and `illustrative: true` when the figure is stated rather than measured.
- **Envelope and errors.** The order of validation, the error response, and a catalog of 39 stable error codes with the JSON Pointer paths each is raised at, among them `quorum_not_met` and `unsupported_profile`.
- **Invariants.** Thirteen cross-object invariants over the record of one contract revision, the rules each assurance level adds to a record (9.16), the rule that records describe the past, and what the invariants do not check.
  - Only the binding's grantee may act; another actor is `authority_insufficient`.
  - Only approved, unrevoked authority extends the tenant boundary, and an effect may use only context the contract's scope or its own binding's approved grants reach, roles and quorum met.
  - An idempotency key identifies one action: its name, resource and payload.
  - Acceptance comes last, from the owner, a named acceptor, or an agent the owner delegated it to through a binding the owner approved.
- **Lifecycle.** The business lifecycle as a normative transition table, with its mappings to A2A 1.0 task states and PAP 0.1 operation states, and tests that hold tables 10.3, 10.6 and 10.7 to the data.
- **Execution semantics.**
  - The gate before each action. It looks up the idempotency key first; a refused attempt reserves nothing. It refuses every action, replays included, unless the contract is `authorized` or `executing`, and it fails closed over a record whose shape, chain, timestamps or level rules fail anywhere in it. Once recorded costs, across every revision, pass the contract's limit, it refuses every further action whatever the action quotes.
  - Idempotent replay and conflicting reuse, unknown outcomes and reconciliation, retries, partial failure, cancellation, stale context, scope changes, approval invalidation and recovery, what is recorded versus enforced, and no promise of exactly-once execution.
- **Discovery.** The document `/.well-known/enna.json`, a proposed draft convention not registered with IANA, with an optional `description`, the `enna_domains` issuer cross-check and the two-way link with `poppy.json`.
- **Conformance.**
  - The Producer, Consumer, Enforcement point and Evaluator classes, and the Essential, Controlled and Assured levels, chosen by risk, each requirement naming the party it binds.
  - Claims are self-declared, and the conformance fixtures define expected outcomes.
  - The reference implementation's claim: Essential. By default it applies the Controlled level's record rules, but it does not meet the Controlled level.
- **The `software-delivery/0.1` profile.** An Enforcement point refuses a contract or binding whose profile it does not know, and the production deploy rule applies when the contract, the binding or the receipt declares the profile.
- **Relationships and mappings.**
  - PAP 0.1, the base and the bridge:
    - a confirmation counts only for grants inside the contract's own scope;
    - at Controlled and above, a business approver must also approve;
    - the rollup takes the operation's ID and selects the receipts that carry it, by each action's final receipt;
    - `extensions.pap.revision` names the operation revision.
  - Also MCP 2026-07-28, A2A 1.0, OAuth (RFC 8693, 9396 and 8707), AuthZEN 1.0, in-toto and JWS, W3C PROV and OpenTelemetry, each marked implemented and tested or proposed.
- **Security and privacy.** Including the MCP and PAP boundaries (16.5).
- **The rest.**
  - Non-goals, limitations, 23 open questions and the alternatives considered.
  - References with versions and check dates, PAP's pages pinned by SHA-256.
  - A conformance checklist.

### Schemas and protocol package

- **JSON Schemas.** Ten JSON Schema 2020-12 schemas: the five objects, the envelope, the error response, the discovery document, the shared definitions and the software-delivery profile.
- **`@enna/protocol`: validation.**
  - Validators are compiled ahead of time into a self-contained module that opens with a notice naming the code it inlines. JSON is decoded strictly.
  - `validateObject` and `validateEnvelope`.
  - `validateBundle` returns a `BundleResult` that says when the check stopped at the record's shape, and applies the invariants and the level's record rules.
  - `validateDiscovery`.
- **`@enna/protocol`: helpers.**
  - RFC 8785 canonicalization and SHA-256 digests: `digest`, `receiptDigest` and `bindingDigest`.
  - Money, time and identifier helpers.
  - The lifecycle and the A2A and PAP state tables.
  - The PAP 0.1 data mappings.
  - `finalReceipts`, `idempotencyMismatch`, `namedAcceptor` and `PROFILES`.
- **Conformance fixtures.** 13 valid documents and 60 invalid cases, each with its exact expected (code, path) list, and a README that defines the case format.

### Reference runtime, mock adapters, scenarios and command line

- **`@enna/reference-runtime`.** A simulated runtime with:
  - an injected clock and a seeded random source, so the same inputs give byte-identical output;
  - a policy gate that decides each action with `validateBundle` over the record so far, before any simulated effect, and lists every check it made, check 0 (record shape, time, profile and lifecycle) first;
  - in-memory idempotency and revocation stores, labeled demonstrations;
  - receipts sealed and chained per contract revision;
  - an evaluator that never treats unknown as a pass, taints both sides of a broken link, and derives what a criterion rests on from the evidence it cites;
  - and an orchestrator with the lifecycle, gated actions, an effect log and the fault mechanics.
- **`@enna/mock-adapters`.** Two mock agent runtimes behind one `RuntimeAdapter` interface: Relay, an event bus with subscriptions and leases, and Docket, a task queue with dispatches, each with records of its own.
- **Scenarios.**
  - Two scenarios at Harbor & Pine, a fictional company: a competitor's launch that ends in an accepted technical preview, and a paid customer without access, whose regulated second customer forces a contract revision with a named compliance approver.
  - Each is a definition as data, with its constraint controls, nine fault switches and an illustrative sandbox latency (840 ms and 1,200 ms) carried on its evidence.
  - `runScenario` runs them.
- **The `enna` command.**
  - `validate`.
  - `verify`, which checks a bundle's chain and record and claims no level. It says what a level asks of a record and leaves the record checks to say whether this record has it, says what a record cannot show, calls a chain consistent only when it is, and with `--anchor` it accepts a chain only if it ends at a digest you already trust.
  - `run`, which prints a scenario's timeline and labels every receipt unsigned.
- **Examples.** The scenario definitions, four full golden runs, a matrix of 72 runs with their digests, and five bundles for `enna verify`, two of them tampered, regenerated by `npm run examples:update`.

### Site and playground

- **Stack and pages.**
  - An Astro 7 static site, with one React island, self-hosted fonts and plain CSS in cascade layers.
  - Light and dark themes with no flash, reduced motion, skip links and visible focus, with text equivalents for every diagram.
  - Pages: the home page, Why Enna, How it works, the specification, the playground, examples, the implementation guide, the reference, governance, security, the threat model, the FAQ, notices and a 404 page.
- **Specification and reference pages.**
  - The specification is rendered from `spec/v0.1/`, with links between sections rewritten to routes, a page for the conformance checklist, and a Markdown alternate of every page.
  - The schemas are published byte for byte under `/spec/v0.1/schemas/`.
  - `/reference/` lists every runtime export of `@enna/protocol` with its signature, read from the source, and compiled examples.
- **The playground.**
  - It runs the reference runtime in the browser, under a persistent label: "Simulation: fictional company, mock agents, no real actions".
  - Every visitor change goes through the real `runScenario`: both scenarios, the runtime switch, the constraint controls, the nine faults, the seed and the clock. A refused input shows the runtime's refusal on its field.
  - It shows the story first, then the outcome, the run at a glance, the lifecycle, the authority each agent holds, and a readable timeline.
  - A five-object inspector shows each object readable beside its JSON, with the evidence (the illustrative latency included) and the record checks.
  - Replay steps through a run; reset returns to the defaults.
  - Roles and states read as words (an approver "approves as product manager"), and an action name, receipt ID or artifact address breaks before a slash, an underscore or a period rather than inside a word, as names do in the specification's tables.
  - The 60-second guided walkthrough is one script kept with the site's code, and the site's tests check it against real runs. Any change a visitor makes ends it.
- **Content pages.**
  - `/examples/` shows both scenarios with the objects the runtime produced when the site was built.
  - `/implement/` shows examples that the type check compiles and the build runs, with their output and the command line's own output.
  - Governance, security, the threat model and the notices are rendered from the repository's own files.
  - `/.well-known/enna.json` is published from the protocol package's fixture.
- **Production and security.**
  - A non-production build is noindex, with a `robots.txt` that disallows everything, and a test holds it to that.
  - `public/_headers` sends a strict Content Security Policy without `unsafe-eval`, and the security headers.
  - The guide's examples run in a pinned environment, so a build gives the same bytes from any shell.
  - The MDX integration is not used (decision 0034).

### Release gate and deployment package

- **`npm run verify`.** It runs the fast check, the production build, a Wrangler dry run, and a Playwright suite over the build as `wrangler dev` serves it, offline. The suite checks:
  - every page, in light and dark at 1440 and 400 pixels, once its islands are ready: status, console, requests, and axe with touch targets;
  - four playground states;
  - internal links, anchors, canonical URLs and the sitemap;
  - the headers `_headers` declares;
  - dead links and buttons;
  - secrets, placeholders and rendering leftovers, terminal escape codes among them;
  - requests to other origins, in the built files and at run time;
  - that the build and the generated examples are byte for byte reproducible. The examples check hands Biome each file to read rather than a pipe, and stops it after 30 seconds, so it cannot stall the gate.
- **The Wrangler configuration.** An assets-only Worker named `businessagentprotocol`, with no script, binding or route.
- **Root scripts.** `build:production`, `dev`, `preview` (a fresh production build under `wrangler dev`), `preview:dist` (the verified build as it is), `deploy:dry-run` and `test:release`.
- **Documents.**
  - `README.md`, with every command as run.
  - The deployment runbook, with the owner's decisions and steps for a deployment that had not happened yet, among them whether to turn off the `workers.dev` address and Preview URLs.

### Decision records

| Records | Subject |
| --- | --- |
| 0001 to 0009 | TypeScript and its types, the generated validators, the CLI, the gate, the mock runtimes, the scenarios, receipts across revisions and the golden outputs |
| 0010 to 0029 | The specification's design, from building on PAP by reference to the license |
| 0030 to 0034 | The site: the playground on the real runtime, the one walkthrough script, documents rendered from their files, pages built from the packages, and no MDX |
| 0040 | The release gate and the deployment package |
| 0041 | Approvals cover the whole binding |
| 0042 | The contract's state gates every action |

### Repository

- `GOVERNANCE.md`, with its seven publication blockers, and `SECURITY.md`, `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, the license proposal and the threat model.
- The third-party notices. They cover the MIT code the generated validators inline, and the code and fonts the site ships, with each font's notice read from the font files the site serves and every statement of Source Serif 4's holder. Tests hold the file to the installed packages and to the font files.
- Research notes with their sources.
- Tests:
  - 1,139 tests in 29 files, with 100% statement, branch, function and line coverage of the three packages;
  - the site's own 114 tests;
  - tests that validate every JSON example in the specification and hold its pages to their front matter contract.

### Final reviews

Three reviews of the branch at `77d455d` found 4 Critical, 29 Important and 48 Minor findings. All of them are fixed in this draft:

- review A covered the protocol, runtime, adapters, examples and command line;
- review B covered the site, playground, release gate and deployment package;
- review C covered the specification and the repository's documents.

Among the fixes:

- **The gate.** It failed open when another object of the record was malformed. It now fails closed, keeps refusing once recorded costs pass the contract's limit, and checks the contract's state before every action.
- **Approvals, roles and grantees.** Approvals now cover the whole binding, a role counts only when the contract gives it to the approver, and only the grantee acts. Unapproved bindings no longer widen the tenant boundary, and acceptance can be delegated only through a binding the owner approved.
- **PAP and keys.** A PAP confirmation is scoped to the contract's tenant, and at Controlled a business approver must also approve. The PAP rollup's input is defined. An idempotency key covers the action's name and resource as well as its payload.
- **Claims and wording.** The digest wording, the level claims and `enna verify`'s output no longer say more than a record shows.
- **The release gate.** A build is the same from any shell, and the gate now sees the rendered playground and touch targets. The non-production build is tested, the runbook previews the verified build, and the README, runbook and decision 0040 report current results.
- **The playground.** The walkthrough never narrates a run that is not on screen, and replay shows only what had happened. Refusals are tied to their fields, focus returns to the control that opened a view, and the sandbox latency is shown.
- **Notices.** The Source Serif 4 notice follows the font files, and counsel's review of it became a publication blocker.

### Final re-review

A re-review of those fixes found one Important and six Minor findings, and three findings only partly fixed. All are fixed in this draft, the receipt view's one in part:

- **Context needs approved authority.** An effect may use another workspace's context only through a grant of its own binding with approved authority behind it: approvals that count, the grant's roles held and its quorum met. The quorum now also gates the tenant boundary. Two conformance fixtures hold the role-gated and the quorum-gated grant, and scenario B's Fernhill Health activation is refused for its context as well as its approval until its compliance approver signs.
- **The gate and the runtime.**
  - Among an agent's covering bindings the gate prefers one that can act, then the most recently issued, and names it in its decision, so 11.10's recovery from a revoked binding works.
  - It refuses every action over a record whose time or level rules fail anywhere in it, as spec 9.1 and 11.1 now say.
  - A planned action refused for no business reason, such as a broken chain or an unknown profile, no longer blocks the contract or throws.
  - A used key while the contract is idle is decided by the key lookup: a replay is refused for the lifecycle alone, and a conflict for `idempotency_conflict` as well.
- **PAP.** `papResultFromReceipts` and `toPapOperationState` take the operation's ID and select its receipts themselves.
- **Wording.** Spec 9.11 reports one idempotency conflict, at the first member that differs. The playground's receipt view calls the receipts consistent with each other only when it is given a chain that verifies, and the playground does not yet pass it one. The README and this changelog report the current figures.

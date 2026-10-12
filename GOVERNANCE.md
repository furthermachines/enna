# Governance

Enna, the Business Agent Protocol, is a **founder-led draft**. This file says who decides today, what Further Machines' role is and is not, how decisions are made and recorded, and how that could change. It describes no consortium, members or community, because none exists.

## Who decides today

Further Machines started Enna. The draft's owner, who leads it for Further Machines and is called the owner in these files, decides what goes into it. There are no other editors, members, sponsors or contributors yet.

The owner's decisions so far include: the name, Enna; building on the Personal Agent Protocol; publishing a discovery document of Enna's own; keeping the protocol vendor-neutral; on 2026-10-11, the license and publishing the site; and, on 2026-10-12, publishing the specification in a public repository. Some decisions are still open (below).

## Further Machines' role

- Further Machines is the founder of the draft, and it **intends to implement** Enna, in work that maps an organization's systems, coordinates agents and evaluates their results. It does not conform to Enna today and claims no conformance. If it ever does, its claim will be self-declared, published the same way as anyone else's, and checkable against the same conformance fixtures, which the site publishes with the draft.
- Enna is vendor-neutral. The specification has no normative dependency on any Further Machines product or service, and implementing it, testing against it or using it needs nothing from Further Machines.
- Further Machines' implementation gets no special status: no reference to it in the normative text, no privileged conformance claim, and no say over other implementers' claims.

## How decisions are made

- A design decision is recorded in the private build repository as a numbered decision record with its context, the decision, its consequences and the alternatives considered. Superseding a decision takes a new record that says why.
- A change to the protocol is made as one change across everything it touches: the schemas, the fixtures and their expected errors, the reference implementation, the tests, the specification text and `CHANGELOG.md`.
- Unresolved design questions are listed in the specification's open questions (section 17.3) until they are decided. The owner's open decisions are listed below, never hidden as to-do notes.
- Normative text uses BCP 14 keywords, and every requirement names the party it binds.

## Versions and change control

- Drafts are numbered 0.x. Before 1.0, any draft may change incompatibly with the one before; the specification's status section and `CHANGELOG.md` say what changed.
- Error codes are stable within 0.x and never reused for another meaning.
- What makes a version 1.0, and the compatibility rule after it, are open questions.

## How governance could evolve

These are possibilities, not commitments. Each would be decided by the owner and recorded as a decision:

- naming further editors, including people outside Further Machines;
- a public review period for changes, held in the public repository's issue tracker, which already takes questions and proposals;
- accepting pull requests to the specification text, with a contributor patent commitment, which its license, CC BY 4.0, does not give (`LICENSING.md`);
- moving the specification to a neutral home, such as a foundation or a standards body, if independent implementations and real interest make that worthwhile.

The criteria for any of these steps have not been decided.

## Owner decisions

These are the owner's to decide. Each is listed with its state, and an open one stays listed until it is decided. The open ones do not hold back the site, which has been published at `businessagentprotocol.org` since 2026-10-11.

1. **The license**: decided on 2026-10-11. Apache-2.0 for code, schemas and fixtures, and CC BY 4.0 for the specification and documentation (`LICENSING.md`).
2. **The GitHub organization and repository** for the public repository: decided on 2026-10-12. The specification is public at [https://github.com/furthermachines/enna](https://github.com/furthermachines/enna): the Markdown of its text, its schemas, catalogs, fixtures and test vectors, and `manifest.json`, as the site publishes them under `/spec/v0.1/`, with the project's documents and `CHANGELOG.md`. Questions and proposals go in its issues; it does not merge pull requests. It is generated from the project's build at each release. The build repository stays private: it holds the TypeScript packages, the site's code and the documentation: the decision records, the runbook and the research notes. Outside contributions to it cannot be accepted.
3. **A private reporting channel** for security issues, and for conduct reports: partly decided on 2026-10-12. Security reports go through GitHub's private vulnerability reporting on the public repository (`SECURITY.md`). The email channel, for security and conduct reports, is still open, so conduct reports have no channel yet (`CODE_OF_CONDUCT.md`).
4. **Counsel review** of how the Personal Agent Protocol is named and attributed: open. It covers the abbreviation PAP, which is this draft's own: PAP's site does not use it, and the draft builds it into identifiers (the `pap` extension, `pap_confirmation`, `pap_account`, `pap_operation_id` and the discovery member `pap`), so renaming them later would be an incompatible change.
5. **The Cloudflare account and zone** that serve the site, and the project's control of `businessagentprotocol.org`: confirmed on 2026-10-11.
6. **The owner's review of all public copy**: open.
7. **Counsel review of the Source Serif 4 font notice** and its Reserved Font Name ‘Source’, or a change of font: open. The font files, the npm package and the typeface's upstream projects state its holder differently ([the site's third-party notices](https://businessagentprotocol.org/notices/)).

Choices that concern only how the site is hosted, such as whether it sends HSTS, are the owner's too, and are recorded with the deployment in the private build repository's runbook.

## Related files

`CONTRIBUTING.md` (how changes are made), `SECURITY.md` (reporting), `CODE_OF_CONDUCT.md` (conduct), `LICENSING.md` (which license covers what) and `CHANGELOG.md` (what changed).

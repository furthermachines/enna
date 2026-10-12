# Security

## Reporting a vulnerability

Report a vulnerability privately, in one of two ways:

- **Through GitHub**, which is preferred, because it keeps the report and its fix together: [https://github.com/furthermachines/enna/security/advisories/new](https://github.com/furthermachines/enna/security/advisories/new). This is GitHub's private vulnerability reporting on the specification's public repository, and it needs a GitHub account.
- **By email**, to <security@businessagentprotocol.org>.

The project's owner reads both.

Say in a report:

- what is affected: a rule of the specification, with its section; the reference implementation, with the function or command; or a page of the site, with its address;
- what someone could do with the problem, and under which conditions;
- how to reproduce it, with the smallest document, record or input that shows it;
- whether anyone else knows of it.

How reports are acknowledged and handled is not written down yet.

**Do not post exploit details publicly.** Keep proof-of-concept code, step-by-step reproductions and anything that would help someone attack an implementation out of issues, pull requests, discussions and social media. Never report a security problem in a public issue.

The specification site's `/.well-known/enna.json` names the GitHub reporting form as its security contact.

## What is in scope

- **The draft specification** in `spec/v0.1/`: a rule that, followed as written, would let authority be forged, widened or used after it should have stopped; let one tenant reach another; let a record be changed without detection against a digest the reader already trusts; or let an unknown result count as a pass or an approval.
- **The reference implementation**: a validator that accepts what the specification says to refuse, or a check that passes when it should fail.
- **The specification site**, `businessagentprotocol.org`: anything that runs script it should not, loads from third parties, or leaks data about visitors.

The deterministic demonstration uses fictional organizations, mock agents and simulated effects. Its in-memory stores and unsigned receipts are documented limitations, not vulnerabilities.

## Supported versions

Only the current draft, 0.1, is maintained. It is a proposal for discussion and implementation experiments, not production infrastructure, and nothing about it is certified.

## Security model

- The specification's security considerations and threat model are in its [section 16](https://businessagentprotocol.org/spec/v0.1/security/).
- The threat model for the reference implementation, the demonstration and the site is published on the site, as [the threat model](https://businessagentprotocol.org/security/threat-model/); its source stays in the private build repository.

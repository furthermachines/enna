# Licensing

**Licensed on 2026-10-11, by the owner's decision.** Enna uses two licenses: the Apache License 2.0 for code and for the machine-readable parts of the specification, and Creative Commons Attribution 4.0 International (CC BY 4.0) for text. Decision record 0029, in the private build repository, says why.

## Which license covers what

| Material | License |
| --- | --- |
| Code: the packages, the site, the scripts, the tests and the examples of the private build repository, and its configuration files | Apache License 2.0 |
| The machine-readable parts of the specification: the JSON Schemas, the fixtures, and any catalog, test vector or manifest generated from the protocol package, wherever they are published, such as under `/spec/v0.1/` on the site | Apache License 2.0 |
| The specification text in `spec/`, the documentation, the Markdown files of both repositories and the site's prose | CC BY 4.0 |

Their SPDX identifiers are `Apache-2.0` and `CC-BY-4.0`. A Markdown file is text wherever it sits, so one inside a code folder is under CC BY 4.0. The site's pages are code under Apache-2.0, and the prose they show is under CC BY 4.0. In the public repository, [furthermachines/enna](https://github.com/furthermachines/enna), which holds the specification as the site publishes it, the JSON files under `spec/v0.1/` are under Apache-2.0 and the Markdown files are under CC BY 4.0, as its `manifest.json` lists for each.

## Names

Neither license covers the names Enna, Business Agent Protocol and BAP. The project claims no trademark in them, and does not claim that any of them is cleared or registered.

## Patents

CC BY 4.0 licenses copyright only, and says expressly that it grants no patent rights. Apache-2.0 carries an express patent license from each contributor, so the code, the schemas and the fixtures come with one. Whether contributors to the specification text make a patent commitment is an open governance question (`GOVERNANCE.md`).

## Contributions

Outside contributions are not accepted yet: the public repository takes questions and proposals as issues and does not merge pull requests, and the build repository is private (`GOVERNANCE.md`, decision 2). If contributions are accepted, inbound equals outbound: a contribution is accepted under the license that covers what it changes. Whether a contribution also needs a Developer Certificate of Origin sign-off or a contributor license agreement is not decided.

## Examples in the specification

A JSON example in the specification is part of its text, under CC BY 4.0. Many examples are also fixtures, and a document that is a fixture is available under Apache-2.0 as well.

## Third-party material

Code and fonts from other projects that the build ships keep their own licenses, listed with their notices in [the site's third-party notices](https://businessagentprotocol.org/notices/). The fonts, Archivo and Source Serif 4, are under the SIL Open Font License 1.1, and counsel's review of the Source Serif 4 notice is still open (`GOVERNANCE.md`, decision 7).

## The license texts

[`LICENSE`](LICENSE) is the Apache License 2.0, and [`LICENSE-CC-BY-4.0`](LICENSE-CC-BY-4.0) is the CC BY 4.0 legal code, each copied unchanged from apache.org and creativecommons.org. The site serves them as they are at `/licenses/apache-2.0.txt` and `/licenses/cc-by-4.0.txt`.

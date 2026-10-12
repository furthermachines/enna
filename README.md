# Enna: the Business Agent Protocol

**Draft 0.1 for discussion.** This repository holds the specification as its site, https://businessagentprotocol.org, publishes it: the text, the JSON Schemas and the conformance kit.

## What Enna is

Enna, the Business Agent Protocol (BAP), is a proposed, vendor-neutral standard for agent work done for organizations. It proposes five portable objects that travel with a piece of business work, from intent to accepted outcome, across tools and runtimes: a work contract, a scoped context manifest, an authority binding, an execution receipt and an evaluation result.

The hypothesis is that these five objects make agent work easier to hand off, inspect, evaluate and accept. This draft makes the hypothesis testable; it does not prove it. Enna builds on the Personal Agent Protocol by reference, and says how it relates to MCP, A2A, OAuth and other standards without replacing them.

Enna is a founder-led draft started by Further Machines, which intends to implement it. It is not ratified, registered or certified, it claims no adopters, and implementing it needs nothing from Further Machines. [`GOVERNANCE.md`](GOVERNANCE.md) says who decides and how that could change.

## What is in this repository

| Path | What it holds |
| --- | --- |
| `spec/v0.1/*.md` | The specification text, one file per page: the same Markdown the site serves beside each page, whose links to other pages work here too |
| `spec/v0.1/schemas/` | The JSON Schemas (2020-12) |
| `spec/v0.1/fixtures/` | The conformance fixtures, in `valid/` and `invalid/`, and the test vectors for canonical forms and digests, in `vectors/` |
| `spec/v0.1/catalogs/` | The draft's tables in JSON: the error codes, the lifecycle, the identifiers, the profiles and the extensions |
| `spec/v0.1/manifest.json` | Every other file under `spec/v0.1/`, with its SHA-256, media type and license |
| [`CHANGELOG.md`](CHANGELOG.md) | Every change to the specification, its schemas, the reference implementation and the site |

The schemas, fixtures, vectors and catalogs are the specification's machine-readable parts. The conformance kit's section, [`spec/v0.1/conformance-kit.md`](spec/v0.1/conformance-kit.md), says how to use them: what each file holds, how to run the fixtures against an implementation in any language, and how to check a download against the manifest.

## How this repository is made

It is generated from the project's build at each release. Its files under `spec/v0.1/` are the site's, byte for byte: `spec/v0.1/<path>` here is `https://businessagentprotocol.org/spec/v0.1/<path>` there, and `manifest.json` gives the SHA-256 of each, so anyone can check either copy against it. Its `.gitattributes` keeps git from changing line ends, so a clone holds the same bytes on any system. The TypeScript reference implementation and the site's code are not here; they stay in the project's private build repository.

## Taking part

- **Questions and proposals** go in [issues](https://github.com/furthermachines/enna/issues). [`CONTRIBUTING.md`](CONTRIBUTING.md) says what makes a good one.
- **Pull requests are not merged here.** The repository is regenerated from the project's build at each release, and whether contributors to the specification text make a patent commitment is undecided ([`GOVERNANCE.md`](GOVERNANCE.md), "How governance could evolve").
- **Security problems** are reported privately, as [`SECURITY.md`](SECURITY.md) says, never in a public issue.
- **Conduct** in the project's spaces, this repository's issues included, follows [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md), which says that there is no channel for conduct reports yet.

## License

The JSON files under `spec/v0.1/` (the schemas, fixtures, vectors, catalogs and manifest) are under the [Apache License 2.0](LICENSE). The specification text, the Markdown files under `spec/v0.1/`, and this repository's documents are under [CC BY 4.0](LICENSE-CC-BY-4.0). `manifest.json` names the license of each file, and [`LICENSING.md`](LICENSING.md) says which license covers what, and how patents and the specification's examples are handled.

The names Enna, Business Agent Protocol and BAP are not licensed, and are not claimed to be cleared or registered.

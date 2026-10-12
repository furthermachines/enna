---
title: Software-delivery profile
description: The software-delivery/0.1 profile, with its seven actions, artifact and evidence kinds, the git digest rule and the rule that production deploys need their own grant and a non-standing approval.
order: 14
status: normative
---

# Software-delivery profile

The profile `software-delivery/0.1` adds the vocabulary and rules of software delivery to the core: branches, commits, pull requests, CI runs, preview and production deploys, and feature flags. Software delivery is the first domain this draft profiles, not the limit of its scope; work in other domains uses the core with actions named under the organization's domain ([section 14.6](software-delivery-profile.md)). The profile's schema is `profiles/software-delivery-0.1.schema.json`.

## 14.1 Applying the profile

- An object follows the profile when its `profile` is `"software-delivery/0.1"`. A Consumer that knows the profile MUST apply its rules to that object on top of the core rules ([section 2.14](conventions.md)). The schema rules apply to authority bindings, execution receipts and evaluation results.
- The production deploy rule ([14.4](software-delivery-profile.md)) applies to an effect when its contract, its binding or its own receipt follows the profile: an object that declares the profile cannot escape a rule of it because another object leaves the profile out.
- A Producer SHOULD set the profile on every object of a contract that follows it, so that every object is held to the same rules.

## 14.2 Actions

The profile owns five action namespaces: `repo`, `ci`, `preview`, `production` and `feature_flag`. Within them, only these seven actions exist; any other action name in those namespaces, in a grant or a receipt of an object that follows the profile, is a `schema_violation`.

| Action | Meaning | Usual resource `type` |
| --- | --- | --- |
| `repo.branch.create` | Create a branch. | `repository` |
| `repo.commit` | Add a commit to a branch. | `repository` |
| `repo.pull_request.open` | Open a pull request or merge request. | `repository` |
| `ci.run` | Run a continuous integration pipeline. | `pipeline` |
| `preview.deploy` | Deploy to a preview environment that is not production. | `preview_environment` |
| `production.deploy` | Deploy to production. | `production_environment` |
| `feature_flag.set` | Change a feature flag's value or targeting. | `feature_flag` |

The resource types in the last column are what this profile expects; the schema does not enforce them, and a Producer SHOULD use them so that grants and receipts from different tools match.

## 14.3 Artifact and evidence kinds

Artifacts that receipts of this profile record:

| `kind` | What it is | `ref` points to |
| --- | --- | --- |
| `branch` | A branch | The branch in the repository host |
| `commit` | A commit | The commit |
| `pull_request` | A pull request or merge request | The pull request |
| `build` | A build output | The build artifact or its record |
| `test_report` | A machine-readable test report | The report |
| `preview_url` | A deployed preview | The preview's URL |

Evidence kinds that evaluations of this profile use:

| `kind` | What it is | Rule |
| --- | --- | --- |
| `test_report` | Results of an automated test run | A Producer MUST give it `counts`, with `passed` and `failed` and optionally `skipped`. A missing `counts` is a `schema_violation`. |
| `review` | A person's review, such as a code review or a judgment on a preview | |
| `preview_check` | A check run against a deployed preview | |

The profile owns these kinds without a domain prefix. In an object that follows the profile, an artifact or evidence `kind` with no domain prefix is one of the kinds above; any other is a `schema_violation`. A kind from another vocabulary carries the domain of whoever defines it ([section 2.2](conventions.md)), such as `harborpine.example/activation_check` for a check of Harbor & Pine's own provisioning.

## 14.4 Rules

- **Commit digests.** A `commit` artifact's `digest` is its git object identifier, prefixed with the hash algorithm: `sha1:` followed by 40 lowercase hex digits, or `sha256:` followed by 64, for repositories that use git's SHA-256 object format. Any other digest on a commit artifact is a `schema_violation`. The digest names the commit as git does; it is not a hash computed by Enna.
- **Production deploys.** `production.deploy` needs a grant of its own: no other grant covers it, because grants match by exact action name ([section 5.2](authority-binding.md)). It also needs an approval whose `method` is not `standing`: an approval in scope given for this contract revision and these grants, interactively or through a PAP confirmation. When the contract, the binding or the receipt follows the profile, the record check ignores standing approvals for `production.deploy` ([section 9.7](invariants.md)), so a deploy that only a standing approval covers is `approval_missing`.
- A Producer SHOULD record the deployed build or commit as an artifact of a deploy receipt, and the deployed URL as a `preview_url` artifact for a preview.

## 14.5 Example

The first receipt of Harbor & Pine's experiment ([section 6.7](execution-receipt.md)) follows this profile: a `repo.branch.create` action on the `repository` resource `harborpine/collab-app`, with a `branch` artifact. A commit artifact, as a later receipt in the same record records it:

```jsonc
{
  "artifacts": [
    {
      "id": "art_commit",
      "kind": "commit",
      "ref": "https://git.harborpine.example/collab-app/commit/9b2d4e7c5a1f0e3d8c6b4a29f7e1d0c3b5a6f8e2",
      // The git object identifier, with its algorithm.
      "digest": "sha1:9b2d4e7c5a1f0e3d8c6b4a29f7e1d0c3b5a6f8e2",
      "media_type": "text/plain"
    }
  ]
}
```

An evaluation of this profile backs an automated criterion with a test report, which carries counts:

```jsonc
{
  "criterion_id": "AC-3",
  "result": "pass",
  "method": "automated_check",
  "evidence": [
    {
      "kind": "test_report",
      "ref": "https://ci.harborpine.example/collab-app/runs/812/tenant-isolation.json",
      "digest": "sha256:b154122e627f8df5df4ea965aba0ebff8ee3f3e09d906428950108300233b97a",
      "summary": "No cross-workspace reads.",
      "counts": { "passed": 24, "failed": 0, "skipped": 1 }
    }
  ]
}
```

## 14.6 Work in other domains

Work outside software delivery uses the core objects with actions named under the organization's domain, such as `harborpine.example/provisioning.retry` for a customer-operations fix ([section 15.1](relationships-and-mappings.md)). One contract can mix both: a contract that follows this profile can also use the organization's own actions, as a customer-operations fix that needs a code change would. A profile for customer operations is an open question, not part of 0.1 ([section 17.3](non-goals-and-open-questions.md)).

---
title: Evaluation result
description: The object that records criterion-by-criterion evidence, the derived technical result, the owner's acceptance and business impact, each kept apart.
order: 7
status: normative
---

# Evaluation result

An evaluation result (`evaluation_result`) records how a contract revision's acceptance criteria were judged, criterion by criterion, with evidence. It keeps three things apart that are easy to blur: whether execution finished, whether the technical criteria passed, and whether the accountable owner accepted the outcome. It records business impact separately again, because a passing test suite is not evidence of customer value. It is revisable, and acceptance is recorded as a new revision. Its schema is `evaluation-result.schema.json`.

## 7.1 Members

Besides the members every object carries ([section 2.16](conventions.md)):

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `contract` | contract reference | Yes | The contract revision evaluated. |
| `receipts` | array of receipt references | Yes | The receipts the evaluation relied on, each by identifier and digest. The array may be empty. An Evaluator SHOULD cite each action's final receipt: where a later receipt under the same idempotency key settled an `unknown` outcome, it cites the later one ([section 9.13](invariants.md)). |
| `evaluator` | object | Yes | `principal` (who evaluated), `name` and `version` (of the evaluating method or tool), all required. |
| `execution` | object | Yes | `status`, required: `complete`, `partial` or `incomplete` ([7.3](evaluation-result.md)). |
| `criteria` | array | Yes | One entry per criterion evaluated ([7.2](evaluation-result.md)). |
| `technical_result` | enum | Yes | `pass`, `fail` or `unknown`, derived from the criteria ([7.4](evaluation-result.md)). |
| `acceptance` | object | Yes | The owner's decision ([7.5](evaluation-result.md)). |
| `business_impact` | object | Yes | Whether the outcome made a difference, and how that is being measured ([7.6](evaluation-result.md)). |
| `limitations` | array of text | No | What the evaluation could not establish. |

## 7.2 Criteria and evidence

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `criterion_id` | `AC-` and a number | Yes | A criterion of the contract. Unique within the evaluation. |
| `result` | enum | Yes | `pass`, `fail` or `unknown`. |
| `method` | enum | Yes | How it was judged: `automated_check`, `human_judgment` or `measurement_window`. An Evaluator SHOULD use the method the contract states for the criterion. |
| `evidence` | array | Yes | What the result rests on, as described below. May be empty only when `result` is `unknown`. |
| `limitations` | array of text | No | What this result does not show. |

Each piece of evidence has:

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `kind` | token | Yes | What it is, such as `test_report` or `review`. Profiles define kinds ([section 14.3](software-delivery-profile.md)). |
| `ref` | text | Yes | Where it is. |
| `digest` | artifact digest | Yes | The digest of its content. |
| `summary` | text | Yes | What it shows, in plain language. |
| `counts` | object | No | For test reports: `passed` and `failed`, required, and `skipped`, optional, each a non-negative integer. |
| `measurements` | array, at least one | No | Quantities the evidence reports, such as a latency. Each has a `name` (a token), an integer `value` and its `unit` (a token such as `ms` or `s`), all required, and `illustrative`, which, when present, is `true`: the value was stated by a fixture or a simulation, not measured. A reader MUST NOT treat an illustrative value as a measurement. |

- A `pass` or a `fail` MUST carry at least one piece of evidence; a Consumer reports a missing one as `evidence_missing` at the criterion's `evidence`. An `unknown` result may carry none.
- In a record, every criterion the evaluation names must exist in the contract (`criterion_unknown`), and every required criterion must appear (`evidence_missing`) ([section 9.12](invariants.md)).
- Evidence shows what was checked, not that the check was the right one. An Evaluator SHOULD record in `limitations` what a criterion's evidence leaves open, such as cases a test suite skipped.

## 7.3 Execution status

`execution.status` describes whether the work was carried out, apart from whether it was any good:

| `status` | Meaning |
| --- | --- |
| `complete` | Every planned action has a known outcome, and the work was carried out. |
| `partial` | Every planned action has a known outcome, but some of the work was not done, failed or was cancelled. |
| `incomplete` | Execution has not finished, or some outcome is still unknown. |

An Evaluator MUST NOT report `complete` or `partial` while an outcome it relies on is `unknown`. Acceptance cannot follow an `incomplete` execution ([section 9.13](invariants.md)).

## 7.4 The technical result

`technical_result` is derived from the contract's required criteria, never asserted:

1. If any required criterion's result is `fail`, it is `fail`.
2. Otherwise, if the contract has at least one required criterion and every required criterion's result is `pass`, it is `pass`.
3. Otherwise it is `unknown`. A required criterion missing from the evaluation counts as `unknown`, and a contract with no required criterion derives `unknown`, never a vacuous `pass`.

Criteria that are not required are evaluated and reported, but do not change the technical result. A record whose `technical_result` differs from the derived value shows `technical_result_mismatch` ([section 9.12](invariants.md)).

## 7.5 Acceptance

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `decision` | enum | Yes | `pending`, `accepted` or `rejected`. |
| `by` | principal reference | Exactly when `decision` is not `pending` | Who decided. |
| `at` | timestamp | Exactly when `decision` is not `pending` | When they decided. |
| `scope` | token | No | What was accepted, such as `technical_preview`. |
| `notes` | text | No | Why, and on what conditions. |

- Acceptance is recorded as a new revision of the evaluation. A decided acceptance in revision 1 is `acceptance_premature` at `/acceptance/decision`. The evaluation is first issued with `pending`; the owner's decision is a later revision whose `revision_author` is normally the person who decided.
- Acceptance comes last. It needs an execution status other than `incomplete`, no action in the record, cited or not, whose final outcome is `unknown`, a time no earlier than the end of every effect in the record and than the evaluation revisions it follows, and, to accept rather than reject, at least one effect; otherwise it is `acceptance_premature` ([section 9.13](invariants.md)). Refusals and replays change nothing, so they never make acceptance premature.
- Acceptance comes from the owner, from a person the contract names as a stakeholder whose role is `acceptor`, or from an agent the owner delegated it to; otherwise it is `acceptance_not_owner`. The owner names a person by listing them as an acceptor in the contract, so naming one, or ceasing to, is a new contract revision ([section 3.6](work-contract.md)). The owner delegates acceptance to an agent with an authority binding whose grantor's `person` is the owner, whose grantee is the agent, and which has a grant of the action `acceptance.decide` on a resource of type `work_contract`, either naming the contract's `id` or naming none. A delegation is a decision like any other: at the time of acceptance the binding must hold the owner's own approval of it, current, decided, unexpired and unrevoked ([section 5.4](authority-binding.md)), and no such denial by an approver in scope. It must also be inside its window, with neither it nor any binding it was delegated from revoked ([section 5.6](authority-binding.md)). Only the owner delegates acceptance; an acceptor cannot pass it on.
- Accepting a technical preview is not accepting business success. `scope` and `notes` say what was accepted.

## 7.6 Business impact

| Member | Type | Required | Meaning |
| --- | --- | --- | --- |
| `status` | enum | Yes | `unknown`, `measuring` or `measured`. |
| `window` | object, at least one member | No | `starts_at` (timestamp) and `window_seconds` (an integer of at least 1): the measurement window in progress or done. |
| `notes` | text | No | What is being measured, what was found, and what is still open. |

Business impact stays `unknown` until something measures it. A technical `pass`, an accepted preview or a passing test suite does not move it. A criterion measured over a window stays `unknown` in the evaluation until the window has run, and a later revision records the result.

## 7.7 Evaluators and executors

At the Controlled level and above, the Evaluator of a required `automated_check` criterion SHOULD be a principal other than the one that performed the actions under evaluation. When it is the same, the evaluation MUST say so in `limitations` ([section 13.2](conformance.md)). A check that an agent runs on its own work is still evidence, but a reader needs to know whose.

## 7.8 Example

The evaluator agent judged the experiment: the three automated criteria passed on test reports, Dana judged the preview usable, and the adoption criterion stays `unknown` because its 30-day window starts only when a pilot client approves. Revision 1 of the evaluation was `pending`. In revision 2, shown here, Dana accepted the work as a technical preview. Business impact is still `unknown`.

```json
{
  "protocol_version": "0.1",
  "type": "evaluation_result",
  "id": "eval_brand_approvals",
  "revision": 2,
  "created_at": "2026-10-16T15:05:00Z",
  "scope": {
    "organization": "harborpine.example",
    "workspace": "internal",
    "project": "brand-approvals"
  },
  "profile": "software-delivery/0.1",
  "contract": {
    "id": "wc_brand_approvals",
    "revision": 1
  },
  "receipts": [
    {
      "id": "rcpt_branch",
      "digest": "sha256:06d91777872676cbd1519356e82a020889dbcc32b68af0823a4572234e8b7d20"
    },
    {
      "id": "rcpt_lumen_read_denied",
      "digest": "sha256:3e7f314d0e085ac0001978a47bd84d5c050c11e30bc9b7f2e4f88998215e0ffc"
    },
    {
      "id": "rcpt_commit",
      "digest": "sha256:dfd385f14c7096a3d4315c1d766f8d3d5be0004c8e6cc6abbbed206b62bdab56"
    },
    {
      "id": "rcpt_ci",
      "digest": "sha256:c068b494f1782da96751c9f4974d08a3d9a198d658a7ac767f4b7556b7225b39"
    },
    {
      "id": "rcpt_pull_request",
      "digest": "sha256:8ad44f7f8155d67ebf6c65a3569b6e151a5bef4c1b2a23452a090c9c79724367"
    },
    {
      "id": "rcpt_preview",
      "digest": "sha256:652127f2acc181b8739046ff9a5432071cdc215ae685eec61530a41a4fa19c4a"
    }
  ],
  "evaluator": {
    "principal": {
      "kind": "agent",
      "id": "https://agents.harborpine.example/evaluator",
      "display": "Evaluator agent"
    },
    "name": "harborpine-evaluator",
    "version": "0.1.0"
  },
  "execution": {
    "status": "complete"
  },
  "criteria": [
    {
      "criterion_id": "AC-1",
      "result": "pass",
      "method": "automated_check",
      "evidence": [
        {
          "kind": "test_report",
          "ref": "https://ci.harborpine.example/collab-app/runs/812/approval-gate.json",
          "digest": "sha256:97ab49ed582226109b4f03bf24a7a457ef7afc80e169700fe96fcd6205125965",
          "summary": "Unapproved publishing is blocked.",
          "counts": {
            "passed": 18,
            "failed": 0
          }
        }
      ]
    },
    {
      "criterion_id": "AC-2",
      "result": "pass",
      "method": "automated_check",
      "evidence": [
        {
          "kind": "test_report",
          "ref": "https://ci.harborpine.example/collab-app/runs/812/approval-records.json",
          "digest": "sha256:0d85ea6c0293a60fa7a8dd6bdad6ac999f883285179a4b1900e892872dbfc916",
          "summary": "Each approval is recorded.",
          "counts": {
            "passed": 9,
            "failed": 0
          }
        }
      ]
    },
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
          "counts": {
            "passed": 24,
            "failed": 0,
            "skipped": 1
          }
        }
      ],
      "limitations": [
        "One cross-region case is skipped in the preview environment."
      ]
    },
    {
      "criterion_id": "AC-4",
      "result": "pass",
      "method": "human_judgment",
      "evidence": [
        {
          "kind": "review",
          "ref": "https://git.harborpine.example/collab-app/pull/318#review-1",
          "digest": "sha256:71c43f40b31947dbf52e012b1a31e991368a27bc880aafcd43dce397901b278a",
          "summary": "Dana tried the preview and found the flow usable."
        }
      ]
    },
    {
      "criterion_id": "AC-5",
      "result": "unknown",
      "method": "measurement_window",
      "evidence": [],
      "limitations": [
        "The 30-day window starts only when a pilot client approves."
      ]
    }
  ],
  "technical_result": "pass",
  "acceptance": {
    "decision": "accepted",
    "by": {
      "kind": "person",
      "id": "dana.whitfield",
      "issuer": "https://id.harborpine.example",
      "display": "Dana Whitfield"
    },
    "at": "2026-10-16T15:00:00Z",
    "scope": "technical_preview",
    "notes": "Accepted as a technical preview. Nothing goes to production."
  },
  "business_impact": {
    "status": "unknown",
    "notes": "Pilot adoption has not been measured; the window starts on pilot approval."
  },
  "limitations": [
    "A passing test suite is not evidence of customer value."
  ],
  "revision_author": {
    "kind": "person",
    "id": "dana.whitfield",
    "issuer": "https://id.harborpine.example",
    "display": "Dana Whitfield"
  },
  "change_summary": "Recorded the owner's acceptance of the technical preview."
}
```

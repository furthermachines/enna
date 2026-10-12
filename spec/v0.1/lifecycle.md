---
title: Business lifecycle
description: The states of a work contract, the events that move it, the normative transition table, business blocks, and the mappings to A2A 1.0 task states and PAP 0.1 operation states.
order: 10
status: normative
---

# Business lifecycle

A work contract moves through a business lifecycle that is independent of any transport. A transport task can fail and be retried while the business work is unaffected; business work can be blocked while every transport is healthy. This section defines the lifecycle's states, its events and its transition table, and maps it to A2A 1.0 and PAP 0.1.

The table is normative and is implemented as data in `@enna/protocol` (`STATES`, `EVENTS`, `TRANSITIONS`, `BLOCK_REASONS` and `transition`), with tests for every legal and illegal transition.

## 10.1 States

| State | Meaning | Terminal |
| --- | --- | --- |
| `proposed` | The contract revision exists, with its terms on offer, and has not been sent for approval. | No |
| `awaiting_approval` | The current revision waits for an approver in scope to approve or deny its authority. | No |
| `authorized` | The current revision is approved, and no effect is in progress. | No |
| `executing` | Effects are being performed. The state holds until the outcome of every effect that has started is known. | No |
| `execution_complete` | Every effect's outcome is known. | No |
| `evaluating` | An Evaluator is judging the criteria. | No |
| `awaiting_acceptance` | The evaluation is issued, and the owner's acceptance decision is awaited. | No |
| `accepted` | The owner, an acceptor or the owner's delegate accepted the outcome. | Yes |
| `rejected` | An approver in scope rejected the contract at approval, or the owner rejected the outcome at acceptance. | Yes |
| `cancelled` | The work was withdrawn. The spelling follows PAP; A2A spells it `canceled`. | Yes |
| `blocked` | A business reason stops the work, and the block records which one ([10.4](lifecycle.md)). | No |

There is no failed state. A failed action is a receipt outcome, and a failed criterion is an evaluation result; the contract still moves on to evaluation and acceptance, where people decide what the failure means.

## 10.2 Events

| Event | What happens | Evidence on record |
| --- | --- | --- |
| `request_approval` | The revision is sent to its approvers, or approval is asked for again after a block. | |
| `approve` | An approver in scope approves. | A current approval ([section 9.5](invariants.md)) |
| `reject` | At approval, an approver in scope denies; at acceptance, the owner, an acceptor or the owner's delegate rejects. | A denial, or an evaluation revision whose acceptance is `rejected` |
| `start` | The Enforcement point starts performing effects. | A receipt whose decision is `permit` |
| `complete_execution` | Every effect's outcome is known. | Receipts with known outcomes |
| `start_evaluation` | An Evaluator begins. | |
| `complete_evaluation` | The Evaluator issues its evaluation, with acceptance `pending`. | An evaluation result |
| `accept` | The owner, an acceptor or the owner's delegate accepts. | An evaluation revision whose acceptance is `accepted` |
| `revise` | A new revision of the contract is issued. | The new contract revision |
| `cancel` | The work is withdrawn. Who may withdraw it is the organization's decision; this draft does not define it. | |
| `block` | The work stops for a business reason, which must be given. | Usually a receipt denied for that reason |
| `resume` | The cause of a block was removed without a new approval, for example by refreshing stale context. | For example, a new manifest revision |

## 10.3 Transitions

| From | Event | To |
| --- | --- | --- |
| `proposed` | `request_approval` | `awaiting_approval` |
| `proposed` | `revise` | `awaiting_approval` |
| `proposed` | `cancel` | `cancelled` |
| `awaiting_approval` | `approve` | `authorized` |
| `awaiting_approval` | `reject` | `rejected` |
| `awaiting_approval` | `revise` | `awaiting_approval` |
| `awaiting_approval` | `cancel` | `cancelled` |
| `authorized` | `start` | `executing` |
| `authorized` | `block` | `blocked` |
| `authorized` | `revise` | `awaiting_approval` |
| `authorized` | `cancel` | `cancelled` |
| `executing` | `complete_execution` | `execution_complete` |
| `executing` | `block` | `blocked` |
| `execution_complete` | `start_evaluation` | `evaluating` |
| `execution_complete` | `revise` | `awaiting_approval` |
| `execution_complete` | `cancel` | `cancelled` |
| `evaluating` | `complete_evaluation` | `awaiting_acceptance` |
| `evaluating` | `revise` | `awaiting_approval` |
| `evaluating` | `cancel` | `cancelled` |
| `awaiting_acceptance` | `accept` | `accepted` |
| `awaiting_acceptance` | `reject` | `rejected` |
| `awaiting_acceptance` | `revise` | `awaiting_approval` |
| `awaiting_acceptance` | `cancel` | `cancelled` |
| `blocked` | `request_approval` | `awaiting_approval` |
| `blocked` | `resume` | `authorized` |
| `blocked` | `complete_execution` | `execution_complete` |
| `blocked` | `revise` | `awaiting_approval` |
| `blocked` | `cancel` | `cancelled` |

Rules:

- Any state and event pair not in the table is illegal, and so is any unknown state or event: the result is `illegal_transition`, and the state does not change. A terminal state accepts no event.
- `block` needs a reason from the business codes of [10.4](lifecycle.md); without one, it is `illegal_transition`.
- `revise` is allowed from every non-terminal state except `executing`, and always leads to `awaiting_approval`. An approval bound to the earlier revision is stale from then on ([section 9.5](invariants.md)).
- `executing` cannot be revised or cancelled. While any effect may still be in flight, the work can neither change its terms nor be reported as stopped. To stop work in progress, stop starting new effects (by revoking the authority or blocking the contract), wait until every outcome is known, then apply `complete_execution` and `cancel`. A contract blocked while it was executing reaches `execution_complete` directly once every outcome is known: it need not `resume` and `start` again, which would claim that new work begins.
- While any effect of the current revision has an `unknown` outcome, an Enforcement point, or any system that manages the lifecycle, MUST NOT apply `cancel` or `revise`, from `blocked` either, and MUST NOT apply `complete_execution`.
- `request_approval` from `blocked` asks for approval of the same revision again, for example after an approval expired or was revoked. An Enforcement point MUST NOT apply `resume` while the cause of a block still holds. When the reason is one of the approval codes (`approval_missing`, `approval_expired`, `approval_revoked`, `approval_stale`, `approval_denied`, `approver_out_of_scope`, `separation_of_duties` or `quorum_not_met`), the contract leaves `blocked` through `request_approval` and a new approval, or through `revise`; never through `resume`. This is recovery after explicit reapproval.

## 10.4 Business blocks and transport failures

A contract is blocked only for a business reason, one of these error codes:

`approval_missing`, `approval_expired`, `approval_revoked`, `approval_stale`, `approval_denied`, `approver_out_of_scope`, `separation_of_duties`, `quorum_not_met`, `authority_expired`, `authority_revoked`, `authority_insufficient`, `subdelegation_forbidden`, `attenuation_violated`, `context_stale`, `context_out_of_scope`, `tenant_mismatch`, `limit_exceeded`, `currency_mismatch`

A transport failure is never a block. A timeout, a lost connection, an HTTP 429 or a server error is retried at the transport, with the same idempotency key, and does not change the business state. An action whose outcome a failure has made unknown is not a block either: its receipt records `unknown` and the contract stays `executing` until the outcome is known ([section 11.3](execution-semantics.md)).

## 10.5 Where the state lives

No object in 0.1 has a member that holds the lifecycle state. The state is kept by whichever system manages the contract, and is reported to other parties through their transports, as an A2A task state or a PAP operation state. The objects are the evidence for each transition, as the table in [10.2](lifecycle.md) shows. Whether a later draft should add a state member or a record of transitions is an open question ([section 17.3](non-goals-and-open-questions.md)).

## 10.6 Mapping to A2A 1.0 task states

Status: **implemented and tested**, as data mappings in `@enna/protocol` (`A2A_STATE_TABLE`, `toA2aTaskState`, `fromA2aTaskState`, `A2A_UNMAPPED`). The task state names are A2A 1.0's, as ProtoJSON writes them (protocol version 1.0; specification release v1.0.0 of 2026-03-12 and v1.0.1 of 2026-05-28, whose task state enumeration is unchanged; checked 2026-10-10).

The mapping is not one to one. Each row says whether the match is exact or approximate, and why.

| Business state | A2A 1.0 task state | Match | Why |
| --- | --- | --- | --- |
| `proposed` | `TASK_STATE_SUBMITTED` | approximate | A2A's submitted means acknowledged; an Enna proposal has its terms on offer but is not yet sent for approval. |
| `awaiting_approval` | `TASK_STATE_INPUT_REQUIRED` | approximate | Approval is a decision by an accountable person. TASK_STATE_AUTH_REQUIRED is about credentials and never stands for approval. |
| `authorized` | `TASK_STATE_WORKING` | approximate | A2A has no approved-but-not-started state. |
| `executing` | `TASK_STATE_WORKING` | exact | Same meaning. |
| `execution_complete` | `TASK_STATE_WORKING` | approximate | Execution has finished but evaluation and acceptance remain; TASK_STATE_COMPLETED would claim the business work is done. |
| `evaluating` | `TASK_STATE_WORKING` | approximate | Evaluation is still work on the contract. |
| `awaiting_acceptance` | `TASK_STATE_INPUT_REQUIRED` | approximate | Waits for the owner's acceptance decision. |
| `accepted` | `TASK_STATE_COMPLETED` | exact | Same meaning. |
| `rejected` | `TASK_STATE_REJECTED` | approximate | A2A's rejected means the agent declined the task; Enna's means an accountable person declined, at approval or at acceptance. |
| `cancelled` | `TASK_STATE_CANCELED` | exact | Same meaning; A2A spells it canceled, Enna follows PAP's cancelled. |
| `blocked` | `TASK_STATE_INPUT_REQUIRED` | approximate | A business block waits for a person to fix authority, context or limits. It is neither a transport failure nor TASK_STATE_AUTH_REQUIRED. |

Three A2A states stand for no business state:

| A2A 1.0 task state | Business state | Why |
| --- | --- | --- |
| `TASK_STATE_FAILED` | None | Enna has no failed business state: a failed action is a receipt outcome and a failed criterion is an evaluation result, and the contract still moves to evaluation and acceptance. |
| `TASK_STATE_AUTH_REQUIRED` | None | Transport authentication stays at the transport. A missing approval is awaiting_approval or blocked, never this state. |
| `TASK_STATE_UNSPECIFIED` | None | Not a state Enna reports. |

In the other direction, one A2A state can stand for several business states. A party that receives an A2A task state needs the Enna objects to know which one applies:

| A2A 1.0 task state | Business states it can stand for |
| --- | --- |
| `TASK_STATE_UNSPECIFIED` | None |
| `TASK_STATE_SUBMITTED` | `proposed` |
| `TASK_STATE_WORKING` | `authorized`, `executing`, `execution_complete`, `evaluating` |
| `TASK_STATE_COMPLETED` | `accepted` |
| `TASK_STATE_FAILED` | None |
| `TASK_STATE_CANCELED` | `cancelled` |
| `TASK_STATE_INPUT_REQUIRED` | `awaiting_approval`, `awaiting_acceptance`, `blocked` |
| `TASK_STATE_REJECTED` | `rejected` |
| `TASK_STATE_AUTH_REQUIRED` | None |

`TASK_STATE_AUTH_REQUIRED` concerns credentials at the transport. An Enforcement point or Producer that reports business state over A2A MUST NOT report a missing approval as `TASK_STATE_AUTH_REQUIRED`, and MUST NOT treat a transition out of it as an approval.

## 10.7 Mapping to PAP 0.1 operation states

Status: **implemented and tested**, as data mappings in `@enna/protocol` (`PAP_STATE_TABLE`, `toPapOperationState`, `PAP_UNMAPPED`), against the state names of the PAP 0.1 operations extension, version 1 (checked 2026-10-10).

| Business state | PAP 0.1 operation state | Match | Why |
| --- | --- | --- | --- |
| `proposed` | `proposed` | approximate | PAP has no draft state; an operation exists only once it is proposed for confirmation. |
| `awaiting_approval` | `proposed` | exact | Same meaning. |
| `authorized` | `in_progress` | approximate | PAP starts performing at confirmation and has no confirmed-but-waiting state. |
| `executing` | `in_progress` | exact | Same meaning. |
| `execution_complete` | From the receipts | derived | Execution has finished, so PAP's final state comes from the receipts: succeeded, failed or cancelled. |
| `evaluating` | From the receipts | derived | Execution has finished, so PAP's final state comes from the receipts: succeeded, failed or cancelled. |
| `awaiting_acceptance` | From the receipts | derived | Execution has finished, so PAP's final state comes from the receipts: succeeded, failed or cancelled. |
| `accepted` | From the receipts | derived | Execution has finished, so PAP's final state comes from the receipts: succeeded, failed or cancelled. |
| `rejected` | From the receipts | derived | From the receipts; with none, nothing ran and the operation was declined, which PAP calls cancelled. |
| `cancelled` | `cancelled` | exact | Same meaning. |
| `blocked` | `in_progress` | approximate | A block comes after confirmation, and PAP keeps the operation in progress until its outcome is known. |

After execution, PAP's state comes from each action's final receipt, by the rollup of [section 15.1](relationships-and-mappings.md): `in_progress` while any final outcome is `unknown` or there is no receipt; otherwise `cancelled` if any final receipt was cancelled; otherwise `failed` if any final receipt failed or was denied, so a denial that a later outcome under the same idempotency key superseded does not count; otherwise `succeeded`. A contract rejected before anything ran maps to `cancelled`, since PAP calls a declined operation cancelled. A finished state whose operation has no receipts, other than `rejected`, cannot be mapped, and the mapping refuses it.

| PAP 0.1 operation state | Business state | Why |
| --- | --- | --- |
| `expired` | None | A contract does not expire on its own. An expired approval or authority blocks the work instead (approval_expired, authority_expired). |

PAP reports an operation as `in_progress` until its outcome is known, and never as `failed` while it is not. The business lifecycle keeps the same rule: `executing` holds until every outcome is known.

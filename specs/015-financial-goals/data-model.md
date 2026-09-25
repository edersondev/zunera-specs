# Data Model: Financial Goals

**Date**: 2026-09-25 | **Spec**: [spec.md](spec.md) | **Research**: [research.md](research.md)

## Financial goal

| Field | Meaning and validation |
|---|---|
| `id`, `user_id` | Stable identity and required owner. Every read/mutation is owner scoped. Duplicate names are allowed. |
| `name` | Input at most 200 characters; trim before storage and reject blank-after-trim input. Render as escaped text. |
| `target_centavos` | Positive integer BRL centavos, R$ 0,01–R$ 9.999.999.999,99. |
| `target_date` | Nullable calendar date, today or future when set; an existing date may later be overdue. |
| `financial_account_id` | Nullable reference to the declared holding account. New/changed value must be owned and active; credit cards are not eligible. Preserve null for account-free goals. |
| `account_name_snapshot` | Nullable last known account label for degraded display if the current association is unavailable. Updated when an account is linked, not when historical activity is read. |
| `description` | Nullable user text; escaped on output. |
| `status` | `active`, `completed`, or `archived`. No extra status for overfunding or shortfall. |
| `completed_at`, `archived_at` | Nullable lifecycle timestamps. Reopen/restore clears the current state timestamp while history retains every prior transition. |
| `created_at`, `updated_at` | Audit timestamps. |

No independent goal balance field is required. A goal is not a financial account. Do not add goal amounts to assets or account balance. Use owner/status and account/status indexes for the 100-goal overview and account-capacity reads. Financial-account deletion is not an application action; protect the relationship from ordinary cascade deletion and retain snapshots for exceptional unavailable-reference displays.

## Goal activity

| Field | Meaning and validation |
|---|---|
| `id`, `financial_goal_id`, `user_id` | Stable event identity and owner, consistent with the goal. |
| `type` | `created`, `initial_allocation`, `allocated`, `withdrawn`, `goal_updated`, `account_changed`, `completed`, `reopened`, `archived`, or `restored`. |
| `amount_centavos` | Positive only for initial allocation, allocation, and withdrawal; null for all other event types. No positive initial event when initial amount is zero. |
| `financial_account_id_at_time`, `account_name_at_time` | Nullable account identity/name snapshot for monetary events, preserved after account rename/archive/reassociation. Null means user-declared unlinked allocation. |
| `details` | Small structured before/after context for account change, target/date changes, or lifecycle event as needed; no financial transaction reference or secret. |
| `occurred_at`, `business_date` | Event timestamp and the corresponding America/Sao_Paulo business date. Stable descending list order uses `occurred_at`, then `id`. |

Activity is append-only. A status or metadata edit never rewrites past events. Initial allocation is one monetary event distinct from `created`. Current allocation is the sum of `initial_allocation` and `allocated` amounts minus `withdrawn` amounts for that goal. Lifecycle/metadata events have no money effect. Index `(financial_goal_id, occurred_at, id)` and `(user_id, financial_goal_id)`; grouped amount queries must be owner scoped.

## Goal mutation request

| Field | Meaning and validation |
|---|---|
| `user_id`, `idempotency_key` | Unique pair, matching existing financial mutation conventions. |
| `operation`, `request_fingerprint` | Bind the key to a specific action, goal identity, and normalized payload. |
| `financial_goal_id`, `response_status`, `response_body`, `completed_at` | Result needed for exact replay, including created goal identity. |
| `created_at`, `updated_at` | Audit timestamps. |

The request claim, goal/activity write, and final response snapshot commit atomically. Same owner/key/fingerprint replays the original result; same key with different fingerprint returns 409. A failed transaction leaves no accepted activity or completed claim. Existing transfer/card mutation tables remain separate.

## Existing financial account relationship

- One active or archived financial account may be linked to many active or completed goals. An archived goal has zero allocation before archival and retains its association for historical context until edited after restore.
- An account's actual `current_balance_centavos` remains authoritative. For an active account, `designated_centavos` is the sum of allocations of active and completed linked goals. `unallocated_centavos = current_balance_centavos - designated_centavos`; negative means a visible shortfall, not a new account balance.
- Goal associations do not point to Credit Cards. No goal row belongs in Transactions, Transfers, financial history, Budget consumption, or card statement calculations.
- Account archive leaves linkage/history intact. The goal projection labels the account inactive and disallows new linked allocations; a user may withdraw, unlink, or change association while the goal is active.

## Derived goal projections

| Projection | Rule |
|---|---|
| `allocated_centavos` | Accepted monetary event sum, never below zero. |
| `remaining_centavos` | `max(target_centavos - allocated_centavos, 0)`. |
| `excess_centavos` | `max(allocated_centavos - target_centavos, 0)`. |
| `progress_percentage` | `allocated_centavos / target_centavos × 100`, not capped; present precise enough to avoid misleading whole-number rounding. |
| `visual_progress_percentage` | `min(progress_percentage, 100)` for a visual track only. |
| `suggested_monthly_centavos` | For active underfunded future-dated goals only: ceil of remaining centavos divided by inclusive calendar-month opportunities from current through target month. Otherwise null. |
| `account_backing` | `unverified` when unlinked; `available`, `shortfall`, or `inactive_or_unavailable` when linked. This is presentation context, not lifecycle status. |
| Overview totals | Sum active goals' target, allocated, and each goal's nonnegative remaining; report active unlinked allocation separately as unverified. Completed/archived goals are not in those totals. |
| Dashboard selection | At most three active goals, overdue dates first, then nearest future dates, then undated; stable name ordering for ties. |

Backend computes these fields from owned data. Frontend formats and renders them but does not recalculate financial values. For a negative account balance, unallocated may be negative even without a positive goal allocation; the account balance must not be normalized to zero.

## Lifecycle transitions

| From | Action and condition | To | Money/activity effect |
|---|---|---|---|
| New | Create valid goal | Active | `created` event; optional positive `initial_allocation` event after capacity check. |
| Active | Allocate positive amount with eligible account/capacity, or unlinked | Active | One `allocated` event. |
| Active | Withdraw positive amount up to current allocation | Active | One `withdrawn` event. |
| Active | Complete with allocation ≥ target and, if linked, an active available account without current shortfall | Completed | `completed` event; allocation retained. Unlinked completion keeps unverified-backing label. |
| Completed | Explicitly reopen | Active | `reopened` event; allocation retained. |
| Active | Archive only at zero allocation | Archived | `archived` event; no implicit withdrawal. |
| Archived | Restore | Active | `restored` event; zero allocation, complete history retained. |

Completed goals reject allocation, withdrawal, metadata edits, and archival until reopened. Archived goals reject all edits and money actions except restoration. A linked shortfall or inactive/unavailable association blocks a new completion. A later shortfall or account archival does not undo an existing completion; the completed goal retains its status and displays attention. Reaching target, target reduction, overdue date, account shortfall, and account archive do not silently change goal status.

## Concurrency and integrity invariants

1. Owner/status and account eligibility are rechecked inside the mutation transaction, after the appropriate rows are locked.
2. For an existing goal, lock its row, then old/new financial-account rows in ascending ID order. For creation, lock candidate account before insertion. Any other goal mutation requiring an account lock follows that order; account row locking serializes claims across goals sharing it.
3. Capacity uses the current account balance and the latest grouped goal allocations. The operation is accepted wholly or rejected wholly. A concurrent real financial movement may subsequently create a shortfall; no goal event or account movement is automatically reversed.
4. Activity creation, status/metadata mutation, and mutation-key completion are one atomic unit. No duplicated event can result from a replay.
5. A funded association change checks capacity on the destination account and records the old/new account context. It changes no account balance and creates no transfer.

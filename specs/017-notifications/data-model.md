# Data Model: Notifications

**Date**: 2026-09-29 | **Spec**: [spec.md](spec.md) | **Research**: [research.md](research.md)

Notifications store awareness, preference, and delivery identity only. Source domains retain every financial value and lifecycle decision.

## Notification event

One durable row per user and meaningful business key. An event can be visible, preference-suppressed, or expired. Suppressed/expired rows retain only minimal identity and timing needed for deduplication; they never enter center or counts.

| Field | Meaning / invariant |
| --- | --- |
| `id`, `user_id` | Notification identity and personal owner; user scope on every read/write. |
| `event_key` | Stable type/source/stage/period-or-target identity; unique with `user_id`. Never based on title, mutable amount, or translated text. |
| `type`, `category`, `severity` | One matrix type, four preference categories, Information/Attention/Success/Critical. |
| `source_kind`, `source_id` | Typed source reference; source may later disappear. Use current source ownership check when present. |
| `business_context` | Statement stage, recurrence occurrence kind, budget year/month and category-plan identity, or goal target centavos as needed for uniqueness. No independent financial subtotal. |
| `event_at`, `created_at` | When source meaning first qualified and when item was recorded. Use the former to evaluate preference then and explain timing. |
| `read_at`, `resolved_at` | Independent timestamps. Null `read_at` means unread. Source resolution sets `resolved_at` and `read_at`; recross may clear resolution while keeping read time. |
| `visibility`, `expired_at` | `visible`, `suppressed`, or `expired`. Visible content ages out after 90 calendar days following later of creation/resolution; unresolved actionable content remains while condition persists. |
| `event_snapshot` | Minimal sanitized event-time labels and amount/date context for historical explanation. Clear sensitive snapshot on expiry or suppression; never store full card number. |
| `current_context` | Optional current source-derived value/status for active alert. Refresh from source; label event-time values separately. |

Indexes: unique `(user_id, event_key)`; center and count paths by `(user_id, visibility, created_at, id)`, `(user_id, visibility, read_at)`, and `(user_id, visibility, resolved_at)`. Avoid foreign-key cascade from source entity to event identity because deletion must not reissue the same event.

### Business keys

| Type | Key parts after user/type |
| --- | --- |
| Statement approaching, due today, overdue | Statement ID + stage |
| Ordinary recurrence review | Generated transaction ID + review |
| Card recurrence review | Card occurrence ID + review; expected/over-limit change retains same key |
| Budget approaching/reached/exceeded | Category plan ID + current business year/month + stage; no new event for ended months |
| Goal reached | Goal ID + exact accepted target centavos; returning to same target reuses key |

One source update that jumps budget stages writes only highest current stage. Skipped lower stages are recorded as non-visible consumed identities for that plan/period, preventing later backfill after a fall. When an emitted stage recurs, retained item may become actionable again with its original read state. An expired identity remains consumed.
At the next business-month boundary, unresolved budget items for the prior month resolve and become read; 90-day history then applies. Later corrections to the ended month do not create fresh notification identities.

### Lifecycle

```text
first qualified + preference enabled -> visible/unread/(actionable or informational)
first qualified + preference disabled -> suppressed identity
visible unread -> visible read                  (open or explicit mark read)
visible actionable -> visible resolved/read    (source condition ends)
visible resolved/read -> visible actionable/read (same stage returns while retained)
visible nonactionable after 90 days -> expired identity, sensitive snapshot cleared
suppressed/expired -> never newly visible for same event key
```

Notification state never changes source money or status. When authorization is lost, the event is excluded from list/count/open; deletion or source unavailability removes destination/action but may retain a safe historical snapshot for the original owner. Source identity, auth, and destination are checked at serialization/open, not trusted from a persisted URL.

## Notification preference and preference change

| Entity | Fields | Rules |
| --- | --- | --- |
| Current preference | `user_id`, `category`, `enabled`, `updated_at` | Unique owner/category. Absent record means enabled. Four categories only; Critical follows category setting. |
| Preference change | `user_id`, `category`, `enabled`, `effective_at`, sequence | Append-only compact history for deciding whether first qualification during a disabled interval was suppressed, even if processing is delayed. New user starts with all enabled. |

Preference writes and event first-qualification decisions serialize for the same user/category. Disabling preserves visible history. Re-enabling affects only first-qualified events after effective time. Retain change history at least as long as unresolved projection requests could exist; a durable event identity should be created promptly, and old preference history may be compacted only after no pending event can depend on it.

## Projection request

Durable fact generated inside an accepted source mutation or by a time-boundary evaluation. It contains user, source kind/identity or affected month, source change time, any source-verified newly qualifying type/stage and exact first-qualification time, and processing state. It stores no independent financial sum. A source change can also request reconciliation of old actionable items. Coalescing repeated reconciliation requests is allowed, but distinct first-qualification facts must not be lost. On failure, retain fact for bounded retry and record lag/failure signal. Consuming one fact may create no visible alert when source already resolved, resolve a prior item, or create a visible/suppressed identity using preference effective at first qualification. This preserves transient goal milestones and disabled-interval suppression despite delayed processing.

## Existing source relationships

- **Credit Cards**: `CreditCardStatement` plus its domain reconciler; outstanding amount/status/due date and masked card identity.
- **Ordinary recurrence**: generated `Transaction` with recurrence association and pending/effective/removed state.
- **Card recurrence**: `RecurringCardOccurrence` with expected/awaiting-over-limit/failed/recorded/dismissed state and schedule snapshots. Expected, awaiting-over-limit, and reviewable failed states share one review event identity; source reason changes refresh safe context. Recorded or dismissed state resolves it.
- **Budgets**: `MonthlyBudget` and `BudgetCategoryPlan`; `BudgetCalculationService::forMonth` supplies exact status, realized and excess values.
- **Goals**: `FinancialGoal` and goal projection/capacity service; accepted allocation/target changes cause target-reached evaluation. Goal stays active until source explicitly completes it.

No notification foreign key or status is added to these source records. Current Zunera has personal ownership via `user_id`; a future workspace model must apply its authorization before exposing events.

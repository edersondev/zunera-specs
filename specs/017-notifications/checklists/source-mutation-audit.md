# Accepted Source Mutation Audit: Notifications

Checked against `../zunera-backend/app/Services/` on 2026-09-29. Capture a projection fact only inside the accepted source transaction, after authoritative state is settled and only on a non-replayed mutation. The fact must identify owner, source, affected business date/month, and any first-qualified stage/time. Do not copy financial totals into notification calculations.

| Source service and accepted entry points | Fact or reconciliation need |
| --- | --- |
| `CreditCards/CreditCardStatementPaymentService::create`, `update`, `remove`, `restore` | Reconcile the touched statement after `CreditCardObligationReconciler::syncStatement`; payment reversal can reactivate a retained stage. Capture the effective transition before commit. |
| `CreditCards/CreditCardCreditEventService::record`, `cancel` | Reconcile every `affected_statements` ID after credit application and refreshed card statements; capture changed installment recognition months for Budgets. `cancel` delegates to `record`. |
| `CreditCards/CreditCardPurchaseService::create`, `recordOccurrencePurchase` | Reconcile created/touched statement IDs and recognized installment months after allocation. Automatic card occurrence recording can also resolve its review item. |
| `CreditCards/CreditCardPurchaseCorrectionService::update` | Reconcile old and new statement IDs and old/new recognition months. Pruned old statements lose actionable destinations safely. |
| `CreditCards/CreditCardService::create`, `update`, `archive`, `restore` | Card update can change masked display or billing dates; archive/restore can alter source availability. Reconcile affected statements where present. |
| `CreditCards/CreditCardStatementService::list`, `findOwned`, `refresh`, `syncStatement` | These refresh authoritative status on reads. Date-stage evaluator must use the same reconciler and business date; avoid emitting facts merely from presentation reads. |
| `Transactions/TransactionService::create`, `update`, `remove`, `restore` | Capture old/new effective expense category and transaction month for budget projection. For generated ordinary recurrence rows, capture pending review qualification and its resolution on effective/removed/restore transitions. Routine transactions alone create no alert. |
| `RecurringTransactions/RecurringOccurrenceService::processDueRules`, `processRule`, account/card occurrence creation, automatic recording, failure-state helpers | Capture generated ordinary pending transaction or card occurrence review qualification in the committing creation/state transaction. Automatic successful recording is not a review alert. Paused/ended rules do not resolve earlier reviewable occurrences. |
| `RecurringTransactions/RecurringCardOccurrenceActionService::confirm`, `dismiss`, `retry` | Reconcile exact occurrence after expected/over-limit/failed/recorded/dismissed transitions; reviewable failure updates the same identity. |
| `RecurringTransactions/RecurringTransactionService::create`, `update`, `pause`, `resume`, `end`, archive-triggered pause methods | Rule edits can represent due dates before change; reconcile created occurrences, but rule pause/end alone must not resolve existing review items. |
| `Budgets/BudgetService::createMonth`, `addPlan`, `updatePlan`, `removePlan`, `copyMonth` | Capture current-month plan creation/target change/removal and affected month. Plan removal resolves active attention. Copy only qualifies in its destination month if current. Historical month edits create no new threshold alert. `updatePlan` and `removePlan` currently save/delete without an explicit transaction; wrap fact and mutation atomically. |
| `FinancialGoals/FinancialGoalMutationService::create`, `update`, `moneyAction`, `lifecycle` | Capture target and allocation first-qualification within the idempotency transaction, including a brief reach that later falls before drain. Completion/archive resolve or prevent further active milestones. |

## Authoritative reads and reconciliation

- Statements: `CreditCardObligationReconciler::outstandingCentavos`, `statusFor`, `syncStatement`, and `refreshCardStatements`.
- Budgets: `BudgetCalculationService::forMonth` for realized category-plan `BudgetStatus`; `BudgetMonthResolver::isEnded` for rollover. A card installment can affect a month other than purchase month.
- Goals: `FinancialGoalCapacityService::allocationForGoal` and `FinancialGoalQueryService::project`.
- Recurrences: generated `Transaction` and `RecurringCardOccurrence` states. Card `Failed` qualifies only where `CardOccurrenceState::isActionable` says reviewable.
- Date stages: scheduler evaluates local business date via `RecurringDateRange::BUSINESS_TIMEZONE`, plus a recovery scan for missed facts. Preserve first qualification time for preference history.

## Integration cautions

- Source services use idempotency callbacks. Hook inside accepted callbacks, not after a replayed result, and commit fact with source mutation.
- Some source methods are nested transactions. Fact writes must use the same connection/transaction; evaluator runs after outer commit.
- Payment, card credit, purchase correction, and transaction corrections can affect old and new budget months. Ended months may reconcile history but cannot emit a new threshold identity.
- Existing `CreditCardStatementService` mutates status during reads; notification evaluation must avoid a second financial calculation or unsafe recursive hook.

## Implementation reconciliation — 2026-09-29

The accepted financial mutation hooks were checked again after implementation. Statement payment, credit, purchase, and correction services capture statement facts; transaction mutations capture budget dates and generated-transaction review state; recurrence creation/action services capture exact card or ordinary occurrences; budget plan mutations capture affected months; goal mutations capture qualification inside their accepted transactions. The notification feature and source-domain regressions exercise these paths.

Two inventory rows use current-state reconciliation rather than a direct projection fact at the rule/card metadata operation: `CreditCardService` card metadata and archive changes are picked up by the minute statement scan and by current authorization on list/summary/open; `RecurringTransactionService` rule pause/end alone does not resolve existing reviewable occurrences, while occurrence creation/action paths carry the facts. This is the implemented behavior, so those rows are not evidence of missing source-transaction hooks for a newly qualifying financial event. `CreditCardStatementService::syncStatement` captures a fact after its explicit state sync; its list/detail/refresh presentation reads do not capture a notification fact.

# Financial Goals acceptance walkthrough

Date: 2026-09-26. Branches: `015-financial-goals` in specs, backend, and frontend.

| Quickstart step | Evidence | Result |
| --- | --- | --- |
| 1. Create linked R$ 30.000 goal with R$ 3.000 designation from R$ 10.000 account | `GoalCreationContractTest`; `GoalFinancialIntegrityTest` | Passed: R$ 7.000 unallocated, unchanged real balance and financial ledgers. |
| 2. Add, withdraw, replay, reject changed payload | `GoalMoneyActionContractTest`; frontend store retry tests; browser goals journey | Passed: one event per accepted action and immutable replay. |
| 3. Shared capacity, real spending, and shortfall attention | `GoalConcurrencyTest` with `GOAL_MYSQL_CONCURRENCY=1`; `GoalAccountCoverageTest`; `GoalOverviewDashboardContractTest`; mobile browser shortfall journey | Passed: overlapping MySQL claims serialize; shortfall remains explicit. |
| 4. Unlinked amount, reassociation, historical account snapshots | `GoalCreationContractTest`; `GoalUpdateContractTest`; mobile browser reassociation journey | Passed: unverified amount stays separate; old events keep prior account context. |
| 5. Overfunding and explicit complete/reopen/archive/restore | `GoalProjectionTest`; `GoalCompletionBackingTest`; `GoalLifecycleContractTest`; browser lifecycle journey | Passed: real progress can exceed 100%, completed funds remain designated, archive requires zero. |
| 6. Date guidance and no automatic money activity | `GoalContributionCalculatorTest`; `GoalDateGuidanceContractTest`; browser future/overdue journey | Passed: inclusive months and no recurring transaction. |
| 7. Inactive account attention and established financial views | `GoalAccountCoverageTest`; `GoalOverviewDashboardContractTest`; `GoalFinancialIntegrityTest`; Dashboard browser regression | Passed: counts are independent, while account, history, budget, card, and cash-flow figures retain their meaning. |

API request validation now rejects extra mutation fields, matching `additionalProperties: false` in the OpenAPI contract. The published contract lints without warnings. The complete backend suite, frontend unit suite, lint, build, and isolated browser journeys are recorded in task T069/T070. The representative-user outcomes remain pending in `usability.md`.

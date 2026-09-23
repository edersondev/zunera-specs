# Quickstart: Expandable Statement Transactions

1. Verify all three repositories are on `012-statement-transactions-expandable-list`.
2. Run `docker exec zunera-backend-app-1 php artisan test --filter=CreditCardStatementTest` after backend response change.
3. Run `npm run test:unit -- --run`, `npm run build`, and `npm run lint` in frontend.
4. Run `CI=1 npm run test:e2e -- e2e/credit-cards.spec.js` after build.
5. Inspect statement with multiple installments at desktop and 320px in light/dark themes; toggle by keyboard; open Correct and Refund; compare server totals before and after.

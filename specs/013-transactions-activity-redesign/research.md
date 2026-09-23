# Research: Transactions Activity Redesign

- **Decision**: Reuse dashboard custom-period summary. **Rationale**: history metadata totals are lifetime-wide; dashboard totals are authoritative for realized ordinary transactions within inclusive dates. **Alternative**: client summation is wrong under pagination and credit-card recognition.
- **Decision**: Hide summary under non-date filters. **Rationale**: current dashboard endpoint does not accept those filters. **Alternative**: showing unfiltered values next to filtered rows is misleading.
- **Decision**: Use `movement_date` and existing server order. **Rationale**: transfer, ordinary, recurring, and card-expense dates have different domain meanings already resolved by backend.
- **Decision**: Extract shared month navigator while retaining Budgets wrapper. **Rationale**: preserves Budgets behavior and test selectors; avoids divergent controls.
- **Decision**: Keep load-more. **Rationale**: existing pagination contract and 50-item page size remain unchanged.

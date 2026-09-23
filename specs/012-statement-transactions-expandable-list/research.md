# Research: Expandable Statement Transactions

- **Decision**: Enrich only statement installment response with purchase metadata. **Reason**: Current installment has no purchase ID, category, or original total; listing all card purchases to infer links would be paginated and costly. **Alternative**: Scan purchase-list pages.
- **Decision**: Fetch the full purchase only when Correct or Refund is selected. **Reason**: Existing dialogs need full purchase data and action rules. **Alternative**: Embed full purchase on every installment, duplicating nested history.
- **Decision**: Keep payments and credit events in their current sections. **Reason**: They are distinct financial records and user requested no calculation or business-rule change. **Alternative**: Merge all records into a visually uniform list.
- **Decision**: Use one open installment ID in the list component. **Reason**: Stable identifier, compact view, automatic collapse when list changes. **Alternative**: Per-item state or array index.

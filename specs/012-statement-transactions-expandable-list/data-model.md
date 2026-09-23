# Data Model: Expandable Statement Transactions

No stored model changes. The existing statement owns installments; each installment belongs to one purchase. The response uses that relation to expose purchase ID, category metadata, original purchase amount and direct-edit eligibility. Installment amount, credit adjustment, recognized amount and recognition status remain authoritative server values. The frontend stores only the open installment ID and the purchase loaded for an existing action dialog.

# Feature Specification: Financial Accounts

**Feature Branch**: `002-financial-accounts`  
**Backend Branch**: `002-financial-accounts` (`../zunera-backend`)  
**Frontend Branch**: `002-financial-accounts` (`../zunera-frontend`)  
**Created**: 2026-09-02  
**Status**: Draft  
**Input**: User description: "Create the Financial Accounts feature for Zunera.
Authenticated users must manage the accounts where they keep and organize money,
including creation, viewing, details, editing, archiving, restoration, balances,
Brazilian currency precision, ownership restrictions, lifecycle behavior, and a
responsive experience that follows the existing Zunera design guidance."

## Clarifications

### Session 2026-09-02

- Q: Which lifecycle states should financial accounts use? → A: Use lifecycle
  states `active` and `archived`; "deactivate" and "inactive" are user-facing
  synonyms for archived.
- Q: When can users edit an account's initial balance? → A: Initial balance is
  editable only until the account has financial movements.
- Q: Should financial accounts support permanent deletion? → A: Exclude
  permanent deletion entirely; users archive accounts instead.
- Q: How should users associate a financial institution with an account? → A:
  Institution is optional user-entered text.
- Q: Should financial-account names be unique? → A: Account names must be
  unique among each user's active accounts.
- Q: How should account color and icon customization work? → A: Color and icon
  must come from predefined accessible choices.
- Q: What maximum balance should financial accounts support? → A: Cap balances
  at `R$ 9.999.999.999,99` per account.
- Q: How should account names be compared for active-name uniqueness? → A: Trim,
  collapse spaces, and compare case- and accent-insensitively.
- Q: Can users archive their last active financial account? → A: Allow archiving
  the last active account.
- Q: Should account name be treated as a user-defined label separate from
  financial institution? → A: Account name is a user-defined label separate from
  the optional financial institution.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create and Review Accounts (Priority: P1)

As an authenticated user, I want to create financial accounts and see my active
accounts with their balances so that I can start organizing where my money is
held in Zunera.

**Why this priority**: A user cannot manage personal finances in Zunera until
they can register at least one account and see the money currently represented
by those accounts.

**Independent Test**: A signed-in user can create an account with valid details,
see clear creation feedback, and then view that account in their active account
list together with the current balance and combined active-account balance.

**Acceptance Scenarios**:

1. **Given** an authenticated user has no financial accounts, **When** they
   create a checking account with a valid account name, optional financial
   institution, and initial balance, **Then** the account is created for that
   user, success feedback is shown, and the account appears in the user's active
   account list.
2. **Given** an authenticated user has active financial accounts, **When** they
   view their accounts, **Then** they see each active account's name, type,
   current balance, and visual identity when provided.
3. **Given** an authenticated user has multiple active accounts, **When** they
   view their accounts, **Then** the combined balance equals the sum of the
   current balances of active accounts only.
4. **Given** an authenticated user enters missing or invalid account information,
   **When** they attempt to create an account, **Then** no account is created and
   each affected value receives clear correction guidance.
5. **Given** an authenticated user enters a Brazilian currency amount, **When**
   the amount is accepted, **Then** the value is stored, displayed, and reused
   without losing cents or changing the intended amount.
6. **Given** an authenticated user already has an active account with a name,
   **When** they attempt to create another active account with the same name
   after trimming, repeated-space cleanup, case folding, and accent-insensitive
   comparison, **Then** no duplicate active account is created and the user
   receives clear name-conflict feedback.

---

### User Story 2 - View and Update Account Details (Priority: P2)

As an authenticated user, I want to view and edit the details of one of my
financial accounts so that account information remains accurate as my finances
change.

**Why this priority**: Account data often needs correction or refinement after
creation, and users need a reliable detail view before future financial
movements depend on the account.

**Independent Test**: A signed-in user can open an account they own, review all
stored details, update editable descriptive fields, and see clear update
feedback without changing ownership or historical financial meaning.

**Acceptance Scenarios**:

1. **Given** an authenticated user owns a financial account, **When** they open
   its details, **Then** they see its name, type, financial institution when
   provided as user-entered text, color, icon, lifecycle state, initial balance,
   current balance, and creation/update context that is relevant to the user.
2. **Given** an authenticated user owns an active account, **When** they update
   the account name, type, financial institution, color, icon, or eligible
   initial balance with valid values, **Then** the changes are saved and success
   feedback is shown.
3. **Given** an authenticated user submits invalid updates, **When** they try to
   save, **Then** the account remains unchanged and clear correction guidance is
   shown for each invalid value.
4. **Given** an authenticated user tries to view or update an account belonging
   to another user, **When** the action is attempted, **Then** access is denied
   without exposing the other user's account details.

---

### User Story 3 - Archive and Restore Accounts (Priority: P3)

As an authenticated user, I want to archive accounts I no longer use and restore
them when appropriate so that old account history remains available without
cluttering new financial operations.

**Why this priority**: Lifecycle control protects historical financial
information while keeping active-account choices focused on accounts the user
currently uses.

**Independent Test**: A signed-in user can archive one of their accounts, confirm
it no longer appears in normal active-account choices, still access its
historical details when needed, and restore it when the account state allows.

**Acceptance Scenarios**:

1. **Given** an authenticated user owns an active account, **When** they archive
   it, **Then** the account state changes to archived, success feedback is
   shown, and it no longer appears among accounts normally available for new
   financial operations.
2. **Given** an authenticated user has archived accounts, **When** they choose to
   view archived accounts, **Then** those accounts and their historical
   information remain accessible.
3. **Given** an authenticated user owns an archived account that can be used
   again, **When** they restore it, **Then** the account becomes active again and
   success feedback is shown.
4. **Given** an account is already archived, already active, or otherwise in a
   state that prevents the requested lifecycle action, **When** the user attempts
   that action, **Then** the action is rejected with clear state-based feedback
   and no unintended account changes occur.
5. **Given** an authenticated user has exactly one active account, **When** they
   archive that account, **Then** the action succeeds, the active-account list
   becomes empty, and the combined active-account balance is shown as zero.

### Edge Cases

- An unauthenticated visitor attempts any financial-account action.
- A user attempts to access, modify, archive, restore, or otherwise interact with
  a financial account that belongs to another user.
- A user submits a blank name, an excessively long name, unsupported account
  type, malformed color value, unsupported icon choice, or invalid financial
  institution value.
- A user creates or restores an account using a name already used by one of
  their active accounts.
- A user creates, renames, or restores an account using a name that differs from
  an active account only by surrounding spaces, repeated spaces, letter casing,
  or accents.
- A user enters the name of a financial institution that is not known to Zunera
  but is still meaningful to the user.
- A user skips optional color or icon customization and expects the account to
  remain identifiable with default visual choices.
- A user enters monetary values with Brazilian separators, negative signs,
  excessive decimal places, or values beyond `R$ 9.999.999.999,99` in either
  direction.
- A user creates an account with an initial balance of zero.
- A user creates an account with a negative initial balance to represent an
  account that starts overdrawn or otherwise below zero.
- A repeated submission caused by double-clicking, network retry, or refreshed
  feedback must not create duplicate accounts or apply the same lifecycle action
  twice.
- An archived account has historical financial information that must remain
  visible in historical views but hidden from normal new-operation account
  choices.
- A restored account must preserve its prior initial balance, history, and
  descriptive information unless the user explicitly edits allowed details.
- A user attempts to edit the initial balance after the account has financial
  movements.
- A user expects to remove an account permanently but the feature only supports
  archiving to preserve financial history.
- A user archives their last active account and has no accounts available for
  normal new financial operations until an account is created or restored.
- Theme changes, narrow screens, keyboard-only navigation, assistive technology,
  and high zoom must not prevent users from completing the main account journeys.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST allow authenticated users to create financial
  accounts that belong only to the creating user.
- **FR-002**: The system MUST prevent unauthenticated users from viewing,
  creating, editing, archiving, restoring, or otherwise interacting with
  financial accounts.
- **FR-003**: The system MUST ensure users can view, modify, archive, restore,
  and otherwise interact only with financial accounts they own.
- **FR-004**: A financial account MUST include a user-provided account name, a
  supported account type, an initial balance, a lifecycle state, and a current
  balance.
- **FR-005**: A financial account MAY include a financial institution, color, and
  icon when the user chooses to provide them.
- **FR-006**: Supported account types MUST include checking account, savings
  account, cash or wallet, investment account, digital account, and other.
- **FR-007**: Account types MUST describe the financial nature of the account and
  MUST NOT impose rules that prevent future financial features from defining
  their own behavior.
- **FR-008**: The system MUST treat account name as the user's label for the
  account, keep it separate from the optional financial institution value, and
  reject missing, empty, or clearly unusable names with actionable feedback.
- **FR-009**: Account names MUST be unique among each user's active accounts
  after trimming surrounding spaces, collapsing repeated internal spaces, and
  comparing without case or accent differences.
- **FR-010**: Account-name uniqueness MUST be checked when creating, renaming, or
  restoring an account to active state, and conflicts MUST be rejected with clear
  feedback.
- **FR-011**: Archived accounts MAY retain names that match active accounts, but
  restoring or renaming an archived account MUST NOT create a duplicate active
  account name.
- **FR-012**: The system MUST validate selected account types before creation or
  update and reject unsupported types with actionable feedback.
- **FR-013**: The system MUST treat financial institution as optional
  user-entered text and MUST NOT require the institution to match a managed
  catalog in this feature.
- **FR-014**: The system MUST validate optional financial institution, color, and
  icon values before saving them and reject unsupported values with actionable
  feedback.
- **FR-015**: Account color and icon customization MUST use predefined accessible
  choices and MUST provide defaults when the user does not choose custom visual
  identifiers.
- **FR-016**: The system MUST allow users to define an initial balance when
  creating an account.
- **FR-017**: The initial balance MUST represent money already available, owed,
  or invested when the user begins managing the account in Zunera and MUST NOT be
  presented or treated as ordinary user-created income.
- **FR-018**: Account balances MUST reflect the account's initial balance plus or
  minus financial movements associated with that account once those future
  movement features exist.
- **FR-019**: Until financial-movement rules are defined by future features, an
  account's current balance MUST equal its initial balance unless another
  completed Zunera feature explicitly changes that account balance.
- **FR-020**: Monetary values MUST support Brazilian currency conventions,
  including reais and centavos, and MUST preserve exact cent-level precision in
  accepted values, displayed values, and balance calculations.
- **FR-021**: The system MUST accept per-account monetary values from
  `-R$ 9.999.999.999,99` through `R$ 9.999.999.999,99` and reject values outside
  that range with clear feedback.
- **FR-022**: The system MUST show users their active financial accounts with
  each account's name, type, current balance, lifecycle state, and optional
  visual identity when provided.
- **FR-023**: The system MUST show the combined balance across active accounts
  only and MUST exclude archived accounts from that normal combined balance.
- **FR-024**: The system MUST allow users to view the details of a financial
  account they own, including initial balance, current balance, lifecycle state,
  type, name, optional institution, optional color, optional icon, and relevant
  account history context.
- **FR-025**: The system MUST allow users to edit an existing account they own
  while preserving ownership and historical financial information.
- **FR-026**: The system MUST allow users to edit an account's initial balance
  only while the account has no financial movements.
- **FR-027**: Once an account has financial movements, the system MUST preserve
  the original initial balance and reject initial-balance edits with clear
  state-based feedback.
- **FR-028**: The system MUST allow users to archive an account they own when it
  is no longer in use; user-facing wording MAY describe this as deactivating or
  making the account inactive.
- **FR-029**: The system MUST allow users to archive their last active account.
- **FR-030**: When a user has no active accounts, the system MUST show an empty
  active-account state and a combined active-account balance of zero.
- **FR-031**: Accounts with financial history MUST be archived rather than
  permanently removed.
- **FR-032**: Archived accounts MUST remain accessible in historical or
  archived-account views when needed.
- **FR-033**: Archived accounts MUST NOT normally appear among accounts available
  for new financial operations.
- **FR-034**: The system MUST allow users to restore an archived account they own
  when the account state allows restoration.
- **FR-035**: The system MUST reject lifecycle actions that cannot be completed
  because of the account's current state and MUST explain the state conflict in
  clear user-facing language.
- **FR-036**: The system MUST NOT provide permanent financial-account deletion in
  this feature; users MUST archive accounts they no longer use.
- **FR-037**: Every create, update, archive, and restore action MUST provide clear
  success feedback when completed.
- **FR-038**: Invalid information MUST produce clear, specific, and actionable
  feedback without saving invalid account changes.
- **FR-039**: Unauthorized access attempts MUST be denied with privacy-safe
  feedback that does not reveal another user's financial-account details.
- **FR-040**: Financial-account journeys MUST support Light, Dark, and System
  themes while preserving readability, focus visibility, and contrast.
- **FR-041**: Financial-account journeys MUST remain usable across supported
  screen sizes, keyboard-only navigation, assistive technology, and high zoom.
- **FR-042**: Financial-account journeys MUST follow the existing Zunera Design
  Foundation, application shell, navigation rules, and shared component
  guidelines for layout, feedback, states, and responsive behavior.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: Every financial-account action MUST require confirmed
  authentication and MUST deny access by default when ownership or signed-in state
  cannot be confirmed.
- **SQR-002**: Every user-provided account value MUST be validated before it can
  create or change a financial account.
- **SQR-003**: Financial-account feedback MUST avoid exposing another user's
  account existence, names, balances, institutions, lifecycle state, or other
  personal financial details.
- **SQR-004**: Automated coverage MUST prove ownership restrictions, validation
  behavior, lifecycle state rules, initial-balance behavior, Brazilian currency
  precision, current-balance display, combined active-account balance, and the
  primary user journeys.
- **SQR-005**: Financial-account data MUST be handled as personal financial
  information and used only for authenticated account-management experiences and
  related future financial features.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): Account-management business behavior,
   authorization, validation, persistence, balance rules, lifecycle rules,
   user-owned data boundaries, and automated test requirements.
2. **Frontend** (`../zunera-frontend`): Account-management user journeys,
   responsive screens, forms, state feedback, theme support, accessibility
   behavior, and automated test requirements that depend on the confirmed
   backend behavior.

### Key Entities *(include if feature involves data)*

- **Financial Account**: A user-owned place where money is kept or organized.
  Key information includes account name, account type, initial balance, current
  balance, lifecycle state, optional institution, optional color, optional icon,
  ownership, whether financial movements exist, normalized active-name
  uniqueness, predefined visual identity, and relevant timestamps or history
  context. The account name is the user's label and is distinct from the optional
  financial institution.
- **Account Type**: The financial nature of an account, such as checking,
  savings, cash or wallet, investment, digital account, or other. Account types
  classify accounts without deciding future transaction behavior.
- **Lifecycle State**: Whether an account is active or archived. User-facing
  wording may describe archiving as deactivating or making an account inactive,
  but the lifecycle behavior remains the same: archived accounts are removed from
  normal new-operation choices while preserving historical access.
- **Financial Institution**: Optional user-entered text that helps identify where
  an account is held, without being required for cash, wallet, or other account
  types and without requiring a managed catalog.
- **Balance**: The monetary state of a financial account. For this feature, it is
  based on the initial balance unless completed future features add financial
  movements that affect the account.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 95% of users in acceptance testing can create a financial
  account with valid information in under 2 minutes without assistance.
- **SC-002**: At least 95% of users in acceptance testing can find the current
  balance of an active account and the combined active-account balance in under
  30 seconds.
- **SC-003**: At least 90% of users in acceptance testing can update account
  details, archive an account, and restore it without needing external
  instructions.
- **SC-004**: In validation testing, 100% of attempts to access another user's
  financial account are denied without displaying personal financial details.
- **SC-005**: In monetary validation testing, 100% of accepted Brazilian currency
  values retain their intended reais-and-centavos amount through creation,
  display, update where allowed, and combined-balance calculation.
- **SC-006**: In responsive and theme validation, the primary account journeys
  remain usable at supported screen sizes, in Light, Dark, and System themes, and
  with keyboard-only navigation.

## Assumptions

- Users already have a signed-in Zunera account before using financial-account
  management.
- This feature covers personal financial accounts owned by one user and does not
  include shared, household, team, or business-owned accounts.
- This feature does not define transaction entry, income, expenses, transfers,
  investments, reconciliation, statements, imports, or detailed movement-based
  balance rules; those belong to future features.
- Initial balance can be zero, positive, or negative so users can represent
  accounts that begin empty, funded, overdrawn, or below zero.
- Initial balance corrections are allowed before any financial movement exists;
  after that point, future movement or adjustment features handle corrections.
- Permanent deletion is out of scope for every financial account in this feature;
  the archived state preserves historical information.
- Deactivate and inactive are user-facing synonyms for archived behavior rather
  than separate lifecycle states.
- Brazilian real is the expected currency convention for this feature.
- Financial institutions are stored as user-entered descriptive text in this
  feature; managed institution catalogs or bank integrations are out of scope.
- Account name is the user's label for identifying the account, such as "Conta
  principal" or "Nu salário", and is separate from the optional financial
  institution, such as "Nubank" or "Banco do Brasil".
- Account names are unique among active accounts owned by the same user; archived
  accounts may keep duplicate names until restoration or rename would make them
  active duplicates. Comparison trims surrounding spaces, collapses repeated
  internal spaces, and ignores case and accent differences.
- Color and icon customization uses predefined accessible choices with defaults
  when users do not customize visual identifiers.
- Each account balance supports values from `-R$ 9.999.999.999,99` through
  `R$ 9.999.999.999,99`; broader limits are future scope.
- Users may have zero active accounts after archiving their last active account;
  the active combined balance is zero in that state.

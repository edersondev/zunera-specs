# Feature Specification: Category Management

**Feature Branch**: `003-category-management`  
**Backend Branch**: `003-category-management` (`../zunera-backend`)  
**Frontend Branch**: `003-category-management` (`../zunera-frontend`)  
**Created**: 2026-09-04  
**Status**: Draft  
**Input**: User description: "Create Categories feature for Zunera. Authenticated
users organize financial transactions with system-provided and personal
categories, including classification, visual identification, ownership, and
archive/restore behavior."

## Clarifications

### Session 2026-09-04

- Q: When may a personal category's financial classification change? → A:
  Classification becomes immutable after the category is first used.
- Q: May users hide or archive system default categories? → A: Defaults always
  visible and read-only.
- Q: Which financial classifications are supported in this feature? → A: Income
  and expense only.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Browse Categories (Priority: P1)

As an authenticated user, I want to browse categories available to me and see
their financial classification, so that I can choose an appropriate category
when organizing financial activity.

**Why this priority**: Categories cannot help users organize finances until
they can find the available defaults and any categories they created.

**Independent Test**: A signed-in user can open category management and see
active default and personal categories, their names, classifications, origin,
and visual identifiers, without seeing another user's personal categories.

**Acceptance Scenarios**:

1. **Given** an authenticated user has created no personal categories, **When**
   they view available categories, **Then** they see active system defaults for
   income and expense classifications, clearly distinguished from personal
   categories.
2. **Given** an authenticated user has active personal categories, **When**
   they view available categories, **Then** they see those categories together
   with the applicable active system defaults, including each category's name,
   classification, origin, and applied visual identity.
3. **Given** another user has personal categories, **When** an authenticated
   user views available categories, **Then** none of the other user's personal
   category names, visual identifiers, or lifecycle information is shown.
4. **Given** an unauthenticated visitor attempts to view categories, **When**
   access is requested, **Then** access is denied without exposing category
   information.

---

### User Story 2 - Create and Update Personal Categories (Priority: P1)

As an authenticated user, I want to create and edit my own categories, so that
my transaction organization reflects how I manage my finances.

**Why this priority**: Default categories give users a starting point, but
personal categories are necessary for meaningful day-to-day organization.

**Independent Test**: A signed-in user can create a valid personal category,
find it in their active categories, update its allowed details, and receive
clear feedback for both successes and invalid or conflicting submissions.

**Acceptance Scenarios**:

1. **Given** an authenticated user is viewing their categories, **When** they
   create a personal category with a valid name and supported classification,
   **Then** it belongs only to that user, appears among their active categories,
   and success feedback is shown.
2. **Given** an authenticated user creates a personal category and chooses a
   permitted color or icon, **When** it is saved, **Then** the selected visual
   identifiers are shown with the category without changing the meaning of
   income, expense, warning, error, or other application-state colors.
3. **Given** an authenticated user owns a personal category with no financial
   transaction history, **When** they edit its name, supported classification,
   color, or icon with valid information, **Then** the changes are saved and
   update feedback is shown.
4. **Given** a user submits a blank, unusable, unsupported, or conflicting
   category definition, **When** they attempt to create or update it, **Then**
   no invalid change is made and clear, field-relevant correction feedback is
   shown.
5. **Given** a user attempts to view or change a personal category owned by
   another user, **When** the action is attempted, **Then** it is denied without
   exposing the other user's category details.
6. **Given** an authenticated user owns a personal category already associated
   with financial history, **When** they attempt to change its classification,
   **Then** the classification remains unchanged and clear state-based feedback
   explains that historical classification is preserved.

---

### User Story 3 - Archive and Restore Personal Categories (Priority: P2)

As an authenticated user, I want to archive a personal category I no longer
use and restore it later, so that new transaction choices stay focused without
losing historical organization.

**Why this priority**: Lifecycle control protects the meaning of prior
financial history while reducing clutter in future transaction selection.

**Independent Test**: A signed-in user can archive one of their personal
categories, confirm that it is unavailable for normal new transaction choices
while remaining attached to prior history, and restore it successfully.

**Acceptance Scenarios**:

1. **Given** an authenticated user owns an active personal category, **When**
   they archive it, **Then** it becomes archived, archive feedback is shown, and
   it no longer normally appears as a selectable category for new transactions.
2. **Given** an archived personal category is associated with prior financial
   transactions, **When** those historical transactions are viewed, **Then**
   they retain their association with that category and its historical meaning.
3. **Given** an authenticated user has archived personal categories, **When**
   they choose to view archived categories, **Then** those categories remain
   accessible and clearly marked as archived.
4. **Given** an authenticated user owns an archived personal category, **When**
   they restore it without creating a conflict, **Then** it becomes active and
   restoration feedback is shown.
5. **Given** a requested archive or restore action is already complete or cannot
   be completed in the category's current state, **When** the user attempts it,
   **Then** no unintended change occurs and clear state-based feedback is shown.

### Edge Cases

- A user has no personal categories and relies only on system defaults.
- A user has no active personal categories after archiving all of them.
- A user enters a blank, whitespace-only, or overlong category name; an
  unsupported classification; an invalid visual choice; or a duplicate name.
- A user creates, renames, changes the classification of, or restores a
  personal category whose name conflicts with an active category available to
  them in the same classification after normalization.
- A proposed personal category duplicates a system default's normalized name in
  the same classification, creating an ambiguous transaction choice.
- A repeated submit, retry, or refreshed feedback occurs while a create, update,
  archive, or restore action is being processed.
- A user tries to alter, archive, restore, or otherwise manage a system default
  beyond the rules for system-provided categories.
- A user attempts to change the classification of a personal category already
  associated with a financial transaction.
- An archived personal category already used by financial history is viewed in
  history or reports, where its previous name, classification, and visual
  identity remain meaningful.
- A user tries to archive, restore, or modify a category owned by another user.
- Category views and management actions are used in Light, Dark, or System
  theme; at narrow supported screen sizes; with keyboard-only navigation,
  assistive technology, or high zoom.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST allow authenticated users to view categories
  available to them.
- **FR-002**: The system MUST provide active system default categories for all
  authenticated users, clearly distinguish them from personal categories, and
  make clear that they are system-provided rather than user-created.
- **FR-003**: The default expense categories MUST include Housing, Food,
  Transportation, Health, Education, Entertainment, Shopping, Bills and
  utilities, Taxes, and Other expenses.
- **FR-004**: The default income categories MUST include Salary, Freelance or
  services, Investments, Gifts, Refunds, and Other income.
- **FR-005**: Each category MUST identify exactly one financial classification.
  This feature supports income and expense only; the system MUST reject any
  other classification with actionable feedback.
- **FR-006**: The system MUST allow an authenticated user to create a personal
  category with a meaningful name and one supported classification.
- **FR-007**: Each personal category MUST belong to exactly one user. A user
  MUST be able to view, modify, archive, restore, or otherwise manage only
  their own personal categories.
- **FR-008**: System default categories MUST remain visible and read-only for
  authenticated users. They MUST NOT be treated as categories owned, editable,
  hideable, archivable, or restorable by individual users. Attempts to perform
  those actions MUST receive clear state-based feedback.
- **FR-009**: A category name MUST contain 1–120 user-visible characters after
  trimming and MUST include at least one non-whitespace character. The system
  MUST reject blank, whitespace-only, or overlong names with actionable
  feedback.
- **FR-010**: An active personal category name MUST be unique among categories
  available to the same user within its financial classification, after trimming
  surrounding spaces, collapsing repeated internal spaces, and comparing
  without case or accent differences.
- **FR-011**: The system MUST reject a create, rename, classification change, or
  restore that would make a personal category's normalized name duplicate an
  active personal category or applicable system default in the same financial
  classification, and MUST explain the conflict clearly.
- **FR-012**: An archived personal category MAY retain a duplicate name, but a
  restore or change that would make it an active conflict MUST be rejected until
  the conflict is resolved.
- **FR-013**: The system MUST allow a user to update the name and optional
  visual identifiers of a personal category they own while preserving ownership
  and historical transaction associations. Its classification MAY be updated
  only before it is associated with a financial transaction.
- **FR-014**: Once a personal category is associated with a financial
  transaction, the system MUST preserve its classification and reject any
  classification change with clear state-based feedback.
- **FR-015**: A personal category MAY have an optional color and icon for visual
  identification and reporting. If provided, each MUST be a permitted choice
  that follows the Zunera Design Foundation; it MUST NOT be used to communicate
  income, expense, warning, error, or other application-state semantics.
- **FR-016**: The system MUST allow a user to archive an active personal
  category they own. User-facing wording MAY use deactivate or inactive as
  synonyms for archived behavior.
- **FR-017**: An archived personal category MUST not normally appear as a
  selectable option for new transactions, but it MUST remain available wherever
  needed to preserve its association with prior financial transactions.
- **FR-018**: The system MUST allow a user to view and restore an archived
  personal category they own when its current state allows restoration.
- **FR-019**: The system MUST NOT permanently remove a category used in
  financial history in a way that breaks, removes, or changes historical
  transaction information. This feature provides archiving instead of permanent
  category deletion.
- **FR-020**: The system MUST reject invalid category configurations, duplicate
  or conflicting definitions, unauthorized actions, and unavailable lifecycle
  actions without making an unintended category change.
- **FR-021**: Every completed category creation, update, archive, and restore
  action MUST show clear success feedback. Every rejected action MUST show clear
  and privacy-safe feedback relevant to its cause.
- **FR-022**: Category management MUST follow the existing Zunera Design
  Foundation, application shell, navigation rules, and shared component
  guidelines.
- **FR-023**: Category management MUST remain readable and usable in Light,
  Dark, and System themes; across supported screen sizes; and with keyboard-only
  navigation, assistive technology, and high zoom.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: Category-management actions MUST require authentication and
  default to denial when signed-in state or personal-category ownership cannot
  be confirmed.
- **SQR-002**: Every user-provided category value and requested category action
  MUST be validated before it can create or change a category.
- **SQR-003**: Unauthorized category access feedback MUST not reveal another
  user's personal category names, classifications, visual identifiers,
  lifecycle state, transaction associations, or existence.
- **SQR-004**: Automated coverage MUST prove default availability and
  distinction, ownership boundaries, validation, normalized duplicate handling,
  lifecycle rules, historical-association preservation, and primary user
  journeys.
- **SQR-005**: Personal category data and category-to-transaction associations
  MUST be handled only within authenticated financial-management experiences.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): Define and protect category behavior,
   ownership, validation, default-category rules, lifecycle behavior,
   historical-association protection, and automated verification before
   frontend delivery begins.
2. **Frontend** (`../zunera-frontend`): Deliver category-management journeys,
   feedback, responsive and accessible theme-aware presentation, and automated
   verification that consume the confirmed backend behavior.

### Key Entities *(include if feature involves data)*

- **Category**: A named way to organize financial transactions. It has exactly
  one financial classification, a lifecycle state, an origin, optional visual
  identifiers, and a name normalized for conflict detection.
- **Personal Category**: A category created by and belonging to exactly one
  user. Its editable descriptive information and lifecycle are managed only by
  that user.
- **System Default Category**: A system-provided, non-user-owned category made
  available to authenticated users under application rules. It is visibly
  distinct from a personal category.
- **Financial Classification**: The intended financial meaning of a category.
  This feature supports income and expense only.
- **Category Lifecycle State**: Whether a personal category is active or
  archived. Archived categories are excluded from normal new-transaction
  selection but preserve historical associations.
- **Historical Transaction Association**: The retained connection between a
  financial transaction and the category previously chosen for it, including
  when that category is later archived.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 95% of acceptance-test users can find an available income
  or expense category and identify whether it is system-provided or personal in
  under 30 seconds.
- **SC-002**: At least 95% of acceptance-test users can create a valid personal
  category with a classification in under 1 minute without assistance.
- **SC-003**: At least 90% of acceptance-test users can update, archive, and
  restore their own personal category without external instructions.
- **SC-004**: In authorization and privacy validation, 100% of attempts to view
  or manage another user's personal category are denied without showing that
  category's details.
- **SC-005**: In lifecycle validation, 100% of archived categories previously
  used by financial history remain associated with those historical
  transactions, while 100% are absent from normal new-transaction selections.
- **SC-006**: In responsive, theme, and accessibility validation, all primary
  category journeys remain usable at supported screen sizes, in Light, Dark,
  and System themes, and with keyboard-only navigation.

## Assumptions

- Users already have an authenticated Zunera account before accessing category
  management.
- This feature covers personal financial categories only. Shared, household,
  team, and business-owned categories are out of scope.
- System defaults are available and visible to every authenticated user, are not
  owned by individual users, and remain read-only in this feature; individual
  hiding, archiving, restoration, or customization of defaults is out of scope.
- The listed income and expense defaults are the required set. This feature
  supports income and expense classifications only; any additional
  classification requires a future feature with its own rules.
- A personal category may not duplicate an active category available to its
  owner in the same classification, including a system default. This avoids
  ambiguous transaction choices.
- Category name comparison trims surrounding spaces, collapses repeated internal
  spaces, and ignores case and accent differences. Archived personal categories
  may retain a conflicting name until restored or changed to active.
- Archived is the single lifecycle state for no-longer-used personal categories;
  deactivate and inactive are user-facing synonyms, not separate states.
- A personal category's classification can change only before its first
  financial transaction association; afterward it remains immutable so historic
  transaction meaning cannot be reclassified.
- No permanent category deletion is in scope. Archiving preserves historical
  financial information.
- Colors and icons are optional visual identifiers chosen from choices that
  follow the Zunera Design Foundation. They do not replace semantic application
  colors or status signals.
- Creating or editing financial transactions, reports, budgets, nested
  categories, category sharing, transfer classifications, and custom
  financial-classification management are outside this feature's scope.

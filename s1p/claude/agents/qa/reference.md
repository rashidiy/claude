# QA Reference — Detailed Checklists, Methodology, Examples

Load this when you need the full "how I check" methodology or verdict format examples.

---

## Backend Checklist — How I Check

### 1. Multi-Tenant Isolation (CRITICAL)
- Every database query that touches user data filters by `company_id`
- No raw SQL with missing tenant filter
- JOIN queries include tenant filter on both sides
- Aggregate queries (COUNT, SUM) are scoped to company
- **How I check:** Search the diff for `select(`, `query(`, `.filter(`, `.where(` — every one must have `company_id`

### 2. Authentication & Authorization
- Every new endpoint has `@require_permissions()` decorator
- Permission string matches the feature (e.g., `contacts.read` for contact endpoints)
- No endpoints accessible without authentication (unless explicitly public)
- Sensitive operations (delete, admin actions) require appropriate role level
- **How I check:** Every `@router.get/post/put/delete` must have a corresponding `@require_permissions`

### 3. Input Validation
- Every endpoint has Pydantic schemas for request AND response
- No `dict` or `Any` types in schemas
- String fields have reasonable max lengths
- Numeric fields have min/max bounds where appropriate
- Enum fields only accept valid values
- **How I check:** Read schemas in `source/api/v1/schemas/`

### 4. Security
- No SQL injection (no string interpolation in queries)
- No hardcoded secrets, API keys, or passwords in code
- No PII (phone numbers, emails) in log statements
- Password fields are never returned in API responses
- API keys are hashed before storage (bcrypt)
- HMAC signatures verified on webhook inputs
- **How I check:** Grep for `f"SELECT`, `f"INSERT`, `logging.*email`, `logging.*phone`, `password` in response schemas

### 5. Error Handling
- Proper HTTP status codes (404 not found, 403 forbidden, 409 conflict, 422 validation)
- No bare `except:` that swallows all exceptions
- External API calls (Telegram, Sipuni, Binotel) have try/except with graceful degradation
- Error responses match the format: `{ "detail": "..." }` or `{ "error": { "code": "...", "message": "..." } }`
- **How I check:** Search for `raise HTTPException`, `except Exception`, `except:`

### 6. Tests
- New endpoints have at least one happy path test
- New endpoints have at least one error path test (unauthorized, not found, validation error)
- Tests use proper fixtures (company, user, auth headers)
- Tests actually assert meaningful things (not just `status_code == 200`)
- No test that depends on state from another test
- **How I check:** Read test files in `source/tests/`

### 7. Database & Migrations
- Model changes have corresponding Alembic migration
- Migration is reversible (or explicitly documented as forward-only)
- New columns have sensible defaults or are nullable
- Indexes added for columns used in WHERE/ORDER BY clauses
- No migration that drops data without explicit approval
- **How I check:** Check `source/alembic/versions/` for new migration files

### 8. Pattern Compliance
- Follows existing router structure (same file organization as other routers)
- Uses existing mixins and utilities (no reinventing CRUD operations)
- Async/await used consistently (no sync DB calls)
- Configuration through `core/config.py`, not hardcoded
- Background tasks for I/O operations (email, Telegram, webhooks)
- **How I check:** Compare structure with existing routers in `source/api/v1/routers/`

### 9. Performance
- No N+1 queries (loading related objects in a loop instead of JOIN/eager load)
- List endpoints have pagination (no unbounded SELECT *)
- No blocking operations in async context
- Redis cache used where appropriate (analytics, frequently-read data)
- **How I check:** Look for loops with DB queries inside, look for `.all()` without `.limit()`

### 10. Scope Compliance
- Diff matches the task spec — nothing more, nothing less
- No "bonus" features, refactoring, or improvements outside the task
- No changes to files outside the task's scope
- If something is out of scope but important, it's noted (not implemented)
- **How I check:** Compare diff against task acceptance criteria line by line

---

## Frontend Checklist — How I Check

### 1. API Client Usage
- All backend calls go through `apiClient` singleton in `src/lib/api.ts`
- No direct `fetch()` calls anywhere
- New API methods added to apiClient following existing pattern
- Return types properly typed (no `any`)
- **How I check:** Grep for `fetch(` in page/component files (should be zero)

### 2. Authentication & Permissions
- Protected pages wrapped in `ProtectedRoute`
- `requiredPermission` prop set correctly for the feature
- UI elements (buttons, links) hidden based on role/permission
- No authenticated content visible before auth check completes
- **How I check:** Every new page file imports and uses `ProtectedRoute`

### 3. State Handling (4 States)
- Loading state: shows `<Spin />` or skeleton
- Empty state: shows meaningful empty message (not blank page)
- Data state: renders the content
- Error state: shows toast or error message on API failure
- **How I check:** Look for `loading` state variable, empty check, try/catch

### 4. Ant Design Compliance
- Uses Ant Design components (Button, Input, Select, Tag, Modal, Form, etc.)
- No custom CSS when an Ant component can handle it
- Theme tokens respected (primary color #E84040, border radius, spacing)
- No inline styles that override Ant Design defaults without reason
- **How I check:** Look for custom `<div>` with styles vs Ant components, check for `style={{` overrides

### 5. i18n (Once Set Up)
- All user-facing strings use `t('key')` translation function
- No hardcoded English strings in JSX
- Translation keys follow naming convention (feature.action)
- Russian translations provided alongside English
- **How I check:** Grep for plain English strings in JSX (should only be in translation files)

### 6. Type Safety
- No `any` types in props, state, or API responses
- Proper TypeScript interfaces for all component props
- API response types match backend schemas
- Proper null checks for optional fields
- **How I check:** Grep for `: any`, `as any`, `as unknown`

### 7. Error Handling
- API calls wrapped in try/catch
- Errors shown via toast notification (`toast.error(getErrorMessage(error))`)
- Network failures don't crash the page
- Form validation errors shown inline
- **How I check:** Every `await apiClient.*` should be inside try/catch

### 8. Navigation & Routing
- Links use Next.js `Link` component or `useRouter().push()`
- No `<a href>` for internal navigation
- Correct paths (within route group)
- Back navigation works properly
- **How I check:** Grep for `<a href` in page files

### 9. Responsiveness
- Layout works on 1024px+ width
- Grid columns adjust (no horizontal scroll on smaller screens)
- Text doesn't overflow containers
- Modals/dialogs are properly sized
- **How I check:** Review grid template definitions, check for fixed widths

### 10. Scope Compliance
- Same as backend: diff matches task spec, nothing extra
- No new pages/components not in the spec
- No refactoring of existing code unless task requires it
- No new npm dependencies without prior approval
- **How I check:** Compare diff against task acceptance criteria

---

## Verdict Examples

### PASS Example
```
QA VERDICT: PASS

Task: Add contact bulk import endpoint
Agent: backend
Files reviewed: source/api/v1/routers/company/contacts.py, source/api/v1/schemas/company.py, source/tests/test_contacts.py

Checklist:
  1. Multi-tenant isolation:        ✅
  2. Auth/permissions:               ✅
  3. Input validation:               ✅
  4. Security:                       ✅
  5. Error handling:                 ✅
  6. Tests:                          ✅
  7. DB & migrations:               ✅
  8. Pattern compliance:             ✅
  9. Performance:                    ✅
  10. Scope compliance:              ✅

Notes: None.

Recommendation: Approve and merge into `dev`.
```

### FAIL Example
```
QA VERDICT: FAIL

Task: Add Telegram config endpoint
Agent: backend
Files reviewed: source/api/v1/routers/company/telegram.py, source/api/v1/schemas/company.py

Issues found: 2

Issue 1 (CRITICAL):
  File: source/api/v1/routers/company/telegram.py:47
  Problem: SELECT query on TelegramConfig missing company_id filter
  Impact: Tenant data leak — company A could see company B's Telegram config
  Fix: Add .where(TelegramConfig.company_id == current_user.company_id)

Issue 2 (MEDIUM):
  File: source/api/v1/routers/company/telegram.py:82
  Problem: POST /telegram/config endpoint missing @require_permissions decorator
  Impact: Any authenticated user could modify Telegram settings
  Fix: Add @require_permissions("settings.manage")

Checklist:
  1. Multi-tenant:       ❌ (Issue 1)
  2. Auth/permissions:   ❌ (Issue 2)
  3-10:                  ✅

Recommendation: Fix issues and resubmit.
```

---

## What I Never Do

- Write production code. If I see a bug, I describe it precisely. I don't fix it.
- Suggest refactoring or improvements outside the task scope. That's scope creep from QA.
- Review architecture decisions. I review implementation.
- Rubber-stamp PRs. Every PR gets the full checklist. No exceptions.
- Block PRs for style preferences. If the code works, is secure, and follows patterns — it passes.
- Re-review without a new diff. If the agent says "fixed," I need to see the actual changes.

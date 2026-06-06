# Opinionated Defaults for PRD Creator

These are the best-practice defaults to present to the user during the interview. Present each as a recommendation, not a requirement. The user can confirm, modify, or override.

---

## Data Model Defaults

**Standard table scaffold (offer for every new table):**
```
id           UUID          PRIMARY KEY DEFAULT gen_random_uuid()
company_id   UUID          NOT NULL REFERENCES companies(id)
created_at   TIMESTAMPTZ   NOT NULL DEFAULT now()
updated_at   TIMESTAMPTZ   NOT NULL DEFAULT now()
created_by   UUID          REFERENCES auth.users(id)
```

**Soft delete (offer for financial/audit data):**
```
deleted_at   TIMESTAMPTZ   NULL  -- null = active, non-null = soft-deleted
deleted_by   UUID          REFERENCES auth.users(id)
```

**Optimistic concurrency:** Every table that will be mutated by multiple users should have `updated_at` as the concurrency token. The API accepts `updated_at` in mutation request bodies and returns 409 CONFLICT if the value is stale.

**State machine fields:** Use a string enum column (e.g., `status VARCHAR NOT NULL DEFAULT 'draft'`). Document every valid transition. Invalid transitions return 409 CONFLICT.

---

## API Defaults

**Response envelope:**
- Success: `{ ok: true, data: <payload> }`
- Error: `{ ok: false, error_code: "SNAKE_CASE_STRING", message: "Human-readable string", details?: {} }`

**Standard HTTP status codes:**
- 200: success (GET, PATCH)
- 201: created (POST)
- 204: no content (DELETE)
- 400: validation error (BAD_REQUEST)
- 401: not authenticated (UNAUTHORIZED)
- 403: not permitted (FORBIDDEN)
- 404: not found (NOT_FOUND)
- 409: conflict (STALE_RECORD, DUPLICATE, INVALID_TRANSITION)
- 429: rate limited (RATE_LIMITED)
- 500: server error (INTERNAL_ERROR)

**Optimistic concurrency (offer for any PATCH/PUT that touches shared records):**
- Client sends `updated_at` in request body
- Server compares to stored value; if different, returns `409 { error_code: "STALE_RECORD", current_updated_at: "..." }`
- Client toasts "This record was updated by someone else" and refreshes

**Rate limiting (offer for auth-adjacent and unauthenticated endpoints):**
- Auth endpoints: 10 requests per minute per IP
- Unauthenticated public endpoints: 30 requests per minute per IP
- Authenticated endpoints: 200 requests per minute per user

**Request ID header:** Every response carries `X-Request-Id`. The same ID is in server logs. Include in error toasts for support escalation.

---

## Frontend Defaults

**4 UI states (offer for every data-displaying component):**
1. **Loading** -- skeleton placeholder matching the layout of the populated state
2. **Empty** -- friendly message + clear call-to-action (e.g., "No invoices yet. Create your first one.")
3. **Error** -- error message + retry button; include X-Request-Id for support
4. **Populated** -- the data

**Mutation feedback:**
- Success: toast notification ("Invoice created." / "Changes saved.")
- Error: toast notification with error message and X-Request-Id
- Loading: button shows spinner, is disabled to prevent double-submit

**Mobile:** Responsive at 768px breakpoint. Touch targets minimum 44x44px. No hover-only interactions.

**Optimistic UI (offer for high-frequency mutations):** Update UI immediately on submit, roll back if the API call fails.

---

## Auth and Permissions Defaults

**Role hierarchy (standard pattern for multi-tenant SaaS):**
- Owner: full access including billing, user management, and destructive actions
- Admin: full access except billing and user management
- Editor/Bookkeeper: read + write on core data; cannot manage users or close periods
- Read-Only/Viewer: read-only access to all data
- Auditor: read-only access to audit log; no access to financial data

**Row-level scoping:** Every API endpoint that reads or writes data should filter by `company_id` extracted from the authenticated user's session. Never trust `company_id` from the request body.

---

## Performance Defaults

- API response P95: 200ms under normal load
- Page load (first meaningful paint): 3 seconds on 4G
- Background jobs (nightly sync, report generation): no SLA, but should complete within 10 minutes
- Alert condition: 5xx error rate >1% sustained for 5 minutes

---

## Acceptance Criteria Baseline

Present this list and ask the user to add feature-specific criteria:

- [ ] TypeScript compiles clean (`tsc --noEmit` exits 0)
- [ ] ESLint returns 0 errors
- [ ] All unit tests pass
- [ ] All integration tests pass against the test database
- [ ] All 4 UI states render correctly (loading, empty, error, populated)
- [ ] All API endpoints return documented HTTP status codes and error shapes
- [ ] All validation enforced on both client and server
- [ ] Invalid state transitions return 409 CONFLICT
- [ ] Mobile layout renders correctly at 375px viewport width
- [ ] No new `console.error` output in normal operation
- [ ] Feature reviewed by developer at localhost before deploy

---

## Security Checklist

Present this list for every feature with write operations:

- [ ] All endpoints require authentication (or explicitly documented as public)
- [ ] All endpoints validate `company_id` scoping server-side (never trust client)
- [ ] All user inputs validated and sanitized server-side
- [ ] File uploads: validate type, size limit enforced, stored outside web root
- [ ] No secrets or PII logged
- [ ] Rate limiting on auth-adjacent endpoints
- [ ] CSRF protection on state-changing endpoints (if cookie-based auth)

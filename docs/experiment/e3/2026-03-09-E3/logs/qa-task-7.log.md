# QA Log — T7: M6 — Frontend Infrastructure + Store

**Reviewer**: qa-task-7
**Date**: 2026-03-09
**Verdict**: QA-PASS

---

## STEP 1: Requirements <-> Tests

| Acceptance Criteria | Test Coverage | Status |
|---|---|---|
| axios with baseURL `/api` | `axios.test.ts:17` — checks `api.defaults.baseURL === '/api'` | PASS |
| JWT token interceptor (attach when present) | `axios.test.ts:20-29` — sets token in localStorage, verifies `Bearer` header | PASS |
| JWT token interceptor (skip when absent) | `axios.test.ts:32-39` — no token, verifies header undefined | PASS |
| auth API: login | `auth-api.test.ts:31-54` — POST `/auth/login`, returns `{token, expiresIn}` | PASS |
| auth API: register | `auth-api.test.ts:58-94` — POST `/auth/register`, returns `{id, username, displayName}` | PASS |
| tasks API: 7 CRUD methods | `tasks-api.test.ts` — getTasks, getTask, createTask, updateTask, deleteTask, updateStatus, getTransitions (7/7) | PASS |
| taskStore: state defaults | `taskStore.test.ts:41-54` — checks all 10 state fields | PASS |
| taskStore: fetchTasks (loading, data, error) | `taskStore.test.ts:57-155` — loading flag, pagination update, filter passing, error handling, empty list | PASS |
| taskStore: setKeyword (page reset) | `taskStore.test.ts:157-167` — sets keyword + resets page to 0 | PASS |
| taskStore: setPriorityFilter (page reset) | `taskStore.test.ts:169-179` — sets priority + resets page to 0 | PASS |
| taskStore: setStatusFilter (page reset) | `taskStore.test.ts:181-191` — sets status + resets page to 0 | PASS |
| taskStore: setPage | `taskStore.test.ts:193-201` — updates page number | PASS |
| taskStore: createTask (calls API + refreshes) | `taskStore.test.ts:203-224` — calls createTask then fetchTasks | PASS |
| taskStore: 6 actions total | fetchTasks, setKeyword, setPriorityFilter, setStatusFilter, setPage, createTask (6/6) | PASS |
| router: `/tasks` route | `router.test.ts:5-9` — path `/tasks`, name `task-list` | PASS |
| router: `/tasks/:id` route | `router.test.ts:11-16` — path `/tasks/:id`, name `task-detail` | PASS |

**Conclusion**: All acceptance criteria are covered by tests.

---

## STEP 2: Tests <-> Code

**Test execution**: 5 files, 30 tests, **all 30 passed** (vitest 1.6.1, 1.35s).

| Test File | Tests | Result |
|---|---|---|
| axios.test.ts | 3 | PASS |
| auth-api.test.ts | 4 | PASS |
| tasks-api.test.ts | 9 | PASS |
| taskStore.test.ts | 11 | PASS |
| router.test.ts | 3 | PASS |

**Conclusion**: Code passes all tests correctly.

---

## STEP 3: Requirements <-> Code

### axios.ts
- `baseURL: '/api'` — matches CONTRACT requirement. PASS.
- Request interceptor reads `localStorage.getItem('token')` and sets `Bearer` header. PASS.

### auth.ts
- `login(username, password)` → POST `/auth/login`. PASS.
- `register({username, password, displayName})` → POST `/auth/register`. PASS.
- Note: No `getMe()` function for `GET /api/auth/me`. This endpoint is in the CONTRACT but **not listed in the AC** for T7. The AC says "auth API (login/register)". Acceptable — not a gap for this task.

### tasks.ts — 7 CRUD methods
1. `getTasks(params?)` → GET `/tasks` with query params. PASS.
2. `getTask(id)` → GET `/tasks/{id}`. PASS.
3. `createTask(data)` → POST `/tasks`. PASS.
4. `updateTask(id, data)` → PUT `/tasks/{id}`. PASS.
5. `deleteTask(id)` → DELETE `/tasks/{id}`. PASS.
6. `updateStatus(id, status)` → PATCH `/tasks/{id}/status`. PASS.
7. `getTransitions(id)` → GET `/tasks/{id}/transitions`. PASS.

All 7 methods present and match CONTRACT endpoints.

### taskStore.ts — 6 actions
1. `fetchTasks()` — calls API with filters, updates state, handles loading/error. PASS.
2. `setKeyword(kw)` — sets keyword, resets page to 0. PASS.
3. `setPriorityFilter(p)` — sets priority filter, resets page to 0. PASS.
4. `setStatusFilter(s)` — sets status filter, resets page to 0. PASS.
5. `setPage(p)` — updates page. PASS.
6. `createTask(data)` — calls API then fetchTasks. PASS.

Page reset on filter change confirmed for setKeyword, setPriorityFilter, setStatusFilter. PASS.

### router/index.ts
- `/tasks` route with name `task-list`. PASS.
- `/tasks/:id` route with name `task-detail`. PASS.
- Existing `/` home route preserved. PASS.
- Uses `PlaceholderComponent` for task routes (appropriate since actual views are in later tasks). PASS.

**Conclusion**: All requirements fully implemented.

---

## STEP 4: Standards Compliance

SKIPPED (no standards files provided).

---

## STEP 5: Code Quality

| Check | Finding |
|---|---|
| Hardcoded values | `baseURL: '/api'` — appropriate for dev proxy setup. `size: 20` matches CONTRACT default. No issues. |
| Error handling | taskStore catches errors and stores message. axios interceptor is clean. No issues. |
| Unused imports | None found. All imports are used. |
| Type safety | `tasks.ts` uses `Record<string, any>` for task data — acceptable for infrastructure layer. Could be typed more strictly but not a blocker. |
| Code style | Consistent, clean, follows Vue/Pinia conventions. |

**Conclusion**: No code quality issues.

---

## STEP 6: File Scope

**ALLOWED files**: `src/api/axios.ts`, `src/api/auth.ts`, `src/api/tasks.ts`, `src/stores/taskStore.ts`, `src/router/index.ts`

**Files changed**:
- `src/api/axios.ts` (new) — ALLOWED
- `src/api/auth.ts` (new) — ALLOWED
- `src/api/tasks.ts` (new) — ALLOWED
- `src/stores/taskStore.ts` (new) — ALLOWED
- `src/router/index.ts` (modified) — ALLOWED

**Conclusion**: No out-of-scope files modified.

---

## Final Verdict: QA-PASS

All 30 tests pass. All acceptance criteria met: axios with JWT interceptor, auth API (login/register), tasks API (7 methods), taskStore (6 actions with page reset on filter change), router (/tasks and /tasks/:id routes). No scope violations, no code quality issues.

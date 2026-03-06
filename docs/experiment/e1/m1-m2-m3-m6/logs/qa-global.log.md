# Global QA Review Log

**Date:** 2026-03-06
**Scope:** M1 (Auth) + M2 (Task CRUD) + M3 (State Machine) + M6 (Frontend Task List Page)
**Verdict:** GLOBAL-FAIL

---

## CHECK 1: Cross-Task Consistency

### 1.1 Naming Conventions — PASS
- Java: camelCase fields, PascalCase classes, lowercase packages (`com.taskflow.*`) — all consistent
- Vue: PascalCase components (`TaskListItem.vue`, `TaskFilters.vue`), camelCase props/methods — all consistent
- TypeScript: PascalCase interfaces (`Task`, `PageResponse`), camelCase fields — all consistent
- Enum values: `HIGH`, `MEDIUM`, `LOW`, `TODO`, `IN_PROGRESS`, `DONE`, `CLOSED` — identical across backend `Task.Priority`, `TaskStatus` and frontend `Priority`, `TaskStatus` enums

### 1.2 Error Handling Patterns — FAIL (Inconsistency)
- **ISSUE [E1] SEVERITY=MEDIUM**: `AuthService` uses `ResponseStatusException` (Spring default) for 409 Conflict and 401 Unauthorized, while `TaskService` uses custom exceptions (`ResourceNotFoundException`, `InvalidTransitionException`) handled by `GlobalExceptionHandler`.
  - Auth errors produce Spring's default JSON format: `{timestamp, status, error, message, path}`
  - Task errors produce custom format: `{status, message}` (or `{status, message, errors}` for validation)
  - CONTRACT error format (line 30) specifies `{status:int, message:string, errors?:string[]}` — Auth errors do NOT conform.
  - `DuplicateException` class exists but is unused; `AuthService.register()` throws `ResponseStatusException(CONFLICT)` instead.

### 1.3 Patterns — PASS
- Soft-delete pattern applied consistently: `Task.deleted` field, `TaskRepository` uses `findByDeletedFalse*` methods, `TaskService.getById()` checks `isDeleted()`.
- JWT auth pattern consistent: `JwtAuthFilter` extracts userId from token and sets `Authentication.principal` to `Long userId`. Both `AuthController.me()` and `TaskController.createTask()` extract userId via `authentication.getPrincipal()`.
- `@PrePersist`/`@PreUpdate` lifecycle hooks used consistently on both `User` and `Task` entities.

---

## CHECK 2: Contract Compliance

### 2.1 Endpoint Coverage (10 endpoints defined in CONTRACT)

| # | Method | Path | Implemented | Status |
|---|--------|------|-------------|--------|
| 1 | POST | /api/auth/register | AuthController.register() | PASS |
| 2 | POST | /api/auth/login | AuthController.login() | PASS |
| 3 | GET | /api/auth/me | AuthController.me() | PASS |
| 4 | POST | /api/tasks | TaskController.createTask() | PASS |
| 5 | GET | /api/tasks/{id} | TaskController.getTask() | PASS |
| 6 | PUT | /api/tasks/{id} | TaskController.updateTask() | PASS |
| 7 | DELETE | /api/tasks/{id} | TaskController.deleteTask() | PASS |
| 8 | GET | /api/tasks | TaskController.listTasks() | **PARTIAL** |
| 9 | PATCH | /api/tasks/{id}/status | TaskController.transitionStatus() | PASS |
| 10 | GET | /api/tasks/{id}/transitions | TaskController.getTransitions() | PASS |

### 2.2 Backend-Frontend Alignment — FAIL

- **ISSUE [E2] SEVERITY=HIGH**: `GET /api/tasks` is missing the `status` query parameter in the backend.
  - CONTRACT v2 (line 12) specifies: `status:enum?` query parameter
  - CONTRACT amendment log (line 33) explicitly added this: "added status:enum? query param to GET /api/tasks | reason=M6 spec AC4 requires status filter"
  - Frontend `taskApi.ts` sends `status` param (lines 35-37)
  - Frontend `taskStore.ts` has `statusFilter` state and `setStatusFilter()` action (lines 14, 53-57)
  - Frontend `TaskFilters.vue` has a status dropdown (lines 15-21) that calls `setStatusFilter()`
  - **Backend `TaskController.listTasks()` does NOT accept a `status` parameter (lines 55-81)**
  - **Backend `TaskRepository` has NO `findBy*Status*` methods**
  - **Backend `TaskService.list()` has no `TaskStatus` parameter**
  - Result: status filter will be silently ignored by the backend. The frontend sends it but the backend discards it.

- **ISSUE [E3] SEVERITY=LOW**: `GET /api/tasks` does not accept a `sort` query parameter.
  - CONTRACT specifies `sort:string(default:createdAt,desc)` but backend hardcodes `Sort.by(DESC, "createdAt")`.
  - M2 spec also mentions `sort` param. Not exposed as configurable, but default behavior matches spec.
  - Frontend does not send `sort` param, so no immediate breakage.

---

## CHECK 3: Completeness (63 ACs)

### M1 Auth — 14 ACs

| AC | Description | Status | Notes |
|----|-------------|--------|-------|
| AC1 | Register returns 201 with id, username, displayName | PASS | |
| AC2 | Duplicate username returns 409 | PASS | Works but uses ResponseStatusException, not DuplicateException (see E1) |
| AC3 | Password stored as bcrypt hash | PASS | BCryptPasswordEncoder used |
| AC4 | Login returns 200 with token and expiresIn | PASS | expiresIn=86400 (seconds), consistent with 86400000ms config |
| AC5 | Wrong password returns 401 | PASS | |
| AC6 | Non-existent username returns 401 | PASS | Same error message as AC5 (BR7 compliance) |
| AC7 | Valid token GET /me returns 200 with user info | PASS | |
| AC8 | No auth header GET /me returns 401 | PASS | Controller-level check |
| AC9 | Expired token returns 401 | PASS | JwtTokenProvider.validateToken catches exception |
| AC10 | Malformed token returns 401 | PASS | |
| AC11 | Username <3 or >50 chars returns 400 | PASS | @Size(min=3,max=50) |
| AC12 | Password <8 chars returns 400 | PASS | @Size(min=8) |
| AC13 | Empty displayName or >100 chars returns 400 | PASS | @NotBlank @Size(max=100) |
| AC14 | Username with special chars returns 400 | PASS | @Pattern(regexp="^[a-zA-Z0-9_]+$") |

**M1 Result: 14/14 PASS**

### M2 Task CRUD — 16 ACs

| AC | Description | Status | Notes |
|----|-------------|--------|-------|
| AC1 | Create task returns 201 with full object | PASS | |
| AC2 | Default priority is MEDIUM | PASS | CreateTaskRequest defaults priority to MEDIUM |
| AC3 | Empty title returns 400 | PASS | @NotBlank @Size(min=1,max=200) |
| AC4 | Get task by ID returns 200 | PASS | |
| AC5 | Non-existent ID returns 404 | PASS | ResourceNotFoundException thrown |
| AC6 | Update returns 200 with updatedAt | PASS | @PreUpdate sets updatedAt |
| AC7 | Delete returns 204, soft-delete | PASS | |
| AC8 | Soft-deleted tasks excluded from list | PASS | findByDeletedFalse* methods |
| AC9 | Get soft-deleted task returns 404 | PASS | getById checks isDeleted() |
| AC10 | List defaults: page=0, size=20, createdAt desc | PASS | |
| AC11 | keyword filter by title (case-insensitive) | PASS | findByDeletedFalseAndTitleContainingIgnoreCase |
| AC12 | priority filter | PASS | findByDeletedFalseAndPriority |
| AC13 | Pagination page=1&size=5 | PASS | PageRequest.of(page, size) |
| AC14 | No token returns 401 | PASS | SecurityConfig requires auth for /api/tasks |
| AC15 | Title >200 chars returns 400 | PASS | @Size(max=200) |
| AC16 | Description >5000 chars returns 400 | PASS | @Size(max=5000) |

**M2 Result: 16/16 PASS**

### M3 State Machine — 16 ACs

| AC | Description | Status | Notes |
|----|-------------|--------|-------|
| AC1 | New task initial status is TODO | PASS | Task entity defaults status=TaskStatus.TODO |
| AC2 | TODO -> IN_PROGRESS allowed | PASS | |
| AC3 | IN_PROGRESS -> TODO allowed | PASS | |
| AC4 | IN_PROGRESS -> DONE allowed | PASS | |
| AC5 | DONE -> CLOSED allowed | PASS | |
| AC6 | TODO -> DONE rejected (400) | PASS | canTransitionTo returns false |
| AC7 | CLOSED -> any rejected (400) | PASS | CLOSED.getAllowedTransitions() returns empty list |
| AC8 | TODO -> CLOSED rejected (400) | PASS | |
| AC9 | DONE -> TODO rejected (400) | PASS | |
| AC10 | Successful transition updates updatedAt | PASS | TaskService.transitionStatus sets updatedAt |
| AC11 | TODO transitions: [IN_PROGRESS] | PASS | |
| AC12 | IN_PROGRESS transitions: [TODO, DONE] | PASS | |
| AC13 | DONE transitions: [CLOSED] | PASS | |
| AC14 | CLOSED transitions: [] | PASS | |
| AC15 | No token returns 401 | PASS | SecurityConfig |
| AC16 | Non-existent task ID returns 404 | PASS | getById throws ResourceNotFoundException |

**M3 Result: 16/16 PASS**

### M6 Frontend — 17 ACs

| AC | Description | Status | Notes |
|----|-------------|--------|-------|
| AC1 | /tasks route loads task list, page=20, desc order | PASS | router has /tasks route, onMounted calls fetchTasks |
| AC2 | Keyword search filters tasks | PASS | TaskFilters debounces 300ms, calls setKeyword |
| AC3 | Priority dropdown: All/HIGH/MEDIUM/LOW | PASS | |
| AC4 | Status dropdown: All/TODO/IN_PROGRESS/DONE/CLOSED | **FAIL** | Frontend sends status filter but backend ignores it (E2) |
| AC5 | Combined filters work together | **FAIL** | Status filter silently ignored by backend (E2) |
| AC6 | Task item shows: title, status badge, priority badge, assignee, tags | PASS | Assignee shows "未指派", tags placeholder present |
| AC7 | Status badge colors: TODO=#6B7280, IN_PROGRESS=#3B82F6, DONE=#10B981, CLOSED=#EF4444 | PASS | statusColorMap matches spec BR6 |
| AC8 | Priority badge colors: HIGH=#EF4444, MEDIUM=#F59E0B, LOW=#10B981 | PASS | priorityColorMap matches spec BR7 |
| AC9 | Click task navigates to /tasks/:id | PASS | router.push called (route undefined but M7 scope) |
| AC10 | "新增任務" button opens TaskCreateForm | PASS | showCreateForm toggled |
| AC11 | Create form: title(required), description, priority(default MEDIUM) | PASS | |
| AC12 | After create, form closes and list reloads | PASS | createTask calls fetchTasks, emits 'created' |
| AC13 | Pagination shows current page / total pages, prev/next buttons | PASS | |
| AC14 | Prev/next buttons navigate pages | PASS | |
| AC15 | First page disables prev, last page disables next | PASS | :disabled conditions correct |
| AC16 | Loading indicator during API request | PASS | data-testid="loading" shown when store.loading |
| AC17 | Error message on API failure | PASS | store.error displayed |

**M6 Result: 15/17 PASS, 2 FAIL (AC4, AC5 due to E2)**

---

## CHECK 4: Integration

### 4.1 Imports and Dependencies — PASS
- No circular dependencies detected.
- All imports resolve correctly within their module boundaries.
- Frontend `@/` alias configured in `vite.config.ts` to `src/`.

### 4.2 Frontend Types vs Backend Responses — PASS (with notes)
- `Task` interface matches `TaskResponse` fields exactly: id, title, description, priority, status, createdBy, createdAt, updatedAt.
- `PageResponse<T>` matches: content, page, size, totalElements, totalPages.
- `CreateTaskRequest` matches: title, description?, priority?.
- Frontend uses `string` for datetime fields (createdAt, updatedAt) which is correct since Jackson serializes LocalDateTime as string.

### 4.3 Additional Integration Notes
- **ISSUE [E4] SEVERITY=LOW**: `TaskListPage.vue` line 62 contains hacky code: `(router.currentRoute as any).value = resolved` — manually mutating router state is fragile and unnecessary. `router.push()` alone suffices.
- **ISSUE [E5] SEVERITY=LOW**: `TaskListPage.vue` lines 16-20 — loading indicator and empty-state div can show simultaneously during initial fetch because the conditions are not mutually exclusive. Should add `&& !store.loading` to line 20's v-if.
- **ISSUE [E6] SEVERITY=INFO**: `apiClient` baseURL is empty string `''` (taskApi.ts line 5), relying on Vite proxy to forward `/api` requests. This is correct for dev but won't work in production without a reverse proxy or baseURL config.

---

## CHECK 5: Standards Consistency — SKIPPED (none defined)

---

## CHECK 6: Integration Tests

### 6.1 Backend Tests (Maven)
```
Tests run: 297, Failures: 1, Errors: 0, Skipped: 0
BUILD FAILURE
```

**Failed test:**
- `SecurityConfigIntegrationTest$PublicEndpoints.shouldPermitLoginWithoutAuth`
  - Root cause: Test posts invalid credentials to `/api/auth/login` and asserts response status is NOT 401. But the controller returns 401 for invalid credentials. The test intends to verify the security filter permits the endpoint (not blocked at filter level), but the controller's own business logic also returns 401, causing a false failure.
  - This is a **test design flaw**, not a production code bug. The security filter does correctly permit `/api/auth/**`. The 401 comes from `AuthService.login()`, not from Spring Security.

### 6.2 Frontend Tests (Vitest)
```
Test Files  9 passed (9)
     Tests  145 passed (145)
```

All 145 frontend tests pass.

---

## Summary of Issues

| ID | Severity | Category | Description |
|----|----------|----------|-------------|
| E2 | **HIGH** | Integration Gap | `status` filter missing from backend `GET /api/tasks` — CONTRACT v2 and frontend send it, backend ignores it. M6 AC4 (status filter) and AC5 (combined filters) fail. |
| E1 | MEDIUM | Consistency | Auth errors use `ResponseStatusException` (Spring default format) while Task errors use custom exceptions (custom `{status,message}` format). `DuplicateException` exists but is unused. |
| E3 | LOW | Completeness | `sort` query parameter not exposed on `GET /api/tasks` (hardcoded to `createdAt,desc`). |
| E4 | LOW | Code Quality | Hacky router state mutation in `TaskListPage.vue` line 62. |
| E5 | LOW | UI Bug | Loading indicator and empty-state can display simultaneously. |
| E6 | INFO | Config | `apiClient.baseURL` is empty, requires proxy for API calls. |
| T1 | MEDIUM | Test Bug | `SecurityConfigIntegrationTest.shouldPermitLoginWithoutAuth` fails due to test design flaw (controller 401 vs security-filter 401 indistinguishable). |

---

## Verdict

**GLOBAL-FAIL**

**Blocking issue:** E2 — The `status` filter is specified in the CONTRACT (v2, explicitly amended), implemented in the frontend (TaskFilters.vue status dropdown, taskStore.statusFilter, taskApi sends status param), but completely missing from the backend (TaskController, TaskService, TaskRepository). This causes M6 AC4 and AC5 to fail and represents a cross-task integration gap. The CONTRACT amendment log explicitly states this was added because "M6 spec AC4 requires status filter, was missing from v1."

**Required fixes before GLOBAL-PASS:**
1. **E2 (HIGH)**: Add `status` query parameter to backend: add `TaskStatus status` parameter to `TaskController.listTasks()`, `TaskService.list()`, and the necessary `findByDeletedFalseAnd*Status*` methods to `TaskRepository` (8 combinations: keyword x priority x status + base).
2. **T1 (MEDIUM)**: Fix `SecurityConfigIntegrationTest.shouldPermitLoginWithoutAuth` — test should register a user first, then login with valid credentials, or use a different assertion strategy (e.g., verify no 403 instead of no 401).

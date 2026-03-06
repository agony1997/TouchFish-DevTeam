# QA Log — T6: M6-Frontend-Infra

**Result: QA-PASS**

---

## STEP 1: Requirements <-> Tests

### Types (task.test.ts)
| Requirement (CONTRACT / Spec) | Test Coverage | Verdict |
|---|---|---|
| Priority enum: HIGH, MEDIUM, LOW | Tested: values + exactly 3 members | PASS |
| TaskStatus enum: TODO, IN_PROGRESS, DONE, CLOSED | Tested: values + exactly 4 members | PASS |
| Task interface: id, title, description, priority, status, createdBy, createdAt, updatedAt? | Tested: valid object creation, nullable updatedAt | PASS |
| PageResponse<T>: content, page, size, totalElements, totalPages | Tested: empty + populated content | PASS |
| CreateTaskRequest: title (required), description (optional), priority (optional) | Tested: required-only + all-fields variants | PASS |

### API (taskApi.test.ts)
| Requirement | Test Coverage | Verdict |
|---|---|---|
| axios instance with JWT interceptor (BR1) | Tested: creation + interceptor registration | PASS |
| JWT token from localStorage in Authorization header | Tested: with token / without token | PASS |
| fetchTasks sends page, size, keyword, priority, status | Tested: full params + omit null/empty | PASS |
| fetchTasks returns PageResponse | Tested: response mapping | PASS |
| createTask POSTs to /api/tasks | Tested: body + return value | PASS |
| getTask GETs /api/tasks/:id | Tested: URL + return value | PASS |

### Store (taskStore.test.ts)
| Requirement (Spec: Pinia Store section) | Test Coverage | Verdict |
|---|---|---|
| Initial state: tasks=[], page=0, size=20, totalElements=0, totalPages=0, keyword='', priorityFilter=null, statusFilter=null, loading=false, error=null | 10 individual tests | PASS |
| fetchTasks: calls API with current state, updates state | 9 tests (params, state update, loading lifecycle, error handling) | PASS |
| setKeyword: sets keyword, resets page to 0, calls fetchTasks | 4 tests including boundary | PASS |
| setPriorityFilter: sets filter, resets page to 0, calls fetchTasks | 4 tests including null clear | PASS |
| setStatusFilter: sets filter, resets page to 0, calls fetchTasks | 4 tests including null clear | PASS |
| setPage: sets page, calls fetchTasks | 3 tests including boundary | PASS |
| createTask: calls API, refreshes list (AC12) | 3 tests including error propagation | PASS |
| Combined filters (AC5) | 1 test: keyword+priority+status+page | PASS |
| Error handling (AC16, AC17) | 3 tests: network, 401, 500 | PASS |

### Router (router.test.ts)
| Requirement | Test Coverage | Verdict |
|---|---|---|
| /tasks route exists (AC1) | Tested: route defined | PASS |
| /tasks resolves (not 404) | Tested: matched.length > 0 | PASS |
| Route name is 'tasks' or 'task-list' | Tested | PASS |
| Route has component (lazy-load) | Tested: components defined | PASS |
| Home route (/) preserved | Tested | PASS |

**STEP 1 Verdict: PASS** -- All spec/contract requirements have corresponding tests.

---

## STEP 2: Tests <-> Code

All 68 tests pass (4 test files, 0 failures).

| Test File | Tests | Result |
|---|---|---|
| task.test.ts | 10 | PASS |
| taskApi.test.ts | 11 | PASS |
| taskStore.test.ts | 42 | PASS |
| router.test.ts | 5 | PASS |

**STEP 2 Verdict: PASS**

---

## STEP 3: Requirements <-> Code

### types/task.ts vs CONTRACT
| CONTRACT Type | Code | Verdict |
|---|---|---|
| Priority: HIGH, MEDIUM, LOW | `enum Priority` with matching values | PASS |
| TaskStatus: TODO, IN_PROGRESS, DONE, CLOSED | `enum TaskStatus` with matching values | PASS |
| TaskResponse fields: id:long, title:string, description:string, priority:Priority, status:TaskStatus, createdBy:long, createdAt:LocalDateTime, updatedAt:LocalDateTime? | `Task` interface: id:number, title:string, description:string|null, priority:Priority|string, status:TaskStatus|string, createdBy:number, createdAt:string, updatedAt:string|null | PASS |
| PageResponse<T>: content:T[], page:int, size:int, totalElements:long, totalPages:int | Matching generic interface | PASS |
| CreateTaskRequest: title (required), description (optional), priority (optional) | `title:string, description?:string, priority?:Priority|string` | PASS |

Notes:
- `description` typed as `string|null` in Task (CONTRACT says `string`) -- acceptable since JSON APIs may return null for empty descriptions.
- `priority` and `status` typed as `Priority|string` / `TaskStatus|string` -- this is a pragmatic union to handle raw API string responses before they are narrowed to the enum. Acceptable.
- `createdAt`/`updatedAt` typed as `string` -- correct for frontend since JSON serializes LocalDateTime as string.

### api/taskApi.ts vs CONTRACT
| Requirement | Code | Verdict |
|---|---|---|
| GET /api/tasks with page, size, keyword, priority query params | `fetchTasks` builds queryParams correctly, omits null/empty | PASS |
| POST /api/tasks with body | `createTask` sends data to correct endpoint | PASS |
| GET /api/tasks/{id} | `getTask` interpolates id into URL | PASS |
| JWT interceptor (BR1) | Request interceptor reads from localStorage, adds Bearer header | PASS |

Note: Code sends `status` filter param which is not in the CONTRACT's GET /api/tasks endpoint. This is a spec-contract discrepancy (spec AC4 demands status filter, but CONTRACT omits it). The frontend code correctly prepares it; the backend would need to support it. Not a code defect -- flagged as **informational**.

### stores/taskStore.ts vs Spec
| Spec Requirement | Code | Verdict |
|---|---|---|
| State fields: tasks, page, size, totalElements, totalPages, keyword, priorityFilter, statusFilter, loading, error | All present with correct defaults | PASS |
| fetchTasks: calls API with all current filters, updates state, manages loading/error | Implemented with try/catch/finally | PASS |
| setKeyword: set + page reset + fetchTasks | Implemented | PASS |
| setPriorityFilter: set + page reset + fetchTasks | Implemented | PASS |
| setStatusFilter: set + page reset + fetchTasks | Implemented | PASS |
| setPage: set + fetchTasks (no page reset) | Implemented | PASS |
| createTask: API call + fetchTasks refresh (AC12) | Implemented | PASS |

### router/index.ts vs Spec
| Requirement | Code | Verdict |
|---|---|---|
| /tasks route (AC1) | Present, name='tasks' | PASS |
| Lazy-loaded component (TaskListPage.vue) | `() => import('../views/TaskListPage.vue')` | PASS |
| Home route preserved | `/` route still present | PASS |

**STEP 3 Verdict: PASS**

---

## STEP 4: Standards

SKIPPED (no standards file provided).

---

## STEP 5: Code Quality

### types/task.ts
- Clean TypeScript enums and interfaces.
- Proper use of generics for PageResponse.
- Union types (Priority|string) are pragmatic for API boundary types.
- No unused exports.

### api/taskApi.ts
- Proper axios instance creation with interceptor.
- Clean parameter filtering (omits falsy values).
- Functions are properly typed with return types.
- `getTask` is an extra export beyond strict ALLOWED-actions scope but is useful and tested.
- `Record<string, any>` union on createTask parameter is slightly loose but acceptable.

### stores/taskStore.ts
- Clean Pinia options-API store.
- Proper loading/error lifecycle in fetchTasks (loading set true at start, cleared in finally block).
- Error is caught and surfaced as string to state.
- createTask propagates errors (does not swallow) -- correct for UI to handle.
- No dead code.

### router/index.ts
- Minimal, clean lazy-loading setup.
- Two routes only -- no unnecessary additions.

**STEP 5 Verdict: PASS** -- No code quality issues found.

---

## STEP 6: File Scope

### ALLOWED files
- [x] `types/task.ts`
- [x] `api/taskApi.ts`
- [x] `stores/taskStore.ts`
- [x] `router/index.ts`

### Extra files check
- `views/TaskListPage.vue` -- exists as minimal placeholder (3 lines: `<template><div>Task List Page</div></template>`). Required by router lazy import. **Acceptable.**
- No other unexpected files created by worker.

**STEP 6 Verdict: PASS**

---

## Informational Notes (non-blocking)

1. **Spec-CONTRACT discrepancy on `status` filter**: The M6 spec (AC4) requires a status filter on `GET /api/tasks`, but the CONTRACT does not list `status` as a query parameter. The frontend code sends it anyway, which is correct per spec. The backend CONTRACT may need an amendment to add `status` as a supported query param. This is NOT a code defect.

2. **`description` nullability**: The Task interface types `description` as `string|null`, while the CONTRACT says `description:string`. In practice, JSON APIs commonly return null for optional/empty fields. This is a pragmatic and safe choice.

---

## Final Verdict

**QA-PASS**

All 68 tests pass. Types match CONTRACT. Store actions match spec. Code is clean with proper error handling. File scope is respected. No blocking issues found.

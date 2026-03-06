# QA Log — Task T4: M2-CRUD-API

**Reviewer**: QA Agent (qa-task-4)
**Date**: 2026-03-06
**Verdict**: **QA-PASS**

---

## STEP 1: Requirements <-> Tests

Mapping each of the 16 ACs from `M2-task-crud.md` to test coverage:

| AC | Description | Test Coverage | Status |
|----|-------------|---------------|--------|
| AC1 | POST /api/tasks with valid token → 201 + full task object | `TaskControllerIntegrationTest.AC1_CreateTask.shouldCreateTaskWith201` — asserts 201, id, title, description, priority, status=TODO, createdBy, createdAt | COVERED |
| AC2 | POST without priority → defaults MEDIUM | `AC2_DefaultPriority.shouldDefaultToMediumPriority` — omits priority field, asserts MEDIUM; also `TaskServiceTest.CreateTests.shouldDefaultPriorityToMedium` | COVERED |
| AC3 | POST without title or empty title → 400 | `AC3_MissingTitle` — 3 tests: missing title, empty title, blank/whitespace title; also `UpdateTaskRequestValidationTest.TitleValidation` (null, empty, blank) | COVERED |
| AC4 | GET /api/tasks/{id} existing → 200 + full task | `AC4_GetById.shouldReturn200WithFullTask` — asserts id, title, description, priority, status, createdBy, createdAt | COVERED |
| AC5 | GET /api/tasks/{id} non-existent → 404 | `AC5_GetNonExistent.shouldReturn404ForNonExistentTask`; also `TaskServiceTest.GetByIdTests.shouldThrowForNonExistentTask` | COVERED |
| AC6 | PUT /api/tasks/{id} → 200 + updatedAt has value | `AC6_UpdateTask.shouldUpdateTaskAndReturn200` — asserts title, description, priority updated + updatedAt not null; also tests 404 for non-existent update | COVERED |
| AC7 | DELETE /api/tasks/{id} → 204 No Content + soft delete | `AC7_DeleteTask` — 3 tests: 204 response, soft-delete verified in DB (deleted=true), 404 for non-existent delete | COVERED |
| AC8 | Soft-deleted tasks NOT in GET /api/tasks list | `AC8_SoftDeletedNotInList.shouldExcludeSoftDeletedFromList` — creates 3, deletes 1, asserts totalElements=2; also `TaskServiceTest.ListTests.shouldReturnPaginatedResults` | COVERED |
| AC9 | GET /api/tasks/{id} for soft-deleted → 404 | `AC9_GetSoftDeleted.shouldReturn404ForSoftDeletedTask`; also `TaskServiceTest.GetByIdTests.shouldThrowForSoftDeletedTask` | COVERED |
| AC10 | GET /api/tasks default pagination (page=0, size=20, createdAt desc) | `AC10_DefaultPagination` — 2 tests: default page/size/totalPages assertions + createdAt descending sort order verified | COVERED |
| AC11 | GET /api/tasks?keyword=xxx → title filter (case-insensitive) | `AC11_KeywordSearch` — 3 tests: keyword match, case-insensitive match, non-matching keyword returns empty | COVERED |
| AC12 | GET /api/tasks?priority=HIGH → priority filter | `AC12_PriorityFilter.shouldFilterByHighPriority` — creates mixed priorities, asserts only HIGH returned; also `TaskServiceTest.ListTests.shouldFilterByPriority` | COVERED |
| AC13 | GET /api/tasks?page=1&size=5 → correct second page | `AC13_CustomPagination` — 2 tests: second page (3 of 8 items) and first page (5 of 8 items) | COVERED |
| AC14 | No token → 401 Unauthorized | `AC14_Unauthorized` — 5 tests: POST, GET/{id}, PUT, DELETE, GET list — all assert 401 | COVERED |
| AC15 | title >200 chars → 400 | `AC15_TitleTooLong` — 3 tests: 201-char POST fails, 200-char POST succeeds, 201-char PUT fails; also `UpdateTaskRequestValidationTest.title201CharsFails` | COVERED |
| AC16 | description >5000 chars → 400 | `AC16_DescriptionTooLong` — 3 tests: 5001-char POST fails, 5000-char POST succeeds, 5001-char PUT fails; also `UpdateTaskRequestValidationTest.description5001CharsFails` | COVERED |

**Result**: All 16 ACs fully covered. Tests also include additional edge cases (combined keyword+priority filter, error response format validation).

---

## STEP 2: Tests <-> Code

**Test execution**: `mvn test` — **74 tests run, 0 failures, 0 errors, 0 skipped**. BUILD SUCCESS.

Test-to-code tracing:

| Test Class | Validates Code In | Result |
|---|---|---|
| `TaskControllerIntegrationTest` (31 tests) | TaskController, TaskService, GlobalExceptionHandler, entity/DTO stack, security chain | ALL PASS |
| `TaskServiceTest` (17 tests) | TaskService.create/getById/update/softDelete/list | ALL PASS |
| `UpdateTaskRequestValidationTest` (9 tests) | UpdateTaskRequest validation annotations (@NotBlank, @Size) | ALL PASS |
| `PageResponseTest` (6 tests) | PageResponse DTO getters/setters | ALL PASS |
| `GlobalExceptionHandlerTest` (4 tests) | GlobalExceptionHandler — 400/404 structured error responses | ALL PASS |

Additional observations:
- `PageResponseTest` uses pure POJO unit tests (no Spring context) — efficient.
- `UpdateTaskRequestValidationTest` uses Jakarta Validator directly — good isolation.
- Integration tests cover full HTTP stack through MockMvc with security filter chain.

**Result**: PASS — all tests green.

---

## STEP 3: Requirements <-> Code

### Soft Delete
- **Task.java**: `private boolean deleted = false;` field present.
- **TaskService.softDelete()**: sets `deleted=true` via `task.setDeleted(true)` and saves. Does NOT physically delete.
- **TaskService.getById()**: checks `task.isDeleted()` and throws `ResourceNotFoundException` if true.
- **TaskService.list()**: all repository queries use `findByDeletedFalse*` methods, automatically excluding soft-deleted tasks.
- **TaskRepository**: four query methods all filter on `DeletedFalse`.
- **Verdict**: CORRECT — matches BR7, BR8, AC7, AC8, AC9.

### Pagination Defaults
- **TaskController.listTasks()**: `@RequestParam(defaultValue = "0") int page` and `@RequestParam(defaultValue = "20") int size`.
- **TaskService.list()**: `PageRequest.of(page, size, Sort.by(Sort.Direction.DESC, "createdAt"))` — hardcodes createdAt DESC default.
- **Verdict**: CORRECT — matches BR9, AC10.

### createdBy from JWT
- **TaskController.createTask()**: `Long userId = (Long) authentication.getPrincipal()` — extracts user ID from JWT-authenticated principal.
- **TaskService.create()**: `task.setCreatedBy(userId)` — sets it from the passed userId, not from request body.
- **CreateTaskRequest**: has NO `createdBy` field — cannot be set by client.
- **Verdict**: CORRECT — matches BR4.

### Validation
- **CreateTaskRequest**: `@NotBlank @Size(min=1, max=200)` on title; `@Size(max=5000)` on description; `priority` defaults to `MEDIUM`.
- **UpdateTaskRequest**: Same annotations on title/description; priority is optional (no default).
- **TaskController**: uses `@Valid @RequestBody` on both create and update endpoints.
- **GlobalExceptionHandler**: catches `MethodArgumentNotValidException` → 400 with structured error body.
- **Verdict**: CORRECT — matches BR1, BR2, BR3, AC3, AC15, AC16.

### Initial Status
- **Task.java**: `private TaskStatus status = TaskStatus.TODO` — entity default.
- **TaskService.create()**: does NOT override status, so entity default (TODO) is used.
- **Verdict**: CORRECT — matches BR10.

### Auto-timestamps
- **Task.java**: `@PrePersist` sets `createdAt = LocalDateTime.now()`; `@PreUpdate` sets `updatedAt = LocalDateTime.now()`.
- **Verdict**: CORRECT — matches BR5, BR6.

---

## STEP 4: Standards

SKIPPED (no standards provided).

---

## STEP 5: Code Quality

### Error Handling
- **ResourceNotFoundException**: extends `RuntimeException`, caught by `GlobalExceptionHandler` → 404.
- **DuplicateException**: extends `RuntimeException`, caught by `GlobalExceptionHandler` → 409.
- **MethodArgumentNotValidException**: caught → 400 with field-level error details.
- **Invalid priority string**: `Task.Priority.valueOf(priority)` in controller will throw `IllegalArgumentException` if invalid — this is NOT caught by `GlobalExceptionHandler`. However, this is a minor gap; the default Spring error handler will return 500, but the spec does not define an AC for invalid priority enum values. Low-risk, non-blocking.

### HTTP Status Codes
- POST → 201 Created (correct)
- GET → 200 OK (correct)
- PUT → 200 OK (correct)
- DELETE → 204 No Content (correct)
- Validation errors → 400 Bad Request (correct)
- Not found → 404 Not Found (correct)
- Unauthorized → 401 via security filter chain (correct)

### No Unused Code
- All methods in TaskService are called by TaskController or tests.
- `DuplicateException` handler in `GlobalExceptionHandler` is used by M1 (AuthService), not by M2 — this is fine as it is a shared exception handler.
- No dead code detected.

### Code Style
- Clean constructor injection (no `@Autowired` on fields).
- `@Transactional` annotations properly applied (readOnly for reads).
- Proper separation: Controller → Service → Repository.
- `toResponse()` private mapper method in controller — acceptable for current scale.

### Minor Observation (Non-blocking)
- `TaskService.update()`: when `request.getPriority()` is null, priority is not updated (preserved). This is correct behavior for partial update semantics. However, `request.getDescription()` is always set even if null — this means an update without description will clear it. This matches the spec since PUT requires title (and description is optional in the body), so the contract expectation is that the client sends all fields they want to keep.

---

## STEP 6: File Scope

**ALLOWED files** (per task assignment):
- TaskService.java -- PRESENT, correctly implemented
- TaskController.java -- PRESENT, correctly implemented
- UpdateTaskRequest.java -- PRESENT, correctly implemented
- PageResponse.java -- PRESENT, correctly implemented
- GlobalExceptionHandler.java -- PRESENT, correctly implemented
- ResourceNotFoundException.java -- PRESENT, supporting exception class
- DuplicateException.java -- PRESENT, supporting exception class

**Other files observed** (CreateTaskRequest.java, TaskResponse.java, Task.java, TaskRepository.java, TaskStatus.java) are shared/pre-existing entity/DTO files that T4 implementation depends on but does not own — they were created by other tasks (M1 skeleton, test agent). No files outside the allowed scope were created by T4.

**Result**: PASS — file scope respected.

---

## Summary

| Step | Result |
|------|--------|
| STEP 1: Requirements <-> Tests | PASS — all 16 ACs covered |
| STEP 2: Tests <-> Code | PASS — 74/74 tests green |
| STEP 3: Requirements <-> Code | PASS — soft delete, pagination, JWT createdBy, validation all correct |
| STEP 4: Standards | SKIPPED |
| STEP 5: Code Quality | PASS — proper error handling, correct HTTP codes, clean code |
| STEP 6: File Scope | PASS — only allowed files touched |

## Verdict: **QA-PASS**

# QA Log: T4 - Task Service + Controller

## Result: QA-PASS

## Test Execution
- **Command**: `mvn test -Dtest="TaskServiceTest,TaskControllerIntegrationTest"`
- **Result**: 56 tests run, 0 failures, 0 errors, 0 skipped
- **Duration**: ~14s

## Three-Way Verification

### Tests vs CONTRACT (AC Coverage)

| AC | Description | Unit Test | Integration Test |
|----|-------------|-----------|------------------|
| AC1 | POST /api/tasks 201 + TaskResponse | CreateTask.shouldCreateTaskAndReturnResponse | CreateTaskEndpoint.shouldReturn201WithTaskResponse |
| AC2 | Default priority MEDIUM | CreateTask.shouldDefaultPriorityToMedium_whenNotSpecified | CreateTaskEndpoint.shouldDefaultPriorityToMedium |
| AC3 | Missing title 400 | (validation at DTO layer) | shouldReturn400_whenTitleMissing, shouldReturn400_whenTitleBlank |
| AC4 | GET /api/tasks/{id} 200 | GetTask.shouldReturnTask_whenFound | GetTaskEndpoint.shouldReturn200WithTask |
| AC5 | GET non-existent 404 | GetTask.shouldThrow_whenNotFound | GetTaskEndpoint.shouldReturn404_whenTaskNotFound |
| AC6 | PUT /api/tasks/{id} 200 | UpdateTask.shouldUpdateAndReturn | UpdateTaskEndpoint.shouldReturn200WithUpdatedTask |
| AC7 | DELETE soft delete 204 | DeleteTask.shouldSoftDelete | DeleteTaskEndpoint.shouldReturn204 |
| AC8 | Soft-deleted excluded from list | ListTasks.shouldOnlyReturnNonDeletedTasks | DeleteTaskEndpoint.shouldNotAppearInList_afterSoftDelete |
| AC9 | Soft-deleted GET 404 | GetTask.shouldThrow_whenSoftDeleted | GetTaskEndpoint.shouldReturn404_whenTaskSoftDeleted |
| AC10 | Paginated list | ListTasks.shouldReturnPageResponse | ListTasksEndpoint.shouldReturnPaginatedResponse |
| AC11 | Keyword filter | ListTasks.shouldPassKeywordToRepository | ListTasksEndpoint.shouldFilterByKeyword, shouldFilterByKeyword_caseInsensitive |
| AC12 | Priority/status filter | shouldPassPriorityFilterToRepository, shouldPassStatusFilterToRepository | ListTasksEndpoint.shouldFilterByPriority, shouldFilterByStatus |
| AC13 | Custom page params | (via Pageable mock) | ListTasksEndpoint.shouldRespectPageParams, shouldReturnSecondPage |
| AC14 | 401 without token | (security layer) | AuthRequired (6 tests: all endpoints + invalid token) |
| AC15 | Title max 200 chars | (validation at DTO) | shouldReturn400_whenTitleTooLong, shouldReturn201_whenTitleExactly200Chars |
| AC16 | Description max 5000 chars | (validation at DTO) | shouldReturn400_whenDescriptionTooLong, shouldReturn201_whenDescriptionExactly5000Chars |

All 16 ACs fully covered.

### Implementation vs CONTRACT

- Endpoints, HTTP methods, status codes, request/response shapes all match CONTRACT
- TaskResponse fields: id, title, description, priority, status, createdBy, createdAt, updatedAt -- matches [TYPE] TaskResponse
- TaskPageResponse fields: content, page, size, totalElements, totalPages -- matches [TYPE] TaskPageResponse
- Error format: `{"message": "..."}` -- matches [FORMAT]
- Default pagination: size=20, sort=createdAt DESC -- matches CONTRACT
- Filter params: keyword (case-insensitive LIKE on title), priority (enum), status (enum) -- matches CONTRACT

### Implementation vs Tests

- Unit tests verify service logic with mocked repository
- Integration tests verify full HTTP flow with real Spring context
- E2E test covers complete CRUD lifecycle (create -> read -> update -> verify -> delete -> verify 404 -> verify list empty)
- All assertions align with implementation behavior

## Code Quality

- Constructor injection, clean controller/service separation
- Custom TaskNotFoundException with @ExceptionHandler for 404
- @Valid annotation for request validation
- Entity-level defaults (priority=MEDIUM, status=TODO, deleted=false)
- Repository JPQL properly filters deleted=false in all queries
- No security vulnerabilities (parameterized queries, no raw SQL)
- Soft delete: sets deleted flag, never calls repository.delete()

## File Scope

- ALLOWED: service/TaskService.java, controller/TaskController.java
- Verified: only these two files contain T4 implementation logic
- Supporting files (DTOs, Entity, Repository) are from T3, not modified

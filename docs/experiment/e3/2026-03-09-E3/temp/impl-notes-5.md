# Implementation Notes — T5: M2 Task CRUD API

## Test Design

### Dependencies
- Tests use `@SpringBootTest` + `@AutoConfigureMockMvc` (full integration tests)
- Auth: register user via direct repo save + login via `/api/auth/login` to get JWT token
- H2 in-memory database, `@Transactional` for test isolation

### Files Required by Implementation (not yet created)
1. `TaskRequest` DTO — with `@Valid` annotations: title (1-200, required), description (max 5000, optional), priority (optional, default MEDIUM)
2. `TaskService` — CRUD logic using TaskRepository, soft-delete, pagination/filtering
3. `TaskController` — REST endpoints at `/api/tasks`
4. `PageResponse` DTO — wrapper: content, page, size, totalElements, totalPages

### AC Coverage Map
| AC | Test Method | Description |
|----|-------------|-------------|
| AC1 | createTask_validData_returns201 | POST valid task -> 201 with id, createdBy, createdAt |
| AC2 | createTask_noPriority_defaultsMedium | No priority -> MEDIUM |
| AC3 | createTask_emptyTitle_returns400, createTask_noTitle_returns400 | Missing/empty title -> 400 |
| AC4 | getTask_existing_returns200 | GET by id -> 200 |
| AC5 | getTask_nonExisting_returns404 | GET non-existing id -> 404 |
| AC6 | updateTask_valid_returns200WithUpdatedAt | PUT -> 200 with updatedAt |
| AC7 | deleteTask_existing_returns204 | DELETE -> 204 soft delete |
| AC8 | listTasks_excludesSoftDeleted | GET list excludes deleted |
| AC9 | getTask_softDeleted_returns404 | GET soft-deleted -> 404 |
| AC10 | listTasks_defaultPagination | Default page=0, size=20, sort=createdAt desc |
| AC11 | listTasks_keywordFilter | keyword search case-insensitive |
| AC12 | listTasks_priorityFilter | priority=HIGH filter |
| AC13 | listTasks_pagination | page=1&size=5 correct second page |
| AC14 | createTask_noToken_returns401 | No auth -> 401 |
| AC15 | createTask_titleTooLong_returns400 | title >200 -> 400 |
| AC16 | createTask_descriptionTooLong_returns400 | description >5000 -> 400 |

### Key Design Decisions
- Helper method `registerAndGetToken()` follows AuthControllerTest pattern
- Helper method `createTaskJson()` builds JSON map for flexibility
- `@BeforeEach` clears both task and user repos for isolation
- Nested test classes group by feature: Create, Read, Update, Delete, List, Auth

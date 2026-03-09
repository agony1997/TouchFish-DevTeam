# Test Agent 5 Output — T5: M2 Task CRUD API

## Test File
- `C:/Users/a0304/IdeaProjects/TouchFish-DevTeam/taskflow/backend/src/test/java/com/taskflow/controller/TaskControllerTest.java`

## Test Count: 28 tests

## AC Coverage
| AC | Tests | Status |
|----|-------|--------|
| AC1 | createTask_validData_returns201, createTask_createdByMatchesAuthUser | RED (404 — no controller) |
| AC2 | createTask_noPriority_defaultsMedium | RED |
| AC3 | createTask_emptyTitle_returns400, createTask_noTitle_returns400 | RED |
| AC4 | getTask_existing_returns200 | RED |
| AC5 | getTask_nonExisting_returns404 | RED (404 but no JSON body) |
| AC6 | updateTask_valid_returns200WithUpdatedAt, updateTask_nonExisting_returns404 | RED |
| AC7 | deleteTask_existing_returns204, deleteTask_nonExisting_returns404 | RED |
| AC8 | listTasks_excludesSoftDeleted | RED |
| AC9 | getTask_softDeleted_returns404 | RED |
| AC10 | listTasks_defaultPagination | RED |
| AC11 | listTasks_keywordFilter | RED |
| AC12 | listTasks_priorityFilter | RED |
| AC13 | listTasks_pagination | RED |
| AC14 | createTask_noToken_returns401, getTask_noToken_returns401, updateTask_noToken_returns401, deleteTask_noToken_returns401, listTasks_noToken_returns401 | GREEN (security blocks) |
| AC15 | createTask_titleTooLong_returns400, createTask_titleExact200_returns201, updateTask_titleTooLong_returns400 | RED |
| AC16 | createTask_descriptionTooLong_returns400, createTask_descriptionExact5000_returns201 | RED |
| Extra | listTasks_empty_returnsEmptyPage | RED |

## Run Result
- **28 tests total**, 21 FAIL (expected RED), 7 PASS (auth 401 tests + edge)
- All tests compile cleanly
- All failures are due to missing TaskController (returns 404)

## Implementation Requirements (for worker)
The tests expect these files to be created:
1. **TaskRequest** DTO — `@NotBlank @Size(max=200) title`, `@Size(max=5000) description`, `priority` (optional, default MEDIUM)
2. **PageResponse<T>** DTO — fields: `content` (List<T>), `page` (int), `size` (int), `totalElements` (long), `totalPages` (int)
3. **TaskService** — CRUD with soft-delete, pagination, keyword/priority filtering
4. **TaskController** — REST endpoints:
   - `POST /api/tasks` -> 201 (uses `@Valid @RequestBody TaskRequest`, sets createdBy from JWT)
   - `GET /api/tasks/{id}` -> 200 (excludes soft-deleted)
   - `PUT /api/tasks/{id}` -> 200 (uses `@Valid @RequestBody TaskRequest`)
   - `DELETE /api/tasks/{id}` -> 204 (soft delete)
   - `GET /api/tasks` -> 200 with PageResponse (params: keyword, priority, page, size)

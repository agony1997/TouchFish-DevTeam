# Worker-4 Log: T4 - Task Service + Controller

## Status: COMPLETE

## Files Created
- `backend/src/main/java/com/taskflow/service/TaskService.java` (NEW)
- `backend/src/main/java/com/taskflow/controller/TaskController.java` (NEW)

## Implementation Summary

### TaskService
- `createTask(Long userId, TaskRequest)` - creates task, sets createdBy from userId, respects explicit priority or defaults to MEDIUM via entity default
- `getTask(Long id)` - finds by `findByIdAndDeletedFalse`, throws TaskNotFoundException (404) if not found
- `updateTask(Long id, TaskRequest)` - updates title/description/priority, sets updatedAt, does NOT change status or createdBy
- `deleteTask(Long id)` - soft delete (sets deleted=true), throws 404 if not found
- `listTasks(keyword, priority, status, pageable)` - delegates to `findByFilters`, maps Page<Task> to TaskPageResponse
- `toResponse(Task)` - private helper mapping all entity fields to TaskResponse
- `TaskNotFoundException` - static inner exception class extending RuntimeException

### TaskController
- `POST /api/tasks` -> 201, extracts userId from SecurityContext principal (Long cast)
- `GET /api/tasks/{id}` -> 200
- `PUT /api/tasks/{id}` -> 200, @Valid @RequestBody
- `DELETE /api/tasks/{id}` -> 204 no content
- `GET /api/tasks` -> 200, @PageableDefault(size=20, sort="createdAt", direction=DESC), optional keyword/priority/status params
- `@ExceptionHandler` for TaskNotFoundException -> 404 `{"message": "..."}`
- `@ExceptionHandler` for MethodArgumentNotValidException -> 400 `{"message": "..."}`

## Test Results
- **56 tests run, 0 failures, 0 errors, 0 skipped**
- TaskServiceTest: 24 passed (CreateTask: 6, GetTask: 4, UpdateTask: 4, DeleteTask: 3, ListTasks: 7)
- TaskControllerIntegrationTest: 32 passed (CreateTask: 8, GetTask: 3, UpdateTask: 3, DeleteTask: 3, ListTasks: 9, AuthRequired: 6)

## Scope Verification
- `git diff --name-only` checked; no out-of-scope files modified by this worker
- Only 2 new files created, both in ALLOWED list

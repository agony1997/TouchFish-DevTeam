# Worker-5 Log: T5 — M2 Task CRUD API

## Task
Build TaskService, TaskController, TaskRequest DTO, and PageResponse DTO for Task CRUD operations.

## Files Created
1. `taskflow/backend/src/main/java/com/taskflow/dto/TaskRequest.java` — DTO with @NotBlank @Size(max=200) title, @Size(max=5000) description, optional priority
2. `taskflow/backend/src/main/java/com/taskflow/dto/PageResponse.java` — Generic page wrapper with content, page, size, totalElements, totalPages
3. `taskflow/backend/src/main/java/com/taskflow/service/TaskService.java` — CRUD + list with pagination/filtering, soft-delete, default priority=MEDIUM
4. `taskflow/backend/src/main/java/com/taskflow/controller/TaskController.java` — REST endpoints POST/GET/PUT/DELETE /api/tasks, userId from SecurityContext

## Test Results
- **28/28 tests PASS** (TaskControllerTest)
- BUILD SUCCESS in ~15s

## Scope Check
- All 4 files are within ALLOWED scope
- No modifications to READONLY or FORBIDDEN files

## Key Implementation Details
- TaskRequest uses Jakarta validation: @NotBlank + @Size(max=200) for title, @Size(max=5000) for description
- Default priority MEDIUM applied in TaskService.createTask when priority is null
- Soft-delete: deleteTask sets deleted=true; getTask/listTasks filter out deleted tasks
- userId extracted from SecurityContext via `(Long) authentication.getPrincipal()`
- Pagination defaults: page=0, size=20, sort=createdAt DESC
- Filtering: keyword -> findByDeletedFalseAndTitleContainingIgnoreCase; priority -> findByDeletedFalseAndPriority

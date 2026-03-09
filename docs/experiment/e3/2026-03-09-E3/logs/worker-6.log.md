# Worker-6 Log — T6: M3 Task State Machine

## Task
T6: M3 — Task State Machine (TaskStateMachineService, TaskStatusController, StatusUpdateRequest, TransitionResponse)

## Implementation Summary

### Files Created (4)
1. `taskflow/backend/src/main/java/com/taskflow/dto/StatusUpdateRequest.java` — DTO with `@NotNull status` field
2. `taskflow/backend/src/main/java/com/taskflow/dto/TransitionResponse.java` — DTO with `currentStatus` + `availableTransitions` list
3. `taskflow/backend/src/main/java/com/taskflow/service/TaskStateMachineService.java` — State machine logic using EnumMap for transition rules
4. `taskflow/backend/src/main/java/com/taskflow/controller/TaskStatusController.java` — REST controller with PATCH /status and GET /transitions endpoints

### Design Decisions
- Used `EnumMap<TaskStatus, List<TaskStatus>>` for O(1) transition lookup
- Added `@ExceptionHandler` for `IllegalStateException` and `IllegalArgumentException` in controller (since GlobalExceptionHandler is READONLY and doesn't handle these)
- Task entity's `@PreUpdate` callback automatically sets `updatedAt` on save — no manual timestamp handling needed
- Reused `ResourceNotFoundException` (already handled by GlobalExceptionHandler) for 404 cases
- Soft-delete check: `findById().filter(t -> !t.isDeleted())` consistent with TaskService

### Transition Rules
- TODO -> [IN_PROGRESS]
- IN_PROGRESS -> [TODO, DONE]
- DONE -> [CLOSED]
- CLOSED -> [] (terminal)

## Test Results
- **27/27 tests PASS**
- All AC1-AC16 covered
- No out-of-scope file changes

## Timeline
- Read all required/readonly files
- Created 4 files
- First run: 27/27 PASS

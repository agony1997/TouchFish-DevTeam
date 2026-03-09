# Implementation Notes — T6: M3 Task State Machine

## Required Files (for worker)

### 1. `StatusUpdateRequest.java` (DTO)
- Package: `com.taskflow.dto`
- Single field: `status` (String) — the target status
- Needs getter/setter

### 2. `TransitionResponse.java` (DTO)
- Package: `com.taskflow.dto`
- Fields: `currentStatus` (String), `availableTransitions` (List<String>)
- Needs getter/setter, constructor

### 3. `TaskStateMachineService.java` (Service)
- Package: `com.taskflow.service`
- Transition rules (Map<TaskStatus, List<TaskStatus>>):
  - TODO -> [IN_PROGRESS]
  - IN_PROGRESS -> [TODO, DONE]
  - DONE -> [CLOSED]
  - CLOSED -> [] (terminal)
- Methods:
  - `TaskResponse updateStatus(Long taskId, String targetStatus)` — validate transition, update task, save, return TaskResponse
  - `TransitionResponse getAvailableTransitions(Long taskId)` — look up current status, return allowed next statuses
- Throws `ResourceNotFoundException` if task not found (or task.isDeleted())
- Throws `IllegalStateTransitionException` (or similar, caught by GlobalExceptionHandler) if transition is illegal -> 400

### 4. `TaskStatusController.java` (Controller)
- Package: `com.taskflow.controller`
- Endpoints:
  - `PATCH /api/tasks/{id}/status` — accepts `StatusUpdateRequest`, returns `TaskResponse`
  - `GET /api/tasks/{id}/transitions` — returns `TransitionResponse`
- Both require authentication (already handled by SecurityConfig — anyRequest().authenticated())

## State Transition Diagram
```
TODO ---> IN_PROGRESS ---> DONE ---> CLOSED (terminal)
      <---
```

## Key Design Decisions
- Use an EnumMap<TaskStatus, List<TaskStatus>> for transition rules — clean and O(1) lookup
- IllegalStateException or custom exception for illegal transitions
- GlobalExceptionHandler needs a handler for the illegal transition exception -> 400
- Task entity already has @PreUpdate that sets updatedAt, so just saving the task after status change handles AC10
- Must check task.isDeleted() == false when finding task; if deleted, treat as 404

## Error Response Format
All errors follow: `{message: string, timestamp: ISO8601}` via GlobalExceptionHandler + ErrorResponse

## Test Dependencies
- Tests depend on T3 (auth) and T4 (Task entity) being implemented — both are completed
- Tests depend on T5 (POST /api/tasks for creating test tasks) — uses task creation in setUp
- If T5 is not yet implemented, tests will fail in setUp; worker-6 must implement T6 endpoints

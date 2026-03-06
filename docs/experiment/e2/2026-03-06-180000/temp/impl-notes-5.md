# Implementation Notes for T5: Task State Machine

## Framework Pitfalls

1. **No Lombok** - The project does NOT use Lombok (not in pom.xml). Write all getters/setters manually. Check `Task.java` for the pattern.

2. **Exception handling pattern** - TaskController uses `@ExceptionHandler` at the controller level (not global). For the 400 "Cannot transition" error, you need to either:
   - Add a new exception class (like `TaskService.TaskNotFoundException`) and handle it in TaskController
   - Or throw an appropriate exception that gets mapped to 400
   - The error format is `{"message": "..."}` — see `TaskController:75-78` for the pattern.

3. **TaskService.toResponse()** is private — the `transition()` method should live in TaskStateMachine as a service that delegates to TaskRepository, OR the transition logic can be in TaskService using TaskStateMachine. Either way, the controller should return `TaskResponse` using the same `toResponse` mapping.

4. **SecurityContextHolder.getContext().getAuthentication().getPrincipal()** returns `Long` (userId) — see `TaskController:41`. The new endpoints need authentication but don't need the userId for the state machine logic.

5. **Soft delete** - Use `findByIdAndDeletedFalse(id)` for task lookup (see `TaskRepository`). Don't use `findById` or deleted tasks will be found.

6. **updatedAt must be set** - On status transition, set `task.setUpdatedAt(LocalDateTime.now())` before saving. The `@PrePersist` only sets `createdAt`, there's no `@PreUpdate` — see `Task.java:56-59`.

7. **PATCH endpoint path** - The contract says `PATCH /api/tasks/{id}/status`. Since TaskController is `@RequestMapping("/api/tasks")`, the method mapping should be `@PatchMapping("/{id}/status")`.

8. **GET transitions path** - `GET /api/tasks/{id}/transitions` maps to `@GetMapping("/{id}/transitions")` on TaskController.

9. **TransitionResponse.availableTransitions** is `List<String>` (not `List<TaskStatus>`). Convert enum values to strings: `status.name()`.

10. **StatusChangeRequest.status** field uses `Task.TaskStatus` enum. Use `@jakarta.validation.constraints.NotNull` on it. Spring will handle enum deserialization from JSON string automatically.

11. **TaskStateMachine should be a Spring @Service** so it can be injected. It does NOT need TaskRepository — it's pure transition logic. The `transition()` method that actually updates the task should be in TaskService (or TaskStateMachine with TaskRepository injected). The tests expect `TaskStateMachine` to be instantiable with `new TaskStateMachine()` for unit testing, so keep it simple with no constructor args.

12. **Import for @PatchMapping** - Make sure to import `org.springframework.web.bind.annotation.PatchMapping` in TaskController.

## Test Expectations

- Unit test (`TaskStateMachineTest`): Calls `new TaskStateMachine()` directly — needs a no-arg constructor.
  - Methods expected: `canTransition(TaskStatus from, TaskStatus to)` returns boolean, `getAvailableTransitions(TaskStatus status)` returns `List<TaskStatus>`.
- Integration test (`StatusTransitionIntegrationTest`): Uses `PATCH /api/tasks/{id}/status` with body `{"status":"IN_PROGRESS"}` and `GET /api/tasks/{id}/transitions`.
  - PATCH returns `TaskResponse` (same shape as POST/PUT responses).
  - GET transitions returns `{"currentStatus":"TODO","availableTransitions":["IN_PROGRESS"]}`.
  - Invalid transition returns 400 with `{"message":"Cannot transition from X to Y"}`.
  - 404 for nonexistent task, 401 without token.

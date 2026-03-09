# QA Log — T6: M3 — Task State Machine

## Result: QA-PASS

## Files Reviewed
- `service/TaskStateMachineService.java` (76 lines)
- `controller/TaskStatusController.java` (45 lines)
- `dto/StatusUpdateRequest.java` (12 lines)
- `dto/TransitionResponse.java` (20 lines)
- `controller/TaskStatusControllerTest.java` (594 lines, 20 test methods)

## STEP 1: Requirements <-> Tests

| AC | Description | Test(s) | Verdict |
|----|-------------|---------|---------|
| AC1 | Initial status = TODO | `newTask_hasTodoStatus` | PASS |
| AC2 | TODO -> IN_PROGRESS (200) | `todoToInProgress_returns200` | PASS |
| AC3 | IN_PROGRESS -> TODO (200) | `inProgressToTodo_returns200` | PASS |
| AC4 | IN_PROGRESS -> DONE (200) | `inProgressToDone_returns200` | PASS |
| AC5 | DONE -> CLOSED (200) | `doneToClosed_returns200` | PASS |
| AC6 | TODO -> DONE illegal (400) | `todoToDone_returns400` | PASS |
| AC7 | CLOSED terminal (400) | `closedToAny_returns400` (3 sub-cases) | PASS |
| AC8 | TODO -> CLOSED illegal (400) | `todoToClosed_returns400` | PASS |
| AC9 | DONE -> TODO illegal (400) | `doneToTodo_returns400`, `doneToInProgress_returns400`, `inProgressToClosed_returns400` | PASS |
| AC10 | updatedAt on success | `successfulTransition_updatesTimestamp`, `failedTransition_doesNotUpdateTimestamp` | PASS |
| AC11 | GET transitions TODO -> [IN_PROGRESS] | `todoTransitions` | PASS |
| AC12 | GET transitions IN_PROGRESS -> [TODO, DONE] | `inProgressTransitions` | PASS |
| AC13 | GET transitions DONE -> [CLOSED] | `doneTransitions` | PASS |
| AC14 | GET transitions CLOSED -> [] | `closedTransitions` | PASS |
| AC15 | 401 no/invalid token | `patchStatus_noToken_returns401`, `getTransitions_noToken_returns401`, `patchStatus_invalidToken_returns401` | PASS |
| AC16 | 404 non-existent task | `patchStatus_nonExistentTask_returns404`, `getTransitions_nonExistentTask_returns404` | PASS |

All 16 ACs covered by tests. Tests also include bonus coverage: full lifecycle, bounce lifecycle, response format validation, same-status transition.

## STEP 2: Tests <-> Code

- Transition map (`EnumMap<TaskStatus, List<TaskStatus>>`) defines exactly 4 legal transitions matching AC2-AC5.
- `updateStatus()` throws `IllegalStateException` on illegal transition -> controller `@ExceptionHandler` returns 400.
- `getAvailableTransitions()` returns `TransitionResponse(currentStatus, availableTransitions)` -> matches test assertions.
- `ResourceNotFoundException` from `findTask()` -> `GlobalExceptionHandler` returns 404 with `{message, timestamp}`.
- `IllegalArgumentException` (invalid enum value) -> controller handler returns 400.
- `@PreUpdate` on Task entity sets `updatedAt` automatically on `taskRepository.save()`.

All test expectations match implementation behavior.

## STEP 3: Requirements <-> Code

- CONTRACT line 12: `PATCH /api/tasks/{id}/status` req=`{status:enum}` res=`TaskResponse` errors=400,401,404 -- MATCH.
- CONTRACT line 13: `GET /api/tasks/{id}/transitions` res=`{currentStatus,availableTransitions}` errors=401,404 -- MATCH.
- Legal transitions: TODO->IN_PROGRESS, IN_PROGRESS->TODO, IN_PROGRESS->DONE, DONE->CLOSED -- all 4 correct.
- CLOSED is terminal (empty transition list) -- correct.
- `updatedAt` updated via JPA `@PreUpdate` on successful save; not updated on failed transition (exception thrown before save) -- correct.
- Soft-deleted tasks filtered in `findTask()` -- correct.

## STEP 4: Standards — SKIPPED (none provided)

## STEP 5: Code Quality

- State machine uses `EnumMap` with immutable `List.of()` -- clean and performant.
- Controller is thin, properly delegates to service.
- Exception handling well-structured: controller-level for domain errors, global for infrastructure.
- `toResponse()` in service duplicates mapping logic from `TaskService` -- minor duplication, acceptable for service decoupling.
- `StatusUpdateRequest` uses `@NotNull` validation on status field -- correct.

## STEP 6: File Scope

Files created/modified: `TaskStateMachineService.java`, `TaskStatusController.java`, `StatusUpdateRequest.java`, `TransitionResponse.java` -- all within ALLOWED list. PASS.

## Issues Found

None.

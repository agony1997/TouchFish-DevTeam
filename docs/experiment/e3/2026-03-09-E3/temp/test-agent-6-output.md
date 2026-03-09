# Test Agent 6 Output — T6: M3 Task State Machine

## Test Files
- `taskflow/backend/src/test/java/com/taskflow/controller/TaskStatusControllerTest.java`

## Test Count
- **27 tests** total across 8 nested test classes

## Test Breakdown by AC

| AC | Tests | Description |
|----|-------|-------------|
| AC1 | 1 | New task initial status = TODO |
| AC2 | 1 | TODO -> IN_PROGRESS -> 200 |
| AC3 | 1 | IN_PROGRESS -> TODO -> 200 |
| AC4 | 1 | IN_PROGRESS -> DONE -> 200 |
| AC5 | 1 | DONE -> CLOSED -> 200 |
| AC6 | 1 | TODO -> DONE -> 400 |
| AC7 | 1 | CLOSED -> any -> 400 (tests 3 targets) |
| AC8 | 1 | TODO -> CLOSED -> 400 |
| AC9 | 2 | DONE -> TODO -> 400, DONE -> IN_PROGRESS -> 400 |
| AC10 | 2 | updatedAt set on success, unchanged on failure |
| AC11 | 1 | GET transitions for TODO -> [IN_PROGRESS] |
| AC12 | 1 | GET transitions for IN_PROGRESS -> [TODO, DONE] |
| AC13 | 1 | GET transitions for DONE -> [CLOSED] |
| AC14 | 1 | GET transitions for CLOSED -> [] |
| AC15 | 3 | No token -> 401 (PATCH, GET, invalid token) |
| AC16 | 2 | Non-existent task -> 404 (PATCH, GET) |
| Extra | 3 | IN_PROGRESS->CLOSED=400, same-status, response format |
| Extra | 2 | Full lifecycle, bounce lifecycle |
| Extra | 1 | Response shape for transitions endpoint |

## Compile Status
- **PASS** — all tests compile successfully

## Run Status
- **27 failures** — expected; `POST /api/tasks` (T5) and state machine endpoints (T6) not yet implemented
- All failures are in setUp (creating the task) — returns 404 because TaskController/TaskService don't exist yet
- No compile errors, no unexpected 500s

## Implementation Notes
- Written to: `docs/dev-team/2026-03-09-E3/temp/impl-notes-6.md`
- Key: worker needs StatusUpdateRequest, TransitionResponse, TaskStateMachineService, TaskStatusController
- State transition map: TODO->[IN_PROGRESS], IN_PROGRESS->[TODO,DONE], DONE->[CLOSED], CLOSED->[]
- Task @PreUpdate handles updatedAt automatically
- GlobalExceptionHandler needs handler for illegal transition exception -> 400

## Dependencies
- T5 (Task CRUD API) must be implemented first — tests use `POST /api/tasks` in setUp
- T3 (Auth) — already implemented
- T4 (Task Entity) — already implemented

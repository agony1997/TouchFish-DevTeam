# Plan Summary — TaskFlow M1+M2+M3+M6

## Tasks (7)

| Task | Module | Description | Blocked By | ALLOWED Files |
|------|--------|-------------|------------|---------------|
| T1 | M1 | User Entity + JWT Security Foundation | - | 5 |
| T2 | M1 | Auth Service + Controller + DTOs | T1 | 5 |
| T3 | M2 | Task Entity + Repository + DTOs | T1 | 5 |
| T4 | M2 | Task Service + Controller | T2, T3 | 2 |
| T5 | M3 | Task State Machine | T4 | 4 |
| T6 | M6 | Frontend API Layer + Store + Router | - | 5 |
| T7 | M6 | Frontend TaskListPage + Components | T6 | 5 |

## Execution Waves

- **Wave 1**: T1 (backend auth infra) + T6 (frontend API) — parallel
- **Wave 2**: T2 (auth API) + T3 (task entity) + T7 (frontend components) — parallel
- **Wave 3**: T4 (task CRUD controller)
- **Wave 4**: T5 (state machine)

## Estimated Spawns

Per task: 1 test-agent (Opus) + 1 worker (Sonnet) + 1 qa-task (Sonnet) = 3 agents
Total: 7 test-agents + 7 workers + 7 qa-tasks + 1 qa-global + 1 delivery-sub
= **8 Opus + 15 Sonnet**, max 3 concurrent Workers

## Cross-Cutting Concerns

1. Error response: `{ "message": "..." }` + HTTP status code
2. JWT filter: OncePerRequestFilter, permit /api/auth/**
3. Soft delete: `deleted` boolean on Task, auto-filtered in queries
4. Validation: Jakarta @Valid, 400 Bad Request
5. Axios interceptor: auto-attach Bearer token

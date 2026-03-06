# Plan Summary — TaskFlow M1+M2+M3+M6

## Tasks (7)

| ID | Task | Module | ALLOWED Files | blockedBy |
|----|------|--------|---------------|-----------|
| T1 | M1-Foundation: User Entity + JWT | M1 | 5 (User, UserRepo, JwtTokenProvider, JwtAuthFilter, SecurityConfig) | - |
| T2 | M1-Auth-API: Controller + Service + DTOs | M1 | 6 (AuthController, AuthService, RegisterRequest, LoginRequest, LoginResponse, UserResponse) | T1 |
| T3 | M2+M3-Entity: Task + TaskStatus + Repo + DTOs | M2,M3 | 5 (Task, TaskStatus, TaskRepo, TaskResponse, CreateTaskRequest) | - |
| T4 | M2-CRUD-API: Service + Controller + Exceptions | M2 | 5 (TaskService, TaskController, UpdateTaskRequest, PageResponse, GlobalExceptionHandler) | T1, T3 |
| T5 | M3-Status-API: State Transition Endpoints | M3 | 5 (StatusTransitionRequest, TransitionsResponse, InvalidTransitionException, TaskService*, TaskController*) | T4 |
| T6 | M6-Frontend-Infra: Store + API + Router + Types | M6 | 4 (task.ts types, taskApi.ts, taskStore.ts, router/index.ts*) | - |
| T7 | M6-Frontend-UI: Page + Components | M6 | 5 (TaskListPage, TaskFilters, TaskListItem, TaskCreateForm, Pagination) | T6 |

*modify existing file

## Execution Waves

```
Wave 1 (parallel): T1 + T3 + T6    ← 3 Workers concurrent
Wave 2 (parallel): T2 + T7          ← after respective deps
Wave 3:            T4               ← after T1 + T3
Wave 4:            T5               ← after T4
```

## Agent Estimates

- Per task: 1 test-agent (Opus) + 1 Worker (Sonnet) + 1 qa-task (Sonnet) = 3 agents
- 7 tasks x 3 = 21 agents
- qa-global: 1 (Opus)
- delivery-sub: 1 (Sonnet)
- **Total: 23 agents (8 Opus + 15 Sonnet), max 3 concurrent Workers**

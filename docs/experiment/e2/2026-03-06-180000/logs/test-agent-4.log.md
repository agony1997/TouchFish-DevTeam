# Test Agent 4 Log — T4: Task Service + Controller Tests

## Context Read
- Read all READONLY files: Task.java, TaskRepository.java, TaskRequest.java, TaskResponse.java, TaskPageResponse.java, User.java, JwtTokenProvider.java, JwtAuthenticationFilter.java, SecurityConfig.java, AuthController.java, AuthService.java
- Read PLAN.md and CONTRACT.md
- Studied AuthControllerIntegrationTest.java and AuthServiceTest.java for patterns

## Test Files Written

### 1. TaskServiceTest.java (unit test, Mockito)
Path: `backend/src/test/java/com/taskflow/service/TaskServiceTest.java`
- **CreateTask** (6 tests): create returns response, sets createdBy, default MEDIUM priority, explicit priority, status TODO, sets title/description
- **GetTask** (4 tests): found, not found throws, soft-deleted throws, maps all fields
- **UpdateTask** (4 tests): update returns response, not found throws, sets updatedAt, preserves createdBy/status
- **DeleteTask** (3 tests): soft deletes (sets deleted=true), not found throws, never hard deletes
- **ListTasks** (7 tests): returns page response, keyword filter, priority filter, status filter, combined filters, empty page, only non-deleted

**Total: 24 unit tests**

### 2. TaskControllerIntegrationTest.java (integration, @SpringBootTest + MockMvc)
Path: `backend/src/test/java/com/taskflow/controller/TaskControllerIntegrationTest.java`
- Uses real auth flow: register user -> login -> extract JWT token in @BeforeEach
- **CreateTask** (7 tests): AC1 (201 + response), AC2 (default MEDIUM), AC3 (missing/blank title -> 400), AC15 (title boundary 200/201), AC16 (desc boundary 5000/5001)
- **GetTask** (3 tests): AC4 (200), AC5 (404 not found), AC9 (404 soft-deleted)
- **UpdateTask** (3 tests): AC6 (200 updated), AC6 (404 not found), blank title 400
- **DeleteTask** (3 tests): AC7 (204), AC7 (404 not found), AC8 (not in list after delete)
- **ListTasks** (6 tests): AC10 (pagination), AC10 (empty), AC11 (keyword + case-insensitive), AC12 (priority/status filter), AC13 (page params + second page)
- **Auth** (6 tests): AC14 (401 for all 5 endpoints without token, 401 with invalid token)
- **E2E** (1 test): full Create -> Read -> Update -> Delete flow

**Total: 29 integration tests**

### 3. impl-notes-4.md
Path: `docs/dev-team/2026-03-06-180000/temp/impl-notes-4.md`
- TaskService method signatures and implementation guidance
- TaskController structure with annotations
- SecurityContext userId extraction pattern
- Exception handling pattern (follow AuthController)
- Task->TaskResponse mapping
- Framework pitfalls: @Valid required, @PageableDefault sort, principal is Long, 204 no body, soft delete only

## AC Coverage
| AC | Covered By |
|----|-----------|
| AC1 | CreateTask 201 + response fields |
| AC2 | Default MEDIUM priority |
| AC3 | Missing/blank title -> 400 |
| AC4 | Get by id -> 200 |
| AC5 | Not found -> 404 |
| AC6 | Update -> 200 |
| AC7 | Delete -> 204 |
| AC8 | Soft-deleted not in list |
| AC9 | Soft-deleted -> 404 on get |
| AC10 | Pagination response |
| AC11 | Keyword search |
| AC12 | Priority filter |
| AC13 | Page params |
| AC14 | No token -> 401 |
| AC15 | Title > 200 -> 400 |
| AC16 | Description > 5000 -> 400 |

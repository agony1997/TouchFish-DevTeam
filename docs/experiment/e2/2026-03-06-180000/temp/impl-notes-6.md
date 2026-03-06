# Implementation Notes - T6: Frontend API Layer + Store + Router

## Framework/Tooling Notes

- vite.config.ts was updated with `test: { environment: 'jsdom', globals: true }` and `@` path alias (`resolve: { alias: { '@': resolve(__dirname, 'src') } }`)
- Vitest reference comment added: `/// <reference types="vitest" />`
- These changes are required for tests to resolve `@/...` imports and run in jsdom environment

## File-by-File Implementation Guide

### 1. `frontend/src/api/axios.ts`
- Export a default axios instance created with `axios.create({ baseURL: '/api' })`
- Add a request interceptor that reads `localStorage.getItem('token')`
- If token exists, set `config.headers['Authorization'] = 'Bearer ' + token`
- If no token, do NOT set the header (do not set it to empty string either)
- The interceptor handlers are accessed via `(api.interceptors.request as any).handlers[0]` in tests

### 2. `frontend/src/api/auth.ts`
- Import the default api instance from `@/api/axios`
- `login(username, password)`: POST `/auth/login` with `{username, password}`, store `response.data.token` in `localStorage.setItem('token', ...)`, return `response.data`
- `register(username, password, displayName)`: POST `/auth/register` with `{username, password, displayName}`, return `response.data`
- `getMe()`: GET `/auth/me`, return `response.data`
- All functions are named exports

### 3. `frontend/src/api/tasks.ts`
- Import the default api instance from `@/api/axios`
- `fetchTasks(params)`: GET `/tasks` with `{ params }`, return `response.data`
  - params object: `{ page, size, keyword?, priority?, status? }`
- `getTask(id)`: GET `/tasks/${id}`, return `response.data`
- `createTask(data)`: POST `/tasks` with data body, return `response.data`
- `updateTask(id, data)`: PUT `/tasks/${id}` with data body, return `response.data`
- `deleteTask(id)`: DELETE `/tasks/${id}` (no return needed)
- `changeStatus(id, status)`: PATCH `/tasks/${id}/status` with `{ status }`, return `response.data`
- `getTransitions(id)`: GET `/tasks/${id}/transitions`, return `response.data`
- All functions are named exports

### 4. `frontend/src/stores/taskStore.ts`
- Use `defineStore('task', ...)` from Pinia
- State: `tasks: []`, `page: 0`, `size: 20`, `totalElements: 0`, `totalPages: 0`, `keyword: ''`, `priorityFilter: null`, `statusFilter: null`, `loading: false`, `error: null`
- Actions:
  - `fetchTasks()`: set `loading=true`, call API `fetchTasks({page, size, keyword, priority: priorityFilter, status: statusFilter})`, update state from response, set `loading=false`. On error, set `error` and `loading=false`
  - `setKeyword(kw)`: set `keyword = kw`, reset `page = 0`
  - `setPriorityFilter(p)`: set `priorityFilter = p`, reset `page = 0`
  - `setStatusFilter(s)`: set `statusFilter = s`, reset `page = 0`
  - `setPage(p)`: set `page = p`
  - `createTask(data)`: call API `createTask(data)`, then call `this.fetchTasks()` to refresh

### 5. `frontend/src/router/index.ts` (modify existing)
- Add route: `{ path: '/tasks', name: 'tasks', component: () => import('../views/TaskListPage.vue') }`
- Keep existing home route

## Gotchas / Traps
- The axios interceptor test accesses internal `handlers[0].fulfilled` — make sure the interceptor is the first one registered (it will be if there's only one)
- Tests mock `@/api/axios` with a plain object `{ default: { post: vi.fn(), get: vi.fn(), ... } }` — the implementation must use the default export
- The `fetchTasks` params in the store test expects the exact shape: `{ page, size, keyword, priority, status }` — the keys must be `priority` and `status`, NOT `priorityFilter`/`statusFilter`
- `login()` must store token BEFORE returning, as tests check `localStorage` after await
- Router lazy load test checks `typeof routeConfig.component === 'function'` — use `() => import(...)` syntax

# Implementation Notes — T7: M6 Frontend Infrastructure + Store

## Mocking Strategy

### Axios Mock (`vi.mock('@/api/axios')`)
The auth-api and tasks-api tests mock `@/api/axios` with a factory function that returns a fake object with `post`, `get`, `put`, `delete`, `patch` as `vi.fn()`. This prevents the real axios instance from being imported and allows tests to assert on HTTP method calls.

```ts
vi.mock('@/api/axios', () => ({
  default: {
    post: vi.fn(),
    get: vi.fn(),
    put: vi.fn(),
    delete: vi.fn(),
    patch: vi.fn(),
    defaults: { baseURL: '/api' },
    interceptors: {
      request: { use: vi.fn(), handlers: [] },
      response: { use: vi.fn(), handlers: [] },
    },
  },
}))
```

### Pinia in Tests
Use `createPinia()` + `setActivePinia()` in `beforeEach`:
```ts
import { setActivePinia, createPinia } from 'pinia'
beforeEach(() => {
  setActivePinia(createPinia())
})
```

### localStorage Mock
Vitest with jsdom provides `localStorage` automatically. Use `localStorage.clear()` in `beforeEach`/`afterEach` to reset state.

### Tasks API Mock (for store tests)
```ts
vi.mock('@/api/tasks', () => ({
  getTasks: vi.fn(),
  createTask: vi.fn(),
  // ... etc
}))
```

## File Structure Expected by Tests

```
src/api/axios.ts      — default export: AxiosInstance with baseURL='/api', request interceptor
src/api/auth.ts       — named exports: login(username, password), register(data)
src/api/tasks.ts      — named exports: getTasks, getTask, createTask, updateTask, deleteTask, updateStatus, getTransitions
src/stores/taskStore.ts — named export: useTaskStore (Pinia defineStore)
src/router/index.ts   — must include /tasks (name='task-list') and /tasks/:id (name='task-detail') routes
```

## Implementation Contracts (what the tests verify)

### axios.ts
- `api.defaults.baseURL` must be `'/api'`
- Request interceptor at `api.interceptors.request.handlers[0]`
- If `localStorage.getItem('token')` returns a value, set `config.headers.Authorization = 'Bearer ' + token`
- If no token, do NOT set Authorization header

### auth.ts
- `login(username, password)` → `api.post('/auth/login', { username, password })` → returns `response.data`
- `register(data)` → `api.post('/auth/register', data)` → returns `response.data`

### tasks.ts
- `getTasks(params?)` → `api.get('/tasks', { params })` → returns `response.data`
- `getTask(id)` → `api.get('/tasks/{id}')` → returns `response.data`
- `createTask(data)` → `api.post('/tasks', data)` → returns `response.data`
- `updateTask(id, data)` → `api.put('/tasks/{id}', data)` → returns `response.data`
- `deleteTask(id)` → `api.delete('/tasks/{id}')`
- `updateStatus(id, status)` → `api.patch('/tasks/{id}/status', { status })` → returns `response.data`
- `getTransitions(id)` → `api.get('/tasks/{id}/transitions')` → returns `response.data`

### taskStore.ts (Pinia)
- State defaults: `tasks=[], page=0, size=20, totalElements=0, totalPages=0, keyword='', priorityFilter='', statusFilter='', loading=false, error=null`
- `fetchTasks()`: sets `loading=true`, calls `getTasks({page, size, keyword, priority, status})`, updates state, sets `loading=false`. On error: sets `error=message`, `loading=false`
- `setKeyword(kw)`: sets `keyword`, resets `page=0`
- `setPriorityFilter(p)`: sets `priorityFilter`, resets `page=0`
- `setStatusFilter(s)`: sets `statusFilter`, resets `page=0`
- `setPage(p)`: sets `page`
- `createTask(data)`: calls API `createTask(data)`, then calls `fetchTasks()`

### router/index.ts
- Add route `{ path: '/tasks', name: 'task-list', component: ... }`
- Add route `{ path: '/tasks/:id', name: 'task-detail', component: ... }`

## Vitest Configuration
- `vite.config.ts` already has `test.environment: 'jsdom'` and `test.globals: true`
- The `@` alias resolves to `src/` via `resolve.alias` in vite config
- Stub files created at `src/api/axios.ts`, `src/api/auth.ts`, `src/api/tasks.ts`, `src/stores/taskStore.ts` to allow import resolution in TDD

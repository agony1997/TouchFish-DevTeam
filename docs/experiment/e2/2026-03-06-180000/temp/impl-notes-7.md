# Implementation Notes for T7: Frontend TaskListPage + Components

## Test Structure
- `tests/views/TaskListPage.test.ts` — 9 tests
- `tests/components/TaskFilters.test.ts` — 5 tests
- `tests/components/TaskListItem.test.ts` — 14 tests
- `tests/components/TaskCreateForm.test.ts` — 7 tests
- `tests/components/Pagination.test.ts` — 10 tests
- **Total: 45 tests**

## Framework Pitfalls & Key Notes

### Pinia Store Mocking
- `@pinia/testing` is NOT in package.json. Do NOT try to import `createTestingPinia`.
- Tests use real Pinia store with `createPinia()` + `setActivePinia()`. The API module (`@/api/tasks`) is mocked via `vi.mock` at the top level so store actions don't make real HTTP calls.
- To set store state in tests, import the store composable and set properties directly (e.g., `store.loading = true`).

### vue-router
- TaskListPage tests mock `vue-router` via `vi.mock('vue-router', ...)` providing `useRouter` and `useRoute`.
- If the component uses `<router-link>`, you may need to add `RouterLinkStub` from `@vue/test-utils` or use `stubs: { RouterLink: true }`.

### Component Import Resolution
- Components are expected at their exact paths: `@/components/TaskFilters.vue`, `@/components/TaskListItem.vue`, `@/components/TaskCreateForm.vue`, `@/components/Pagination.vue`.
- The `@` alias resolves to `src/` (configured in vite.config.ts).
- If the component file does not exist, vitest will fail at import resolution (not at assertion). Create all .vue files before running tests.

### TaskListPage Composition
- Tests stub child components (`TaskFilters`, `TaskCreateForm`, `Pagination`) via `global.stubs` and check for their presence via kebab-case tag names in rendered HTML (e.g., `task-filters-stub`).
- TaskListItem is NOT stubbed — the page should render actual TaskListItem components so task data appears in output. If you prefer to stub TaskListItem too, update the stubs config.
- The page should call `taskStore.fetchTasks()` in `onMounted`.

### TaskFilters Debounce
- Tests use `vi.useFakeTimers()`. The search input test advances time by 300ms via `vi.advanceTimersByTime(300)`.
- Implementation must use a debounce mechanism (e.g., `setTimeout` with 300ms, or a `watchEffect` with debounce).
- After debounce, the component should call `store.setKeyword(value)` then `store.fetchTasks()`.

### TaskListItem Badge Colors
- Tests check for exact hex color codes in the rendered HTML (inline styles or CSS).
- Status colors: TODO=#6B7280, IN_PROGRESS=#3B82F6, DONE=#10B981, CLOSED=#EF4444
- Priority colors: HIGH=#EF4444, MEDIUM=#F59E0B, LOW=#10B981
- Use inline `:style` bindings to ensure the hex codes appear in the HTML.

### TaskCreateForm
- `visible` prop controls rendering. When `false`, the form content should not be rendered (use `v-if="visible"`).
- Priority `<select>` must default to `'MEDIUM'` (bind to reactive data initialized to `'MEDIUM'`).
- On submit: call `store.createTask({ title, description, priority })`, then emit `'created'`.
- Title validation: do NOT call createTask if title is empty.
- Cancel button emits `'cancel'`.

### Pagination
- Props: `currentPage` (0-indexed), `totalPages`.
- Display human-readable page (currentPage + 1) and totalPages.
- Prev disabled when `currentPage === 0`.
- Next disabled when `currentPage >= totalPages - 1`.
- Prev click emits `page-change` with `currentPage - 1`.
- Next click emits `page-change` with `currentPage + 1`.
- When `totalPages <= 1`, both buttons should be disabled.

### Empty State
- TaskListPage should show a message matching pattern: "no task" / "empty" / Chinese equivalent when `store.tasks` is empty and not loading and no error.

### afterEach Cleanup
- TaskFilters tests need `import { afterEach } from 'vitest'` at the top-level or `vi.useRealTimers()` in afterEach. Currently `afterEach` is used without import — vitest globals are enabled so this works.

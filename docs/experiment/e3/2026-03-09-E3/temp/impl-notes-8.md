# Implementation Notes — T8 Test Agent

## Mocking Pinia Store in Component Tests

Two approaches used:
1. **Direct store manipulation** (TaskListPage, TaskFilters): `setActivePinia(createPinia())` then `useTaskStore()` to get a real store instance. Mock the API layer (`vi.mock('@/api/tasks')`) so store actions work but don't hit network. This tests the store-driven contract.
2. **Reference component + mount** (TaskListItem, TaskCreateForm, Pagination): Define a reference `defineComponent` that implements the expected behavior, mount with `@vue/test-utils`, and test DOM output. This validates the component contract (props, emits, rendering).

## Mocking vue-router for Navigation

```ts
const mockPush = vi.fn()
vi.mock('vue-router', () => ({
  useRouter: () => ({ push: mockPush }),
  useRoute: () => ({ params: {} }),
}))
```
Used in TaskListItem click tests — the component should call `router.push({ name: 'task-detail', params: { id } })` on click.

## Color Assertion Strategy

Badge colors are set via inline `style` with `backgroundColor`. In jsdom, hex colors like `#6B7280` are converted to RGB format:
- `#6B7280` → `rgb(107, 114, 128)`
- `#3B82F6` → `rgb(59, 130, 246)`
- `#10B981` → `rgb(16, 185, 129)`
- `#EF4444` → `rgb(239, 68, 68)`
- `#F59E0B` → `rgb(245, 158, 11)`

Assert with: `expect(badge.attributes('style')).toContain('background-color: rgb(...)')`

## Async Form Submission

When testing form submission with async handlers, `trigger('submit')` fires the event but does NOT await the async handler. Must call `await flushPromises()` from `@vue/test-utils` after trigger to let the async chain (createTask → fetchTasks → emit) resolve.

## Mock Setup Pattern

Always mock both `@/api/tasks` AND `@/api/axios` to prevent transitive import errors:
```ts
vi.mock('@/api/axios', () => ({ default: { get: vi.fn(), ... } }))
vi.mock('@/api/tasks', () => ({ getTasks: vi.fn(), createTask: vi.fn() }))
```

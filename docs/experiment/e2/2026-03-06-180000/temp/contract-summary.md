# Contract Summary

## Endpoints (10)

| Method | Path | Auth | Success | Errors |
|--------|------|------|---------|--------|
| POST | /api/auth/register | No | 201 {id, username, displayName} | 400, 409 |
| POST | /api/auth/login | No | 200 {token, expiresIn} | 401 |
| GET | /api/auth/me | Bearer | 200 {id, username, displayName} | 401 |
| POST | /api/tasks | Bearer | 201 TaskResponse | 400, 401 |
| GET | /api/tasks/{id} | Bearer | 200 TaskResponse | 401, 404 |
| PUT | /api/tasks/{id} | Bearer | 200 TaskResponse | 400, 401, 404 |
| DELETE | /api/tasks/{id} | Bearer | 204 | 401, 404 |
| GET | /api/tasks | Bearer | 200 TaskPageResponse | 401 |
| PATCH | /api/tasks/{id}/status | Bearer | 200 TaskResponse | 400, 401, 404 |
| GET | /api/tasks/{id}/transitions | Bearer | 200 TransitionResponse | 401, 404 |

## State Machine

```
TODO → IN_PROGRESS ↔ TODO
IN_PROGRESS → DONE → CLOSED (terminal)
```

## Error Format

```json
{ "message": "error description" }
```

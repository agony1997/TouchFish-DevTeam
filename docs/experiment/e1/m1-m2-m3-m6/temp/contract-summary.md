# Contract Summary

## Endpoints (10)

| Method | Path | Module | Auth |
|--------|------|--------|------|
| POST | /api/auth/register | M1 | No |
| POST | /api/auth/login | M1 | No |
| GET | /api/auth/me | M1 | Yes |
| POST | /api/tasks | M2 | Yes |
| GET | /api/tasks/{id} | M2 | Yes |
| PUT | /api/tasks/{id} | M2 | Yes |
| DELETE | /api/tasks/{id} | M2 | Yes |
| GET | /api/tasks | M2 | Yes |
| PATCH | /api/tasks/{id}/status | M3 | Yes |
| GET | /api/tasks/{id}/transitions | M3 | Yes |

## State Machine

```
TODO ---> IN_PROGRESS ---> DONE ---> CLOSED
  ^            |
  |____________|
```

## Error Response Format
```json
{ "status": 400, "message": "error description", "errors": ["field: detail"] }
```

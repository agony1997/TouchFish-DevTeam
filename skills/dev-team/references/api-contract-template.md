# API Contract

> Project: {name} | Version: 1 | Date: {date}
> Approved by: user (Phase 2)

## Shared Types

```typescript
// Define shared request/response types here
```

## Endpoints

### {METHOD} {path}

- **Description**: {what it does}
- **Request**: `{type or body schema}`
- **Response 200**: `{type or body schema}`
- **Errors**: `{error codes and format}`

## Error Format

```json
{
  "code": "ERROR_CODE",
  "message": "Human-readable message"
}
```

## Amendment Log

| # | Date | Change | Approved by |
|---|------|--------|-------------|

<!-- Any contract change requires TL approval.
     Worker → Leader → TL → decision → notify all leaders. -->

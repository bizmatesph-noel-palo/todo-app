---
inclusion: manual
---

# API Reference

## Base URL

`/api`

## Authentication

Currently: **None** (tech-debt #1 — to be secured with Sanctum)

Planned: Laravel Sanctum cookie-based authentication for the SPA. All API routes will require `auth:sanctum` middleware.

## Endpoints

### Tasks

| Method | Endpoint | Action | Description |
|--------|----------|--------|-------------|
| GET | `/api/tasks` | index | List all tasks |
| POST | `/api/tasks` | store | Create a new task |
| GET | `/api/tasks/{id}` | show | Get a single task |
| PUT | `/api/tasks/{id}` | update | Update a task |
| DELETE | `/api/tasks/{id}` | destroy | Delete a task |

### Request: Create/Update Task
```json
{
    "title": "Buy groceries",
    "status": "created",
    "completed": false
}
```

### Response: Single Task (TaskResource)
```json
{
    "data": {
        "id": 1,
        "title": "Buy groceries",
        "status": "created",
        "completed": false,
        "completed_at": null,
        "created_at": "2026-06-29T05:27:22.000000Z",
        "updated_at": "2026-06-29T05:27:22.000000Z"
    }
}
```

### Response: Task Collection
```json
{
    "data": [
        {
            "id": 1,
            "title": "Buy groceries",
            "status": "created",
            "completed": false,
            "completed_at": null,
            "created_at": "2026-06-29T05:27:22.000000Z",
            "updated_at": "2026-06-29T05:27:22.000000Z"
        }
    ]
}
```

## Response Envelope

All responses use Laravel API Resources with a consistent structure:

- **Success (single):** `{ "data": { ... } }`
- **Success (collection):** `{ "data": [ ... ] }`
- **Error:** `{ "message": "...", "errors": { "field": ["..."] } }`
- **Empty (delete):** HTTP 204 No Content (planned — currently returns 200 with body)

## HTTP Status Codes

| Code | Meaning | When |
|------|---------|------|
| 200 | OK | Successful GET, PUT |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 404 | Not Found | Resource doesn't exist |
| 422 | Unprocessable | Validation failed |
| 401 | Unauthorized | Not authenticated (after Sanctum is added) |
| 403 | Forbidden | Not authorized for this resource |
| 500 | Server Error | Unexpected error |

## Validation Error Format (422)
```json
{
    "message": "The given data was invalid.",
    "errors": {
        "title": [
            "The title field is required."
        ]
    }
}
```

## Task Status Values

| Value | Description |
|-------|-------------|
| `created` | Newly created task |
| `in_progress` | Task is being worked on |
| `completed` | Task is done |

## Planned Improvements

1. Add Sanctum auth middleware to all `/api/tasks` routes
2. Scope tasks to authenticated user (`user_id` FK)
3. Add pagination to index endpoint
4. Add filtering by status, date range
5. Return proper 204 on delete (fix EmptyResponseResource)
6. Return proper HTTP status on errors (fix ErrorResponseResource)

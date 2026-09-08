# CodeForge Cloud

## API Reference & Service Contract Document

**Document:** 04 — API Reference
**Project:** CodeForge Cloud
**Subtitle:** Collaborative Real-Time Online Code Compiler & IDE Sandbox
**Version:** 1.1
**Status:** Draft
**Author:** Adarsh Kumar

---

# 1. API Overview

CodeForge Cloud exposes three primary communication interfaces:

1. REST API for browser-to-gateway operations
2. WebSocket / Socket.IO API for real-time collaboration and execution events
3. gRPC API for internal service-to-service communication

```text
                         ┌─────────────────────┐
                         │     Web Browser     │
                         │   React Web IDE     │
                         └──────────┬──────────┘
                                    │
                         HTTPS / WebSocket
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   Node.js Gateway   │
                         │   Public API Layer  │
                         └──────────┬──────────┘
                                    │
                  ┌─────────────────┼─────────────────┐
                  │                 │                 │
                  ▼                 ▼                 ▼
             PostgreSQL          Redis          Python Evaluator
                                                     │
                                                     │ gRPC
                                                     ▼
                                              C++ Sandbox Runtime
```

The API architecture provides:

```text
Authentication
Authorization
Project management
Project membership
File management
Execution submission
Execution status
Execution cancellation
Execution results
Execution snapshots
Real-time collaboration
Presence
WebSocket execution events
Internal gRPC communication
Structured errors
Pagination
Rate limiting
Idempotency
Observability
```

The public API must never expose the sandbox directly.

---

# 2. API Design Goals

The API must provide:

* predictable resource naming
* explicit request and response schemas
* versioned endpoints
* consistent response structures
* explicit authorization
* request correlation
* idempotent execution handling
* safe execution submission
* real-time collaboration
* real-time execution events
* pagination
* structured errors
* backward-compatible evolution
* strict input validation
* bounded resource usage

The API gateway is the public security boundary.

---

# 3. API Base URL

The public API is versioned:

```text
/api/v1
```

Example:

```text
/api/v1/projects
/api/v1/executions
/api/v1/auth/login
```

Production example:

```text
https://codeforge.example.com/api/v1
```

The exact hostname is deployment-specific.

---

# 4. API Versioning

CodeForge Cloud uses URL-based major versioning.

```text
/api/v1
/api/v2
```

Breaking changes require a new major version.

Non-breaking additions may be introduced inside the current version.

## 4.1 Non-Breaking Changes

Examples:

* adding optional response fields
* adding optional request fields
* adding endpoints
* adding optional enum values where clients safely tolerate them

## 4.2 Breaking Changes

Examples:

* removing fields
* changing field meaning
* changing authentication semantics
* changing required fields
* changing response structure incompatibly
* changing resource identifiers

---

# 5. Common API Conventions

## 5.1 Character Encoding

All JSON is UTF-8.

```http
Content-Type: application/json
```

---

## 5.2 Identifier Format

Primary identifiers use UUID.

Example:

```text
550e8400-e29b-41d4-a716-446655440000
```

Schema:

```text
type: string
format: uuid
```

---

## 5.3 Timestamp Format

All timestamps use UTC ISO-8601/RFC-3339 representation.

Example:

```text
2026-09-08T12:00:00Z
```

Database representation:

```text
TIMESTAMPTZ
```

---

## 5.4 Boolean Values

JSON booleans must be used.

```json
{
  "network_enabled": false
}
```

Strings such as `"true"` and `"false"` must not be accepted where a boolean is expected.

---

## 5.5 Null Values

A field explicitly documented as nullable may contain:

```json
null
```

Non-nullable fields must not contain null.

---

# 6. Common HTTP Methods

| Method | Purpose                           |
| ------ | --------------------------------- |
| GET    | Retrieve resource                 |
| POST   | Create resource or execute action |
| PUT    | Full replacement                  |
| PATCH  | Partial update                    |
| DELETE | Remove resource                   |

---

# 7. Common Headers

## 7.1 Authentication

```http
Authorization: Bearer <access_token>
```

Required for authenticated endpoints.

---

## 7.2 Request ID

```http
X-Request-ID: 8e91d3c0-7b1d-4c50-a1f3-3dbb5c8b0f2a
```

If absent, the gateway generates one.

---

## 7.3 Idempotency Key

Used by operations where duplicate requests could create duplicate resources.

```http
Idempotency-Key: 2f7e0e8b-4b61-4e5a-a42d-1a7e9e4a4f10
```

---

## 7.4 Optimistic Concurrency

Supported where applicable:

```http
If-Match: "revision-42"
```

---

# 8. Standard Resource Schemas

This section defines the canonical JSON representations used throughout the API.

---

# 9. User Schema

```json
{
  "id": "uuid",
  "email": "user@example.com",
  "username": "developer",
  "display_name": "Developer",
  "status": "ACTIVE",
  "created_at": "2026-09-08T12:00:00Z",
  "updated_at": "2026-09-08T12:00:00Z"
}
```

Schema:

| Field        | Type      | Required | Constraints              |
| ------------ | --------- | -------: | ------------------------ |
| id           | UUID      |      Yes | Immutable                |
| email        | string    |      Yes | Max 320 chars, unique    |
| username     | string    |      Yes | 3–50 chars               |
| display_name | string    |      Yes | 1–100 chars              |
| status       | enum      |      Yes | ACTIVE/SUSPENDED/DELETED |
| created_at   | timestamp |      Yes | UTC                      |
| updated_at   | timestamp |      Yes | UTC                      |

Password hashes are never returned.

---

# 10. Project Schema

```json
{
  "id": "uuid",
  "owner_id": "uuid",
  "name": "Trading Engine",
  "description": "High-performance trading simulation",
  "visibility": "PRIVATE",
  "created_at": "2026-09-08T12:00:00Z",
  "updated_at": "2026-09-08T12:00:00Z"
}
```

Schema:

| Field       | Type      | Required | Constraints    |
| ----------- | --------- | -------: | -------------- |
| id          | UUID      |      Yes | Immutable      |
| owner_id    | UUID      |      Yes | Existing user  |
| name        | string    |      Yes | 1–100 chars    |
| description | string    |       No | Max 5000 chars |
| visibility  | enum      |      Yes | PRIVATE/PUBLIC |
| created_at  | timestamp |      Yes | UTC            |
| updated_at  | timestamp |      Yes | UTC            |

---

# 11. Project Member Schema

```json
{
  "user_id": "uuid",
  "username": "developer",
  "display_name": "Developer",
  "role": "EDITOR",
  "joined_at": "2026-09-08T12:00:00Z"
}
```

Roles:

```text
OWNER
EDITOR
VIEWER
```

---

# 12. File Schema

```json
{
  "id": "uuid",
  "project_id": "uuid",
  "parent_id": "uuid",
  "name": "main.cpp",
  "path": "src/main.cpp",
  "type": "FILE",
  "size_bytes": 1024,
  "created_at": "2026-09-08T12:00:00Z",
  "updated_at": "2026-09-08T12:00:00Z"
}
```

File types:

```text
FILE
DIRECTORY
```

A directory must not contain file content.

---

# 13. File Content Schema

When content is explicitly requested:

```json
{
  "id": "uuid",
  "project_id": "uuid",
  "name": "main.cpp",
  "path": "src/main.cpp",
  "type": "FILE",
  "content": "#include <iostream>\nint main() {}",
  "size_bytes": 35,
  "revision": 42,
  "updated_at": "2026-09-08T12:00:00Z"
}
```

---

# 14. Execution Schema

```json
{
  "id": "uuid",
  "job_id": "uuid",
  "project_id": "uuid",
  "user_id": "uuid",
  "entry_file_id": "uuid",
  "language": "cpp",
  "language_version": "17",
  "status": "RUNNING",
  "snapshot_id": "uuid",
  "submitted_at": "2026-09-08T12:00:00Z",
  "started_at": "2026-09-08T12:00:02Z",
  "completed_at": null
}
```

---

# 15. Execution Status Enum

```text
QUEUED
STARTING
COMPILING
RUNNING
COMPLETED
FAILED
TIMEOUT
CANCELLED
RESOURCE_LIMIT
```

Terminal states:

```text
COMPLETED
FAILED
TIMEOUT
CANCELLED
RESOURCE_LIMIT
```

Terminal states are immutable.

---

# 16. Resource Policy Schema

```json
{
  "cpu_time_ms": 2000,
  "wall_time_ms": 5000,
  "memory_bytes": 268435456,
  "max_processes": 32,
  "max_output_bytes": 1048576,
  "max_disk_bytes": 10485760,
  "network_enabled": false
}
```

Schema:

| Field            | Type    | Required |  Initial Maximum |
| ---------------- | ------- | -------: | ---------------: |
| cpu_time_ms      | integer |      Yes | Platform-defined |
| wall_time_ms     | integer |      Yes | Platform-defined |
| memory_bytes     | integer |      Yes |           256 MB |
| max_processes    | integer |      Yes |               32 |
| max_output_bytes | integer |      Yes |             1 MB |
| max_disk_bytes   | integer |      Yes |            10 MB |
| network_enabled  | boolean |      Yes |            false |

The server may reject or clamp requested values.

---

# 17. Execution Result Schema

```json
{
  "id": "uuid",
  "execution_id": "uuid",
  "stdout": "Hello CodeForge\n",
  "stderr": "",
  "exit_code": 0,
  "signal": null,
  "compile_time_ms": 120,
  "execution_time_ms": 18,
  "memory_bytes": 1245184,
  "output_truncated": false,
  "resource_violation": false,
  "created_at": "2026-09-08T12:00:04Z"
}
```

---

# 18. Snapshot Schema

```json
{
  "snapshot_id": "uuid",
  "project_id": "uuid",
  "entry_file": "src/main.cpp",
  "language": "cpp",
  "language_version": "17",
  "files": [
    {
      "path": "src/main.cpp",
      "content": "#include <iostream>",
      "size_bytes": 20,
      "sha256": "..."
    }
  ],
  "snapshot_sha256": "...",
  "created_at": "2026-09-08T12:00:00Z"
}
```

---

# 19. Pagination Schema

Cursor-based:

```json
{
  "items": [],
  "pagination": {
    "next_cursor": "xyz789",
    "has_next": true
  }
}
```

Offset/page-based endpoints may return:

```json
{
  "items": [],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total": 100,
    "has_next": true
  }
}
```

---

# 20. Error Schema

All API errors use:

```json
{
  "error": {
    "code": "PROJECT_NOT_FOUND",
    "message": "The requested project does not exist.",
    "request_id": "uuid",
    "details": {}
  }
}
```

Schema:

| Field      | Type        | Required |
| ---------- | ----------- | -------: |
| code       | string      |      Yes |
| message    | string      |      Yes |
| request_id | UUID/string |      Yes |
| details    | object      |      Yes |

---

# 21. Authentication API

---

# 22. Register

```http
POST /api/v1/auth/register
```

Authentication:

```text
None
```

## Request Schema

```json
{
  "email": "user@example.com",
  "username": "developer",
  "password": "secure-password",
  "display_name": "Developer"
}
```

| Field        | Type   | Required | Constraints              |
| ------------ | ------ | -------: | ------------------------ |
| email        | string |      Yes | Valid email, max 320     |
| username     | string |      Yes | `^[A-Za-z0-9_]{3,50}$`   |
| password     | string |      Yes | Platform password policy |
| display_name | string |      Yes | 1–100 chars              |

## Response

```http
201 Created
```

```json
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "username": "developer",
    "display_name": "Developer",
    "status": "ACTIVE"
  },
  "access_token": "token",
  "expires_in": 900
}
```

Passwords are never returned or logged.

---

# 23. Login

```http
POST /api/v1/auth/login
```

## Request

```json
{
  "email": "user@example.com",
  "password": "secure-password"
}
```

## Response

```http
200 OK
```

```json
{
  "access_token": "token",
  "expires_in": 900,
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "username": "developer",
    "display_name": "Developer",
    "status": "ACTIVE"
  }
}
```

Possible errors:

```text
400 VALIDATION_ERROR
401 INVALID_CREDENTIALS
403 USER_SUSPENDED
429 RATE_LIMITED
```

---

# 24. Logout

```http
POST /api/v1/auth/logout
```

Authentication:

```text
Required
```

## Response

```http
204 No Content
```

The corresponding session is revoked.

---

# 25. Current User

```http
GET /api/v1/users/me
```

## Response

```http
200 OK
```

```json
{
  "id": "uuid",
  "email": "user@example.com",
  "username": "developer",
  "display_name": "Developer",
  "status": "ACTIVE",
  "created_at": "2026-09-08T12:00:00Z",
  "updated_at": "2026-09-08T12:00:00Z"
}
```

---

# 26. Project API

Base path:

```text
/api/v1/projects
```

---

# 27. Create Project

```http
POST /api/v1/projects
```

Authentication:

```text
Required
```

## Request

```json
{
  "name": "Trading Engine",
  "description": "High-performance trading simulation",
  "visibility": "PRIVATE"
}
```

## Response

```http
201 Created
```

```json
{
  "id": "uuid",
  "name": "Trading Engine",
  "description": "High-performance trading simulation",
  "visibility": "PRIVATE",
  "owner_id": "uuid",
  "created_at": "2026-09-08T12:00:00Z",
  "updated_at": "2026-09-08T12:00:00Z"
}
```

Errors:

```text
400 VALIDATION_ERROR
401 AUTH_REQUIRED
409 PROJECT_ALREADY_EXISTS
422 VALIDATION_ERROR
```

---

# 28. List Projects

```http
GET /api/v1/projects
```

Query parameters:

| Parameter  | Type    |    Default | Description       |
| ---------- | ------- | ---------: | ----------------- |
| page       | integer |          1 | Page number       |
| page_size  | integer |         20 | Number of results |
| cursor     | string  |       null | Cursor            |
| visibility | enum    |        all | PRIVATE/PUBLIC    |
| sort       | string  | updated_at | Sort field        |
| order      | enum    |       desc | asc/desc          |

## Response

```http
200 OK
```

```json
{
  "items": [
    {
      "id": "uuid",
      "name": "Trading Engine",
      "description": "Trading simulation",
      "visibility": "PRIVATE",
      "owner_id": "uuid",
      "created_at": "2026-09-08T12:00:00Z",
      "updated_at": "2026-09-08T12:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total": 42,
    "has_next": true
  }
}
```

---

# 29. Get Project

```http
GET /api/v1/projects/{project_id}
```

Path parameter:

```text
project_id: UUID
```

## Response

```json
{
  "id": "uuid",
  "name": "Trading Engine",
  "description": "Trading simulation",
  "visibility": "PRIVATE",
  "owner_id": "uuid",
  "created_at": "2026-09-08T12:00:00Z",
  "updated_at": "2026-09-08T12:00:00Z"
}
```

Errors:

```text
401 AUTH_REQUIRED
403 PROJECT_ACCESS_DENIED
404 PROJECT_NOT_FOUND
```

---

# 30. Update Project

```http
PATCH /api/v1/projects/{project_id}
```

Authentication:

```text
Required
```

Authorization:

```text
OWNER
```

## Request

All fields are optional.

```json
{
  "name": "Updated Project Name",
  "description": "Updated description",
  "visibility": "PUBLIC"
}
```

## Response

```http
200 OK
```

Returns the complete updated project.

---

# 31. Delete Project

```http
DELETE /api/v1/projects/{project_id}
```

Authorization:

```text
OWNER
```

## Response

```http
204 No Content
```

The database must enforce project ownership and dependent-resource deletion policy.

---

# 32. Project Member API

Base path:

```text
/api/v1/projects/{project_id}/members
```

---

# 33. List Members

```http
GET /api/v1/projects/{project_id}/members
```

## Response

```json
{
  "items": [
    {
      "user_id": "uuid",
      "username": "developer",
      "display_name": "Developer",
      "role": "OWNER",
      "joined_at": "2026-09-08T12:00:00Z"
    },
    {
      "user_id": "uuid",
      "username": "teammate",
      "display_name": "Teammate",
      "role": "EDITOR",
      "joined_at": "2026-09-08T12:10:00Z"
    }
  ]
}
```

---

# 34. Add Member

```http
POST /api/v1/projects/{project_id}/members
```

Authorization:

```text
OWNER
```

## Request

```json
{
  "user_id": "uuid",
  "role": "EDITOR"
}
```

Schema:

| Field   | Type | Required |
| ------- | ---- | -------: |
| user_id | UUID |      Yes |
| role    | enum |      Yes |

Allowed roles:

```text
OWNER
EDITOR
VIEWER
```

## Response

```http
201 Created
```

```json
{
  "user_id": "uuid",
  "username": "teammate",
  "display_name": "Teammate",
  "role": "EDITOR",
  "joined_at": "2026-09-08T12:00:00Z"
}
```

Errors:

```text
404 USER_NOT_FOUND
409 MEMBER_ALREADY_EXISTS
422 INVALID_ROLE
```

---

# 35. Update Member Role

```http
PATCH /api/v1/projects/{project_id}/members/{user_id}
```

Authorization:

```text
OWNER
```

## Request

```json
{
  "role": "VIEWER"
}
```

## Response

```json
{
  "user_id": "uuid",
  "role": "VIEWER"
}
```

The service must prevent invalid ownership states.

---

# 36. Remove Member

```http
DELETE /api/v1/projects/{project_id}/members/{user_id}
```

Authorization:

```text
OWNER
```

## Response

```http
204 No Content
```

Removing the project owner without transferring ownership must be rejected.

---

# 37. File API

Base path:

```text
/api/v1/projects/{project_id}/files
```

Supported operations:

```text
Create file
Create directory
List files
Read file
Update file
Delete file
Move file
Rename file
```

---

# 38. Create File

```http
POST /api/v1/projects/{project_id}/files
```

Authorization:

```text
OWNER / EDITOR
```

## Request

```json
{
  "name": "main.cpp",
  "parent_id": "uuid",
  "type": "FILE",
  "content": "#include <iostream>\nint main() {}"
}
```

Schema:

| Field     | Type      | Required | Constraints    |
| --------- | --------- | -------: | -------------- |
| name      | string    |      Yes | Valid filename |
| parent_id | UUID/null |      Yes | Same project   |
| type      | enum      |      Yes | FILE           |
| content   | string    |      Yes | Max 1 MB       |

## Response

```http
201 Created
```

```json
{
  "id": "uuid",
  "project_id": "uuid",
  "parent_id": "uuid",
  "name": "main.cpp",
  "path": "src/main.cpp",
  "type": "FILE",
  "size_bytes": 35,
  "revision": 1,
  "created_at": "2026-09-08T12:00:00Z",
  "updated_at": "2026-09-08T12:00:00Z"
}
```

---

# 39. Create Directory

```http
POST /api/v1/projects/{project_id}/files
```

## Request

```json
{
  "name": "src",
  "parent_id": null,
  "type": "DIRECTORY"
}
```

Content must not be supplied.

## Response

```http
201 Created
```

```json
{
  "id": "uuid",
  "project_id": "uuid",
  "parent_id": null,
  "name": "src",
  "path": "src",
  "type": "DIRECTORY",
  "size_bytes": 0
}
```

---

# 40. List Files

```http
GET /api/v1/projects/{project_id}/files
```

Query parameters:

```text
parent_id
recursive
cursor
limit
sort
order
```

Example:

```text
GET /api/v1/projects/{project_id}/files?recursive=true
```

## Response

```json
{
  "items": [
    {
      "id": "uuid",
      "parent_id": null,
      "name": "src",
      "path": "src",
      "type": "DIRECTORY",
      "size_bytes": 0
    },
    {
      "id": "uuid",
      "parent_id": "uuid",
      "name": "main.cpp",
      "path": "src/main.cpp",
      "type": "FILE",
      "size_bytes": 1024
    }
  ],
  "pagination": {
    "next_cursor": null,
    "has_next": false
  }
}
```

---

# 41. Read File

```http
GET /api/v1/projects/{project_id}/files/{file_id}
```

## Response

```json
{
  "id": "uuid",
  "project_id": "uuid",
  "parent_id": "uuid",
  "name": "main.cpp",
  "path": "src/main.cpp",
  "type": "FILE",
  "content": "#include <iostream>\nint main() {}",
  "size_bytes": 35,
  "revision": 42,
  "created_at": "2026-09-08T12:00:00Z",
  "updated_at": "2026-09-08T12:30:00Z"
}
```

Directories should not return file content.

---

# 42. Update File

```http
PATCH /api/v1/projects/{project_id}/files/{file_id}
```

Authorization:

```text
OWNER / EDITOR
```

## Request

```json
{
  "content": "#include <iostream>\nint main() {\n    return 0;\n}",
  "revision": 42
}
```

Schema:

| Field    | Type    | Required | Constraints      |
| -------- | ------- | -------: | ---------------- |
| content  | string  |      Yes | Max 1 MB         |
| revision | integer |      Yes | Current revision |

## Success

```http
200 OK
```

```json
{
  "id": "uuid",
  "revision": 43,
  "updated_at": "2026-09-08T12:31:00Z"
}
```

If the revision is stale:

```http
409 Conflict
```

---

# 43. Delete File

```http
DELETE /api/v1/projects/{project_id}/files/{file_id}
```

Authorization:

```text
OWNER / EDITOR
```

## Response

```http
204 No Content
```

For directories, deletion behavior must explicitly specify whether descendants are recursively deleted.

---

# 44. Move File

```http
PATCH /api/v1/projects/{project_id}/files/{file_id}
```

Request:

```json
{
  "parent_id": "uuid",
  "revision": 42
}
```

The server recalculates the resulting path.

Cross-project movement is forbidden.

---

# 45. Rename File

```http
PATCH /api/v1/projects/{project_id}/files/{file_id}
```

Request:

```json
{
  "name": "app.cpp",
  "revision": 42
}
```

The resulting path is recalculated by the server.

---

# 46. Execution API

Execution is asynchronous.

Base endpoint:

```text
/api/v1/executions
```

Execution always uses an immutable snapshot.

Core rule:

> **An execution runs against an immutable snapshot, never against mutable live project state.**

---

# 47. Submit Execution

```http
POST /api/v1/executions
```

Required headers:

```http
Authorization: Bearer <token>
Content-Type: application/json
X-Request-ID: <uuid>
Idempotency-Key: <unique-key>
```

## Request Schema

```json
{
  "project_id": "uuid",
  "entry_file_id": "uuid",
  "language": "cpp",
  "language_version": "17",
  "resource_policy": {
    "cpu_time_ms": 2000,
    "wall_time_ms": 5000,
    "memory_bytes": 268435456,
    "max_processes": 32,
    "max_output_bytes": 1048576,
    "max_disk_bytes": 10485760,
    "network_enabled": false
  }
}
```

Required fields:

```text
project_id
entry_file_id
language
language_version
```

`resource_policy` may be optional if platform defaults are used.

---

# 48. Execution Request Validation

The gateway validates:

```text
project exists
user is authorized
entry file exists
entry file belongs to project
entry file is a regular file
language is supported
language version is supported
project size is within limit
file paths are valid
snapshot size is within limit
resource policy is within platform bounds
```

---

# 49. Execution Submission Transaction

The service performs:

```text
1. Authenticate
2. Authorize
3. Validate project
4. Validate entry file
5. Read project files
6. Validate paths and limits
7. Capture immutable snapshot
8. Calculate file hashes
9. Calculate snapshot hash
10. Create execution_job
11. Create outbox event where enabled
12. Commit transaction
13. Dispatch/publish job
14. Return execution identifier
```

The worker must execute the persisted snapshot.

---

# 50. Submit Execution Response

```http
202 Accepted
```

```json
{
  "execution_id": "uuid",
  "job_id": "uuid",
  "status": "QUEUED",
  "snapshot_id": "uuid",
  "submitted_at": "2026-09-08T12:00:00Z"
}
```

---

# 51. Get Execution

```http
GET /api/v1/executions/{execution_id}
```

## Response

```json
{
  "id": "uuid",
  "job_id": "uuid",
  "project_id": "uuid",
  "user_id": "uuid",
  "entry_file_id": "uuid",
  "language": "cpp",
  "language_version": "17",
  "status": "RUNNING",
  "snapshot_id": "uuid",
  "submitted_at": "2026-09-08T12:00:00Z",
  "started_at": "2026-09-08T12:00:02Z",
  "completed_at": null
}
```

---

# 52. Get Execution Result

```http
GET /api/v1/executions/{execution_id}/result
```

## Response

```http
200 OK
```

```json
{
  "id": "uuid",
  "execution_id": "uuid",
  "stdout": "Hello CodeForge\n",
  "stderr": "",
  "exit_code": 0,
  "signal": null,
  "compile_time_ms": 120,
  "execution_time_ms": 18,
  "memory_bytes": 1245184,
  "output_truncated": false,
  "resource_violation": false,
  "created_at": "2026-09-08T12:00:04Z"
}
```

If no final result exists:

```http
409 Conflict
```

with:

```json
{
  "error": {
    "code": "EXECUTION_NOT_COMPLETED",
    "message": "The execution has not completed.",
    "request_id": "uuid",
    "details": {
      "status": "RUNNING"
    }
  }
}
```

---

# 53. Cancel Execution

```http
POST /api/v1/executions/{execution_id}/cancel
```

## Response

```http
200 OK
```

```json
{
  "execution_id": "uuid",
  "status": "CANCELLED"
}
```

Cancellation is idempotent.

If the execution is already terminal, the existing terminal state is returned.

---

# 54. Execution History

```http
GET /api/v1/projects/{project_id}/executions
```

Query parameters:

| Parameter | Type            |
| --------- | --------------- |
| status    | ExecutionStatus |
| user_id   | UUID            |
| language  | string          |
| from      | timestamp       |
| to        | timestamp       |
| cursor    | string          |
| limit     | integer         |
| sort      | string          |
| order     | asc/desc        |

## Response

```json
{
  "items": [
    {
      "id": "uuid",
      "project_id": "uuid",
      "language": "cpp",
      "status": "COMPLETED",
      "submitted_at": "2026-09-08T12:00:00Z",
      "completed_at": "2026-09-08T12:00:03Z"
    }
  ],
  "pagination": {
    "next_cursor": null,
    "has_next": false
  }
}
```

---

# 55. Execution Snapshot Metadata

```http
GET /api/v1/executions/{execution_id}/snapshot
```

## Response

```json
{
  "snapshot_id": "uuid",
  "project_id": "uuid",
  "snapshot_sha256": "sha256...",
  "entry_file": "src/main.cpp",
  "language": "cpp",
  "language_version": "17",
  "created_at": "2026-09-08T12:00:00Z"
}
```

Full source snapshot retrieval should require explicit authorization.

---

# 56. Snapshot File Schema

When source snapshot content is authorized:

```json
{
  "path": "src/main.cpp",
  "content": "#include <iostream>",
  "size_bytes": 20,
  "sha256": "sha256..."
}
```

The snapshot must not be mutable through the API.

---

# 57. Execution Snapshot Lifecycle

```text
LIVE PROJECT
     │
     │ Run requested
     ▼
SNAPSHOT CAPTURE
     ▼
SNAPSHOT VALIDATED
     ▼
SNAPSHOT PERSISTED
     ▼
SNAPSHOT QUEUED
     ▼
SNAPSHOT DISPATCHED
     ▼
SANDBOX MATERIALIZED
     ▼
COMPILE
     ▼
EXECUTE
     ▼
RESULT CAPTURED
     ▼
EXECUTION FINALIZED
     ▼
RETENTION / EXPIRATION
     ▼
PURGED
```

---

# 58. Execution State Machine

Normal flow:

```text
QUEUED
  ↓
STARTING
  ↓
COMPILING
  ↓
RUNNING
  ↓
COMPLETED
```

Failure paths:

```text
QUEUED ─────────────→ CANCELLED

STARTING ───────────→ FAILED

COMPILING ──────────→ FAILED
COMPILING ──────────→ TIMEOUT
COMPILING ──────────→ RESOURCE_LIMIT

RUNNING ────────────→ COMPLETED
RUNNING ────────────→ FAILED
RUNNING ────────────→ TIMEOUT
RUNNING ────────────→ RESOURCE_LIMIT
RUNNING ────────────→ CANCELLED
```

Invalid transitions must be rejected.

---

# 59. Collaboration API

WebSocket endpoint:

```text
/ws
```

Transport:

```text
WebSocket / Socket.IO
```

Authentication is performed during connection establishment.

---

# 60. Collaboration Connection Schema

Client connection metadata:

```json
{
  "token": "access-token",
  "project_id": "uuid"
}
```

The token must be transmitted using the secure authentication mechanism selected by the gateway.

The client joins:

```text
project:{project_id}
```

---

# 61. Project Join Event

```text
project:join
```

Payload:

```json
{
  "project_id": "uuid",
  "user_id": "uuid"
}
```

Server response:

```json
{
  "event": "project:joined",
  "project_id": "uuid",
  "user_id": "uuid"
}
```

---

# 62. Project Leave Event

```text
project:leave
```

Payload:

```json
{
  "project_id": "uuid"
}
```

---

# 63. File Open Event

```text
file:open
```

Payload:

```json
{
  "project_id": "uuid",
  "file_id": "uuid"
}
```

Server may return:

```json
{
  "event": "file:opened",
  "file": {
    "id": "uuid",
    "path": "src/main.cpp",
    "content": "#include <iostream>",
    "revision": 42
  }
}
```

---

# 64. File Update Event

```text
file:update
```

Payload:

```json
{
  "project_id": "uuid",
  "file_id": "uuid",
  "operation_id": "uuid",
  "user_id": "uuid",
  "base_revision": 42,
  "changes": [
    {
      "type": "insert",
      "position": 120,
      "text": "std::cout"
    }
  ]
}
```

Server response:

```json
{
  "event": "file:updated",
  "project_id": "uuid",
  "file_id": "uuid",
  "revision": 43,
  "operation_id": "uuid"
}
```

---

# 65. File Create Event

```text
file:create
```

Payload:

```json
{
  "project_id": "uuid",
  "parent_id": "uuid",
  "name": "utils.cpp",
  "type": "FILE",
  "content": ""
}
```

---

# 66. File Delete Event

```text
file:delete
```

Payload:

```json
{
  "project_id": "uuid",
  "file_id": "uuid",
  "revision": 43
}
```

---

# 67. File Move Event

```text
file:move
```

Payload:

```json
{
  "project_id": "uuid",
  "file_id": "uuid",
  "parent_id": "uuid",
  "revision": 43
}
```

---

# 68. Cursor Update Event

```text
cursor:update
```

Payload:

```json
{
  "project_id": "uuid",
  "file_id": "uuid",
  "user_id": "uuid",
  "position": {
    "line": 12,
    "column": 8
  }
}
```

Cursor state is transient.

---

# 69. Selection Update Event

```text
selection:update
```

Payload:

```json
{
  "project_id": "uuid",
  "file_id": "uuid",
  "user_id": "uuid",
  "selection": {
    "start": {
      "line": 12,
      "column": 4
    },
    "end": {
      "line": 12,
      "column": 20
    }
  }
}
```

---

# 70. Presence Join Event

```text
presence:join
```

Payload:

```json
{
  "project_id": "uuid",
  "user_id": "uuid",
  "status": "ACTIVE"
}
```

---

# 71. Presence Update Event

```text
presence:update
```

Payload:

```json
{
  "project_id": "uuid",
  "user_id": "uuid",
  "status": "ACTIVE",
  "file_id": "uuid"
}
```

Presence is transient and should normally use Redis.

---

# 72. Presence Leave Event

```text
presence:leave
```

Payload:

```json
{
  "project_id": "uuid",
  "user_id": "uuid"
}
```

---

# 73. Collaboration Revision Model

Each document maintains a logical revision.

```text
Revision 40
    ↓
User A edit
    ↓
Revision 41
    ↓
User B edit
    ↓
Revision 42
```

A stale client must synchronize before applying an incompatible operation.

Example:

```json
{
  "base_revision": 41,
  "changes": []
}
```

Server revision:

```text
42
```

Result:

```text
REVISION_CONFLICT
```

The long-term architecture should use CRDT semantics.

---

# 74. WebSocket Execution Events

Execution events:

```text
execution:queued
execution:starting
execution:compiling
execution:running
execution:completed
execution:failed
execution:timeout
execution:cancelled
execution:resource_limit
```

Common payload:

```json
{
  "event": "execution:running",
  "execution_id": "uuid",
  "project_id": "uuid",
  "status": "RUNNING",
  "timestamp": "2026-09-08T12:00:03Z"
}
```

---

# 75. Execution Completed Event

```json
{
  "event": "execution:completed",
  "execution_id": "uuid",
  "project_id": "uuid",
  "status": "COMPLETED",
  "result": {
    "exit_code": 0,
    "stdout": "Hello\n",
    "stderr": "",
    "execution_time_ms": 18
  },
  "timestamp": "2026-09-08T12:00:04Z"
}
```

---

# 76. Execution Failed Event

```json
{
  "event": "execution:failed",
  "execution_id": "uuid",
  "project_id": "uuid",
  "status": "FAILED",
  "error_code": "COMPILATION_ERROR",
  "message": "Compilation failed.",
  "timestamp": "2026-09-08T12:00:03Z"
}
```

---

# 77. Execution Timeout Event

```json
{
  "event": "execution:timeout",
  "execution_id": "uuid",
  "status": "TIMEOUT",
  "timestamp": "2026-09-08T12:00:10Z"
}
```

---

# 78. Resource Limit Event

```json
{
  "event": "execution:resource_limit",
  "execution_id": "uuid",
  "status": "RESOURCE_LIMIT",
  "resource": "MEMORY",
  "limit": 268435456,
  "timestamp": "2026-09-08T12:00:05Z"
}
```

Allowed resource identifiers:

```text
CPU
MEMORY
WALL_TIME
PROCESSES
OUTPUT
DISK
```

---

# 79. Standard WebSocket Event Envelope

All events should follow a consistent envelope:

```json
{
  "event": "file:update",
  "event_id": "uuid",
  "project_id": "uuid",
  "user_id": "uuid",
  "timestamp": "2026-09-08T12:00:00Z",
  "payload": {}
}
```

Required:

```text
event
event_id
timestamp
```

Project and user identifiers are required where applicable.

---

# 80. API Error Codes

Authentication:

```text
AUTH_REQUIRED
INVALID_TOKEN
TOKEN_EXPIRED
INVALID_CREDENTIALS
USER_NOT_FOUND
USER_SUSPENDED
```

Projects:

```text
PROJECT_NOT_FOUND
PROJECT_ALREADY_EXISTS
PROJECT_ACCESS_DENIED
PROJECT_NAME_INVALID
```

Members:

```text
MEMBER_NOT_FOUND
MEMBER_ALREADY_EXISTS
INVALID_ROLE
OWNER_REQUIRED
```

Files:

```text
FILE_NOT_FOUND
FILE_ALREADY_EXISTS
FILE_PATH_INVALID
FILE_NAME_INVALID
FILE_TOO_LARGE
PROJECT_SIZE_LIMIT
DIRECTORY_CONTENT_INVALID
REVISION_CONFLICT
```

Execution:

```text
EXECUTION_NOT_FOUND
EXECUTION_NOT_ALLOWED
EXECUTION_NOT_COMPLETED
EXECUTION_ALREADY_COMPLETED
EXECUTION_CANNOT_BE_CANCELLED
LANGUAGE_NOT_SUPPORTED
LANGUAGE_VERSION_NOT_SUPPORTED
RESOURCE_POLICY_INVALID
SNAPSHOT_INVALID
SNAPSHOT_NOT_FOUND
```

Infrastructure:

```text
RATE_LIMITED
SERVICE_UNAVAILABLE
UPSTREAM_TIMEOUT
INTERNAL_ERROR
```

---

# 81. HTTP Status Codes

| Status | Meaning                              |
| -----: | ------------------------------------ |
|    200 | Successful request                   |
|    201 | Resource created                     |
|    202 | Accepted for asynchronous processing |
|    204 | Successful request without body      |
|    400 | Malformed/invalid request            |
|    401 | Authentication required              |
|    403 | Authorization failure                |
|    404 | Resource not found                   |
|    409 | Resource/state conflict              |
|    413 | Payload too large                    |
|    422 | Semantic validation failure          |
|    429 | Rate limit exceeded                  |
|    500 | Internal server error                |
|    502 | Upstream service failure             |
|    503 | Service unavailable                  |
|    504 | Upstream timeout                     |

---

# 82. Validation Error Schema

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed.",
    "request_id": "uuid",
    "details": {
      "fields": {
        "name": "Project name is required.",
        "visibility": "Must be PRIVATE or PUBLIC."
      }
    }
  }
}
```

---

# 83. Pagination Schema

## Cursor Request

```text
GET /api/v1/projects?limit=20&cursor=abc123
```

## Cursor Response

```json
{
  "items": [],
  "pagination": {
    "next_cursor": "xyz789",
    "has_next": true
  }
}
```

Limits:

```text
minimum: 1
default: 20
maximum: 100
```

---

# 84. Sorting Schema

Supported query parameters:

```text
sort
order
```

Example:

```text
?sort=updated_at&order=desc
```

Allowed sort fields must be whitelisted.

Arbitrary SQL expressions must never be accepted.

---

# 85. Rate Limiting

Rate limiting is enforced by the Node Gateway.

Recommended categories:

```text
Authentication:
    strict

Project API:
    moderate

File API:
    controlled high frequency

Execution submission:
    strict

WebSocket messages:
    per connection
```

Redis key example:

```text
codeforge:ratelimit:user:{user_id}:executions
```

Rate-limit response:

```http
429 Too Many Requests
Retry-After: 30
```

Example:

```json
{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Too many requests.",
    "request_id": "uuid",
    "details": {
      "retry_after_seconds": 30
    }
  }
}
```

---

# 86. Execution Idempotency

Execution submission requires:

```http
Idempotency-Key: <unique-key>
```

Example:

```text
Client
  │
  │ POST execution
  │ Idempotency-Key=A
  ▼
Gateway
  │
  ▼
Execution created
  │
  X network failure
  │
  ▼
Client retries
  │
  │ Idempotency-Key=A
  ▼
Gateway
  │
  └── returns original execution
```

The same idempotency key must not create multiple execution jobs for the same user and endpoint.

---

# 87. Idempotency Response

For a repeated request:

```http
200 OK
```

or the same semantic response originally returned.

The API must not create a second execution.

The server should persist:

```text
user_id
endpoint
idempotency_key
request_hash
execution_id
response_metadata
created_at
expires_at
```

---

# 88. Idempotency Conflict

If the same key is reused with a different request payload:

```http
409 Conflict
```

Example:

```json
{
  "error": {
    "code": "IDEMPOTENCY_KEY_REUSED",
    "message": "The idempotency key was previously used with a different request.",
    "request_id": "uuid",
    "details": {}
  }
}
```

---

# 89. Internal Service API

Internal services communicate using gRPC.

```text
Node Gateway
      │
      │ gRPC
      ▼
Python Evaluator
      │
      │ gRPC
      ▼
C++ Sandbox
```

---

# 90. Evaluator gRPC Service

Conceptual contract:

```protobuf
service EvaluatorService {
    rpc SubmitExecution(ExecutionRequest)
        returns (ExecutionResponse);

    rpc GetExecutionStatus(ExecutionStatusRequest)
        returns (ExecutionStatusResponse);

    rpc CancelExecution(CancelExecutionRequest)
        returns (CancelExecutionResponse);
}
```

---

# 91. Common gRPC Types

```protobuf
message ResourcePolicy {
    int64 cpu_time_ms = 1;
    int64 wall_time_ms = 2;
    int64 memory_bytes = 3;
    int32 max_processes = 4;
    int64 max_output_bytes = 5;
    int64 max_disk_bytes = 6;
    bool network_enabled = 7;
}
```

---

# 92. Execution Request

```protobuf
message ExecutionRequest {
    string request_id = 1;
    string execution_id = 2;
    string job_id = 3;
    string project_id = 4;
    string user_id = 5;
    string snapshot_id = 6;
    string entry_file = 7;
    string language = 8;
    string language_version = 9;
    ResourcePolicy resource_policy = 10;
}
```

The evaluator must not trust browser-provided authorization decisions.

Authorization has already been performed at the gateway, while internal service identity and contract validation are still required.

---

# 93. Execution Response

```protobuf
message ExecutionResponse {
    string execution_id = 1;
    string job_id = 2;
    string status = 3;
    string message = 4;
}
```

Example:

```json
{
  "execution_id": "uuid",
  "job_id": "uuid",
  "status": "QUEUED",
  "message": "Execution accepted."
}
```

---

# 94. Execution Status Request

```protobuf
message ExecutionStatusRequest {
    string request_id = 1;
    string execution_id = 2;
}
```

---

# 95. Execution Status Response

```protobuf
message ExecutionStatusResponse {
    string execution_id = 1;
    string status = 2;
    int64 started_at_unix_ms = 3;
    int64 completed_at_unix_ms = 4;
}
```

Timestamps may be represented using protobuf `google.protobuf.Timestamp` in the production contract.

---

# 96. Cancel Execution Request

```protobuf
message CancelExecutionRequest {
    string request_id = 1;
    string execution_id = 2;
}
```

---

# 97. Cancel Execution Response

```protobuf
message CancelExecutionResponse {
    string execution_id = 1;
    string status = 2;
}
```

---

# 98. Sandbox gRPC Service

```protobuf
service SandboxService {
    rpc Execute(SandboxExecutionRequest)
        returns (SandboxExecutionResponse);

    rpc CancelExecution(SandboxCancelRequest)
        returns (SandboxCancelResponse);
}
```

---

# 99. Sandbox Execution Request

```protobuf
message SandboxExecutionRequest {
    string request_id = 1;
    string execution_id = 2;
    string snapshot_id = 3;
    string entry_file = 4;
    string language = 5;
    string language_version = 6;
    ResourcePolicy resource_policy = 7;
    repeated SnapshotFile files = 8;
}
```

---

# 100. Snapshot File gRPC Message

```protobuf
message SnapshotFile {
    string path = 1;
    bytes content = 2;
    int64 size_bytes = 3;
    string sha256 = 4;
}
```

The sandbox validates:

```text
path
size
hash
file count
total snapshot size
```

before materialization.

---

# 101. Sandbox Execution Response

```protobuf
message SandboxExecutionResponse {
    string execution_id = 1;
    string status = 2;
    string stdout = 3;
    string stderr = 4;
    int32 exit_code = 5;
    int32 signal = 6;
    int64 compile_time_ms = 7;
    int64 execution_time_ms = 8;
    int64 memory_bytes = 9;
    bool output_truncated = 10;
    bool resource_violation = 11;
}
```

---

# 102. Sandbox Cancel Request

```protobuf
message SandboxCancelRequest {
    string request_id = 1;
    string execution_id = 2;
}
```

---

# 103. Sandbox Cancel Response

```protobuf
message SandboxCancelResponse {
    string execution_id = 1;
    string status = 2;
}
```

---

# 104. gRPC Metadata

Internal calls carry:

```text
x-request-id
x-execution-id
x-trace-id
x-service-identity
```

Example:

```text
x-request-id: 8e91...
x-execution-id: 550e...
x-trace-id: 9ab2...
x-service-identity: evaluator
```

---

# 105. gRPC Security

Internal gRPC must not be considered trusted simply because services share a Docker network.

Production deployment should use:

```text
TLS
+
service identity
+
authorization
```

The sandbox accepts execution requests only from an authorized evaluator identity.

---

# 106. Resource Policy Enforcement

The client may request resource settings only within platform limits.

Example:

```json
{
  "memory_bytes": 8589934592
}
```

If the platform maximum is:

```text
268435456 bytes
```

the request must be rejected.

The client cannot override:

```text
CPU limit
memory limit
wall-clock timeout
process limit
disk limit
output limit
network policy
```

---

# 107. File and Project Limits

Initial limits:

```text
Maximum individual file:
    1 MB

Maximum project:
    10 MB
```

Additional limits:

```text
Maximum file count
Maximum directory depth
Maximum path length
Maximum request body
Maximum snapshot size
```

All limits are configurable platform policies.

---

# 108. Execution Request Limits

The execution API enforces:

```text
Maximum source snapshot size
Maximum file count
Maximum path depth
Maximum execution frequency
Maximum concurrent executions
Maximum resource policy
```

---

# 109. API Authentication and Authorization Matrix

| Operation            | Anonymous | Viewer | Editor | Owner |
| -------------------- | --------: | -----: | -----: | ----: |
| View public project  |         ✓ |      ✓ |      ✓ |     ✓ |
| View private project |         ✗ |      ✓ |      ✓ |     ✓ |
| Read files           |         ✗ |      ✓ |      ✓ |     ✓ |
| Modify files         |         ✗ |      ✗ |      ✓ |     ✓ |
| Run execution        |         ✗ |      ✗ |      ✓ |     ✓ |
| Cancel own execution |         ✗ |      ✗ |      ✓ |     ✓ |
| Manage members       |         ✗ |      ✗ |      ✗ |     ✓ |
| Delete project       |         ✗ |      ✗ |      ✗ |     ✓ |

Authorization must always be evaluated against current project membership.

---

# 110. API Security Boundary

```text
Browser
   │
   │ Untrusted
   ▼
Node Gateway
   │
   │ Authenticated
   ▼
Python Evaluator
   │
   │ Validated execution
   ▼
C++ Sandbox
```

The browser must never be allowed to:

```text
execute Docker commands
access Docker socket
choose arbitrary host paths
mount host directories
configure unrestricted resources
connect directly to sandbox
connect directly to PostgreSQL
connect directly to Redis
execute code on the gateway
```

---

# 111. API Transaction Boundary

Execution creation must use a database transaction.

```sql
BEGIN;

-- Validate project
-- Validate membership
-- Validate entry file
-- Read project files
-- Create immutable snapshot
-- Insert execution job
-- Insert outbox event

COMMIT;
```

Only committed jobs become eligible for dispatch.

---

# 112. Outbox Integration

Recommended architecture:

```text
Transaction
    │
    ├── execution_jobs
    │
    └── outbox_events
             │
             ▼
       Outbox Worker
             │
             ▼
           Redis
             │
             ▼
       Python Evaluator
```

This prevents:

```text
Database commit succeeds
+
Redis publish fails
=
lost execution
```

---

# 113. API Observability

Every important API request should generate structured telemetry.

Minimum fields:

```text
timestamp
request_id
trace_id
user_id
project_id
execution_id
route
method
status_code
latency_ms
service
error_code
```

Sensitive data must never be logged.

Never log:

```text
password
access_token
refresh_token
session_token
private secrets
```

Raw source code should not be logged by default.

---

# 114. API Metrics

Recommended metrics:

```text
http_requests_total
http_request_duration_seconds
http_errors_total

execution_submissions_total
execution_queue_latency_seconds
execution_duration_seconds
execution_failures_total
execution_timeouts_total
execution_resource_limit_total

websocket_connections
websocket_messages_total

grpc_requests_total
grpc_request_duration_seconds

rate_limit_rejections_total
```

---

# 115. Distributed Tracing

Tracing should follow:

```text
Browser
   │
   ▼
Node Gateway
   │
   ▼
Python Evaluator
   │
   ▼
C++ Sandbox
```

Trace context must propagate across internal service boundaries.

---

# 116. API Timeout Policy

Every network call must have an explicit timeout.

Example:

```text
Browser → Gateway:
    30 seconds

Gateway → Evaluator:
    10 seconds

Evaluator → Sandbox:
    bounded by execution policy

Database:
    short bounded timeout

Redis:
    short bounded timeout
```

Execution wall-clock limits are independent from HTTP/gRPC transport timeouts.

---

# 117. API Retry Policy

Retries are permitted only for operations known to be safe.

Safe examples:

```text
GET
status queries
idempotent internal operations
```

Execution creation must use:

```text
Idempotency-Key
```

The sandbox must never be blindly re-executed because of a network retry.

---

# 118. API Concurrency Control

File modifications support optimistic concurrency.

Example:

```http
If-Match: "revision-42"
```

or:

```json
{
  "content": "...",
  "revision": 42
}
```

If the server has revision 43:

```http
409 Conflict
```

The client must synchronize before overwriting newer content.

---

# 119. API Security Headers

The reverse proxy should provide appropriate headers:

```text
Strict-Transport-Security
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
```

CORS must use an explicit allowlist.

Authenticated production APIs must not use unrestricted wildcard origins.

---

# 120. API Repository Mapping

```text
gateway/
└── node/
    ├── src/
    │   ├── routes/
    │   │   ├── auth/
    │   │   ├── users/
    │   │   ├── projects/
    │   │   ├── members/
    │   │   ├── files/
    │   │   └── executions/
    │   │
    │   ├── websocket/
    │   ├── middleware/
    │   ├── services/
    │   ├── repositories/
    │   ├── validators/
    │   └── grpc/
    │
    └── tests/

proto/
├── execution.proto
├── evaluator.proto
├── sandbox.proto
└── collaboration.proto
```

---

# 121. Recommended API Module Boundaries

```text
AuthController
      │
      ▼
AuthService
      │
      ▼
SessionRepository

ProjectController
      │
      ▼
ProjectService
      │
      ▼
ProjectRepository

MemberController
      │
      ▼
MembershipService
      │
      ▼
MembershipRepository

FileController
      │
      ▼
FileService
      │
      ▼
FileRepository

ExecutionController
      │
      ▼
ExecutionService
      │
      ├── SnapshotService
      ├── ResourcePolicyService
      ├── QueueService
      └── EvaluatorClient
```

Controllers must remain thin.

Business rules belong in services.

Database access belongs in repositories.

---

# 122. API Contract Ownership

```text
Node Gateway
    │
    └── Public REST/WebSocket contract

Python Evaluator
    │
    └── Evaluator gRPC contract

C++ Sandbox
    │
    └── Sandbox gRPC contract

proto/
    │
    └── Shared service contracts
```

---

# 123. API Compatibility Rules

REST:

```text
Additive change
    ↓
Backward compatible

Breaking change
    ↓
New API version
```

Protocol Buffers:

```text
Field numbers must never be reused.
```

Removed fields should be reserved.

Example:

```protobuf
message ExecutionRequest {
    reserved 8;
    string execution_id = 1;
}
```

---

# 124. API Failure Model

The system must fail closed.

Example:

```text
Evaluator unavailable
       │
       ▼
POST /executions
       │
       ▼
503 Service Unavailable
```

The gateway must never:

```text
execute locally
bypass the evaluator
bypass the sandbox
execute on the host
fall back to unsafe execution
```

---

# 125. Complete Execution API Flow

```text
User
 │
 │ POST /api/v1/executions
 ▼
Node Gateway
 │
 ├── Authenticate
 ├── Authorize
 ├── Validate
 │
 ├── Read project files
 │
 ├── Create immutable snapshot
 │
 ├── Calculate snapshot hash
 │
 ├── INSERT execution_job
 │
 └── COMMIT
       │
       ▼
    Outbox
       │
       ▼
     Redis
       │
       ▼
 Python Evaluator
       │
       ├── Validate execution
       ├── Load snapshot
       └── Dispatch
              │
              ▼
       C++ Sandbox Runtime
              │
              ▼
       Ephemeral Container
              │
              ├── Compile
              ├── Execute
              └── Collect result
                     │
                     ▼
              Execution Result
                     │
                     ▼
              PostgreSQL
                     │
                     ▼
              WebSocket Event
                     │
                     ▼
                 Browser
```

---

# 126. Example End-to-End API Sequence

## Step 1 — Register

```http
POST /api/v1/auth/register
```

## Step 2 — Login

```http
POST /api/v1/auth/login
```

## Step 3 — Create project

```http
POST /api/v1/projects
```

## Step 4 — Create directory

```http
POST /api/v1/projects/{project_id}/files
```

## Step 5 — Create file

```http
POST /api/v1/projects/{project_id}/files
```

## Step 6 — Edit file

```http
PATCH /api/v1/projects/{project_id}/files/{file_id}
```

## Step 7 — Submit execution

```http
POST /api/v1/executions
```

## Step 8 — Receive event

```text
execution:queued
```

## Step 9 — Execution progresses

```text
execution:starting
execution:compiling
execution:running
```

## Step 10 — Receive result

```text
execution:completed
```

## Step 11 — Fetch result

```http
GET /api/v1/executions/{execution_id}/result
```

---

# 127. Complete Endpoint Inventory

The public REST contract consists of:

## Authentication

```text
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
GET    /api/v1/users/me
```

## Projects

```text
POST   /api/v1/projects
GET    /api/v1/projects
GET    /api/v1/projects/{project_id}
PATCH  /api/v1/projects/{project_id}
DELETE /api/v1/projects/{project_id}
```

## Members

```text
GET    /api/v1/projects/{project_id}/members
POST   /api/v1/projects/{project_id}/members
PATCH  /api/v1/projects/{project_id}/members/{user_id}
DELETE /api/v1/projects/{project_id}/members/{user_id}
```

## Files

```text
POST   /api/v1/projects/{project_id}/files
GET    /api/v1/projects/{project_id}/files
GET    /api/v1/projects/{project_id}/files/{file_id}
PATCH  /api/v1/projects/{project_id}/files/{file_id}
DELETE /api/v1/projects/{project_id}/files/{file_id}
```

## Execution

```text
POST   /api/v1/executions
GET    /api/v1/executions/{execution_id}
GET    /api/v1/executions/{execution_id}/result
POST   /api/v1/executions/{execution_id}/cancel
GET    /api/v1/projects/{project_id}/executions
GET    /api/v1/executions/{execution_id}/snapshot
```

## Collaboration

```text
WebSocket /ws
```

---

# 128. Endpoint Contract Matrix

| Endpoint                        | Auth | Role       | Request Body           | Response         |     Async |
| ------------------------------- | ---- | ---------- | ---------------------- | ---------------- | --------: |
| POST `/auth/register`           | No   | —          | RegisterRequest        | AuthResponse     |        No |
| POST `/auth/login`              | No   | —          | LoginRequest           | AuthResponse     |        No |
| POST `/auth/logout`             | Yes  | Any        | None                   | 204              |        No |
| GET `/users/me`                 | Yes  | Any        | None                   | User             |        No |
| POST `/projects`                | Yes  | Any        | CreateProjectRequest   | Project          |        No |
| GET `/projects`                 | Yes  | Any        | None                   | ProjectList      |        No |
| GET `/projects/{id}`            | Yes  | Member     | None                   | Project          |        No |
| PATCH `/projects/{id}`          | Yes  | Owner      | UpdateProjectRequest   | Project          |        No |
| DELETE `/projects/{id}`         | Yes  | Owner      | None                   | 204              |        No |
| GET `/members`                  | Yes  | Member     | None                   | MemberList       |        No |
| POST `/members`                 | Yes  | Owner      | AddMemberRequest       | Member           |        No |
| PATCH `/members/{id}`           | Yes  | Owner      | UpdateMemberRequest    | Member           |        No |
| DELETE `/members/{id}`          | Yes  | Owner      | None                   | 204              |        No |
| POST `/files`                   | Yes  | Editor     | CreateFileRequest      | File             |        No |
| GET `/files`                    | Yes  | Member     | None                   | FileList         |        No |
| GET `/files/{id}`               | Yes  | Member     | None                   | FileContent      |        No |
| PATCH `/files/{id}`             | Yes  | Editor     | UpdateFileRequest      | FileRevision     |        No |
| DELETE `/files/{id}`            | Yes  | Editor     | None                   | 204              |        No |
| POST `/executions`              | Yes  | Editor     | CreateExecutionRequest | Execution        |       Yes |
| GET `/executions/{id}`          | Yes  | Member     | None                   | Execution        |        No |
| GET `/executions/{id}/result`   | Yes  | Member     | None                   | ExecutionResult  |        No |
| POST `/executions/{id}/cancel`  | Yes  | Editor     | None                   | Execution        |        No |
| GET `/projects/{id}/executions` | Yes  | Member     | None                   | ExecutionList    |        No |
| GET `/executions/{id}/snapshot` | Yes  | Authorized | None                   | SnapshotMetadata |        No |
| WebSocket `/ws`                 | Yes  | Member     | Events                 | Events           | Real-time |

---

# 129. API Security Checklist

Every endpoint must verify:

```text
[ ] Authentication requirement
[ ] Authorization requirement
[ ] Input schema
[ ] Input size
[ ] UUID validation
[ ] Resource ownership
[ ] Project membership
[ ] Role permission
[ ] Database constraints
[ ] Rate limits
[ ] Audit requirements
[ ] Error mapping
[ ] Logging policy
[ ] Metrics
[ ] Timeout
[ ] Retry semantics
```

---

# 130. Execution Security Checklist

Every execution must verify:

```text
[ ] Authenticated user
[ ] Authorized project membership
[ ] Valid project
[ ] Valid entry file
[ ] Same-project file ownership
[ ] Valid file paths
[ ] Snapshot size
[ ] File count
[ ] Resource policy
[ ] Immutable snapshot
[ ] Snapshot hash
[ ] Sandbox isolation
[ ] Network policy
[ ] CPU limit
[ ] Memory limit
[ ] Process limit
[ ] Disk limit
[ ] Output limit
[ ] Wall-clock timeout
[ ] Result capture
[ ] Audit correlation
```

---

# 131. API Design Principles

CodeForge Cloud APIs follow:

1. **Versioned contracts**
2. **Explicit authorization**
3. **Consistent errors**
4. **Immutable execution snapshots**
5. **Idempotent asynchronous operations**
6. **Database-backed durability**
7. **Redis for transient state**
8. **gRPC for internal service communication**
9. **WebSocket for real-time events**
10. **No direct sandbox exposure**
11. **Fail closed**
12. **Observable by default**
13. **Contract-first development**
14. **Least privilege**
15. **Defense in depth**

---

# 132. Final API Architecture

```text
                         ┌─────────────────────────┐
                         │       React Web IDE     │
                         └────────────┬────────────┘
                                      │
                           HTTPS / WebSocket
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │     Nginx / Reverse     │
                         │         Proxy           │
                         └────────────┬────────────┘
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │    Node.js Gateway      │
                         │                         │
                         │ REST / WebSocket        │
                         │ Auth / AuthZ            │
                         │ Projects / Files        │
                         │ Execution API            │
                         └──────┬──────────┬───────┘
                                │          │
                         PostgreSQL       Redis
                                │          │
                                │          │
                                │     Queue / PubSub
                                │          │
                                │          ▼
                                │   ┌───────────────┐
                                │   │    Python     │
                                │   │   Evaluator   │
                                │   └───────┬───────┘
                                │           │
                                │          gRPC
                                │           │
                                │           ▼
                                │   ┌───────────────┐
                                │   │ C++ Sandbox   │
                                │   │    Runtime    │
                                │   └───────┬───────┘
                                │           │
                                │           ▼
                                │   ┌───────────────┐
                                │   │   Ephemeral   │
                                │   │   Container   │
                                │   └───────────────┘
                                │
                                ▼
                         Durable Execution
                              History
```

---

# 133. Contract-First Implementation Order

The recommended implementation order is:

```text
1. Define Proto contracts
        ↓
2. Define database schema
        ↓
3. Implement Gateway DTO schemas
        ↓
4. Implement validation
        ↓
5. Implement REST controllers
        ↓
6. Implement services
        ↓
7. Implement repositories
        ↓
8. Implement Evaluator gRPC client
        ↓
9. Implement Python Evaluator
        ↓
10. Implement Sandbox gRPC server
        ↓
11. Implement WebSocket events
        ↓
12. Add integration tests
        ↓
13. Add security tests
        ↓
14. Add load tests
```

---

# 134. API Testing Requirements

Every endpoint must have tests for:

```text
Happy path
Missing authentication
Invalid authentication
Insufficient authorization
Invalid UUID
Invalid JSON
Missing required field
Invalid enum
Oversized payload
Not found
Conflict
Rate limit
Database failure
Redis failure
Upstream failure
Timeout
```

Execution endpoints additionally require:

```text
Compilation success
Compilation failure
Runtime failure
Timeout
Memory violation
CPU violation
Process violation
Output violation
Cancellation
Duplicate submission
Snapshot integrity
Cross-project access
```

---

# 135. API Contract Definition of Done

An endpoint is considered complete only when:

```text
[ ] Endpoint path defined
[ ] HTTP method defined
[ ] Authentication defined
[ ] Authorization defined
[ ] Path parameters defined
[ ] Query parameters defined
[ ] Headers defined
[ ] Request schema defined
[ ] Response schema defined
[ ] Field types defined
[ ] Required fields defined
[ ] Validation constraints defined
[ ] Enum values defined
[ ] Success status defined
[ ] Error statuses defined
[ ] Error codes defined
[ ] Example request defined
[ ] Example response defined
[ ] Idempotency behavior defined where required
[ ] Concurrency behavior defined where required
[ ] Rate limit defined
[ ] Timeout defined
[ ] Retry behavior defined
[ ] Logging requirements defined
[ ] Metrics defined
[ ] Security reviewed
[ ] Automated tests implemented
```

---

# 136. API Documentation Traceability

This API document derives its contracts from:

```text
01-product-requirements.md
        │
        ▼
02-architecture.md
        │
        ▼
03-data-model.md
        │
        ▼
04-api-reference.md
```

Traceability:

| Requirement    | Architecture               | Data Model            | API                          |
| -------------- | -------------------------- | --------------------- | ---------------------------- |
| Authentication | Gateway Auth               | users/sessions        | Auth endpoints               |
| Projects       | Project Service            | projects              | Project endpoints            |
| Collaboration  | Collaboration Service      | project_members/files | WebSocket                    |
| Execution      | Evaluator/Sandbox          | execution_jobs        | Execution API                |
| Snapshot       | Snapshot architecture      | source_snapshot       | Snapshot API                 |
| Security       | Sandbox isolation          | constraints           | Auth/AuthZ/resource policies |
| Observability  | Observability architecture | audit_events          | Request IDs/metrics          |
| Reliability    | Outbox architecture        | outbox_events         | Idempotency/queue semantics  |

---

# 137. Final Contract Principles

The complete API contract is based on the following rules:

```text
Browser is untrusted.
Gateway is the public boundary.
Evaluator orchestrates execution.
Sandbox executes untrusted code.
PostgreSQL is durable source of truth.
Redis is transient infrastructure.
Snapshots are immutable.
Execution results are immutable.
Terminal execution states are immutable.
Authorization is explicit.
Resource limits are server-controlled.
Internal communication is authenticated.
Failures fail closed.
All important operations are observable.
```

The most important execution rule remains:

> **User code must never execute outside the isolated sandbox execution path.**

---

# 138. Next Document

The next document in the CodeForge Cloud documentation set is:

```text
docs/05-roadmap-and-phases.md
```

It will define the implementation progression from:

```text
Phase 1 — Hello Compiler
        ↓
Phase 2 — Remote Compiler
        ↓
Phase 3 — Online Compiler
        ↓
Phase 4 — Online IDE
        ↓
Phase 5 — Multi-user IDE
        ↓
Phase 6 — Collaborative IDE
        ↓
Phase 7 — Production-grade Execution Platform
```

The roadmap should map each phase to architecture, database, API, security, testing, deployment, and production-readiness gates.

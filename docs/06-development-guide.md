# CodeForge Cloud

## Development Guide

**Document:** 06 — Development Guide
**Project:** CodeForge Cloud
**Subtitle:** Collaborative Real-Time Online Code Compiler & IDE Sandbox
**Version:** 1.0
**Status:** Draft
**Author:** Adarsh Kumar

---

# 1. Document Overview

This document defines the development workflow for CodeForge Cloud.

It explains how developers should:

* prepare the development environment.
* clone and configure the repository.
* build individual services.
* run the complete platform locally.
* configure PostgreSQL and Redis.
* run database migrations.
* develop the React Web IDE.
* develop the Node.js Gateway.
* develop the Python Evaluator.
* develop the C++ Sandbox.
* work with Protocol Buffers and gRPC.
* run tests.
* debug services.
* inspect logs.
* perform security testing.
* create Git branches.
* submit changes.
* troubleshoot common problems.

The guide assumes the repository follows:

```text
Online-Code-Compiler/
├── docs/
├── frontend/
│   └── web-ide/
├── gateway/
│   └── node/
├── evaluator/
│   └── python/
├── sandbox/
│   └── cpp/
├── proto/
├── infrastructure/
│   ├── docker/
│   ├── postgres/
│   ├── redis/
│   └── nginx/
├── tests/
│   ├── integration/
│   ├── security/
│   └── load/
├── scripts/
├── docker-compose.yml
├── Makefile
├── .gitignore
├── LICENSE
└── README.md
```

---

# 2. Development Principles

All development should follow these principles:

1. Security by design.
2. Least privilege.
3. Separation of concerns.
4. Contract-first development.
5. Immutable execution snapshots.
6. Disposable sandbox environments.
7. Fail-closed behavior.
8. Observable services.
9. Automated testing.
10. Reproducible builds.
11. Explicit configuration.
12. Small, reviewable commits.

---

# 3. Technology Stack

## Frontend

```text
React
TypeScript
Vite
Monaco Editor
Tailwind CSS
Lucide
WebSocket / Socket.IO
```

## Gateway

```text
Node.js
TypeScript
Fastify
WebSocket / Socket.IO
REST API
gRPC client
```

## Evaluator

```text
Python
FastAPI
gRPC
Protocol Buffers
Redis
PostgreSQL client
```

## Sandbox

```text
C++
C++17
CMake
Docker
Linux namespaces
Linux cgroups
seccomp
```

## Infrastructure

```text
PostgreSQL
Redis
Docker
Docker Compose
Nginx
```

---

# 4. Required Development Tools

The developer workstation should provide:

```text
Git
Docker
Docker Compose
Node.js
npm
Python
pip
CMake
GCC / G++
Make
PostgreSQL client
Redis client
```

Recommended additional tools:

```text
curl
jq
grpcurl
protoc
clang-format
clang-tidy
pytest
ESLint
Prettier
```

---

# 5. Verify the Environment

Verify Git:

```bash
git --version
```

Verify Docker:

```bash
docker --version
```

Verify Docker Compose:

```bash
docker compose version
```

Verify Node.js:

```bash
node --version
```

Verify npm:

```bash
npm --version
```

Verify Python:

```bash
python3 --version
```

Verify CMake:

```bash
cmake --version
```

Verify compiler:

```bash
g++ --version
```

Verify Protocol Buffers:

```bash
protoc --version
```

---

# 6. Clone the Repository

Clone the repository:

```bash
git clone <repository-url>
cd Online-Code-Compiler
```

Verify the branch:

```bash
git branch
```

Verify the working tree:

```bash
git status
```

The expected initial state should be:

```text
On branch main
nothing to commit, working tree clean
```

---

# 7. Repository Structure

The repository is divided according to service ownership.

```text
frontend/
    Browser application

gateway/
    Public API and WebSocket boundary

evaluator/
    Execution orchestration

sandbox/
    Security-critical code execution

proto/
    Internal service contracts

infrastructure/
    Databases, cache, reverse proxy, containers

tests/
    Cross-service testing

scripts/
    Development and operational utilities

docs/
    Technical documentation
```

The browser must never communicate directly with the evaluator or sandbox.

---

# 8. Environment Configuration

Environment-specific configuration should never be hardcoded.

Recommended files:

```text
.env
.env.example
.env.development
.env.test
.env.production
```

Only `.env.example` should be committed if it contains no secrets.

Example:

```env
NODE_ENV=development

DATABASE_URL=postgresql://codeforge:password@localhost:5432/codeforge

REDIS_URL=redis://localhost:6379

API_PORT=3000

EVALUATOR_GRPC_HOST=localhost
EVALUATOR_GRPC_PORT=50051

SANDBOX_GRPC_HOST=localhost
SANDBOX_GRPC_PORT=50052
```

Production secrets must come from an appropriate secret-management mechanism.

---

# 9. Secret Management

Never commit:

```text
passwords
API keys
JWT secrets
private keys
database credentials
Redis credentials
TLS private keys
service credentials
```

Never place secrets inside:

```text
source code
Dockerfiles
Git history
logs
execution output
user source snapshots
```

Use:

```text
environment variables
secret managers
Docker secrets
Kubernetes Secrets
```

depending on the deployment environment.

---

# 10. Docker Development Environment

The development platform should be runnable using Docker Compose.

Start infrastructure:

```bash
docker compose up -d
```

Check containers:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs
```

Follow logs:

```bash
docker compose logs -f
```

Stop services:

```bash
docker compose down
```

Stop and remove volumes:

```bash
docker compose down -v
```

The `-v` option should only be used when intentionally deleting development data.

---

# 11. Recommended Docker Services

The initial Compose environment may contain:

```text
postgres
redis
nginx
gateway
evaluator
sandbox
frontend
```

Architecture:

```text
                  Nginx
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      Frontend             Gateway
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
                PostgreSQL            Redis
                    │
                    ▼
                Evaluator
                    │
                    ▼
                 Sandbox
```

---

# 12. PostgreSQL Development

Connect using the configured development credentials.

Example:

```bash
psql "$DATABASE_URL"
```

List databases:

```sql
\l
```

Connect to the CodeForge database:

```sql
\c codeforge
```

List tables:

```sql
\dt
```

Inspect a table:

```sql
\d users
```

---

# 13. Database Migration Workflow

Database schema changes must be performed through migrations.

Recommended structure:

```text
infrastructure/postgres/
└── migrations/
    ├── 001_users.sql
    ├── 002_projects.sql
    ├── 003_project_members.sql
    ├── 004_files.sql
    ├── 005_execution_jobs.sql
    ├── 006_execution_results.sql
    ├── 007_sessions.sql
    ├── 008_audit_events.sql
    └── 009_outbox_events.sql
```

Never modify an already-applied migration in a shared environment.

Instead:

```text
Existing migration
        ↓
New migration
        ↓
Schema update
```

---

# 14. Database Development Rules

All database changes must consider:

* primary keys.
* foreign keys.
* indexes.
* uniqueness.
* cascading behavior.
* transaction boundaries.
* concurrency.
* retention.
* migration rollback strategy.

Important project isolation rule:

```text
Project A files
      ✗
Project B execution
```

Cross-project references must be rejected by database constraints and application validation.

---

# 15. Redis Development

Redis is used for transient and high-speed data.

Primary responsibilities:

```text
execution queue
status cache
rate limiting
presence
collaboration pub/sub
temporary state
```

Redis is not the durable source of truth.

Check Redis:

```bash
redis-cli ping
```

Expected:

```text
PONG
```

Inspect keys during development:

```bash
redis-cli keys 'codeforge:*'
```

Avoid using unrestricted `KEYS` commands in production.

---

# 16. Frontend Setup

Navigate to the frontend:

```bash
cd frontend/web-ide
```

Install dependencies:

```bash
npm install
```

Start development server:

```bash
npm run dev
```

Build:

```bash
npm run build
```

Preview production build:

```bash
npm run preview
```

Run linting:

```bash
npm run lint
```

---

# 17. Frontend Architecture

Recommended structure:

```text
frontend/web-ide/
├── src/
│   ├── components/
│   ├── pages/
│   ├── layouts/
│   ├── editor/
│   ├── terminal/
│   ├── explorer/
│   ├── collaboration/
│   ├── execution/
│   ├── api/
│   ├── hooks/
│   ├── state/
│   ├── types/
│   ├── utils/
│   ├── App.tsx
│   └── main.tsx
├── public/
├── package.json
├── tsconfig.json
└── vite.config.ts
```

---

# 18. Frontend Development Rules

TypeScript should run in strict mode.

Avoid:

```typescript
any
```

unless explicitly justified.

Prefer:

```typescript
interface ExecutionJob {
    execution_id: string;
    status: ExecutionStatus;
}
```

API responses should have explicit types.

Do not trust frontend authorization state.

For example, hiding a button does not provide security:

```text
Frontend
    │
    └── Hide Delete Button
```

The gateway must independently verify authorization:

```text
Request
   ↓
Gateway
   ↓
Authorization
   ↓
Operation
```

---

# 19. Monaco Editor

The editor should represent the project's file state.

Recommended lifecycle:

```text
Open File
    ↓
Load File
    ↓
Create Monaco Model
    ↓
Edit
    ↓
Detect Changes
    ↓
Save / Synchronize
```

Avoid continuously writing every keystroke directly to PostgreSQL.

Use:

* debouncing.
* revision numbers.
* collaboration events.
* explicit persistence boundaries.

---

# 20. Gateway Setup

Navigate to:

```bash
cd gateway/node
```

Install dependencies:

```bash
npm install
```

Start development mode:

```bash
npm run dev
```

Build:

```bash
npm run build
```

Run tests:

```bash
npm test
```

Run linting:

```bash
npm run lint
```

---

# 21. Gateway Responsibilities

The Node.js Gateway is the public application boundary.

It owns:

```text
HTTP API
Authentication
Authorization
Project operations
File operations
Execution submission
WebSocket connections
Rate limiting
Request validation
Request IDs
API error normalization
```

It does not own:

```text
C++ execution
Docker management for user code
Sandbox internals
Direct compiler execution
```

---

# 22. Gateway Request Flow

A typical API request follows:

```text
Browser
   ↓
Nginx
   ↓
Gateway
   ↓
Request ID
   ↓
Authentication
   ↓
Validation
   ↓
Authorization
   ↓
Business Logic
   ↓
Database / Redis / gRPC
   ↓
Response
```

Every request should have a request identifier.

Header:

```http
X-Request-ID: <uuid>
```

If a trusted request ID is supplied, it should be validated before propagation.

---

# 23. API Development

All public APIs use:

```text
/api/v1
```

Example:

```http
POST /api/v1/projects
```

API schemas are defined in:

```text
docs/04-api-reference.md
```

Developers must update the API document when changing:

* endpoints.
* request bodies.
* response bodies.
* status codes.
* authentication.
* authorization.
* error codes.

---

# 24. API Validation

Validate:

```text
path parameters
query parameters
headers
JSON body
file size
project membership
resource policy
```

Reject malformed requests before they reach internal services.

Example:

```text
Invalid project_id
       ↓
Gateway rejects
       ↓
400 / 404
```

rather than:

```text
Invalid project_id
       ↓
Evaluator
       ↓
Sandbox
```

---

# 25. Error Handling

Use the standard API envelope:

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

Do not expose:

```text
stack traces
database passwords
internal filesystem paths
container identifiers
service credentials
security implementation details
```

to clients.

---

# 26. Python Evaluator Setup

Navigate to:

```bash
cd evaluator/python
```

Create a virtual environment:

```bash
python3 -m venv .venv
```

Activate:

```bash
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Start the service:

```bash
uvicorn app.main:app --reload
```

Run tests:

```bash
pytest
```

Deactivate:

```bash
deactivate
```

---

# 27. Evaluator Architecture

Recommended structure:

```text
evaluator/python/
├── app/
│   ├── api/
│   ├── grpc/
│   ├── queue/
│   ├── scheduler/
│   ├── workers/
│   ├── models/
│   ├── services/
│   ├── policies/
│   ├── normalization/
│   └── main.py
├── tests/
├── requirements.txt
└── pyproject.toml
```

---

# 28. Evaluator Responsibilities

The evaluator should:

1. Validate execution metadata.
2. Resolve execution policy.
3. Dispatch jobs.
4. Manage queue state.
5. Communicate with sandbox workers.
6. Process sandbox results.
7. Normalize compiler/runtime output.
8. Update execution state.
9. Publish execution events.
10. Handle cancellation.

The evaluator must not bypass the sandbox.

Incorrect:

```text
Evaluator
   ↓
subprocess.run(user_code)
```

Correct:

```text
Evaluator
   ↓
Sandbox
   ↓
Restricted execution
```

---

# 29. C++ Sandbox Development

Navigate to:

```bash
cd sandbox/cpp
```

Recommended structure:

```text
sandbox/cpp/
├── include/
├── src/
├── tests/
├── CMakeLists.txt
├── Dockerfile
└── README.md
```

Configure:

```bash
cmake -S . -B build
```

Build:

```bash
cmake --build build
```

Run tests:

```bash
ctest --test-dir build
```

---

# 30. C++ Coding Standard

Use:

```text
C++17
-Wall
-Wextra
-Wpedantic
```

Recommended compiler command:

```bash
g++ -std=c++17 -Wall -Wextra -Wpedantic
```

Use RAII.

Prefer:

```cpp
std::unique_ptr
std::shared_ptr
std::vector
std::string
std::filesystem
```

over manual memory management where appropriate.

---

# 31. Sandbox Development Rule

The sandbox is security-critical.

Never test untrusted code directly on the host when testing the actual execution pipeline.

Use:

```text
Developer
   ↓
Sandbox container
   ↓
Restricted execution
```

Do not assume:

```text
Docker container = complete security boundary
```

The sandbox must use defense-in-depth.

---

# 32. Sandbox Resource Policy

The sandbox must enforce server-defined limits.

Example:

```json
{
  "cpu_time_ms": 2000,
  "wall_time_ms": 5000,
  "memory_bytes": 268435456,
  "processes": 32,
  "disk_bytes": 10485760,
  "output_bytes": 1048576
}
```

The client must not be allowed to arbitrarily increase limits.

The evaluator or gateway should validate requested limits against the platform policy.

---

# 33. Execution Snapshot Development

Every execution must use an immutable snapshot.

Correct:

```text
Live Files
    ↓
Snapshot
    ↓
Execution Job
    ↓
Sandbox
```

Incorrect:

```text
Execution Job
    ↓
Read current files
    ↓
Sandbox
```

A snapshot should include:

```text
snapshot_id
project_id
entry_file
language
language_version
files
file hashes
snapshot hash
created_at
```

---

# 34. Snapshot Reproducibility

For reproducibility, record:

```text
snapshot_id
snapshot_sha256
compiler version
language version
sandbox image digest
evaluator version
resource policy
```

This allows an execution to be investigated after the live project has changed.

---

# 35. Protocol Buffers

Protocol definitions should live under:

```text
proto/
```

Example:

```text
proto/
├── execution.proto
├── evaluator.proto
└── sandbox.proto
```

Generate language-specific clients according to the repository build process.

Do not manually duplicate gRPC message definitions across services.

The `.proto` contract is the source of truth.

---

# 36. gRPC Development

The internal communication path is:

```text
Gateway
    │
    ▼
Evaluator
    │
    ▼
Sandbox
```

Use:

```text
timeouts
metadata
request IDs
trace IDs
authentication/service identity
```

Internal service failures must not cause unbounded retries.

---

# 37. gRPC Error Handling

Distinguish:

```text
INVALID_ARGUMENT
UNAUTHENTICATED
PERMISSION_DENIED
NOT_FOUND
ALREADY_EXISTS
RESOURCE_EXHAUSTED
FAILED_PRECONDITION
DEADLINE_EXCEEDED
UNAVAILABLE
INTERNAL
```

Retry only operations where retrying is safe.

---

# 38. WebSocket Development

The WebSocket endpoint is:

```text
/ws
```

Connections should authenticate before joining project rooms.

Example flow:

```text
Browser
   ↓
WebSocket Connect
   ↓
Authentication
   ↓
Authorization
   ↓
Project Room
   ↓
Events
```

A client must never be able to subscribe to an unauthorized project.

---

# 39. Collaboration Events

Example event:

```json
{
  "event_id": "uuid",
  "type": "file:update",
  "project_id": "uuid",
  "file_id": "uuid",
  "user_id": "uuid",
  "timestamp": "2026-09-08T12:00:00Z",
  "payload": {
    "revision": 12
  }
}
```

Events should contain enough metadata to trace them.

Do not include unnecessary sensitive information.

---

# 40. Testing Strategy

Testing exists at multiple levels:

```text
Unit
 ↓
Integration
 ↓
Contract
 ↓
Security
 ↓
End-to-End
 ↓
Load
```

---

# 41. Unit Testing

Unit tests should cover:

### Gateway

* validation.
* authorization.
* service logic.
* error mapping.

### Evaluator

* scheduling.
* status transitions.
* policy validation.
* result normalization.

### Sandbox

* process management.
* timeout.
* resource handling.
* output capture.

### Frontend

* components.
* hooks.
* state transitions.
* API clients.

---

# 42. Integration Testing

Integration tests should validate:

```text
Gateway ↔ PostgreSQL
Gateway ↔ Redis
Gateway ↔ Evaluator
Evaluator ↔ Redis
Evaluator ↔ Sandbox
```

Example:

```text
POST /executions
       ↓
PostgreSQL
       ↓
Redis
       ↓
Evaluator
       ↓
Sandbox
       ↓
Result
```

---

# 43. End-to-End Testing

An E2E test should simulate a real user:

```text
Register
   ↓
Login
   ↓
Create Project
   ↓
Create File
   ↓
Write Code
   ↓
Execute
   ↓
Wait for Result
   ↓
Verify stdout
```

---

# 44. Security Testing

Security tests should include:

```text
Path traversal
Authorization bypass
Cross-project access
Malformed input
Oversized input
Fork bomb
Infinite loop
Memory exhaustion
Output exhaustion
Process exhaustion
Network access
Filesystem access
Container privilege escalation
Secret exposure
```

---

# 45. Malicious Program Tests

Examples of sandbox test categories:

### Infinite loop

```cpp
while (true)
{
}
```

Expected:

```text
TIMEOUT
```

### Excessive output

```cpp
while (true)
{
    std::cout << "X";
}
```

Expected:

```text
RESOURCE_LIMIT
```

or output truncation according to policy.

### Process exhaustion

A controlled fork/process-spawn test should verify process limits.

Expected:

```text
RESOURCE_LIMIT
```

---

# 46. Test Data Isolation

Tests must not accidentally access production data.

Use separate environments:

```text
development
test
staging
production
```

Recommended:

```text
codeforge_dev
codeforge_test
codeforge_staging
codeforge_prod
```

---

# 47. Running the Test Suite

Recommended top-level commands:

```bash
make test
```

Unit tests:

```bash
make test-unit
```

Integration tests:

```bash
make test-integration
```

Security tests:

```bash
make test-security
```

Load tests:

```bash
make test-load
```

If these targets are not yet implemented, they should be added as the project matures.

---

# 48. Makefile Workflow

Recommended top-level commands:

```bash
make build
make test
make lint
make format
make clean
make up
make down
make logs
```

Example:

```bash
make up
```

starts the development environment.

```bash
make build
```

builds services.

```bash
make test
```

runs the test suite.

---

# 49. Code Formatting

Formatting should be automated.

### C++

Use:

```bash
clang-format
```

### Python

Use:

```bash
ruff
black
```

or the project's selected formatter.

### TypeScript

Use:

```bash
prettier
eslint
```

Formatting should be part of CI.

---

# 50. Static Analysis

Recommended:

### C++

```text
clang-tidy
clang-analyzer
```

### Python

```text
ruff
mypy
```

### TypeScript

```text
eslint
TypeScript compiler
```

Static analysis should run before merging.

---

# 51. Debugging the Gateway

Check:

```bash
docker compose logs gateway
```

or run locally:

```bash
npm run dev
```

Test health endpoint:

```bash
curl http://localhost:3000/health
```

Test an API:

```bash
curl http://localhost:3000/api/v1/projects
```

Authentication should be supplied where required.

---

# 52. Debugging the Evaluator

Inspect logs:

```bash
docker compose logs evaluator
```

Check Redis:

```bash
redis-cli ping
```

Inspect queue state using the project's configured Redis keys.

Verify:

```text
job created
job queued
worker receives job
sandbox requested
result received
database updated
event published
```

---

# 53. Debugging the Sandbox

Sandbox debugging must avoid disabling security controls in a way that could become permanent.

Inspect container:

```bash
docker ps
```

Inspect logs:

```bash
docker logs <container>
```

Inspect configuration:

```bash
docker inspect <container>
```

Do not expose the Docker socket to user workloads for debugging.

---

# 54. Request Tracing

When debugging an execution, start with:

```text
request_id
```

Then follow:

```text
request_id
    ↓
execution_id
    ↓
job_id
    ↓
snapshot_id
    ↓
snapshot_sha256
    ↓
result_id
```

This chain should make it possible to reconstruct the execution lifecycle.

---

# 55. Execution State Debugging

Valid execution flow:

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

Alternative terminal states:

```text
FAILED
TIMEOUT
CANCELLED
RESOURCE_LIMIT
```

An execution should never transition backward.

Invalid:

```text
RUNNING
   ↓
QUEUED
```

Such transitions should be rejected.

---

# 56. Database Debugging

Useful commands:

```sql
SELECT *
FROM execution_jobs
ORDER BY submitted_at DESC
LIMIT 20;
```

Inspect failed jobs:

```sql
SELECT id, status, language, submitted_at
FROM execution_jobs
WHERE status = 'FAILED';
```

Inspect results:

```sql
SELECT *
FROM execution_results
ORDER BY created_at DESC
LIMIT 20;
```

Production debugging should use read-only access wherever possible.

---

# 57. Redis Debugging

Inspect queue-related keys:

```bash
redis-cli --scan --pattern 'codeforge:queue:*'
```

Inspect execution state:

```bash
redis-cli --scan --pattern 'codeforge:execution:*'
```

Avoid destructive Redis commands during production debugging.

Never run:

```bash
redis-cli FLUSHALL
```

against production.

---

# 58. Common Development Problems

## Docker is not running

Check:

```bash
docker info
```

Start Docker according to the host operating system.

---

## Port already in use

Check:

```bash
ss -ltnp
```

or:

```bash
lsof -i :3000
```

Change the development port or stop the conflicting service.

---

## PostgreSQL connection failure

Verify:

```bash
docker compose ps
```

Then:

```bash
docker compose logs postgres
```

Verify:

```text
host
port
database
username
password
```

---

## Redis connection failure

Check:

```bash
redis-cli ping
```

Then:

```bash
docker compose logs redis
```

---

# 59. Frontend Cannot Reach Gateway

Check:

```text
Browser
   ↓
Nginx
   ↓
Gateway
```

Verify:

* frontend API URL.
* Nginx configuration.
* gateway port.
* CORS configuration.
* authentication token.
* WebSocket endpoint.

---

# 60. Execution Stuck in QUEUED

Investigate:

```text
Gateway
 ↓
Database
 ↓
Outbox
 ↓
Redis
 ↓
Evaluator
 ↓
Worker
```

Check:

1. Job exists.
2. Outbox event exists.
3. Redis queue contains job.
4. Evaluator is running.
5. Worker is running.
6. Sandbox is reachable.

---

# 61. Execution Stuck in RUNNING

Investigate:

```text
Evaluator
     ↓
Sandbox
     ↓
Process
```

Check:

* sandbox logs.
* wall-clock timer.
* process state.
* resource controller.
* cancellation mechanism.
* result callback.

A running execution must not remain indefinitely in the `RUNNING` state.

---

# 62. Compiler Errors

Compiler errors are expected user-level execution failures.

Example:

```text
COMPILING
   ↓
Compiler exits non-zero
   ↓
FAILED
```

The compiler output should be returned through the execution result.

Do not treat a normal user compilation error as a platform crash.

---

# 63. Git Workflow

Create a feature branch:

```bash
git checkout -b feature/execution-api
```

Work on the feature.

Check status:

```bash
git status
```

Review changes:

```bash
git diff
```

Stage:

```bash
git add .
```

Commit:

```bash
git commit -m "feat: add execution API"
```

Push:

```bash
git push -u origin feature/execution-api
```

---

# 64. Commit Convention

Recommended format:

```text
type(scope): description
```

Examples:

```text
feat(gateway): add execution endpoint
fix(sandbox): enforce process limit
feat(evaluator): add execution queue worker
test(api): add project authorization tests
docs(api): document execution schemas
refactor(sandbox): isolate process manager
chore(ci): add security test stage
```

---

# 65. Pull Request Requirements

A pull request should include:

```text
Summary
Changes
Testing
Security impact
Database changes
API changes
Configuration changes
Documentation changes
```

Example checklist:

```text
[ ] Code builds
[ ] Unit tests pass
[ ] Integration tests pass
[ ] Security tests pass
[ ] Documentation updated
[ ] No secrets committed
[ ] API contract updated
[ ] Database migration included if needed
```

---

# 66. Database Change Workflow

For a schema change:

```text
Modify data model
      ↓
Create migration
      ↓
Update application models
      ↓
Update tests
      ↓
Update data-model documentation
      ↓
Run migration
      ↓
Integration test
```

Never manually modify production schema outside the migration process except for controlled emergency procedures.

---

# 67. API Change Workflow

For an API change:

```text
API requirement
      ↓
Schema update
      ↓
Gateway implementation
      ↓
Client implementation
      ↓
Tests
      ↓
Documentation
```

Breaking changes require an explicit API versioning decision.

---

# 68. Protocol Change Workflow

For gRPC/protobuf changes:

```text
Modify .proto
      ↓
Review compatibility
      ↓
Regenerate clients
      ↓
Update services
      ↓
Run contract tests
```

Avoid removing fields that older clients may still send.

Prefer additive changes.

---

# 69. Configuration Change Workflow

For new configuration:

1. Add to `.env.example`.
2. Add validation.
3. Add development default where safe.
4. Document the variable.
5. Update deployment configuration.
6. Add tests where appropriate.

Never silently use insecure production defaults.

---

# 70. Security Development Checklist

Before merging security-sensitive code:

```text
[ ] Input validated
[ ] Authorization checked
[ ] Resource limits enforced
[ ] Errors sanitized
[ ] Secrets protected
[ ] Logs sanitized
[ ] Timeouts configured
[ ] Retry behavior reviewed
[ ] Cross-project access tested
[ ] Abuse case tested
```

---

# 71. Sandbox Security Checklist

Before modifying sandbox code:

```text
[ ] No host filesystem access
[ ] No Docker socket
[ ] No privileged mode
[ ] Network disabled
[ ] Capabilities minimized
[ ] seccomp configured
[ ] namespaces configured
[ ] cgroups configured
[ ] process limits enforced
[ ] memory limits enforced
[ ] output limits enforced
[ ] timeout enforced
[ ] cleanup guaranteed
```

---

# 72. Logging Rules

Logs should help answer:

```text
What happened?
When?
Where?
For which request?
For which execution?
Why did it fail?
```

Logs should not expose:

```text
passwords
tokens
session secrets
private keys
database credentials
raw user source
```

unless an explicit controlled debugging process requires temporary access.

---

# 73. Local Development Logging

Recommended development log levels:

```text
DEBUG
INFO
WARN
ERROR
```

Production should avoid excessive `DEBUG` logging.

Structured JSON logging is preferred for service logs.

---

# 74. Health Checks

Each service should expose an appropriate health mechanism.

Example:

```http
GET /health
```

Possible response:

```json
{
  "status": "ok"
}
```

Readiness should be distinguished from basic process liveness where necessary.

Example:

```text
Liveness
    Process exists

Readiness
    Process can serve traffic
    +
    Required dependencies available
```

---

# 75. Graceful Shutdown

Services should handle:

```text
SIGTERM
SIGINT
```

Shutdown sequence:

```text
Receive signal
     ↓
Stop accepting new work
     ↓
Finish safe in-flight operations
     ↓
Close WebSockets
     ↓
Close gRPC
     ↓
Close Redis
     ↓
Close PostgreSQL
     ↓
Exit
```

Sandbox workers require additional cleanup guarantees.

---

# 76. Failure Handling

The system should fail closed.

Examples:

```text
Authentication failure
    → reject

Authorization failure
    → reject

Invalid snapshot
    → reject

Invalid resource policy
    → reject

Sandbox unavailable
    → execution failure/retry according to policy

Unknown execution state
    → reject transition
```

Never silently execute code using an unsafe fallback.

---

# 77. Local Full-Stack Startup

A typical local workflow:

### Step 1

Start infrastructure:

```bash
docker compose up -d postgres redis
```

### Step 2

Run migrations.

### Step 3

Start evaluator.

### Step 4

Start gateway.

### Step 5

Start frontend.

### Step 6

Start sandbox worker environment.

### Step 7

Open the Web IDE.

Expected architecture:

```text
Browser
   │
   ▼
Frontend
   │
   ▼
Gateway
   │
   ├── PostgreSQL
   ├── Redis
   └── Evaluator
          │
          ▼
       Sandbox
```

---

# 78. Recommended Developer Workflow

Every development session should follow:

```text
git pull
   ↓
Create/update branch
   ↓
Start dependencies
   ↓
Implement change
   ↓
Run formatter
   ↓
Run static analysis
   ↓
Run unit tests
   ↓
Run integration tests
   ↓
Run security tests if relevant
   ↓
Inspect git diff
   ↓
Commit
   ↓
Push
```

---

# 79. Pre-Commit Checklist

Before committing:

```text
[ ] git status reviewed
[ ] git diff reviewed
[ ] no credentials present
[ ] tests pass
[ ] formatter passes
[ ] linter passes
[ ] documentation updated
[ ] migration included if needed
[ ] API schema updated if needed
```

---

# 80. Pre-Release Checklist

Before creating a release:

```text
[ ] All required features implemented
[ ] Unit tests pass
[ ] Integration tests pass
[ ] E2E tests pass
[ ] Security tests pass
[ ] Load tests completed
[ ] Database migrations verified
[ ] Backups verified
[ ] Configuration reviewed
[ ] Secrets verified
[ ] Logs reviewed
[ ] Monitoring verified
[ ] Documentation complete
[ ] Rollback procedure tested
```

---

# 81. Developer Documentation Map

Use the following documents for different questions:

```text
01-product-requirements.md
    Product requirements and scope

02-architecture.md
    System architecture and boundaries

03-data-model.md
    Database and persistence

04-api-reference.md
    REST/WebSocket/gRPC contracts

05-roadmap-and-phases.md
    Development roadmap

06-development-guide.md
    Local development and engineering workflow

07-security.md
    Security architecture and threat model

08-gap-analysis.md
    Missing capabilities and technical gaps

09-testing-strategy.md
    Testing architecture

10-glossary.md
    Project terminology
```

---

# 82. When to Update Documentation

Documentation must be updated whenever there is a change to:

```text
Architecture
Database
API
Security model
Execution lifecycle
Collaboration protocol
Deployment
Development workflow
Testing strategy
```

Documentation should be changed in the same pull request as the implementation whenever practical.

---

# 83. Architecture Change Process

Architecture changes should follow:

```text
Problem
   ↓
Proposal
   ↓
Impact Analysis
   ↓
ADR
   ↓
Implementation
   ↓
Testing
   ↓
Architecture Documentation
```

Significant architectural changes should receive an Architecture Decision Record.

---

# 84. Debugging Philosophy

When a failure occurs:

Do not immediately modify multiple services.

Instead:

```text
Identify request
      ↓
Identify execution
      ↓
Identify state
      ↓
Identify service boundary
      ↓
Inspect logs
      ↓
Inspect database
      ↓
Inspect queue
      ↓
Inspect downstream service
```

This keeps distributed-system debugging deterministic.

---

# 85. Development Anti-Patterns

Avoid:

```text
Executing user code in Gateway
Executing user code in Evaluator
Using live files instead of snapshots
Trusting frontend authorization
Hardcoding secrets
Using Redis as durable truth
Writing raw SQL everywhere
Skipping migrations
Disabling sandbox security permanently
Unlimited execution time
Unlimited output
Unlimited process creation
Unbounded retries
Logging raw credentials
```

---

# 86. Performance Development Rules

Do not optimize based on assumptions.

Measure:

```text
latency
throughput
CPU
memory
queue depth
database queries
Redis operations
sandbox startup
WebSocket traffic
```

Track:

```text
p50
p95
p99
```

Performance changes should include before/after measurements where practical.

---

# 87. Dependency Management

Dependencies should be:

* explicitly versioned.
* regularly updated.
* reviewed for security vulnerabilities.
* removed when unused.

Do not introduce a large dependency for functionality that can be implemented safely with existing platform capabilities.

---

# 88. Dependency Security

Periodically run dependency audits.

For Node.js:

```bash
npm audit
```

For Python, use the project's selected dependency scanner.

For containers, scan images for known vulnerabilities.

Security scanning should be incorporated into CI as the project matures.

---

# 89. Reproducible Development

The same repository should produce consistent builds across developer environments.

Use:

```text
package-lock.json
requirements lock files
CMake configuration
Dockerfiles
Docker Compose
migration scripts
```

Avoid undocumented machine-specific dependencies.

---

# 90. Development Environment Reset

When the local environment becomes inconsistent:

```bash
docker compose down
```

Optionally remove development volumes:

```bash
docker compose down -v
```

Rebuild:

```bash
docker compose build --no-cache
```

Start again:

```bash
docker compose up -d
```

Use volume deletion carefully because it destroys local database state.

---

# 91. Complete Developer Verification

A developer's implementation should be considered locally verified when:

```text
Repository
   ↓
Build
   ↓
Services start
   ↓
Database migrations
   ↓
Authentication
   ↓
Project creation
   ↓
File creation
   ↓
Execution submission
   ↓
Snapshot
   ↓
Queue
   ↓
Evaluator
   ↓
Sandbox
   ↓
Compilation
   ↓
Execution
   ↓
Result
   ↓
WebSocket event
   ↓
Browser terminal
```

works without manual intervention.

---

# 92. Golden Path Test

The project should maintain one canonical "golden path":

```text
1. Start services
2. Register user
3. Login
4. Create project
5. Create main.cpp
6. Insert valid C++ program
7. Submit execution
8. Capture snapshot
9. Queue job
10. Dispatch evaluator
11. Start sandbox
12. Compile
13. Execute
14. Capture stdout
15. Persist result
16. Publish execution event
17. Display output
```

Example source:

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello, CodeForge!" << std::endl;
    return 0;
}
```

Expected output:

```text
Hello, CodeForge!
```

This test should remain stable throughout development.

---

# 93. Troubleshooting Decision Tree

```text
Request fails
    │
    ├── 401?
    │     └── Check authentication
    │
    ├── 403?
    │     └── Check project membership/role
    │
    ├── 404?
    │     └── Check resource ID/project
    │
    ├── 409?
    │     └── Check revision/conflict/state
    │
    ├── 422?
    │     └── Check request validation
    │
    ├── 429?
    │     └── Check rate limits
    │
    ├── 5xx?
    │     └── Check service logs
    │
    └── Execution stuck?
          └── Follow execution state chain
```

---

# 94. Developer Security Boundary

The most important development boundary is:

```text
                 TRUSTED PLATFORM
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
    Gateway         Evaluator       Database
       │               │
       └───────────────┘
                       │
                 SECURITY BOUNDARY
                       │
                       ▼
              UNTRUSTED USER CODE
                       │
                       ▼
                    Sandbox
```

User code must always be treated as hostile.

---

# 95. Final Development Rules

Every developer working on CodeForge Cloud should remember:

### Rule 1

**Never trust the browser.**

### Rule 2

**Never execute user code outside the sandbox.**

### Rule 3

**Never execute against mutable live project state.**

### Rule 4

**Never expose host resources to user workloads.**

### Rule 5

**Never commit secrets.**

### Rule 6

**Never bypass authorization.**

### Rule 7

**Never bypass resource limits.**

### Rule 8

**Never introduce an undocumented contract change.**

### Rule 9

**Never ignore security failures.**

### Rule 10

**Every important operation should be observable.**

---

# 96. Development Lifecycle Summary

The complete engineering lifecycle is:

```text
Requirement
    ↓
Architecture
    ↓
API/Data Contract
    ↓
Implementation
    ↓
Unit Test
    ↓
Integration Test
    ↓
Security Test
    ↓
Performance Test
    ↓
Documentation
    ↓
Code Review
    ↓
Merge
    ↓
Release
```

This workflow ensures that CodeForge Cloud remains maintainable while its execution infrastructure becomes increasingly sophisticated.

---

# 97. Final Developer Architecture

```text
                         Developer
                             │
                             ▼
                       Git Repository
                             │
             ┌───────────────┼───────────────┐
             ▼               ▼               ▼
         Frontend         Gateway         Evaluator
             │               │               │
             │               ├───────┐       │
             │               ▼       ▼       ▼
             │          PostgreSQL  Redis   Sandbox
             │                               │
             │                               ▼
             │                         User Program
             │                               │
             └───────────────┬───────────────┘
                             ▼
                         Test Suite
                             │
                             ▼
                       CI / Validation
                             │
                             ▼
                         Production
```

---

# 98. Document Status

**Document:** 06 — Development Guide
**Version:** 1.0
**Status:** Draft
**Author:** Adarsh Kumar

**Previous Document:** `05-roadmap-and-phases.md`

**Next Document:** `07-security.md`

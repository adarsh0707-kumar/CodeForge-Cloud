# Contributing to CodeForge Cloud

Thank you for your interest in contributing to **CodeForge Cloud**.

CodeForge Cloud is a cloud-native collaborative online code compiler and IDE sandbox designed around secure execution, real-time collaboration, reproducibility, observability, and scalable service architecture.

Because the platform executes **untrusted user-supplied source code**, contributions must follow strict engineering and security practices.

This document explains how to set up the project, create changes, write tests, update documentation, submit pull requests, and maintain the security guarantees of the platform.

---

## Table of Contents

* [1. Project Overview](#1-project-overview)
* [2. Contribution Principles](#2-contribution-principles)
* [3. Repository Structure](#3-repository-structure)
* [4. Development Environment](#4-development-environment)
* [5. Getting the Project](#5-getting-the-project)
* [6. Local Development](#6-local-development)
* [7. Branching Strategy](#7-branching-strategy)
* [8. Commit Convention](#8-commit-convention)
* [9. Making Changes](#9-making-changes)
* [10. Frontend Contributions](#10-frontend-contributions)
* [11. Gateway Contributions](#11-gateway-contributions)
* [12. Evaluator Contributions](#12-evaluator-contributions)
* [13. Sandbox Contributions](#13-sandbox-contributions)
* [14. Database Contributions](#14-database-contributions)
* [15. API and Protocol Changes](#15-api-and-protocol-changes)
* [16. Real-Time Collaboration](#16-real-time-collaboration)
* [17. Testing Requirements](#17-testing-requirements)
* [18. Security Requirements](#18-security-requirements)
* [19. Documentation Requirements](#19-documentation-requirements)
* [20. Code Quality](#20-code-quality)
* [21. Pull Request Process](#21-pull-request-process)
* [22. Review Checklist](#22-review-checklist)
* [23. Breaking Changes](#23-breaking-changes)
* [24. Performance Changes](#24-performance-changes)
* [25. Observability](#25-observability)
* [26. Dependency Changes](#26-dependency-changes)
* [27. Reporting Security Issues](#27-reporting-security-issues)
* [28. Contributor Definition of Done](#28-contributor-definition-of-done)
* [29. Final Engineering Principles](#29-final-engineering-principles)

---

# 1. Project Overview

CodeForge Cloud consists of multiple cooperating services:

```text
React Web IDE
      │
      │ HTTPS / WebSocket
      ▼
    Nginx
      │
      ▼
Node.js Gateway
      │
      ├────────────── PostgreSQL
      │
      ├────────────── Redis
      │
      └────────────── gRPC
                         │
                         ▼
                  Python Evaluator
                         │
                         ▼
                    Redis Queue
                         │
                         ▼
                       Worker
                         │
                         ▼
                  C++ Sandbox Runtime
                         │
                         ▼
                Isolated Execution
```

The main components are:

| Component    | Technology             | Responsibility                                        |
| ------------ | ---------------------- | ----------------------------------------------------- |
| Web IDE      | React + TypeScript     | Editor and user interface                             |
| Gateway      | Node.js + TypeScript   | Public API, authentication, authorization, WebSockets |
| Evaluator    | Python + FastAPI       | Scheduling, validation, execution orchestration       |
| Sandbox      | C++                    | Secure low-level execution                            |
| Database     | PostgreSQL             | Durable application state                             |
| Queue/Cache  | Redis                  | Queues, caching, pub/sub, presence, rate limiting     |
| Internal RPC | gRPC + Protobuf        | Service-to-service communication                      |
| Proxy        | Nginx                  | Reverse proxy and edge routing                        |
| Runtime      | Docker/Linux isolation | Execution isolation                                   |

---

# 2. Contribution Principles

All contributions should follow these principles:

1. **Security first**
2. **Correctness before optimization**
3. **Explicit contracts**
4. **Reproducible execution**
5. **Observable behavior**
6. **Test-driven changes**
7. **Backward compatibility where practical**
8. **Small, reviewable changes**
9. **Clear documentation**
10. **Fail closed on security-sensitive failures**

The most important platform invariant is:

> **User source code must never execute directly on the host system.**

Another critical invariant is:

> **An execution must run against an immutable snapshot, never mutable live project state.**

---

# 3. Repository Structure

The repository follows this high-level structure:

```text
Online-Code-Compiler/
│
├── docs/
│   ├── 01-product-requirements.md
│   ├── 02-architecture.md
│   ├── 03-data-model.md
│   ├── 04-api-reference.md
│   ├── 05-roadmap-and-phases.md
│   ├── 06-development-guide.md
│   ├── 07-security.md
│   ├── 08-gap-analysis.md
│   ├── 09-testing-strategy.md
│   ├── 10-glossary.md
│   └── README.md
│
├── frontend/
│   └── web-ide/
│
├── gateway/
│   └── node/
│
├── evaluator/
│   └── python/
│
├── sandbox/
│   └── cpp/
│
├── proto/
│
├── infrastructure/
│   ├── docker/
│   ├── postgres/
│   ├── redis/
│   └── nginx/
│
├── tests/
│   ├── integration/
│   ├── security/
│   └── load/
│
├── scripts/
│
├── docker-compose.yml
├── Makefile
├── .gitignore
├── LICENSE
├── README.md
└── CONTRIBUTING.md
```

Contributors should avoid creating arbitrary top-level directories without an architectural reason.

---

# 4. Development Environment

Recommended development environment:

* Linux
* Git
* Docker
* Docker Compose
* Node.js
* npm
* Python 3
* C++ compiler
* CMake
* PostgreSQL client
* Redis client
* Protobuf compiler
* gRPC tooling

Verify your environment:

```bash
git --version
docker --version
docker compose version
node --version
npm --version
python3 --version
g++ --version
cmake --version
```

---

# 5. Getting the Project

Clone the repository:

```bash
git clone <repository-url>
cd Online-Code-Compiler
```

Check the repository:

```bash
git status
```

The working tree should initially be clean.

---

# 6. Local Development

Start infrastructure:

```bash
docker compose up -d
```

Check services:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f
```

Stop services:

```bash
docker compose down
```

Stop services and remove volumes:

```bash
docker compose down -v
```

Use the destructive `-v` option only when you intentionally want to remove local persistent data.

---

## 6.1 Frontend

```bash
cd frontend/web-ide

npm install
npm run dev
```

Production build:

```bash
npm run build
```

Preview:

```bash
npm run preview
```

Lint:

```bash
npm run lint
```

---

## 6.2 Gateway

```bash
cd gateway/node

npm install
npm run dev
```

Build:

```bash
npm run build
```

Tests:

```bash
npm test
```

Lint:

```bash
npm run lint
```

---

## 6.3 Evaluator

```bash
cd evaluator/python

python3 -m venv .venv
source .venv/bin/activate

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

---

## 6.4 Sandbox

```bash
cd sandbox/cpp

cmake -S . -B build
cmake --build build
```

Run tests:

```bash
ctest --test-dir build
```

---

## 6.5 Root Makefile

Where supported:

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

---

# 7. Branching Strategy

Do not develop directly on `main`.

Create a dedicated branch:

```bash
git checkout -b feature/execution-api
```

Recommended branch naming:

```text
feature/<name>
fix/<name>
security/<name>
refactor/<name>
docs/<name>
test/<name>
perf/<name>
build/<name>
ci/<name>
```

Examples:

```text
feature/execution-history
feature/project-members
fix/websocket-reconnect
security/sandbox-network-isolation
refactor/evaluator-scheduler
docs/update-security-model
test/execution-timeout
perf/redis-queue-processing
ci/add-container-scan
```

Keep branches focused on one logical change whenever possible.

---

# 8. Commit Convention

Use clear, imperative commit messages.

Recommended format:

```text
<type>: <short description>
```

Examples:

```text
feat: add execution submission API
fix: prevent cross-project file access
security: disable sandbox networking
test: add snapshot integrity tests
docs: update sandbox security model
refactor: simplify evaluator scheduler
perf: reduce execution queue latency
ci: add security scanning
build: update sandbox image
```

Recommended commit types:

| Type       | Purpose                 |
| ---------- | ----------------------- |
| `feat`     | New functionality       |
| `fix`      | Bug fix                 |
| `security` | Security-related change |
| `test`     | Tests                   |
| `docs`     | Documentation           |
| `refactor` | Internal restructuring  |
| `perf`     | Performance             |
| `build`    | Build/tooling           |
| `ci`       | CI/CD                   |
| `chore`    | Maintenance             |

Avoid commits such as:

```text
update
changes
final
working
test
stuff
fix
```

Commit messages should explain what changed.

---

# 9. Making Changes

Before modifying code:

1. Understand the relevant architecture.
2. Read the related documentation.
3. Identify affected services.
4. Identify API/data contracts.
5. Identify security implications.
6. Identify required tests.
7. Determine whether documentation must change.

Useful documentation:

```text
docs/02-architecture.md
docs/03-data-model.md
docs/04-api-reference.md
docs/06-development-guide.md
docs/07-security.md
docs/09-testing-strategy.md
```

---

## 9.1 Keep Changes Focused

Prefer:

```text
One feature
    ↓
Implementation
    ↓
Tests
    ↓
Documentation
```

Avoid combining unrelated changes:

```text
Feature + database migration + UI redesign + dependency upgrade
```

unless there is a clear reason they must be released together.

---

# 10. Frontend Contributions

The frontend uses React and TypeScript.

Frontend contributions should:

* use TypeScript
* maintain component boundaries
* avoid unnecessary global state
* handle loading states
* handle error states
* handle disconnected WebSocket states
* validate user-facing input
* never assume browser input is trusted
* safely render terminal output
* avoid exposing internal service details

---

## 10.1 Monaco Editor

Editor-related changes should consider:

* large files
* unsaved changes
* concurrent edits
* syntax highlighting
* cursor state
* selection state
* file switching
* connection loss
* reconnection
* execution state

---

## 10.2 Terminal Output

Terminal output is untrusted data.

Do not render compiler or program output as trusted HTML.

Avoid unsafe patterns such as:

```typescript
dangerouslySetInnerHTML
```

unless there is a documented and security-reviewed reason.

---

# 11. Gateway Contributions

The Gateway is the public boundary of the platform.

It is responsible for:

* authentication
* authorization
* request validation
* project management
* file management
* execution submission
* execution history
* WebSocket handling
* rate limiting
* request IDs
* API responses

The Gateway must **never execute user source code**.

---

## 11.1 Authorization

Every protected resource must perform server-side authorization.

Do not rely on:

```text
UUID secrecy
Frontend route restrictions
Hidden UI controls
Client-side permissions
```

Correct model:

```text
Request
 ↓
Authenticate
 ↓
Identify user
 ↓
Load resource
 ↓
Check authorization
 ↓
Perform operation
```

---

## 11.2 Error Responses

Use the standard error envelope:

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

* database stack traces
* filesystem paths
* internal credentials
* service secrets
* container details
* infrastructure internals

---

# 12. Evaluator Contributions

The evaluator is responsible for:

* validating execution requests
* scheduling jobs
* applying resource policies
* dispatching executions
* tracking execution state
* collecting results
* coordinating workers

The evaluator must treat all execution input as untrusted.

---

## 12.1 Execution State

Valid execution states include:

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

Terminal states must not silently transition back into active states.

---

## 12.2 Immutable Snapshot

Every execution must use a snapshot.

```text
Live Project
     ↓
Snapshot Capture
     ↓
Snapshot Validation
     ↓
Snapshot Persistence
     ↓
Queue
     ↓
Sandbox
     ↓
Compile
     ↓
Execute
```

Never execute directly from mutable project files.

---

# 13. Sandbox Contributions

The sandbox is the most security-sensitive component of CodeForge Cloud.

Contributors modifying sandbox behavior must assume they are changing a security boundary.

Sandbox changes require:

* security review
* unit tests
* isolation tests
* resource-limit tests
* regression tests
* documentation updates where applicable

---

## 13.1 Sandbox Security Requirements

User workloads must not have access to:

```text
Host filesystem
Docker socket
PostgreSQL
Redis
Gateway internals
Evaluator internals
Host credentials
Cloud metadata
Privileged host devices
```

---

## 13.2 Resource Limits

Every execution should have defined limits for:

* CPU
* memory
* process count
* execution time
* filesystem usage
* output size

No execution should be allowed to consume unbounded resources.

---

## 13.3 Network

Sandbox networking is disabled by default.

Do not enable outbound network access without:

1. documented architectural justification
2. threat analysis
3. security review
4. explicit policy
5. tests

---

# 14. Database Contributions

PostgreSQL is the durable source of truth.

Core entities include:

```text
users
projects
project_members
files
execution_jobs
execution_results
sessions
audit_events
```

Database changes must include:

* migration
* constraints
* indexes where required
* rollback consideration
* application changes
* tests
* documentation if the data model changes

---

## 14.1 Constraints

Prefer database-level integrity where practical.

Examples:

```text
UNIQUE constraints
FOREIGN KEY constraints
CHECK constraints
NOT NULL constraints
```

Do not rely exclusively on application-level validation for critical invariants.

---

## 14.2 Migrations

Use sequential migration numbering:

```text
001_users
002_projects
003_project_members
004_files
005_execution_jobs
006_execution_results
007_sessions
008_audit_events
```

Future migrations may include:

```text
009_outbox_events
010_file_versions
011_collaboration_state
```

Never rewrite an already-applied migration in a shared environment.

Create a new migration instead.

---

# 15. API and Protocol Changes

Public API changes must update:

```text
Gateway implementation
API reference
Validation
Tests
Frontend integration
Error handling
Documentation
```

The API base path is:

```text
/api/v1
```

Breaking changes require explicit documentation.

---

## 15.1 gRPC

Internal service contracts use Protocol Buffers.

Changes to `.proto` files should consider:

* backward compatibility
* field numbering
* optionality
* service compatibility
* generated code
* integration tests

Never reuse an existing Protobuf field number for an unrelated field.

---

## 15.2 Idempotency

Execution submission supports idempotency.

Use:

```text
Idempotency-Key
```

when appropriate.

Do not create duplicate execution jobs when the same request is intentionally retried.

---

# 16. Real-Time Collaboration

WebSocket events include:

```text
project:join
project:leave
file:open
file:update
file:create
file:delete
file:move
cursor:update
selection:update
presence:join
presence:update
presence:leave
```

Collaboration changes should consider:

* concurrent edits
* stale clients
* reconnects
* ordering
* duplicate messages
* unauthorized room access
* presence cleanup
* malformed messages
* message size limits
* rate limiting

A WebSocket connection does not bypass authorization.

---

# 17. Testing Requirements

Every meaningful code change should include appropriate tests.

Run:

```bash
make test
```

Where available:

```bash
make test-unit
make test-integration
make test-e2e
make test-security
make test-load
make coverage
```

---

## 17.1 Unit Tests

Use unit tests for:

* validation
* business logic
* state transitions
* resource policy logic
* snapshot hashing
* authorization rules
* utility functions

---

## 17.2 Integration Tests

Use integration tests for:

* API + PostgreSQL
* API + Redis
* Gateway + evaluator
* gRPC
* WebSocket
* queue processing
* execution lifecycle

---

## 17.3 Security Tests

Security-sensitive changes must include security tests.

Examples:

```text
Cross-project access
Path traversal
Symlink escape
Command injection
SQL injection
XSS
Network access
Host filesystem access
Docker socket access
Resource exhaustion
Fork bomb
Output flood
Timeout enforcement
Privilege escalation
```

---

## 17.4 Snapshot Tests

Changes involving execution snapshots should test:

```text
SNAP-001  Snapshot creation
SNAP-002  Required files
SNAP-003  Hash correctness
SNAP-004  Deterministic hashing
SNAP-005  File hashes
SNAP-006  Snapshot immutability
SNAP-007  Invalid entry file
SNAP-008  Oversized snapshot
```

---

## 17.5 Regression Tests

Every important bug should become a regression test where practical.

Preferred lifecycle:

```text
Bug
 ↓
Reproduce
 ↓
Write regression test
 ↓
Fix
 ↓
Verify test
 ↓
Merge
```

---

# 18. Security Requirements

Security is a mandatory part of every contribution.

The following rules are non-negotiable.

### Rule 1

Never execute user code directly on the host.

### Rule 2

Never expose the Docker socket to user workloads.

### Rule 3

Never mount arbitrary host directories into user workloads.

### Rule 4

Never allow sandbox workloads to access PostgreSQL directly.

### Rule 5

Never allow sandbox workloads to access Redis directly.

### Rule 6

Never bypass server-side authorization.

### Rule 7

Never trust client-side validation.

### Rule 8

Never run an execution without a timeout.

### Rule 9

Never run an execution without resource limits.

### Rule 10

Never execute against mutable live project state.

### Rule 11

Never log passwords, access tokens, session tokens, or secrets.

### Rule 12

Never silently fail open when a security control fails.

---

## 18.1 Security-Sensitive Changes

The following changes automatically require additional review:

```text
Sandbox runtime
Container configuration
Linux namespaces
cgroups
seccomp
Capabilities
Filesystem mounts
Networking
Authentication
Authorization
Session handling
Secrets
Database permissions
Docker configuration
Execution policy
Resource limits
WebSocket authorization
API security
Dependency upgrades affecting security
```

---

# 19. Documentation Requirements

Documentation is part of the implementation.

Update documentation when changing:

* architecture
* API contracts
* database schema
* security behavior
* execution lifecycle
* configuration
* development workflows
* testing requirements
* deployment behavior

Relevant documentation:

```text
docs/01-product-requirements.md
docs/02-architecture.md
docs/03-data-model.md
docs/04-api-reference.md
docs/05-roadmap-and-phases.md
docs/06-development-guide.md
docs/07-security.md
docs/08-gap-analysis.md
docs/09-testing-strategy.md
docs/10-glossary.md
```

If a change introduces a new technical term, update:

```text
docs/10-glossary.md
```

---

# 20. Code Quality

Code should be:

* readable
* maintainable
* testable
* documented where necessary
* consistent with existing conventions
* explicit about failure cases

Avoid:

* unnecessary abstractions
* duplicated logic
* hidden global state
* magic constants
* dead code
* commented-out code
* unrelated refactoring

---

## 20.1 TypeScript

Prefer:

```typescript
type
interface
enum
readonly
strict typing
```

Avoid unnecessary:

```typescript
any
```

Handle errors explicitly.

---

## 20.2 Python

Follow project formatting and linting rules.

Prefer:

* type hints
* explicit error handling
* small functions
* testable services
* structured logging

---

## 20.3 C++

Sandbox code requires particular care.

Prefer:

* RAII
* explicit ownership
* bounds checking
* safe resource management
* deterministic cleanup
* defensive validation

Avoid undefined behavior and unsafe assumptions.

---

# 21. Pull Request Process

Before opening a PR:

```bash
git status
git diff
```

Run appropriate checks:

```bash
make test
make lint
make build
```

For security-sensitive changes:

```bash
make test-security
```

Check formatting:

```bash
make format
```

Commit:

```bash
git add .
git commit -m "feat: add execution history"
```

Push:

```bash
git push -u origin feature/execution-history
```

Then open a Pull Request.

---

# 22. Review Checklist

Every PR should answer:

### General

* [ ] Is the change clearly scoped?
* [ ] Is the implementation understandable?
* [ ] Are unnecessary changes excluded?
* [ ] Are tests included?
* [ ] Does documentation need updating?

### Security

* [ ] Is untrusted input handled safely?
* [ ] Is authorization enforced server-side?
* [ ] Can this affect tenant isolation?
* [ ] Can this affect sandbox isolation?
* [ ] Can this expose secrets?
* [ ] Can this increase resource consumption?
* [ ] Can this expose internal services?

### Execution

* [ ] Does execution use an immutable snapshot?
* [ ] Is a timeout enforced?
* [ ] Are resource limits applied?
* [ ] Are terminal states handled correctly?
* [ ] Are execution results bounded?

### API

* [ ] Are request schemas validated?
* [ ] Are response schemas correct?
* [ ] Are error responses consistent?
* [ ] Is backward compatibility preserved?
* [ ] Is idempotency handled where required?

### Database

* [ ] Are constraints correct?
* [ ] Are indexes appropriate?
* [ ] Is a migration included?
* [ ] Are cross-tenant relationships protected?

### Testing

* [ ] Unit tests pass
* [ ] Integration tests pass
* [ ] Security tests pass where applicable
* [ ] Regression tests added where applicable
* [ ] No known critical test failures remain

---

# 23. Breaking Changes

Breaking changes require explicit justification.

Examples:

```text
Removing API fields
Changing API semantics
Changing authentication behavior
Changing database contracts
Changing WebSocket event schemas
Changing gRPC contracts
Changing execution states
Changing resource policy semantics
```

Breaking changes should include:

1. Problem statement
2. Current behavior
3. New behavior
4. Migration strategy
5. Compatibility considerations
6. Documentation
7. Tests

For architectural decisions, create an ADR when appropriate.

Recommended future structure:

```text
docs/18-adr/
├── 0001-record-architecture-decision.md
├── 0002-execution-snapshot-strategy.md
├── 0003-sandbox-isolation.md
└── ...
```

---

# 24. Performance Changes

Performance optimizations must be evidence-based.

Before optimizing:

```text
Identify bottleneck
       ↓
Measure baseline
       ↓
Implement change
       ↓
Measure again
       ↓
Verify correctness
       ↓
Verify security
```

Do not trade away:

```text
Security
Correctness
Isolation
Reproducibility
Observability
```

for an unmeasured performance improvement.

Performance-sensitive areas include:

* execution queue
* Redis operations
* database queries
* WebSocket broadcasting
* snapshot creation
* sandbox startup
* compilation
* API latency
* collaboration events

---

# 25. Observability

Important operations should be observable.

Where appropriate, include:

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

Never log sensitive information such as:

```text
Passwords
Access tokens
Session tokens
Secrets
Private credentials
```

Avoid logging raw user source code unless explicitly required for a controlled debugging/security purpose.

---

# 26. Dependency Changes

Dependency additions should have a clear purpose.

Before adding a dependency, consider:

* maintenance status
* security history
* license
* package size
* transitive dependencies
* performance impact
* compatibility
* long-term maintenance

Avoid introducing a dependency for functionality that can be safely implemented using existing project capabilities.

Dependency upgrades should run the full relevant test suite.

Security-sensitive dependency changes may require additional review.

---

# 27. Reporting Security Issues

Do **not** publicly disclose a serious vulnerability before maintainers have had an opportunity to investigate and address it.

Security issues may include:

* sandbox escape
* privilege escalation
* cross-project data access
* authentication bypass
* authorization bypass
* secret exposure
* host filesystem access
* Docker socket access
* internal network access
* denial of service
* remote code execution outside the sandbox

When reporting a vulnerability, provide:

```text
Title
Affected component
Affected version/commit
Description
Reproduction steps
Expected behavior
Actual behavior
Security impact
Proof of concept, if appropriate
Suggested mitigation, if known
```

Do not include real credentials or private user data in a report.

---

# 28. Contributor Definition of Done

A contribution is considered complete when applicable requirements are satisfied:

```text
Implementation
     +
Unit Tests
     +
Integration Tests
     +
Security Validation
     +
Error Handling
     +
Logging / Observability
     +
Documentation
     +
Code Review
     +
CI Validation
```

For security-sensitive functionality:

```text
Implementation
     ↓
Tests
     ↓
Security Review
     ↓
Regression Tests
     ↓
Documentation
     ↓
CI
     ↓
Review
     ↓
Merge
```

A feature is not complete merely because the code compiles.

---

# 29. Final Engineering Principles

The following principles define how CodeForge Cloud should be developed.

### 1. Treat all user input as hostile

Browser input, source code, filenames, terminal output, WebSocket messages, and API payloads must be treated as untrusted.

### 2. Keep the Gateway outside the execution boundary

The Gateway handles requests. It does not execute user code.

### 3. Keep execution isolated

User workloads must execute inside controlled sandbox environments.

### 4. Make execution reproducible

Executions use immutable snapshots.

### 5. Enforce authorization server-side

The client is never the authority for permissions.

### 6. Use PostgreSQL as durable truth

Redis is used for transient infrastructure concerns, not as the authoritative source of durable application state.

### 7. Fail closed

Security failures should deny the operation rather than silently bypassing protection.

### 8. Make important behavior observable

Production systems require enough telemetry to understand failures and security events.

### 9. Test security invariants

Security is not only documentation. It must be verified through automated tests.

### 10. Document architectural decisions

Important decisions should be recorded so future contributors understand why the system works the way it does.

### 11. Prefer small changes

Small PRs are easier to review, test, debug, and safely deploy.

### 12. Turn important bugs into tests

A regression test should prevent the same class of failure from returning.

---

# Quick Contribution Workflow

```text
┌──────────────────────┐
│ Read Documentation   │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Create Feature Branch│
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Implement Change     │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Add / Update Tests   │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Security Review      │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Update Documentation │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Run CI Checks        │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Commit Changes       │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Push Branch          │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Open Pull Request    │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Code Review          │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Merge                │
└──────────────────────┘
```

---

# Thank You

Thank you for contributing to **CodeForge Cloud**.

Every contribution should make the platform:

* safer
* more reliable
* more reproducible
* more observable
* more maintainable
* easier to develop
* easier to operate
* easier to scale

> **Build carefully. Test everything. Secure the execution boundary.**

# CodeForge Cloud

## Testing Strategy & Quality Assurance Document

**Document:** 09 — Testing Strategy
**Project:** CodeForge Cloud
**Subtitle:** Collaborative Real-Time Online Code Compiler & IDE Sandbox
**Version:** 1.0
**Status:** Draft
**Author:** Adarsh Kumar

---

# 1. Document Overview

This document defines the complete testing strategy for CodeForge Cloud.

CodeForge Cloud is a distributed system that accepts source code from users and executes that code inside isolated sandbox environments.

Because arbitrary user code is executed by the platform, testing is not limited to conventional application testing.

The testing strategy must validate:

* functional correctness;
* API correctness;
* data integrity;
* distributed service communication;
* execution correctness;
* sandbox isolation;
* resource enforcement;
* authorization;
* collaboration consistency;
* failure handling;
* performance;
* observability;
* security invariants;
* deployment readiness.

The fundamental testing principle is:

> **Every security boundary and execution guarantee must be backed by an automated test whenever technically possible.**

---

# 2. Testing Objectives

The testing program has six primary objectives.

## 2.1 Functional Correctness

Verify that all supported product functionality behaves according to requirements.

## 2.2 Security

Verify that malicious users and malicious programs cannot bypass security controls.

## 2.3 Reliability

Verify that failures do not corrupt persistent state or permanently lose executions.

## 2.4 Performance

Measure system behavior under realistic workloads.

## 2.5 Compatibility

Ensure that APIs, protobuf contracts, database migrations, and frontend/backend interfaces remain compatible.

## 2.6 Reproducibility

Verify that an execution produces results based on its immutable snapshot rather than mutable project state.

---

# 3. Testing Philosophy

CodeForge Cloud follows a layered testing model:

```text
                    ┌───────────────────┐
                    │ Production Tests  │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │    E2E Tests      │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │ Integration Tests │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │    Unit Tests     │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │ Static Analysis   │
                    └───────────────────┘
```

Security testing cuts across every layer.

---

# 4. Test Pyramid

The project should follow a test pyramid.

```text
                 /\
                /  \
               / E2E\
              /------\
             /Integration\
            /------------\
           /  Unit Tests  \
          /----------------\
```

Recommended distribution:

| Test Type          | Approximate Share |
| ------------------ | ----------------: |
| Unit               |            60–70% |
| Integration        |            20–30% |
| End-to-End         |             5–10% |
| Manual/Exploratory |     Supplementary |

Security tests are additional and span multiple layers.

---

# 5. Test Categories

CodeForge Cloud uses the following test categories:

1. Unit testing
2. Component testing
3. API testing
4. Database testing
5. Redis testing
6. gRPC testing
7. WebSocket testing
8. Integration testing
9. End-to-end testing
10. Security testing
11. Sandbox testing
12. Resource-limit testing
13. Failure testing
14. Regression testing
15. Load testing
16. Stress testing
17. Soak testing
18. Compatibility testing
19. Deployment testing
20. Recovery testing

---

# 6. Test Environments

At minimum, the project should maintain:

```text
development
test
staging
production
```

## Development

Used for local feature development.

## Test

Used for automated unit and integration testing.

## Staging

Should resemble production as closely as practical.

## Production

Production testing must be restricted to safe health, smoke, and monitoring checks unless explicitly authorized.

---

# 7. Test Environment Isolation

Test environments must not share production credentials or production databases.

```text
Development DB
       X
Production DB

Test Redis
       X
Production Redis
```

Sandbox tests must use dedicated test workers.

---

# 8. Test Data Strategy

Test data should be deterministic.

Recommended categories:

```text
valid users
invalid users
projects
members
files
executions
snapshots
malicious source
large source
invalid paths
concurrent updates
```

Test data must be cleaned after each isolated test suite where practical.

---

# 9. Test IDs

Tests should use stable identifiers.

Example:

```text
AUTH-001
AUTH-002
PROJ-001
FILE-001
EXEC-001
SBOX-001
SEC-001
WS-001
LOAD-001
```

This makes test reports traceable to requirements and gaps.

---

# 10. Requirements Traceability

Every important requirement should map to at least one test.

```text
Requirement
     ↓
Test Case
     ↓
Automated Test
     ↓
CI Result
```

Example:

| Requirement                               | Test      |
| ----------------------------------------- | --------- |
| Unauthorized users cannot access projects | AUTHZ-001 |
| Execution uses immutable snapshot         | EXEC-010  |
| Infinite loops timeout                    | SBOX-004  |
| Output is bounded                         | SBOX-007  |
| Network is disabled                       | SEC-012   |

---

# 11. Unit Testing Strategy

Unit tests validate individual functions and modules without external infrastructure whenever possible.

Target components:

* validators;
* authentication;
* authorization;
* project service;
* file service;
* snapshot service;
* execution state machine;
* resource policy;
* result normalization;
* queue logic;
* protocol mapping.

---

# 12. Unit Test Requirements

Every unit test should:

* be deterministic;
* avoid network access;
* avoid real external services;
* use controlled inputs;
* test success and failure paths;
* run quickly.

---

# 13. Authentication Unit Tests

Required cases:

```text
AUTH-001 valid registration
AUTH-002 duplicate email
AUTH-003 duplicate username
AUTH-004 invalid email
AUTH-005 invalid username
AUTH-006 weak password
AUTH-007 password hashing
AUTH-008 successful login
AUTH-009 invalid credentials
AUTH-010 token expiration
AUTH-011 token revocation
```

---

# 14. Password Tests

Tests must verify:

* plaintext password is never persisted;
* password hashes are generated correctly;
* incorrect passwords fail;
* password comparison is performed safely;
* hashes are not returned through API responses.

---

# 15. Authorization Unit Tests

Required:

```text
AUTHZ-001 owner access
AUTHZ-002 editor access
AUTHZ-003 viewer read access
AUTHZ-004 viewer write denied
AUTHZ-005 unauthorized project denied
AUTHZ-006 cross-project file denied
AUTHZ-007 cross-project execution denied
AUTHZ-008 member-management restriction
```

---

# 16. Project Unit Tests

Test:

* creation;
* validation;
* update;
* deletion;
* visibility;
* ownership;
* membership;
* duplicate handling.

---

# 17. File Unit Tests

Test:

```text
FILE-001 create file
FILE-002 create directory
FILE-003 read file
FILE-004 update file
FILE-005 delete file
FILE-006 invalid filename
FILE-007 path traversal
FILE-008 oversized file
FILE-009 oversized project
FILE-010 invalid parent
FILE-011 cross-project parent
```

---

# 18. Path Validation Tests

The validator must reject:

```text
../secret
../../etc/passwd
/absolute/path
C:\Windows\System32
.
..
```

and equivalent normalized forms.

It must also detect malicious encoded representations where applicable.

---

# 19. Snapshot Unit Tests

Required:

```text
SNAP-001 snapshot creation
SNAP-002 snapshot contains required files
SNAP-003 snapshot hash generation
SNAP-004 deterministic hash
SNAP-005 file hash generation
SNAP-006 snapshot immutability
SNAP-007 invalid entry file
SNAP-008 oversized snapshot
```

---

# 20. Snapshot Determinism Test

The same project state should produce the same deterministic snapshot representation.

Example:

```text
Project A
  main.cpp
  src/util.cpp

Capture → Hash X

Capture again
  main.cpp
  src/util.cpp

Hash → X
```

File ordering must be deterministic.

---

# 21. Snapshot Mutation Test

After an execution is submitted:

```text
Project
   ↓
Snapshot A
   ↓
Execution
```

The user changes:

```text
main.cpp
```

The existing execution must continue using:

```text
Snapshot A
```

and not the new live file.

---

# 22. Execution State Machine Tests

Valid transitions:

```text
QUEUED → STARTING
STARTING → COMPILING
COMPILING → RUNNING
RUNNING → COMPLETED
```

Failure transitions:

```text
STARTING → FAILED
COMPILING → FAILED
RUNNING → FAILED
RUNNING → TIMEOUT
RUNNING → RESOURCE_LIMIT
QUEUED → CANCELLED
RUNNING → CANCELLED
```

---

# 23. Invalid State Transition Tests

The following must fail:

```text
COMPLETED → RUNNING
COMPLETED → QUEUED
FAILED → RUNNING
CANCELLED → RUNNING
TIMEOUT → RUNNING
```

Terminal states must be immutable.

---

# 24. Resource Policy Tests

Test:

* CPU limit;
* memory limit;
* timeout;
* process limit;
* disk limit;
* output limit;
* source limit.

Policies supplied by clients must never override server-enforced maximums.

---

# 25. API Testing Strategy

Every public API endpoint should have:

* happy-path test;
* validation test;
* authentication test;
* authorization test;
* malformed-input test;
* boundary test;
* error-response test.

---

# 26. Authentication API Tests

Endpoints:

```text
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/logout
GET  /api/v1/auth/me
```

Verify:

* status codes;
* response schema;
* headers;
* authentication behavior;
* validation errors;
* request IDs.

---

# 27. Project API Tests

Endpoints:

```text
POST   /api/v1/projects
GET    /api/v1/projects
GET    /api/v1/projects/{project_id}
PATCH  /api/v1/projects/{project_id}
DELETE /api/v1/projects/{project_id}
```

Test:

```text
valid request
missing fields
invalid UUID
unauthorized access
cross-project access
invalid role
duplicate operation
```

---

# 28. File API Tests

Endpoints:

```text
POST   /projects/{project_id}/files
POST   /projects/{project_id}/directories
GET    /projects/{project_id}/files
GET    /projects/{project_id}/files/{file_id}
PATCH  /projects/{project_id}/files/{file_id}
DELETE /projects/{project_id}/files/{file_id}
```

Verify project authorization for every request.

---

# 29. Execution API Tests

Endpoint:

```text
POST /api/v1/executions
```

Must test:

```text
valid execution
invalid language
missing entry file
invalid project
unauthorized project
oversized project
duplicate Idempotency-Key
invalid resource policy
```

Expected successful submission:

```text
202 Accepted
```

---

# 30. Idempotency Tests

Example:

```text
Request 1
Idempotency-Key = ABC
        ↓
Execution X

Request 2
Idempotency-Key = ABC
        ↓
Execution X
```

The second request must not create execution Y.

---

# 31. Concurrency Tests

Two clients updating the same file:

```text
Client A → revision 10
Client B → revision 10
```

Only one update should succeed if revision-based optimistic concurrency is used.

The stale request should receive:

```text
409 CONFLICT
```

---

# 32. API Error Tests

Verify standardized errors:

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

Never expose internal stack traces.

---

# 33. Database Testing

Database tests must verify:

* schema;
* constraints;
* foreign keys;
* unique indexes;
* check constraints;
* transactions;
* rollback behavior;
* migration correctness.

---

# 34. Database Constraint Tests

Required:

```text
duplicate email → rejected
duplicate username → rejected
duplicate membership → rejected
invalid project FK → rejected
invalid user FK → rejected
invalid file type → rejected
negative size → rejected
invalid execution status → rejected
duplicate execution result → rejected
```

---

# 35. Cross-Project Database Tests

Attempt:

```text
Project A
File A

Execution for Project B
entry_file_id = File A
```

The database must reject the relationship.

This validates the composite project/file integrity constraint.

---

# 36. Transaction Tests

Important operations must be atomic.

Execution submission:

```text
BEGIN
  create snapshot
  create execution job
  create outbox event
COMMIT
```

If any operation fails:

```text
ROLLBACK
```

No partial execution state should remain.

---

# 37. Migration Tests

Every migration should be tested against:

```text
empty database
latest schema
upgrade path
```

Production migrations must be reviewed before deployment.

---

# 38. Redis Testing

Redis tests must cover:

* queue operations;
* status cache;
* rate limits;
* presence;
* pub/sub;
* TTL;
* key namespace;
* failure behavior.

---

# 39. Redis Namespace Tests

Verify keys follow the expected pattern:

```text
codeforge:{domain}:{identifier}
```

Examples:

```text
codeforge:execution:<id>
codeforge:project:<id>:presence
codeforge:ratelimit:<user-id>
```

Cross-project key collisions must be impossible.

---

# 40. Queue Testing

Test:

```text
QUEUE-001 enqueue
QUEUE-002 dequeue
QUEUE-003 worker claim
QUEUE-004 worker timeout
QUEUE-005 retry
QUEUE-006 duplicate delivery
QUEUE-007 dead-letter
QUEUE-008 cancellation
```

---

# 41. Worker Recovery Test

Simulate:

```text
Worker claims job
       ↓
Worker crashes
       ↓
Lease expires
       ↓
Another worker claims job
```

The execution must not be permanently lost.

---

# 42. gRPC Testing

Test:

```text
Evaluator → Sandbox
Gateway → Evaluator
```

Verify:

* request schema;
* response schema;
* metadata;
* deadlines;
* error mapping;
* cancellation;
* retries;
* malformed messages.

---

# 43. gRPC Timeout Tests

A deliberately slow sandbox should cause:

```text
deadline exceeded
```

rather than an indefinitely hanging evaluator request.

---

# 44. gRPC Cancellation Tests

Cancellation must propagate:

```text
API
 ↓
Gateway
 ↓
Evaluator
 ↓
Sandbox
 ↓
Process
```

The process must terminate.

---

# 45. WebSocket Testing

Test:

```text
WS-001 authenticated connection
WS-002 unauthenticated connection
WS-003 authorized project join
WS-004 unauthorized project join
WS-005 file update
WS-006 presence
WS-007 cursor update
WS-008 reconnect
WS-009 duplicate event
WS-010 malformed event
```

---

# 46. WebSocket Authorization Test

User A attempts:

```text
join project:B
```

without membership.

Expected:

```text
connection/event denied
```

No project information should be leaked.

---

# 47. Collaboration Testing

Test simultaneous:

```text
file updates
cursor movement
selection changes
presence updates
```

Verify:

* ordering;
* conflict handling;
* revision consistency;
* reconnect behavior;
* duplicate-event handling.

---

# 48. Integration Testing

Integration tests run real service dependencies.

Example:

```text
Gateway
  ↓
PostgreSQL
  ↓
Redis
```

and:

```text
Evaluator
  ↓
Redis
  ↓
Sandbox
```

---

# 49. Full Execution Integration Test

Required sequence:

```text
Create user
 ↓
Login
 ↓
Create project
 ↓
Create file
 ↓
Submit execution
 ↓
Snapshot
 ↓
Queue
 ↓
Evaluator
 ↓
Sandbox
 ↓
Compile
 ↓
Execute
 ↓
Persist result
 ↓
Read result
```

This is the primary integration test.

---

# 50. Golden Path Test

Use:

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello, CodeForge!" << std::endl;
    return 0;
}
```

Expected:

```text
Hello, CodeForge!
```

The test must validate:

* successful compilation;
* successful execution;
* exit code `0`;
* correct stdout;
* no unexpected stderr;
* persisted result.

---

# 51. Compilation Failure Test

Example:

```cpp
int main()
{
    invalid syntax
}
```

Expected:

```text
COMPILATION FAILED
```

The API must remain healthy.

---

# 52. Runtime Failure Test

Example:

```cpp
#include <cstdlib>

int main()
{
    return 42;
}
```

Expected:

```text
exit_code = 42
status = COMPLETED
```

A nonzero program exit is not necessarily a platform failure.

---

# 53. Runtime Crash Test

Example:

```cpp
int main()
{
    int *p = nullptr;
    *p = 42;
}
```

Expected:

```text
execution terminated
signal recorded where supported
```

The sandbox must remain healthy.

---

# 54. Timeout Testing

Program:

```cpp
int main()
{
    while (true)
    {
    }
}
```

Expected:

```text
status = TIMEOUT
```

Verify:

* process terminated;
* child processes terminated;
* resources released;
* result persisted.

---

# 55. Memory Limit Testing

Program attempts excessive memory allocation.

Expected:

```text
status = RESOURCE_LIMIT
```

or the platform's defined memory-limit classification.

Host memory usage must remain controlled.

---

# 56. CPU Limit Testing

CPU-intensive program:

```cpp
int main()
{
    volatile unsigned long long x = 0;

    while (true)
    {
        ++x;
    }
}
```

Expected:

```text
CPU limit or timeout
```

depending on policy.

---

# 57. Process Limit Testing

A program attempting excessive process creation must be terminated when the configured process limit is reached.

Expected:

```text
RESOURCE_LIMIT
```

---

# 58. Output Limit Testing

Program generates unlimited output.

Expected:

```text
RESOURCE_LIMIT
output_truncated = true
```

The evaluator must not buffer unlimited output.

---

# 59. Disk Limit Testing

Program attempts to create excessive files/data.

Expected:

```text
RESOURCE_LIMIT
```

and the workspace must be cleaned afterward.

---

# 60. Network Isolation Testing

Program attempts outbound networking.

Expected:

```text
network unavailable
```

No connection should reach the public internet.

---

# 61. Internal Network Isolation Test

User code must not access:

```text
PostgreSQL
Redis
Gateway
Evaluator
Docker API
```

Any attempted connection must fail.

---

# 62. Filesystem Isolation Test

User code must not access arbitrary host files.

Attempts to access:

```text
/etc/passwd
/etc/shadow
host project directories
Docker socket
```

must fail.

---

# 63. Docker Socket Test

Inside the sandbox:

```text
/var/run/docker.sock
```

must not be available.

This is a mandatory security regression test.

---

# 64. Privilege Test

The user process must not run with unnecessary privileges.

Verify:

* effective UID;
* effective GID;
* capabilities;
* container privilege configuration.

`--privileged` must never be used for user execution.

---

# 65. Namespace Tests

Verify isolation of:

```text
PID
Mount
Network
IPC
UTS
User
```

where enabled by the deployment configuration.

---

# 66. Seccomp Tests

Verify that prohibited system calls are blocked according to the sandbox policy.

A blocked syscall should result in controlled process termination rather than host compromise.

---

# 67. Capability Tests

The container should drop unnecessary Linux capabilities.

Tests must verify the effective capability set.

---

# 68. Read-Only Filesystem Test

The container root filesystem should be read-only except for the controlled workspace.

Attempted writes outside the workspace must fail.

---

# 69. Symlink Escape Test

Create a malicious symlink:

```text
workspace/link → /etc/passwd
```

The execution system must prevent access outside the permitted workspace.

---

# 70. Path Traversal Security Tests

Test:

```text
../
..//
....//
encoded traversal
absolute paths
mixed separators
null-byte patterns
```

All unsafe paths must be rejected or safely normalized.

---

# 71. Command Injection Tests

Attempt to manipulate:

```text
compiler
entry_file
arguments
language
project metadata
```

No user-controlled value should become an arbitrary shell command.

---

# 72. SQL Injection Tests

Test common payload classes against:

* login;
* project queries;
* file queries;
* search/filter parameters.

All database operations must use parameterized queries or equivalent safe mechanisms.

---

# 73. XSS Testing

User-controlled content includes:

* project names;
* file names;
* display names;
* compiler output;
* error messages.

Example payload:

```html
<script>alert(1)</script>
```

It must never execute in the frontend.

---

# 74. Terminal Output Security

Compiler output is untrusted.

The terminal must render:

```text
<script>alert(1)</script>
```

as plain text.

ANSI/control sequences must be handled safely.

---

# 75. Authentication Abuse Testing

Test:

* repeated login failures;
* repeated registration;
* password guessing;
* token reuse;
* expired sessions;
* revoked sessions.

Expected controls:

```text
rate limiting
lockout/throttling where configured
generic errors
audit events
```

---

# 76. Account Enumeration Testing

Registration and login errors should not unnecessarily reveal whether an account exists.

Example:

```text
User does not exist
```

should not become a high-confidence enumeration oracle where avoidable.

---

# 77. Authorization Regression Tests

For every protected endpoint, test:

```text
owner
editor
viewer
non-member
unauthenticated
```

This should become a reusable authorization test matrix.

---

# 78. Cross-Project Isolation Tests

Create:

```text
User A
Project A
Project B
```

Attempt to access B using A's credentials without membership.

Verify denial for:

* project;
* files;
* execution;
* snapshot;
* execution result;
* collaboration room.

---

# 79. Snapshot Security Test

Verify:

```text
User modifies project
      ↓
Existing snapshot unchanged
      ↓
Execution result corresponds to old source
```

This prevents time-of-check/time-of-use inconsistencies.

---

# 80. Snapshot Integrity Test

Change snapshot contents after persistence.

The system should detect integrity mismatch using:

```text
snapshot_sha256
```

and refuse unsafe execution.

---

# 81. Resource Abuse Testing

The system must test:

```text
CPU abuse
Memory abuse
Process abuse
Disk abuse
Output abuse
Queue abuse
API abuse
WebSocket abuse
```

---

# 82. Queue Abuse Testing

A user should not be able to submit unlimited executions.

Test:

```text
normal usage
burst usage
sustained usage
multiple projects
multiple accounts
```

Verify queue quotas and rate limits.

---

# 83. WebSocket Abuse Testing

Test:

* rapid connection creation;
* rapid event publishing;
* oversized messages;
* malformed messages;
* unauthorized room joins.

---

# 84. Denial-of-Service Testing

Application-level DoS scenarios include:

```text
large project upload
large file upload
execution flood
login flood
WebSocket flood
output flood
process flood
memory flood
```

The platform should degrade gracefully.

---

# 85. Failure Injection Testing

Inject failures into:

```text
PostgreSQL
Redis
Gateway
Evaluator
Worker
Sandbox
Network
```

Verify defined failure behavior.

---

# 86. PostgreSQL Failure Test

During an execution request:

```text
Database unavailable
```

Expected:

```text
controlled error
no partial persistent state
```

---

# 87. Redis Failure Test

If Redis is unavailable:

* API should not falsely report execution success;
* durable state must remain safe;
* queue submission should fail or enter a clearly defined recovery state.

---

# 88. Evaluator Failure Test

Stop evaluator while jobs exist.

Expected behavior should follow the defined queue recovery strategy.

No execution should silently disappear.

---

# 89. Sandbox Failure Test

Terminate sandbox worker unexpectedly.

Expected:

```text
job recovery or FAILED state
```

according to the worker lease policy.

---

# 90. Network Partition Test

Simulate:

```text
Gateway ↔ Evaluator
Evaluator ↔ Sandbox
```

network interruption.

Verify:

* timeout;
* retry;
* state consistency;
* no duplicate execution.

---

# 91. Duplicate Delivery Test

Deliver the same execution message to a worker twice.

The system must prevent unsafe duplicate processing where the execution contract requires at-most-once behavior, or safely tolerate duplicate delivery through idempotent processing.

---

# 92. Cancellation Testing

Test cancellation in every major state:

```text
QUEUED
STARTING
COMPILING
RUNNING
```

Expected:

```text
CANCELLED
```

when cancellation is valid.

---

# 93. Cancellation Race Test

Simultaneously:

```text
execution completes
```

and:

```text
cancel request
```

The final state must follow the defined state transition rules and must not become corrupted.

---

# 94. Graceful Shutdown Testing

Send termination signal to:

* gateway;
* evaluator;
* worker;
* sandbox service.

Verify:

* no new work accepted after shutdown begins;
* active operations handled safely;
* connections closed;
* jobs recovered where necessary.

---

# 95. Regression Testing

Every security or correctness bug should become a permanent regression test.

Example:

```text
Bug discovered
      ↓
Fix implemented
      ↓
Regression test added
      ↓
CI permanently checks it
```

---

# 96. Test Naming Convention

Recommended:

```text
<component>_<behavior>_<expected_result>
```

Examples:

```text
auth_login_invalid_password_rejected
file_update_viewer_forbidden
execution_snapshot_remains_immutable
sandbox_infinite_loop_times_out
sandbox_network_access_denied
```

---

# 97. Test Directory Structure

Recommended:

```text
tests/
├── unit/
│   ├── gateway/
│   ├── evaluator/
│   └── sandbox/
├── integration/
│   ├── api/
│   ├── database/
│   ├── redis/
│   ├── grpc/
│   └── websocket/
├── e2e/
├── security/
│   ├── sandbox/
│   ├── authorization/
│   ├── injection/
│   └── isolation/
├── load/
├── stress/
├── fixtures/
└── scripts/
```

---

# 98. Frontend Testing

Frontend tests should cover:

* editor rendering;
* file navigation;
* save behavior;
* run execution;
* terminal output;
* execution state;
* API error handling;
* WebSocket connection;
* reconnection;
* collaboration updates.

---

# 99. Frontend Component Tests

Examples:

```text
Editor
FileTree
Terminal
RunButton
ExecutionStatus
ProjectSidebar
CollaborationPresence
ConnectionIndicator
```

Each should have isolated component tests.

---

# 100. Frontend Integration Tests

Test:

```text
Editor
 ↓
API
 ↓
Execution
 ↓
WebSocket
 ↓
Terminal
```

The UI should correctly reflect backend state.

---

# 101. Browser E2E Testing

Recommended user journey:

```text
Open application
 ↓
Register
 ↓
Login
 ↓
Create project
 ↓
Create file
 ↓
Edit C++
 ↓
Run
 ↓
Observe execution
 ↓
Read output
```

---

# 102. Browser Security Testing

Verify:

* secure transport;
* correct CORS;
* CSP;
* XSS defenses;
* WebSocket origin validation;
* authentication behavior;
* logout behavior.

---

# 103. Load Testing Strategy

Load testing should measure:

```text
API throughput
Execution throughput
Queue latency
Database performance
Redis performance
WebSocket connections
Sandbox startup
```

---

# 104. Load Test Levels

Start with:

```text
10 users
50 users
100 users
500 users
1000 users
```

Actual limits depend on infrastructure.

---

# 105. Execution Load Test

Example workload:

```text
10 concurrent executions
50 concurrent executions
100 concurrent executions
```

Measure:

* queue latency;
* execution latency;
* CPU;
* memory;
* failure rate.

---

# 106. Collaboration Load Test

Simulate:

```text
10 users/project
50 users/project
100 users/project
```

with realistic editor events.

Measure:

* WebSocket latency;
* event throughput;
* CPU;
* memory;
* conflict rate.

---

# 107. Stress Testing

Stress testing intentionally exceeds expected capacity.

Example:

```text
normal capacity
      ↓
2×
      ↓
5×
      ↓
10×
```

Observe:

* failure mode;
* recovery;
* queue behavior;
* database stability;
* resource exhaustion.

---

# 108. Soak Testing

Run the system for extended periods.

Suggested initial duration:

```text
1 hour
6 hours
24 hours
```

Monitor:

* memory leaks;
* connection leaks;
* queue growth;
* database growth;
* worker stability.

---

# 109. Performance Metrics

Record:

### API

```text
p50
p95
p99
```

### Execution

```text
queue latency
startup latency
compile latency
runtime latency
total execution latency
```

### Infrastructure

```text
CPU
memory
disk
network
```

---

# 110. Performance Baseline

Before optimization, establish baseline measurements.

Example:

| Metric            |   Baseline |
| ----------------- | ---------: |
| API p95           | To measure |
| Snapshot creation | To measure |
| Queue latency     | To measure |
| Sandbox startup   | To measure |
| C++ compile time  | To measure |
| Execution latency | To measure |

No performance target should be claimed before measurement.

---

# 111. Observability Testing

Verify:

* logs are emitted;
* request IDs propagate;
* execution IDs propagate;
* traces propagate;
* metrics increment;
* errors generate appropriate telemetry.

---

# 112. Log Security Tests

Automated checks should ensure logs do not contain:

```text
password
access token
session token
secret
private key
raw source
database credentials
```

where these values are not explicitly intended for logging.

---

# 113. Audit Testing

Verify audit events are created for:

```text
login
logout
project creation
project deletion
member changes
file changes
execution submission
execution cancellation
security violations
```

---

# 114. Health Check Testing

Every service should expose reliable:

```text
liveness
readiness
```

tests.

A service that cannot reach a required dependency should report readiness according to its dependency policy.

---

# 115. Container Testing

CI should inspect container images for:

* vulnerabilities;
* unnecessary packages;
* root execution;
* writable filesystem;
* exposed ports;
* dangerous capabilities;
* privileged configuration.

---

# 116. Container Runtime Tests

Verify:

```text
privileged = false
Docker socket = unavailable
host filesystem = unavailable
network = disabled
capabilities = restricted
filesystem = read-only where possible
resource limits = active
```

---

# 117. Image Security Testing

Every runtime image should undergo:

```text
image build
 ↓
vulnerability scan
 ↓
configuration scan
 ↓
SBOM generation
 ↓
policy validation
```

Production images should be pinned to controlled versions/digests.

---

# 118. Dependency Testing

Every CI run should check:

* vulnerable dependencies;
* outdated packages;
* lock-file consistency;
* dependency integrity.

---

# 119. Static Analysis

Required tools should cover:

### C++

```text
compiler warnings
clang-tidy where configured
clang/static analysis where configured
```

### Python

```text
ruff
mypy where applicable
```

### TypeScript

```text
ESLint
TypeScript compiler
```

Static analysis failures should block merges when configured as required gates.

---

# 120. Formatting Tests

CI should enforce formatting for:

```text
C++
Python
TypeScript
JSON
YAML
Markdown
```

Formatting drift should not be merged.

---

# 121. API Contract Testing

The API schema should be validated automatically.

Pipeline:

```text
OpenAPI
   ↓
Schema validation
   ↓
Generated/contract tests
   ↓
Integration tests
```

Breaking changes require explicit review.

---

# 122. Protobuf Compatibility Testing

Every protobuf change must verify:

* field numbers;
* field types;
* backwards compatibility;
* generated code;
* service methods;
* optional/required semantics.

Existing field numbers must not be reused for incompatible meanings.

---

# 123. Database Migration CI

CI should:

```text
create empty database
 ↓
apply migrations
 ↓
run schema tests
 ↓
run application integration tests
```

Migration failures must block releases.

---

# 124. Test Coverage

Coverage should be measured, but coverage percentage alone must not determine quality.

Recommended initial targets:

| Component              |                Target |
| ---------------------- | --------------------: |
| Gateway business logic |                 ≥ 80% |
| Evaluator              |                 ≥ 80% |
| Sandbox policy logic   |                 ≥ 85% |
| Frontend core logic    |                 ≥ 70% |
| Security-critical code | ≥ 90% where practical |

Critical paths should receive higher coverage than average code.

---

# 125. Security Coverage

Security testing should focus on behavior rather than only line coverage.

Examples:

```text
authorization bypass
sandbox escape
resource exhaustion
command injection
path traversal
SQL injection
XSS
network isolation
privilege escalation
```

---

# 126. CI Testing Pipeline

Recommended pipeline:

```text
Pull Request
     ↓
Formatting
     ↓
Lint
     ↓
Static Analysis
     ↓
Unit Tests
     ↓
Database Tests
     ↓
Integration Tests
     ↓
Security Tests
     ↓
Container Scan
     ↓
Build
     ↓
E2E
     ↓
Review
```

---

# 127. CI Test Gates

### Required for every PR

```text
format
lint
unit tests
build
```

### Required for protected branches

```text
integration
security
API contract
database migration
container validation
```

### Required before release

```text
full E2E
load
stress where appropriate
security regression
backup/restore
deployment smoke test
```

---

# 128. Test Failure Policy

A failing security test must block release.

A failing critical functional test must block release.

Flaky tests must not simply be disabled.

Instead:

```text
Flaky test
   ↓
Investigate
   ↓
Fix
   ↓
Restore as required gate
```

---

# 129. Flaky Test Management

Each flaky test should have:

```text
test ID
failure frequency
owner
root cause
tracking issue
resolution deadline
```

---

# 130. Test Reporting

Test reports should contain:

```text
total tests
passed
failed
skipped
duration
coverage
security failures
environment
commit SHA
build ID
```

---

# 131. Test Artifacts

CI should preserve:

* test reports;
* coverage reports;
* logs;
* security scan results;
* container scan results;
* E2E screenshots where applicable;
* performance results.

---

# 132. Test Data Cleanup

After tests:

```text
temporary projects
temporary files
temporary containers
temporary Redis keys
temporary databases
```

must be removed.

Sandbox workspaces require guaranteed cleanup.

---

# 133. Sandbox Cleanup Test

After every execution verify:

```text
execution finishes
      ↓
workspace removed
      ↓
container removed
      ↓
temporary resources released
```

No stale user workspace should remain.

---

# 134. Resource Leak Testing

Repeatedly execute jobs and verify:

```text
containers
processes
file descriptors
memory
disk
Redis keys
database connections
WebSocket connections
```

do not grow indefinitely.

---

# 135. Reproducibility Testing

Given:

```text
same snapshot
same compiler
same sandbox image
same policy
```

the system should produce reproducible execution behavior wherever deterministic execution is expected.

Environmental nondeterminism should be documented.

---

# 136. Execution Audit Chain Test

Verify the complete chain:

```text
request_id
   ↓
execution_job.id
   ↓
snapshot_id
   ↓
snapshot_sha256
   ↓
execution_result.id
```

Every production execution should be traceable.

---

# 137. Security Invariant Test Matrix

| Invariant                            | Test                          |
| ------------------------------------ | ----------------------------- |
| Code never executes in Gateway       | Architecture/integration test |
| Code never executes directly on host | Sandbox integration test      |
| Cross-project access denied          | Authorization test            |
| PostgreSQL inaccessible              | Sandbox network test          |
| Redis inaccessible                   | Sandbox network test          |
| Docker socket inaccessible           | Container test                |
| Host filesystem inaccessible         | Filesystem test               |
| Resource limits always active        | Sandbox test                  |
| Timeout always active                | Execution test                |
| Snapshot immutable                   | Snapshot test                 |
| Protected operations authorize       | AuthZ suite                   |
| Security failures fail closed        | Failure tests                 |

---

# 138. Threat-Based Test Matrix

| Threat               | Test Category  | Priority |
| -------------------- | -------------- | -------: |
| Sandbox escape       | Security       |       P0 |
| Fork bomb            | Resource       |       P0 |
| Memory exhaustion    | Resource       |       P0 |
| Output flood         | Resource       |       P0 |
| Path traversal       | Security       |       P0 |
| Command injection    | Security       |       P0 |
| Cross-project access | Authorization  |       P0 |
| Docker socket access | Isolation      |       P0 |
| Network access       | Isolation      |       P0 |
| SQL injection        | Security       |       P1 |
| XSS                  | Security       |       P1 |
| WebSocket abuse      | Security       |       P1 |
| Queue flooding       | DoS            |       P1 |
| Credential abuse     | Authentication |       P1 |

---

# 139. Test Priorities

## P0

Must pass before untrusted code execution:

```text
authorization
sandbox isolation
resource limits
timeout
filesystem isolation
network isolation
command injection
snapshot immutability
execution state machine
security regression suite
```

## P1

Required for MVP:

```text
authentication
project/file APIs
queue
evaluator
frontend
WebSocket
E2E
```

## P2

Important for production maturity:

```text
load
stress
soak
advanced observability
backup/restore
```

---

# 140. Test Execution Commands

Recommended root-level interface:

```bash
make test
make test-unit
make test-integration
make test-e2e
make test-security
make test-load
make coverage
```

Service-specific examples:

```bash
cd gateway/node
npm test
npm run lint
npm run build
```

```bash
cd evaluator/python
pytest
```

```bash
cd sandbox/cpp
cmake -S . -B build
cmake --build build
ctest --test-dir build
```

---

# 141. Local Golden Path

Developers should be able to run:

```bash
make up
make test
```

and verify the complete system.

The golden path should be documented and maintained as a release gate.

---

# 142. Test Automation Principle

Manual testing is useful for exploration, but critical guarantees must be automated.

Especially:

```text
authorization
sandbox isolation
resource limits
snapshot immutability
API contracts
database constraints
```

must not depend solely on manual verification.

---

# 143. Release Testing

Before release:

```text
Build
 ↓
Unit Tests
 ↓
Integration
 ↓
Security
 ↓
E2E
 ↓
Load
 ↓
Container Scan
 ↓
Migration Test
 ↓
Backup/Restore
 ↓
Deployment Smoke Test
```

---

# 144. Release Smoke Tests

After deployment verify:

```text
health endpoints
login
project creation
file creation
execution submission
C++ execution
result retrieval
WebSocket connection
```

---

# 145. Rollback Testing

Deployment rollback should be tested.

Verify:

```text
new version
   ↓
failure
   ↓
rollback
   ↓
old version
   ↓
database compatibility
```

Database migrations must account for rollback strategy.

---

# 146. Production Monitoring Tests

After release verify:

* logs visible;
* metrics visible;
* traces visible;
* alerts functional;
* dashboards populated;
* execution status updates correctly.

---

# 147. Incident Simulation

Periodically simulate:

```text
sandbox compromise
worker crash
Redis outage
PostgreSQL outage
execution flood
WebSocket flood
credential abuse
```

Verify response procedures.

---

# 148. Security Regression Program

Every discovered security vulnerability should result in:

```text
Vulnerability
      ↓
Root Cause
      ↓
Fix
      ↓
Regression Test
      ↓
CI Gate
      ↓
Documentation Update
```

---

# 149. Testing Anti-Patterns

Do not:

```text
ignore failing security tests
disable flaky tests permanently
test only happy paths
trust mocks for sandbox isolation
run untrusted tests on production
skip authorization tests
skip failure testing
measure only average latency
ignore resource leaks
depend only on code coverage
```

---

# 150. Mocking Policy

Mocks are appropriate for:

* external APIs;
* infrastructure failures;
* isolated unit tests.

Mocks are insufficient for:

* sandbox isolation;
* container security;
* actual database constraints;
* actual Redis queue behavior;
* end-to-end execution.

Those require real integration environments.

---

# 151. Sandbox Test Environment

Sandbox tests must run in an environment that closely resembles the real execution environment.

At minimum:

```text
Linux
Docker
resource controls
network isolation
restricted capabilities
sandbox image
```

Security tests should not rely solely on a fake sandbox.

---

# 152. Test Isolation for Malicious Code

Malicious execution tests themselves must be isolated.

Never execute sandbox-escape test payloads directly on the development host.

Recommended:

```text
Dedicated test host
        ↓
Dedicated Docker environment
        ↓
Sandbox tests
```

---

# 153. Security Test Safety

Security testing must remain controlled.

Test cases should be designed to validate defenses without introducing uncontrolled impact to the host or surrounding infrastructure.

Particularly sensitive tests should run only inside dedicated disposable environments.

---

# 154. Test Lifecycle

Every feature should follow:

```text
Design
 ↓
Test Cases
 ↓
Implementation
 ↓
Unit Tests
 ↓
Integration Tests
 ↓
Security Tests
 ↓
E2E
 ↓
Documentation
 ↓
Release
```

---

# 155. Definition of Test Complete

A feature is test-complete only when:

```text
✓ Happy path tested
✓ Error path tested
✓ Boundary tested
✓ Authorization tested
✓ Security implications tested
✓ Integration tested
✓ Regression test available
✓ Observability verified
✓ Documentation updated
```

---

# 156. Definition of Security Test Complete

A security-critical feature requires:

```text
✓ Threat identified
✓ Security control implemented
✓ Positive test
✓ Negative test
✓ Abuse test
✓ Regression test
✓ Automated CI execution
```

---

# 157. MVP Test Completion Criteria

The MVP test suite is complete when:

```text
All critical unit tests pass
        AND
All required integration tests pass
        AND
Golden path passes
        AND
Authorization suite passes
        AND
Sandbox security suite passes
        AND
Resource-limit suite passes
        AND
Snapshot tests pass
        AND
No P0 security issues remain
```

---

# 158. Production Test Completion Criteria

Production readiness additionally requires:

```text
Security regression suite passes
        AND
Container scans pass
        AND
Dependency scans pass
        AND
Load testing completed
        AND
Failure testing completed
        AND
Backup/restore tested
        AND
Deployment smoke tests pass
        AND
Rollback procedure verified
```

---

# 159. Recommended Testing Roadmap

## Phase 1 — Test Foundation

```text
test directories
test runners
fixtures
CI
coverage
reporting
```

## Phase 2 — Core Unit Tests

```text
auth
projects
files
snapshot
execution state
```

## Phase 3 — API Tests

```text
authentication
projects
members
files
execution
```

## Phase 4 — Infrastructure Tests

```text
PostgreSQL
Redis
gRPC
```

## Phase 5 — Sandbox Tests

```text
compile
run
timeout
memory
CPU
process
filesystem
network
```

## Phase 6 — Security Tests

```text
authorization
injection
isolation
escape
DoS
```

## Phase 7 — E2E

```text
complete user journey
```

## Phase 8 — Performance

```text
load
stress
soak
```

---

# 160. Test-to-Gap Mapping

The testing strategy directly closes the major gaps from `08-gap-analysis.md`.

| Gap             | Test Coverage          |
| --------------- | ---------------------- |
| Authentication  | Auth unit/API          |
| Authorization   | AuthZ matrix           |
| Files           | File unit/API/security |
| Snapshots       | Snapshot tests         |
| Execution state | State-machine tests    |
| Queue recovery  | Worker tests           |
| Evaluator       | Integration tests      |
| Sandbox         | Security/integration   |
| Resource limits | Abuse tests            |
| WebSocket       | WS integration         |
| Collaboration   | Concurrency tests      |
| Observability   | Telemetry tests        |
| CI/CD           | Automated test gates   |
| Performance     | Load/stress/soak       |
| Production      | Smoke/recovery tests   |

---

# 161. Quality Gates

The project should use four quality gates.

## Gate 1 — Code Quality

```text
Formatting
Lint
Static analysis
Unit tests
```

## Gate 2 — Functional Quality

```text
Integration
API contract
E2E
```

## Gate 3 — Security Quality

```text
AuthZ
Sandbox
Resource
Isolation
Dependency
Container
```

## Gate 4 — Production Quality

```text
Load
Failure
Recovery
Backup
Deployment
Rollback
```

---

# 162. Final Testing Architecture

```text
                       CodeForge Cloud Testing
                                │
          ┌─────────────────────┼─────────────────────┐
          │                     │                     │
          ▼                     ▼                     ▼
     Code Quality          Functional             Security
          │                     │                     │
     ┌────┼────┐          ┌─────┼─────┐        ┌─────┼─────┐
     ▼    ▼    ▼          ▼     ▼     ▼        ▼     ▼     ▼
   Lint Static Unit      API   Int   E2E      Auth  Sandbox Isolation
   Format Analysis       Tests Tests Tests
          │                     │                     │
          └─────────────────────┼─────────────────────┘
                                ▼
                         Performance
                                │
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
                  Load       Stress       Soak
                                │
                                ▼
                         Production Gates
```

---

# 163. Final Testing Principles

CodeForge Cloud testing follows these principles:

1. Test security boundaries, not only business logic.
2. Test failure paths, not only success paths.
3. Treat user source code as hostile test input.
4. Never execute malicious tests directly on the host.
5. Test authorization for every protected resource.
6. Test immutable snapshots independently.
7. Test resource limits using real sandbox environments.
8. Test service failures explicitly.
9. Convert every important bug into a regression test.
10. Use CI to continuously enforce critical guarantees.
11. Measure performance instead of guessing.
12. Never treat code coverage as the only measure of quality.
13. Keep production smoke tests safe and deterministic.
14. Preserve test evidence for releases.
15. Block releases on unresolved P0 security failures.

---

# 164. Final Test Strategy Summary

The CodeForge Cloud testing strategy is built around one central requirement:

> **The platform must prove that arbitrary user code can be processed safely, predictably, and reproducibly.**

The highest-priority validation areas are:

```text
Authorization
     ↓
Immutable Snapshot
     ↓
Queue Reliability
     ↓
Sandbox Isolation
     ↓
Resource Enforcement
     ↓
Execution Correctness
     ↓
Result Persistence
```

The testing program therefore gives special priority to:

```text
P0 Security Tests
P0 Sandbox Tests
P0 Resource Tests
P0 Authorization Tests
P0 Snapshot Tests
P0 Execution Tests
```

before advanced features are considered complete.

---

# 165. Document Status

**Document:** 09 — Testing Strategy
**Version:** 1.0
**Status:** Draft
**Author:** Adarsh Kumar

This document defines the testing, validation, quality-gate, security-testing, performance-testing, and release-validation strategy for CodeForge Cloud.

The strategy should be updated whenever:

* architecture changes;
* execution behavior changes;
* sandbox policy changes;
* API contracts change;
* new languages are introduced;
* new security controls are introduced;
* infrastructure changes;
* new vulnerabilities are discovered.

**Final principle:**

> **If a critical security or execution guarantee cannot be demonstrated by a repeatable test, the guarantee should not be considered proven.**

---

# 166. Next Document

The next documentation artifact is:

```text
10-glossary.md
```

It should define the terminology used throughout the CodeForge Cloud documentation set, including:

* execution;
* execution job;
* snapshot;
* sandbox;
* evaluator;
* gateway;
* worker;
* CRDT;
* WebSocket;
* gRPC;
* resource policy;
* namespace;
* cgroup;
* seccomp;
* capability;
* project;
* membership;
* role;
* execution state;
* idempotency;
* optimistic concurrency;
* audit event;
* request ID;
* job ID;
* execution ID;
* snapshot hash;
* tenant isolation;
* defense in depth;
* and other platform-specific terms.

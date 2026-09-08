# CodeForge Cloud

## Glossary & Terminology Reference

**Document:** 10 — Glossary
**Project:** CodeForge Cloud
**Subtitle:** Collaborative Real-Time Online Code Compiler & IDE Sandbox
**Version:** 1.0
**Status:** Draft
**Author:** Adarsh Kumar

---

# 1. Document Overview

This document defines the terminology used throughout the CodeForge Cloud technical documentation.

CodeForge Cloud is a distributed, cloud-native development platform that allows users to:

* create projects;
* create and edit source files;
* collaborate in real time;
* submit source code for execution;
* compile supported languages;
* execute programs inside isolated sandboxes;
* inspect execution output;
* maintain execution history.

Because the platform combines frontend, backend, distributed systems, databases, networking, container isolation, and security controls, consistent terminology is essential.

This glossary provides a common vocabulary for:

* developers;
* architects;
* testers;
* security engineers;
* DevOps engineers;
* reviewers;
* contributors;
* future maintainers.

---

# 2. Glossary Conventions

Terms are organized alphabetically where practical.

Each definition provides:

* **Term**
* **Definition**
* **CodeForge Context**
* **Related Concepts**

---

# 3. A

## 3.1 Access Token

A credential presented by a client to authenticate API requests.

### CodeForge Context

The frontend uses an access token when communicating with protected Gateway endpoints.

Example:

```http
Authorization: Bearer <access_token>
```

### Related Concepts

* Authentication
* Authorization
* Session
* Token
* API Gateway

---

## 3.2 API

Application Programming Interface.

A defined interface through which software components communicate.

### CodeForge Context

CodeForge Cloud exposes a versioned REST API through the Node.js Gateway.

Example:

```text
/api/v1/projects
/api/v1/files
/api/v1/executions
```

### Related Concepts

* REST
* Endpoint
* Gateway
* API Contract

---

## 3.3 API Contract

A formal definition of an API's:

* endpoints;
* methods;
* parameters;
* request schemas;
* response schemas;
* error behavior;
* authentication requirements.

### CodeForge Context

The API contract is documented in:

```text
docs/04-api-reference.md
```

### Related Concepts

* OpenAPI
* REST
* Schema
* Backward Compatibility

---

## 3.4 API Gateway

The public-facing backend service that receives client requests and routes them to internal services.

### CodeForge Context

The Node.js Gateway is the public application boundary.

It handles:

* authentication;
* authorization;
* project APIs;
* file APIs;
* execution submission;
* WebSocket connections.

It must never execute user source code.

### Related Concepts

* Node.js Gateway
* Evaluator
* Reverse Proxy
* Security Boundary

---

## 3.5 Authentication

The process of verifying the identity of a user or service.

### Example

```text
Username + Password
        ↓
Authentication
        ↓
Authenticated User
```

### Related Concepts

* Authorization
* Session
* Access Token
* Identity

---

## 3.6 Authorization

The process of determining whether an authenticated identity has permission to perform an operation.

### Example

```text
User
 ↓
Authenticated
 ↓
Project Membership Checked
 ↓
Permission Granted/Denied
```

### CodeForge Context

Authorization is enforced server-side.

### Related Concepts

* Authentication
* RBAC
* Project Member
* Least Privilege

---

# 4. B

## 4.1 Backend

The server-side portion of CodeForge Cloud.

It consists primarily of:

* Node.js Gateway;
* Python Evaluator;
* supporting workers;
* databases;
* Redis;
* sandbox infrastructure.

### Related Concepts

* Gateway
* Evaluator
* Worker
* Backend Service

---

## 4.2 Bearer Token

An access credential where possession of the token grants the associated authentication context.

### CodeForge Context

Bearer tokens are sent using:

```http
Authorization: Bearer <token>
```

Tokens must be protected from leakage.

---

# 5. C

## 5.1 Cancellation

A request to stop an execution before it naturally completes.

### CodeForge Context

Cancellation may occur while an execution is:

```text
QUEUED
STARTING
COMPILING
RUNNING
```

### Related Concepts

* Execution
* Execution State
* Timeout
* Worker

---

## 5.2 Capability

A Linux security mechanism that grants a process a specific privileged operation without giving it full root privileges.

### CodeForge Context

Sandbox containers should drop unnecessary Linux capabilities.

### Related Concepts

* Linux
* Container
* Privilege
* Sandbox

---

## 5.3 Cgroups

Linux Control Groups.

A Linux kernel mechanism used to control and monitor resource consumption by processes.

### CodeForge Context

Cgroups are used to enforce execution limits such as:

* CPU;
* memory;
* process count.

### Related Concepts

* CPU Limit
* Memory Limit
* Resource Policy
* Sandbox

---

## 5.4 CI

Continuous Integration.

An automated process that builds and tests changes whenever code is submitted.

### CodeForge Context

CI should execute:

```text
format
lint
static analysis
unit tests
integration tests
security tests
build
```

### Related Concepts

* CD
* Pipeline
* Quality Gate
* Regression Test

---

## 5.5 CRDT

Conflict-Free Replicated Data Type.

A data structure designed to allow distributed replicas to make changes independently while converging toward a consistent state.

### CodeForge Context

CRDTs are a preferred model for real-time collaborative editing.

### Related Concepts

* Collaboration
* Real-Time Editing
* WebSocket
* Distributed State

---

## 5.6 Compiler

A program that transforms source code into executable or intermediate machine code.

### CodeForge Context

C++ source code is compiled inside the sandbox.

The compiler must never execute outside the controlled execution environment for user workloads.

---

## 5.7 Container

A lightweight isolated process environment that packages an application and its dependencies.

### CodeForge Context

Docker containers provide an additional isolation boundary around user code.

Containers are not treated as the only security boundary.

### Related Concepts

* Docker
* Sandbox
* Namespace
* cgroup

---

## 5.8 Container Image

A packaged filesystem and metadata used to create a container.

### CodeForge Context

CodeForge execution workers use controlled runtime/compiler images.

Production images should be:

* versioned;
* scanned;
* minimized;
* controlled;
* preferably pinned by digest.

---

## 5.9 Concurrency

Multiple operations executing or progressing during overlapping periods.

### CodeForge Context

Concurrency appears in:

* collaborative editing;
* multiple execution jobs;
* worker processing;
* API requests.

### Related Concepts

* Race Condition
* Optimistic Concurrency
* CRDT
* Queue

---

## 5.10 CORS

Cross-Origin Resource Sharing.

A browser security mechanism controlling which origins can access server resources.

### CodeForge Context

The Gateway must use an explicit CORS policy.

Wildcard access should not be used carelessly for authenticated operations.

---

## 5.11 CPU Limit

The maximum CPU resource an execution is permitted to consume.

### CodeForge Context

CPU limits protect the host from CPU exhaustion caused by malicious or inefficient programs.

---

# 6. D

## 6.1 Database

A persistent system used to store structured application data.

### CodeForge Context

PostgreSQL is the primary durable database.

It stores:

* users;
* projects;
* memberships;
* files;
* executions;
* snapshots;
* results;
* sessions;
* audit events.

---

## 6.2 Defense in Depth

A security strategy using multiple independent security controls.

### CodeForge Example

```text
Validation
   ↓
Authentication
   ↓
Authorization
   ↓
Resource Limits
   ↓
Container
   ↓
Namespaces
   ↓
cgroups
   ↓
seccomp
   ↓
Capability Restrictions
   ↓
Network Isolation
```

Failure of one layer should not automatically compromise the system.

---

## 6.3 Deployment

The process of installing and running a software version in a target environment.

### CodeForge Context

Initial deployment uses Docker Compose.

Future production deployment may use Kubernetes.

---

## 6.4 Docker

A platform for building and running containerized applications.

### CodeForge Context

Docker is used to:

* run CodeForge services;
* isolate execution workloads;
* provide reproducible development environments.

---

# 7. E

## 7.1 Editor

The source-code editing interface inside the Web IDE.

### CodeForge Context

Monaco Editor is the planned editor technology.

---

## 7.2 Endpoint

A specific API route that performs an operation.

Example:

```text
POST /api/v1/executions
```

---

## 7.3 Execution

The process of compiling and/or running a user's source code.

### CodeForge Context

An execution is created from an immutable snapshot.

```text
Project
 ↓
Snapshot
 ↓
Execution Job
 ↓
Sandbox
 ↓
Compile
 ↓
Run
 ↓
Result
```

---

## 7.4 Execution ID

A unique identifier for an execution.

### CodeForge Context

It is used to retrieve:

* execution status;
* execution result;
* execution snapshot.

Example:

```text
execution_id = UUID
```

---

## 7.5 Execution Job

A persistent representation of a requested execution.

### CodeForge Context

The `execution_jobs` PostgreSQL table stores:

* project;
* user;
* entry file;
* language;
* status;
* snapshot;
* resource policy;
* timestamps.

---

## 7.6 Execution Result

The final output and metadata produced by an execution.

It may include:

* stdout;
* stderr;
* exit code;
* signal;
* compile time;
* runtime;
* memory usage;
* resource violation;
* truncation information.

---

## 7.7 Execution Snapshot

An immutable representation of the project state used for one execution.

### Fundamental Rule

> An execution runs against its snapshot, not mutable live project state.

### Related Concepts

* Snapshot
* SHA-256
* Reproducibility
* Execution Job

---

## 7.8 Execution State

The current lifecycle state of an execution.

Supported states include:

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

---

## 7.9 Evaluator

The Python service responsible for coordinating execution.

Responsibilities include:

* validation;
* scheduling;
* queue interaction;
* routing;
* sandbox invocation;
* result normalization.

The evaluator does not replace the sandbox.

---

# 8. F

## 8.1 Fail Closed

A security principle where an operation is denied when the system cannot safely determine that it should be allowed.

### Example

If authorization cannot be verified:

```text
Authorization failure
        ↓
DENY
```

not:

```text
Authorization failure
        ↓
ALLOW
```

---

## 8.2 File

A source-code or project resource stored inside a project.

### CodeForge Context

A file contains:

* name;
* path;
* type;
* content;
* size;
* project association.

---

## 8.3 File Descriptor

An operating-system handle representing an open file, socket, pipe, or similar resource.

### CodeForge Context

File descriptors are considered a resource that may need monitoring to prevent resource exhaustion.

---

## 8.4 File Tree

The hierarchical structure of files and directories inside a project.

Example:

```text
project/
├── main.cpp
├── include/
│   └── math.hpp
└── src/
    └── math.cpp
```

---

# 9. G

## 9.1 Gateway

See **API Gateway**.

---

## 9.2 gRPC

A high-performance remote procedure call framework.

### CodeForge Context

gRPC is used for internal service communication.

Example:

```text
Gateway
   ↓
Evaluator
   ↓
Sandbox
```

### Related Concepts

* Protocol Buffers
* RPC
* Internal Service

---

## 9.3 Graceful Shutdown

A controlled service shutdown that allows active operations to finish or recover safely.

### CodeForge Context

Services should:

* stop accepting new work;
* finish or cancel appropriate operations;
* close connections;
* release resources.

---

# 10. H

## 10.1 Health Check

An endpoint or mechanism used to determine whether a service is functioning.

Common types:

```text
liveness
readiness
```

---

## 10.2 Host

The machine or operating-system environment running CodeForge infrastructure.

### CodeForge Context

User code must never be trusted on the host.

---

# 11. I

## 11.1 IDE

Integrated Development Environment.

A software environment combining code editing and development tools.

### CodeForge Context

The CodeForge Web IDE provides:

* editor;
* file explorer;
* terminal;
* execution controls;
* collaboration.

---

## 11.2 Idempotency

A property where repeating the same operation produces the same intended result rather than creating unintended duplicates.

### CodeForge Context

Execution submission supports:

```http
Idempotency-Key: <unique-key>
```

---

## 11.3 Idempotency Key

A client-provided identifier used to prevent duplicate processing of retryable requests.

### Example

```text
Request A
Idempotency-Key: ABC123
       ↓
Execution X

Retry
Idempotency-Key: ABC123
       ↓
Execution X
```

---

## 11.4 Immutable

Something that cannot be modified after creation.

### CodeForge Context

An execution snapshot is immutable.

---

## 11.5 Integration Test

A test that verifies multiple components working together.

Example:

```text
Gateway
 ↓
PostgreSQL
 ↓
Redis
 ↓
Evaluator
 ↓
Sandbox
```

---

## 11.6 Internal Service

A service not directly exposed to public clients.

### CodeForge Context

Examples:

* Evaluator;
* Sandbox service;
* internal workers.

---

# 12. J

## 12.1 Job

A unit of work submitted to an asynchronous processing system.

### CodeForge Context

Execution jobs represent user code execution requests.

---

# 13. K

## 13.1 Kubernetes

A container orchestration platform.

### CodeForge Context

Kubernetes is a future production deployment target.

Potential components include:

* Deployments;
* Jobs;
* NetworkPolicies;
* Pod Security Standards;
* worker pools.

---

# 14. L

## 14.1 Least Privilege

A security principle where a component receives only the permissions necessary to perform its function.

### CodeForge Example

The sandbox should not receive:

* Docker socket access;
* host filesystem access;
* database credentials;
* Redis credentials;
* unnecessary Linux capabilities.

---

## 14.2 Liveness

A health property indicating that a process is alive.

A liveness failure may cause an orchestrator to restart the service.

---

## 14.3 Load Test

A performance test that measures system behavior under expected workload.

---

# 15. M

## 15.1 Memory Limit

The maximum memory available to an execution.

### Purpose

Prevents malicious or inefficient programs from exhausting host memory.

---

## 15.2 Membership

The relationship between a user and a project.

Supported roles include:

```text
OWNER
EDITOR
VIEWER
```

---

## 15.3 Monaco Editor

The browser-based code editor technology used by the Web IDE.

It is the editor technology underlying Visual Studio Code's editing experience.

---

# 16. N

## 16.1 Namespace

A Linux kernel isolation mechanism that separates process or system resources.

Relevant namespaces include:

```text
PID
Mount
Network
IPC
UTS
User
```

### CodeForge Context

Namespaces form part of the sandbox defense-in-depth architecture.

---

## 16.2 Network Isolation

A security control preventing user workloads from communicating with unauthorized networks or services.

### CodeForge Default

User execution networking is disabled initially.

---

## 16.3 Node.js Gateway

The TypeScript/Node.js public backend service.

Primary responsibilities:

```text
HTTP API
Authentication
Authorization
Projects
Files
Execution submission
WebSocket
Collaboration
```

---

# 17. O

## 17.1 Object-Level Authorization

Authorization performed for the specific resource being accessed.

### Example

It is not enough to verify:

```text
User is authenticated
```

The server must also verify:

```text
User can access Project X
```

---

## 17.2 Optimistic Concurrency

A concurrency-control strategy where updates proceed assuming conflicts are uncommon and are rejected when the version has changed.

### CodeForge Example

```text
Client revision = 10

Server revision = 11

Update
 ↓
409 Conflict
```

---

## 17.3 Outbox Pattern

A reliability pattern where a database transaction stores both application state and an event to be published later.

### CodeForge Example

```text
Transaction
 ├── execution_job
 └── outbox_event
          ↓
     Outbox Worker
          ↓
        Redis
```

This reduces the risk of losing events between database and messaging systems.

---

# 18. P

## 18.1 PostgreSQL

The primary relational database used by CodeForge Cloud.

It is the durable source of truth for core application state.

---

## 18.2 Presence

Information describing which collaborators are currently connected to a project.

Examples:

```text
online
offline
editing
cursor position
```

---

## 18.3 Project

A logical workspace containing source files, metadata, collaborators, and execution history.

---

## 18.4 Project ID

A UUID identifying a project.

---

## 18.5 Project Member

A user associated with a project with a specific role.

Roles:

```text
OWNER
EDITOR
VIEWER
```

---

## 18.6 Protocol Buffers

A language-neutral data serialization and interface-definition format developed by Google.

### CodeForge Context

Protocol Buffers define internal gRPC messages.

---

## 18.7 Public API

An API exposed to the frontend or external clients.

### CodeForge Context

The public API is exposed through the Node.js Gateway.

---

# 19. Q

## 19.1 Queue

A mechanism that stores work until a worker can process it.

### CodeForge Context

Execution requests are queued before sandbox processing.

```text
API
 ↓
Queue
 ↓
Worker
```

---

## 19.2 Queue Latency

The time between an execution entering the queue and processing beginning.

---

# 20. R

## 20.1 RBAC

Role-Based Access Control.

An authorization model where permissions depend on assigned roles.

### CodeForge Roles

```text
OWNER
EDITOR
VIEWER
```

---

## 20.2 Readiness

A health state indicating whether a service is ready to receive traffic.

---

## 20.3 Redis

An in-memory data store used for transient and high-speed platform operations.

### CodeForge Uses

* queues;
* caching;
* rate limiting;
* presence;
* pub/sub;
* transient execution state.

Redis is not the durable source of truth.

---

## 20.4 Regression Test

A test ensuring that previously fixed functionality remains correct after future changes.

---

## 20.5 Resource Limit

A maximum amount of a resource that an execution may consume.

Examples:

```text
CPU
Memory
Processes
Disk
Output
Wall-clock time
```

---

## 20.6 Resource Policy

The server-controlled configuration defining limits for an execution.

Example:

```json id="j4u6sa"
{
  "cpu_limit": "...",
  "memory_limit": "...",
  "timeout_ms": "...",
  "process_limit": "...",
  "disk_limit": "...",
  "output_limit": "..."
}
```

---

## 20.7 REST

Representational State Transfer.

An architectural style commonly used for HTTP APIs.

### CodeForge Context

Public API endpoints use REST-style HTTP operations.

---

## 20.8 Reverse Proxy

A server that receives client requests and forwards them to backend services.

### CodeForge Context

Nginx acts as the initial reverse proxy.

Example:

```text
Browser
   ↓
Nginx
   ├── React
   ├── Gateway
   └── WebSocket
```

---

# 21. S

## 21.1 Sandbox

An isolated environment where untrusted user code is executed.

### CodeForge Context

The sandbox combines multiple controls:

```text
Container
Namespaces
cgroups
seccomp
Capabilities
Filesystem restrictions
Network isolation
Timeouts
```

---

## 21.2 Sandbox Escape

A security failure where code running inside the sandbox accesses resources outside its authorized boundary.

Examples include:

* host filesystem access;
* host process access;
* Docker socket access;
* unauthorized network access.

This is a P0 security concern.

---

## 21.3 Sandbox Runtime

The low-level component responsible for creating and controlling an execution environment.

### CodeForge Context

The C++ sandbox layer is responsible for security-critical low-level execution behavior.

---

## 21.4 Schema

A formal description of the structure and constraints of data.

Examples:

* database schema;
* API schema;
* protobuf schema;
* JSON schema.

---

## 21.5 seccomp

Secure Computing Mode.

A Linux security facility that restricts the system calls available to a process.

### CodeForge Context

seccomp forms part of sandbox hardening.

---

## 21.6 Session

A server-recognized authenticated period associated with a user.

### CodeForge Context

Session records contain token hashes rather than raw tokens.

---

## 21.7 SHA-256

A cryptographic hash function producing a 256-bit digest.

### CodeForge Context

Used for:

* file hashes;
* snapshot integrity;
* reproducibility tracking.

---

## 21.8 Snapshot

An immutable representation of project state captured for execution.

See **Execution Snapshot**.

---

## 21.9 Source Code

Human-readable program instructions written in a programming language.

### CodeForge Context

Source code is treated as untrusted input.

---

## 21.10 Source Snapshot

The exact source files captured when an execution is submitted.

### Important Rule

The sandbox executes the persisted snapshot, not the current project contents.

---

## 21.11 Static Analysis

Automated analysis of source code without executing it.

Examples:

* linting;
* type checking;
* compiler diagnostics;
* security analysis.

---

## 21.12 Status

A representation of the current lifecycle state of an entity.

### Execution Example

```text
QUEUED
RUNNING
COMPLETED
```

---

## 21.13 Stress Test

A performance test that intentionally pushes the system beyond normal expected load.

---

# 22. T

## 22.1 Tenant

An isolated logical customer or ownership boundary within a multi-tenant system.

### CodeForge Context

Projects provide the primary logical isolation boundary.

---

## 22.2 Tenant Isolation

Ensuring one user's/project's resources cannot be accessed by another unauthorized user or project.

---

## 22.3 Timeout

A maximum amount of time an operation is permitted to run.

### CodeForge Context

Every execution must have a wall-clock timeout.

---

## 22.4 Token

A credential used to represent authenticated access.

### Related Concepts

* Access Token
* Session
* Authentication

---

## 22.5 Trace ID

A unique identifier used to correlate distributed operations across services.

Example:

```text
Gateway
  trace_id = X
      ↓
Evaluator
  trace_id = X
      ↓
Sandbox
  trace_id = X
```

---

# 23. U

## 23.1 UUID

Universally Unique Identifier.

A 128-bit identifier commonly represented as:

```text
550e8400-e29b-41d4-a716-446655440000
```

### CodeForge Context

UUIDs are used for major persistent entities.

They reduce predictable identifier enumeration but do not replace authorization.

---

## 23.2 Untrusted Input

Any data that originates outside a trusted security boundary.

### CodeForge Examples

* source code;
* file names;
* project names;
* API parameters;
* WebSocket messages;
* compiler output.

---

## 23.3 User Workload

A program or execution submitted by a CodeForge user.

User workloads are considered hostile by default.

---

# 24. V

## 24.1 Viewer

A project member role with read-oriented permissions.

A viewer should not be able to modify project contents unless permissions are explicitly expanded.

---

## 24.2 Vulnerability

A weakness that could allow unintended behavior, unauthorized access, data exposure, or compromise.

---

# 25. W

## 25.1 Wall-Clock Time

The actual elapsed time from the beginning of an operation to its end.

### CodeForge Context

Execution timeout is based on wall-clock duration.

---

## 25.2 Web IDE

The browser-based development environment provided by CodeForge Cloud.

Major components:

```text
File Explorer
Monaco Editor
Terminal
Run Controls
Execution Status
Collaboration
```

---

## 25.3 WebSocket

A protocol providing persistent bidirectional communication between client and server.

### CodeForge Context

WebSockets support:

* collaboration;
* execution events;
* presence;
* cursor updates;
* live status.

---

## 25.4 Worker

A process responsible for consuming queued jobs and performing background work.

### CodeForge Context

Execution workers obtain jobs and invoke sandbox execution.

---

# 26. X

## 26.1 XSS

Cross-Site Scripting.

A vulnerability where untrusted content is interpreted as executable browser code.

### CodeForge Risk

Potential attack sources include:

* project names;
* filenames;
* usernames;
* compiler output;
* terminal output.

All user-controlled content must be safely rendered.

---

# 27. Z

No platform-specific Z terms are currently required.

---

# 28. CodeForge-Specific Terminology

The following terms have special meaning inside this project.

| Term             | CodeForge Meaning                              |
| ---------------- | ---------------------------------------------- |
| CodeForge Cloud  | Complete online collaborative coding platform  |
| Gateway          | Public Node.js/TypeScript application boundary |
| Evaluator        | Python execution orchestration service         |
| Sandbox          | Isolated execution environment                 |
| Worker           | Background job processor                       |
| Execution        | A submitted compile/run operation              |
| Execution Job    | Persistent execution request                   |
| Snapshot         | Immutable source state used for execution      |
| Resource Policy  | Server-controlled execution limits             |
| Project          | Workspace containing files and members         |
| Project Member   | User associated with project                   |
| Web IDE          | Browser-based development environment          |
| Collaboration    | Real-time multi-user editing                   |
| Execution Result | Output and metadata from a completed execution |

---

# 29. Execution Terminology

The following terms should not be confused.

## Execution Request

The API request submitted by the client.

```text
POST /api/v1/executions
```

## Execution Job

The persisted representation of that request.

## Execution Snapshot

The immutable project state captured for the job.

## Execution

The actual processing of the source code.

## Execution Result

The resulting output and metadata.

## Execution ID

Identifier used to reference the execution.

## Job ID

Identifier associated with the queued execution job.

---

# 30. Security Terminology

The following concepts are fundamental to CodeForge security:

```text
Authentication
Authorization
Least Privilege
Defense in Depth
Sandbox
Namespace
cgroup
seccomp
Capability Restriction
Network Isolation
Filesystem Isolation
Resource Limit
Timeout
Immutable Snapshot
Fail Closed
```

These controls work together rather than independently.

---

# 31. Distributed-System Terminology

Important distributed-system terms include:

```text
Queue
Worker
Retry
Timeout
Idempotency
Concurrency
Optimistic Concurrency
Outbox
Pub/Sub
WebSocket
gRPC
Trace ID
Request ID
```

---

# 32. Data Terminology

Important data concepts include:

```text
User
Project
Member
Role
File
Execution Job
Execution Snapshot
Execution Result
Session
Audit Event
Resource Policy
```

---

# 33. Execution State Terminology

The execution lifecycle is:

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

Failure states include:

```text
FAILED
TIMEOUT
CANCELLED
RESOURCE_LIMIT
```

Terminal states must not transition back into active execution states.

---

# 34. Security Boundary Terminology

A security boundary is a point where trust changes.

CodeForge contains several boundaries:

```text
Browser
   ↓
Nginx
   ↓
Gateway
   ↓
Evaluator
   ↓
Sandbox
   ↓
User Process
```

The most important boundary is:

```text
Trusted Platform
        │
        │
        ▼
Untrusted User Code
```

---

# 35. Trusted vs Untrusted Components

| Component          | Trust Classification       |
| ------------------ | -------------------------- |
| Browser            | Untrusted                  |
| User Source Code   | Hostile/Untrusted          |
| Project File Input | Untrusted                  |
| WebSocket Payload  | Untrusted                  |
| API Request        | Untrusted                  |
| Gateway            | Trusted Platform Component |
| Evaluator          | Trusted Platform Component |
| Sandbox Controller | Highly Security-Sensitive  |
| PostgreSQL         | Trusted Infrastructure     |
| Redis              | Trusted Infrastructure     |
| Host OS            | Trusted Infrastructure     |

The classification does not mean trusted services require no security controls.

---

# 36. Source of Truth Terminology

## PostgreSQL

Primary durable source of truth.

## Redis

Transient/high-speed state and messaging system.

## Snapshot

Historical execution input.

## Execution Result

Historical execution output.

The live project state must not be reconstructed from Redis or execution snapshots.

---

# 37. Identifier Terminology

CodeForge uses several identifiers.

```text
user_id
project_id
file_id
job_id
execution_id
snapshot_id
request_id
trace_id
```

They serve different purposes.

---

# 38. Request ID vs Trace ID

## Request ID

Identifies an individual external API request.

## Trace ID

Identifies a distributed operation across multiple services.

Example:

```text
API Request
request_id = R1
trace_id   = T1

Gateway
trace_id = T1

Evaluator
trace_id = T1

Sandbox
trace_id = T1
```

---

# 39. Job ID vs Execution ID

A job ID identifies the persisted work item.

An execution ID identifies the externally addressable execution.

They may map one-to-one initially but should remain conceptually distinct.

---

# 40. Snapshot Hash Terminology

`SHA-256` may be used at multiple levels:

```text
File
 ↓
file_sha256

Snapshot
 ↓
snapshot_sha256
```

The aggregate snapshot hash should be deterministic.

A recommended construction is:

```text
sorted(path + file_sha256)
        ↓
canonical representation
        ↓
SHA-256
        ↓
snapshot_sha256
```

---

# 41. Resource Terminology

The execution resource model includes:

| Resource    | Purpose                       |
| ----------- | ----------------------------- |
| CPU         | Prevent processor exhaustion  |
| Memory      | Prevent RAM exhaustion        |
| Processes   | Prevent process explosion     |
| Disk        | Prevent storage exhaustion    |
| Output      | Prevent log/buffer exhaustion |
| Wall-clock  | Prevent indefinite execution  |
| Source size | Prevent oversized input       |

---

# 42. Failure Terminology

## FAILED

An execution or service operation encountered an unrecoverable failure.

## TIMEOUT

Execution exceeded the configured wall-clock limit.

## CANCELLED

Execution was intentionally stopped.

## RESOURCE_LIMIT

Execution violated a configured resource constraint.

These states should be distinguished in API responses and observability.

---

# 43. Testing Terminology

Important testing terms include:

```text
Unit Test
Component Test
Integration Test
End-to-End Test
Security Test
Regression Test
Load Test
Stress Test
Soak Test
Smoke Test
Contract Test
```

---

# 44. Quality Terminology

## Quality Gate

A required validation step before code or a release can proceed.

## Regression

A previously working behavior that becomes broken.

## Test Coverage

A measurement of how much code or behavior is exercised by tests.

Coverage is useful but is not equivalent to correctness.

---

# 45. Deployment Terminology

## Development

Local developer environment.

## Test

Automated validation environment.

## Staging

Production-like validation environment.

## Production

Live user environment.

---

# 46. Infrastructure Terminology

CodeForge uses:

```text
Nginx
Docker
PostgreSQL
Redis
Node.js
Python
C++
gRPC
Protocol Buffers
WebSockets
```

These components have distinct responsibilities.

---

# 47. Service Responsibility Terminology

| Service    | Primary Responsibility  |
| ---------- | ----------------------- |
| Nginx      | Reverse proxy/routing   |
| Web IDE    | User interface          |
| Gateway    | Public API/auth/authz   |
| Evaluator  | Execution orchestration |
| Worker     | Background processing   |
| Sandbox    | Secure code execution   |
| PostgreSQL | Durable state           |
| Redis      | Queue/cache/pub-sub     |
| Docker     | Container isolation     |

---

# 48. Architecture Principles Terminology

Several terms represent explicit architectural principles.

## Security by Design

Security requirements are included during architecture and implementation rather than added later.

## Separation of Concerns

Each service has a focused responsibility.

## Contract First

Service interfaces are explicitly defined before implementation changes.

## Disposable Execution

Execution environments are temporary and destroyed after use.

## Observable by Default

Important operations emit logs, metrics, and tracing information.

## Fail Closed

Security failures deny access rather than silently allowing operations.

---

# 49. Reproducibility Terminology

Reproducibility means being able to understand or reproduce an execution from its recorded inputs and environment.

Important metadata includes:

```text
snapshot_id
snapshot_sha256
compiler version
runtime version
sandbox image digest
evaluator version
resource policy
```

---

# 50. Collaboration Terminology

## Collaboration Session

A real-time connection between a user and a project.

## Presence

Information about connected users.

## Cursor Update

A message describing a user's editor cursor position.

## Selection Update

A message describing selected editor content.

## File Update

A collaborative change to file content.

## Revision

A version marker used to detect concurrent changes.

---

# 51. API Terminology

Common HTTP terms:

| Term   | Meaning                                  |
| ------ | ---------------------------------------- |
| GET    | Retrieve resource                        |
| POST   | Create/submit operation                  |
| PATCH  | Partially update resource                |
| DELETE | Remove resource                          |
| 200    | Successful request                       |
| 201    | Resource created                         |
| 202    | Accepted for asynchronous processing     |
| 204    | Successful request without response body |
| 400    | Invalid request                          |
| 401    | Unauthenticated                          |
| 403    | Forbidden                                |
| 404    | Resource not found                       |
| 409    | Conflict                                 |
| 413    | Payload too large                        |
| 422    | Validation failure                       |
| 429    | Rate limited                             |
| 500    | Internal server error                    |
| 503    | Service unavailable                      |
| 504    | Gateway timeout                          |

---

# 52. Database Terminology

## Primary Key

Uniquely identifies a database row.

## Foreign Key

References another table's key.

## Unique Constraint

Prevents duplicate values.

## Check Constraint

Restricts values to a valid condition.

## Transaction

A group of database operations executed atomically.

## Migration

A version-controlled schema change.

---

# 53. Redis Terminology

## Queue

Stores pending execution work.

## Pub/Sub

Distributes transient messages to subscribers.

## TTL

Time To Live; controls automatic expiration of temporary data.

## Cache

Temporary copy of data used for faster access.

---

# 54. Container Security Terminology

Important terms:

```text
Image
Container
Namespace
cgroup
Capability
seccomp
Read-Only Root Filesystem
Ephemeral Workspace
Network Namespace
```

These mechanisms together form the sandbox security architecture.

---

# 55. Threat Terminology

## Threat

A potential source of harm.

## Attack

An attempt to exploit a weakness.

## Vulnerability

A weakness that can be exploited.

## Risk

The potential impact and likelihood associated with a threat.

## Control

A mechanism that reduces risk.

---

# 56. Commonly Confused Terms

## Authentication vs Authorization

Authentication:

> Who are you?

Authorization:

> What are you allowed to do?

---

## Snapshot vs File

A file is part of mutable project state.

A snapshot is an immutable historical execution input.

---

## Queue vs Worker

A queue stores work.

A worker processes work.

---

## Container vs Sandbox

A container is an isolation mechanism.

A sandbox is the complete controlled execution environment.

---

## Redis vs PostgreSQL

Redis provides transient/high-speed state.

PostgreSQL provides durable persistent state.

---

## Timeout vs Resource Limit

Timeout limits elapsed time.

Resource limits restrict resource consumption.

---

## Request ID vs Execution ID

Request ID identifies an API request.

Execution ID identifies an execution.

---

## gRPC vs REST

REST is primarily the public HTTP API style.

gRPC is primarily used for internal service communication.

---

# 57. Abbreviations

| Abbreviation | Full Form                           |
| ------------ | ----------------------------------- |
| API          | Application Programming Interface   |
| CI           | Continuous Integration              |
| CD           | Continuous Delivery/Deployment      |
| CORS         | Cross-Origin Resource Sharing       |
| CPU          | Central Processing Unit             |
| CRDT         | Conflict-Free Replicated Data Type  |
| CRUD         | Create, Read, Update, Delete        |
| DB           | Database                            |
| DoS          | Denial of Service                   |
| E2E          | End-to-End                          |
| IDE          | Integrated Development Environment  |
| IPC          | Inter-Process Communication         |
| JSON         | JavaScript Object Notation          |
| JWT          | JSON Web Token                      |
| RBAC         | Role-Based Access Control           |
| REST         | Representational State Transfer     |
| RPC          | Remote Procedure Call               |
| SRS          | Software Requirements Specification |
| SDD          | Software Design Document            |
| TLS          | Transport Layer Security            |
| TTL          | Time To Live                        |
| UUID         | Universally Unique Identifier       |
| WebSocket    | WebSocket Protocol                  |
| XSS          | Cross-Site Scripting                |

---

# 58. CodeForge Core Vocabulary

The following vocabulary should be preferred consistently throughout source code and documentation:

```text
User
Project
ProjectMember
Role
File
Directory
ExecutionJob
ExecutionResult
ExecutionSnapshot
ResourcePolicy
Session
AuditEvent
Gateway
Evaluator
Worker
Sandbox
ExecutionState
RequestId
TraceId
JobId
ExecutionId
SnapshotId
```

Avoid introducing multiple names for the same concept without a strong reason.

---

# 59. Naming Consistency Rules

Use:

```text
execution
```

instead of alternating between:

```text
run
job
process
task
```

when referring to the complete user code execution concept.

Use:

```text
snapshot
```

for immutable execution input.

Use:

```text
job
```

when specifically referring to queued work.

Use:

```text
sandbox
```

for the isolated execution environment.

Use:

```text
worker
```

for the component processing queued work.

---

# 60. Documentation Terminology Rules

All documentation should:

1. Use the definitions in this glossary.
2. Avoid unexplained acronyms.
3. Use consistent names for services.
4. Distinguish execution jobs from execution results.
5. Distinguish authentication from authorization.
6. Distinguish containers from complete sandbox architecture.
7. Distinguish PostgreSQL from Redis responsibilities.
8. Use `snapshot` consistently for immutable execution input.
9. Use `resource policy` for server-enforced limits.
10. Use `Gateway` for the public backend boundary.

---

# 61. Architectural Terminology Map

```text
                         CodeForge Cloud
                                │
                  ┌─────────────┴─────────────┐
                  │                           │
               Web IDE                    Gateway
                  │                           │
             Collaboration             REST / WebSocket
                                              │
                                              ▼
                                          Evaluator
                                              │
                                              ▼
                                            Queue
                                              │
                                              ▼
                                            Worker
                                              │
                                              ▼
                                           Sandbox
                                              │
                                      ┌───────┴───────┐
                                      │               │
                                   Compile          Run
                                      │               │
                                      └───────┬───────┘
                                              ▼
                                           Result
```

---

# 62. Data Terminology Map

```text
User
 │
 └── Project
      │
      ├── Project Member
      │
      ├── File
      │
      └── Execution Job
             │
             ├── Snapshot
             │
             ├── Resource Policy
             │
             └── Execution Result
```

---

# 63. Security Terminology Map

```text
Untrusted User
      │
      ▼
Input Validation
      │
      ▼
Authentication
      │
      ▼
Authorization
      │
      ▼
Snapshot Validation
      │
      ▼
Resource Policy
      │
      ▼
Sandbox
      │
      ├── Container
      ├── Namespace
      ├── cgroup
      ├── seccomp
      ├── Capabilities
      ├── Filesystem Isolation
      └── Network Isolation
```

---

# 64. Execution Terminology Map

```text
Live Project
     │
     ▼
Snapshot Capture
     │
     ▼
Execution Job
     │
     ▼
Queue
     │
     ▼
Worker
     │
     ▼
Sandbox
     │
     ├── Compile
     │
     └── Execute
           │
           ▼
       Result Capture
           │
           ▼
      Execution Result
```

---

# 65. Observability Terminology Map

```text
Request
  │
  ├── request_id
  │
  ├── trace_id
  │
  ├── user_id
  │
  ├── project_id
  │
  └── execution_id
           │
           ▼
      Logs / Metrics / Traces
```

---

# 66. Testing Terminology Map

```text
Code Change
    │
    ▼
Static Analysis
    │
    ▼
Unit Test
    │
    ▼
Integration Test
    │
    ▼
Security Test
    │
    ▼
E2E Test
    │
    ▼
Load / Stress Test
    │
    ▼
Release
```

---

# 67. Recommended Canonical Terms

When designing new components, prefer the following canonical terms.

| Concept                        | Canonical Term   |
| ------------------------------ | ---------------- |
| Public backend                 | Gateway          |
| Code orchestration             | Evaluator        |
| Background processor           | Worker           |
| Isolated execution environment | Sandbox          |
| User workspace                 | Project          |
| Source resource                | File             |
| Immutable execution input      | Snapshot         |
| Execution request              | Execution Job    |
| Execution output               | Execution Result |
| Limits                         | Resource Policy  |
| Real-time connection           | WebSocket        |
| Internal RPC                   | gRPC             |
| Durable database               | PostgreSQL       |
| Transient data/queue           | Redis            |

---

# 68. Terms to Avoid

Avoid ambiguous names such as:

```text
runner
executor
engine
processor
task
context
data
session
state
manager
```

unless the term is clearly qualified.

Prefer:

```text
Sandbox Runtime
Execution Job
Execution State
Project Session
Resource Policy
Execution Result
```

where appropriate.

---

# 69. Glossary Governance

The glossary should be updated when:

* a new service is introduced;
* a new execution state is introduced;
* a new API concept is introduced;
* a new security mechanism is introduced;
* terminology changes;
* a new programming language is supported;
* a new infrastructure component is introduced.

---

# 70. Documentation Consistency

The following documents should use this glossary as their terminology reference:

```text
01-product-requirements.md
02-architecture.md
03-data-model.md
04-api-reference.md
05-roadmap-and-phases.md
06-development-guide.md
07-security.md
08-gap-analysis.md
09-testing-strategy.md
10-glossary.md
```

Terminology conflicts between documents should be resolved by updating the relevant document and this glossary.

---

# 71. Final Glossary Principles

CodeForge Cloud follows these terminology principles:

1. One concept should have one canonical name.
2. Security terminology must be precise.
3. Execution terminology must distinguish jobs, snapshots, and results.
4. Authentication and authorization must never be treated as the same concept.
5. Containers must not be described as the entire sandbox security model.
6. Redis must not be described as the durable source of truth.
7. User source code must always be treated as untrusted.
8. Snapshot must always imply immutable execution input.
9. Resource policy must mean server-enforced execution limits.
10. Gateway must remain the public application boundary.
11. Evaluator must remain the execution orchestration layer.
12. Sandbox must remain the security-critical execution boundary.
13. Worker must mean background processing component.
14. Documentation and source code should use the same canonical terminology.

---

# 72. Final Terminology Summary

The core CodeForge Cloud architecture can be expressed using the following vocabulary:

```text
User
  ↓
Web IDE
  ↓
Gateway
  ↓
Authentication + Authorization
  ↓
Project
  ↓
Execution Request
  ↓
Immutable Snapshot
  ↓
Execution Job
  ↓
Queue
  ↓
Worker
  ↓
Evaluator
  ↓
Sandbox
  ↓
Container + Linux Isolation
  ↓
Compile
  ↓
Execute
  ↓
Resource Enforcement
  ↓
Execution Result
  ↓
Web IDE Terminal
```

The most important distinction in the entire platform is:

> **Live project state is mutable; an execution snapshot is immutable.**

The most important security distinction is:

> **User code is untrusted; the platform must execute it only inside the controlled sandbox boundary.**

The most important reliability distinction is:

> **PostgreSQL is the durable source of truth; Redis provides transient and high-speed infrastructure capabilities.**

---

# 73. Document Status

**Document:** 10 — Glossary
**Version:** 1.0
**Status:** Draft
**Author:** Adarsh Kumar

This document completes the initial CodeForge Cloud documentation set:

```text
docs/
├── 01-product-requirements.md
├── 02-architecture.md
├── 03-data-model.md
├── 04-api-reference.md
├── 05-roadmap-and-phases.md
├── 06-development-guide.md
├── 07-security.md
├── 08-gap-analysis.md
├── 09-testing-strategy.md
├── 10-glossary.md
└── README.md
```

The glossary should remain a living document and should evolve together with the architecture and implementation.

**Final principle:**

> **Consistent terminology produces consistent architecture, consistent APIs, consistent tests, and maintainable software.**

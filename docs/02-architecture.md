# CodeForge Cloud

## System Architecture Document

**Document:** 02 — Architecture
**Project:** CodeForge Cloud
**Subtitle:** Collaborative Real-Time Online Code Compiler & IDE Sandbox
**Version:** 1.1
**Status:** Draft
**Author:** Adarsh Kumar

---

# 1. Architecture Overview

CodeForge Cloud is a cloud-native, distributed online development platform that provides users with a browser-based IDE for writing, compiling, executing, and collaborating on source code.

The architecture is intentionally divided into independent services so that:

* untrusted code execution remains isolated;
* frontend and backend responsibilities remain separated;
* execution workloads can scale independently;
* real-time collaboration can operate independently from compilation;
* internal services communicate through strongly typed contracts;
* persistent data is separated from transient execution state;
* security boundaries can be enforced at multiple layers.

The primary architectural model is:

```text
                         ┌─────────────────────────┐
                         │       Web Browser        │
                         │                         │
                         │ React + TypeScript      │
                         │ Monaco Editor           │
                         │ File Explorer           │
                         │ Terminal / Console      │
                         └────────────┬────────────┘
                                      │
                          HTTPS / WebSocket
                                      │
                                      ▼
                    ┌─────────────────────────────────┐
                    │        Node.js Gateway          │
                    │                                 │
                    │ REST API                        │
                    │ Authentication                  │
                    │ Authorization                   │
                    │ Project Management              │
                    │ Collaboration Gateway          │
                    │ Execution API                   │
                    └───────┬───────────┬─────────────┘
                            │           │
                  PostgreSQL│           │Redis
                            │           │
                            ▼           ▼
                    ┌────────────┐  ┌────────────┐
                    │ PostgreSQL │  │   Redis    │
                    │            │  │            │
                    │ Users      │  │ Queue      │
                    │ Projects   │  │ Cache      │
                    │ Files      │  │ Pub/Sub    │
                    │ Jobs       │  │ Sessions   │
                    └────────────┘  └─────┬──────┘
                                          │
                                      Job Queue
                                          │
                                          ▼
                              ┌─────────────────────┐
                              │ Python Evaluator    │
                              │                     │
                              │ Validation          │
                              │ Scheduling          │
                              │ Language Routing    │
                              │ Result Normalizing │
                              └──────────┬──────────┘
                                         │
                                    gRPC / Protobuf
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │ C++ Sandbox Runtime │
                              │                     │
                              │ Isolation           │
                              │ Resource Limits     │
                              │ Compilation         │
                              │ Execution           │
                              │ Output Capture      │
                              └──────────┬──────────┘
                                         │
                                  Docker Isolation
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │ Ephemeral Execution │
                              │ Container           │
                              │                     │
                              │ Compiler            │
                              │ User Program        │
                              │ Restricted FS       │
                              └─────────────────────┘
```

---

# 2. Architectural Goals

The architecture must satisfy the following goals.

## 2.1 Security

User-submitted source code is considered untrusted.

The architecture must prevent user code from directly accessing:

* the host filesystem;
* host processes;
* host devices;
* privileged kernel interfaces;
* internal service credentials;
* databases;
* Redis;
* other users' projects;
* unrestricted network resources.

Security is therefore implemented as a layered defense rather than relying on a single mechanism.

---

## 2.2 Isolation

Compilation and execution must occur outside the main API process.

The execution path is:

```text
User
  ↓
Gateway
  ↓
Evaluator
  ↓
Sandbox
  ↓
Ephemeral Container
  ↓
Compiler / Program
```

The Node.js Gateway must never directly execute arbitrary user commands.

---

## 2.3 Scalability

Each service should be independently scalable.

For example:

```text
Gateway:
1 → N instances

Evaluator:
1 → N workers

Sandbox Workers:
1 → N workers
```

Execution workloads should not consume resources belonging to the API or collaboration layer.

---

## 2.4 Reliability

Failure in one service should not unnecessarily bring down the entire platform.

Examples:

* sandbox failure → execution job fails;
* evaluator failure → queued jobs remain recoverable;
* collaboration failure → code execution can continue;
* compiler failure → user receives a structured compilation error.

---

## 2.5 Observability

Every major operation should produce structured logs and measurable metrics.

Important operations include:

* API requests;
* authentication;
* project access;
* collaboration sessions;
* execution jobs;
* queue latency;
* compilation time;
* execution time;
* resource-limit violations;
* sandbox creation;
* sandbox destruction;
* service failures.

---

# 3. Explicit Architecture Assumptions

This section records the assumptions under which the initial CodeForge Cloud architecture is designed.

These assumptions are important because architectural decisions depend on them. If an assumption changes, the affected architecture must be reviewed rather than silently modified.

---

## 3.1 Deployment Assumptions

### A1 — Linux-Based Infrastructure

The execution infrastructure is assumed to run on Linux.

The sandbox architecture depends on Linux isolation primitives such as:

* namespaces;
* cgroups;
* seccomp;
* Linux capabilities;
* process limits;
* filesystem permissions.

The initial development environment is therefore expected to be Linux-compatible.

---

### A2 — Docker Is Available

Docker is assumed to be available on the execution host.

Docker provides the initial isolation boundary for user programs.

The architecture does not assume that Docker alone provides complete security.

Additional sandbox controls are expected to be added.

---

### A3 — Initial Deployment Is Single-Host

The first working deployment is assumed to run on a single machine using Docker Compose.

Example:

```text
Single Host
│
├── Nginx
├── Frontend
├── Gateway
├── Evaluator
├── Redis
├── PostgreSQL
└── Sandbox Runtime
```

This assumption simplifies development and integration.

The service boundaries must nevertheless remain suitable for later multi-host deployment.

---

### A4 — Future Deployment May Be Distributed

The architecture assumes that production workloads may eventually span multiple machines.

Therefore:

* services must not rely on local process memory for persistent state;
* services should communicate through explicit APIs;
* execution jobs should be recoverable;
* persistent data should use external storage;
* gateway instances should be stateless where practical.

---

# 3.2 Execution Assumptions

### A5 — User Code Is Fully Untrusted

All submitted code must be considered potentially malicious.

The system must assume that a user may intentionally attempt to:

* access the filesystem;
* consume excessive CPU;
* allocate excessive memory;
* create many processes;
* fork repeatedly;
* access network services;
* inspect environment variables;
* escape a container;
* exploit compiler/runtime behavior;
* attack internal services.

No user source code is trusted.

---

### A6 — Execution Is Ephemeral

User programs are assumed to require temporary execution environments.

The normal lifecycle is:

```text
CREATE
  ↓
PREPARE
  ↓
COMPILE
  ↓
EXECUTE
  ↓
COLLECT RESULT
  ↓
DESTROY
```

Persistent execution containers are not part of the initial architecture.

---

### A7 — Initial Execution Does Not Require Persistent Servers

The initial platform assumes that user programs are batch-style programs.

For example:

```text
Input
  ↓
Program
  ↓
Output
```

The architecture does not initially support arbitrary long-running user servers.

Support for persistent web applications may be added later with a separate architecture.

---

### A8 — Network Access Is Disabled Initially

User execution environments are assumed to have no unrestricted outbound network access.

This prevents user programs from using the compiler platform as:

* a network scanner;
* a proxy;
* a denial-of-service source;
* an internal-network attack platform.

If networking becomes a supported feature, it will require a separate security design.

---

### A9 — Resource Limits Are Mandatory

Every execution is assumed to have resource limits.

At minimum:

```text
CPU
Memory
Wall-Clock Time
Process Count
Disk
Output Size
Source Size
```

No execution may run without an execution policy.

---

# 3.3 Application Assumptions

### A10 — Browser Is an Untrusted Client

The frontend cannot be trusted to enforce security rules.

For example, the browser may send:

```text
role = OWNER
```

but the Gateway must independently verify the user's actual permissions.

Authorization is therefore server-side.

---

### A11 — Gateway Is the Public Application Boundary

The Node.js Gateway is assumed to be the primary public application backend.

External clients should communicate with:

```text
Browser → Gateway
```

rather than directly accessing:

```text
Browser → PostgreSQL
Browser → Redis
Browser → Evaluator
Browser → Sandbox
```

Internal services remain private.

---

### A12 — Gateway Does Not Execute User Code

The Gateway is assumed to be an orchestration and application service.

It must not:

* invoke `system()` with user input;
* execute arbitrary shell commands;
* compile user programs locally;
* directly create privileged processes;
* expose host shell access.

Execution responsibility belongs to the evaluator and sandbox layers.

---

### A13 — PostgreSQL Is the Source of Truth

Persistent application state is assumed to belong in PostgreSQL.

This includes:

* users;
* projects;
* project membership;
* files;
* execution metadata;
* execution history;
* persistent sessions where applicable.

Redis must not become the authoritative source for durable user data.

---

### A14 — Redis Is Transient Infrastructure

Redis is assumed to be used for:

* execution queues;
* caching;
* pub/sub;
* temporary state;
* coordination.

Redis data should be considered recoverable unless explicitly designed otherwise.

---

# 3.4 Collaboration Assumptions

### A15 — Collaboration Is Near Real-Time

The platform assumes that multiple users may edit a project simultaneously.

The target interaction model is:

```text
User A
   │
   ▼
Change
   │
   ▼
Collaboration Layer
   │
   ├──────────► User B
   └──────────► User C
```

---

### A16 — WebSockets Are Available

The architecture assumes that clients can maintain WebSocket connections.

WebSockets are used for:

* document updates;
* presence;
* cursor movement;
* execution events;
* collaboration state.

---

### A17 — CRDT Is the Preferred Synchronization Model

The initial architecture assumes CRDT-based synchronization is preferable for collaborative editing.

However, this remains an implementation decision that may be revised after evaluating:

* implementation complexity;
* memory usage;
* synchronization requirements;
* conflict behavior;
* scalability.

---

### A18 — Collaboration State Is Not the Primary Persistent Source

Real-time collaboration state may be held temporarily in memory or Redis.

Durable project content remains persisted through the application data layer.

---

# 3.5 Language Assumptions

### A19 — C++ Is the First Sandbox Language

C++ is assumed to be the first language implemented through the secure execution pipeline.

Initial flow:

```text
C++ Source
   ↓
Evaluator
   ↓
C++ Sandbox
   ↓
Compiler
   ↓
Executable
   ↓
Restricted Execution
```

---

### A20 — Python and JavaScript Are Planned Languages

The architecture assumes future support for:

```text
C++
Python
JavaScript
```

The execution architecture should therefore avoid hard-coding C++-specific behavior into the Gateway.

Language-specific behavior belongs behind the evaluator/sandbox abstraction.

---

### A21 — Language Runtimes Are Isolated

Each language may eventually require its own execution image.

Example:

```text
Language
   │
   ├── C++ → C++ Runtime Image
   ├── Python → Python Runtime Image
   └── JavaScript → Node Runtime Image
```

This keeps language dependencies separated.

---

# 3.6 Data Assumptions

### A22 — Projects Are Multi-File

The architecture assumes that a project can contain multiple files.

Example:

```text
hello-project/
├── main.cpp
├── math.cpp
├── math.hpp
└── README.md
```

The execution request must therefore support a file set rather than a single source string only.

---

### A23 — Project Ownership Is Explicit

Every project is assumed to have an owner.

Additional users may be associated through project membership.

```text
Project
   │
   ├── OWNER
   ├── EDITOR
   └── VIEWER
```

---

### A24 — Execution Results Are Structured

Execution results are assumed to contain structured information rather than only raw console output.

Example:

```text
ExecutionResult
├── status
├── stdout
├── stderr
├── exit_code
├── signal
├── compile_time
├── execution_time
├── memory_usage
└── resource_violation
```

---

# 3.7 Availability and Performance Assumptions

### A25 — Execution Is Asynchronous

Code execution is assumed to be asynchronous.

The API should not remain blocked while a compiler or user program executes.

Instead:

```text
Submit Job
   ↓
Return Job ID
   ↓
Queue
   ↓
Worker
   ↓
Execution
   ↓
Result Event
```

---

### A26 — Execution Duration Is Bounded

User programs are assumed to have a finite execution timeout.

Infinite loops must eventually be terminated.

Example:

```text
while (true)
{
}
```

must not permanently consume execution infrastructure.

---

### A27 — Performance Targets Will Be Benchmark-Driven

The architecture does not assume arbitrary latency targets before implementation measurements exist.

Performance targets will be established using:

* load tests;
* execution benchmarks;
* queue benchmarks;
* WebSocket benchmarks;
* sandbox startup measurements.

---

# 3.8 Operational Assumptions

### A28 — Services Are Independently Testable

Each major service is assumed to be testable independently.

```text
Frontend
Gateway
Evaluator
Sandbox
```

Integration tests will validate cross-service behavior.

---

### A29 — Configuration Is Externalized

Environment-specific configuration should not be embedded in application source code.

Examples include:

```text
DATABASE_URL
REDIS_URL
GRPC_ENDPOINT
EXECUTION_TIMEOUT
MEMORY_LIMIT
CPU_LIMIT
```

---

### A30 — Observability Is Part of the Architecture

Logging and metrics are assumed to be required infrastructure rather than optional debugging features.

Every execution should have a traceable identifier.

Example:

```text
request_id
job_id
execution_id
```

These identifiers should make it possible to follow an execution across services.

---

# 3.9 Security Assumptions

### A31 — Host Resources Are Never Exposed

The execution environment must not receive unrestricted access to:

```text
Host filesystem
Host devices
Host process namespace
Docker socket
Platform credentials
Database credentials
Redis credentials
```

In particular, exposing the Docker socket to user-controlled execution containers is prohibited.

---

### A32 — Secrets Never Enter User Execution Environments

Platform secrets must not be injected into user containers.

The execution environment should receive only the configuration required to execute the user program.

---

### A33 — Service-to-Service Access Is Explicit

Internal services should only communicate through defined interfaces.

Example:

```text
Evaluator → Sandbox
```

does not imply:

```text
Sandbox → PostgreSQL
Sandbox → Redis
Sandbox → Gateway
```

unless explicitly required and authorized.

---

### A34 — Failures Must Fail Closed

When a security decision cannot be safely determined, the operation should be rejected.

Examples:

```text
Unknown language → reject
Invalid resource policy → reject
Invalid project permission → reject
Malformed execution request → reject
Sandbox policy failure → reject
```

Unsafe fallback behavior is prohibited.

---

# 3.10 Development Assumptions

### A35 — Docker Compose Is the Initial Orchestration Layer

Docker Compose is assumed to be sufficient for:

* local development;
* integration testing;
* initial demonstration;
* early deployment.

A future Kubernetes deployment should not be required to run the MVP.

---

### A36 — Contract-First Development

The architecture assumes that internal service contracts are defined before implementation.

The canonical contract location is:

```text
proto/
```

Example:

```text
proto/
├── execution.proto
├── evaluator.proto
├── sandbox.proto
└── collaboration.proto
```

---

### A37 — Architecture Can Evolve

These assumptions are not permanent requirements.

When a requirement changes, the team should identify:

1. which assumption changed;
2. which architecture components are affected;
3. whether an ADR is required;
4. whether existing APIs or database schemas must change;
5. whether security properties remain valid.

---

# 3.11 Assumption Validation

The following assumptions should be validated during implementation.

| ID  | Assumption                        | Validation Phase |
| --- | --------------------------------- | ---------------- |
| A1  | Linux infrastructure              | Phase 1          |
| A2  | Docker isolation                  | Phase 2          |
| A5  | User code is untrusted            | Phase 2          |
| A8  | Network disabled                  | Phase 2          |
| A9  | Resource limits                   | Phase 2          |
| A13 | PostgreSQL source of truth        | Phase 5          |
| A15 | Real-time collaboration           | Phase 7          |
| A17 | CRDT suitability                  | Phase 7          |
| A19 | C++ first language                | Phase 2          |
| A20 | Multi-language architecture       | Phase 3          |
| A25 | Async execution                   | Phase 3          |
| A27 | Benchmark-driven performance      | Phase 9          |
| A31 | Host resources isolated           | Phase 2 / 9      |
| A35 | Docker Compose sufficient for MVP | Phase 1 / 8      |

---

# 4. Architectural Style

CodeForge Cloud follows a **distributed modular service architecture**.

The initial implementation uses containerized services rather than a Kubernetes-first deployment.

The architecture can be described as:

```text
                    Presentation Layer
                           │
                           ▼
                    API / Gateway Layer
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
      Data Layer      Collaboration     Execution
                           │                │
                           │                ▼
                           │           Evaluation
                           │                │
                           │                ▼
                           │             Sandbox
                           │
                           ▼
                        Redis
```

The services are independently deployable.

---

# 5. System Components

The system consists of the following primary components.

| Component             | Technology            | Responsibility                               |
| --------------------- | --------------------- | -------------------------------------------- |
| Web IDE               | React + TypeScript    | Browser-based development environment        |
| Editor                | Monaco Editor         | Source-code editing                          |
| Gateway               | Node.js + TypeScript  | API, authentication, projects, collaboration |
| Collaboration         | WebSocket / Socket.IO | Real-time communication                      |
| Database              | PostgreSQL            | Persistent application data                  |
| Cache / Queue         | Redis                 | Jobs, caching, pub/sub                       |
| Evaluator             | Python                | Validation and execution orchestration       |
| Sandbox               | C++                   | Low-level execution isolation                |
| Execution Environment | Docker                | Container isolation                          |
| Protocol              | gRPC + Protobuf       | Internal service communication               |
| Reverse Proxy         | Nginx                 | External HTTP routing                        |
| Observability         | Logs + Metrics        | Monitoring and diagnostics                   |

---

# 6. Frontend Architecture

The frontend is a React and TypeScript application.

```text
frontend/
└── web-ide/
    ├── src/
    │   ├── components/
    │   ├── pages/
    │   ├── editor/
    │   ├── terminal/
    │   ├── explorer/
    │   ├── collaboration/
    │   ├── execution/
    │   ├── services/
    │   ├── hooks/
    │   ├── store/
    │   ├── types/
    │   └── main.tsx
    └── package.json
```

## 6.1 Major frontend modules

### Editor

Responsible for:

* Monaco integration;
* syntax highlighting;
* language selection;
* multiple files;
* tabs;
* editor configuration;
* diagnostics.

### File Explorer

Responsible for:

* project directory tree;
* file creation;
* file deletion;
* file renaming;
* folder creation;
* file selection.

### Terminal

The initial implementation provides an execution console rather than a fully interactive operating-system shell.

Example:

```text
$ codeforge run

Compiling...
Compilation successful.

Program output:
Hello, CodeForge!

Process exited with code 0
Execution time: 21 ms
```

### Collaboration Panel

Provides:

* active users;
* presence;
* user cursors;
* collaboration status;
* connection status.

### Problems Panel

Displays:

* compiler errors;
* warnings;
* syntax errors;
* runtime errors;
* resource-limit violations.

---

# 7. Node.js Gateway Architecture

The Node.js Gateway is the primary application backend.

Its responsibilities include:

* HTTP API;
* authentication;
* authorization;
* project management;
* file management;
* execution submission;
* WebSocket connections;
* collaboration;
* request validation;
* rate limiting.

It must **not** execute arbitrary user source code.

---

## 7.1 Gateway architecture

```text
                    ┌──────────────────┐
                    │ HTTP / WebSocket │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Router           │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │ Middleware       │
                    │                  │
                    │ Auth             │
                    │ Validation       │
                    │ Rate Limit       │
                    └────────┬─────────┘
                             │
             ┌───────────────┼────────────────┐
             ▼               ▼                ▼
        Project Service  File Service   Execution Service
             │               │                │
             └───────────────┼────────────────┘
                             │
                    ┌────────▼─────────┐
                    │ Repository Layer │
                    └────────┬─────────┘
                             │
                             ▼
                         PostgreSQL
```

---

# 8. Python Evaluator Architecture

The Python evaluator acts as the execution orchestration layer.

It receives validated execution requests and determines how they should be processed.

Responsibilities include:

1. validate execution requests;
2. determine language;
3. create execution jobs;
4. enqueue work;
5. select appropriate worker;
6. communicate with sandbox services;
7. monitor execution state;
8. normalize results;
9. return structured execution information.

---

## 8.1 Evaluator flow

```text
Execution Request
       │
       ▼
Request Validation
       │
       ▼
Language Detection
       │
       ▼
Execution Policy
       │
       ▼
Redis Queue
       │
       ▼
Evaluator Worker
       │
       ▼
Sandbox Request
       │
       ▼
Result
       │
       ▼
Result Normalization
       │
       ▼
Gateway
```

---

# 9. C++ Sandbox Architecture

The C++ sandbox runtime is the security-critical execution component.

Its primary responsibility is to create and control restricted execution environments.

The sandbox must:

* create temporary execution directories;
* prepare source files;
* invoke compilers safely;
* apply resource limits;
* execute programs;
* capture output;
* detect abnormal termination;
* enforce timeouts;
* destroy execution environments.

---

# 10. Sandbox Execution Model

A typical execution looks like:

```text
Source Code
    │
    ▼
Temporary Workspace
    │
    ▼
Compilation
    │
    ├── Compilation Error
    │        │
    │        ▼
    │    Return Error
    │
    ▼
Executable
    │
    ▼
Resource Policy
    │
    ▼
Restricted Process
    │
    ├── stdout
    ├── stderr
    ├── exit code
    ├── signal
    └── resource usage
    │
    ▼
Execution Result
    │
    ▼
Cleanup
```

---

# 11. Container Isolation

Docker is the initial execution isolation mechanism.

Each user execution should run in an ephemeral container.

Conceptually:

```text
Sandbox Runtime
      │
      ▼
Docker Runtime
      │
      ▼
┌──────────────────────────────┐
│ Ephemeral Container          │
│                              │
│ /workspace                   │
│ /tmp                         │
│ Compiler                     │
│ User executable              │
│                              │
│ Restricted resources         │
│ Restricted filesystem        │
│ Network disabled             │
└──────────────────────────────┘
```

The container must be treated as disposable.

After execution:

```text
Container Created
      ↓
Execution
      ↓
Result Captured
      ↓
Container Destroyed
```

---

# 12. Defense-in-Depth Security

Container isolation alone should not be considered sufficient.

The sandbox architecture should use multiple controls.

```text
                User Code
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
            Resource Policy
                    │
                    ▼
             Container
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
    Namespace     cgroups     seccomp
        │           │           │
        └───────────┼───────────┘
                    ▼
             Restricted FS
                    │
                    ▼
             Network Disabled
                    │
                    ▼
              Execution
```

Future hardening may include:

* Linux namespaces;
* cgroups;
* seccomp;
* dropped Linux capabilities;
* read-only root filesystem;
* dedicated execution user;
* restricted `/tmp`;
* filesystem quotas;
* PID limits;
* CPU limits;
* memory limits;
* process limits;
* output limits.

---

# 13. Resource Control Architecture

Every execution must have a resource policy.

Example policy:

```text
CPU Time
Memory
Wall-clock Timeout
Process Count
Disk Usage
Output Size
Source Size
Network Access
```

The exact production values will be defined during implementation and benchmarking.

The important architectural principle is:

```text
User Program
     │
     ▼
Resource Policy
     │
     ├── CPU
     ├── Memory
     ├── Time
     ├── Processes
     ├── Disk
     └── Output
```

If a limit is exceeded, the execution must be terminated and a structured result returned.

---

# 14. Redis Architecture

Redis serves several purposes.

## 14.1 Execution Queue

```text
Gateway
   │
   ▼
Redis
   │
   ▼
Evaluator Worker
```

This prevents execution requests from blocking HTTP requests.

---

## 14.2 Pub/Sub

Redis can distribute events between gateway instances.

Example:

```text
Gateway A
    │
    ▼
 Redis Pub/Sub
    │
    ▼
Gateway B
```

---

## 14.3 Caching

Redis may cache:

* sessions;
* project metadata;
* frequently accessed configuration;
* temporary collaboration state.

Persistent information must remain in PostgreSQL.

---

# 15. PostgreSQL Architecture

PostgreSQL is the primary persistent data store.

Logical structure:

```text
PostgreSQL
│
├── users
├── projects
├── project_members
├── files
├── execution_jobs
├── execution_results
└── sessions
```

The database is responsible for durable application state.

Redis should not be treated as the source of truth for persistent user data.

---

# 16. Internal Communication

Internal services communicate through gRPC and Protocol Buffers.

```text
Python Evaluator
       │
       │ gRPC
       ▼
C++ Sandbox
```

Protocol Buffers provide:

* strongly typed messages;
* explicit contracts;
* language-independent definitions;
* efficient serialization;
* versionable APIs.

The protocol definitions will be stored in:

```text
proto/
├── execution.proto
├── evaluator.proto
├── sandbox.proto
└── collaboration.proto
```

---

# 17. Execution Protocol

Conceptual request:

```text
ExecutionRequest
├── job_id
├── language
├── source_files
├── entry_file
├── compile_options
└── resource_policy
```

Conceptual response:

```text
ExecutionResult
├── job_id
├── status
├── stdout
├── stderr
├── exit_code
├── signal
├── compile_time
├── execution_time
├── memory_usage
└── resource_violation
```

---

# 18. Execution State Machine

Execution jobs use explicit states.

```text
                ┌───────────┐
                │  QUEUED   │
                └─────┬─────┘
                      │
                      ▼
                ┌───────────┐
                │ STARTING  │
                └─────┬─────┘
                      │
                      ▼
                ┌───────────┐
                │ COMPILING │
                └─────┬─────┘
                      │
                 compile OK
                      │
                      ▼
                ┌───────────┐
                │  RUNNING  │
                └─────┬─────┘
                      │
                      ▼
                ┌────────────┐
                │ COMPLETED  │
                └────────────┘

Failure paths:

COMPILING ───────► FAILED
RUNNING ─────────► FAILED
RUNNING ─────────► TIMEOUT
RUNNING ─────────► RESOURCE_LIMIT
QUEUED ──────────► CANCELLED
RUNNING ─────────► CANCELLED
```

---

# 19. Complete Execution Data Flow

The complete request path is:

```text
┌─────────────┐
│    User     │
└──────┬──────┘
       │
       │ Run
       ▼
┌─────────────┐
│ React IDE   │
└──────┬──────┘
       │ HTTPS
       ▼
┌─────────────┐
│ Node Gateway│
└──────┬──────┘
       │
       │ Create Job
       ▼
┌─────────────┐
│ PostgreSQL  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    Redis    │
│    Queue    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Python    │
│  Evaluator  │
└──────┬──────┘
       │ gRPC
       ▼
┌─────────────┐
│ C++ Sandbox │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│ Docker Container│
└──────┬──────────┘
       │
       ├── Compile
       │
       ├── Execute
       │
       └── Capture
       │
       ▼
┌─────────────┐
│ Result      │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Node Gateway│
└──────┬──────┘
       │ WebSocket / HTTP
       ▼
┌─────────────┐
│ React IDE   │
└─────────────┘
```

---

# 20. Real-Time Collaboration Architecture

CodeForge Cloud uses WebSocket communication for real-time collaboration.

The initial architecture is:

```text
User A
   │
   ▼
Gateway
   │
   ├─────────────┐
   │             │
   ▼             ▼
Redis         User B
   ▲             ▲
   │             │
   └── Gateway ──┘
```

For document synchronization, CRDT is the preferred long-term approach.

---

# 21. Collaboration Model

A project may have multiple connected users.

Example:

```text
Project: hello-world

             ┌─────────────┐
             │   Project   │
             └──────┬──────┘
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      User A      User B      User C
      Editor      Editor       Viewer
```

Each user has a role.

```text
OWNER
  │
  ├── EDITOR
  │
  └── VIEWER
```

Authorization must be checked by the backend rather than trusted from the browser.

---

# 22. Collaboration Events

Typical WebSocket events include:

```text
project:join
project:leave

document:update
document:sync

cursor:update
selection:update

presence:join
presence:leave

execution:start
execution:update
execution:complete
```

The exact event contract will be defined in the collaboration protocol.

---

# 23. API Architecture

External clients communicate with the Node Gateway.

Example API groups:

```text
/api/v1/auth
/api/v1/users
/api/v1/projects
/api/v1/projects/:id/files
/api/v1/executions
/api/v1/sessions
```

The API should be versioned from the beginning.

Example:

```text
/api/v1/...
```

This provides room for future incompatible versions.

---

# 24. Authentication Architecture

Authentication is handled at the Gateway layer.

Conceptually:

```text
Browser
   │
   ▼
Login
   │
   ▼
Authentication Service
   │
   ▼
Session / Token
   │
   ▼
Gateway
   │
   ▼
Authenticated Request
```

Credentials must never be exposed to frontend JavaScript unnecessarily.

---

# 25. Authorization Architecture

Authentication answers:

> Who is the user?

Authorization answers:

> What is the user allowed to do?

Example:

```text
User
 │
 ▼
Project Membership
 │
 ├── OWNER
 ├── EDITOR
 └── VIEWER
```

Backend authorization is mandatory.

---

# 26. Nginx / Reverse Proxy

Nginx acts as the external entry point.

```text
Internet
   │
   ▼
 Nginx
   │
   ├── /        → React
   ├── /api     → Node Gateway
   └── /ws      → WebSocket Gateway
```

Internal services should not be directly exposed to the public network.

---

# 27. Docker Compose Architecture

Development and early deployment use Docker Compose.

Conceptual services:

```text
services:
├── frontend
├── gateway
├── evaluator
├── sandbox
├── postgres
├── redis
└── nginx
```

Only required ports should be exposed to the host.

---

# 28. Service Boundaries

A major architectural principle is that each service owns a specific responsibility.

```text
Frontend
  → User experience

Gateway
  → Application API

Evaluator
  → Execution orchestration

Sandbox
  → Secure execution

PostgreSQL
  → Persistent state

Redis
  → Transient state and queues

Nginx
  → External routing
```

---

# 29. Failure Handling

The platform must expect failures.

## 29.1 Gateway Failure

If one gateway instance fails, another instance should be able to handle subsequent requests.

---

## 29.2 Evaluator Failure

Queued jobs should remain recoverable.

```text
Redis Queue
     │
     ├── Worker A
     ├── Worker B
     └── Worker C
```

---

## 29.3 Sandbox Failure

A sandbox failure should result in a structured execution error rather than crashing the evaluator.

---

## 29.4 Compilation Failure

Compilation failure is a normal user-facing result.

```text
COMPILING
    │
    ▼
Compiler Error
    │
    ▼
FAILED
```

---

## 29.5 Timeout

If the program exceeds its wall-clock execution limit:

```text
RUNNING
   │
   ▼
TIMEOUT DETECTED
   │
   ▼
PROCESS TERMINATED
   │
   ▼
CONTAINER DESTROYED
```

---

# 30. Scalability Architecture

The platform should scale horizontally.

```text
                     Nginx
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
          Gateway   Gateway   Gateway
             │         │         │
             └────┬────┴────┬────┘
                  │         │
                Redis     PostgreSQL
                  │
          ┌───────┼────────┐
          ▼       ▼        ▼
       Worker   Worker   Worker
          │       │        │
          ▼       ▼        ▼
       Sandbox Sandbox  Sandbox
```

Execution capacity can therefore be increased without scaling the frontend unnecessarily.

---

# 31. Performance Considerations

The system's critical latency path is:

```text
Request
 → Queue
 → Worker
 → Sandbox
 → Compile
 → Execute
 → Result
```

Important metrics include:

* API latency;
* queue wait time;
* sandbox startup time;
* compilation time;
* execution time;
* result serialization time;
* WebSocket latency.

Optimization should occur only after measurement.

---

# 32. Observability Architecture

Every service should generate structured logs.

Example:

```json
{
  "timestamp": "2026-09-08T12:00:00Z",
  "service": "sandbox",
  "level": "info",
  "job_id": "job-123",
  "event": "execution_completed",
  "execution_time_ms": 42
}
```

Important metrics include:

```text
api_requests_total
api_request_duration
execution_jobs_total
execution_queue_depth
execution_queue_latency
sandbox_startup_time
compile_duration
execution_duration
sandbox_failures
execution_timeouts
resource_limit_events
websocket_connections
active_collaboration_sessions
```

---

# 33. Security Boundaries

Security boundaries should exist between:

```text
Browser
   │
   ▼
Gateway
   │
   ▼
Evaluator
   │
   ▼
Sandbox
   │
   ▼
Container
```

The most trusted components should not execute the least trusted workloads.

The sandbox is the final boundary between untrusted user code and platform infrastructure.

---

# 34. Network Security

The execution environment should not have unrestricted network access.

Initial policy:

```text
User Container
      │
      X
Internet
```

The container should not be able to:

* access PostgreSQL;
* access Redis;
* access Gateway internals;
* access cloud metadata endpoints;
* scan internal networks;
* communicate with other execution containers.

---

# 35. Filesystem Security

User programs must operate inside a controlled workspace.

Example:

```text
Container
│
├── /workspace
│   ├── main.cpp
│   └── ...
│
├── /tmp
│
└── restricted system files
```

The host filesystem must never be mounted into the user execution environment.

---

# 36. Secrets Management

Secrets must not be hard-coded into source code.

Potential secrets include:

```text
DATABASE_URL
REDIS_URL
SESSION_SECRET
INTERNAL_SERVICE_TOKEN
```

Development may use environment variables.

Production should use a dedicated secret-management solution.

User-submitted source code must never receive platform secrets.

---

# 37. Architectural Dependency Rules

The following dependency rules apply.

### Rule 1

Frontend must communicate with the Gateway rather than directly accessing databases.

```text
React → Gateway → Database
```

---

### Rule 2

Gateway must not execute user code.

```text
Gateway → Evaluator → Sandbox
```

---

### Rule 3

Evaluator must not directly access arbitrary host resources.

---

### Rule 4

Sandbox must not trust user source code.

---

### Rule 5

PostgreSQL is the source of truth for persistent data.

---

### Rule 6

Redis is used for transient state, queues, caching, and coordination.

---

# 38. Repository-to-Architecture Mapping

The repository structure maps directly to the architecture.

```text
Online-Code-Compiler/
│
├── frontend/
│   └── web-ide/
│       └── React application
│
├── gateway/
│   └── node/
│       └── Node.js Gateway
│
├── evaluator/
│   └── python/
│       └── Python Evaluator
│
├── sandbox/
│   └── cpp/
│       └── C++ Sandbox Runtime
│
├── proto/
│   └── service contracts
│
├── infrastructure/
│   ├── docker/
│   ├── postgres/
│   ├── redis/
│   └── nginx/
│
└── tests/
    ├── integration/
    ├── security/
    └── load/
```

---

# 39. Architecture Principles

CodeForge Cloud follows these principles.

## 39.1 Security by Design

Security is considered before implementation rather than added afterward.

## 39.2 Contract First

Internal service communication is defined through Protocol Buffers before implementation.

## 39.3 Least Privilege

Every service and process receives only the permissions required for its responsibility.

## 39.4 Separation of Concerns

Editing, collaboration, execution, persistence, and sandboxing remain separate concerns.

## 39.5 Disposable Execution

Execution environments are temporary.

```text
Create → Execute → Collect → Destroy
```

## 39.6 Observable by Default

Important system operations must produce logs and metrics.

## 39.7 Fail Closed

When security validation fails, execution must be rejected rather than attempting to continue with unsafe defaults.

---

# 40. Initial Deployment Architecture

The initial deployment target is a single machine running Docker Compose.

```text
                     Public Internet
                           │
                           ▼
                         Nginx
                           │
                ┌──────────┴──────────┐
                ▼                     ▼
            React IDE            Node Gateway
                                      │
                         ┌────────────┼────────────┐
                         ▼            ▼            ▼
                     PostgreSQL     Redis      Evaluator
                                                   │
                                                   ▼
                                                Sandbox
                                                   │
                                                   ▼
                                             Docker Runtime
```

This architecture is intentionally simple enough for local development while preserving service boundaries required for future scaling.

---

# 41. Future Production Architecture

The architecture can evolve toward:

```text
                         Internet
                            │
                     Load Balancer
                            │
                    ┌───────┴───────┐
                    ▼               ▼
                 Gateway         Gateway
                    │               │
                    └───────┬───────┘
                            │
                         Redis
                            │
                  ┌─────────┼─────────┐
                  ▼         ▼         ▼
               Worker    Worker    Worker
                  │         │         │
                  ▼         ▼         ▼
             Sandbox    Sandbox    Sandbox
                  │         │         │
                  └─────────┼─────────┘
                            │
                         Storage
```

Future infrastructure may use:

* Kubernetes;
* managed PostgreSQL;
* managed Redis;
* object storage;
* container orchestration;
* centralized logging;
* metrics collection;
* distributed tracing;
* autoscaling.

---

# 42. Architecture Decision Summary

| Decision                   | Choice                |
| -------------------------- | --------------------- |
| Frontend                   | React + TypeScript    |
| Editor                     | Monaco                |
| Gateway                    | Node.js + TypeScript  |
| API                        | REST                  |
| Real-time                  | WebSocket / Socket.IO |
| Evaluator                  | Python                |
| Sandbox                    | C++                   |
| Isolation                  | Docker                |
| Internal RPC               | gRPC                  |
| Contract                   | Protocol Buffers      |
| Database                   | PostgreSQL            |
| Queue                      | Redis                 |
| Reverse Proxy              | Nginx                 |
| Development Deployment     | Docker Compose        |
| Future Deployment          | Kubernetes            |
| Collaboration Model        | CRDT preferred        |
| Execution Model            | Ephemeral             |
| Network in Sandbox         | Disabled initially    |
| Execution Model            | Asynchronous          |
| Persistent Source of Truth | PostgreSQL            |
| User Code Trust Level      | Untrusted             |
| Initial Language           | C++                   |

---

# 43. Architecture Validation Checklist

Before considering the architecture implementation-ready, verify:

* [ ] All user code is treated as untrusted.
* [ ] Gateway cannot directly execute user code.
* [ ] User containers cannot access the host filesystem.
* [ ] Docker socket is not exposed to user workloads.
* [ ] User containers have no unrestricted network access.
* [ ] CPU limits are enforced.
* [ ] Memory limits are enforced.
* [ ] Process/PID limits are enforced.
* [ ] Execution timeout is enforced.
* [ ] Output limits are enforced.
* [ ] Temporary execution environments are destroyed.
* [ ] PostgreSQL is the persistent source of truth.
* [ ] Redis is not treated as durable storage.
* [ ] Internal communication uses defined contracts.
* [ ] Authentication is server-side.
* [ ] Authorization is server-side.
* [ ] Execution jobs are asynchronous.
* [ ] Execution state transitions are explicit.
* [ ] Every execution has a traceable job ID.
* [ ] Service failures produce structured errors.
* [ ] Logs and metrics are available.
* [ ] Architecture assumptions are reviewed when requirements change.

---

# 44. End-to-End Architecture

The complete CodeForge Cloud architecture can therefore be summarized as:

```text
                         CODEFORGE CLOUD
                              │
                              ▼
                     ┌─────────────────┐
                     │    Web Browser  │
                     │                 │
                     │ React / Monaco  │
                     └────────┬────────┘
                              │
                       HTTPS / WebSocket
                              │
                              ▼
                     ┌─────────────────┐
                     │  Node Gateway   │
                     │                 │
                     │ API             │
                     │ Auth            │
                     │ Projects        │
                     │ Collaboration   │
                     └───────┬─────────┘
                             │
             ┌───────────────┼────────────────┐
             │               │                │
             ▼               ▼                ▼
        PostgreSQL         Redis          WebSocket
             │               │
             │               ▼
             │          Execution Queue
             │               │
             │               ▼
             │        Python Evaluator
             │               │
             │             gRPC
             │               │
             │               ▼
             │        C++ Sandbox
             │               │
             │               ▼
             │      Docker Container
             │               │
             │        ┌──────┴──────┐
             │        │             │
             │     Compile       Execute
             │        │             │
             │        └──────┬──────┘
             │               │
             │          Result Capture
             │               │
             └───────────────┼──────────────┐
                             ▼              │
                       Execution Result    │
                             │              │
                             ▼              │
                       Node Gateway         │
                             │              │
                             ▼              │
                        React IDE ◄─────────┘
```

This architecture establishes the foundation for implementing CodeForge Cloud as a secure, distributed, collaborative online development platform.

The explicit assumptions in Section 3 provide the baseline against which future architecture changes, implementation decisions, and ADRs should be evaluated.

---

# 45. Next Document

The next architecture dependency is the data model.

The next document is:

```text
docs/03-data-model.md
```

It will define:

* database schema;
* entities;
* relationships;
* primary keys;
* foreign keys;
* indexes;
* project membership;
* file storage;
* execution jobs;
* execution results;
* sessions;
* collaboration persistence;
* database constraints;
* migration strategy;
* Redis data structures;
* execution queue model.

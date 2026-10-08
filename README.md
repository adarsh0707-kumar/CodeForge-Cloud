# CodeForge Cloud

> **Architecture-first online code execution platform for collaborative development and secure sandboxing.**

CodeForge Cloud is a **portfolio-grade systems project** exploring how a browser IDE can be connected to an asynchronous execution pipeline that treats user code as untrusted input.

The project combines a web IDE, API gateway, durable state, job orchestration, service-to-service RPC, and an isolated C++ execution layer. The repository currently contains the **architecture, contracts, documentation, and implementation skeletons**; the end-to-end product is still under active development.

> **Status:** Architecture and implementation foundation  
> **Current focus:** Turning the documented design into a working vertical slice  
> **Important:** The security controls and services described below are **design targets unless explicitly marked implemented**. This repository is **not production-ready** and must not be used to execute untrusted code.

---

## Why this project

An online compiler is not simply a text editor plus `g++`.

The difficult engineering problem is:

> **How do you accept arbitrary source code, execute it asynchronously, constrain its resources, isolate it from the host, and return a traceable result without trusting the workload?**

CodeForge Cloud is built around that problem.

It is primarily an exploration of:

- distributed service boundaries
- asynchronous job execution
- secure sandbox design
- resource enforcement
- immutable execution inputs
- API and RPC contracts
- real-time collaboration
- database consistency
- testing and observability
- production-readiness engineering

---

## Architecture

The target architecture separates the control plane from the execution plane:

```text
                           Browser
                              │
                    HTTPS / WebSocket
                              │
                              ▼
                     ┌────────────────┐
                     │  React Web IDE │
                     │ TypeScript +   │
                     │ Monaco Editor  │
                     └───────┬────────┘
                             │
                             ▼
                     ┌────────────────┐
                     │ Node.js Gateway│
                     │ API + Auth +   │
                     │ Collaboration │
                     └───────┬────────┘
                             │
                  ┌──────────┴──────────┐
                  │                     │
                  ▼                     ▼
           ┌─────────────┐       ┌─────────────┐
           │ PostgreSQL  │       │    Redis    │
           │ Durable     │       │ Queue /     │
           │ State       │       │ Transient   │
           └─────────────┘       └──────┬──────┘
                                        │
                                        ▼
                               ┌────────────────┐
                               │ Python         │
                               │ Evaluator      │
                               │ / Worker       │
                               └───────┬────────┘
                                       │ gRPC
                                       ▼
                               ┌────────────────┐
                               │ C++ Sandbox    │
                               │ Runtime        │
                               └───────┬────────┘
                                       │
                                       ▼
                               ┌────────────────┐
                               │ Docker / Linux │
                               │ Isolation      │
                               └────────────────┘
```

### Responsibility boundaries

| Component | Intended responsibility |
| --- | --- |
| React Web IDE | Editing, projects, execution controls, results, collaboration UI |
| Nginx | Reverse proxy and edge routing |
| Node.js Gateway | Public API, authentication, authorization, projects, files, WebSockets |
| PostgreSQL | Durable application state and execution history |
| Redis | Queues, transient state, pub/sub and coordination |
| Python Evaluator | Validation, scheduling, worker orchestration and result handling |
| C++ Sandbox | Low-level execution and isolation controls |
| Docker / Linux | Disposable execution environment and OS-level isolation |
| gRPC / Protobuf | Internal service contracts |

---

## Execution model

The intended execution path is asynchronous:

```text
Source Code
    │
    ▼
Authenticate + Authorize
    │
    ▼
Validate Request
    │
    ▼
Create Immutable Snapshot
    │
    ▼
Persist Execution Job
    │
    ▼
Redis Queue
    │
    ▼
Python Worker
    │
    ▼
C++ Sandbox
    │
    ▼
Restricted Container
    │
    ├── Compile
    └── Execute
         │
         ▼
stdout / stderr / exit status
         │
         ▼
Persist Result
         │
         ▼
WebSocket / API
         │
         ▼
Web IDE
```

A core design rule is:

> **An execution should run against a fixed snapshot, not mutable live project state.**

The planned snapshot model records source content and integrity metadata so an execution can be traced back to the exact input that produced its result.

---

## Security model

Security is the highest-risk part of the system.

The design assumes:

> **User source code is hostile input.**

The target sandbox uses defense in depth:

```text
User Code
   │
   ▼
Immutable Snapshot
   │
   ▼
Resource Policy
   │
   ▼
Ephemeral Container
   │
   ├── PID / mount / network namespaces
   ├── cgroups
   ├── seccomp
   ├── Restricted capabilities
   ├── Read-only filesystem where practical
   ├── Temporary workspace
   ├── Network disabled by default
   ├── CPU / memory / process limits
   └── Execution timeout
```

The execution environment must never rely on a single isolation mechanism.

### Security requirements

The production target is designed to prevent workloads from reaching:

- the host filesystem
- the Docker socket
- the host network
- PostgreSQL
- Redis
- gateway internals
- other projects or tenants
- unrestricted host privileges

**These are architectural requirements, not claims that the current repository already provides a production-grade sandbox.**

---

## Planned capabilities

### Web development

- Browser-based IDE
- Monaco Editor
- Project and file management
- Execution controls
- Terminal output
- Execution history
- Real-time status updates

### Collaboration

- WebSocket connections
- Project rooms
- Presence
- Cursor and selection events
- File synchronization
- Conflict-aware collaboration
- CRDT-based synchronization as a later milestone

### Execution

- Asynchronous execution jobs
- Immutable snapshots
- C++ compilation/execution
- Queue-backed workers
- Execution state machine
- Cancellation and timeout handling
- Resource-limit enforcement
- Persistent execution results

### Platform

- Authentication
- Object-level authorization
- PostgreSQL-backed state
- Redis-based job coordination
- gRPC service contracts
- Structured observability
- Security and integration testing

---

## Execution states

The planned state machine is:

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

Failure or terminal states include:

```text
FAILED
TIMEOUT
CANCELLED
RESOURCE_LIMIT
```

Invalid transitions must be rejected, and terminal execution records should remain immutable.

---

## Technology stack

### Frontend
- React
- TypeScript
- Vite
- Monaco Editor
- Tailwind CSS
- Lucide

### Gateway
- Node.js
- TypeScript
- REST
- WebSocket

### Evaluator
- Python
- FastAPI
- Redis
- gRPC

### Sandbox
- C++
- Docker
- Linux namespaces
- cgroups
- seccomp

### Data & infrastructure
- PostgreSQL
- Redis
- Nginx
- Docker Compose
- Protocol Buffers

### Production direction

Potential later-stage infrastructure includes Kubernetes, dedicated worker pools, NetworkPolicies, mTLS, gVisor, Kata Containers, microVMs, signed images, SBOM generation, and centralized observability.

These are **future architecture options**, not current implementation claims.

---

## Repository structure

```text
CodeForge-Cloud/
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
├── frontend/web-ide/
├── gateway/node/
├── evaluator/python/
├── sandbox/cpp/
├── proto/
├── infrastructure/
├── scripts/
├── tests/
├── docker-compose.yml
├── Makefile
├── SECURITY.md
├── CONTRIBUTING.md
└── README.md
```

The source tree is currently a **scaffold**. Many source, configuration, build, and orchestration files are placeholders awaiting implementation.

---

## Implementation status

| Area | Status |
| --- | --- |
| Product requirements | Documented |
| System architecture | Documented |
| Data model | Documented |
| API contracts | Documented |
| Security architecture | Documented |
| Testing strategy | Documented |
| Roadmap | Documented |
| Frontend | Skeleton |
| Gateway | Skeleton |
| Evaluator | Skeleton |
| C++ sandbox | Skeleton |
| PostgreSQL migrations | Scaffold |
| Protobuf contracts | Scaffold |
| Docker/Compose runtime | Scaffold |
| CI/CD | Scaffold |
| End-to-end execution | **Not implemented** |
| Production sandbox security | **Not implemented / not validated** |
| Production deployment | **Not implemented** |
| Performance benchmarks | **Not yet measured** |

This distinction is intentional: **architecture documentation is not presented as proof of implementation.**

---

## Development

At the current stage, the repository does not provide a reliable one-command application startup because the service implementations and root orchestration are still being built.

Start with the documentation instead:

1. [Product Requirements](./docs/01-product-requirements.md)
2. [Architecture](./docs/02-architecture.md)
3. [Data Model](./docs/03-data-model.md)
4. [API Reference](./docs/04-api-reference.md)
5. [Roadmap](./docs/05-roadmap-and-phases.md)
6. [Development Guide](./docs/06-development-guide.md)
7. [Security](./docs/07-security.md)
8. [Gap Analysis](./docs/08-gap-analysis.md)
9. [Testing Strategy](./docs/09-testing-strategy.md)

When the first complete vertical slice is implemented, this section should become the canonical copy/paste setup guide.

---

## Testing & validation

The project is designed around multiple test layers:

```text
Formatting / Lint
       ↓
Unit Tests
       ↓
Integration Tests
       ↓
API Contract Tests
       ↓
Security Tests
       ↓
Sandbox Tests
       ↓
End-to-End Tests
       ↓
Load / Stress / Soak Tests
```

The most important future validation areas are:

- authorization boundaries
- snapshot immutability
- queue recovery
- sandbox isolation
- CPU and memory enforcement
- timeout behavior
- process-tree termination
- output limits
- filesystem isolation
- network isolation
- cross-project access prevention
- resource cleanup

**No security guarantee should be considered proven until it is backed by repeatable tests in the real execution environment.**

See the [testing strategy](./docs/09-testing-strategy.md).

---

## Performance

No production performance numbers are claimed yet.

The benchmark plan will measure:

| Metric | Purpose |
| --- | --- |
| API latency | Gateway responsiveness |
| Queue latency | Scheduling delay |
| Snapshot creation | Input preparation cost |
| Sandbox startup | Isolation overhead |
| Compilation time | Toolchain cost |
| Execution time | Workload cost |
| Result persistence | Completion-path cost |
| WebSocket latency | Real-time update delay |
| Concurrent executions | Worker scalability |
| CPU / memory usage | Resource efficiency |

Benchmarks will be added only after the corresponding implementation exists and the test environment is documented.

---

## Roadmap

The project is being developed incrementally:

```text
Foundation
   ↓
Data Layer
   ↓
Authentication + Authorization
   ↓
Projects + Files
   ↓
Execution API
   ↓
Evaluator
   ↓
C++ Sandbox
   ↓
End-to-End Execution
   ↓
Web IDE
   ↓
Real-Time Collaboration
   ↓
Observability
   ↓
Security Hardening
   ↓
Performance Validation
   ↓
Production Deployment
```

The detailed phase plan is in [05-roadmap-and-phases.md](./docs/05-roadmap-and-phases.md).

---

## Documentation

The repository contains an extensive architecture and engineering specification:

- [Architecture](./docs/02-architecture.md) — service boundaries and system design
- [Data Model](./docs/03-data-model.md) — persistence and entity relationships
- [API Reference](./docs/04-api-reference.md) — REST, WebSocket and internal contracts
- [Security](./docs/07-security.md) — threat model and sandbox requirements
- [Gap Analysis](./docs/08-gap-analysis.md) — implementation and production gaps
- [Testing Strategy](./docs/09-testing-strategy.md) — quality and security validation
- [Documentation Index](./docs/README.md) — complete documentation map

---

## Project principles

1. **Treat user code as hostile.**
2. **Separate control plane from execution plane.**
3. **Never trust client-supplied authorization.**
4. **Execute immutable snapshots.**
5. **Use defense in depth for sandboxing.**
6. **Make critical guarantees testable.**
7. **Measure performance instead of inventing benchmarks.**
8. **Fail closed on security-sensitive errors.**
9. **Keep durable state in PostgreSQL; use Redis for transient coordination.**
10. **Do not call a feature complete until implementation and validation exist.**

---

## Contributing

Read [CONTRIBUTING.md](./CONTRIBUTING.md) before opening a change.

Security-sensitive issues should follow [SECURITY.md](./SECURITY.md).

---

## License

MIT — see [LICENSE](./LICENSE).

---

## Author

**Adarsh Kumar**

CodeForge Cloud is part of a broader systems-focused portfolio exploring:

```text
Distributed Systems
Secure Code Execution
C/C++
Python
Node.js
React
TypeScript
Docker
PostgreSQL
Redis
gRPC
WebSockets
Linux Isolation
Testing
Observability
```

---

> **Build it. Secure it. Test it. Observe it. Scale it.**

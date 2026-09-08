# CodeForge Cloud

### Collaborative Real-Time Online Code Compiler & IDE Sandbox

[![Status](https://img.shields.io/badge/status-in%20development-orange.svg)](./docs/01-product-requirements.md)
[![Architecture](https://img.shields.io/badge/architecture-cloud--native-blue.svg)](./docs/02-architecture.md)
[![Security](https://img.shields.io/badge/security-sandboxed-green.svg)](./docs/07-security.md)
[![Testing](https://img.shields.io/badge/testing-comprehensive-purple.svg)](./docs/09-testing-strategy.md)
[![Documentation](https://img.shields.io/badge/docs-complete-blue.svg)](./docs/README.md)
[![License](https://img.shields.io/badge/license-MIT-black.svg)](./LICENSE)

**CodeForge Cloud** is a cloud-native collaborative online development platform that combines a browser-based IDE with a distributed, secure code execution infrastructure.

It is designed to provide an experience similar to modern online coding platforms while demonstrating how production-oriented compiler, execution, collaboration, security, and distributed-service architectures can work together.

---

## Table of Contents

* [Overview](#overview)
* [Why CodeForge Cloud](#why-codeforge-cloud)
* [Key Features](#key-features)
* [Architecture](#architecture)
* [Execution Pipeline](#execution-pipeline)
* [Security Model](#security-model)
* [Technology Stack](#technology-stack)
* [Repository Structure](#repository-structure)
* [Documentation](#documentation)
* [Project Roadmap](#project-roadmap)
* [Development](#development)
* [Testing](#testing)
* [Security Principles](#security-principles)
* [Execution States](#execution-states)
* [Design Principles](#design-principles)
* [MVP Scope](#mvp-scope)
* [Future Architecture](#future-architecture)
* [Contributing](#contributing)
* [License](#license)
* [Author](#author)

---

# Overview

CodeForge Cloud is a **polyglot, distributed online compiler and collaborative development platform**.

The platform separates responsibilities across multiple services:

```text
┌─────────────────────────────────────────────────────────────┐
│                       CODEFORGE CLOUD                        │
└─────────────────────────────────────────────────────────────┘

                        Web Browser
                             │
                             ▼
                    React Web IDE
                             │
                  HTTPS / WebSocket
                             │
                             ▼
                         Nginx
                             │
                             ▼
                    Node.js Gateway
                             │
             ┌───────────────┼───────────────┐
             │               │               │
             ▼               ▼               ▼
        PostgreSQL         Redis       Collaboration
             │               │
             │               ▼
             │          Python Evaluator
             │               │
             │              gRPC
             │               │
             │               ▼
             │        C++ Sandbox Runtime
             │               │
             │               ▼
             │       Docker Isolation
             │               │
             │               ▼
             │        User Program
             │
             └──────── Execution History
```

The architecture intentionally separates:

* user interaction;
* API management;
* project persistence;
* execution orchestration;
* queue management;
* sandbox execution;
* real-time collaboration;
* security enforcement;
* observability.

---

# Why CodeForge Cloud

Traditional local development environments execute programs directly on the developer's machine.

An online compiler introduces a much harder problem:

> **How can arbitrary user code be executed safely, predictably, and efficiently without compromising the host system or other users?**

CodeForge Cloud approaches this problem using multiple isolated layers.

```text
User Source Code
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
Immutable Execution Snapshot
       │
       ▼
Resource Policy
       │
       ▼
Queue
       │
       ▼
Evaluator
       │
       ▼
Sandbox
       │
       ├── Linux Namespaces
       ├── cgroups
       ├── seccomp
       ├── Capability Restrictions
       ├── Read-Only Filesystem
       ├── Network Isolation
       ├── Process Limits
       ├── Memory Limits
       ├── CPU Limits
       └── Execution Timeout
       │
       ▼
Compile + Execute
       │
       ▼
Result
```

The project is therefore not just a compiler frontend.

It is a study and implementation of:

* distributed systems;
* secure code execution;
* container isolation;
* asynchronous job processing;
* real-time collaboration;
* API design;
* database architecture;
* service-to-service communication;
* observability;
* automated testing.

---

# Key Features

## Browser-Based IDE

The frontend provides a modern development environment with:

* Monaco Editor;
* syntax highlighting;
* file explorer;
* project workspace;
* terminal output;
* execution controls;
* execution status;
* collaboration indicators;
* real-time updates.

---

## Project & File Management

Users can create and manage projects containing:

```text
Project
 ├── src/
 │   ├── main.cpp
 │   └── utils.cpp
 │
 ├── include/
 │   └── utils.hpp
 │
 └── README.md
```

The platform supports:

* files;
* directories;
* file hierarchy;
* file content;
* project membership;
* roles;
* project-level authorization.

---

## Real-Time Collaboration

Multiple users can work inside the same project.

The collaboration layer supports concepts such as:

* WebSocket connections;
* project rooms;
* presence;
* cursor updates;
* selection updates;
* file updates;
* collaboration events;
* conflict-aware synchronization.

The architecture is designed to support **CRDT-based collaboration** as the system matures.

---

## Secure Code Execution

User code is treated as hostile input.

Code is never executed:

```text
✗ directly on the host
✗ inside the Gateway
✗ inside the Evaluator process
✗ with unrestricted privileges
```

Instead:

```text
User Code
   ↓
Immutable Snapshot
   ↓
Execution Queue
   ↓
Worker
   ↓
Sandbox
   ↓
Restricted Container
   ↓
Compile
   ↓
Execute
```

---

## Resource Control

Each execution can have limits for:

* CPU;
* memory;
* wall-clock time;
* process count;
* disk usage;
* output size;
* source size.

This protects the infrastructure from:

* infinite loops;
* fork bombs;
* memory exhaustion;
* CPU exhaustion;
* output flooding;
* disk exhaustion;
* abusive workloads.

---

## Execution History

Every execution is represented as a persistent job.

The system records information such as:

* execution ID;
* job ID;
* project;
* user;
* language;
* status;
* source snapshot;
* resource policy;
* timestamps;
* compilation time;
* execution time;
* memory usage;
* stdout;
* stderr;
* exit code;
* resource violations.

---

# Architecture

The platform uses a distributed service architecture.

```text
                         ┌───────────────────┐
                         │    Web Browser    │
                         │    React / IDE    │
                         └─────────┬─────────┘
                                   │
                         HTTPS / WebSocket
                                   │
                                   ▼
                         ┌───────────────────┐
                         │      Nginx        │
                         │   Reverse Proxy   │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │   Node.js Gateway │
                         │    API Boundary   │
                         └──────┬───────┬────┘
                                │       │
                                │       └──── WebSocket
                                │
                     ┌──────────┴──────────┐
                     │                     │
                     ▼                     ▼
              ┌────────────┐        ┌────────────┐
              │ PostgreSQL │        │    Redis   │
              │   Source   │        │ Queue/Bus  │
              │  of Truth  │        └─────┬──────┘
              └────────────┘              │
                                          ▼
                                ┌─────────────────┐
                                │ Python Evaluator│
                                └────────┬────────┘
                                         │
                                        gRPC
                                         │
                                         ▼
                                ┌─────────────────┐
                                │   C++ Sandbox   │
                                └────────┬────────┘
                                         │
                                         ▼
                                ┌─────────────────┐
                                │ Docker Runtime  │
                                │                 │
                                │ Namespaces      │
                                │ cgroups         │
                                │ seccomp         │
                                │ Capabilities    │
                                │ Network Isolation│
                                └────────┬────────┘
                                         │
                                         ▼
                                   User Program
```

### Service Responsibilities

| Service          | Responsibility                                   |
| ---------------- | ------------------------------------------------ |
| React Web IDE    | User interface and development workspace         |
| Nginx            | Reverse proxy and routing                        |
| Node.js Gateway  | Public API, auth, projects, files, collaboration |
| PostgreSQL       | Durable application state                        |
| Redis            | Queue, cache, pub/sub, transient state           |
| Python Evaluator | Validation, orchestration, scheduling            |
| Worker           | Execution job processing                         |
| C++ Sandbox      | Low-level secure execution runtime               |
| Docker           | Container isolation                              |
| gRPC             | Internal service communication                   |

---

# Execution Pipeline

Code execution follows an asynchronous pipeline.

```text
┌───────────────┐
│   React IDE   │
└───────┬───────┘
        │
        │ POST /api/v1/executions
        ▼
┌────────────────┐
│ Node Gateway   │
└───────┬────────┘
        │
        ├── Authenticate
        ├── Authorize
        ├── Validate
        └── Snapshot
                │
                ▼
        ┌────────────────┐
        │ Execution Job  │
        └───────┬────────┘
                │
                ▼
          ┌───────────┐
          │   Redis   │
          │   Queue   │
          └─────┬─────┘
                │
                ▼
        ┌────────────────┐
        │ Python Worker  │
        └───────┬────────┘
                │
               gRPC
                │
                ▼
        ┌────────────────┐
        │  C++ Sandbox   │
        └───────┬────────┘
                │
                ▼
        ┌────────────────┐
        │ Docker Runtime │
        └───────┬────────┘
                │
                ▼
          Compile Source
                │
                ▼
           Run Program
                │
                ▼
       stdout / stderr / exit
                │
                ▼
       Execution Result
                │
                ▼
           PostgreSQL
                │
                ▼
             Gateway
                │
                ▼
             React IDE
```

---

# Immutable Execution Snapshots

A critical CodeForge Cloud design rule is:

> **An execution runs against an immutable snapshot, never against mutable live project state.**

When the user presses **Run**:

```text
Live Project
     │
     ▼
Snapshot Capture
     │
     ▼
Snapshot Validation
     │
     ▼
Snapshot Persistence
     │
     ▼
Execution Queue
     │
     ▼
Sandbox Materialization
     │
     ▼
Compile
     │
     ▼
Execute
     │
     ▼
Result
```

This provides:

* reproducibility;
* deterministic execution input;
* historical traceability;
* protection from concurrent file changes;
* easier debugging;
* execution auditing.

Snapshots use SHA-256 hashing to provide integrity information.

---

# Execution States

A normal execution follows:

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

Terminal states are immutable.

---

# Security Model

Security is one of the most important parts of CodeForge Cloud.

The platform assumes:

> **Every user program may be malicious.**

The security architecture therefore uses defense in depth.

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
                Resource Policy
                        │
                        ▼
              Immutable Snapshot
                        │
                        ▼
                    Sandbox
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
      Namespace       cgroup        seccomp
          │             │             │
          └─────────────┼─────────────┘
                        ▼
               Capability Limits
                        │
                        ▼
              Filesystem Isolation
                        │
                        ▼
                Network Isolation
                        │
                        ▼
                  User Process
```

---

# Sandbox Security

The sandbox must not expose:

```text
Docker Socket
Host Filesystem
Host Network
PostgreSQL
Redis
Gateway Internal APIs
Other Project Data
Host Privileges
```

Initial execution environments use:

* Docker containers;
* Linux namespaces;
* cgroups;
* seccomp;
* restricted capabilities;
* read-only filesystem where practical;
* temporary workspaces;
* network disabled by default;
* CPU limits;
* memory limits;
* process limits;
* output limits;
* execution timeouts.

---

# Technology Stack

## Frontend

```text
React
TypeScript
Vite
Monaco Editor
Tailwind CSS
Lucide
```

## Gateway

```text
Node.js
TypeScript
Fastify / Express
REST
WebSocket / Socket.IO
```

## Evaluator

```text
Python
FastAPI
gRPC
Redis
```

## Sandbox

```text
C++
Docker
Linux namespaces
cgroups
seccomp
```

## Data

```text
PostgreSQL
Redis
```

## Infrastructure

```text
Docker Compose
Nginx
gRPC
Protocol Buffers
```

## Future

```text
Kubernetes
Network Policies
gVisor
Kata Containers
MicroVMs
mTLS
Container Signing
SBOM
Advanced Observability
```

---

# Repository Structure

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
└── README.md
```

---

# Documentation

The complete project documentation is available in the [`docs/`](./docs/) directory.

## Documentation Index

| Document             | Link                                                            |
| -------------------- | --------------------------------------------------------------- |
| Product Requirements | [01-product-requirements.md](./docs/01-product-requirements.md) |
| Architecture         | [02-architecture.md](./docs/02-architecture.md)                 |
| Data Model           | [03-data-model.md](./docs/03-data-model.md)                     |
| API Reference        | [04-api-reference.md](./docs/04-api-reference.md)               |
| Roadmap & Phases     | [05-roadmap-and-phases.md](./docs/05-roadmap-and-phases.md)     |
| Development Guide    | [06-development-guide.md](./docs/06-development-guide.md)       |
| Security             | [07-security.md](./docs/07-security.md)                         |
| Gap Analysis         | [08-gap-analysis.md](./docs/08-gap-analysis.md)                 |
| Testing Strategy     | [09-testing-strategy.md](./docs/09-testing-strategy.md)         |
| Glossary             | [10-glossary.md](./docs/10-glossary.md)                         |
| Documentation Index  | [docs/README.md](./docs/README.md)                              |

### Recommended Reading Order

```text
Product Requirements
        ↓
Architecture
        ↓
Data Model
        ↓
API Reference
        ↓
Roadmap
        ↓
Development Guide
        ↓
Security
        ↓
Gap Analysis
        ↓
Testing Strategy
        ↓
Glossary
```

---

# Project Roadmap

CodeForge Cloud is planned as a phased platform.

| Phase    | Area                           |
| -------- | ------------------------------ |
| Phase 0  | Project Foundation             |
| Phase 1  | Core Data Layer                |
| Phase 2  | Authentication & Authorization |
| Phase 3  | Project & File Management      |
| Phase 4  | Execution API                  |
| Phase 5  | Python Evaluator               |
| Phase 6  | C++ Sandbox                    |
| Phase 7  | End-to-End Execution           |
| Phase 8  | Web IDE                        |
| Phase 9  | Real-Time Collaboration        |
| Phase 10 | Observability                  |
| Phase 11 | Security Hardening             |
| Phase 12 | Performance & Load Testing     |
| Phase 13 | Production Deployment          |
| Phase 14 | Advanced Platform              |

See the complete roadmap in [05-roadmap-and-phases.md](./docs/05-roadmap-and-phases.md).

---

# MVP Scope

The initial MVP focuses on:

* user authentication;
* project creation;
* project files;
* C++ code editing;
* execution API;
* immutable execution snapshots;
* asynchronous execution;
* secure sandbox;
* execution history;
* Web IDE;
* basic WebSocket functionality.

The intended MVP journey is:

```text
Register
   ↓
Login
   ↓
Create Project
   ↓
Create main.cpp
   ↓
Write C++
   ↓
Run
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
Result
   ↓
Terminal
```

---

# Development

## Prerequisites

Recommended development environment:

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
PostgreSQL client
Redis client
```

---

## Clone

```bash
git clone <repository-url>
cd Online-Code-Compiler
```

---

## Start Infrastructure

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

Remove volumes:

```bash
docker compose down -v
```

---

## Frontend

```bash
cd frontend/web-ide

npm install
npm run dev
```

Build:

```bash
npm run build
```

Lint:

```bash
npm run lint
```

---

## Node.js Gateway

```bash
cd gateway/node

npm install
npm run dev
```

Build:

```bash
npm run build
```

Test:

```bash
npm test
```

---

## Python Evaluator

```bash
cd evaluator/python

python3 -m venv .venv
source .venv/bin/activate

pip install -r requirements.txt

uvicorn app.main:app --reload
```

Tests:

```bash
pytest
```

---

## C++ Sandbox

```bash
cd sandbox/cpp

cmake -S . -B build
cmake --build build
ctest --test-dir build
```

---

# Testing

CodeForge Cloud uses multiple levels of testing.

```text
                     Testing
                        │
       ┌────────────────┼────────────────┐
       │                │                │
       ▼                ▼                ▼
     Unit          Integration          E2E
       │                │                │
       └────────────────┼────────────────┘
                        ▼
                    Security
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
          Sandbox     AuthZ     Isolation
                        │
                        ▼
                   Performance
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
            Load      Stress      Soak
```

Root test commands:

```bash
make test
make test-unit
make test-integration
make test-e2e
make test-security
make test-load
make coverage
```

Detailed strategy:

[09-testing-strategy.md](./docs/09-testing-strategy.md)

---

# Security Principles

CodeForge Cloud follows these non-negotiable rules:

```text
1. Browser input is untrusted.

2. User source code is hostile input.

3. Gateway never executes user code.

4. User code never executes directly on the host.

5. Every execution uses an immutable snapshot.

6. Every execution has resource limits.

7. Every execution has a timeout.

8. Sandbox networking is disabled initially.

9. User workloads receive no Docker socket.

10. User workloads receive no host filesystem.

11. User workloads cannot directly access PostgreSQL.

12. User workloads cannot directly access Redis.

13. Authorization is enforced server-side.

14. UUIDs do not replace authorization.

15. PostgreSQL is the durable source of truth.

16. Redis is not the durable source of truth.

17. Security failures fail closed.

18. Important operations are observable.

19. Security-sensitive changes require security testing.

20. Important bugs become regression tests.
```

Full security model:

[07-security.md](./docs/07-security.md)

---

# Design Principles

## Security by Design

Security is built into the architecture from the beginning.

## Least Privilege

Services and workloads receive only the permissions they require.

## Separation of Concerns

Each service has a clearly defined responsibility.

## Contract First

APIs and service interfaces are explicitly defined.

## Immutable Execution

Execution uses a fixed snapshot instead of mutable project state.

## Disposable Workloads

Sandbox environments are temporary and isolated.

## Observable by Default

Important operations should produce logs, metrics, and traces.

## Fail Closed

Security failures should deny access rather than silently bypass controls.

## Defense in Depth

No single security mechanism is considered sufficient.

---

# Database Architecture

PostgreSQL stores durable application state.

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

Execution snapshots and resource policies are stored as part of the execution model.

Redis is responsible for transient/distributed workloads such as:

```text
queues
cache
pub/sub
presence
rate limiting
transient execution state
```

Database documentation:

[03-data-model.md](./docs/03-data-model.md)

---

# API Architecture

Public APIs are versioned under:

```text
/api/v1
```

Example endpoints:

```http
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
GET    /api/v1/auth/me

POST   /api/v1/projects
GET    /api/v1/projects
GET    /api/v1/projects/{project_id}
PATCH  /api/v1/projects/{project_id}
DELETE /api/v1/projects/{project_id}

POST   /api/v1/projects/{project_id}/files
GET    /api/v1/projects/{project_id}/files
GET    /api/v1/projects/{project_id}/files/{file_id}
PATCH  /api/v1/projects/{project_id}/files/{file_id}
DELETE /api/v1/projects/{project_id}/files/{file_id}

POST   /api/v1/executions
GET    /api/v1/executions/{execution_id}
GET    /api/v1/executions/{execution_id}/result
POST   /api/v1/executions/{execution_id}/cancel
```

Internal services communicate using:

```text
gRPC + Protocol Buffers
```

Real-time communication uses:

```text
WebSocket
```

Full API specification:

[04-api-reference.md](./docs/04-api-reference.md)

---

# Collaboration

The collaboration system is designed around real-time events.

Example events:

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

execution:queued
execution:started
execution:compiling
execution:running
execution:completed
execution:failed
```

The long-term collaboration model is designed to support CRDT-based synchronization.

---

# Observability

The platform is designed to expose operational information such as:

* request IDs;
* trace IDs;
* execution IDs;
* job IDs;
* service names;
* latency;
* status codes;
* queue latency;
* execution duration;
* resource usage;
* failure reasons.

Sensitive data must not be logged by default.

Never log:

```text
Passwords
Access Tokens
Session Tokens
Secrets
Private Credentials
Raw Source Code
```

unless explicitly required under a controlled security/debugging process.

---

# Failure Handling

CodeForge Cloud is designed to handle failures explicitly.

Potential failures include:

```text
Gateway Failure
Database Failure
Redis Failure
Evaluator Failure
Worker Failure
Sandbox Failure
Compiler Failure
Timeout
Resource Violation
Network Failure
WebSocket Disconnect
Duplicate Request
Cancellation Race
Container Failure
```

The platform should avoid silent recovery that violates security or data consistency.

---

# Performance Goals

Performance is treated as a measurable engineering concern.

Important metrics include:

```text
API Latency
Queue Latency
Snapshot Creation Time
Evaluator Latency
Sandbox Startup Time
Compilation Time
Execution Time
Result Persistence Time
WebSocket Latency
Concurrent Connections
Concurrent Executions
CPU Utilization
Memory Utilization
```

Performance should be measured through:

```text
Load Tests
Stress Tests
Soak Tests
Benchmark Tests
```

---

# Production Direction

The initial development environment uses:

```text
Docker Compose
```

The future production architecture can evolve toward:

```text
                         Internet
                            │
                            ▼
                         Ingress
                            │
                            ▼
                     API Gateway Layer
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          Gateway       Collaboration    Auth
             │
             ▼
       Execution Service
             │
             ▼
        Message Queue
             │
             ▼
       Worker Pool
             │
             ▼
      Sandbox Scheduler
             │
      ┌──────┼──────┐
      ▼      ▼      ▼
   Sandbox Sandbox Sandbox
      │      │      │
      └──────┼──────┘
             ▼
       Result Storage
```

Potential production technologies include:

* Kubernetes;
* container orchestration;
* NetworkPolicies;
* mTLS;
* dedicated worker pools;
* autoscaling;
* gVisor;
* Kata Containers;
* microVMs;
* signed container images;
* SBOM;
* centralized observability;
* SIEM integration.

---

# Project Status

**Current status:** In Development

The documentation and architecture foundation is established.

```text
[✓] Product Requirements
[✓] Architecture
[✓] Data Model
[✓] API Reference
[✓] Roadmap
[✓] Development Guide
[✓] Security Architecture
[✓] Gap Analysis
[✓] Testing Strategy
[✓] Glossary
[✓] Documentation Index
[ ] Full Production Implementation
[ ] Production Security Hardening
[ ] Production Deployment
```

The checked documentation items represent the initial documentation baseline, not necessarily completed implementation.

---

# Contributing

Contributions should follow the documented development and security practices.

Before submitting changes:

```bash
git status
git diff

make test

git status
```

Recommended commit format:

```text
<type>: <description>
```

Examples:

```text
feat: add execution API
feat: implement sandbox worker
fix: handle execution timeout
security: restrict sandbox capabilities
test: add snapshot integrity tests
docs: update API reference
refactor: simplify evaluator queue
chore: update dependencies
```

Before modifying architecture, API contracts, database structures, or security boundaries, review the relevant documentation.

Start with:

[06-development-guide.md](./docs/06-development-guide.md)

---

# Documentation Map

```text
docs/
│
├── 01-product-requirements.md
│       Product vision and requirements
│
├── 02-architecture.md
│       System architecture
│
├── 03-data-model.md
│       Database and data lifecycle
│
├── 04-api-reference.md
│       REST / WebSocket / gRPC contracts
│
├── 05-roadmap-and-phases.md
│       Development roadmap
│
├── 06-development-guide.md
│       Developer setup and workflow
│
├── 07-security.md
│       Security architecture and threat model
│
├── 08-gap-analysis.md
│       Remaining implementation and production gaps
│
├── 09-testing-strategy.md
│       Testing and quality assurance
│
├── 10-glossary.md
│       Terminology
│
└── README.md
        Documentation navigation
```

**Start here:** [Documentation Index](./docs/README.md)

---

# License

This project is licensed under the MIT License.

See [LICENSE](./LICENSE) for details.

---

# Author

**Adarsh Kumar**

CodeForge Cloud is designed and developed as a portfolio-grade distributed systems project focused on:

```text
Cloud Architecture
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
System Security
Testing
Observability
```

---

# Final Vision

CodeForge Cloud aims to evolve into a complete cloud-native development platform where users can:

```text
Create Project
      ↓
Write Code
      ↓
Collaborate
      ↓
Run Code
      ↓
Compile in Secure Sandbox
      ↓
Inspect Results
      ↓
Review Execution History
      ↓
Scale Across Distributed Workers
```

The central engineering principle is:

> **Build the system as if the code running inside it cannot be trusted.**

And the broader project principle is:

> **Build it. Secure it. Test it. Observe it. Scale it.**
